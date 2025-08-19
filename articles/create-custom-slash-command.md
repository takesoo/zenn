---
title: "Claude Codeのカスタムスラッシュコマンドを作ってみた"
emoji: "⚡"
type: "tech"
topics: ["claude", "ai", "productivity", "automation"]
published: true
---

# はじめに

Claude Codeは、Anthropic社が提供するAIアシスタントツールです。コード編集、ファイル操作、コマンド実行など、開発作業を強力にサポートしてくれます。

その中でも特に便利なのが「カスタムスラッシュコマンド」機能です。よく使う作業をコマンド化することで、作業効率を大幅に向上させることができます。

この記事では、Claude Codeのカスタムスラッシュコマンドの作り方と、実際に作成した3つのコマンドを紹介します。

# カスタムスラッシュコマンドとは

カスタムスラッシュコマンドは、マークダウンファイルにプロンプトを記述するだけで作成できる、Claude Code用の独自コマンドです。

## 2種類のコマンド

カスタムコマンドには2種類あります：

### 1. パーソナルコマンド
- **保存場所**: `~/.claude/commands/`
- **スコープ**: すべてのプロジェクトで利用可能
- **用途**: 汎用的なタスクや個人的なワークフロー

### 2. プロジェクトコマンド  
- **保存場所**: `.claude/commands/`（プロジェクトルート）
- **スコープ**: 特定のプロジェクトでのみ利用可能
- **用途**: プロジェクト固有のタスクや規約に沿った作業

## 基本的な作成方法

カスタムコマンドの作成は驚くほど簡単です：

1. 適切なディレクトリに移動
2. `[コマンド名].md`というファイルを作成
3. frontmatter（オプション）とプロンプトを記述

```markdown
---
allowed-tools: []
description: "コマンドの説明"
argument-hint: "[引数の説明]"
---

ここにプロンプトを書く
$ARGUMENTS
```

# 実践編1: シンプルな質問応答コマンド `/ask`

まずは最もシンプルなコマンドから始めましょう。`/ask`コマンドは、コードの実行や編集を行わず、純粋に質問に答えるためのコマンドです。

## 実装コード

`~/.claude/commands/ask.md`:

```markdown
---
allowed-tools: []
description: "質問に回答します"
argument-hint: "[質問内容]"
---

以下の質問に回答してください。コードの実行や編集は行わないでください。

質問: $ARGUMENTS
```

## ポイント解説

- **`allowed-tools: []`**: 空配列を指定することで、すべてのツールを無効化
- **`$ARGUMENTS`**: ユーザーが入力した引数がここに展開される
- シンプルなプロンプトで、Claude Codeの挙動を制限

## 使用例

```bash
/ask Pythonのリスト内包表記の利点を教えて

/ask Gitのrebaseとmergeはどう使い分けるべき？
```

このコマンドは、コードを触らずに純粋な情報を得たい時に便利です。

# 実践編2: コマンド作成支援コマンド `/create-command`

次は、メタプログラミング的なアプローチです。新しいカスタムコマンドを作成するコマンドを作りました。

## 実装コード（一部抜粋）

`~/.claude/commands/create-command.md`:

```markdown
---
allowed-tools: Write, Read, Bash(mkdir:*)
description: "カスタムslashコマンドの定義ファイルを作成します"
argument-hint: "[--project | --personal] [コマンド名] [コマンドの目的]"
---

あなたはClaude Codeのカスタムslashコマンド作成の専門家です。
ユーザーの要求に基づいて、適切なカスタムslashコマンドの定義ファイルを作成してください。

## 入力情報
$ARGUMENTS

## 作成手順

1. 入力を解析：
   - スコープフラグの確認（--project または --personal）
   - コマンド名の抽出
   - コマンドの目的の理解
   
   注意：
   - フラグが指定されていない場合は、デフォルトでパーソナルコマンドとして作成
   - --projectの場合: `.claude/commands/`に作成
   - --personalの場合: `~/.claude/commands/`に作成

2. コマンドの目的に基づいて以下を決定：
   - 必要なツール（allowed-tools）
   - 適切な説明文（description）
   - 引数のヒント（argument-hint）
   - 使用するモデル（処理速度重視ならhaiku、精度重視ならsonnet/opus）

[以下省略...]
```

## ポイント解説

- **`Bash(mkdir:*)`**: Bashツールの使用を`mkdir`コマンドに限定
- **構造化されたプロンプト**: 番号付きリストで明確な手順を指示
- **条件分岐の処理**: フラグによってファイルの作成場所を変更
- **専門家としての役割定義**: 冒頭でClaude Codeの役割を明確化

## 使用例

