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

  