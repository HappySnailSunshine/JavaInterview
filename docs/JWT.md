# JWT

(JSON Web Token)



## 目录

[TOC]



上面我们基本测试了一下Rest API的实现。但是没有使用rest接口的鉴权。

在实际开发过程中，rest风格的api也需要访问控制，并不是任意访问就可以去往数据库里插入东西的。所以我们需要JWT来做访问控制。

## 认识

JSON Web Token (JWT)是一个开放标准(RFC
7519)，它定义了一种紧凑的、自包含的方式，用于作为JSON对象在各方之间安全地传输信息。该信息可以被验证和信任，因为它是数字签名的。

## 使用场景

下列场景中使用JSON Web Token是很有用的：

- Authorization (授权) :
  这是使用JWT的最常见场景。一旦用户登录，后续每个请求都将包含JWT，允许用户访问该令牌允许的路由、服务和资源。单点登录是现在广泛使用的JWT的一个特性，因为它的开销很小，并且可以轻松地跨域使用。
- Information Exchange (信息交换) : 对于安全的在各方之间传输信息而言，JSON Web
  Tokens无疑是一种很好的方式。因为JWTs可以被签名，例如，用公钥/私钥对，你可以确定发送人就是它们所说的那个人。另外，由于签名是使用头和有效负载计算的，您还可以验证内容有没有被篡改

## 为啥要用JWT？

##### 跨域访问问题

比如你网站有几个不同的后台的域名，

你在a域名下登录，登录信息记录到cookie，那么后面只有访问a域名才会带cookie，其他域名是不会带cookie的。

那么怎么办？

这就是跨域请求问题。

浏览器的同源策略

##### 单点登录的问题

回顾我们之前集群的案例。

当时为了解决识别登录用户的问题，给出的两个解决方案。

方案1：nginx负载均衡做iphash，同一个用户的请求永远给到同一个后台服务器。

方案2：session复制。

都不好。

那么如何优化呢？我们来使用token解决这个问题。

## Token vs Session

基于Token的身份认证 与 基于服务器的身份认证

### Session

在讨论基于Token的身份认证是如何工作的以及它的好处之前，我们先来看一下以前我们是怎么做的：（基于session的身份认证）

HTTP协议是无状态的，也就是说，如果我们已经认证了一个用户，那么他下一次请求的时候，服务器不知道我是谁，我们必须再次认证

传统的做法是将已经认证过的用户信息存储在服务器上，比如Session。用户下次请求的时候带着Session
ID，然后服务器以此检查用户是否认证过。

这种基于服务器的身份认证方式存在一些问题：

- Sessions :
  每次用户认证通过以后，服务器需要创建一条记录保存用户信息，通常是在内存中，随着认证通过的用户越来越多，服务器的在这里的开销就会越来越大。
- Scalability : 由于Session是在内存中的，这就带来一些扩展性的问题。
- CORS :
  当我们想要扩展我们的应用，让我们的数据被多个移动设备使用时，我们必须考虑跨资源共享问题。当使用AJAX调用从另一个域名下获取资源时，我们可能会遇到禁止请求的问题。
- CSRF : 用户很容易受到CSRF攻击

### Token

基于token的身份认证其具体做法是这样的：

使用基于 Token 的身份验证方法，在服务端不需要存储用户的登录记录，大概流程如下：

　　1）客户端使用用户名、密码请求登录

　　2）服务端收到请求、去验证用户名与密码

　　3）验证成功后，服务端会签发一个 token ，再把这个 token 发送给客户端

　　4）客户端收到 token 以后可以把它存储起来，存到客户端内存或者其他地方。

　　5）客户端每次向服务器请求资源的时候需要带着服务器签发的 token

　　6）服务端收到请求，然后去验证客户端请求里面带着的
token，如果验证成功，就向客户端返回请求的数据。

相同点是，它们都是存储用户信息；然而，Session是在服务器端的，而JWT是在客户端的。

Session方式存储用户信息的最大问题在于要占用大量服务器内存，增加服务器的开销。

而token的方式将用户状态分散到了客户端中，可以明显减轻服务端的内存压力。

Session的状态是存储在服务器端，客户端只有session
id；而Token的状态是存储在客户端。

![](../media/pictures/JWT.assets/4b54f09c34952e641f4df81a99d9deb7.png)

![](../media/pictures/JWT.assets/de093a2a18b179d5b7cf51fb2270b620.png)

基于Token的身份认证是无状态的，服务器或者Session中不会存储任何用户信息。没有会话信息意味着应用程序可以根据需要扩展和添加更多的机器，而不必担心用户登录的位置。

虽然这一实现可能会有所不同，但其主要流程如下：

1. 用户携带用户名和密码请求访问
2. 服务器校验用户凭据
3. 应用提供一个token给客户端
4. 客户端存储token，并且在随后的每一次请求中都带着它
5. 服务器校验token并返回数据

注意：

1. 每一次请求都需要token
2. Token应该放在请求header中
3. 我们还需要将服务器设置为接受来自所有域的请求，用Access-Control-Allow-Origin：\*

### 用Token的好处

- 无状态和可扩展性：Tokens存储在客户端。完全无状态，可扩展。我们的负载均衡器可以将用户传递到任意服务器，因为在任何地方都没有状态或会话信息。
- 安全：Token不是Cookie。（The token, not a cookie.）每次请求的时候Token都会被发送。而且，由于没有Cookie被发送，还有助于防止CSRF攻击。即使在你的实现中将token存储到客户端的Cookie中，这个Cookie也只是一种存储机制，而非身份认证机制。没有基于会话的信息可以操作，因为我们没有会话!

## JWT token的基本格式

JSON Web Token的结构是什么样的

JSON Web Token由三部分组成，它们之间用圆点(.)连接。这三部分分别是：

- Header
- Payload
- Signature

因此，一个典型的JWT看起来是这个样子的：

xxxxx.yyyyy.zzzzz

![](../media/pictures/JWT.assets/d0c8efb0b67525fcc953fe7553c14f23.png)

具体如下图：

![](../media/pictures/JWT.assets/53ae025b667b66934296fba707138eef.png)

# Guns整合JWT签名机制 

## 介绍

### 基本流程

![](../media/pictures/JWT.assets/37c6c7f05a754f36d8b92e5dbe04a321.png)

## 实践：

### 第一步，开启配置

![](../media/pictures/JWT.assets/fd6b85c8ecd7741757f7c1b004aa69f6.png)

### 第二步 去登录获取钥匙

![](../media/pictures/JWT.assets/fab9d73279aa48982520fa4a102d95c3.png)

请求这个路径，传入用户名和密码，获取钥匙

![](../media/pictures/JWT.assets/e80154993b7afef8200087bbb4955de6.png)

响应报文：

{

"**randomKey**"**:** "yvg1ye",

"**token**"**:** "eyJhbGciOiJIUzUxMiJ9.eyJyYW5kb21LZXkiOiJ5dmcxeWUiLCJzdWIiOiJhZG1pbiIsImV4cCI6MTU1NTg0ODk0MiwiaWF0IjoxNTU1MjQ0MTQyfQ.clEVOLBNQr4dFAeK_4fEpPlzCo-zDjNxEsPxyBftbmjJRLGY9rrdSHUTmeckk-v8XCKmPKlxqXICyWkKGYZ7jA"

}

这个token就是我们的钥匙

### 使用token

#### 选择1 ：对数据不签名如果是不带参数 或者没有在请求正文

![](../media/pictures/JWT.assets/748bfd1182d62083e1b462db625c5dc0.png)

#### 选择2 需要对数据进行签名

第三步 客户端（前端），对即将传入服务器的数据进行签名

上面的盐值需要使用，token带回来的randomkey

![](../media/pictures/JWT.assets/f7ff93558fd6442b0f88e36b32590a13.png)

![](../media/pictures/JWT.assets/28cf499e854d2d60f4d9059081379e4f.png)

把提交的数据，

进行base64加密（编码）操作，得到encode 也就是object值

然后把encode和随机盐值 进行md5计算签名。得到如下：

{

"object":"eyJjaWQiOjIyMiwiZGVzY3JpcHRpb24iOiJhYWFhIiwiZXN0b3JlUHJpY2UiOjIyLjAsImltZ1VybCI6Ii9hZmRhc2QvYWRzZnNhZGYiLCJtYXJrUHJpY2UiOjQ0LjAsInBpZCI6MTgsInBuYW1lIjoi6LW15peg5p6BIiwicG51bSI6MjIyfQ==",

"sign":"cddb228fbfac92415afa12ae57861fd1"}

