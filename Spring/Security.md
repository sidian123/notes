# 开始

## 介绍

- 介绍：主要通过Servlet过滤器实现
- 主要功能：认证（authentication）和授权（authorization）
  - 认证：识别用户。支持很很多种认证模型，如微信用户的OpenID认证。
  - 授权：对于web请求、方法、域，拥有何种权限
  - 攻击防护
- Maven依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>
```

## 默认配置

- 配置思路：`@EnableWebSecurity`注解在`WebSecurityConfigurerAdapter`上，它提供默认配置，然后覆盖它的方法来自定义配置security。

- 默认配置：`WebSecurityConfigurerAdapter`在`configure(HttpSecurity http)`方法中提供了默认配置，如下所示：

  ```java
  @EnableWebSecurity//启动security配置
  public class SecurityConfig extends WebSecurityConfigurerAdapter {//adapter提供默认配置，之后覆盖它的方法来自定义配置
      @Override
      protected void configure(HttpSecurity http) throws Exception {
          //这里的配置和adapter默认的一致
          http
              .authorizeRequests()//配置URL授权
                  .anyRequest().authenticated()//所有URL只能被已认证用户访问
                  .and()
              .formLogin()//表单认证，无论声明顺序，都比http basic优先级高
                  .and()
              .httpBasic();//也可以使用HTTP basic认证
      }
  }
  ```

  - 确保所有请求都要认证，否则会跳转到验证页面。
  - 通过form表单验证，使用HTTP Basic验证方法
  - 默认账号`user`，密码输出到`console`

  等价于xml配置：

  ```xml
  <http>
      <intercept-url pattern="/**" access="authenticated"/>
      <form-login />
      <http-basic />
  </http>
  ```

  > `and()`方法等同于结束标签，返回parent

  > 不懂为啥`login`,`logout`就不用授权

## 自定义配置(简单介绍)

> 详细见HttpSecurity

- [登录页面](<https://docs.spring.io/spring-security/site/docs/5.2.0.BUILD-SNAPSHOT/reference/htmlsingle/#jc-form>)（`.formLogin()`）：由`spring security`默认生成，默认url为`login`，也可自行指定，如

  ```java
  protected void configure(HttpSecurity http) throws Exception {
      http
          .authorizeRequests()
              .anyRequest().authenticated()
              .and()
          .formLogin()
              .loginPage("/login") //手动设置登录页面
              .permitAll();        //该页面允许所有人访问
  }
  ```

- [授权请求](<https://docs.spring.io/spring-security/site/docs/5.2.0.BUILD-SNAPSHOT/reference/htmlsingle/#jc-authorize-requests>)（`.authorizeRequests()`）

  ```java
  protected void configure(HttpSecurity http) throws Exception {
      http
          .authorizeRequests()
              .antMatchers("/resources/**", "/signup", "/about").permitAll()
              .antMatchers("/admin/**").hasRole("ADMIN")
              .antMatchers("/db/**").access("hasRole('ADMIN') and hasRole('DBA')")
              .anyRequest().authenticated()
              .and()
          // ...
          .formLogin();
  }
  ```

- [退出登录](<https://docs.spring.io/spring-security/site/docs/5.2.0.BUILD-SNAPSHOT/reference/htmlsingle/#jc-logout>)（`.logout()`）：默认提供，URL为`logout`，登出后重定向到，可自行配置：

  ```java
  protected void configure(HttpSecurity http) throws Exception {
      http
          .logout()
              .logoutUrl("/my/logout")
              .logoutSuccessUrl("/my/index")
              .logoutSuccessHandler(logoutSuccessHandler)
              .invalidateHttpSession(true)
              .addLogoutHandler(logoutHandler)
              .deleteCookies(cookieNamesToClear)
              .and()
          ...
  }
  ```

# HttpSecurity

用于配置**URL授权**、**登录**、**登出**，一个例子如下：

```java
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
            .csrf().disable()
            .authorizeRequests()//开始配置URL授权，最先配置的优先级最高
                .antMatchers("/admin/**").hasRole("ADMIN")//设置访问用户的角色。注意，会自动添加前缀 ROLE_
                .antMatchers("/anonymous*").anonymous()//不懂
                .antMatchers("/login*").permitAll()//登录界面允许任何人访问
                .anyRequest().authenticated()//所有请求都需要已认证（注意，这里优先级最低）
                .and()
            .formLogin()//开始配置登录
                .loginPage("/login.html")//自定义登录界面
                .loginProcessingUrl("/perform_login")//处理账户密码的url
                .defaultSuccessUrl("/homepage.html", true)//登录成功后跳转的页面
                //.failureUrl("/login.html?error=true")//登录失败后跳转的页面
                .failureHandler(authenticationFailureHandler())//登录失败的处理器，比failureUrl要底层点
                .and()
            .logout()//登出配置，略
                .logoutUrl("/perform_logout")
                .deleteCookies("JSESSIONID")
                .logoutSuccessHandler(logoutSuccessHandler());

    }
