# 介绍

JSON Web Token(JWT)是一个安全传递数据的token中的一种, 产生的token体积比较小, 但信息齐全.

JWT只是一种标准, 其实现库有很多.

## 结构

其结构由如下三部分组成, 分别以`.`分隔:

* **Header**: token的元数据. 通常由两个字段组成, 加密的算法和token的类型, 如

  ```json
  {
    "alg": "HS256",
    "typ": "JWT"
  }
  ```

  然后进行进行**Base64Url**编码

* **Payload**: token承载的数据, 数据中的每个字段被称为**claim**. JWT规定了标准的[claim](https://tools.ietf.org/html/rfc7519#section-4.1), 我个人比较喜欢自定义字段(claim). 一个例子如下:

  ```json
  {
    "sub": "1234567890",
    "name": "John Doe",
    "admin": true
  }
  ```

  然后进行**Base64Url**编码

  > 这里千万不要放敏感信息, 如密码

* **Signature**(签名): 是用于验证token未被修改的保障. 通过将前面两部分(包括分隔符`.`)+密钥Secret输入到加密算法中产生的签名. 如

  ```
  HMACSHA256(
    base64UrlEncode(header) + "." +
    base64UrlEncode(payload),
    secret)
  ```

然后将这三部分通过`.`组合起来就是前端收到的token了, 如:

![encoded-jwt3](.JWT/encoded-jwt3.png)

前端发送token给后端时, 通常将token放入头字段的`Authorization`中, 并加上前缀, 如:

```
Authorization: jwt <token>
```

> 前缀指定认证方式, 这里为`jwt`, 可自定义为其他的, 后端将解析它.

## 原理

### 为啥前面两部分使用Base64Url编码?

因为一般token要存入HTTP头字段中, 头字段中只能使用ascii编码, 因此要进行Base64Url编码, 转化为ascii码.

> 因此拿到token的人可以看到前面两部分的信息, 因此Payload中不要放敏感信息, 如密码.

### 如何保证数据不被篡改?

签名是通过Header, Payload, Secret和算法生成的, 只要拿到token的人不知道secret, 就不能伪造token. 

对于前端传来的假token, 后端再次通过上述步骤生成签名, 并与假token中的签名比较, 此时肯定会不同. 这样就判别出来了伪造的token.

> 只要Secret不泄露, 前端传来的token便将得到保证.

# 使用

这里使用[JWT的实现JJWT](https://github.com/jwtk/jjwt).

## 依赖引入

```xml
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt-api</artifactId>
    <version>0.10.7</version>
</dependency>
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt-impl</artifactId>
    <version>0.10.7</version>
    <scope>runtime</scope>
</dependency>
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt-jackson</artifactId>
    <version>0.10.7</version>
    <scope>runtime</scope>
</dependency>
<!-- Uncomment this next dependency if you want to use RSASSA-PSS (PS256, PS384, PS512) algorithms:
<dependency>
    <groupId>org.bouncycastle</groupId>
    <artifactId>bcprov-jdk15on</artifactId>
    <version>1.60</version>
    <scope>runtime</scope>
</dependency>
-->
```

## 密钥

JWT规范指定了三类可用签名算法, `HMAC`, `ECDSA`, `RSA`. 每一种算法都对密码长度, 强度都有要求.

密钥在Java中由`Key`表示,  JJWT提供了创建该类型对象的工具类`Keys`, 如

* 指定算法, 让JJWT自动创建一个随机的, 安全的, 满足算法要求的密钥

  ```java
  SecretKey key = Keys.secretKeyFor(SignatureAlgorithm.HS256); //or HS384 or HS512
  ```

* 指定密钥内容, 使用HMAC算法, 密钥内容必须满足该算法要求, 如至少32个字节

  ```java
  SecretKey key = Keys.hmacShaKeyFor(keyBytes);
  ```

* 生成非对称密钥, 略

> 关于`Key`接口, 它含有密钥和加密算法等信息

## 创建JWT

步骤

1. Use the `Jwts.builder()` method to create a `JwtBuilder` instance.
2. Call `JwtBuilder` methods to add header parameters and claims as desired.
3. Specify the `SecretKey` or asymmetric `PrivateKey` you want to use to sign the JWT.
4. Finally, call the `compact()` method to compact and sign, producing the final jws.

例子

```java
String jws = Jwts.builder() // (1)

    .setSubject("Bob")      // (2) 

    .signWith(key)          // (3)
     
    .compact();             // (4)
```

注意点

* ` JwtBuilder `提供了标准字段(Claim)的Setter方法, 也可自定义
* `Key`中包含了加密算法信息, 会自动设置头部.

## 解析JWT

步骤

1. Use the `Jwts.parser()` method to create a `JwtParser` instance.
2. Specify the `SecretKey` or asymmetric `PublicKey` you want to use to verify the JWS signature.1
3. Finally, call the `parseClaimsJws(String)` method with your jws `String`, producing the original JWS.
4. The entire call is wrapped in a try/catch block in case parsing or signature validation fails. We'll cover exceptions and causes for failure later.

例子

```java
Jws<Claims> jws;

try {
    jws = Jwts.parser()         // (1)
    .setSigningKey(key)         // (2)
    .parseClaimsJws(jwsString); // (3)
    
    // we can safely trust the JWT
     
} catch (JwtException ex) {     // (4)
    
    // we *cannot* use the JWT as intended by its creator
}
```

# 参考

* [JWT标准](https://jwt.io/introduction/)

* [JWT实现: JJWT](https://github.com/jwtk/jjwt)

