* 使用页面403问题

  * 问题根源

    增强了CSRF功能

    https://www.jenkins.io/doc/upgrade-guide/2.176/

  * 临时解决方案---刷新, 不断重试

  * 长久解决方案---禁用

    1. 安装Strict Crumb Issuer插件

    2. 在全局安全配置中, 使用该插件

       ![image-20201219155209282](.Jenkins/image-20201219155209282.png)

    3. 取消`Check the session ID`

       ![image-20201219155233646](.Jenkins/image-20201219155233646.png)

* 语言切换

  1. 安装插件`Locale plugin` , 该插件提供了语言切换功能

  2. Manage Jenkins --> Configure System --> Local

     中文填入`zh_CN` , 英文填入`en_US` , 同时勾选Ignore browser preference and force this language to all users

* 不同分支同一Jenkinsfile配置方案

  [End-to-End Multibranch Pipeline Project Creation](https://www.jenkins.io/doc/tutorials/build-a-multibranch-pipeline-project/)

  思路: 条件筛选`when`

# 安装

1. 创建Dockerfile

   ```dockerfile
   FROM jenkins/jenkins:2.263.1-lts-slim
   USER root
   RUN apt-get update && apt-get install -y apt-transport-https \
          ca-certificates curl gnupg2 \
          software-properties-common
   RUN curl -fsSL https://download.docker.com/linux/debian/gpg | apt-key add -
   RUN apt-key fingerprint 0EBFCD88
   RUN add-apt-repository \
          "deb [arch=amd64] https://download.docker.com/linux/debian \
          $(lsb_release -cs) stable"
   RUN apt-get update && apt-get install -y docker-ce-cli
   USER jenkins
   RUN jenkins-plugin-cli --plugins blueocean:1.24.3
   ```

   该镜像中引入了docker cli, docker engine的客户端.

2. 构建成镜像

   ```shell
   docker build -t myjenkins-blueocean:1.1 .
   ```

3. 运行容器

   ```shell
   docker run --name jenkins-blueocean \
     --detach \
     --env DOCKER_HOST=tcp://sh.sidian.live:2376 \
     --publish 8082:8080 --publish 50000:50000 \
     myjenkins-blueocean:1.1
   ```

   需要修改`DOCKER_HOST`, 指向docker engine

> 参考
>
> * https://www.jenkins.io/doc/book/installing/docker/
> * https://github.com/jenkinsci/docker/blob/master/README.md