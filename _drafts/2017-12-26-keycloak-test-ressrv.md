---
title: Testing examples for Resource Server as Keycloak Client
category: keycloak
tags:
- keycloak
- spring boot
- spring security
- kotlin
- junit
- testing
---

## 概要
Keycloak による SSO を利用する、リソースサーバー (Spring Boot クライアント) でのテスト実装メモ。

この手順で使用したコードは、以下に公開しているので、こちらも参考にしてください。<br>
https://github.com/yo1000/kc-resource

また、テストコード以外の部分については、
[過去のポスト](http://blog.yo1000.com/keycloak/keycloak-collabo-ressrv-rescli.html)を前提としています。
関連するものについては軽く触れますが、詳細を確認したい場合は、そちらを確認してください。

## 要件
### 環境
今回の作業環境は以下のとおりです。

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

## テストターゲットの準備
### プロジェクトの作成
Spring Initializr でプロジェクトを作成します。

```console
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
  -d description="Keycloak Client Boilerplate - Resource Server" \
  -d packageName="com.yo1000.keycloak.resource.server" \
  -d baseDir="kc-resource-server" \
  -d applicationName="KcResourceServerApplication" \
  | tar -xzvf -

$ ls kc-resource-server
mvnw		mvnw.cmd	pom.xml		src

$ cd kc-resource-server
```

### 設定ファイルの配置
以下、2ファイルを変更します。

- `pom.xml`
- `application.yml`

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

### セキュリティ構成の実装
内容の詳細については、以下を確認してください。<br>
[/keycloak/keycloak-collabo-ressrv-rescli.html#implements-security-configuration-for-resource-server](http://blog.yo1000.com/keycloak/keycloak-collabo-ressrv-rescli.html#implements-security-configuration-for-resource-server) 

ここでは、テストを理解するのに役立つ実装の一部のみ説明します。

### grantedAuthoritiesMapper(): GrantedAuthoritiesMapper
認証基盤でロール名を小文字や、大文字小文字混在で設定しても、
`mapper.setConvertToUpperCase(true)` を設定することで、
プログラムから扱う場合に、すべて大文字で統一することができます。

### configure(http: HttpSecurity)
エンドポイントと、そのアクセスに必要なロールのマッピングを定義します。

`antMatchers("/kc/resource/server/admin").hasRole("ADMIN")` では、
`ADMIN` ロールをもっているユーザーのみが、`/kc/resource/server/admin` にアクセスできるように設定されます。

`antMatchers("/kc/resource/server/user").hasRole("USER")` では、
`USER` ロールをもっているユーザーのみが、`/kc/resource/server/user` にアクセスできるように設定されます。

```KcSecurityConfigurer.kt
package com.yo1000.keycloak.resource.server

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
```

### コントローラーの実装
リソースを返却するエンドポイントとなる、API 用コントローラーを実装します。

```KcResourceServerController.kt
package com.yo1000.keycloak.resource.server

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
```

## テストの実装
### テストコード
[Spek](http://spekframework.org/) で書いてしまいたいところですが、
Spek だとフィールドインジェクションとの相性が非常に悪いので、JUnit で書いてしまったほうがすっきり書けます。
DI を必要とするテストについては、(現時点では) Spek の適用は避けたほうが良いといえるでしょう。

```KcResourceServerControllerTests.kt
package com.yo1000.keycloak.resource.server

import org.junit.Before
import org.junit.Test
import org.junit.runner.RunWith
import org.keycloak.KeycloakPrincipal
import org.keycloak.adapters.RefreshableKeycloakSecurityContext
import org.keycloak.adapters.springsecurity.account.SimpleKeycloakAccount
import org.keycloak.adapters.springsecurity.token.KeycloakAuthenticationToken
import org.mockito.Mockito
import org.springframework.beans.factory.annotation.Autowired
import org.springframework.boot.test.context.SpringBootTest
import org.springframework.security.test.web.servlet.request.SecurityMockMvcRequestPostProcessors
import org.springframework.security.test.web.servlet.setup.SecurityMockMvcConfigurers
import org.springframework.test.context.junit4.SpringRunner
import org.springframework.test.web.servlet.MockMvc
import org.springframework.test.web.servlet.request.MockMvcRequestBuilders
import org.springframework.test.web.servlet.result.MockMvcResultHandlers
import org.springframework.test.web.servlet.result.MockMvcResultMatchers
import org.springframework.test.web.servlet.setup.DefaultMockMvcBuilder
import org.springframework.test.web.servlet.setup.MockMvcBuilders
import org.springframework.web.context.WebApplicationContext

@RunWith(SpringRunner::class)
@SpringBootTest(webEnvironment= SpringBootTest.WebEnvironment.RANDOM_PORT)
class KcResourceServerControllerTests {
    @Autowired
    lateinit var context: WebApplicationContext
    lateinit var mockMvc: MockMvc

    @Before
    fun beforeTestEach() {
        mockMvc = MockMvcBuilders
                .webAppContextSetup(context)
                .apply<DefaultMockMvcBuilder>(SecurityMockMvcConfigurers.springSecurity())
                .build()
    }

    /**
     * When the user has Admin and User roles,
     * then can access endpoints that require Admin role or User role.
     */
    @Test
    fun when_the_user_has_Admin_and_User_roles_then_can_access_endpoints_that_require_Admin_role_or_User_role() {
        val token = KeycloakAuthenticationToken(
                SimpleKeycloakAccount(
                        Mockito.mock(KeycloakPrincipal::class.java),
                        setOf("admin", "user"),
                        Mockito.mock(RefreshableKeycloakSecurityContext::class.java)),
                false)

        mockMvc.perform(MockMvcRequestBuilders
                .get("/kc/resource/server/admin")
                .with(SecurityMockMvcRequestPostProcessors
                        .authentication(token)))
                .andDo(MockMvcResultHandlers
                        .print())
                .andExpect(MockMvcResultMatchers
                        .status().isOk)

        mockMvc.perform(MockMvcRequestBuilders
                .get("/kc/resource/server/user")
                .with(SecurityMockMvcRequestPostProcessors
                        .authentication(token)))
                .andDo(MockMvcResultHandlers
                        .print())
                .andExpect(MockMvcResultMatchers
                        .status().isOk)
    }

    /**
     * When the user has only User roles,
     * then can access endpoints that require User role,
     * but can't access endpoints that require Admin role.
     */
    @Test
    fun when_the_user_has_only_User_roles_then_can_access_endpoints_that_require_User_role_but_cant_access_endpoints_that_require_Admin_role() {
        val token = KeycloakAuthenticationToken(
                SimpleKeycloakAccount(
                        Mockito.mock(KeycloakPrincipal::class.java),
                        setOf("user"),
                        Mockito.mock(RefreshableKeycloakSecurityContext::class.java)),
                false)

        mockMvc.perform(MockMvcRequestBuilders
                .get("/kc/resource/server/admin")
                .with(SecurityMockMvcRequestPostProcessors
                        .authentication(token)))
                .andDo(MockMvcResultHandlers
                        .print())
                .andExpect(MockMvcResultMatchers
                        .status().isForbidden)

        mockMvc.perform(MockMvcRequestBuilders
                .get("/kc/resource/server/user")
                .with(SecurityMockMvcRequestPostProcessors
                        .authentication(token)))
                .andDo(MockMvcResultHandlers
                        .print())
                .andExpect(MockMvcResultMatchers
                        .status().isOk)
    }
}
```



