
## demo

### Step 1 建模

```jdl
application {
  config {
    baseName gateway,
    packageName tech.jhipster.gateway,
    applicationType gateway,
    authenticationType jwt,
    prodDatabaseType postgresql,
    serviceDiscoveryType eureka,
    websocket spring-websocket,
    messageBroker kafka,
    nativeLanguage zh-cn,
    languages [en, zh-cn],
    testFrameworks [cucumber, gatling],
    serverPort 8580
  }
  entities *
}

application {
  config {
    baseName blog,
    packageName tech.jhipster.blog,
    applicationType microservice,
    authenticationType jwt,
    prodDatabaseType postgresql,
    serviceDiscoveryType eureka,
    messageBroker kafka,
    nativeLanguage zh-cn,
    languages [en, zh-cn],
    testFrameworks [cucumber, gatling],
    serverPort 8581
  }
  entities *
}

application {
  config {
    baseName comment,
    packageName tech.jhipster.comment,
    applicationType microservice,
    authenticationType jwt,
    databaseType sql,
    devDatabaseType h2Disk,
    prodDatabaseType postgresql,
    enableHibernateCache false,
    serviceDiscoveryType eureka,
    messageBroker kafka,
    nativeLanguage zh-cn,
    languages [en, zh-cn],
    testFrameworks [cucumber, gatling],
    serverPort 8582
  }
  entities *
}

entity Blog {
  name String required minlength(3),
  keyword String minlength(2)
}

entity Post {
  title String required,
  content TextBlob required,
  date Instant required
}

entity Tag {
  name String required minlength(2)
}

entity Comment {
  content TextBlob required,
  date Instant required
}

relationship OneToMany {
  Blog{post} to Post{blog}
  Post{comment} to Comment{tag}
}

relationship ManyToMany {
  Post{tag} to Tag{post}
}

paginate Post, Tag, Comment with pagination

deployment {
  deploymentType docker-compose
  appsFolders [gateway, blog, comment]
  dockerRepositoryName "valdanito"
  consoleOptions [zipkin]
}
```

### Step 2 打包镜像

先把jdk基础镜像替换成国内仓库的镜像
  在三个项目的pom文件中搜索openjdk，替换原有镜像为
  hub.c.163.com/cloudndp/library/openjdk:8

  修改gateway/src/main/webapp/app/modules/administration/gateway/gateway.tsx
  `<a href={instance.uri} target="_blank">`
  加入 `rel="noopener noreferrer"` 变为
  `<a href={instance.uri} target="_blank" rel="noopener noreferrer">`

可以选择依次打包：

```sh
  ./mvnw -ntp -Pprod verify jib:dockerBuild in gateway_dir
  ./mvnw -ntp -Pprod verify jib:dockerBuild in blog_dir
  ./mvnw -ntp -Pprod verify jib:dockerBuild in comment_dir
```

也可以创建一个pom文件，把三个项目组织起来

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>tech.jhipster</groupId>
    <artifactId>jhipster-parent</artifactId>
    <version>1.0.0-SNAPSHOT</version>
    <packaging>pom</packaging>
    <name>jhipster-parent</name>
    <modules>
        <module>gateway</module>
        <module>blog</module>
        <module>comment</module>
    </modules>
</project>
```

再在根目录下执行：

```sh
  mvn -Pprod verify com.google.cloud.tools:jib-maven-plugin:dockerBuild
```

### Step 3 启动

检查docker-compose.yml文件中的端口映射， 尽量不使用默认的8080，避免冲突。