## 服务发现

JHipster提供两种服务发现

1. JHipster Registry
2. Consul

## JHipster Registry

### JHipster注册中心由以下各项组成

1. Netflix Eureka服务器
2. Spring Cloud配置服务器

#### JHipster注册中心有三个主要用途：

1. 它是一个Eureka server,作为应用程序的发现服务器。这就是JHipster如何处理所有应用程序的路由、负载平衡和可伸缩性。
2. 它是一个 Spring Cloud Config server, 为所有应用程序提供运行时配置的。
3. 它是一个管理服务器，带有用于监视和管理应用程序的仪表盘。

#### 可通过Spring Cloud 应用配置集中管理接入的应用的配置

1. 优点：提供两种配置源（Native configuration 和 Git configuration）。
2. 缺点：管理UI中只能对配置进行查看，修改配置需要在配置源修改
3. 缺点：只支持jhipster框架的springboot微服务注册

#### 提供管理仪表盘

1. 指标仪表盘（提供指标：the JVM；HTTP requests；cache usage； database connection pool）
2. 健康仪表盘
3. 配置仪表盘
4. 日志仪表盘

Eureka（JHipster注册中心）要求每个应用程序将其API用于注册并发现自己。它着重于可用性一致性。它仅支持用Spring Boot编写的应用程序或服务

### eureka demo
```jdl
application {
  config {
    baseName gateway
    packageName org.aasrnd.gateway
    applicationType gateway
    authenticationType jwt
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
    authenticationType jwt
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
    authenticationType jwt
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

## Consul

Consul提供服务发现，故障检测（健康发现），多数据中心配置和存储。

Consul的主要优点：

1. 使用go编写，具有较低的内存占用量
2. 可以与以任何编程语言编写的服务一起使用语言
3. 注重一致性而不是可用性，集群状态的更改传播得更快
4. 通过 DNS interface 或 HTTP API与现有应用程序进行简单的互操作
5. 在多节点集群中操作比Eureka更容易

缺点：

1. 没有JHipster Registry那么多丰富的仪表盘

Consul的Key/Value（多数据中心配置和存储）可在管理UI进行编辑操作

### consul demo

```jdl
application {
  config {
    baseName gateway
    packageName org.aasrnd.gateway
    applicationType gateway
    authenticationType jwt
    prodDatabaseType postgresql
    serviceDiscoveryType consul
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
    authenticationType jwt
    prodDatabaseType postgresql
    serverPort 8881
    serviceDiscoveryType consul
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
    authenticationType jwt
    databaseType mongodb
    devDatabaseType mongodb
    prodDatabaseType mongodb
    enableHibernateCache false
    serverPort 8882
    serviceDiscoveryType consul
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

## 建议

1. 若以后接入的都是基于jhipster框架的微服务框架，则二者都适用
2. 若对微服务集群的状态监控要求高的话，使用JHipster Registry
3. 若需接入非jhipster框架的微服务，consul更适合



## 参考资料

- https://www.jhipster.tech/jhipster-registry/
- https://www.jhipster.tech/consul/
- https://github.com/jhipster/jhipster-registry/issues/165