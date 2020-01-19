[TOC]

# 一 介绍

Spring Security是Spring提供的一个用于**认证**和**授权**的一个安全框架. 

Security基于Servelt的**过滤器**和Spring的**AOP**实现, 也就是说, 除了可以为不同的URL设置访问权限外, 还可以为方法设置访问权限.

认证的方式很多, 如Spring提供了HTTP Basic, form表单等认证方式. 然而这并不能满足多样的认证方式, 开发者可以提供自己的认证方式的实现, 但这要求开发者对Spring Security**过滤器**和**核心组件**有比较深刻的认识.

# 二 原理

为了与JWT整合, 需要了解其核心组件与过滤器

## 核心组件

- `SecurityContextHolder`: 存储一个`SecurityContext`的地方, 默认是**线程局部**的

  > 即同样的一个`SecurityContext`字段, 在不同线程中获取, 得到的对象都是不同的, 线程结束后将自动清除该对象. 通过`ThreadLocal`实现的.

- `SecurityContext`: 与**当前线程**相关的上下文环境, 实际上就是存放`Authentication`的地方.

  > 通过`SecurityContextHolder`的静态方法, 可以获取属于该线程的`SecurityContext`.

- `Authentication`: 授权的凭证. 

  [`AuthenticationManager.authenticate(Authentication)`](https://docs.spring.io/spring-security/site/docs/current/api/org/springframework/security/authentication/AuthenticationManager.html#authenticate-org.springframework.security.core.Authentication-)负责认证传入的凭证, 如果验证成功, 则返回被**填充完整**的`Authentication`对象, 接着存入`SecurityContext`中, 表示认证成功.

  除此之外, 一种显式验证的方式, 就是手动放置`Authentication`, 如:

  ```java
  SecurityContextHolder.getContext().setAuthentication(anAuthentication);
  ```

  > `Authentication.isAuthenticated()`返回`true`时, 则不会被其他认证器拦截, 提高效率. 并且`true`时就是表示已填充完整

  > `AuthenticationManager`实际上是一个委派器, 将认证处理委派给其他认证器认证.

- `UserDetails`: 代表用户, 含用户,角色等信息.

  用户信息最终会存入`Authentication`中, 然后去认证. 这个信息可以是任何`Object`对象, 只要认证器能够识别它. 但通常会用`UserDetails`表示, 同时它也是自己数据库用户类与Spring Security所需要类型的一个桥梁.

  Spring Security提供了一个实现类`User`来简化该接口的使用.

  > 那通过什么途径将用户信息提供给Spring Security呢?
  >
  > 实现`UserDetailsService`并注入到容器中即可. 实现它的`loadUserByUsername`函数, 通过用户名来获取`UserDetails`.
  >
  > > 该接口已经存在了很多实现类, 如常用的`InMemoryDaoImpl`等.
  > >
  > > 但还是自己实现一个比较好.

- `GrantedAuthority`: 表示角色, 如`ROLE_USER`, `ROLE_ADMIN`等, 每个角色都对应一定的权限, 在构建`UserDetails`对象时须提供.

  > 该权限是系统范围的, 即某一范围的人属于某种角色, 并拥有该角色的权限.

## 过滤器

过滤器既用于认证, 也用于授权.

> 注意, AOP也参与到了方法级的授权, 但这里暂且不谈.

Spring Security提供了很多过滤器来完成认证和授权的工作. **但并不是每个过滤器都参与了请求的处理**, 如已认证过的请求不会被处理认证的过滤器处理. 但是它们的执行顺序很关键, 不管其是否真的使用, 排序如下:

- `ChannelProcessingFilter`, because it might need to redirect to a different protocol

- `SecurityContextPersistenceFilter`, so a `SecurityContext` can be set up in the `SecurityContextHolder` at the beginning of a web request, and any changes to the `SecurityContext` can be copied to the `HttpSession` when the web request ends (ready for use with the next web request).

  > 默认行为中, 仍是Cookie标识用户, 用户完整认证信息存于Session中, 因此认证后多次请求, 仍处于认证状态.
  >
  > 注意, `SecurityContextHolder`中含有完整填充过的`Authentication`才是认证标志. 因此, Spring Security可被配置无Session化, 仍能正常功能, 见[第四章-集成JWT]

- `ConcurrentSessionFilter`, because it uses the `SecurityContextHolder` functionality and needs to update the `SessionRegistry` to reflect ongoing requests from the principal

- Authentication processing mechanisms - `UsernamePasswordAuthenticationFilter`, `CasAuthenticationFilter`, `BasicAuthenticationFilter` etc - so that the `SecurityContextHolder` can be modified to contain a valid `Authentication` request token

  >实现认证处理机制的过滤器, 有很多种实现, 具体使用那个过滤器取决于认证方式, 该认证方式通过`HttpSecurity`配置的. 
  >
  >如果想提供自己的认证实现, 可尝试实现`OncePerRequestFilter`接口, 然后通过`HttpSecurity`注册到过滤器链中的某个位置上, 一般为认证处理器的位置旁.
  >
  >> 使用自己的认证过滤器实现时, `httpSecurity`中不要设置其他认证机制, 防止与自己的过滤器冲突.
  >
  >> `OncePerRequestFilter`好像保证了, 每次只被一个请求线程执行.

- The `SecurityContextHolderAwareRequestFilter`, if you are using it to install a Spring Security aware `HttpServletRequestWrapper` into your servlet container

- The `JaasApiIntegrationFilter`, if a `JaasAuthenticationToken` is in the `SecurityContextHolder` this will process the `FilterChain` as the `Subject` in the `JaasAuthenticationToken`

- `RememberMeAuthenticationFilter`, so that if no earlier authentication processing mechanism updated the `SecurityContextHolder`, and the request presents a cookie that enables remember-me services to take place, a suitable remembered `Authentication` object will be put there

  > **记住我**, 这个功能是通过该处理器实现的. 将凭证存在Cookie中( 一般Cookie无凭证信息 ), 如果前面的认证过程没有通过, 但Cookie中有凭证, 于是就提取数据并认证.

- `AnonymousAuthenticationFilter`, so that if no earlier authentication processing mechanism updated the `SecurityContextHolder`, an anonymous `Authentication` object will be put there

  > 没有认证过的用户, 就是匿名状态

- `ExceptionTranslationFilter`, to catch any Spring Security exceptions so that either an HTTP error response can be returned or an appropriate `AuthenticationEntryPoint` can be launched

  > 下面的过滤器实现授权的功能, 无权限时会抛出异常, 该处理器就是将异常转换为对应的HTTP响应状态码, 或者直接重定向到登录URL上.
  >
  > 我发现这个过滤器很不错, 在Form表单的认证方式下, 通过浏览器发起的未认证请求会被重定向; 通过ajax发起的未认证请求会返回403状态码. 稍作修改就能满足Rest API的需求.

- `FilterSecurityInterceptor`, to protect web URIs and raise exceptions when access is denied

  >查看用户是否有权限访问该URL

可以看到, Spring Security使用的过滤器并不是很多, 真正起着关键作用的也就那几个. 

在了解了每个过滤器后, 我们可以实现自己的过滤器, 这要考虑的有很多, 在后面章节中会介绍到.

## 小结

### 最终认证成功的标识是什么?

`SecurityContextHolder`中存在已填充完整的`Authentication`

填充完整, 意味着`Authentication.isAuthenticated()`返回true.

### 过滤器与核心组件如何工作的?

Spring Security主要涉及认证和授权.

认证方面, 由和认证相关的过滤器验证用户输入的凭证, 并生成填充完整的`Authentication`对象, 并放置.

> 具体过程, 用户输入凭证, 将之组装一个`Authentication`对象, `AuthenticationManager.authenticate()`验证该对象, 并返回填充完整的`Authentication`对象, 然后放置到`SecurityContextHolder`中

授权方面, 分两步走, 可以是过滤请求, 也可以过滤方法.

1. 过滤请求: `FilterSecurityInterceptor`过滤器判断`Authentication`中代表的角色`GrantedAuthority`是否满足该URL的要求
2. 过滤方法: AOP拦截方法, 通过判断`Authentication`中代表的角色`GrantedAuthority`是否满足访问该方法的要求.

### 认证,授权流程?

认证流程:

1. 用户登录

2. 将用户信息保证到`Authentication`中

3. `AuthenticationManager.authenticate(Authentication)`验证该凭证是否正确. 验证的参考对象来源于数据库, 由`UserDetailsService`提供.

   > 它本身是一个委派器, 交给其他认证器验证.
   >
   > `UserDetailsService`将返回`UserDetails`, 代表用户, 包含权限信息`GrantedAuthority`

4. 验证成功则返回填充过的`Authentication`, 并存入`SecurityContextHolder`中

5. 认证成功

---------

授权流程

1. 登录成功后, 再次发出请求

2. `SecurityContextPersistenceFilter`校验cookie, 正确后放置已填充完整的`Authentication`到`SecurityContextHolder`中, 表示用户处于已认证状态.

3. `FilterSecurityInterceptor`看到`SecurityContextHolder`中存在已填充的`Authentication`对象, 并符合权限要求, 则运行访问URL

   > 如果是方法上过滤的, 则拦截该方法的拦截器, 同样判断`SecurityContextHolder`中是否存在已填充的`Authentication`对象, 若符合权限要求, 则运行执行该方法. 

### 如何保持多请求下的认证状态?

`SecurityContextPersistenceFilter`过滤器通过cookie, 将存在session的`Authentication`放置好. 之后的过滤器则会认为请求已经认证过了.

### 认证实现需自己提供时为何还用Spring Security?

1. 避免重复造轮子

   认证只是Spring Security的一部分, 还有很多细节要自己考虑, 重复造轮子的代价要大的多, 并且Spring Security功能还是很全的

2. Spring Security只是基础

   Spring很多功能(或框架)是基于Spring Security的, 使用Spring Security可以轻松的与其他框架整合.

3. 折腾可以学到很多

   认证方式多样, Spring Security可能不符合我们的需求, 但是我们可以提供自己的实现. 这需要深入的了解Spring Security内部的运行原理和机制. 这将会耗费很多时间, 但是也会让我们更深入的了解Web应用如何保证自身安全的.

### 为何Servelt的过滤器可以通过Spring容器注入?

Spring Security通过`DelegatingFilterProxy`类作为Servelt Filter与容器中Bean的桥梁.

### 与传统Cookie认证相比, Spring Security认证有何异同?

* 用户信息上
  * Cookie认证中, Cookie本身并没有记录和权限相关的信息, 仅用作认证, 后端可通过Cookie即可判断用户身份
  * Spring Security认证中, Cookie同样不保存太多信息, 仅判断用户身份, 而真正作为认证标识的是小节2.3.1. 默认Session中会保存所有用户认证,授权信息, 但是这不是必须的, Spring Security也能够做到后端去Session化, 认证,授权信息保存在前端.

* 功能上
  * Spring Security除了认证功能外, 还有授权的功能.

# 三 使用

这里介绍Spring Security的使用和配置方法. 

主要就是配置两个类:

* `HttpSecurity`: 主要用于配置访问URL所需要的角色, 何种认证方式, 添加自定义的过滤器等.

  > 没有配置认证方式时, 那些认证过滤器则不存在.

* `AuthenticationManager`: 配置认证过程中用到的组件, 如使用什么加密器? 何种方式提供用户信息到Spring Security, 用以验证用户数据等等.

还会介绍方法级授权的使用.

## Quick Start

先引入依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>
```

使用思路

继承`WebSecurityConfigurerAdapter`类, 并加上注解`@EnableWebSecurity`开启Security功能. 

>`WebSecurityConfigurerAdapter`提供了默认配置, 我们只需要覆盖它的某个方法即可, 来提供自己想要的行为.

一个例子如下:

> 注意, 下面的配置和默认行为是一样的

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

----------

> 可以看到, 默认行为分别开启了表单, HTTP Basic认证方式. 实际稍大一点的项目中, 这些认证方式是不满足使用的, 需要提供自己的认证实现.

## 配置与基本认证,授权

> 不好分类, `HttpSecurity`即可配置认证, 又可配置授权. 还有一种在方法上授权的注解, 见[3.3方法级授权]

### HttpSecurity

主要用于配置**URL授权**、**登录**、**登出**，下面详细讲解... :

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

---------

这里给出一个综合性的例子:

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

### AuthenticationManager

`AuthenticationManager`负责具体的认证过程, 但其最终将认证委派给其他过滤器处理. 

可以配置的内容有

* **数据提供**: 要配置如何提供数据给Spring Security, 用以验证用户输入的凭证的正确性. 数据提供的方式有:

  - in memory authentication: 测试时最常用的方式.
  - LDAP authentication
  - JDBC based authentication: 不好用
  - adding `UserDetailsService`: 最常用的方式
  - adding`AuthenticationProvider`: 最具扩展性的方式

* **加密器**(PasswordEncoder): 数据库中存的一般是加密过的密码, 通过上述方式获取到用户密码后, 再经加密器加密, 才能比较是否正确. 

  > 注意, Spring5后, 要求必须存在一个加密器, 官方推荐`BCryptPasswordEncoder`. 当然测试时可用无加密的加密器`NoOpPasswordEncoder`

* **其他配置**: 略.

例子一

```java
    @Bean
    public UserDetailsService userDetailsService(){
        return new UserDetailsService() {
            @Override
            public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
                if(username.equals("luo"))
                    return new AuthUser("1","luo",passwordEncoder.encode("123456"), Arrays.asList(new SimpleGrantedAuthority("ROLE_USER")));
                else
                    return new AuthUser("2","lou",passwordEncoder.encode("123456"),Arrays.asList(new SimpleGrantedAuthority("ROLE_admin")));
            }
        };
    }

  @Bean
    public BCryptPasswordEncoder bCryptPasswordEncoder(){
        return new BCryptPasswordEncoder();
    }
