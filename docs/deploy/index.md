
## 本地调试

### 单个微服务中测试
  使用Junit
  使用gateway中的swagger ui进行调试
### 多个微服务联调
  在本地启动一个consul容器，在IntelliJ IDEA中Run/Debug Configurations中的Override parameters中加入一条`spring.cloud.consul.host`, value为localhost。使用本地consul进行服务发现，不受线上的微服务影响。

## JHipster+gogs+jenkins+jira持续集成

### step 1

  在Jenkins中创建pipeline项目，配置gogs和jira地址，选择过滤掉的分支
  前提是jenkins中已经配置有拉取此项目代码权限的帐号

### step 2

  在gogs的仓库设置中添加webhook，推送地址设置为`{jenkins地址}/git/notifyCommit?url={gogs仓库的ssh地址}&delay=0sec`，其他保持默认。
  `delay=0sec`是指有新的提交0秒后就进行部署

### step 3

  1. 在JHipster项目下执行`jhipster ci-cd`
  2. 选择Jenkins pipeline
  3. 根据需要选择要包含的tasks/integrations
  4. 选择In Jenkins pipeline
  5. 修改Jenkinsfile，增加部署的和jira comment的步骤
  6. 要使jira自动评论成功，需要在jenkins中配置一个jira帐号

### step 4

  提交代码，在commit message的开头加上jira任务的id
  推送代码，查看jenkisn是否构建成功，构建成功后jira中是否有机器人的评论
