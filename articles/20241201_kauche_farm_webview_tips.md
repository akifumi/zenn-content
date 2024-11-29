---
title: "カウシェファームの裏側 : WebとモバイルをつなぐJavaScript Interfaceの活用例"
emoji: "🧑‍🌾"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["ios", "android", "javascript"]
published: false
---

こんにちは。株式会社カウシェの [@akifumi](https://x.com/akifumifukaya) です。
本記事では、カウシェ内で作物を育てることができるゲーム「カウシェファーム」の裏側の仕組みについてご説明します。

## はじめに

カウシェアプリの中に、育てた作物が無料でもらえる「カウシェファーム」というゲーム機能を提供しています。
カウシェファームについて: https://prtimes.jp/main/html/rd/p/000000053.000064598.html

カウシェアプリはネイティブアプリとして Swift, Kotlin を用いて各プラットフォームごとに開発されていますが、カウシェファームは Web で開発されており、WebView を使用してアプリ内に表示されています。

Webとアプリ間で情報のやりとりが必要な場面があり、以下の方法を用いて実現しています。
- Internal URL Scheme
- JavaScript Interface

次の章では、これらの方法について具体的な活用例を交えて解説します。

## Webとアプリ間の通信方法

Webとアプリ間の通信は、以下の2つの方法で実現しています。
1. Internal URL Scheme
2. JavaScript Interface

### Internal URL Scheme

URL Scheme はアプリ開発者の方々にとって、馴染みがあるものかと思います。
一般的な URL Scheme は、他アプリやウェブブラウザから URL を指定してアプリを呼び出す方法として利用されています。

今回、Internal URL Scheme と定義している仕組みは、他アプリやウェブブラウザから呼び出すことはできない（外部公開されていない）が、特定の WebView からのみ URL を指定してアプリの任意の機能を呼び出せる仕組みを意味しています。
カウシェファームでは、Internal URL Scheme を定義することで、Web からアプリの任意の機能呼び出せるようにしています。

以下は URL Scheme の例です。
- `kauche://` : 外部公開している URL Scheme
- `kauche-internal://` : 特定の WebView 内のみ使用できる Internal URL Scheme

### JavaScript Interface

カウシェファームではWebとアプリ間の通信方法として、前述の Internal URL Scheme 以外に JavaScript Interface を活用しています。

この機能は、アプリ側に JavaScript 関数の受け取り処理を定義し、Web 側からは定義された JavaScript 関数を実行することで、Webとアプリ間の通信を行う機能です。
この機能を使うことで、Webからアプリの機能を呼び出したり、実行結果をアプリからWebに返すことが可能になります。

JavaScript Interface の特徴
- Webからアプリに対して、処理の実行ができる
- JavaScript 関数の引数として、複数のパラメータをWebからアプリに渡すことができる
- アプリの処理実行結果を JavaScript 関数で返せる

上記の特徴も踏まえ、カウシェファームでは Internal URL Scheme と JavaScript Interface を以下のように使い分けています。

◆主な使い分け
- Web → アプリ への一方的な処理実行
    - → Internal URL Scheme を利用
    - 使用例）
        - Webからアプリに対して任意の処理を実行
        - Webからアプリの特定画面に遷移
- Web ↔ アプリ 間での双方向な処理
    - → JavaScript Interface を利用
    - 使用例）
        - Webからアプリに対して任意の処理を実行し、実行結果をアプリからWebに返す

## 注意点

JavaScript Interface はWebとアプリ間の処理をつなぐのに便利な一方でいくつか注意点もあります。

### 返り値の型に制限

iOS の場合、`WKScriptMessageHandlerWithReply` protocol に準拠することで、JavaScript からメッセージを受け取り、`Any` 型で任意のオブジェクトを返すことができます。
例えば、iOS が key value pair のオブジェクトを返すと、JavaScript 側ではそのまま値を受け取ることができます。
一方で、Android の `@JavascriptInterface` アノテーションを付与した関数の返り値は、Int, Boolean といった Primitive 型か String しか対応していません。
そのため、iOS と Android で仕様が異なり、JavaScript の受け取り側で処理を分岐する必要があります。
カウシェファームの場合、Android で key value pair 等で複数の値を返す必要がある際は、JSON 形式の文字列を作成して返し、JavaScript 側では文字列を受け取った後で JSON 型に変換して利用しています。

### Android の非同期処理がサポートされていない

クライアント側で非同期処理を行い、その結果を Web に返したいケースがあると思います。
iOS の場合、前述と同様に `WKScriptMessageHandlerWithReply` protocol に準拠することで、JavaScript からメッセージを受け取った後に非同期処理を行った結果を返すことができます。
JavaScript 側では `Promise` を用いることで、非同期処理の結果を待ってから後続の処理を実行することができます。
一方で、Android の JavaScript Interface は非同期処理に対応されていません。
Android 側で非同期処理を行い、その結果を用いて JavaScript 側で後続処理を行いたい場合、以下のような工夫が必要となります。

1. JavaScript Interface もしくは URL Scheme で非同期処理の開始を通知
2. Android 側で非同期処理を実行
3. [fun evaluateJavascript(script:String, resultCallback:ValueCallback<String!>?):Unit](https://developer.android.com/reference/kotlin/android/webkit/WebView#evaluatejavascript) で JavaScript 関数を実行して、非同期処理の結果を渡す

こちらも iOS, Android でできることが異なるため、Webとアプリ間の通信フローの仕様を調整する必要があります。

## まとめ

カウシェアプリ内にあるカウシェファームというゲーム機能は Web で開発されており、アプリ内では WebView を用いて表示されています。
その際、Webとアプリ間で通信を行う必要があり、カウシェファームでは外部に公開しない Internal URL Scheme と JavaScript Interface という仕組みを組み合わせて実現しています。
一方で注意点も存在するため、知見を共有させていただきました。
アプリとWebを組み合わせるという機会は多くないかもしれませんが、サービス開発の参考になりましたら幸いです。

