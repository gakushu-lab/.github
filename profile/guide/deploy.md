# gakushu-lab でのデプロイ手順

HTML などの静的ファイルだけを置く場合は、3、4、6-10 〜 6-13 を飛ばす。

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

## 6. CloudFront を作成 ([5](https://github.com/gakushu-lab/.github/blob/main/profile/README.md#アプリのデプロイ手順))
1. CloudFront → Distributions → Create distribution を押下
2. Flat-rate plan → Free を選ぶ
3. Distribution name に任意の名前 → Single website or app → Next
4. Domain setup は空のまま Next
5. Origin type は Amazon S3 → S3 origin の Browse S3 で 5 のバケットを選ぶ
6. Settings は Use recommended origin settings のまま Next
7. Enable security は Use monitor mode を付けずに Next
8. Create distribution を押下
9. Settings → General → Edit → Default root object に `index.html` → Save changes
10. Origins → Create origin → Origin domain に 4-8 のドメイン名、Protocol は HTTPS only → Create origin
11. Behaviors → Create behavior → Path pattern に `/api/*`、Origin or origin group に 6-10 の origin
12. Viewer protocol policy は Redirect HTTP to HTTPS、Allowed HTTP methods は GET, HEAD, OPTIONS, PUT, POST, PATCH, DELETE
13. Cache policy は `CachingDisabled`、Origin request policy は `AllViewerExceptHostHeader` → Create behavior

## 7. URL を付ける ([6 と 7](https://github.com/gakushu-lab/.github/blob/main/profile/README.md#アプリのデプロイ手順))
1. 6 の distribution → Settings → Alternate domain names → Add a domain を押下
2. `<アプリ名>.<ユーザー名>.gakushu.now` → Next
3. Create a wildcard certificate は付けずに Create certificate → Next → Add domains を押下
4. Settings → Route domains to CloudFront → 自動で設定する。ダイアログは何も表示せずに閉じるが、それで設定できている
5. 6 の distribution → Manage plan → Route 53 DNS の Attach → Select a hosted zone で `<ユーザー名>.gakushu.now` を選ぶ → Attach を押下。ホストゾーンの月額料金が無料プランに含まれる
