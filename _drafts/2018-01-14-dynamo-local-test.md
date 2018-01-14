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