这就是经过签名之后的钥匙形式。

##### 第四步 将签名以后的数据传到服务器

![](../media/pictures/JWT.assets/e10a30afbcea24877c845ef09bf1d139.png)

##### 第五步，服务器拿到数据并验证

拦截器里验证token的有效性。

Token有效即放行。

请求参数封装之前验证数据签名的有效性。签名ok做请求参数封装。

结果如下：

![](../media/pictures/JWT.assets/09134b0327227938b4ca00ff84827c1d.png)

说明使用JWT token验证成功。数据成功插入数据库！

![](../media/pictures/JWT.assets/f6db2057dc91984496335aaaecd3789d.png)



## 使用JWT实现Token认证



为什么使用JWT？
随着技术的发展，分布式web应用的普及，通过session管理用户登录状态成本越来越高，因此慢慢发展成为token的方式做登录身份校验，然后通过token去取redis中的缓存的用户信息，随着之后jwt的出现，校验方式更加简单便捷化，无需通过redis缓存，而是直接根据token取出保存的用户信息，以及对token可用性校验，单点登录更为简单。

JWT架构图



![img](../media/pictures/JWT.assets/3383598-c82676bb8445bae9.webp)

JWT架构图.png

使用JWT核心代码：
maven依赖：

```xml
        <dependency>
            <groupId>com.auth0</groupId>
            <artifactId>java-jwt</artifactId>
            <version>3.2.0</version>
        </dependency>
        <dependency>
            <groupId>io.jsonwebtoken</groupId>
            <artifactId>jjwt</artifactId>
            <version>0.7.0</version>
        </dependency>
```

JWT工具类：
用于生成Token，和Token验证



```java
public class JwtUtils {
    /**
     * 签发JWT
     * @param id
     * @param subject 可以是JSON数据 尽可能少
     * @param ttlMillis
     * @return  String
     *
     */
    public static String createJWT(String id, String subject, long ttlMillis) {
        SignatureAlgorithm signatureAlgorithm = SignatureAlgorithm.HS256;
        long nowMillis = System.currentTimeMillis();
        Date now = new Date(nowMillis);
        SecretKey secretKey = generalKey();
        JwtBuilder builder = Jwts.builder()
                .setId(id)
                .setSubject(subject)   // 主题
                .setIssuer("user")     // 签发者
                .setIssuedAt(now)      // 签发时间
                .signWith(signatureAlgorithm, secretKey); // 签名算法以及密匙
        if (ttlMillis >= 0) {
            long expMillis = nowMillis + ttlMillis;
            Date expDate = new Date(expMillis);
            builder.setExpiration(expDate); // 过期时间
        }
        return builder.compact();
    }
    /**
     * 验证JWT
     * @param jwtStr
     * @return
     */
    public static CheckResult validateJWT(String jwtStr) {
        CheckResult checkResult = new CheckResult();
        Claims claims = null;
        try {
            claims = parseJWT(jwtStr);
            checkResult.setSuccess(true);
            checkResult.setClaims(claims);
        } catch (ExpiredJwtException e) {
            checkResult.setErrCode(SystemConstant.JWT_ERRCODE_EXPIRE);
            checkResult.setSuccess(false);
        } catch (SignatureException e) {
            checkResult.setErrCode(SystemConstant.JWT_ERRCODE_FAIL);
            checkResult.setSuccess(false);
        } catch (Exception e) {
            checkResult.setErrCode(SystemConstant.JWT_ERRCODE_FAIL);
            checkResult.setSuccess(false);
        }
        return checkResult;
    }
    public static SecretKey generalKey() {
        byte[] encodedKey = Base64.decode(SystemConstant.JWT_SECERT);
        SecretKey key = new SecretKeySpec(encodedKey, 0, encodedKey.length, "AES");
        return key;
    }
    
    /**
     * 
     * 解析JWT字符串
     * @param jwt
     * @return
     * @throws Exception
     */
    public static Claims parseJWT(String jwt) throws Exception {
        SecretKey secretKey = generalKey();
        return Jwts.parser()
            .setSigningKey(secretKey)
            .parseClaimsJws(jwt)
            .getBody();
    }
}
```

如何使用？
代码实例：



```kotlin
public class LoginController {
    @Autowired
    UserRepository userRepository;
    
    @ApiOperation(value="用户登陆")
    @RequestMapping(value="login",method = RequestMethod.POST)
    public ReturnVo login(String username, String password,HttpServletResponse
            response) {
        User user =  userRepository.findByUsername(username);
        if(user!=null){
            if(user.getPassword().equals(password)){
                //把token返回给客户端-->客户端保存至cookie-->客户端每次请求附带cookie参数
                String JWT = JwtUtils.createJWT("1", username, SystemConstant.JWT_TTL);
                return ReturnVo.ok(JWT);
            }else{
                return ReturnVo.error();
            }
        }else{
            return ReturnVo.error();
        }
    }
    @ApiOperation(value="获取用户信息")
    @RequestMapping(value="description",method = RequestMethod.POST)
    public ReturnVo description(String username) {
        User user =  userRepository.findByUsername(username);
        return ReturnVo.ok(user.getDescription());
    }
}
```

## 使用JWT实现单点登录（完全跨域方案）

### 首先介绍一下什么是JSON Web Token（JWT）？

官方文档是这样解释的：JSON Web Token（JWT）是一个开放标准（RFC 7519），它定义了一种紧凑且独立的方式，可以在各方之间作为JSON对象安全地传输信息。此信息可以通过数字签名进行验证和信任。JWT可以使用秘密（使用HMAC算法）或使用RSA或ECDSA的公钥/私钥对进行签名。
虽然JWT可以加密以在各方之间提供保密，但只将专注于签名令牌。签名令牌可以验证其中包含的声明的完整性，而加密令牌则隐藏其他方的声明。当使用公钥/私钥对签署令牌时，签名还证明只有持有私钥的一方是签署私钥的一方。

通俗来讲，JWT是一个含签名并携带用户相关信息的加密串，页面请求校验登录接口时，请求头中携带JWT串到后端服务，后端通过签名加密串匹配校验，保证信息未被篡改。校验通过则认为是可靠的请求，将正常返回数据。

### 什么情况下使用JWT比较适合？

授权：这是最常见的使用场景，解决单点登录问题。因为JWT使用起来轻便，开销小，服务端不用记录用户状态信息（无状态），所以使用比较广泛；
信息交换：JWT是在各个服务之间安全传输信息的好方法。因为JWT可以签名，例如，使用公钥/私钥对儿 - 可以确定请求方是合法的。此外，由于使用标头和有效负载计算签名，还可以验证内容是否未被篡改。
JWT的结构体是什么样的？
JWT由三部分组成，分别是头信息、有效载荷、签名，中间以（.）分隔，如下格式：

```java
xxx.yyy.zzz
```



#### header（头信息）

由两部分组成，令牌类型（即：JWT）、散列算法（HMAC、RSASSA、RSASSA-PSS等），例如：

```java
{
  "alg": "HS256",
  "typ": "JWT"
}
```

然后，这个JSON被编码为Base64Url，形成JWT的第一部分。

然后，这个JSON被编码为Base64Url，形成JWT的第一部分。

**Payload（有效载荷）**

JWT的第二部分是payload，其中包含claims。claims是关于实体（常用的是用户信息）和其他数据的声明，claims有三种类型： registered, public, and private claims。
**Registered claims**： 这些是一组预定义的claims，非强制性的，但是推荐使用， iss（发行人）， exp（到期时间）， sub（主题）， aud（观众）等；
**Public claims:** 自定义claims，注意不要和JWT注册表中属性冲突，这里可以查看JWT注册表
**Private claims**: 这些是自定义的claims，用于在同意使用这些claims的各方之间共享信息，它们既不是Registered claims，也不是Public claims。
以下是payload示例：

```java
{
  "sub": "1234567890",
  "name": "John Doe",
  "admin": true
}
```

然后，再经过Base64Url编码，形成JWT的第二部分；

**注意**：对于签名令牌，此信息虽然可以防止篡改，但任何人都可以读取。除非加密，否则不要将敏感信息放入到Payload或Header元素中。

#### Signature

