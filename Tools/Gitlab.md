# 备份与恢复

* 备份限制

  只能恢复同版本和类型的GitLab

* 前提条件

  确保装有`rsync`命令

* 备份命名

  ```
  <Timestamp>_<datetime>-<gitVersion>_gitlab_backup.tar
  ```

  如`1493107454_2018_04_25_10.6.4-ce_gitlab_backup.tar`

---------

* 备份

  1. 修改配置文件

     ```shell
     vim /etc/Gitlab/Gitlab.rb
     ```

  2. 修改配置

     ```shell
     # Gitlab备份路径
     Gitlab_rails['backup_path'] = "/data/Gitlab-data/backups/"
     # 备份存留时间
     Gitlab_rails['backup_keep_time'] = 604800
     ```

  3. 使配置生效

     ```shell
     Gitlab-ctl reconfigure
     ```

  4. 添加定时任务, 定时备份

     ```shell
     crontab -e
     ```

     ```
     0 2 * * * /opt/gitlab/bin/gitlab-backup create CRON=1
     ```

     > GitLab 12.1及之前, 应该用`gitlab-rake gitlab:backup:create`命令

* 恢复

  > 详细步骤: [Restore for Omnibus GitLab installations](https://docs.gitlab.com/ee/raketasks/backup_restore.html#restore-for-omnibus-gitlab-installations)
  
  ```shell
  # This command will overwrite the contents of your GitLab database!
sudo gitlab-backup restore BACKUP=11493107454_2018_04_25_10.6.4-ce
  ```
  
  > GitLab 12.1及之前, 应该用`gitlab-rake gitlab:backup:restore`命令

> 参考[Back up and restore GitLab](https://docs.gitlab.com/ee/raketasks/backup_restore.html)

# 命令

* 查看状态

  ```shell
  gitlab-ctl status
  ```

* 启动

  ```shell
  gitlab-ctl start
  ```

* 停止

  ```shell
  gitlab-ctl stop
  ```

  

# Gitlab CI/CD

## runner安装

> 参考[gitlab ci/cd 简单流程介绍](https://www.jianshu.com/p/43a2b50a5b3d)

* 选择linux安装方式

  [Install GitLab Runner](https://docs.gitlab.com/runner/install/)

* 将Runner注册到Gitlab中

  ```
  gitlab-runner register
  ```

  > 配置url和token即可(管理员可在管理员区域查看), tag不要配置了. 

* 命令`gitlab-runner`

  * 启动

    ```
    gitlab-runner start
    ```

  * 停止

    ```
    gitlab-runner stop
    ```

  * 重启

    ```
    gitlab-runner restart
    ```


## .gitlab-ci.yml

> 参考
>
> * [gitlab-ci.yml 配置Gitlab pipeline以达到持续集成的方法](https://www.jianshu.com/p/b69304279c5f)
> * [The .gitlab-ci.yml file](https://docs.gitlab.com/ee/ci/yaml/gitlab_ci_yaml.html)

* `stage` 当前阶段. 可选: build|test|deploy
* `script` 脚本
* `only` 作用分支

Demo

```yml
 stages:
  - clear_dir
  - git_clone
  - build
  - deploy  
clear_dir:
    stage: clear_dir
    script:
      - cd /home/gitlab-runner/
      - rm -rf ./h5
    only:
      - master
    tags:
      - shell
git_clone:
    stage: git_clone
    script:
      - cd /home/gitlab-runner/
      - git clone git@192.168.1.97:xiaojian/h5.git
    only:
      - master
    tags:
      - shell
build:
    stage: build
    script:
      - cd /home/gitlab-runner/h5
      - cnpm install
    only:
      - master
    tags:
      - shell
deploy:
    stage: deploy
    script:
      - echo "发布中...."
    only:
      - master
    tags:
      - shell
```







