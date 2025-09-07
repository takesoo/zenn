# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## リポジトリ概要

これはZennプラットフォーム用のコンテンツリポジトリです。技術記事や本を執筆・管理するために使用されています。

## よく使うコマンド

### コンテンツ管理
- `npx zenn preview` - ローカルプレビューサーバーを起動（http://localhost:8000）
- `npx zenn new:article` - 新しい記事を作成
- `npx zenn new:book` - 新しい本を作成
- `npx zenn list:articles` - 記事の一覧を表示
- `npx zenn list:books` - 本の一覧を表示

### パッケージ管理
- `npm install` - 依存関係をインストール（主にzenn-cli）
- `npm install zenn-cli@latest` - Zenn CLIを最新版に更新

## コンテンツ構造

### 記事（articles）
- **保存場所**: `articles/`ディレクトリ
- **形式**: YAMLフロントマター付きMarkdownファイル
- **命名規則**: 内容を表す分かりやすいスラッグ（例: `create-custom-slash-command.md`）
- **フロントマター必須項目**:
  - `title`: 記事タイトル
  - `emoji`: 記事を表す絵文字
  - `type`: 記事タイプ（技術記事は"tech"）
  - `topics`: トピックタグの配列（最大5個）
  - `published`: 公開状態（true/false）

### 本（books）
- **保存場所**: `books/`ディレクトリ
- **構造**: 各本は独自のディレクトリを持ち、その中に章のファイルを配置

## 執筆時の注意点

Zennコンテンツを作成・編集する際は：
1. 記事は日本語で執筆する
2. 技術的な内容にはコードブロックを適切に使用
3. フロントマターに適切なメタデータを設定
4. トピックは最大5個まで、関連性の高いものを選択
5. ファイル名は内容を反映した分かりやすい名前にする

## 開発フロー

1. `npx zenn new:article`または`npx zenn new:book`で新規コンテンツを作成
2. `npx zenn preview`でプレビューサーバーを起動
3. Markdownファイルで内容を編集
4. ブラウザでhttp://localhost:8000にアクセスして確認
5. 公開準備ができたらフロントマターの`published: true`に設定
6. Gitリポジトリに変更をコミット