# gakushu-lab でのデプロイ手順

HTML などの静的ファイルだけを置く場合は、3、4、6-11 〜 6-14 を飛ばす。

## 1. ログイン
1. https://d-c7677a6954.awsapps.com/start を開く
2. ユーザー名とパスワードでログイン
3. 自分の名前のアカウント → `LabDeveloper` を押下
4. 右上のリージョンを Asia Pacific (Osaka) にする

## 2. 自分の DNS ゾーンを確認 ([1](https://github.com/gakushu-lab/.github/blob/main/profile/README.md#アプリのデプロイ手順))
1. Route 53 → Hosted zones を開く
2. `<ユーザー名>.gakushu.now` があることを確認する。管理者が作成済みで、レジストラへの登録と gakushu.now からの NS レコードも登録済み

## 3. DynamoDB のテーブルを作成 ([2](https://github.com/gakushu-lab/.github/blob/main/profile/README.md#アプリのデプロイ手順))
1. DynamoDB → Tables → Create table を押下
2. Table name と Partition key に任意の名前を入力
3. Table settings は Default settings のまま Create table を押下

## 4. Web Function を作成 ([3](https://github.com/gakushu-lab/.github/blob/main/profile/README.md#アプリのデプロイ手順))
1. Lambda → Web Functions → Create Web Function を押下
2. Function name に任意の名前、Code source は Hello World template
3. Endpoint name に任意の名前、Type は Home-Region
4. Authentication type は Application-managed authentication、Auto deployment は Latest revision
5. Additional configurations を開き、既存のロールとして `LabLambdaExec` を選ぶ
6. 同じ Additional configurations の環境変数に、3 のテーブル名を入れる (例: `TABLE_NAME`)
7. Create Web Function を押下
8. Endpoints → 4-3 のエンドポイント → ドメイン名を控える

## 5. S3 にフロントエンドを置く ([4](https://github.com/gakushu-lab/.github/blob/main/profile/README.md#アプリのデプロイ手順))
1. S3 → General purpose buckets → Create bucket を押下
2. Bucket name に任意の名前を入力し、ほかは既定のまま Create bucket を押下
3. 作ったバケット → Upload → Add file または Add folder で HTML、CSS、JS を選ぶ → Upload を押下

## 6. CloudFront を作成 ([5 と 7](https://github.com/gakushu-lab/.github/blob/main/profile/README.md#アプリのデプロイ手順))
1. CloudFront → Distributions → Create distribution を押下
2. Flat-rate plan → Free を選ぶ
3. Distribution name に任意の名前、Distribution type は Single website configuration
4. Route 53 managed domain に `<ユーザー名>.gakushu.now` → Check domain → Add a subdomain → `<アプリ名>` を入力 → Next。`<ユーザー名>.gakushu.now` 自体も配信先に含まれ、外せない
5. Origin type は Amazon S3 → S3 origin の Browse S3 で 5 のバケットを選ぶ
6. Settings は Use recommended origin settings のまま Next
7. Enable security は Use monitor mode を付けずに Next
8. TLS certificate は Create a wildcard certificate を付けずに Create certificate → Next
9. Create distribution を押下。ホストゾーンは自動で無料プランに含まれ、Manage plan の Route 53 DNS に表示される
10. Settings → General → Edit → Default root object に `index.html` → Save changes
11. Origins → Create origin → Origin domain に 4-8 のドメイン名、Protocol は HTTPS only → Create origin
12. Behaviors → Create behavior → Path pattern に `/api/*`、Origin or origin group に 6-11 の origin
13. Viewer protocol policy は Redirect HTTP to HTTPS、Allowed HTTP methods は GET, HEAD, OPTIONS, PUT, POST, PATCH, DELETE
14. Cache policy は `CachingDisabled`、Origin request policy は `AllViewerExceptHostHeader` → Create behavior

## 7. URL を付ける ([6](https://github.com/gakushu-lab/.github/blob/main/profile/README.md#アプリのデプロイ手順))
1. 6 の distribution → Settings → Route domains to CloudFront → 自動で設定する。ダイアログは何も表示せずに閉じるが、それで設定できている
