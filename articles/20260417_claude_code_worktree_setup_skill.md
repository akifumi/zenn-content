---
title: Claude Code の worktree で設定ファイルを一元管理するスキルを作成した
emoji: 🌲
type: tech
topics:
  - claudecode
  - git
  - worktree
  - ai
  - productivity
published: true
---

こんにちは！[akifumi](https://x.com/akifumifukaya) です。

Claude Code で複数の機能ブランチを並行して開発していると、ブランチの切り替えがストレスになります。

`git stash` してブランチを切り替えて、作業が終わったら戻して `git stash pop` して...。環境を切り替えるたびにこれをやっていると、スイッチングコストが非常に高まります。

Git の worktree を使えば、複数のブランチ環境を同時にチェックアウトできます。Claude Code にも組み込みの `/worktree` コマンドがあり、手軽に並列開発環境を作れます。

ただ、実際に運用してみるとひとつ困ることがありました。`.env` などの gitignore されたファイルが worktree 間で同期されないのです。

組み込みの `/worktree` は `.worktreeinclude` で gitignore ファイルをコピーできますが、これは作成時の一度きりのコピーです。後から `.env` に新しい環境変数を追加しても、既存の worktree には反映されません。開発中に環境変数が増えることはよくあるので、その都度全 worktree の `.env` を手動で更新するのは辛い。

そこで、`shared_config/` に設定ファイルの実体を一元管理し、各 worktree からはシンボリックリンクで参照する仕組みを、Claude Code のカスタムスキルとして作りました。`shared_config/.env` を編集するだけで全環境に一括反映されます。

## `/worktree` と `.worktreeinclude` の不足点

まず前提として、Claude Code には組み込みの worktree 機能があります。

```bash
# Claude Code の組み込みコマンド
/worktree
```

これだけで `.claude/worktrees/` 配下に worktree が作られ、並列開発が始められます。便利です。

gitignore されたファイル（`.env` など）を worktree にコピーしたい場合は、プロジェクトルートに `.worktreeinclude` を置きます。

```
# .worktreeinclude
.env
.env.local
.node-version
```

ただし、これは **worktree 作成時に一度だけコピーされる**仕組みです。

```
1. worktree を3つ作成（.env がコピーされる）
2. 開発が進み、.env に NEW_API_KEY=xxx を追加
3. worktree-a の .env → 古いまま（NEW_API_KEY がない）
4. worktree-b の .env → 古いまま
5. worktree-c の .env → 古いまま
```

開発中に環境変数が増えるたびに全 worktree の `.env` を手動で更新する必要があります。worktree が3つ4つあると地味に辛い。

## shared_config + シンボリックリンクで解決する

自作スキルでは、`shared_config/` ディレクトリに設定ファイルの実体を配置し、各 worktree からはシンボリックリンクで参照する設計にしています。

```
$HOME/.claude/worktrees/<org>/<repo>/
├── shared_config/           # 設定ファイルの実体はここだけ
│   ├── .env
│   ├── .node-version
│   └── ...
├── worktree-a/               # worktree 1
│   ├── .env -> ../shared_config/.env    # シンボリックリンク
│   ├── .env -> ../shared_config/.node-version    # シンボリックリンク
│   └── ...
├── worktree-b/               # worktree 2
│   └── ...
└── worktree-c/               # worktree 3
    └── ...
```

`.env` に新しい環境変数を追加したいときは `shared_config/.env` を編集するだけ。全 worktree に一括で反映することができます。

```
shared_config/.env          ← ここだけ編集すれば OK
worktree-a/.env → ../shared_config/.env   ✓ 即反映
worktree-b/.env → ../shared_config/.env   ✓ 即反映
worktree-c/.env → ../shared_config/.env   ✓ 即反映
```

## `/worktree-setup` スキルを作成

上記を実現するカスタムスキル `/worktree-setup` を作りました。対象リポジトリ内で実行すると、対話的に並列開発環境が構築されます。

```
❯ /worktree-setup akifumi/my-repo 3

共有したい設定ファイルはありますか？
> .env, .node-version

✓ 作成完了

| 環境       | パス                                          | ブランチ              |
|-----------|-----------------------------------------------|----------------------|
| worktree-a | ~/.claude/worktrees/akifumi/my-repo/worktree-a | worktree/worktree-a |
| worktree-b | ~/.claude/worktrees/akifumi/my-repo/worktree-b | worktree/worktree-b |
| worktree-c | ~/.claude/worktrees/akifumi/my-repo/worktree-c | worktree/worktree-c |
```

やっていることは以下の3つです。

1. **worktree の作成** — 指定した数だけ `$HOME/.claude/worktrees/<org>/<repo>/` 配下に worktree を作成
2. **共有ファイルのコピー** — 指定した `.env` などを `shared_config/` にコピー
3. **シンボリックリンクの作成** — 各 worktree から `shared_config/` への symlink を張る

あとは各 worktree で Claude Code を起動するだけです。

```bash
cd ~/.claude/worktrees/akifumi/my-repo/worktree-a && claude
```

普段の `git checkout` によるブランチ切り替えと異なり、**作業中のファイルがそのまま残った状態で別ブランチの作業に移れます**。stash も不要です。

### シンボリックリンクの作成

各 worktree から `shared_config/` のファイルに対してシンボリックを作成しています。

```bash
for name in worktree-a worktree-b worktree-c; do
  WORKTREE_PATH="$WORKTREE_BASE/$name"

  # ルート直下のファイル
  ln -sf "../shared_config/.env" "$WORKTREE_PATH/.env"

  # サブディレクトリのファイル（例: path/to/file）
  mkdir -p "$WORKTREE_PATH/path/to/"
  ln -sf "../../../shared_config/path/to/file" \
    "$WORKTREE_PATH/path/to/file"
done
```

### 他の Git スキルとの組み合わせ

worktree-setup は単体でも便利ですが、他の Git ワークフロースキルと組み合わせるとさらに効果的です。

```
/worktree-setup    → 並列開発環境を構築
（worktree-a で作業）
/branch feat/add-user-auth  → ブランチ作成
（実装作業）
/commit            → コミット
/commit-push-pr    → プッシュ + PR 作成
（worktree-b に移動して別の作業）
```

## まとめ

Claude Code の組み込み `/worktree` により複数開発環境を作ることができて便利ですが、`.worktreeinclude` によるファイル共有は作成時のコピーのみで、後から `.env` が変わった際は手動で更新が必要です。

`/worktree-setup` スキルでは、`shared_config/` に設定ファイルの実体を一元管理し、各 worktree からシンボリックリンクで参照する設計にしました。これにより、環境変数の追加・変更を一箇所の編集で全環境に反映することができます。

スキルのコードは以下で公開していますので、ご興味のある方は見ていただけると嬉しいです。

**英語版**
https://github.com/akifumi/skills/blob/main/skills/worktree-setup/SKILL.md

**日本語版**
https://github.com/akifumi/skills/blob/main/skills/worktree-setup-ja/SKILL.md
