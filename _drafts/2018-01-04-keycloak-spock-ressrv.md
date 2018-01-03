---
title: Keycloak 認証を使うリソースサーバー (Spring Boot クライアント) の Groovy Spock テスト
category: keycloak
tags:
- keycloak
- spring boot
- spring security
- testing
- groovy
- spock
---

## 概要
Keycloak による認証を使う、リソースサーバー (Spring Boot クライアント) での Groovy Spock テスト実装メモ。

この手順で使用したコードは、以下に公開しているので、こちらも参考にしてください。<br>
[https://github.com/yo1000/kc-resource/ac9914ae02#try-testing-with-only-kc-resource-server](https://github.com/yo1000/kc-resource/tree/ac9914ae027583a167fa677892a3c6982379a6eb#try-testing-with-only-kc-resource-server)

また、テストについては、既に[過去のポスト](http://blog.yo1000.com/keycloak/keycloak-test-ressrv.html)で触れているため、
ここでは、Groovy Spock を適用するにあたって、変更が必要となる部分について書いていきます。

### 目次

- [要件](#要件)
  - [環境](#環境)
- [pom.xml の変更](#pom.xml-の変更)
  - [依存関係の追加](#依存関係の追加)
  - [ビルド構成の変更](#ビルド構成の変更)
- [テストの実装](#テストの実装)
  - [テストコード](#テストコード)
- [デモ](#デモ)

## 要件
### 環境
今回の作業環境は以下のとおりです。

- Java 1.8.0_131
- Spring Boot 1.5.9.RELEASE
- Keycloak 3.4.1.Final
- Groovy 2.4.11
- Spock 1.1-groovy-2.4

```console
$ sw_vers
ProductName:	Mac OS X
ProductVersion:	10.12.5
BuildVersion:	16F2073

$ java -version
java version "1.8.0_131"
Java(TM) SE Runtime Environment (build 1.8.0_131-b11)
Java HotSpot(TM) 64-Bit Server VM (build 25.131-b11, mixed mode)
```

## pom.xml の変更
### 依存関係の追加
以下の依存を追加します。

### ビルド構成の変更
以下のようにビルドプラグインを追加します。

## テストの実装
### テストコード

[過去のポスト](http://blog.yo1000.com/keycloak/keycloak-test-ressrv.html#%E3%83%86%E3%82%B9%E3%83%88%E3%82%B3%E3%83%BC%E3%83%89)
で書いたテストと、内容は同じものです。Groovy Spock 用に書き改めています。

```groovy
```

## デモ
参考までに実際に動かした結果の一部を、以下に残しておきます。

```console
$ ./mvnw clean test

..

2018-01-04 02:34:09.638  INFO 74374 --- [           main] .y.k.r.s.KcResourceServerControllerSpecs : Started KcResourceServerControllerSpecs in 3.334 seconds (JVM running for 13.218)
2018-01-04 02:34:09.685  INFO 74374 --- [           main] o.a.c.c.C.[Tomcat].[localhost].[/]       : Initializing Spring FrameworkServlet ''
2018-01-04 02:34:09.685  INFO 74374 --- [           main] o.s.t.web.servlet.TestDispatcherServlet  : FrameworkServlet '': initialization started
2018-01-04 02:34:09.698  INFO 74374 --- [           main] o.s.t.web.servlet.TestDispatcherServlet  : FrameworkServlet '': initialization completed in 12 ms

MockHttpServletRequest:
      HTTP Method = GET
      Request URI = /kc/resource/server/admin
       Parameters = {}
          Headers = {}

Handler:
             Type = com.yo1000.keycloak.resource.server.KcResourceServerController
           Method = public java.lang.String com.yo1000.keycloak.resource.server.KcResourceServerController.getAdminResource()

Async:
    Async started = false
     Async result = null

Resolved Exception:
             Type = null

ModelAndView:
        View name = null
             View = null
            Model = null

FlashMap:
       Attributes = null

MockHttpServletResponse:
           Status = 200
    Error message = null
          Headers = {X-Content-Type-Options=[nosniff], X-XSS-Protection=[1; mode=block], Cache-Control=[no-cache, no-store, max-age=0, must-revalidate], Pragma=[no-cache], Expires=[0], X-Frame-Options=[DENY], Content-Type=[text/plain;charset=UTF-8], Content-Length=[16]}
     Content type = text/plain;charset=UTF-8
             Body = ADMIN Resource!!
    Forwarded URL = null
   Redirected URL = null
          Cookies = []

MockHttpServletRequest:
      HTTP Method = GET
      Request URI = /kc/resource/server/user
       Parameters = {}
          Headers = {}

Handler:
             Type = com.yo1000.keycloak.resource.server.KcResourceServerController
           Method = public java.lang.String com.yo1000.keycloak.resource.server.KcResourceServerController.getUserResource()

Async:
    Async started = false
     Async result = null

Resolved Exception:
             Type = null

ModelAndView:
        View name = null
             View = null
            Model = null

FlashMap:
       Attributes = null

MockHttpServletResponse:
           Status = 200
    Error message = null
          Headers = {X-Content-Type-Options=[nosniff], X-XSS-Protection=[1; mode=block], Cache-Control=[no-cache, no-store, max-age=0, must-revalidate], Pragma=[no-cache], Expires=[0], X-Frame-Options=[DENY], Content-Type=[text/plain;charset=UTF-8], Content-Length=[14]}
     Content type = text/plain;charset=UTF-8
             Body = USER Resource.
    Forwarded URL = null
   Redirected URL = null
          Cookies = []

..
```
