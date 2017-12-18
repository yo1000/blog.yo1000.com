Doc Status: `WIP`

# Keycloak Demo

Keycloak による SSO 検証のデモサイト構築メモ。

## Summary

- [Requirements](#requirements)
- [Keycloak (SSO Server)](#keycloak-sso-server)
  - Add AdminUser
  - Run Keycloak
  - Add Realm
  - Add Roles
  - Add Users
- [Resource Server (SSO Client A)](#resource-server-sso-client-a)
- [Resource Client (SSO Client B)](#resource-client-sso-client-b)

## Requirements

TODO: 今回の構成図やインフラ要件を書く

## Keycloak (SSO Server)

Keycloak のダウンロード、管理ユーザーの作成、起動。

```console
$ # Download & Unarchive
$ curl https://downloads.jboss.org/keycloak/3.4.1.Final/keycloak-3.4.1.Final.tar.gz -o keycloak-3.4.1.Final.tar.gz
$ tar -zxvf keycloak-3.4.1.Final.tar.gz
$ cd keycloak-3.4.1.Final

$ # Add admin user for WildFly Management Console
$ bin/add-user.sh -u wildfly -p wildfly1234
Added user 'wildfly' to file '/${baseDir}/keycloak-3.4.1.Final/standalone/configuration/mgmt-users.properties'
Added user 'wildfly' to file '/${baseDir}/keycloak-3.4.1.Final/domain/configuration/mgmt-users.properties'

$ # Add admin user for Keycloak Admin Console
$ bin/add-user-keycloak.sh -r master -u keycloak -p keycloak1234
Added 'admin' to '/${baseDir}/keycloak-3.4.1.Final/standalone/configuration/keycloak-add-user.json', restart server to load user

$ # Run Keycloak
$ bin/standalone.sh -b 0.0.0.0 &
..
18:38:14,161 INFO  [org.jboss.as] (Controller Boot Thread) WFLYSRV0025: Keycloak 3.4.1.Final (WildFly Core 3.0.8.Final) started in 61427ms - Started 545 of 881 services (604 services are lazy, passive or on-demand)

$ # Register credentials
$ bin/kcadm.sh config credentials --server http://127.0.0.1:8080/auth --realm master --user keycloak --password keycloak1234
Logging into http://127.0.0.1:8080/auth as user admin of realm master

$ # Create realm
$ bin/kcadm.sh create realms -s realm=kc-resource-demo -s enabled=true
Created new realm with id 'kc-resource-demo'

$ # Create realm roles
$ bin/kcadm.sh create roles -r kc-resource-demo -s name=admin
Created new role with id 'admin'
$ bin/kcadm.sh create roles -r kc-resource-demo -s name=user
Created new role with id 'user'


```

各コマンドの説明は以下。

### Download & unarchive

執筆時点での最新は、[3.4.1.Final](http://www.keycloak.org/archive/downloads-3.4.1.html)

Download URL:
https://downloads.jboss.org/keycloak/3.4.1.Final/keycloak-3.4.1.Final.tar.gz


### Add admin user

管理ユーザーは、以下いずれかの方法で追加する。
__リモートホストから、直接追加することはできない。__

- ホスト内の add-user スクリプトによる登録 (今回はこちらを採用)
- 同一ホストからのアクセスによる Admin Console での登録

### Run keycloak

`-b` オプションで bind アドレスを指定する。このアドレス以外からの接続は拒否する。
未指定時は同一ホスト内からの接続のみ許可される。
`0.0.0.0` ですべて許可。

Admin Console URL:
http://127.0.0.1:8080/auth/admin/

#### Add Realm

http://192.168.128.5:8080/auth/admin/master/console/#/create/realm

- Name: `kc-resource-demo`
- Enabled: `ON`

## Resource Server (SSO Client A)

SSO クライアント A を追加する。<br>
[http://192.168.128.5:8080/auth/admin/master/console/#/create/client/kc-resource-demo](http://192.168.128.5:8080/auth/admin/master/console/#/create/client/kc-resource-demo)

以下を設定して、[Save] する。

- Client ID: `kc-resource-server`
- Client Protocol: `openid-connect`

Client が追加されると、[Settings] に遷移するので、引き続き以下を設定して、[Save] する。

- Client ID: `kc-resource-server` (そのまま)
- Client Protocol: `openid-connect` (そのまま)
- Access Type: `bearer only`

## Resource Client (SSO Client B)

SSO クライアント B を追加する。<br>
[http://192.168.128.5:8080/auth/admin/master/console/#/create/client/kc-resource-demo](http://192.168.128.5:8080/auth/admin/master/console/#/create/client/kc-resource-demo)

以下を設定して、[Save] する。

- Client ID: `kc-resource-client`
- Client Protocol: `openid-connect`

Client が追加されると、[Settings] に遷移するので、引き続き以下を設定して、[Save] する。

- Access Type: `confidential`
- Standard Flow Enabled: `ON`
- Valid Redirect URIs: `http://localhost:8080/*`


