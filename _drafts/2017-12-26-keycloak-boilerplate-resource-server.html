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

## Summary
Keycloak による SSO を利用する Spring Boot クライアントでのテストについてのメモ。

この手順で使用したコードは、以下に公開しているので、こちらも参考にしてください。
https://github.com/yo1000/kc-resource

Keycloak 側の設定は、[/keycloak/keycloak-example-resource-server-client.html](http://blog.yo1000.com/keycloak/keycloak-example-resource-server-client.html) をご確認ください。

## プロジェクトの作成
Spring Initializr で、リソースサーバー用のプロジェクトを作成します。

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

## 構成ファイルのセットアップ
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

## セキュリティ構成の実装
セキュリティ構成を実装します。内容の詳細については、[/keycloak/keycloak-example-resource-server-client.html#implements-security-configuration-for-resource-server](http://blog.yo1000.com/keycloak/keycloak-example-resource-server-client.html#implements-security-configuration-for-resource-server) をご確認ください。

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


  
--