```bash
# パーソナルコマンドとして作成
/create-command test-runner すべてのテストを実行して結果を整形

# プロジェクトコマンドとして作成
/create-command --project format-code プロジェクトのコード整形を実行

# モデルを考慮したコマンド作成
/create-command quick-check 簡単な構文チェック（高速処理重視）
```

# 実践編3: 既存コマンド改善コマンド `/improve-command`

最後は、既存のカスタムコマンドを改善するコマンドです。

## 実装コード（一部抜粋）

`~/.claude/commands/improve-command.md`:

```markdown
---
allowed-tools: Read, Write, Edit, Glob
description: "既存のカスタムスラッシュコマンドに指定された改善を適用します"
argument-hint: "[/コマンド名] [改善内容の指示]"
---

あなたはClaude Codeのカスタムslashコマンド改善の専門家です。
ユーザーが指定した改善内容に従って、既存のカスタムコマンドを更新してください。

## 処理手順

1. 入力を解析：
   - 第1引数: /コマンド名（スラッシュ付き）
   - 第2引数以降: 適用する改善内容の指示
   
   コマンドファイルの探索：
   - プロジェクトコマンド: `.claude/commands/[コマンド名].md`
   - パーソナルコマンド: `~/.claude/commands/[コマンド名].md`

2. 既存コマンドの読み込み：
   - 現在のfrontmatter設定を確認
   - 現在のプロンプト内容を理解
   - バックアップファイル作成（[コマンド名].md.bak）

3. 改善内容の適用：
   **よくある改善指示の例と対応：**
   - "日本語で応答するように": プロンプトに日本語応答の指示を追加
   - "ツールを追加": allowed-toolsに指定ツールを追加
   - "高速化": modelをhaikuに変更
   - "より詳細な出力": プロンプトに詳細出力の指示を追加

[以下省略...]
```

## ポイント解説

- **`Glob`ツールの活用**: ファイルの探索に使用
- **バックアップの作成**: 変更前の状態を保存
- **改善パターンの定義**: よくある改善要求に対する対応を明記
- **両方のコマンドタイプに対応**: プロジェクト/パーソナル両方を探索

## 使用例

```bash
# 日本語対応を追加
/improve-command /ask 日本語で応答するように

# ツールを追加
/improve-command /test-runner WebSearchツールを追加して最新のテスト手法を参照できるように

# 高速化
/improve-command /format-code 処理を高速化してください

# エラーハンドリング追加
/improve-command /deploy エラーハンドリングを強化して
```

# Tips & Tricks

## frontmatterの活用

### allowed-tools
ツールへのアクセスを制限できます：

```yaml
# すべてのツールを無効化
allowed-tools: []

# 特定のツールのみ許可
allowed-tools: [Read, Write]

# ワイルドカードで部分的に許可
allowed-tools: [Bash(git:*), Bash(npm:*)]
```

### model指定
処理内容に応じて最適なモデルを選択：

```yaml
# 高速処理（簡単なタスク）
model: claude-3-5-haiku-20241022

# バランス型（標準的なタスク）
model: claude-3-5-sonnet-20241022
```

## 動的な要素の活用

### $ARGUMENTS
ユーザー入力を受け取る：

```markdown
処理対象: $ARGUMENTS
```

### @ファイル参照
ファイルの内容を直接参照：

```markdown
以下のファイルを参考にしてください：
@package.json
@README.md
```

### !コマンド実行
Bashコマンドの実行結果を取得：

```markdown
現在のGitブランチ：
!git branch --show-current
```

## ディレクトリ構造での整理

サブディレクトリを使ってコマンドを整理できます：

```
~/.claude/commands/
├── git/
│   ├── commit.md
│   └── push.md
├── test/
│   ├── unit.md
│   └── e2e.md
└── ask.md
```

使用時：`/git/commit`、`/test/unit`など

# まとめ

Claude Codeのカスタムスラッシュコマンドは、マークダウンファイルを作成するだけで簡単に作れる強力な機能です。

- **簡単**: マークダウンファイルを置くだけ
- **柔軟**: frontmatterで細かい制御が可能
- **強力**: ツールの制限、動的要素の活用など多彩な機能

今回紹介した3つのコマンドは、それぞれ異なるアプローチを示しています：

1. **`/ask`**: シンプルさの極致
2. **`/create-command`**: メタプログラミング的アプローチ
3. **`/improve-command`**: 既存資産の改善

これらを参考に、ぜひ自分だけのカスタムコマンドを作ってみてください。日々の開発作業が格段に効率化されるはずです。

## 参考リンク

- [Claude Code公式ドキュメント - Slash Commands](https://docs.anthropic.com/en/docs/claude-code/slash-commands)
- [Claude Code GitHub](https://github.com/anthropics/claude-code)

---

*この記事が参考になった場合は、ぜひ自作のカスタムコマンドをコメントで共有してください！*
