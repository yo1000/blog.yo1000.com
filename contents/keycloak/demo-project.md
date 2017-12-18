# Keycloak Demo Project

 セットアップメモ。

## Summary

- Requirements
- 
- Keycloak (SSO Server)
  - Download
  - Register AdminUser
  - Run
  - Setup Realm
    - Add Realm
- Resource Server (SSO Client A)
  - Add Client
- Resource Client (SSO Client B)
  - Add Client


# Keycloak (SSO Server)

Keycloak のダウンロードと起動。

```console
$ curl https://downloads.jboss.org/keycloak/3.4.1.Final/keycloak-3.4.1.Final.tar.gz -o keycloak-3.4.1.Final.tar.gz
$ tar -zxvf keycloak-3.4.1.Final.tar.gz
$ cd keycloak-3.4.1.Final
$ bin.add-user ************
$ bin/standalone.sh -b 0.0.0.0
```

## Download

執筆時点での最新は、[3.4.1.Final](http://www.keycloak.org/archive/downloads-3.4.1.html)

- https://downloads.jboss.org/keycloak/3.4.1.Final/keycloak-3.4.1.Final.tar.gz

## Register AdminUser

管理者の作成は、以下いずれかの方法にでおこなう。

- 同一ホストからのアクセスによる Admin Console での登録
- ホスト内の add-user スクリプトによる登録


## Run

`-b` オプションで bind アドレスの指定。このアドレス以外からの接続を拒否する。
未指定時は同一ホスト内からの接続のみ許可される。
`0.0.0.0` はすべて許可。

Admin Console URL:
http://192.168.128.5:8080/auth/admin/


## Setup Realm

### Add Realm

http://192.168.128.5:8080/auth/admin/master/console/#/create/realm

- Name: `kc-resource-demo`
- Enabled: `ON`

# Resource Server (SSO Client A)


## Add Client

Add Client

http://192.168.128.5:8080/auth/admin/master/console/#/create/client/kc-resource-demo

以下を設定して [Save]

- Client ID: `kc-resource-server`
- Client Protocol: `openid-connect`

Kc-resource-server

http://192.168.128.5:8080/auth/admin/master/console/#/realms/kc-resource-demo/clients/${client-uuid}

以下を設定して [Save]

- Client ID: `kc-resource-server` (そのまま)
- Client Protocol: `openid-connect` (そのまま)
- Access Type: `bearer only`


# Resource Client (SSO Client B)



- Client ID: `kc-resource-client`
- Client Protocol: `openid-connect`
- Access Type: `confidential`
- Standard Flow Enabled: `ON`
- Valid Redirect URIs: http://

