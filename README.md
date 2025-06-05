dotenvx環境変数管理プロジェクト

このプロジェクトは、dotenvxを使用して環境変数を暗号化し、
AWS Parameter Storeと連携して安全に管理するためのTaskfileベースの環境です。

------------------------------
概要
------------------------------
- dotenvx: 環境変数の暗号化・復号化
- AWS Parameter Store: 復号化キーの安全な保管・共有
- 個人用設定の分離: GITHUB_TOKEN等の個人トークンを安全に管理
- Taskfile: 環境構築・運用の自動化

------------------------------
前提条件
------------------------------
- Node.js (npm)
- AWS CLI
- Task (go-task/task)
- jq

------------------------------
セットアップ
------------------------------
1. 初期化
task init
  - AWS Parameter Storeから復号化キー（DOTENV_PRIVATE_KEY_PUBLIC）を取得し.env.keysに保存
  - 個人用設定ファイル.env.personalを作成

------------------------------
使用方法
------------------------------
環境変数の追加・更新
共通設定（.env.public）に追加:
  task set-dotenv -- API_KEY=your_api_key_here
  task set-dotenv -- DB_HOST=localhost

個人用設定（.env.personal）に追加:
  # .env.personalファイルを直接編集
  vim .env.personal

------------------------------
ファイル構成
------------------------------
.
├── Taskfile.yml          # タスク定義
├── .env.keys            # 復号化キー（gitignore推奨）
├── .env.personal        # 個人用トークン（gitignore推奨）
├── .env.public          # 共通設定（暗号化対象）
├── .env.vault           # 暗号化されたファイル（git管理OK）
└── README.md

------------------------------
重要な注意事項
------------------------------
Git管理について
コミットしてはいけないファイル:
  .env.keys
  .env.personal
コミットしても良いファイル:
  .env.public

個人用トークンの管理
- GITHUB_TOKEN, DATADOG_KEYなどの個人用トークンは.env.personalで管理
- チームメンバー間で個人トークンが混在することを防ぐ
- .env.personalは各自のローカル環境でのみ管理

------------------------------
AWS設定
------------------------------
Parameter Store設定
以下のパラメータがAWS Parameter Storeにあります。
  パラメータ名: DOTENV_PRIVATE_KEY_PUBLIC
  タイプ: SecureString
  値: dotenvxで生成された復号化キー

AWSプロファイル
Taskfile.ymlのAWS_PROFILE変数を環境に合わせて変更してください：
  vars:
    AWS_PROFILE: your-aws-profile-name

------------------------------
チーム運用の流れ
------------------------------
新しいメンバーのオンボーディング
1. リポジトリをクローン
2. task init を実行
3. .env.personalに個人のトークンを設定
4. アプリケーション実行

環境変数の追加・更新
1. 共通設定は task set-dotenv -- KEY=VALUE で追加
2. 個人設定は .env.personal を直接編集
3. .env.public をコミット・プッシュ
4. チームメンバーは git pull で最新の暗号化設定を取得

------------------------------
トラブルシューティング
------------------------------
よくあるエラー

1. AWS認証エラー
aws configure list
# または
aws sso login --profile your-profile

2. dotenvxコマンドが見つからない
npm install @dotenvx/dotenvx --global

3. 引数の形式エラー
正しい形式
  task set-dotenv -- KEY=VALUE
間違った形式
  task set-dotenv KEY=VALUE  # -- が抜けている

------------------------------
セキュリティのベストプラクティス
------------------------------
1. 復号化キー（.env.keys）は絶対にコミットしない
2. 個人用トークン（.env.personal）は各自で管理
3. AWS Parameter Storeへのアクセス権限を適切に設定
4. 定期的な復号化キーのローテーション

------------------------------
参考リンク
------------------------------
dotenvx公式ドキュメント: https://dotenvx.com/docs/
Taskfile公式ドキュメント: https://taskfile.dev/
AWS Parameter Store: https://docs.aws.amazon.com/systems-manager/latest/userguide/systems-manager-parameter-store.html

