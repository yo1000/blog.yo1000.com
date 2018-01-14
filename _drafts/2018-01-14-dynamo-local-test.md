---
title: DynamoDB Local を使用したテスト
category: dynamodb
tags:
- dynamodb
- kotlin
- testing
---

## 概要
[Amazon DynamoDB](https://docs.aws.amazon.com/ja_jp/amazondynamodb/latest/developerguide/Introduction.html) を、
AWS に依存せず、ローカルでテストする流れのメモ。

DynamoDB では、AWS を利用せずとも、ローカルで検証できるように、
AWS 自身から [DynaoDB Local](https://docs.aws.amazon.com/ja_jp/amazondynamodb/latest/developerguide/DynamoDBLocal.html) という
モジュールが提供されています。

このモジュールは AWS が管理している Maven リポジトリにもホスティングされており、これを利用することで、
事前に特別なコマンド等を発行することなく、JVM 言語のビルドプロセス過程で、ローカルに DynamoDB を用意することができるようになります。

このポストでは、Kotlin プロジェクトで、DynamoDB Local を使用してテストする場合に、
どのような設定が必要になるのかを中心に書いていきます。

この手順で使用したコードは、以下に公開しているので、こちらも参考にしてください。<br>
[https://github.com/yo1000/ddb-local/tree/40c9061dc6/ddb-local-test](https://github.com/yo1000/ddb-local/tree/40c9061dc671a88508a85937295306927fef1f0c/ddb-local-test)

### 目次

## 要件

### 環境
今回の作業環境は以下のとおりです。

- Java 1.8.0_131
- DynamoDB Local 1.11.86

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

## 依存関係解決

### pom.xml
ここが肝なので、とりあえず全文掲載してしまいます。要点は後述。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.yo1000</groupId>
    <artifactId>ddb-local-test</artifactId>
    <version>1.0.0-SNAPSHOT</version>
    <packaging>jar</packaging>

    <name>ddb-local-test</name>
    <description>DynamoDB Local Testing Example</description>

    <properties>
        <kotlin.compiler.incremental>true</kotlin.compiler.incremental>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
        <java.version>1.8</java.version>
        <kotlin.version>1.2.10</kotlin.version>
        <dynamodblocal.version>[1.11,2.0)</dynamodblocal.version>
        <sqlite4java.version>1.0.392</sqlite4java.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.jetbrains.kotlin</groupId>
            <artifactId>kotlin-stdlib-jre8</artifactId>
            <version>${kotlin.version}</version>
        </dependency>
        <dependency>
            <groupId>org.jetbrains.kotlin</groupId>
            <artifactId>kotlin-reflect</artifactId>
            <version>${kotlin.version}</version>
        </dependency>

        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.12</version>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>com.amazonaws</groupId>
            <artifactId>DynamoDBLocal</artifactId>
            <version>${dynamodblocal.version}</version>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>com.almworks.sqlite4java</groupId>
            <artifactId>sqlite4java</artifactId>
            <version>${sqlite4java.version}</version>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <build>
        <sourceDirectory>${project.basedir}/src/main/kotlin</sourceDirectory>
        <testSourceDirectory>${project.basedir}/src/test/kotlin</testSourceDirectory>
        <plugins>
            <plugin>
                <artifactId>kotlin-maven-plugin</artifactId>
                <groupId>org.jetbrains.kotlin</groupId>
                <version>${kotlin.version}</version>
                <executions>
                    <execution>
                        <id>compile</id>
                        <phase>compile</phase>
                        <goals>
                            <goal>compile</goal>
                        </goals>
                    </execution>
                    <execution>
                        <id>test-compile</id>
                        <phase>test-compile</phase>
                        <goals>
                            <goal>test-compile</goal>
                        </goals>
                    </execution>
                </executions>
                <configuration>
                    <jvmTarget>${java.version}</jvmTarget>
                </configuration>
                <dependencies>
                    <dependency>
                        <groupId>org.jetbrains.kotlin</groupId>
                        <artifactId>kotlin-maven-allopen</artifactId>
                        <version>${kotlin.version}</version>
                    </dependency>
                </dependencies>
            </plugin>

            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-surefire-plugin</artifactId>
                <version>2.19.1</version>
                <configuration>
                    <argLine>-Dsqlite4java.library.path=${basedir}/target/dependencies</argLine>
                </configuration>
            </plugin>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-dependency-plugin</artifactId>
                <version>2.10</version>
                <executions>
                    <execution>
                        <id>copy-dependencies</id>
                        <phase>process-test-resources</phase>
                        <goals>
                            <goal>copy-dependencies</goal>
                        </goals>
                        <configuration>
                            <outputDirectory>${project.build.directory}/dependencies</outputDirectory>
                            <overWriteReleases>false</overWriteReleases>
                            <overWriteSnapshots>false</overWriteSnapshots>
                            <overWriteIfNewer>true</overWriteIfNewer>
                        </configuration>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>

    <repositories>
        <repository>
            <id>dynamodb-local-oregon</id>
            <name>DynamoDB Local Release Repository</name>
            <url>https://s3-us-west-2.amazonaws.com/dynamodb-local/release</url>
        </repository>
        <repository>
            <id>dynamodb-local-tokyo</id>
            <name>DynamoDB Local Release Repository</name>
            <url>https://s3-ap-northeast-1.amazonaws.com/dynamodb-local-tokyo/release</url>
        </repository>
    </repositories>
</project>
```

#### sqlite4java
なぜ DynamoDB を使うのに、SQLite が出てくるのか、という部分ですが、DynamoDB Local では、

- DynamoDB とインターフェースが等しいこと
- データを永続化できること

の2点が満たせればよいだけなので、内部実装としては、SQLite を使用した永続化が行われています。
そのため、この依存が必要になってきます。

`maven-surefire-plugin` で、引数に `-Dsqlite4java.library.path=${basedir}/target/dependencies` を渡して、
SQLite のライブラリパスを指定している部分からも、これを読み取ることができます。

#### maven-dependency-plugin
`outputDirectory` に、`${project.build.directory}/dependencies` を設定して、
通常、`~/.m2/repository` 配下にしか保存されない依存ライブラリを、自身の環境下にコピーします。

この指定により、`maven-surefire-plugin` で、
引数に設定した `-Dsqlite4java.library.path=${basedir}/target/dependencies` を参照可能になります。

#### dynamodb-local-oregon, dynamodb-local-tokyo
DynamoDB Local は、Maven Central にアップロードされておらず、
Amazon S3 上に、必要なファイル群がアップロードされているだけなので、
追加リポジトリの指定が必要になります。

また、依存解決するロケーションにより、ネットワーク的に有利なリージョンを選択することが可能です。
使用可能なリージョン別のリポジトリについては、以下を確認してください。<br>
[https://docs.aws.amazon.com/ja_jp/amazondynamodb/latest/developerguide/DynamoDBLocal.html#DynamoDBLocal.Maven](https://docs.aws.amazon.com/ja_jp/amazondynamodb/latest/developerguide/DynamoDBLocal.html#DynamoDBLocal.Maven)

