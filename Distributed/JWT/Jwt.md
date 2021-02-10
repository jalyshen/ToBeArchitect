# JWT，你必须了解的认证登录方案

原文：https://xie.infoq.cn/article/6b40282148451df8073209a8d



JWT 全称是 JSON Web Token，是目前非常流行的跨域认证解决方案，在单点登录场景中经常使用到。



JWT 就是一种Cookie-Session改造版的具体实现，让你省去自己造轮子的时间，JWT 还有个好处，那就是你可以不用在服务端存储认证信息（比如 token），完全由客户端提供，服务端只要根据 JWT 自身提供的解密算法就可以验证用户合法性，而且这个过程是安全的。

如果你是刚接触 JWT，最有疑问的一点可能就是： JWT 为什么可以完全依靠客户端(比如浏览器端)就能实现认证功能，认证信息全都存在客户端，怎么保证安全性？



## JWT数据结构

JWT 最后的形式就是个字符串，它由**头部**、**载荷**与**签名**这三部分组成，中间以「**.**」分隔。像下面这样：

![](images/structure.png)

​                                                            图 1 JWT格式

### 头部

头部以JSON格式表示，用于指明令牌类型和加密算法。形式如下，表示使用JWT格式，加密算法采用HS256，这是最常用的算法，除此之外还有很多其他的。

```json
{
  "alg": "HS256",
  "typ": "JWT"
}
```

对应图1中红色的head部分，需要Base64部分。

### 载荷

用来存储服务器需要的数据，比如用户信息（例如姓名、性别、年龄等），要注意的是重要的机密信息最好不要放到这里，比如密码等。

```json
{
  "name": "古时的风筝",
  "introduce": "英俊潇洒"
}
```

另外，JWT还规定了7个字段供开发者选用。

* iss(issuer): 签发人
* exp(expiration time): 过期时间
* sub(subject): 主题
* aud(audience): 受众
* nbf(Not Before): 生效时间
* iat(issued at): 签发时间
* jti(JWT ID): 编号

这部分信息也要用Base64编码。

### 签名

签名有一个计算公式：

```javascript
HMACSHA256(
  base64UrlEncode(header) + "." +
  base64UrlEncode(payload),
  Secret
)
```

适用HMACSHA256算法计算得出，这个方法有两个参数，前一个参数是（base64编码的头部 + base64编码的载荷）用点蚝相连，后一个参数是自定义的字符串密钥，密钥不要暴露在客户端，而应该服务器知道。

### 使用方式

了解了JWT的结构和算法后，那怎么使用呢？假设这儿有个网站。

1. 在用户登录网站的时候，需要输入用户名、密码或者短信验证码的方式登录，登录请求到达服务端的时候，服务端对账号、密码进行验证，然后计算出JWT字符串，返回给客户端
2. 客户端拿到这个JWT字符串后，存储到Cookie或者浏览器的LocalStorage中
3. 再次发送请求，比如请求用户设置页面的时候，在HTTP请求头中加入JWT字符串，或者直接放到请求主体中
4. 服务端拿到这串JWT字符串后，适用base64的头部和base64的载荷部分，通过HMACSHA256算法计算签名部分，比较计算结果和传来的签名部分是否一致。如果一致，说明此次请求没有问题，如果不一致，说明请求过期或者是非法请求

![](./images/JWT_using.png)

​                                                            图 2 服务端分发JWT

![](./images/JWT_using_2.png)

​                                                            图 3 服务端使用JWT

### 如何保证安全

保证安全性的关键就是HMACSHA256或者与它同类型的加密算法，因为加密过程是不可逆的，所以不能根据传到前端的JWT串解密到密钥信息。

另外，不同的头部和载荷加密之后得到的签名都是不同的。所以，如果有人改了载荷部分的信息，那最后加密出来的结果肯定就和改之前的不一样了。所以，最后验证的结果就是不合法的请求。

### 别人拿走了还安全吗

