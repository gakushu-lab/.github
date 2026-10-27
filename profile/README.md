# gakushu-lab

![gakushu-lab の構成](guide/architecture.svg)

図の元ファイルは [architecture.drawio](https://github.com/gakushu-lab/.github/blob/main/profile/guide/architecture.drawio)。

詳細な gakushu-lab の構成は[こちら](https://github.com/gakushu-lab/.github/tree/main/profile/architecture)。

## アプリのデプロイ手順

1. ドメイン (example.com のような URL のホスト名部分) を取得してDNSに登録する (Route 53)
2. データベースを作る (DynamoDB)
3. API サーバーをデプロイしてデータベースに接続させる (Lambda Web Function)
4. フロントエンドの静的ファイルをホスティングする (S3)
5. リバースプロキシで `/api/*` だけを API サーバーに転送する (CloudFront)
6. DNS の A レコードにリバースプロキシの IP アドレスを登録する (Route 53)
7. 認証局に TLS 証明書を発行してもらいリバースプロキシに設定する (ACM, CloudFront)

### AWS 上でのやり方

[こちら](https://github.com/gakushu-lab/.github/blob/main/profile/guide/deploy.md)を参照

## 制約

使えるサービスは限られている。上の構成図の「使えるサービス」を参照

例: [hapax](https://github.com/gakushu-lab/hapax)