```

分别对应`authorizeRequests()`、`formLogin()`、`logout()`函数，以`and()`分隔。

# AuthenticationManager

用于配置具体用户认证方式的，下面的例子中配置了使用内存数据进行验证的方式，所有的认证方式有：

- in memory authentication
- LDAP authentication
- JDBC based authentication
- adding `UserDetailsService`
- adding`AuthenticationProvider`

```java
@Override
protected void configure(AuthenticationManagerBuilder auth) throws Exception {
    auth.inMemoryAuthentication().withUser("user").password("{noop}password").roles("USER");
}
```

> 注意，在Spring5中，必须定义加密器。

常用`UserDetailsService`提供数据库中的用户数据, `AuthenticationProvider`可定制范围更大. 

# 过滤器

# 架构与实现

## 核心组件

* `SecurityContextHolder`: 存储`SecurityContext`的地方, 是**线程局部**的, 即同样声明的`SecurityContext`字段, 在每个线程中使用的都是不同的, 线程结束后自动被清除.

  > 变量的线程局部性是通过`ThreadLocal`实现的, 这种模式还可以更改.
  >
  > 不要将**线程作用域**与**会话作用域**混淆

* `SecurityContext`: 含有与当前线程相关的安全信息. 实际上就是存放`Authentication`的地方.

  > 通过`SecurityContextHolder`的静态方法, 可以获取属于该线程的`SecurityContext`.

* `Authentication`: 已授权的凭证. 

  [`AuthenticationManager.authenticate(Authentication)`](https://docs.spring.io/spring-security/site/docs/current/api/org/springframework/security/authentication/AuthenticationManager.html#authenticate-org.springframework.security.core.Authentication-)负责认证传入的对象, 如果验证成功, 则返回被**填充完整**的`Authentication`对象, 接着存入`SecurityContext`中, 表示认证成功.

  初次之外, 一种显式验证的方式, 就是手动放置`Authentication`, 如:

  ```java
  SecurityContextHolder.getContext().setAuthentication(anAuthentication);
  ```

  > `isAuthenticated()`返回`true`时, 则不会被拦截器拦截, 提高效率. 并且`true`时就是表示已填充完整

* `UserDetails`: 代表用户, 含用户,角色等信息.

  用户信息最终会存入`Authentication`中, 然后去认证. 这个信息可以是任何`Object`对象, 只要认证器能够识别它. 但通常会用`UserDetails`表示, 同时它也是自己数据库用户类与Spring Security所需要类型的一个桥梁.

  Spring Security提供了一个实现类`User`来简化该接口的使用.

  >那通过什么途径将用户信息提供给Spring Security呢?
  >
  >实现`UserDetailsService`并注入到容器中即可. 实现它的`loadUserByUsername`函数, 通过用户名来获取`UserDetails`.
  >
  >> 该接口已经存在了很多实现类, 如常用的`InMemoryDaoImpl`等.
  >>
  >> 但还是自己实现一个比较好.

* `GrantedAuthority`: 表示角色, 如`ROLE_USER`, `ROLE_ADMIN`等, 每个角色都对应一定的权限, 在构建`UserDetails`对象时须提供.

  > 该权限是系统范围的, 即某一范围的人属于某种角色, 并拥有该角色的权限.

----------

* 小结

  1. 在`UserDetailsService`中实现获取用户数据的逻辑, 并与角色`GrantedAuthority`组成`UserDetails`, 再返回该`UserDetails`对象.
  
  2. `UserDetails`存入`Authentication`中.
  
  3. `AuthenticationManager.authenticate(Authentication)`中验证`Authentication`是否通过验证

  4. 验证成功则返回填充过的`Authentication`, 并存入`SecurityContextHolder`中

  5. 认证成功
  
     > 用户认证成功, 当且仅当`SecurityContextHolder`中存在完全填充过的`Authentication`对象.
  
     > 疑问
     >
     > `SecurityContextHolder`中`Authentication`是线程局部的, 下次请求怎么办?
  
  之后的请求中, 可以方便的从`SecurityContextHolder`中获取用户信息, 如:

  ```java
  Object principal = SecurityContextHolder.getContext().getAuthentication().getPrincipal();
  
  if (principal instanceof UserDetails) {
  String username = ((UserDetails)principal).getUsername();
  } else {
  String username = principal.toString();
  }
  ```
  
  









# 其他

- Rememver-me：`HttpSecurity`有对应函数设置，本质是使用了cookie实现的。参考[Spring Security Remember Me](<https://www.baeldung.com/spring-security-remember-me>)
- 官方样例：[samples](<https://github.com/spring-projects/spring-security/tree/master/samples>)
- 在rest web服务中，授权用户应该访问正确的url，否则返回401（`UNAUTHORIZED`）状态码

# 问题

- 无加密器：[Spring Security – There is no PasswordEncoder mapped for the id “null”](<https://www.mkyong.com/spring-boot/spring-security-there-is-no-passwordencoder-mapped-for-the-id-null/>)

# 推荐阅读

* [OAuth 2 Developers Guide](https://projects.spring.io/spring-security-oauth/docs/oauth2.html): 关于OAuth2授权服务器的搭建

* [security 教程](https://www.baeldung.com/security-spring)