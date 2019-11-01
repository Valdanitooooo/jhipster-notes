## Protractor的依赖被墙

 - 建议不使用Protractor做测试
 - npm/yarn代理

## 更新或导入新的jdl时跳过git自动提交, 方便对比代码的更改

```sh
  jhipster import-jdl apps.jh --skip-git
```

## noopener警告导致打包失败

`react/jsx-no-target-blank: Using target="_blank" without rel="noopener noreferrer" is a security risk: see https://mathiasbynens.github.io/rel-noopener`
解决办法：在相应位置的A标签加上`rel="noopener noreferrer"`

## Liquibase 连续报错

检查数据库中databasechangeloglock表中locked字段是否为true，为true说明已经锁住，将其改为false再重启
