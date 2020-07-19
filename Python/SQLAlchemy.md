* 安装

  安装`SQLAlchemy`

  ```shell
  pip install SQLAlchemy
  ```

  安装MySQL驱动包

  ```shell
  # 未使用Flask时
  pip install mysqlclient
  # 使用了Flask
  pip install Flask-SQLAlchemy
  ```

* Overview

  ![_images/sqla_arch_small.png](.SQLAlchemy/sqla_arch_small.png)

  * ORM: 提供高级的对象-实体映射功能
  * SQL Expression Language: 低级的, 更灵活操作数据的功能
  * DBAPI: Python访问数据的API规范, 类似Java的JDBC.

# ORM

# SQL Expression Language

# 参考

* [学习资料概述](https://www.sqlalchemy.org/library.html#tutorials)
* [官方文档](https://docs.sqlalchemy.org/en/13/)

* [SQLALchemy之Python连接MySQL](https://blog.csdn.net/weixin_44080811/article/details/90030744)