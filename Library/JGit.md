# 基础

JGit, 一个由Java实现的Git客户端. 提供了命令行和API接口供开发者使用, 使用习惯与我们常用的Git命令一致.

这里只介绍API使用, 先导入Maven依赖:

```xml
  <!-- Core Library -->
  <dependencies>
    <dependency>
      <groupId>org.eclipse.jgit</groupId>
      <artifactId>org.eclipse.jgit</artifactId>
      <version>5.5.0.201909110433-r</version>
    </dependency>
  </dependencies>
```

-------

[Repository](https://download.eclipse.org/jgit/site/5.5.0.201909110433-r/apidocs/org/eclipse/jgit/lib/Repository.html)代表一个Git仓库, 存储着仓库所有的objects和refs.

[Git](https://download.eclipse.org/jgit/site/5.5.0.201909110433-r/apidocs/org/eclipse/jgit/api/Git.html)是操纵Git仓库的API接口, 它的每个方法都返回一个代表命令的对象, 最后调用该对象的`call()`方法执行这个命令.

> 如`git.commit()`返回一个提交对象, 用于提交修改.
>
> 注意, 使用了`Git`, 要关闭资源. 尽量不要手动传入`Repository`给`Git`, 这样关闭`Git`时, 它会自动关闭`Repository`

然后... 没了, 了解着两个类和看API文档就够了.

# 参考

* [JGit documentation](https://www.eclipse.org/jgit/documentation/)

* [JGit API](http://download.eclipse.org/jgit/site/5.5.0.201909110433-r/apidocs/index.html)
* [JGit tutorial](https://www.vogella.com/tutorials/JGit/article.html#cloning-a-git-repository-with-jgit)

* [JGit Maven](https://www.eclipse.org/jgit/download/)