```

例子二, 覆盖`WebSecurityConfigurerAdapter`的方法来配置`AuthenticationManager`, 如:

```java
@Override
protected void configure(AuthenticationManagerBuilder auth) throws Exception {
    auth.inMemoryAuthentication().withUser("user").password("{noop}password").roles("USER");
}
```

## 方法级授权

通过Spring AOP实现的一种方法级授权方法, 只需在方法上标注`@Secured`, 并指定访问权限即可.

使用步骤:

1. 开启方法注解, 在配置类` @Configuration `上标注` @EnableGlobalMethodSecurity `

2. 使用`@Secured`注解, 指定访问权限

   ```java
   public interface BankService {
   
   @Secured("IS_AUTHENTICATED_ANONYMOUSLY")
   public Account readAccount(Long id);
   
   @Secured("IS_AUTHENTICATED_ANONYMOUSLY")
   public Account[] findAccounts();
   
   @Secured("ROLE_TELLER")
   public Account post(Account account, double amount);
   }
   ```

> **注意**
>
> 通过查看源码`org.springframework.security.access.vote.vote()`, 发现, 注解的角色名必须以`ROLE_`开头, 导致`UserDetailsService.loadUserByUsername()`中返回的`UserDetails`的角色必须也已`ROLE_`为前缀, 注解才能生效, 坑!!!!

## 其他

### 获取当前用户凭证

```java
SecurityContextHolder.getContext().getAuthentication().getPrincipal()
```

# 四 集成JWT

> 集成JWT, 本质是更换了认证方式, 将Token从后端移入前端.

## 介绍

在并发数提高时, 我们通常采用微服务化和集群的方式, 保证服务对流量的承载性. 应用的对Session的存放方式决定了它的集群方式:

* Session在后端: 这是最常见的方式, 应用集群后需要解决Session共享的问题, 通常采用Redis来解决.

* Session在前端: 后端不保存用户信息, 而是登录后将用户数据存在token中并返回给用户, 由前端保存token. 后端不存在Session共享问题, 只需运行多个应用实例即可实现集群.

  > 但是否存在Sesion对于使用token来说, 不是必须的.

这里介绍的是第二种方式, 涉及到如何保证前端发给后端的token是可靠的, 正确的, 没有被篡改的问题. [JWT](https://jwt.io/introduction/)提供了该方案的可行性, 具体原理见第五章.

## 实现细节

要想Spring Security与JWT集成, 并实现后端无Session化, 需要考虑的问题有很多. 最主要的思路是, **提供自己的认证过滤器和登录控制器**. 

* 登录控制器在登录成功后返回一个token给前端, 前端之后的每次请求都携带token; 
* 认证过滤器拦截所有请求, 一旦发现有token并通过了验证, 则放置好对应的`Authentication`.

> 注意, 构建`Authentication`的数据都是来源与解析token得到的.

那集成JWT需要考虑的问题呢?

* 保证Spring Security的认证过滤器不会影响自定义的认证过滤器

  只需在配置`HttpSession`时, 不设置Spring Security提供的认证方式即可. 如无`.formLogin()`, `.HttpBasic()`的存在.

* 注意自定义认证过滤器的位置

  放置在和授权相关过滤器之前即可, 如将过滤器放到了其他认证过滤器之前:

  ```java
  http.addFilterBefore(jwtAuthenticationFilter(), UsernamePasswordAuthenticationFilter.class);
  ```

* 阻止Spring Security默认行为中`SecurityContextPersistenceFilter`使用cookie和Session保证多请求自动认证的行为.

  ```java
  http
      .sessionManagement()    
      .sessionCreationPolicy(SessionCreationPolicy.STATELESS)
  ```

具体实现见[Spring_Security_Demo](https://github.com/sidian123/Spring_Security_Demo)

> 在Git的提交历史中, 第一个提交是仅含Spring Security时的配置, 后面的提交才加入了JWT的集成.

# 其他

## JWT

见[Library/JWT](../Library/JWT.md)

## 角色,权限

Spring Security本身没有提供角色和权限的功能, 需要自己实现.

实现的关键点: 授权时, 是根据`Authentication`中的权限信息`GrantedAuthority`判断的. 若有访问该方法, URL的权限, 则放行. (见2.3.3小节)

因此将该用户对应角色的所有权限存入到`Authentication`中, 即可实现角色与权限的功能.

> 在`userDetailsService.loadUserByUsername()`中修改代码

至于`ROLE_`前缀问题, 完全可以避免, 见参考链接

> 参考:
>
> * [Spring Security – Roles and Privileges](https://www.baeldung.com/role-and-privilege-for-spring-security-registration) 一篇关于角色和权限的实现教程
> * [Avoid using the prefix ROLE_](https://github.com/spring-projects/spring-security/issues/4912#issuecomment-353129360)

# 参考

* [Spring Security Reference](https://docs.spring.io/spring-security/site/docs/5.2.0.BUILD-SNAPSHOT/reference/htmlsingle/)
* [Spring Boot + Spring Security + JWT + MySQL + React Full Stack Polling app - Part 2](https://www.callicoder.com/spring-boot-spring-security-jwt-mysql-react-app-part-2/): 集成方案主要参考于此




