要创建签名部分，必须采用编码的Header，编码的Payload，秘钥，Header中指定的算法，并对其进行签名。
例如，如果要使用HMAC SHA256算法，将按以下方式创建签名：

```java
HMACSHA256(
  base64UrlEncode(header) + "." +
  base64UrlEncode(payload),
  secre
```

签名用于验证消息在此过程中未被篡改，并且，在使用私钥签名令牌的情况下，它还可以验证JWT的请求方是否是它所声明的请求方。
输出是三个由点分隔的Base64-URL字符串，可以在HTML和HTTP环境中轻松传递，与SAML等基于XML的标准相比更加紧凑。
例如：

```java
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.
eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ.
SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c
```

### JWT工作机制？

在身份验证中，当用户使用其凭据成功登录时，将返回JSON Web Token（即：JWT）。由于令牌是凭证，因此必须非常小心以防止出现安全问题。一般情况下，不应将令牌保留的时间超过要求。理论上超时时间越短越好。

每当用户想要访问受保护的路由或资源时，用户代理应该使用Bearer模式发送JWT，通常在Authorization header中。标题内容应如下所示：

```java
Authorization: Bearer <token>
```

在某些情况下，这可以作为无状态授权机制。服务器的受保护路由将检查Authorization header中的有效JWT ，如果有效，则允许用户访问受保护资源。如果JWT包含必要的数据，则可以减少查询数据库或缓存信息。
如果在Authorization header中发送令牌，则跨域资源共享（CORS）将不会成为问题，因为它不使用cookie。

注意：使用签名令牌，虽然他们无法更改，但是令牌中包含的所有信息都会向用户或其他方公开。这意味着不应该在令牌中放置敏感信息。

#### 使用JWT的好处是什么？

相比Simple Web Tokens (SWT)（简单Web令牌） and Security Assertion Markup Language Tokens (SAML)（安全断言标记语言令牌）；

JWT比SAML更简洁，在HTML和HTTP环境中传递更方便；
在安全方面，SWT只能使用HMAC算法通过共享密钥对称签名。但是，JWT和SAML令牌可以使用X.509证书形式的公钥/私钥对进行签名。与签名JSON的简单性相比，使用XML数字签名可能会存在安全漏洞；
JSON解析成对象相比XML更流行、方便。

#### 以下是我实际项目中的应用分析

首先看一下大致的架构及流程图：

![image-20191220103857340](../media/pictures/JWT.assets/image-20191220103857340.png)

![image-20191220103915311](../media/pictures/JWT.assets/image-20191220103915311.png)

#### 主要有以下三步：

项目一开始我先封装了一个JWTHelper工具包（GitHub下载），主要提供了生成JWT、解析JWT以及校验JWT的方法，其他还有一些加密相关操作，稍后我会以代码的形式介绍下代码。工具包写好后我将打包上传到私服，能够随时依赖下载使用；
接下来，我在客户端项目中依赖JWTHelper工具包，并添加Interceptor拦截器，拦截需要校验登录的接口。拦截器中校验JWT有效性，并在response中重新设置JWT的新值；
最后在JWT服务端，依赖JWT工具包，在登录方法中，需要在登录校验成功后调用生成JWT方法，生成一个JWT令牌并且设置到response的header中。
以下是部分代码分享：

**JwtHelper工具类：**

```java
/**

 * @Author: Helon

 * @Description: JWT工具类

 * 参考官网：https://jwt.io/

 * JWT的数据结构为：A.B.C三部分数据，由字符点"."分割成三部分数据

 * A-header头信息

 * B-payload 有效负荷 一般包括：已注册信息（registered claims），公开数据(public claims)，私有数据(private claims)

 * C-signature 签名信息 是将header和payload进行加密生成的

 * @Data: Created in 2018/7/19 14:11

 * @Modified By:
   */
   public class JwtHelper {

   private static Logger logger = LoggerFactory.getLogger(JwtHelper.class);

   /**

    * @Author: Helon
    * @Description: 生成JWT字符串
    * 格式：A.B.C
    * A-header头信息
    * B-payload 有效负荷
    * C-signature 签名信息 是将header和payload进行加密生成的
    * @param userId - 用户编号
    * @param userName - 用户名
    * @param identities - 客户端信息（变长参数），目前包含浏览器信息，用于客户端拦截器校验，防止跨域非法访问
    * @Data: 2018/7/28 19:26
    * @Modified By:
      */
      public static String generateJWT(String userId, String userName, String ...identities) {
      //签名算法，选择SHA-256
      SignatureAlgorithm signatureAlgorithm = SignatureAlgorithm.HS256;
      //获取当前系统时间
      long nowTimeMillis = System.currentTimeMillis();
      Date now = new Date(nowTimeMillis);
      //将BASE64SECRET常量字符串使用base64解码成字节数组
      byte[] apiKeySecretBytes = DatatypeConverter.parseBase64Binary(SecretConstant.BASE64SECRET);
      //使用HmacSHA256签名算法生成一个HS256的签名秘钥Key
      Key signingKey = new SecretKeySpec(apiKeySecretBytes, signatureAlgorithm.getJcaName());
      //添加构成JWT的参数
      Map<String, Object> headMap = new HashMap<>();
      /*
          Header
          {
            "alg": "HS256",
            "typ": "JWT"
          }
       */
      headMap.put("alg", SignatureAlgorithm.HS256.getValue());
      headMap.put("typ", "JWT");
      JwtBuilder builder = Jwts.builder().setHeader(headMap)
              /*
                  Payload
                  {
                    "userId": "1234567890",
                    "userName": "John Doe",
                  }
               */
              //加密后的客户编号
              .claim("userId", AESSecretUtil.encryptToStr(userId, SecretConstant.DATAKEY))
              //客户名称
              .claim("userName", userName)
              //客户端浏览器信息
              .claim("userAgent", identities[0])
              //Signature
              .signWith(signatureAlgorithm, signingKey);
      //添加Token过期时间
      if (SecretConstant.EXPIRESSECOND >= 0) {
          long expMillis = nowTimeMillis + SecretConstant.EXPIRESSECOND;
          Date expDate = new Date(expMillis);
          builder.setExpiration(expDate).setNotBefore(now);
      }
      return builder.compact();
      }

   /**

    * @Author: Helon
    * @Description: 解析JWT
    * 返回Claims对象
    * @param jsonWebToken - JWT
    * @Data: 2018/7/28 19:25
    * @Modified By:
      */
      public static Claims parseJWT(String jsonWebToken) {
      Claims claims = null;
      try {
          if (StringUtils.isNotBlank(jsonWebToken)) {
              //解析jwt
              claims = Jwts.parser().setSigningKey(DatatypeConverter.parseBase64Binary(SecretConstant.BASE64SECRET))
                      .parseClaimsJws(jsonWebToken).getBody();
          }else {
              logger.warn("[JWTHelper]-json web token 为空");
          }
      } catch (Exception e) {
          logger.error("[JWTHelper]-JWT解析异常：可能因为token已经超时或非法token");
      }
      return claims;
      }

   /**

    * @Author: Helon
    * @Description: 校验JWT是否有效
    * 返回json字符串的demo:
    * {"freshToken":"A.B.C","userName":"Judy","userId":"123", "userAgent":"xxxx"}
    * freshToken-刷新后的jwt
    * userName-客户名称
    * userId-客户编号
    * userAgent-客户端浏览器信息
    * @param jsonWebToken - JWT
    * @Data: 2018/7/24 15:28
    * @Modified By:
      */
      public static String validateLogin(String jsonWebToken) {
      Map<String, Object> retMap = null;
      Claims claims = parseJWT(jsonWebToken);
      if (claims != null) {
          //解密客户编号
          String decryptUserId = AESSecretUtil.decryptToStr((String)claims.get("userId"), SecretConstant.DATAKEY);
          retMap = new HashMap<>();
          //加密后的客户编号
          retMap.put("userId", decryptUserId);
          //客户名称
          retMap.put("userName", claims.get("userName"));
          //客户端浏览器信息
          retMap.put("userAgent", claims.get("userAgent"));
          //刷新JWT
          retMap.put("freshToken", generateJWT(decryptUserId, (String)claims.get("userName"), (String)claims.get("userAgent"), (String)claims.get("domainName")));
      }else {
          logger.warn("[JWTHelper]-JWT解析出claims为空");
      }
      return retMap!=null?JSONObject.toJSONString(retMap):null;
      }

   public static void main(String[] args) {
      String jsonWebKey = generateJWT("123", "Judy",
              "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/68.0.3440.106 Safari/537.36");
      System.out.println(jsonWebKey);
      Claims claims =  parseJWT(jsonWebKey);
       System.out.println(claims);
      System.out.println(validateLogin(jsonWebKey));
   }

   AES加密工具类：
   /**

 * @Author: Helon

 * @Description: AES加密工具类

 * @Data: Created in 2018/7/28 18:38

 * @Modified By:
   */
   public class AESSecretUtil {

   /**秘钥的大小*/
   private static final int KEYSIZE = 128;

   /**

    * @Author: Helon
    * @Description: AES加密
    * @param data - 待加密内容
    * @param key - 加密秘钥
    * @Data: 2018/7/28 18:42
    * @Modified By:
      */
      public static byte[] encrypt(String data, String key) {
      if(StringUtils.isNotBlank(data)){
          try {
              KeyGenerator keyGenerator = KeyGenerator.getInstance("AES");
              //选择一种固定算法，为了避免不同java实现的不同算法，生成不同的密钥，而导致解密失败
              SecureRandom random = SecureRandom.getInstance("SHA1PRNG");
              random.setSeed(key.getBytes());
              keyGenerator.init(KEYSIZE, random);
              SecretKey secretKey = keyGenerator.generateKey();
              byte[] enCodeFormat = secretKey.getEncoded();
              SecretKeySpec secretKeySpec = new SecretKeySpec(enCodeFormat, "AES");
              Cipher cipher = Cipher.getInstance("AES");// 创建密码器
              byte[] byteContent = data.getBytes("utf-8");
              cipher.init(Cipher.ENCRYPT_MODE, secretKeySpec);// 初始化
              byte[] result = cipher.doFinal(byteContent);
              return result; // 加密
          } catch (Exception e) {
              e.printStackTrace();
          }
      }
      return null;
      }

   /**

    * @Author: Helon

    * @Description: AES加密，返回String

    * @param data - 待加密内容

    * @param key - 加密秘钥

    * @Data: 2018/7/28 18:59

    * @Modified By:
      */
      public static String encryptToStr(String data, String key){

      return StringUtils.isNotBlank(data)?parseByte2HexStr(encrypt(data, key)):null;
      }
```

 **AES加密工具类：** 

