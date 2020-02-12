# 入门

* 介绍

  JGit, 一个由Java实现的Git客户端. 提供了命令行和API接口供开发者使用, 使用习惯与我们常用的Git命令一致.

* 依赖

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
    
* 貌似一些配置信息, JGit会从环境中获取, 具体不晓滴

# 使用

## 基本使用

[Repository](https://download.eclipse.org/jgit/site/5.5.0.201909110433-r/apidocs/org/eclipse/jgit/lib/Repository.html)代表一个Git仓库, 存储着仓库所有的objects和refs.

[Git](https://download.eclipse.org/jgit/site/5.5.0.201909110433-r/apidocs/org/eclipse/jgit/api/Git.html)是操纵Git仓库的API接口, 它的每个方法都返回一个代表命令的对象, 最后调用该对象的`call()`方法执行这个命令.

> 如`git.commit()`返回一个提交对象, 用于提交修改.
>
> 注意, 使用了`Git`, 要关闭资源. 尽量不要手动传入`Repository`给`Git`, 这样关闭`Git`时, 它会自动关闭`Repository`

然后... 没了, 了解着两个类和看API文档就够了.

## 认证

### HTTP(S)

```java
CloneCommand cloneCommand = Git.cloneRepository();
cloneCommand.setURI( "https://example.com/repo.git" );
cloneCommand.setCredentialsProvider( new UsernamePasswordCredentialsProvider( "user", "password" ) );
```

### SSH

```java
CloneCommand cloneCommand = Git.cloneRepository();
cloneCommand.setTransportConfigCallback(transport -> {
    SshTransport sshTransport = (SshTransport) transport;
    sshTransport.setSshSessionFactory(new JschConfigSessionFactory() {
        @Override
        protected void configure(OpenSshConfig.Host host, Session session) {
            //允许任何host连接
            session.setConfig("StrictHostKeyChecking", "no");
        }
    });
});
cloneCommand.setURI("git@github.com:sidian123/notes-private.git")
    .setDirectory(new File("C:\\Users\\sidian\\Desktop\\test3"))
    .setProgressMonitor(new TextProgressMonitor())
    .call()
    .close();
}
```

默认配置文件来源于`$HOME/.ssh/`目录, 可修改, 覆盖`JschConfigSessionFactory.createDefaultJSch()`方法, 如:

```java
@Override
protected JSch createDefaultJSch(FS fs) throws JSchException {
    JSch jSch = super.createDefaultJSch(fs);
    jSch.addIdentity("C:/Users/xxx/.ssh/id_rsa");//私钥
    jSch.setKnownHosts("C:/Users/xxx/.ssh/known_hosts");//known hosts
    return jSch;
}
```

若密钥有密码

```java
addIdentity( "/path/to/private_key", "passphrase" );
```

----

若报错

```
invalid privatekey: [B@59c40796.
```

是因为JSch ( JGit的依赖 ) 不支持密钥的格式, 换种格式重新生成即可, 如

```shell
ssh-keygen -m PEM
```

> 参考
>
> * [JGit Authentication Explained](https://www.codeaffine.com/2014/12/09/jgit-authentication/) JGit不同认证方式的使用
> * [“Invalid privatekey” when using JSch](https://stackoverflow.com/a/55740276/12574399)

# 参考

* [JGit documentation](https://www.eclipse.org/jgit/documentation/)
* [JGit API](http://download.eclipse.org/jgit/site/5.5.0.201909110433-r/apidocs/index.html)
* [JGit tutorial](https://www.vogella.com/tutorials/JGit/article.html#cloning-a-git-repository-with-jgit)
* [JGit Maven](https://www.eclipse.org/jgit/download/)
* [jgit-cookbook](https://github.com/centic9/jgit-cookbook) 具有完整的JGit使用例子, 极力推荐
* [JGit Authentication Explained](https://www.codeaffine.com/2014/12/09/jgit-authentication/) JGit不同认证方式的使用