假设载荷部分存储了权限级别关键的字段，强盗拿到了JWT串后想要修改为更高级别的权限，上面说了，这种情况是不会得逞的，因为加密出来的签名会不一样，服务器可以很容易的判别出来。

如果抢到拿到后不做更改，直接使用呢？那就没有办法了。为了更大程度上防止被强盗盗取，应该使用HTTPS协议，这样可以有效地防止一些中间劫持攻击行为。

当然，如果终端设备被抢走了，而且从终端设备窃取到了JWT，那就没办法了。

所以使用JWT的时候仍然要注意合理设置过期时间。

### 一个问题

JWT有个问题，导致很多开发团队放弃使用它，那就是一旦颁发一个JWT令牌，服务端就没办法废弃它，除非等到它过期。很多应用默认只允许最新登录的一个客户端可以正常使用，不允许多端登录，JWT就没办法做到了。因为颁发了新令牌，但是老的令牌还没有过期仍然可以使用。这种情况下，需要服务端增加相应的逻辑。

## 常用的JWT库

JWT官方列出了各种语言的库，其中Java的有如下几个：

![](./images/JWT_libs.png)

​                                                            图 4 JWT官方推荐的Java类库

以 java-jwt为例：

1. 引入对应的Maven包

   ```xml
   <dependency>
       <groupId>com.auth0</groupId>
       <artifactId>java-jwt</artifactId>
       <version>3.10.3</version>
   </dependency>
   ```

2. 在登录时，调用create方法得到一个令牌，并返回给客户端

   ```java
   public static String create(){
     try {
       Algorithm algorithm = Algorithm.HMAC256("secret");
       String token = JWT.create()
         .withIssuer("auth0")
         .withSubject("subject")
         .withClaim("name","古时的风筝")
         .withClaim("introduce","英俊潇洒")
         .sign(algorithm);
       System.out.println(token);
       return token;
     } catch (JWTCreationException exception){
       //Invalid Signing configuration / Couldn't convert Claims.
       throw exception;
     }
   }
   
   ```

3. 登录成功后，再次发起请求的时候将token放到header或者请求体中，服务端对tokne进行验证

   ```java
   public static Boolean verify(String token){
     try {
       Algorithm algorithm = Algorithm.HMAC256("secret");
       JWTVerifier verifier = JWT.require(algorithm)
         .withIssuer("auth0")
         .build(); //Reusable verifier instance
       DecodedJWT jwt = verifier.verify(token);
       String payload = jwt.getPayload();
       String name = jwt.getClaim("name").asString();
       String introduce = jwt.getClaim("introduce").asString();
       System.out.println(payload);
       System.out.println(name);
       System.out.println(introduce);
       return true;
     } catch (JWTVerificationException exception){
       //Invalid signature/claims
       return false;
     }
   }
   ```

4. 用create方法生成token，并用verify方法验证

   ```java
   public static void main(String[] args){
     String token = create();
     Boolean result = verify(token);
     System.out.println(result);
   }
   ```

   得到如下结果：

   ```shell
   eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJzdWIiOiJzdWJqZWN0IiwiaW50cm9kdWNlIjoi6Iux5L-K5r2H5rSSIiwiaXNzIjoiYXV0aDAiLCJuYW1lIjoi5Y-k5pe255qE6aOO562dIn0.ooQ1K_XyljjHf34Nv5iJvg1MQgVe6jlphxv4eeFt8pA
   eyJzdWIiOiJzdWJqZWN0IiwiaW50cm9kdWNlIjoi6Iux5L-K5r2H5rSSIiwiaXNzIjoiYXV0aDAiLCJuYW1lIjoi5Y-k5pe255qE6aOO562dIn0
   古时的风筝
   英俊潇洒
   true
   ```

使用create方法创建JWT串可以通过验证

而如果我将 JWT 串中的载荷部分，两个点号中间的部分修改一下，然后再调用 verify 方法验证，会出现 JWTVerificationException异常，不能通过验证。