```java
/**
 * @Author: Helon
 * @Description: AES解密
 * @param data - 待解密字节数组
 * @param key - 秘钥
 * @Data: 2018/7/28 19:01
 * @Modified By:
 */
public static byte[] decrypt(byte[] data, String key) {
    if (ArrayUtils.isNotEmpty(data)) {
        try {
            KeyGenerator keyGenerator = KeyGenerator.getInstance("AES");
            //选择一种固定算法，为了避免不同java实现的不同算法，生成不同的密钥，而导致解密失败
            SecureRandom random = SecureRandom.getInstance("SHA1PRNG");
            random.setSeed(key.getBytes());
            keyGenerator.init(KEYSIZE, random);
            SecretKey secretKey = keyGenerator.generateKey();
            byte[] enCodeFormat = secretKey.getEncoded();
            SecretKeySpec secretKeySpec = new SecretKeySpec(enCodeFormat, "AES");
            Cipher cipher = Cipher.getInstance("AES");// 创建密码器
            cipher.init(Cipher.DECRYPT_MODE, secretKeySpec);// 初始化
            byte[] result = cipher.doFinal(data);
            return result; // 加密
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
    return null;
}

/**
 * @Author: Helon
 * @Description: AES解密，返回String
 * @param enCryptdata - 待解密字节数组
 * @param key - 秘钥
 * @Data: 2018/7/28 19:01
 * @Modified By:
 */
public static String decryptToStr(String enCryptdata, String key) {
    return StringUtils.isNotBlank(enCryptdata)?new String(decrypt(parseHexStr2Byte(enCryptdata), key)):null;
}

/**
 * @Author: Helon
 * @Description: 将二进制转换成16进制
 * @param buf - 二进制数组
 * @Data: 2018/7/28 19:12
 * @Modified By:
 */
public static String parseByte2HexStr(byte buf[]) {
    StringBuffer sb = new StringBuffer();
    for (int i = 0; i < buf.length; i++) {
        String hex = Integer.toHexString(buf[i] & 0xFF);
        if (hex.length() == 1) {
            hex = '0' + hex;
        }
        sb.append(hex.toUpperCase());
    }
    return sb.toString();
}

/**
 * @Author: Helon
 * @Description: 将16进制转换为二进制
 * @param hexStr - 16进制字符串
 * @Data: 2018/7/28 19:13
 * @Modified By:
 */
public static byte[] parseHexStr2Byte(String hexStr) {
    if (hexStr.length() < 1)
        return null;
    byte[] result = new byte[hexStr.length()/2];
    for (int i = 0;i< hexStr.length()/2; i++) {
        int high = Integer.parseInt(hexStr.substring(i*2, i*2+1), 16);
        int low = Integer.parseInt(hexStr.substring(i*2+1, i*2+2), 16);
        result[i] = (byte) (high * 16 + low);
    }
    return result;
}

public static void main(String[] args) {
    String ss = encryptToStr("eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1c2VySWQiOiIxMjMiLCJ1c2VyTmFtZSI6Ikp1ZHkiLCJleHAiOjE1MzI3Nzk2MjIsIm5iZiI6MTUzMjc3NzgyMn0.sIw_leDZwG0pJ8ty85Iecd_VXjObYutILNEwPUyeVSo", SecretConstant.DATAKEY);
    System.out.println(ss);
    System.out.println(decryptToStr(ss, SecretConstant.DATAKEY));
}
```

**所使用的常量类：**

```java
/**

 * @Author: Helon

 * @Description: JWT使用常量值

 * @Data: Created in 2018/7/27 14:37

 * @Modified By:
   */
   public class SecretConstant {

   //签名秘钥 自定义
   public static final String BASE64SECRET = "***********";

   //超时毫秒数（默认30分钟）
   public static final int EXPIRESSECOND = 1800000;

   //用于JWT加密的密匙 自定义
   public static final String DATAKEY = "************";

}
```

**客户端pom依赖：**

```java
     <!--jwt工具类-->
        <dependency>
            <groupId>com.chtwm.component</groupId>
            <artifactId>jwt-helper</artifactId>
            <version>xxx</version>
        </dependency>
```

**客户端拦截器：**

```java
/**

 * @Author: Helon

 * @Description: 校验是否登录拦截器

 * @Data: Created in 2018/7/30 14:30

 * @Modified By:
   */
   @Slf4j
   public class ValidateLoginInterceptor implements HandlerInterceptor {

   @Override
   public boolean preHandle(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, Object o) throws Exception {
       //首先从请求头中获取jwt串，与页面约定好存放jwt值的请求头属性名为User-Token
       String jwt = httpServletRequest.getHeader("User-Token");
       log.info("[登录校验拦截器]-从header中获取的jwt为:{}", jwt);
       //判断jwt是否有效
       if(StringUtils.isNotBlank(jwt)){
           //校验jwt是否有效,有效则返回json信息，无效则返回空
           String retJson = JwtHelper.validateLogin(jwt);
           log.info("[登录校验拦截器]-校验JWT有效性返回结果:{}", retJson);
           //retJSON为空则说明jwt超时或非法
           if(StringUtils.isNotBlank(retJson)){
               JSONObject jsonObject = JSONObject.parseObject(retJson);
               //校验客户端信息
               String userAgent = httpServletRequest.getHeader("User-Agent");
               if (userAgent.equals(jsonObject.getString("userAgent"))) {
                   //获取刷新后的jwt值，设置到响应头中
                   httpServletResponse.setHeader("User-Token", jsonObject.getString("freshToken"));
                   //将客户编号设置到session中
                   httpServletRequest.getSession().setAttribute(GlobalConstant.SESSION_CUSTOMER_NO_KEY, jsonObject.getString("userId"));
                   return true;
               }else{
                   log.warn("[登录校验拦截器]-客户端浏览器信息与JWT中存的浏览器信息不一致，重新登录。当前浏览器信息:{}", userAgent);
               }
           }else {
               log.warn("[登录校验拦截器]-JWT非法或已超时，重新登录");
           }
       }
       //输出响应流
       JSONObject jsonObject = new JSONObject();
       jsonObject.put("hmac", "");
       jsonObject.put("status", "");
       jsonObject.put("code", "4007");
       jsonObject.put("msg", "未登录");
       jsonObject.put("data", "");
       httpServletResponse.setCharacterEncoding("UTF-8");
       httpServletResponse.setContentType("application/json; charset=utf-8");
       httpServletResponse.getOutputStream().write(jsonObject.toJSONString().getBytes("UTF-8"));
       return false;
   }

   @Override
   public void postHandle(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, Object o, ModelAndView modelAndView) throws Exception {

   }

   @Override
   public void afterCompletion(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, Object o, Exception e) throws Exception {

   }
   }
```



