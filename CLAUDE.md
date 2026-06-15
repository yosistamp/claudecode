### コーディング規約
@import .claude/rules/coding-standards.md

### テスト規約
@import .claude/rules/testing.md

### デプロイ手順
@import .claude/rules/deployment_lambda.md

## 絶対遵守ルール

@import .claude/rules/critical.md


## プロジェクト概要
- [プロジェクト名]: [一言説明]
- 技術スタック: AWS Lambda / Python 3.13 / APIGateway / DynamoDB / Powertools / ADOT

## コーディング規約
- 言語: Python（バックエンド）
- すべての関数にdocstringを記載すること
- テストはpytestで記述
- 変数名: snake_case（Python）、camelCase（TypeScript）
- インデント: スペース4つ（Python）、スペース2つ（TypeScript）

## ファイル構成
- src/api/ - APIエンドポイント
- src/models/ - データモデル
- src/services/ - ビジネスロジック
- tests/ - テストコード

## 禁止事項
- print文でのデバッグ（loggerを使うこと）
- 環境変数のハードコード
- テストなしのコミット

## コミットルール
- メッセージは日本語
- prefix: feat / fix / refactor / test / docs