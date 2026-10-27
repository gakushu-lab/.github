# gakushu-lab での DevOps

例: [hapax](https://github.com/gakushu-lab/hapax) の `.github/workflows/` と `infra/`

## 1. GitHub Actions から AWS にログイン
1. ワークフローに `permissions: id-token: write` を付ける
2. `aws-actions/configure-aws-credentials` で `arn:aws:iam::<アカウント ID>:role/LabGitHubDeploy` を引き受ける
3. 受け付けるのは gakushu-lab のリポジトリで本人が起こした実行だけ。本人の GitHub の ID は gakushu-lab の管理者が登録する
4. アクセスキーは使わない

## 2. CI ([1](https://github.com/gakushu-lab/.github/blob/main/profile/README.md#devops))
1. `.github/workflows/ci.yaml` を作り、`pull_request` と `main` への `push` で動かす
2. 型検査、lint、テスト、ビルドを順に実行する
3. Terraform を使うなら `terraform plan` も実行し、PR でインフラの差分を見る
4. 例: [hapax の ci.yaml](https://github.com/gakushu-lab/hapax/blob/main/.github/workflows/ci.yaml)

## 3. CD ([2](https://github.com/gakushu-lab/.github/blob/main/profile/README.md#devops))
1. `.github/workflows/cd.yaml` を作り、`main` への `push` で動かす
2. インフラを適用してから、API サーバーとフロントエンドを配置する
3. API サーバーは `aws lambda-web deploy`、フロントエンドは `aws s3 sync` と CloudFront のキャッシュ削除
4. 例: [hapax の cd.yaml](https://github.com/gakushu-lab/hapax/blob/main/.github/workflows/cd.yaml)

## 4. Infrastructure as Code ([3](https://github.com/gakushu-lab/.github/blob/main/profile/README.md#devops))
1. Terraform の状態を置く S3 バケットを先に手で作る
2. `backend "s3"` にそのバケットを書き、`use_lockfile = true` で同時実行を防ぐ
3. IAM ロールは作れないので、`data "aws_iam_role"` で用意済みのロールを参照する
4. 例: [hapax の infra/](https://github.com/gakushu-lab/hapax/tree/main/infra)

## 5. 監視とログ ([4](https://github.com/gakushu-lab/.github/blob/main/profile/README.md#devops))
1. Lambda のログは CloudWatch Logs に自動で出る。保持期間はロググループで決める
2. 見たいメトリクスを CloudWatch のダッシュボードにまとめ、Terraform で管理する
3. 例: [hapax の dashboard.tf](https://github.com/gakushu-lab/hapax/blob/main/infra/dashboard.tf)