**客户端拦截器在XML文件中配置：**

```java
<!--拦截器配置-->
<mvc:interceptors>
    <mvc:interceptor>
        <!--需拦截url配置-->
        <mvc:exclude-mapping path="/api/aa/bb/**" />
        <mvc:exclude-mapping path="/api/aa/cc/test" />
        <bean id="validateLoginInterceptor" class="com.xxx.ValidateLoginInterceptor" />
    </mvc:interceptor>
</mvc:interceptors>
```

到此，后台服务的配置已经完成，下一步就需要前端页面将JWT令牌从response响应头中取出，然后存入localstorage或cookie中。但是遇到跨域场景，处理起来就会比较复杂，因为一旦在浏览器中跨域将获取不到localstorage中的JWT令牌。例如www.a.com域下的JWT，在www.b.com域下是获取不到的，所以我选择了一种页面跨域的方式进行处理，使用iframe+H5的postMessage（参考博文），具体我使用代码分享的方式来分析。

客户端拦截器在XML文件中配置：

```java
<!--拦截器配置-->
<mvc:interceptors>
    <mvc:interceptor>
        <!--需拦截url配置-->
        <mvc:exclude-mapping path="/api/aa/bb/**" />
        <mvc:exclude-mapping path="/api/aa/cc/test" />
        <bean id="validateLoginInterceptor" class="com.xxx.ValidateLoginInterceptor" />
    </mvc:interceptor>
</mvc:interceptors>
```

到此，后台服务的配置已经完成，下一步就需要前端页面将JWT令牌从response响应头中取出，然后存入localstorage或cookie中。但是遇到跨域场景，处理起来就会比较复杂，因为一旦在浏览器中跨域将获取不到localstorage中的JWT令牌。例如www.a.com域下的JWT，在www.b.com域下是获取不到的，所以我选择了一种页面跨域的方式进行处理，使用iframe+H5的postMessage（参考博文），具体我使用代码分享的方式来分析。


到此，后台服务的配置已经完成，下一步就需要前端页面将JWT令牌从response响应头中取出，然后存入localstorage或cookie中。但是遇到跨域场景，处理起来就会比较复杂，因为一旦在浏览器中跨域将获取不到localstorage中的JWT令牌。例如www.a.com域下的JWT，在www.b.com域下是获取不到的，所以我选择了一种页面跨域的方式进行处理，使用iframe+H5的postMessage（参考博文），具体我使用代码分享的方式来分析。

**前端页面js代码（服务端）：**

```java
 /**CURD本地存储信息 start**/
          (function(doc,win){
              var fn=function(){};
              fn.prototype={
                  /*本地数据存储 t:cookie有效时间，单位s; domain:cookie存储所属的domain域*/
                  setLocalCookie: function (k, v, t,domain) {
                      //如果当前浏览器不支持localStorage将存储在cookie中
                      typeof window.localStorage !== "undefined" ? localStorage.setItem(k, v) :
                          (function () {
                              t = t || 365 * 12 * 60 * 60;
                              domain=domain?domain:".jwtserver.com";
                              document.cookie = k + "=" + v + ";max-age=" + t+";domain="+domain+";path=/";
                          })()
                  },
                  /*获取本地存储数据*/
                  getLocalCookie: function (k) {
                      k = k || "localDataTemp";
                      return typeof window.localStorage !== "undefined" ? localStorage.getItem(k) :
                          (function () {
                              var all = document.cookie.split(";");
                              var cookieData = {};
                              for (var i = 0, l = all.length; i < l; i++) {
                                  var p = all[i].indexOf("=");
                                  var dataName = all[i].substring(0, p).replace(/^[\s\uFEFF\xA0]+|[\s\uFEFF\xA0]+$/g,"");
                                  cookieData[dataName] = all[i].substring(p + 1);
                              }
                              return cookieData[k]
                          })();
                  },
                  /*删除本地存储数据*/
                  clearLocalData: function (k) {
                      k = k || "localDataTemp";
                      typeof window.localStorage !== "undefined" ? localStorage.removeItem(k) :
                          (function () {
                              document.cookie = k + "=temp" + ";max-age=0";
                          })()
                  },
                  init:function(){
                      this.bindEvent();
                  },
                  //事件绑定
                  bindEvent:function(){
                      var _this=this;
                      win.addEventListener("message",function(evt){
                          if(win.parent!=evt.source){return}
                          var options=JSON.parse(evt.data);
                          if(options.type=="GET"){
                              var data=tools.getLocalCookie(options.key);
                              win.parent.postMessage(data, "*");
                          }
                          options.type=="SET"&&_this.setLocalCookie(options.key,options.value);
                          options.type=="REM"&&_this.clearLocalData(options.key);
                      },false)
                  }
              };
              var tools=new fn();
              tools.init();
          })(document,window);
          /**CURD本地存储信息 end**/

```

**前端页面js代码（客户端）：**
      

```js
  //页面初始化向iframe域名发送消息
        window.onload = function() {
            console.log('get key value......................')
            window.frames[0].postMessage(JSON.stringify({type:"GET",key:"User-Token"}),'*');
        }
        //监听message信息，接收从iframe域下获取到的token信息，然后存储到localstorage或cookie中
        window.addEventListener('message', function(e) {
            console.log('listen.....');
            var data = e.data;
            console.log(data);
            if(data != null){
                localStorage.setItem("User-Token", data);
            }
        }, false);

```

总结：
优点：在非跨域环境下使用JWT机制是一个非常不错的选择，实现方式简单，操作方便，能够快速实现。由于服务端不存储用户状态信息，因此大用户量，对后台服务也不会造成压力；
缺点：跨域实现相对比较麻烦，安全性也有待探讨。因为JWT令牌返回到页面中，可以使用js获取到，如果遇到XSS攻击令牌可能会被盗取，在JWT还没超时的情况下，就会被获取到敏感数据信息。





### JWT

JSON Web Token（JWT）是一个非常轻巧的规范。这个规范允许我们使用JWT在用户和服务器之间传递安全可靠的信息。

一个JWT实际上就是一个字符串，它由三部分组成，头部、载荷与签名。

**头部（Header）**

头部用于描述关于该JWT的最基本的信息，例如其类型以及签名所用的算法等。这也可以被表示成一个JSON对象。

{"typ":"JWT","alg":"HS256"}

