# AWS 上でのデプロイ手順

## 1. ログイン
1. https://d-c7677a6954.awsapps.com/start を開く
2. ユーザー名とパスワードでログイン
3. 自分の名前のアカウント → `LabDeveloper` を押下
4. 右上のリージョンを Asia Pacific (Osaka) にする

## 2. 自分の DNS ゾーンを作成
1. Route 53 → Hosted zones → Create hosted zone を押下
2. Domain name に `<任意>.gakushu.now`、Type は Public hosted zone → Create hosted zone を押下
3. NS レコードの値4つを gakushu.now の管理者に送る
4. 管理者が追加するまで `<任意>.gakushu.now` 配下の URL はブラウザで開けない

## 3. Web Function を作成
1. Lambda → Web Functions → Create Web Function を押下
2. Function name に任意の名前、Code source は Hello World template
3. Endpoint name に任意の名前、Type は Home-Region
4. Authentication type は Application-managed authentication、Auto deployment は Latest revision
5. Additional configurations を開き、既存のロールとして `LabLambdaExec` を選ぶ
6. Create Web Function を押下
7. Endpoints → 3 のエンドポイント → ドメイン名を控える

## 4. CloudFront を作成
1. CloudFront → Distributions → Create distribution を押下
2. Flat-rate plan → Free を選ぶ
3. Distribution name に任意の名前 → Single website or app → Next
4. Domain setup は空のまま Next
5. Origin type は Amazon S3 や Elastic Load Balancing に当てはまらないものを選ぶ。次の Settings に「View default settings for Custom origin」と出れば正しい
6. Origin に 3-7 のドメイン名を入力
7. Cache settings → Customize cache settings → Cache policy: `CachingDisabled`、Origin request policy: `AllViewerExceptHostHeader` → Next
8. Enable security は Use monitor mode を付けずに Next
9. Create distribution を押下

## 5. URL を付ける
1. 4 の distribution → Settings → Alternate domain names → Add a domain を押下
2. `<アプリ名>.<任意>.gakushu.now` → Next
3. Create a wildcard certificate は付けずに Create certificate → Next → Add domains を押下
4. Settings → Route domains to CloudFront → 自動で設定する。ダイアログは何も表示せずに閉じるが、それで設定できている
