# sample-spring-session-with-servlet

[![GitHub release](https://img.shields.io/github/release/u6k/sample-spring-session-with-servlet.svg)](https://github.com/u6k/sample-spring-session-with-servlet/releases)
[![license](https://img.shields.io/github/license/u6k/sample-spring-session-with-servlet.svg)](https://github.com/u6k/sample-spring-session-with-servlet/blob/master/LICENSE)
[![standard-readme compliant](https://img.shields.io/badge/readme%20style-standard-brightgreen.svg?style=flat-square)](https://github.com/RichardLitt/standard-readme)

> Spring Sessionの挙動検証を行うサンプル

__Table of Contents:__

<!-- TOC depthFrom:2 -->

- [Install](#install)
- [Usage](#usage)
    - [Spring Sessionを組み込んだだけで、何も設定しない場合](#spring-sessionを組み込んだだけで何も設定しない場合)
        - [依存関係を追加](#依存関係を追加)
        - [Spring Java Configurationを作成](#spring-java-configurationを作成)
        - [Servletコンテナを初期化](#servletコンテナを初期化)

<!-- /TOC -->

## Install

Redisを起動します。

```
$ docker run -d -p 6379:6379 redis
```

Webアプリケーションを起動します。

```
$ ./mvnw jetty:run
```

[http://localhost:8080/](http://localhost:8080/)にアクセスすると、カウントが表示されます。

## Usage

[u6k/sample-http-session](https://github.com/u6k/sample-http-session)を基に、Spring Sessionを組み込んでWebアプリケーションを作成します。あらかじめ、sample-http-sessionをご覧いただけると、理解が進むと思います。

セッション・タイムアウトの設定と通信の挙動の関係、およびRedisの状態を調べます。

### Spring Sessionを組み込んだだけで、何も設定しない場合

[Spring Session - HttpSession (Quick Start)](https://docs.spring.io/spring-session/docs/current/reference/html5/guides/httpsession.html)を参考に実装を進めます。この検証では説明をいろいろ省略するので、詳細は[Spring Session - HttpSession (Quick Start)](https://docs.spring.io/spring-session/docs/current/reference/html5/guides/httpsession.html)をご覧ください。

#### 依存関係を追加

`pom.xml`の依存関係に、次の設定を追加します。

```
<dependency>
    <groupId>org.springframework.session</groupId>
    <artifactId>spring-session-data-redis</artifactId>
    <version>1.3.1.RELEASE</version>
    <type>pom</type>
</dependency>
<dependency>
    <groupId>biz.paluch.redis</groupId>
    <artifactId>lettuce</artifactId>
    <version>3.5.0.Final</version>
</dependency>
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-web</artifactId>
    <version>4.3.4.RELEASE</version>
</dependency>
```

#### Spring Java Configurationを作成

Spring設定を行う`Config`クラスを作成します。`src/main/java/me/u6k/sample/sample_spring_session_with_servlet/Config.java`ファイルを次のように作成します。

```
package me.u6k.sample.sample_spring_session_with_servlet;

import org.springframework.context.annotation.Bean;
import org.springframework.data.redis.connection.lettuce.LettuceConnectionFactory;
import org.springframework.session.data.redis.config.annotation.web.http.EnableRedisHttpSession;

@EnableRedisHttpSession
public class Config {
    @Bean
    public LettuceConnectionFactory connectionFactory() {
        return new LettuceConnectionFactory();
    }
}
```

`@EnableRedisHttpSession`によって、`HttpSession`をSpring Sessionで置き換えます。

`LettuceConnectionFactory`によって、`localhost:6379`で待ち受けるRedisに接続するように設定されます。

#### Servletコンテナを初期化

先ほどのSpring設定を適用するため、Servletコンテナの初期化を行う`Initializer`クラスを作成します。`src/main/java/me/u6k/sample/sample_spring_session_with_servlet/Initializer.java`ファイルを次のように作成します。

```
package me.u6k.sample.sample_spring_session_with_servlet;

import org.springframework.session.web.context.AbstractHttpSessionApplicationInitializer;

public class Initializer extends AbstractHttpSessionApplicationInitializer {
    public Initializer() {
        super(Config.class);
    }
}
```

コンストラクタの`super()`に先ほどの`Config`クラスを渡すことで、Spring設定を適用します。
