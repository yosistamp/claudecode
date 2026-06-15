## プロジェクト概要
- [プロジェクト名]: [一言説明]
- 技術スタック: AWS Lambda / Python 3.13 / APIGateway / DynamoDB / Powertools / ADOT

## コーディング規約
- すべての関数にdocstringを記載する
- テストはpytestで記述
- 変数名: snake_case（Python）
- インデント: スペース4つ（Python）
- flake8によるLintの実施 ただし１行８０文字のルールは無視して良い

## ファイル構成
- src/handler.py エントリーポイント
- src/api/ - APIエンドポイント
- src/models/ - データモデル
- src/services/ - ビジネスロジック
- tests/ - テストコード

## 実装ルール
- Pownertoolsによるロギング、モニタリング、トレーシングの実装
  - ログにも必ずトレースIDを含める
- ログは構造化ログ（JSON）
- Lambdaの単位は１機能のCRUDを基本とする
- インターフェースはOpenAPIに準拠
- エラーレスポンスはRFC 9457に準拠
- Lambda関数はDockerイメージを作成して、ECRに保管したものをデプロイ
- デプロイにはAWS SAMを利用
- ビルド、デプロイはGitHubActions
  - ビルドとデプロイでアクションは分離する
  - ビルド時に必要なIAMロールはGitHubのシークレット変数AWS_ECR_PUSH_ROLEから取得する
  - デプロイ時に必要なIAMロールはGitHubのシークレット変数AWS_LAMBDA_DEPLOY_ROLEから取得する

## 禁止事項
- .env、各種APIキー、認証トークン、プライベートキー、パスワードなどの認証情報の読み取りおよび出力
- 環境変数のハードコード
- rm -rf / など破壊的コマンドの実行禁止
- print文でのデバッグ（loggerを使うこと）
- テストなしのコミット

## コミットルール
- メッセージは日本語
- prefix: feat / fix / refactor / test / docs