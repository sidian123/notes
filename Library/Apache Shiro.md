* 核心功能

  * Authentication - proving user identity, often called user ‘login’.
  * Authorization - access control
  * Cryptography - protecting or hiding data from prying eyes
  * Session Management - per-user time-sensitive state

* 核心概念

  * Subject

    指与系统正在交互的当前"用户", 可以是一个人, 三方客户端等等.

    获取Subject

    ```java
    Subject currentUser = SecurityUtils.getSubject();
    ```

  * SecurityManager

    所有Subject的管理器

  * Realms

    用于提供用户的安全数据, 如账号密码, 来比对认证

https://zhuanlan.zhihu.com/p/75848572