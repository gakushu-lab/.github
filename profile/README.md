# gakushu-lab

![gakushu-lab の構成](architecture.svg)

図の元ファイルは [architecture.drawio](architecture.drawio)。

## アプリのデプロイ手順

1. gakushu-lab にリポジトリを作る
2. `aws sso login` で自分の AWS アカウントに入る
3. アプリを HTTP サーバーとして書く。待ち受け先は環境変数 `AWS_LAMBDA_HTTP_ENDPOINT` (host:port)
4. Lambda Web Function に配置する
   ```sh
   aws lambda-web deploy --name <名前> --code <ディレクトリ> --create \
     --execution-role-arn arn:aws:iam::<アカウント ID>:role/LabLambdaExec
   ```
5. 独自ドメインを付けるなら、CloudFront (Free プラン) を前に置き、自分のホストゾーンにレコードを作る
6. 自動デプロイは GitHub Actions で `LabGitHubDeploy` を引き受ける
   ```yaml
   permissions:
     id-token: write
   # aws-actions/configure-aws-credentials の with:
   role-to-assume: arn:aws:iam::<アカウント ID>:role/LabGitHubDeploy
   aws-region: ap-northeast-3
   ```

## 制約

- リージョンは大阪 (ap-northeast-3)。CloudFront の証明書と AWS WAF だけ us-east-1
- IAM ロールは作れない。Lambda は `LabLambdaExec`、GitHub Actions は `LabGitHubDeploy` を使う
- 実行ロールが要るサービス (EventBridge Scheduler など) は使えない
- アクセスキーは使わない
- `LabGitHubDeploy` は gakushu-lab のリポジトリで本人が起こした実行だけを受け付ける
- 1人 月 3 USD まで。超えると翌月1日まで AWS の操作が止まる。動いているアプリは止まらない
- CloudFront の Free プランは1アカウント3つまで。AWS WAF の web ACL が必須

例: [hapax](https://github.com/gakushu-lab/hapax)
