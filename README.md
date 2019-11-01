# jhipster 实战笔记

    JHipster是一个免费的开源应用程序生成器，用于使用Angular或React（JavaScript库）和Spring Framework快速开发现代Web应用程序和微服务。

## 运行项目

- clone项目源码


- 方式一：本机运行
  
    安装docsify-cli, 关于[docsify](https://docsify.js.org)

    ```bash
    npm i docsify-cli -g
    ```

    在项目根目录下执行：

    ```bash
    docsify serve ./docs (-p 指定端口号)
    ```

    访问项目主页http://localhost:3000

- 方式二：docker容器运行
  
    在项目根目录下执行：

    ```bash
    docker-compose up -d
    ```

    访问项目主页http://localhost:3000