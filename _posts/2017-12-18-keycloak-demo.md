---
title: Keycloak DEMO
tags:
- keycloak
- wip
---

Keycloak による SSO 検証のデモサイト構築メモ。
SSO サーバーのセットアップと、SSO クライアントの開発を順に見ていく。

## Summary

- [Requirements](#requirements)
- [Set up Keycloak (SSO Server)](#set-up-keycloak-sso-server)
  - [Download & Unarchive](#download--unarchive)
  - [Initial settings for Keycloak](#initial-settings-for-keycloak)
  - [Set up Realm](#set-up-realm)
  - [Set up Users](#set-up-users)
  - [Set up Clients](#set-up-clients)
- [Develop Resource Server (SSO Client - RSrv)](#develop-resource-server-sso-client---rsrv)
  - [Create Project for Resource Server](#create-project-for-resource-server)
  - [Set up Configuration files for Resource Server](#set-up-configuration-files-for-resource-server)
  - [Implements Security Configuration for Resource Server](#implements-security-configuration-for-resource-server)
  - [Implements RestController for Resource Server](#implements-restcontroller-for-resource-server)
  - [Build and Run Resource Server](#build-and-run-resource-server)
- [Develop Resource Client (SSO Client - RCli)](#develop-resource-client-sso-client---rcli)
  - [Create Project for Resource Client](#create-project-for-resource-client)
  - [Set up Configuration files for Resource Client](#set-up-configuration-files-for-resource-client)
  - [Implements Security Configuration for Resource Client](#implements-security-configuration-for-resource-client)
  - [Implements Controller with use KeycloakRestTemplate for Resource Client](#implements-controller-with-use-keycloakresttemplate-for-resource-client)
  - [Build and Run Resource Client](#build-and-run-resource-client)

## Requirements

今回の作業環境は以下。

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

TODO: 今回の構成図やインフラ要件を書く

以降、一連の流れを実施するにあたり、ディレクトリ移動が数回発生するため、
便宜上、`${BASE_DIR}` をディレクトリの基点として使用します。

```console
$ BASE_DIR=`pwd`
```

## Set up Keycloak (SSO Server)

### Download & Unarchive

執筆時点での最新は、[3.4.1.Final](http://www.keycloak.org/archive/downloads-3.4.1.html)<br>
Download URL:
[https://downloads.jboss.org/keycloak/3.4.1.Final/keycloak-3.4.1.Final.tar.gz](https://downloads.jboss.org/keycloak/3.4.1.Final/keycloak-3.4.1.Final.tar.gz)

```console
$ cd ${BASE_DIR}
$ curl https://downloads.jboss.org/keycloak/3.4.1.Final/keycloak-3.4.1.Final.tar.gz | tar -zxvf -
$ cd keycloak-3.4.1.Final
```

### Initial settings for Keycloak

管理ユーザーは、以下いずれかの方法で追加する。

- ホスト内の `add-user-keycloak.sh` による登録 _(今回はこちらを使用)_
- 同一ホストからのアクセスによる Admin Console での登録

今回はすべてローカルホストに構築しているため、気にする必要はないが、
セットアップ後、初めて作成する管理ユーザーは、 __リモートホストから、直接追加することができない__ ため、
セットアップ時に一緒に作成しておく必要がある。

```console
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
```

### Login to Keycloak

以降、`kcadm.sh` を使用する上で、ログイン状態が必要になるため、ログインする。
`kcadm.sh` 実行時に以下のようなメッセージが出力された場合は、
ログインセッションが期限切れとなっているため、改めてログインする。

> Session has expired. Login again with 'kcadm.sh config credentials'

```console
$ # Login to Keycloak
$ bin/kcadm.sh config credentials --server http://127.0.0.1:8080/auth --realm master --user keycloak --password keycloak1234
Logging into http://127.0.0.1:8080/auth as user admin of realm master
```

### Set up Realm

レルムは `領域・範囲・部門` といった単語に略されるもので、
そのログインが、どのような種類のリソースにアクセスするためのものかを管理するための単位。
より端的には、レルムが分離されていると、ログイン画面の URL が分離する。

使用例としては、顧客が使用するシステムと、システム管理者が使用するシステムで、
同じ SSO 基盤 (Keycloak) を使用しながらも、
ログインフォームや、ログイン後に使用されるロールやグループの管理を完全に分離したいような場合に活用できる。

```console
$ # Create realm
$ bin/kcadm.sh create realms -s realm=kc-resource -s enabled=true
Created new realm with id 'kc-resource'

$ # Create realm roles
$ bin/kcadm.sh create roles -r kc-resource -s name=admin
Created new role with id 'admin'
$ bin/kcadm.sh create roles -r kc-resource -s name=user
Created new role with id 'user'
```

### Set up Users

`kc-resource` レルムにユーザーを追加していく。
管理ユーザーとして `alice` を、一般ユーザーとして `bob` を、それぞれ作成する。

```console
$ # Create realm users
$ bin/kcadm.sh create users -r kc-resource -s username=alice -s enabled=true
Created new user with id '26c64aeb-6d09-4d58-afac-fd7550d4ff7b'
$ bin/kcadm.sh create users -r kc-resource -s username=bob -s enabled=true
Created new user with id '8ab8768a-a2b1-479d-be11-3d7dd1b3d3db'

$ # Update password
$ bin/kcadm.sh set-password -r kc-resource --username alice -p alice1234
$ bin/kcadm.sh set-password -r kc-resource --username bob -p bob1234

$ # Add realm roles to users
$ bin/kcadm.sh add-roles -r kc-resource --uusername alice --rolename admin --rolename user
$ bin/kcadm.sh add-roles -r kc-resource --uusername bob --rolename user
```

### Set up Clients

SSO 基盤を使用するアプリケーション (SSO サーバーに対する、クライアント) を登録する。
リソースサーバーとして `kc-resource-server` を、リソースクライアントとして `kc-resource-client` をそれぞれ作成する。

リソースサーバー、リソースクライアントという、 __リソースに対して、サーバー・クライアントの関係にある2つのアプリケーション__ を作成するが、
これらアプリケーションは、 __いずれも SSO サーバーから見れば、等しく SSO クライアント__ となる。

```console
$ # Add realm client for Resource server
$ RES_SRV_ID=`bin/kcadm.sh create clients -r kc-resource -s clientId=kc-resource-server -s bearerOnly=true -i`; echo $RES_SRV_ID
58f8a1ad-a409-4f22-9bb8-de10f9ca5365

$ # Add realm client for Resource client
$ RES_CLI_ID=`bin/kcadm.sh create clients -r kc-resource -s clientId=kc-resource-client -s 'redirectUris=["http://localhost:28080/*"]' -i`; echo $RES_CLI_ID
373d1ce7-19c2-4a40-b1a3-deb3e4a02c83
```

## Develop Resource Server (SSO Client - RSrv)

### Create Project for Resource Server

Spring Initializr で、リソースサーバー用のプロジェクトを作成します。

```console
$ cd ${BASE_DIR}
$ curl https://start.spring.io/starter.tgz \
  -d dependencies="web,security,keycloak" \
  -d language="kotlin" \
  -d javaVersion="1.8" \
  -d packaging="jar" \
  -d bootVersion="1.5.9.RELEASE" \
  -d type="maven-project" \
  -d groupId="com.yo1000" \
  -d artifactId="kc-resource-server" \
  -d version="1.0.0-SNAPSHOT" \
  -d name="kc-resource-server" \
  -d description="Keycloak Client Demo - Resource Server" \
  -d packageName="com.yo1000.keycloak.resource.server" \
  -d baseDir="kc-resource-server" \
  -d applicationName="KcResourceServerApplication" \
  | tar -xzvf -

$ ls kc-resource-server
mvnw		mvnw.cmd	pom.xml		src

$ cd kc-resource-server
```

### Set up Configuration files for Resource Server

リソースサーバー用の、構成ファイルをセットアップします。

```console
$ sed -i '' \
  's/<keycloak.version>3.4.0.Final<\/keycloak.version>/<keycloak.version>3.4.1.Final<\/keycloak.version>/g' \
  pom.xml

$ mv \
  src/main/resources/application.properties \
  src/main/resources/application.yml

$ echo 'server.port: 18080

keycloak:
  realm: kc-resource
  resource: kc-resource-server
  bearer-only: true
  auth-server-url: http://127.0.0.1:8080/auth
  ssl-required: external
' > src/main/resources/application.yml
```

### Implements Security Configuration for Resource Server

```console
$ echo 'package com.yo1000.keycloak.resource.server

import org.keycloak.adapters.springsecurity.config.KeycloakWebSecurityConfigurerAdapter
import org.springframework.context.annotation.Bean
import org.springframework.context.annotation.Configuration
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity
import org.springframework.security.web.authentication.session.NullAuthenticatedSessionStrategy
import org.springframework.security.web.authentication.session.SessionAuthenticationStrategy
import org.springframework.security.core.authority.mapping.SimpleAuthorityMapper
import org.springframework.security.core.authority.mapping.GrantedAuthoritiesMapper
import org.keycloak.adapters.springsecurity.authentication.KeycloakAuthenticationProvider
import org.springframework.security.config.annotation.authentication.builders.AuthenticationManagerBuilder
import org.springframework.security.config.annotation.web.builders.HttpSecurity
import org.springframework.boot.web.servlet.FilterRegistrationBean
import org.keycloak.adapters.springsecurity.filter.KeycloakPreAuthActionsFilter
import org.keycloak.adapters.springsecurity.filter.KeycloakAuthenticationProcessingFilter
import org.keycloak.adapters.springboot.KeycloakSpringBootConfigResolver
import org.keycloak.adapters.KeycloakConfigResolver

@Configuration
@EnableWebSecurity
class KcSecurityConfigurer: KeycloakWebSecurityConfigurerAdapter() {
    @Bean
    fun grantedAuthoritiesMapper(): GrantedAuthoritiesMapper {
        val mapper = SimpleAuthorityMapper()
        mapper.setConvertToUpperCase(true)
        return mapper
    }

    @Bean
    fun keycloakConfigResolver(): KeycloakConfigResolver {
        return KeycloakSpringBootConfigResolver()
    }

    @Bean
    fun keycloakAuthenticationProcessingFilterRegistrationBean(
            filter: KeycloakAuthenticationProcessingFilter): FilterRegistrationBean {
        val registrationBean = FilterRegistrationBean(filter)
        registrationBean.isEnabled = false
        return registrationBean
    }

    @Bean
    fun keycloakPreAuthActionsFilterRegistrationBean(
            filter: KeycloakPreAuthActionsFilter): FilterRegistrationBean {
        val registrationBean = FilterRegistrationBean(filter)
        registrationBean.isEnabled = false
        return registrationBean
    }

    override fun sessionAuthenticationStrategy(): SessionAuthenticationStrategy {
        return NullAuthenticatedSessionStrategy()
    }

    override fun keycloakAuthenticationProvider(): KeycloakAuthenticationProvider {
        val provider = super.keycloakAuthenticationProvider()
        provider.setGrantedAuthoritiesMapper(grantedAuthoritiesMapper())
        return provider
    }

    override fun configure(auth: AuthenticationManagerBuilder?) {
        auth!!.authenticationProvider(keycloakAuthenticationProvider())
    }

    override fun configure(http: HttpSecurity) {
        super.configure(http)
        http
                .authorizeRequests()
                .antMatchers("/kc/resource/server/admin").hasRole("ADMIN")
                .antMatchers("/kc/resource/server/user").hasRole("USER")
                .anyRequest().permitAll()
    }
}
' > src/main/kotlin/com/yo1000/keycloak/resource/server/KcSecurityConfigurer.kt
```

### Implements RestController for Resource Server

```console
$ echo 'package com.yo1000.keycloak.resource.server

import org.springframework.web.bind.annotation.GetMapping
import org.springframework.web.bind.annotation.RequestMapping
import org.springframework.web.bind.annotation.RestController

@RestController
@RequestMapping("/kc/resource/server")
class KcResourceServerController {
    @GetMapping("/admin")
    fun getAdminResource(): String {
        return "ADMIN Resource!!"
    }

    @GetMapping("/user")
    fun getUserResource(): String {
        return "USER Resource."
    }
}
' > src/main/kotlin/com/yo1000/keycloak/resource/server/KcResourceServerController.kt 
```

### Build and Run Resource Server

```console
$ ./mvnw clean spring-boot:run &
```

## Develop Resource Client (SSO Client - RCli)

### Create Project for Resource Client

Spring Initializr で、リソースクライアント用のプロジェクトを作成します。

```console
$ cd ${BASE_DIR}
$ curl https://start.spring.io/starter.tgz \
  -d dependencies="web,security,keycloak" \
  -d language="kotlin" \
  -d javaVersion="1.8" \
  -d packaging="jar" \
  -d bootVersion="1.5.9.RELEASE" \
  -d type="maven-project" \
  -d groupId="com.yo1000" \
  -d artifactId="kc-resource-client" \
  -d version="1.0.0-SNAPSHOT" \
  -d name="kc-resource-client" \
  -d description="Keycloak Client Demo - Resource Client" \
  -d packageName="com.yo1000.keycloak.resource.client" \
  -d baseDir="kc-resource-client" \
  -d applicationName="KcResourceClientApplication" \
  | tar -xzvf -

$ ls kc-resource-client
mvnw		mvnw.cmd	pom.xml		src

$ cd kc-resource-client
```

### Set up Configuration files for Resource Client

リソースクライアント用の、構成ファイルをセットアップします。

こちらでは、リソースサーバーでは設定しなかった `keycloak.json` が必要になります。
[Set up Clients](#set-up-clients) で、`$RES_CLI_ID` 変数に取ったクライアント ID を使用して、`keycloak.json` を出力します。

```console
$ # Update Keycloak dependency version
$ sed -i '' \
  's/<keycloak.version>3.4.0.Final<\/keycloak.version>/<keycloak.version>3.4.1.Final<\/keycloak.version>/g' \
  pom.xml

$ # Configure application.yml
$ mv \
  src/main/resources/application.properties \
  src/main/resources/application.yml
$ echo 'server.port: 28080

keycloak:
  realm: kc-resource
  resource: kc-resource-client
  auth-server-url: http://127.0.0.1:8080/auth
' > src/main/resources/application.yml

$ # Install credentials
$ mkdir -p src/main/webapp/WEB-INF
$ ${BASE_DIR}/keycloak-3.4.1.Final/bin/kcadm.sh \
  get clients/${RES_CLI_ID}/installation/providers/keycloak-oidc-keycloak-json \
  -r kc-resource \
  > src/main/webapp/WEB-INF/keycloak.json 
```

### Implements Security Configuration for Resource Client

```console
$ echo 'package com.yo1000.keycloak.resource.client

import org.keycloak.adapters.springsecurity.KeycloakConfiguration
import org.keycloak.adapters.springsecurity.authentication.KeycloakAuthenticationProvider
import org.keycloak.adapters.springsecurity.client.KeycloakClientRequestFactory
import org.keycloak.adapters.springsecurity.client.KeycloakRestTemplate
import org.keycloak.adapters.springsecurity.config.KeycloakWebSecurityConfigurerAdapter
import org.keycloak.adapters.springsecurity.filter.KeycloakAuthenticationProcessingFilter
import org.keycloak.adapters.springsecurity.filter.KeycloakPreAuthActionsFilter
import org.springframework.beans.factory.annotation.Autowired
import org.springframework.beans.factory.config.ConfigurableBeanFactory
import org.springframework.boot.autoconfigure.SpringBootApplication
import org.springframework.boot.autoconfigure.condition.ConditionalOnClass
import org.springframework.boot.web.servlet.FilterRegistrationBean
import org.springframework.context.annotation.Bean
import org.springframework.context.annotation.Scope
import org.springframework.security.config.annotation.authentication.builders.AuthenticationManagerBuilder
import org.springframework.security.config.annotation.web.builders.HttpSecurity
import org.springframework.security.core.session.SessionRegistryImpl
import org.springframework.security.web.authentication.session.RegisterSessionAuthenticationStrategy
import org.springframework.security.web.authentication.session.SessionAuthenticationStrategy
import org.springframework.security.core.authority.mapping.SimpleAuthorityMapper
import org.springframework.security.core.authority.mapping.GrantedAuthoritiesMapper

@KeycloakConfiguration
class SecurityConfiguration : KeycloakWebSecurityConfigurerAdapter() {
    @Autowired
    fun configureGlobal(auth: AuthenticationManagerBuilder) {
        auth.authenticationProvider(keycloakAuthenticationProvider())
    }

    @Bean
    @Scope(ConfigurableBeanFactory.SCOPE_PROTOTYPE)
    fun keycloakRestTemplate(keycloakClientRequestFactory: KeycloakClientRequestFactory): KeycloakRestTemplate {
        return KeycloakRestTemplate(keycloakClientRequestFactory)
    }

    @Bean
    override fun sessionAuthenticationStrategy(): SessionAuthenticationStrategy {
        return RegisterSessionAuthenticationStrategy(SessionRegistryImpl())
    }

    @Bean
    fun keycloakAuthenticationProcessingFilterRegistrationBean(
            filter: KeycloakAuthenticationProcessingFilter): FilterRegistrationBean {
        val registrationBean = FilterRegistrationBean(filter)
        registrationBean.isEnabled = false
        return registrationBean
    }

    @Bean
    @ConditionalOnClass(SpringBootApplication::class)
    fun keycloakPreAuthActionsFilterRegistrationBean(
            filter: KeycloakPreAuthActionsFilter): FilterRegistrationBean {
        val registrationBean = FilterRegistrationBean(filter)
        registrationBean.isEnabled = false
        return registrationBean
    }

    @Bean
    fun grantedAuthoritiesMapper(): GrantedAuthoritiesMapper {
        val mapper = SimpleAuthorityMapper()
        mapper.setConvertToUpperCase(true)
        return mapper
    }

    override fun keycloakAuthenticationProvider(): KeycloakAuthenticationProvider {
        val provider = super.keycloakAuthenticationProvider()
        provider.setGrantedAuthoritiesMapper(grantedAuthoritiesMapper())
        return provider
    }

    override fun configure(http: HttpSecurity) {
        super.configure(http)
        http
                .authorizeRequests()
                .antMatchers("/kc/resource/client/user").hasRole("USER")
                .antMatchers("/kc/resource/client/admin").hasRole("ADMIN")
                .anyRequest().permitAll()
    }
}
' > src/main/kotlin/com/yo1000/keycloak/resource/client/SecurityConfiguration.kt
```

### Implements Controller with use `KeycloakRestTemplate` for Resource Client

```console
$ echo 'package com.yo1000.keycloak.resource.client

import org.keycloak.adapters.springsecurity.client.KeycloakRestTemplate
import org.springframework.stereotype.Controller
import org.springframework.web.bind.annotation.GetMapping
import org.springframework.web.bind.annotation.RequestMapping
import org.springframework.web.bind.annotation.ResponseBody

@Controller
@RequestMapping("/kc/resource/client")
class KcResourceClientController(
        val template: KeycloakRestTemplate
) {
    companion object {
        const val ENDPOINT = "http://localhost:18080/{role}/resource"
    }

    @GetMapping("/admin")
    @ResponseBody
    fun getAdmin(): String {
        val resp = template.getForObject(ENDPOINT, String::class.java, mapOf("role" to "admin"))
        return """
            This page is Resource Client side.
            Resource Server Response: [$resp]
            """.trimIndent()
    }

    @GetMapping("/user")
    @ResponseBody
    fun getUser(): String {
        val resp = template.getForObject(ENDPOINT, String::class.java, mapOf("role" to "user"))
        return """
            This page is Resource Client side.
            Resource Server Response: [$resp]
            """.trimIndent()
    }
}
' > src/main/kotlin/com/yo1000/keycloak/resource/client/KcResourceClientController.kt
```

### Build and Run Resource Client

```console
$ ./mvnw clean spring-boot:run &
```

## Refs

- [http://www.keycloak.org/](http://www.keycloak.org/)
- [https://github.com/foo4u/keycloak-spring-demo](https://github.com/foo4u/keycloak-spring-demo)

- [http://blog.keycloak.org/2017/01/administer-keycloak-server-from-shell.html](http://blog.keycloak.org/2017/01/administer-keycloak-server-from-shell.html)
- [http://keycloak-documentation.openstandia.jp/master/ja_JP/securing_apps/index.html](http://keycloak-documentation.openstandia.jp/master/ja_JP/securing_apps/index.html)
- [https://sandor-nemeth.github.io/java/spring/2017/06/15/spring-boot-with-keycloak.html](https://sandor-nemeth.github.io/java/spring/2017/06/15/spring-boot-with-keycloak.html)
- [https://github.com/foo4u/keycloak-spring-demo](https://github.com/foo4u/keycloak-spring-demo)

### :thinking:

- [http://www.atmarkit.co.jp/ait/articles/1711/08/news009.html](http://www.atmarkit.co.jp/ait/articles/1711/08/news009.html
)
