
JHipster提供四种安全机制

1. JSON Web Tokens (JWT)
2. Session-based authentication
3. OAuth2 and OpenID Connect
4. JHipster User Account and Authentication (UAA)

参考一些资料的结论：

1. Session和OAuth是有状态的安全机制，在微服务中使用不方便
2. oauth2有client和scope的概念，jwt没有。如果只是拿来用于颁布token的话，二者没区别。常用的bearer算法oauth. jwt都可以用。应用场景不同而已
3. Spring Cloud 的权限框架就是用的jwt实现的oauth2 。二者没有必然联系
4. Token功能不一样，JWT的token是包含用户基本信息的，然后通过加密的方式生成的字符串，服务器端拿到这个token之后不需要再去查询用户基本信息，解析完token之后就能拿到。想想在微服务架构下，用户服务是一个单独的服务，但是其他服务大部分情况下也会需要用户信息，难道要每次用到都去取一次吗？ JWT非常适合微服务。
5. OAuth2用在使用第三方账号登录的情况(比如使用weibo, qq, github登录某个app)。OAuth2是一个相对复杂的协议, 有4种授权模式, 其中的access code模式在实现时可以使用jwt才生成code, 也可以不用. 它们之间没有必然的联系.
6. JWT是用在前后端分离, 需要简单的对后台API进行保护时使用.(前后端分离无session, 频繁传用户密码不安全)
7. JWT是一种认证协议 。JWT提供了一种用于发布接入令牌（Access Token),并对发布的签名接入令牌进行验证的方法。 令牌（Token）本身包含了一系列声明，应用程序可以根据这些声明限制用户对资源的访问。
8. OAuth2是一种授权框架。提供了一套详细的授权机制（指导）。用户或应用可以通过公开的或私有的设置，授权第三方应用访问特定资源。
9. 综上：微服务中认证选用JWT

### OAuth2 demo

```jdl
application {
  config {
    baseName gateway
    packageName org.aasrnd.gateway
    applicationType gateway
    authenticationType oauth2
    prodDatabaseType postgresql
    serviceDiscoveryType eureka
    serverPort 8880
    testFrameworks [cucumber, gatling]
    languages [zh-cn, en]
    nativeLanguage zh-cn
  }
  entities Blog, Post, Tag, Product
}

application {
  config {
    baseName blog
    packageName org.aasrnd.blog
    applicationType microservice
    authenticationType oauth2
    prodDatabaseType postgresql
    serverPort 8881
    serviceDiscoveryType eureka
    testFrameworks [cucumber, gatling]
    languages [zh-cn, en]
    nativeLanguage zh-cn
  }
  entities Blog, Post, Tag
}

application {
  config {
    baseName store
    packageName org.aasrnd.store
    applicationType microservice
    authenticationType oauth2
    databaseType mongodb
    devDatabaseType mongodb
    prodDatabaseType mongodb
    enableHibernateCache false
    serverPort 8882
    serviceDiscoveryType eureka
    testFrameworks [cucumber, gatling]
    languages [zh-cn, en]
    nativeLanguage zh-cn
  }
  entities Product
}

entity Blog {
  name String required minlength(3)
  handle String required minlength(2)
}
entity Post {
  title String required
  content TextBlob required
  date Instant required
}
entity Tag {
  name String required minlength(2)
}
entity Product {
  title String required
  price BigDecimal required min(0)
  image ImageBlob
}
relationship ManyToOne {
  Blog{user(login)} to User
  Post{blog(name)} to Blog
}
relationship ManyToMany {
  Post{tag(name)} to Tag{post}
}
paginate Post, Tag with infinite-scroll
paginate Product with pagination
microservice Product with store
microservice Blog, Post, Tag with blog
deployment {
  deploymentType docker-compose
  appsFolders [ gateway, blog, store]
  dockerRepositoryName "valdanito"
  consoleOptions [zipkin]
}
```

### UAA demo

```jdl
application {
  config {
    baseName uaa
    packageName org.aasrnd.uaa
    applicationType uaa
    authenticationType uaa
    prodDatabaseType postgresql
    serviceDiscoveryType eureka
    cacheProvider hazelcast
    testFrameworks [cucumber, gatling]
    languages [zh-cn, en]
    nativeLanguage zh-cn
  }
}

application {
  config {
    baseName gateway
    packageName org.aasrnd.gateway
    applicationType gateway
    authenticationType uaa
    uaaBaseName "uaa"
    prodDatabaseType postgresql
    serviceDiscoveryType eureka
    serverPort 8880
    testFrameworks [cucumber, gatling]
    languages [zh-cn, en]
    nativeLanguage zh-cn
  }
  entities Blog, Post, Tag, Product
}

application {
  config {
    baseName blog
    packageName org.aasrnd.blog
    applicationType microservice
    authenticationType uaa
    uaaBaseName "uaa"
    prodDatabaseType postgresql
    serverPort 8881
    serviceDiscoveryType eureka
    testFrameworks [cucumber, gatling]
    languages [zh-cn, en]
    nativeLanguage zh-cn
  }
  entities Blog, Post, Tag
}

application {
  config {
    baseName store
    packageName org.aasrnd.store
    applicationType microservice
    authenticationType uaa
    uaaBaseName "uaa"
    databaseType mongodb
    devDatabaseType mongodb
    prodDatabaseType mongodb
    enableHibernateCache false
    serverPort 8882
    serviceDiscoveryType eureka
    testFrameworks [cucumber, gatling]
    languages [zh-cn, en]
    nativeLanguage zh-cn
  }
  entities Product
}

entity Blog {
  name String required minlength(3)
  handle String required minlength(2)
}
entity Post {
  title String required
  content TextBlob required
  date Instant required
}
entity Tag {
  name String required minlength(2)
}
entity Product {
  title String required
  price BigDecimal required min(0)
  image ImageBlob
}
relationship ManyToOne {
  Blog{user(login)} to User
  Post{blog(name)} to Blog
}
relationship ManyToMany {
  Post{tag(name)} to Tag{post}
}
paginate Post, Tag with infinite-scroll
paginate Product with pagination
microservice Product with store
microservice Blog, Post, Tag with blog

deployment {
  deploymentType docker-compose
  appsFolders [uaa, gateway, blog, store]
  dockerRepositoryName "valdanito"
  consoleOptions [zipkin]
}
```

## 参考资料

- https://www.jhipster.tech/using-uaa/
- https://blog.csdn.net/f641385712/article/details/83930699