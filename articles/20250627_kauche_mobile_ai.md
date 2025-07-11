---
title: "カウシェ Mobile Team のAI活用最前線〜考え方・実践・未来〜"
emoji: "⚔️"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["ai", "llm", "chatgpt", "ios", "android"]
published: true
publication_name: "kauche"
---

こんにちは！カウシェで Mobile Team の Engineering Manager を担当している [@akifumi](https://x.com/akifumifukaya) です。
本記事では、カウシェのモバイル領域におけるAI活用の考え方、直近の取り組み、そして今後の展望についてご紹介いたします。

## カウシェ Mobile Team のAI活用の考え方

カウシェ Mobile Team では、AIを積極的に活用して **「一人ひとりの開発生産性を最大限まで高める」** ことを目指しています。

モバイル開発の世界では、Android と iOS それぞれ別の専任エンジニアが担当するのが一般的です。しかし私たちは、次のような目的を達成するために、あえて異なるアプローチを選択しました。
- お客様にいち早く価値を届けること
- 一人のエンジニアが両OSを開発することで、OS間の機能差異を極力減らし、サービスの安定性を高めること
- 開発時に関わる人数を最適化し、認知負荷やコミュニケーションコストの軽減して、全体の開発効率を高めること

このような目的を実現するため、カウシェ Mobile Team では Android Engineer / iOS Engineer と区別せず、全員を **「Mobile Engineer」** と呼んでいます。
「Mobile Engineer」が開発生産性を最大限まで高められるように、AIを単なる補助ツールではなく、**エンジニアの能力を拡張する「右腕」** のような存在と位置づけています。
日々の開発のコーディングの効率化はもちろん、ドキュメント生成、壁打ち相手など、開発プロセスに深く浸透させることで、少人数でも高い成果をあげられる状況を目指しています。

また、AIを活用することでOSの垣根を越えて効率的に開発しやすい環境の整備も同時に進めています。

## 直近の取り組み

その上で、直近では次のような取り組みを実施しています。

### **AIコーディングアシスタントを全員に導入**

Mobile Engineer 全員に、AIコーディングアシスタントを導入し、日々の業務に取り入れています。
使用ツールは、**Cline**、**Cursor**、**Claude Code**、**Codex** など多岐にわたります。
各メンバーに約2種類程のツールを付与し、各業務の中でどのAIアシスタントが良いか、どんな使い方が良いか、を模索しながら使用しています。

日々のコーディングはもちろん、設計、ドキュメント生成など、日常的な開発プロセスの中でAIを取り入れるようにしています。

### **モノレポ化の実施**

これまで Android（`kauche-android`）と iOS（`kauche-ios`）をプラットフォームごとに分けてコードを管理していました。
AIをより効率的に活用するために、`kauche-mobile` というひとつのレポジトリに統合しました。

具体的には、モノレポ化によって以下の効果を狙い実施しました。
- **AI Rules の共通化**
  - 両OSの AI Rules を一元管理し、チーム内でナレッジやルールを共有できる
- **AIによるコード理解幅の拡大**
  - Android/iOS 両方のコードが一箇所にあるため、AIが両OSの実装を横断的に理解した上でコード生成ができ、アウトプットの精度と汎用性の向上が期待できる
- **クロスプラットフォームでの開発効率化**
  - Android ↔ iOS 相互にコード変換が可能
  - また機能開発において、Android/iOS 両OSの同時開発の可能性
- **コンテキストスイッチの低減**
  - Mobile Engineer が1つの環境で実装やレビューを行えるため、作業環境の切り替えによる認知負荷の軽減

### **AI情報共有会の開催**

Mobile Team では、毎週の定例ミーティング内に **AI活用に関する共有時間** を設けています。
この時間では、メンバーが実際に試して効果を感じたAI活用の事例や、PoC（技術検証）の結果を共有したりしています。
実際の実務レベルでの活用事例が共有されることが多く、**「自分も使ってみよう」** と思えるようなきっかけが生まれる場となっています。
そのため、チーム全体でAI活用の底上げができるようになっています。

共有された知見の中でも文章として残しておいた方が良いものは、Notion上の **「AI 活用 Tips」** に溜めていき、チーム内外のメンバーが参考にしやすいように蓄積しています。
さらに、開発中で活用できそうなものは、前述の **AI Rules** に組み込み、日々の開発に取り入れられています。

## LLM 活用度の変化

上記の活動を通じて、カウシェ Mobile Team のLLMを活用した開発に変化が見られました。
LLMを活用して作成されたPR作成率は、3月はもちろん 0% だったのですが、4月: 37%, 5月: 50%, 6月: 61% と着実に活用度が上昇しております。
![LLM起点PR作成率](/images/20250627_kauche_mobile_ai/2025-06-26_9.58.54.png)
6月に入ってからは、1週間で作成されたPRの内 **73%** がLLM起点で作成された週もあり、開発プロセスにAIが浸透してきています。

## 今後の展望

今後の展望として、以下のようなことを考えています。

- **AI活用によるモバイル開発のさらなる加速化**
  - Android/iOS 両OSのコードをAIで横断的に開発し、コード変換や並列開発によって、さらに一段と高い開発生産性の向上に挑戦します
  - Gemini in Android Studio, ChatGPT with Xcode など、各プラットフォームの純正IDEに搭載されたAI機能も積極的に調査・導入を進めていきます
  - mcp server（Figma, Notion, Slack など）と連携し、機能仕様・UI仕様を理解した上でのUIコード自動生成の検証・模索を行います
- **AI×自動テストの強化**
  - UIテストコードの自動生成・自己修復による、開発の品質向上とテスト効率化を図ります
  - 手作業によるテスト負担を軽減しつつ、テクノロジーで品質担保できる体制を整えていきます
- **1 Team : 1 Mobile Engineer 体制の実現**
  - 1 Mobile Engineer が Android/iOS 両OSの開発ができるようになっていくことで、1 Team の機能開発を 1 Mobile Engineer が担えるような体制を目指します
  - Mobile Engineer を極端に増員しなくても、より多くの機能をよりスピーディに開発できるようになり、結果としてお客様へ価値を最速で提供できるようになると考えています

## まとめ

カウシェ Mobile Team でのAI活用における **考え方・直近の取り組み・今後の展望** をご紹介させていただきました。
少しでもみなさまのお役に立てたら嬉しいです。

## 最後に
### イベントの案内
以下のイベントが開催予定です。
テックイベントは私も会場に行く予定ですので、直接お話したい方がいましたら、是非ご参加をお願いいたします。

https://kauche.connpass.com/event/360073/
https://kauche.connpass.com/event/358309/
https://kauche.connpass.com/event/359790/

### もっと話したい方へ
今日の記事を見ていただき、質問したい方、もう少し深堀って話したいという方、「新しい生活圏のカタチをつくる」をテクノロジーで実現する！というテーマに興味を持っていただいた方、YOUTRUSTカジュアル面談も公開していますので、ぜひお話しましょう！
https://youtrust.jp/recruitment_posts/e5c6604159843f153ac24d94889043a2

