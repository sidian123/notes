# 邮件通知

邮件通知, 需满足几个条件:

1. 用户个人信息需含有**邮件地址**和**VCS Usernames**

   ![image-20200817222717307](.Teamcity/image-20200817222717307.png)

   VCS Username是提交记录时用的名字, 通过如下方式可查到:

   ```shell
   $ git config -l
   ...
   user.email=sidian123@qq.com
   ```

   这里的`sidian123`就是VSC名字.

2. 配置邮件服务器

   ![image-20200817223016310](.Teamcity/image-20200817223016310.png)

3. 配置通知规则

   ![image-20200817223149837](.Teamcity/image-20200817223149837.png)

   以管理员身份, 配置的通知规则作用域一个组的所有用户. 默认规则是, 当**代码提交**后, 项目部署失败, 会邮件通知导致部署失败的用户. 但手动Run, 是不会有邮件通知的.

-----------

**原理:** Teamcity能拿到提交记录的邮箱`user.email`, 取`@`前的名字, 与VCS Names比对, 找到对用用户. 再向用户填写的邮箱上发送通知. 

> 具体功能由VCS root提供, 其配置Username style制定匹配规则

> 参考
>
> * [What is the correct VCS Username settings with Github so that “My Changes” works correctly?](https://stackoverflow.com/questions/9295649/what-is-the-correct-vcs-username-settings-with-github-so-that-my-changes-works)
> * [Subscribing to Notifications](https://www.jetbrains.com/help/teamcity/2019.2/subscribing-to-notifications.html)
> * [Configuring VCS Roots](https://www.jetbrains.com/help/teamcity/2019.2/configuring-vcs-roots.html)