在头部指明了签名算法是HS256算法。 我们进行BASE64编码[http://base64.xpcha.com/](https://links.jianshu.com/go?to=http%3A%2F%2Fbase64.xpcha.com%2F)，编码后的字符串如下：



```undefined
eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9

```

**载荷（playload）**

载荷就是存放有效信息的地方。这个名字像是特指飞机上承载的货品，这些有效信息包含三个部分

（1）标准中注册的声明（建议但不强制使用）



```cpp
iss: jwt签发者
sub: jwt所面向的用户
aud: 接收jwt的一方
exp: jwt的过期时间，这个过期时间必须要大于签发时间
nbf: 定义在什么时间之前，该jwt都是不可用的.
iat: jwt的签发时间
jti: jwt的唯一身份标识，主要用来作为一次性token。

```

（2）公共的声明

公共的声明可以添加任何的信息，一般添加用户的相关信息或其他业务需要的必要信息.但不建议添加敏感信息，因为该部分在客户端可解密.

（3）私有的声明

私有声明是提供者和消费者所共同定义的声明，一般不建议存放敏感信息，因为base64是对称解密的，意味着该部分信息可以归类为明文信息。

这个指的就是自定义的claim。比如前面那个结构举例中的admin和name都属于自定的claim。这些claim跟JWT标准规定的claim区别在于：JWT规定的claim，JWT的接收方在拿到JWT之后，都知道怎么对这些标准的claim进行验证(还不知道是否能够验证)；而private claims不会验证，除非明确告诉接收方要对这些claim进行验证以及规则才行。

定义一个payload:



```json
{"sub":"1234567890","name":"John Doe","admin":true}

```

然后将其进行base64加密，得到Jwt的第二部分。



```undefined
eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiYWRtaW4iOnRydWV9

```

**签证（signature）**

jwt的第三部分是一个签证信息，这个签证信息由三部分组成：

> header (base64后的)
>
> payload (base64后的)
>
> secret

这个部分需要base64加密后的header和base64加密后的payload使用.连接组成的字符串，然后通过header中声明的加密方式进行加盐secret组合加密，然后就构成了jwt的第三部分。



```css
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiYWRtaW4iOnRydWV9.TJVA95OrM7E2cBab30RMHrHDcEfxjoYZgeFONFh7HgQ

```

**注意**：secret是保存在服务器端的，jwt的签发生成也是在服务器端的，secret就是用来进行jwt的签发和jwt的验证，所以，它就是你服务端的私钥，在任何场景都不应该流露出去。一旦客户端得知这个secret, 那就意味着客户端是可以自我签发jwt了。



## JWT签发与验证token

JJWT是一个提供端到端的JWT创建和验证的Java库。永远免费和开源(Apache License，版本2.0)，JJWT很容易使用和理解。它被设计成一个以建筑为中心的流畅界面，隐藏了它的大部分复杂性。

官方文档：

[https://github.com/jwtk/jjwt](https://links.jianshu.com/go?to=https%3A%2F%2Fgithub.com%2Fjwtk%2Fjjwt)

#### 创建token

（1）新建项目中的pom.xml中添加依赖：



```xml
<dependency>
 <groupId>io.jsonwebtoken</groupId>
 <artifactId>jjwt</artifactId>
 <version>0.9.0</version>
</dependency>

```

（2）创建测试类，代码如下



```csharp
JwtBuilder builder= Jwts.builder()
 .setId("888")   //设置唯一编号
 .setSubject("小白")//设置主题  可以是JSON数据
 .setIssuedAt(new Date())//设置签发日期
 .signWith(SignatureAlgorithm.HS256,"hahaha");//设置签名 使用HS256算法，并设置SecretKey(字符串)
//构建 并返回一个字符串 
System.out.println( builder.compact() );

```

运行打印结果：



```css
eyJhbGciOiJIUzI1NiJ9.eyJqdGkiOiI4ODgiLCJzdWIiOiLlsI_nmb0iLCJpYXQiOjE1NTc5MDQxODF9.ThecMfgYjtoys3JX7dpx3hu6pUm0piZ0tXXreFU_u3Y

```

再次运行，会发现每次运行的结果是不一样的，因为我们的载荷中包含了时间。

#### 解析token

我们刚才已经创建了token ，在web应用中这个操作是由服务端进行然后发给客户端，客户端在下次向服务端发送请求时需要携带这个token（这就好像是拿着一张门票一样），那服务端接到这个token 应该解析出token中的信息（例如用户id）,根据这些信息查询数据库返回相应的结果。



```csharp
 String compactJwt="eyJhbGciOiJIUzI1NiJ9.eyJqdGkiOiI4ODgiLCJzdWIiOiLlsI_nmb0iLCJpYXQiOjE1NTc5MDQxODF9.ThecMfgYjtoys3JX7dpx3hu6pUm0piZ0tXXreFU_u3Y";
 Claims claims = Jwts.parser().setSigningKey("hahaha").parseClaimsJws(compactJwt).getBody();
 System.out.println(claims);

```

运行打印效果：
 {jti=888, sub=小白, iat=1557904181}</pre>

试着将token或签名秘钥篡改一下，会发现运行时就会报错，所以解析token也就是验证token.

#### 设置过期时间

有很多时候，我们并不希望签发的token是永久生效的，所以我们可以为token添加一个过期时间。

（1）创建token 并设置过期时间



```cpp
long now=System.currentTimeMillis();
long exp=now+1000*30;//30秒过期
JwtBuilder jwtBuilder = Jwts.builder().setId( "888" )
 .setSubject( "小白" )
 .setIssuedAt( new Date() )//签发时间
 .setExpiration( new Date( exp ) )//过期时间
 .signWith( SignatureAlgorithm.HS256, "hahaha" );
String token = jwtBuilder.compact();
System.out.println(token);

```

运行，打印效果如下：

eyJhbGciOiJIUzI1NiJ9.eyJqdGkiOiI4ODgiLCJzdWIiOiLlsI_nmb0iLCJpYXQiOjE1NTc5MDUzMDgsImV4cCI6MTU1NzkwNTMwOH0.4q5AaTyBRf8SB9B3Tl-I53PrILGyicJC3fgR3gWbvUI

（2）解析TOKEN



```csharp
 String compactJwt="eyJhbGciOiJIUzI1NiJ9.eyJqdGkiOiI4ODgiLCJzdWIiOiLlsI_nmb0iLCJpYXQiOjE1NTc5MDUzMDgsImV4cCI6MTU1NzkwNTMwOH0.4q5AaTyBRf8SB9B3Tl-I53PrILGyicJC3fgR3gWbvUI";
 Claims claims = Jwts.parser().setSigningKey("hahaha").parseClaimsJws(compactJwt).getBody();
 System.out.println(claims);

```

当前时间超过过期时间，则会报错。

#### 自定义claims

我们刚才的例子只是存储了id和subject两个信息，如果你想存储更多的信息（例如角色）可以定义自定义claims。



```cpp
long now=System.currentTimeMillis();
long exp=now+1000*30;//30秒过期
JwtBuilder jwtBuilder = Jwts.builder().setId( "888" )
 .setSubject( "小白" )
 .setIssuedAt( new Date() )//签发时间
 .setExpiration( new Date( exp ) )//过期时间
 .claim( "roles","admin" )
 .signWith( SignatureAlgorithm.HS256, "hahaha" );
String token = jwtBuilder.compact();
System.out.println(token);

```

运行打印效果：

eyJhbGciOiJIUzI1NiJ9.eyJqdGkiOiI4ODgiLCJzdWIiOiLlsI_nmb0iLCJpYXQiOjE1NTc5MDU4MDIsImV4cCI6MTU1NzkwNjgwMiwicm9sZXMiOiJhZG1pbiJ9.AS5Y2fNCwUzQQxXh_QQWMpaB75YqfuK-2P7VZiCXEJI

解析TOKEN:



```csharp
String token ="eyJhbGciOiJIUzI1NiJ9.eyJqdGkiOiI4ODgiLCJzdWIiOiLlsI_nmb0iLCJpYXQiOjE1NjIyNTM3NTQsImV4cCI6MTU2MjI1Mzc4Mywicm9sZXMiOiJhZG1pbiJ9.CY6CMembCi3mAkBHS3ivzB5w9uvtZim1HkizRu2gWaI";
Claims claims = Jwts.parser().setSigningKey( "hahaha" ).parseClaimsJws( token ).getBody();
System.out.println(claims);
System.out.println(claims.get( "roles" ));

```

前端页面js代码（客户端）：
       

```js
 //页面初始化向iframe域名发送消息
        window.onload = function() {
            console.log('get key value......................')
            window.frames[0].postMessage(JSON.stringify({type:"GET",key:"User-Token"}),'*');
        }
        //监听message信息，接收从iframe域下获取到的token信息，然后存储到localstorage或cookie中
        window.addEventListener('message', function(e) {
            console.log('listen.....');
            var data = e.data;
            console.log(data);
            if(data != null){
                localStorage.setItem("User-Token", data);
            }
        }, false);

```

总结：
优点：在非跨域环境下使用JWT机制是一个非常不错的选择，实现方式简单，操作方便，能够快速实现。由于服务端不存储用户状态信息，因此大用户量，对后台服务也不会造成压力；
缺点：跨域实现相对比较麻烦，安全性也有待探讨。因为JWT令牌返回到页面中，可以使用js获取到，如果遇到XSS攻击令牌可能会被盗取，在JWT还没超时的情况下，就会被获取到敏感数据信息。



## 本次商城项目中用到的JWT

首先需要依赖:

```xml
<dependency>
    <groupId>com.auth0</groupId>
    <artifactId>java-jwt</artifactId>
    <version>3.4.1</version>
</dependency>

```

首先有一个JWT工具类:

```java
package org.linlinjava.litemall.wx.util;

import java.util.Calendar;
import java.util.Date;
import java.util.GregorianCalendar;
import java.util.HashMap;
import java.util.Map;

import com.auth0.jwt.JWT;
import com.auth0.jwt.JWTVerifier;
import com.auth0.jwt.algorithms.Algorithm;
import com.auth0.jwt.exceptions.JWTCreationException;
import com.auth0.jwt.exceptions.JWTVerificationException;
import com.auth0.jwt.interfaces.Claim;
import com.auth0.jwt.interfaces.DecodedJWT;

public class JwtHelper {
	// 秘钥
	static final String SECRET = "X-Litemall-Token";
	// 签名是有谁生成
	static final String ISSUSER = "LITEMALL";
	// 签名的主题
	static final String SUBJECT = "this is litemall token";
	// 签名的观众
	static final String AUDIENCE = "MINIAPP";
	
	
	public String createToken(Integer userId){
		try {
		    Algorithm algorithm = Algorithm.HMAC256(SECRET);
		    Map<String, Object> map = new HashMap<String, Object>();
		    Date nowDate = new Date();
		    // 过期时间：2小时
		    Date expireDate = getAfterDate(nowDate,0,0,0,2,0,0);
	        map.put("alg", "HS256");
	        map.put("typ", "JWT");
		    String token = JWT.create()
		    	// 设置头部信息 Header
		    	.withHeader(map)
		    	// 设置 载荷 Payload
		    	.withClaim("userId", userId)
		        .withIssuer(ISSUSER)
		        .withSubject(SUBJECT)
		        .withAudience(AUDIENCE)
		        // 生成签名的时间 
		        .withIssuedAt(nowDate)
		        // 签名过期的时间 
		        .withExpiresAt(expireDate)
		        // 签名 Signature
		        .sign(algorithm);
		    return token;
		} catch (JWTCreationException exception){
			exception.printStackTrace();
		}
		return null;
	}
	
	public Integer verifyTokenAndGetUserId(String token) {
		try {
		    Algorithm algorithm = Algorithm.HMAC256(SECRET);
		    JWTVerifier verifier = JWT.require(algorithm)
		        .withIssuer(ISSUSER)
		        .build();
		    DecodedJWT jwt = verifier.verify(token);
		    Map<String, Claim> claims = jwt.getClaims();
		    Claim claim = claims.get("userId");
		    return claim.asInt();
		} catch (JWTVerificationException exception){
//			exception.printStackTrace();
		}
		
		return 0;
	}
	
	public  Date getAfterDate(Date date, int year, int month, int day, int hour, int minute, int second){
		if(date == null){
			date = new Date();
		}
		
		Calendar cal = new GregorianCalendar();
		
		cal.setTime(date);
		if(year != 0){
			cal.add(Calendar.YEAR, year);
		}
		if(month != 0){
			cal.add(Calendar.MONTH, month);
		}
		if(day != 0){
			cal.add(Calendar.DATE, day);
		}
		if(hour != 0){
			cal.add(Calendar.HOUR_OF_DAY, hour);
		}
		if(minute != 0){
			cal.add(Calendar.MINUTE, minute);
		}
		if(second != 0){
			cal.add(Calendar.SECOND, second);
		}
		return cal.getTime();
	}
}


```

一个维护用户token的Manager

```java
package org.linlinjava.litemall.wx.service;

import org.linlinjava.litemall.wx.util.JwtHelper;

/**
 * 维护用户token
 */
public class UserTokenManager {
	public static String generateToken(Integer id) {
        JwtHelper jwtHelper = new JwtHelper();
        return jwtHelper.createToken(id);
    }
    public static Integer getUserId(String token) {
    	JwtHelper jwtHelper = new JwtHelper();
    	Integer userId = jwtHelper.verifyTokenAndGetUserId(token);
    	if(userId == null || userId == 0){
    		return null;
    	}
        return userId;
    }
}

```



为了配合项目可以快速取得用户userId

项目中自定义了一个解析器:

```java
package org.linlinjava.litemall.wx.annotation.support;

import org.linlinjava.litemall.wx.annotation.LoginUser;
import org.linlinjava.litemall.wx.service.UserTokenManager;
import org.springframework.core.MethodParameter;
import org.springframework.web.bind.support.WebDataBinderFactory;
import org.springframework.web.context.request.NativeWebRequest;
import org.springframework.web.method.support.HandlerMethodArgumentResolver;
import org.springframework.web.method.support.ModelAndViewContainer;


public class LoginUserHandlerMethodArgumentResolver implements HandlerMethodArgumentResolver {
    public static final String LOGIN_TOKEN_KEY = "X-Litemall-Token";

    @Override
    public boolean supportsParameter(MethodParameter parameter) {
        return parameter.getParameterType().isAssignableFrom(Integer.class) && parameter.hasParameterAnnotation(LoginUser.class);
    }

    @Override
    public Object resolveArgument(MethodParameter parameter, ModelAndViewContainer container,
                                  NativeWebRequest request, WebDataBinderFactory factory) throws Exception {

        //return new Integer(1);
        String token = request.getHeader(LOGIN_TOKEN_KEY);  //从这里拿到token 
        if (token == null || token.isEmpty()) {
            return null;
        }

        return UserTokenManager.getUserId(token);   //利用Manager工具从这里拿到userId
    }
}

```

**首先从request中拿到token,然后通过Manager工具从这里拿到用户id**



**在自定义解析器中,给@LoginUser这个注解增加了新的意义,就是以后那个方法想要userId,只需要通过@LoginUser就可以获得userId**

![image-20191220111745764](../media/pictures/JWT.assets/image-20191220111745764.png)

就像这样:

![image-20191220112026743](../media/pictures/JWT.assets/image-20191220112026743.png)

即使前端没有传userId,这里加一个注解,就可以获取到userId



# 知识拓展:

**HandlerMethodArgumentResolver**

这个是自定义解析器:

## HandlerMethodArgumentResolver-自定参数解析器



## 1. 前置

理解Springmvc自带的一些实现，我们可以根据业务进行接口重写



```dart
- ServletRequestMethodArgumentResolver和ServletResponseMethodArgumentResolver处理了自动绑定HttpServletRequest和HttpServletResponse
- RequestParamMapMethodArgumentResolver处理了@RequestParam
- RequestHeaderMapMethodArgumentResolver处理了@RequestHeader
- PathVariableMapMethodArgumentResolver处理了@PathVariable
- ModelAttributeMethodProcessor处理了@ModelAttribute
RequestResponseBodyMethodProcessor处理了@RequestBody

```

## 2. 方法分析

自定义解析器需要实现**HandlerMethodArgumentResolver**接口，**HandlerMethodArgumentResolver**接口包含两个接口：



```java
package org.springframework.web.method.support;

import org.springframework.core.MethodParameter;
import org.springframework.web.bind.WebDataBinder;
import org.springframework.web.bind.support.WebDataBinderFactory;
import org.springframework.web.context.request.NativeWebRequest;

/**
 * Strategy interface for resolving method parameters into argument values in
 * the context of a given request.
 *
 * @author Arjen Poutsma
 * @since 3.1
 * @see HandlerMethodReturnValueHandler
 */
public interface HandlerMethodArgumentResolver {

    boolean supportsParameter(MethodParameter parameter);
    Object resolveArgument(MethodParameter parameter, ModelAndViewContainer mavContainer,
            NativeWebRequest webRequest, WebDataBinderFactory binderFactory) throws Exception;

}

```

### 2.1 接口说明：

> **supportsParameter**：用于判定是否需要处理该参数分解，返回true为需要，并会去调用下面的方法**resolveArgument**。
> **resolveArgument**：真正用于处理参数分解的方法，返回的Object就是controller方法上的形参对象。

## 3. 使用方法

### 3.1 自定义注解



```css
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

/**
 * 登录用户信息
 */
@Target(ElementType.PARAMETER)
@Retention(RetentionPolicy.RUNTIME)
public @interface LoginUser {

}

```

### 3.2 自定义参数解析器，实现HandlerMethodArgumentResolver接口



```kotlin
package io.renren.modules.app.resolver;

import io.renren.modules.app.annotation.LoginUser;
import io.renren.modules.app.entity.UserEntity;
import io.renren.modules.app.interceptor.AuthorizationInterceptor;
import io.renren.modules.app.service.UserService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.core.MethodParameter;
import org.springframework.stereotype.Component;
import org.springframework.web.bind.support.WebDataBinderFactory;
import org.springframework.web.context.request.NativeWebRequest;
import org.springframework.web.context.request.RequestAttributes;
import org.springframework.web.method.support.HandlerMethodArgumentResolver;
import org.springframework.web.method.support.ModelAndViewContainer;

/**
 * 有@LoginUser注解的方法参数，注入当前登录用户
 */
@Component
public class LoginUserHandlerMethodArgumentResolver implements HandlerMethodArgumentResolver {
    @Autowired
    private UserService userService;

    @Override
    public boolean supportsParameter(MethodParameter parameter) {
        return parameter.getParameterType().isAssignableFrom(UserEntity.class) && parameter.hasParameterAnnotation(LoginUser.class);
    }

    @Override
    public Object resolveArgument(MethodParameter parameter, ModelAndViewContainer container,
                                  NativeWebRequest request, WebDataBinderFactory factory) throws Exception {
        //获取用户ID
        Object object = request.getAttribute(AuthorizationInterceptor.USER_KEY, RequestAttributes.SCOPE_REQUEST);
        if(object == null){
            return null;
        }
        //获取用户信息
        UserEntity user = userService.selectById((Long)object);
        return user;
    }
}

```

### 3.3 注册自定义参数解析器

Springboot中使用方法：



```java
import java.util.List;

/**
 * MVC配置
 */
@Configuration
public class WebMvcConfig extends WebMvcConfigurerAdapter {
    @Autowired
    private AuthorizationInterceptor authorizationInterceptor;
    @Autowired
    private LoginUserHandlerMethodArgumentResolver loginUserHandlerMethodArgumentResolver;

    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(authorizationInterceptor).addPathPatterns("/api/**");
    }

    @Override
    public void addArgumentResolvers(List<HandlerMethodArgumentResolver> argumentResolvers) {
        argumentResolvers.add(loginUserHandlerMethodArgumentResolver);
    }
}

```

### 3.4 Controller使用自定义注解



```kotlin
    @GetMapping("info")
    public UserEntity userInfo(@LoginUser UserEntity user){
        return user;
    }

```

## 4. 总结

**多读Spring全家桶源码，多了解，多理解，多实践！！！**



## springboot自定义参数解析HandlerMethodArgumentResolver

自定义解析器需要实现`HandlerMethodArgumentResolver`接口，`HandlerMethodArgumentResolver`接口包含两个接口函数：



```java
public interface HandlerMethodArgumentResolver {
    boolean supportsParameter(MethodParameter var1);

    @Nullable
    Object resolveArgument(MethodParameter var1, @Nullable ModelAndViewContainer var2, NativeWebRequest var3, @Nullable WebDataBinderFactory var4) throws Exception;
}

```

### 自定义一个解析器`CurrentUserMethodArgumentResolver`

我们在解析器中返回一个固定的UserBean`new UserBean(1L,"admin")`，实际情况是从Session、数据库或者缓存中查。



```kotlin
import org.springframework.core.MethodParameter;
import org.springframework.web.bind.support.WebDataBinderFactory;
import org.springframework.web.context.request.NativeWebRequest;
import org.springframework.web.method.support.HandlerMethodArgumentResolver;
import org.springframework.web.method.support.ModelAndViewContainer;

/**
 * 用于绑定@CurrentUser的方法参数解析器
 *
 * @author lism
 */
public class CurrentUserMethodArgumentResolver implements HandlerMethodArgumentResolver {

    public CurrentUserMethodArgumentResolver() {
    }

    @Override
    public boolean supportsParameter(MethodParameter parameter) {
        if (parameter.getParameterType().isAssignableFrom(UserBean.class) && parameter.hasParameterAnnotation(CurrentUser.class)) {
            return true;
        }
        return false;
    }

    @Override
    public Object resolveArgument(MethodParameter parameter, ModelAndViewContainer mavContainer, NativeWebRequest webRequest, WebDataBinderFactory binderFactory) throws Exception {
        CurrentUser currentUserAnnotation = parameter.getParameterAnnotation(CurrentUser.class);
        //从Session 获取用户
        Object object = webRequest.getAttribute(currentUserAnnotation.value(), NativeWebRequest.SCOPE_SESSION);
//从  accessToken获得用户信息
       if (object == null) {
            String token = webRequest.getHeader("Authorization");
            if (token == null) {
                token = webRequest.getParameter("accessToken");
            }
            //为了测试先写死用户名
            //TODO: 取真实用户
            return new UserBean(1L,"admin");
        }
        return object;
    }
}

```

### 自定义注解`@CurrentUser`



```kotlin
import java.lang.annotation.*;

/**
 * <p>绑定当前登录的用户</p>
 * <p>不同于@ModelAttribute</p>
 *
 * @author lism
 */
@Target({ElementType.PARAMETER})
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface CurrentUser {

    /**
     * 当前用户在request中的名字
     *
     * @return
     */
    String value() default "user";

}

```

### 在控制器中使用`@CurrentUser`

在控制器方法上加入`@CurrentUser UserBean userBean`即可自动注入userBean的值



```kotlin
@RestController
@RequestMapping(value = "/test")
public class TestController  {

    /**
     * 根据name查询
     *
     * @param request
     * @return
     */
    @RequestMapping(value = "/testCurrentUser", method = RequestMethod.POST, produces = "application/json", consumes = "application/json")
    @ResponseBody
    public void test(@CurrentUser UserBean userBean, @RequestBody SubjectRequest request) {
        String createdBy = userBean.getUsername();
        log.info(createdBy);
    }
}

```

#### User实体`UserBean`



```java
import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;

import java.io.Serializable;

@Data
@NoArgsConstructor
@AllArgsConstructor
public class UserBean implements Serializable {
    private Long id;
    private String username;
}

```

## 总结

我们可以通过实现`HandlerMethodArgumentResolver`接口来实现对自定义的参数进行解析。
比如可以解析自定义的时间格式、自定义解析Map对象等这些spring原本不支持的对象格式。





# Interview

## 1.JWT是什么，具体是怎么样做到的？



## 2.我看到你使用了JWT，是在什么情况下使用的，使用流程是什么？



## 3.你们的JWT里面存了什么？(用户的信息)，那你们拿到用户id了之后还要走一遍mysql才能检查出用户的信息，这样不是很耗时吗？



## 4.单点登录相关

### Jwt单点登录

-  用户携带用户名和密码请求访问

-  服务器校验用户凭据

- 应用提供一个token给客户端

- 客户端存储token，并且在随后的每一次请求中都带着它

-  服务器校验token并返回数据



### 什么叫单点登录？如何实现的单点登录？

### 讲讲单点登录怎么实现，怎么理解单点登录的意思



## 5.JWT的权限控制原理？JWT的实现原理？



## 6.讲一下项目中如何使用jwt的？

用户模块是用JWT来做，具体介绍blahblah

假设token被盗了，每次利用token去请求不就攻破了？（拿着用户和密码去登录网站我怎么判断你是不是我的用户？我佛了）



jwt payload存的是什么信息



## 7.说一下JWT的具体流程



## 6.权限是怎么控制的？



## 7.如何处理jwt的有效期问题（在后端jwt已经无效了，但是jwt还带有一些消息，过期jwt传过来后端怎么处理）





## 8.使用jwt的优势在于哪里？jwt带有很大的数据量，有些操作可能不需要传那么大的数据量，怎么处理?





## 9.如果传来了错误的token无法解析，呈现给客户是什么样的





## 10.前端的权限拦截依靠什么做的？





## 11.权限控制是服务器处理后下发的还是纯前端控制的？





## 12.如果有个需求针对用户首次登陆后需要修改密码？在数据库，在功能上如何进行加强



## 13.用户登录，如何验证数据是否被篡改，以及数据的完整性？



## 14.Session 共享问题

用户信息中，你们的session共享是怎么解决的？(我说了jwt)

为什么要做session共享这个问题？

如果不使用jwt，又怎么解决session共享的问题？（说了每台服务器都保存session信息，然后他问还要什么方法？。。。。然后说了一部分保存在A服务器，一部分保存在B服务器，然后他问怎么确保下一次登录的还在A？）





































































