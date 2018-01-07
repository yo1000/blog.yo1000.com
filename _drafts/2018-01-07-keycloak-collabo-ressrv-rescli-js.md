### Keycloak ヘログイン
以降、`kcadm.sh` を使用する上で、ログイン状態が必要になるため、ログインします。
`kcadm.sh` 実行時に、以下のようなメッセージが出力された場合は、
ログインセッションが期限切れとなっているため、改めてログインします。

> Session has expired. Login again with ‘kcadm.sh config credentials’

```console
$ # Login to Keycloak
$ bin/kcadm.sh config credentials \
  --server http://127.0.0.1:8080/auth \
  --realm master \
  --user keycloak \
  --password keycloak1234
Logging into http://127.0.0.1:8080/auth as user admin of realm master
```

### SSO クライアントの登録
SSO 基盤を使用するアプリケーション (SSO サーバーに対する、クライアント) を登録します。
リソースサーバーとして `kc-resource-server` は既に作成されているものとします。
ここではリソースクライアントとして、新たに `kc-resource-client-js` を作成します。

```console
$ # Add realm client for Resource client (for Javascript)
$ RES_CLI_ID=`\
  bin/kcadm.sh create clients \
  -r kc-resource \
  -s clientId=kc-resource-client-js \
  -s publicClient=true \
  -s 'redirectUris=["http://127.0.0.1:28081/*"]' \
  -s 'webOrigins=["http://127.0.0.1:28081"]' \
  -i\
  `; echo $RES_CLI_ID
f970945c-67dc-4c09-8126-423158ff1248
```
