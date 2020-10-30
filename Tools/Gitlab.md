# 备份与恢复

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