# 微服务应用

前面我们已经完成了SpringCloudAlibaba的学习，我们对一个微服务项目的架构体系已经有了一定的了解，那么本章我们将在应用层面继续探讨微服务。

## 分布式权限校验

虽然完成前面的部分，我们已经可以自己去编写一个比较中规中矩的微服务项目了，但是还有一个问题我们没有解决，登录问题。假如现在要求用户登录之后，才能进行图书的查询、借阅等操作，那么我们又该如何设计这个系统呢？

回顾我们之前进行权限校验的原理，服务器是如何判定一个请求是来自哪个用户的呢？

* 首先浏览器会向服务端发送请求，访问我们的网站。
* 服务端收到请求后，会创建一个SESSION ID，并暂时存储在服务端，然后会发送给浏览器作为Cookie保存。
* 之后浏览器会一直携带此Cookie访问服务器，这样在收到请求后，就能根据携带的Cookie中的SESSION ID判断是哪个用户了。
* 这样服务端和浏览器之间可以轻松地建立会话了。

但是我们想一下，我们现在采用的是分布式的系统，那么在用户服务进行登录之后，其他服务比如图书服务和借阅服务，它们会知道用户登录了吗？

![image-20230306234928969](https://s2.loli.net/2023/03/06/hV2JkERda4qKtjB.png)

实际上我们登录到用户服务之后，Session中的用户数据只会在用户服务的应用中保存，而在其他服务中，并没有对应的信息，但是我们现在希望的是，所有的服务都能够同步这些Session信息，这样我们才能实现在用户服务登录之后其他服务都能知道，那么我们该如何实现Session的同步呢？

1. 我们可以在每台服务器上都复制一份Session，但是这样显然是很浪费时间的，并且用户验证数据占用的内存会成倍的增加。
2. 将Session移出服务器，用统一存储来存放，比如我们可以直接在Redis或是MySQL中存放用户的Session信息，这样所有的服务器在需要获取Session信息时，统一访问Redis或是MySQL即可，这样就能保证所有服务都可以同步Session了（是不是越来越感觉只要有问题，没有什么是加一个中间件解决不了的）

![image-20230306234940672](https://s2.loli.net/2023/03/06/pqZolFN6eIPza52.png)

那么，我们就着重来研究一下，然后实现2号方案，这里我们就使用Redis作为Session统一存储，我们把一开始的压缩包重新解压一次，又来从头开始编写吧。

这里我们就只使用Nacos就行了，和之前一样，我们把Nacos的包导入一下，然后进行一些配置：

![image-20230306234948408](https://s2.loli.net/2023/03/06/FYcNvAuZ7z8rj2V.png)

现在我们需要为每个服务都添加验证机制，首先导入依赖：

```xml
<!--  SpringSession Redis支持  -->
<dependency>
    <groupId>org.springframework.session</groupId>
    <artifactId>spring-session-data-redis</artifactId>
</dependency>
<!--  添加Redis的Starter  -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```

然后我们依然使用SpringSecurity框架作为权限校验框架：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>
```

接着我们在每个服务都编写一下对应的配置文件：

```yaml
spring:
  session:
  	# 存储类型修改为redis
    store-type: redis
  redis:
  	# Redis服务器的信息，该咋写咋写
    host: 1.14.121.107
```

这样，默认情况下，每个服务的接口都会被SpringSecurity所保护，只有登录成功之后，才可以被访问。

我们来打开Nacos看看：

![image-20230306234959085](https://s2.loli.net/2023/03/06/SyCJXKgO3qGx8EL.png)

可以看到三个服务都正常注册了，接着我们去访问图书服务：

![image-20230306235007563](https://s2.loli.net/2023/03/06/gytqnZjTMvVEUm3.png)

可以看到，访问失败，直接把我们给重定向到登陆页面了，也就是说必须登陆之后才能访问，同样的方式去访问其他服务，也是一样的效果。

由于现在是统一Session存储，那么我们就可以在任意一个服务登录之后，其他服务都可以正常访问，现在我们在当前页面登录，登录之后可以看到图书服务能够正常访问了：

![image-20230306235015909](https://s2.loli.net/2023/03/06/xfV5oYGvc1jKqTM.png)

同时用户服务也能正常访问了：

![image-20230306235021694](https://s2.loli.net/2023/03/06/OH6wjLVreot4IiA.png)

我们可以查看一下Redis服务器中是不是存储了我们的Session信息：

![image-20230306235046117](https://s2.loli.net/2023/03/06/nNIkoXOAYuMH8aV.png)

虽然看起来好像确实没啥问题了，但是借阅服务炸了，我们来看看为什么：

![image-20230306235053840](https://s2.loli.net/2023/03/06/wls5vCajnuMBOkU.png)

在RestTemplate进行远程调用的时候，由于我们的请求没有携带对应SESSION的Cookie，所以导致验证失败，访问不成功，返回401，所以虽然这种方案看起来比较合理，但是在我们的实际使用中，还是存在一些不便的。

***

## OAuth 2.0 实现单点登录

**注意：**第一次接触可能会比较难，不太好理解，需要多实践和观察。

前面我们虽然使用了统一存储来解决Session共享问题，但是我们发现就算实现了Session共享，依然存在一些问题，由于我们每个服务都有自己的验证模块，实际上整个系统是存在冗余功能的、同时还有我们上面出现的问题，那么能否实现只在一个服务进行登录，就可以访问其他的服务呢？

![image-20230306235102514](https://s2.loli.net/2023/03/06/46ukOAiDzMZBX15.png)

实际上之前的登录模式称为多点登录，而我们希望的是实现单点登陆，因此，我们得找一个更好的解决方案。

这里我们首先需要了解一种全新的登录方式：**OAuth 2.0**，我们经常看到一些网站支持第三方登录，比如淘宝、咸鱼我们就可以使用支付宝进行登录，腾讯游戏可以用QQ或是微信登陆，以及微信小程序都可以直接使用微信进行登录。我们知道它们并不是属于同一个系统，比如淘宝和咸鱼都不属于支付宝这个应用，但是由于需要获取支付宝的用户信息，这时我们就需要使用 OAuth2.0 来实现第三方授权，基于第三方应用访问用户信息的权限（本质上就是给别人调用自己服务接口的权限)，那么它是如何实现的呢？

### 四种授权模式

我们还是从理论开始讲解，OAuth 2.0一共有四种授权模式：

1. **客户端模式（Client Credentials）**

   这是最简单的一种模式，我们可以直接向验证服务器请求一个Token（这里可能有些小伙伴对Token的概念不是很熟悉，Token相当于是一个令牌，我们需要在验证服务器**（User Account And Authentication）**服务拿到令牌之后，才能去访问资源，比如用户信息、借阅信息等，这样资源服务器才能知道我们是谁以及是否成功登录了）

   当然，这里的前端页面只是一个例子，它还可以是其他任何类型的**客户端**，比如App、小程序甚至是第三方应用的服务。

   ![image-20230306235141113](https://s2.loli.net/2023/03/06/4i16wzqOnYeaB2c.png)

   虽然这种模式比较简便，但是已经失去了用户验证的意义，压根就不是给用户校验准备的，而是更适用于服务内部调用的场景。

2. **密码模式（Resource Owner Password Credentials）**

   密码模式相比客户端模式，就多了用户名和密码的信息，用户需要提供对应账号的用户名和密码，才能获取到Token。

   ![image-20230306235151635](https://s2.loli.net/2023/03/06/JEreS9nQD8ojMca.png)

   虽然这样看起来比较合理，但是会直接将账号和密码泄露给客户端，需要后台完全信任客户端不会拿账号密码去干其他坏事，所以这也不是我们常见的。

3. **隐式授权模式（Implicit Grant）**

   首先用户访问页面时，会重定向到认证服务器，接着认证服务器给用户一个认证页面，等待用户授权，用户填写信息完成授权后，认证服务器返回Token。

   ![image-20230306235200365](https://s2.loli.net/2023/03/06/MRxnKyWT3br5Zj2.png)

   它适用于没有服务端的第三方应用页面，并且相比前面一种形式，验证都是在验证服务器进行的，敏感信息不会轻易泄露，但是Token依然存在泄露的风险。

4. **授权码模式（Authrization Code）**

   这种模式是最安全的一种模式，也是推荐使用的一种，比如我们手机上的很多App都是使用的这种模式。

   相比隐式授权模式，它并不会直接返回Token，而是返回授权码，真正的Token是通过应用服务器访问验证服务器获得的。在一开始的时候，应用服务器（客户端通过访问自己的应用服务器来进而访问其他服务）和验证服务器之间会共享一个`secret`，这个东西没有其他人知道，而验证服务器在用户验证完成之后，会返回一个授权码，应用服务器最后将授权码和`secret`一起交给验证服务器进行验证，并且Token也是在服务端之间传递，不会直接给到客户端。

   ![image-20230306235211335](https://s2.loli.net/2023/03/06/2EIPfirBOKbcndk.png)

   这样就算有人中途窃取了授权码，也毫无意义，因为，Token的获取必须同时携带授权码和secret，但是`secret`第三方是无法得知的，并且Token不会直接丢给客户端，大大减少了泄露的风险。

但是乍一看，OAuth 2.0不应该是那种第三方应用为了请求我们的服务而使用的吗，而我们这里需要的只是实现同一个应用内部服务之间的认证，其实我也可以利用 OAuth2.0 来实现单点登录，只是少了资源服务器这一角色，客户端就是我们的整个系统，接下来就让我们来实现一下。

### 搭建验证服务器

第一步就是最重要的，我们需要搭建一个验证服务器，它是我们进行权限校验的核心，验证服务器有很多的第三方实现也有Spring官方提供的实现，这里我们使用Spring官方提供的验证服务器。

这里我们将最开始保存好的项目解压，就重新创建一个新的项目，首先我们在父项目中添加最新的SpringCloud依赖：

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-dependencies</artifactId>
    <version>2021.0.1</version>
    <type>pom</type>
    <scope>import</scope>
</dependency>
```

接着创建一个新的模块`auth-service`，添加依赖：

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>

    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-security</artifactId>
    </dependency>
    
    <!--  OAuth2.0依赖，不再内置了，所以得我们自己指定一下版本  -->
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-oauth2</artifactId>
        <version>2.2.5.RELEASE</version>
    </dependency>
</dependencies>
```

接着我们修改一下配置文件：

```yaml
server:
  port: 8500
  servlet:
  	#为了防止一会在服务之间跳转导致Cookie打架（因为所有服务地址都是localhost，都会存JSESSIONID）
  	#这里修改一下context-path，这样保存的Cookie会使用指定的路径，就不会和其他服务打架了
  	#但是注意之后的请求都得在最前面加上这个路径
    context-path: /sso
```

接着我们需要编写一下配置类，这里需要两个配置类，一个是OAuth2的配置类，还有一个是SpringSecurity的配置类：

```java
@Configuration
public class SecurityConfiguration extends WebSecurityConfigurerAdapter {

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
                .authorizeRequests()
                .anyRequest().authenticated()  //
                .and()
                .formLogin().permitAll();    //使用表单登录
    }
  
    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        BCryptPasswordEncoder encoder = new BCryptPasswordEncoder();
        auth
                .inMemoryAuthentication()   //直接创建一个用户，懒得搞数据库了
                .passwordEncoder(encoder)
                .withUser("test").password(encoder.encode("123456")).roles("USER");
    }
  
  	@Bean   //这里需要将AuthenticationManager注册为Bean，在OAuth配置中使用
    @Override
    public AuthenticationManager authenticationManagerBean() throws Exception {
        return super.authenticationManagerBean();
    }
}
```

```java
@EnableAuthorizationServer   //开启验证服务器
@Configuration
public class OAuth2Configuration extends AuthorizationServerConfigurerAdapter {

    @Resource
    private AuthenticationManager manager;

    private final BCryptPasswordEncoder encoder = new BCryptPasswordEncoder();

    /**
     * 这个方法是对客户端进行配置，一个验证服务器可以预设很多个客户端，
     * 之后这些指定的客户端就可以按照下面指定的方式进行验证
     * @param clients 客户端配置工具
     */
    @Override
    public void configure(ClientDetailsServiceConfigurer clients) throws Exception {
        clients
                .inMemory()   //这里我们直接硬编码创建，当然也可以像Security那样自定义或是使用JDBC从数据库读取
                .withClient("web")   //客户端名称，随便起就行
                .secret(encoder.encode("654321"))      //只与客户端分享的secret，随便写，但是注意要加密
                .autoApprove(false)    //自动审批，这里关闭，要的就是一会体验那种感觉
                .scopes("book", "user", "borrow")     //授权范围，这里我们使用全部all
                .authorizedGrantTypes("client_credentials", "password", "implicit", "authorization_code", "refresh_token");
                //授权模式，一共支持5种，除了之前我们介绍的四种之外，还有一个刷新Token的模式
                //这里我们直接把五种都写上，方便一会实验，当然各位也可以单独只写一种一个一个进行测试
                //现在我们指定的客户端就支持这五种类型的授权方式了
    }

    @Override
    public void configure(AuthorizationServerSecurityConfigurer security) {
        security
                .passwordEncoder(encoder)    //编码器设定为BCryptPasswordEncoder
                .allowFormAuthenticationForClients()  //允许客户端使用表单验证，一会我们POST请求中会携带表单信息
                .checkTokenAccess("permitAll()");     //允许所有的Token查询请求
    }

    @Override
    public void configure(AuthorizationServerEndpointsConfigurer endpoints) {
        endpoints
                .authenticationManager(manager);
        //由于SpringSecurity新版本的一些底层改动，这里需要配置一下authenticationManager，才能正常使用password模式
    }
}
```

接着我们就可以启动服务器了：

![image-20230306235226961](https://s2.loli.net/2023/03/06/2FhnOKe1BorP5NE.png)

然后我们使用Postman进行接口测试，首先我们从最简单的客户端模式进行测试，客户端模式只需要提供id和secret即可直接拿到Token，注意需要再添加一个grant_type来表明我们的授权方式，默认请求路径为http://localhost:8500/sso/oauth/token：

![image-20230306235236376](https://s2.loli.net/2023/03/06/X81T7mz5gQK3iBk.png)

发起请求后，可以看到我们得到了Token，它是以JSON格式给到我们的：

![image-20230306235248456](https://s2.loli.net/2023/03/06/84IKgq2xdvBeLTm.png)

我们还可以访问 http://localhost:8500/sso/oauth/check_token 来验证我们的Token是否有效：

![image-20230306235257995](https://s2.loli.net/2023/03/06/SXD8FjzZn7ev2B3.png)

![image-20230306235309409](https://s2.loli.net/2023/03/06/B9TzojnUq4KvVPr.png)

可以看到active为true，表示我们刚刚申请到的Token是有效的。

接着我们来测试一下第二种password模式，我们还需要提供具体的用户名和密码，授权模式定义为password即可：

![image-20230306235318785](https://s2.loli.net/2023/03/06/jt5XPZKvRFqr73x.png)

接着我们需要在请求头中添加Basic验证信息，这里我们直接填写id和secret即可：

![image-20230306235327745](https://s2.loli.net/2023/03/06/K9ZpIv8SzcfsHd4.png)

可以看到在请求头中自动生成了Basic验证相关内容：

![image-20230306235335919](https://s2.loli.net/2023/03/06/JHxPKgFU5wY7SB8.png)

![image-20230306235345388](https://s2.loli.net/2023/03/06/F3WU7XhqridywVn.png)

响应成功，得到Token信息，并且这里还多出了一个refresh_token，这是用于刷新Token的，我们之后会进行讲解。

![image-20230306235355181](https://s2.loli.net/2023/03/06/zjuc2qxQmBas5r1.png)

查询Token信息之后还可以看到登录的具体用户以及角色权限等。

接着我们来看隐式授权模式，这种模式我们需要在验证服务器上进行登录操作，而不是直接请求Token，验证登录请求地址：http://localhost:8500/sso/oauth/authorize?client_id=web&response_type=token

注意response_type一定要是token类型，这样才会直接返回Token，浏览器发起请求后，可以看到熟悉而又陌生的界面，没错，实际上这里就是使用我们之前讲解的SpringSecurity进行登陆，当然也可以配置一下记住我之类的功能，这里就不演示了：

![image-20230306235441824](https://s2.loli.net/2023/03/06/OYeRQpEXFSoZMhc.png)

但是登录之后我们发现出现了一个错误：

![image-20230306235501474](https://s2.loli.net/2023/03/06/qLUkJFZau8eQ6WO.png)

这是因为登录成功之后，验证服务器需要将结果给回客户端，所以需要提供客户端的回调地址，这样浏览器就会被重定向到指定的回调地址并且请求中会携带Token信息，这里我们随便配置一个回调地址：

```java
@Override
public void configure(ClientDetailsServiceConfigurer clients) throws Exception {
    clients
            .inMemory()
            .withClient("web")
            .secret(encoder.encode("654321"))
            .autoApprove(false)
            .scopes("book", "user", "borrow")
            .redirectUris("http://localhost:8201/login")   //可以写多个，当有多个时需要在验证请求中指定使用哪个地址进行回调
            .authorizedGrantTypes("client_credentials", "password", "implicit", "authorization_code", "refresh_token");
}
```

接着重启验证服务器，再次访问：

![image-20230306235513330](https://s2.loli.net/2023/03/06/PnTwQhlYXDgBvry.png)

可以看到这里会让我们选择哪些范围进行授权，就像我们在微信小程序中登陆一样，会让我们授予用户信息权限、支付权限、信用查询权限等，我们可以自由决定要不要给客户端授予访问这些资源的权限，这里我们全部选择授予：

![image-20230306235521432](https://s2.loli.net/2023/03/06/p7nMEVZIKjXWAl5.png)

授予之后，可以看到浏览器被重定向到我们刚刚指定的回调地址中，并且携带了Token信息，现在我们来校验一下看看：

![image-20230306235530288](https://s2.loli.net/2023/03/06/g1JhS9WDfcz6QEK.png)

可以看到，Token也是有效的。

最后我们来看看第四种最安全的授权码模式，这种模式其实流程和上面是一样的，但是请求的是code类型：http://localhost:8500/sso/oauth/authorize?client_id=web&response_type=code

可以看到访问之后，依然会进入到回调地址，但是这时给的就是授权码了，而不是直接给Token，那么这个Token该怎么获取呢？

![image-20230306235536317](https://s2.loli.net/2023/03/06/da4WseDt172hbLV.png)

按照我们之前讲解的原理，我们需要携带授权码和secret一起请求，才能拿到Token，正常情况下是由回调的服务器进行处理，这里我们就在Postman中进行，我们复制刚刚得到的授权码，接口依然是`localhost:8500/sso/oauth/token`：

![image-20230306235545717](https://s2.loli.net/2023/03/06/e1Zdt9IP7vp2zMO.png)

可以看到结果也是正常返回了Token信息：

![image-20230306235554061](https://s2.loli.net/2023/03/06/qY5kxgBWSzMJXco.png)

这样我们四种最基本的Token请求方式就实现了。

最后还有一个是刷新令牌使用的，当我们的Token过期时，我们就可以使用这个refresh_token来申请一个新的Token：

![image-20230306235603731](https://s2.loli.net/2023/03/06/d2ojclCLB3mQu7D.png)

但是执行之后我们发现会直接出现一个内部错误：

![image-20230306235611413](https://s2.loli.net/2023/03/06/BcFMIg4NqCx8kdh.png)

![image-20230306235618838](https://s2.loli.net/2023/03/06/cA9WF1KxyUDZ8Bi.png)

查看日志发现，这里还需要我们单独配置一个UserDetailsService，我们直接把Security中的实例注册为Bean：

```java
@Bean
@Override
public UserDetailsService userDetailsServiceBean() throws Exception {
    return super.userDetailsServiceBean();
}
```

然后在Endpoint中设置：

```java
@Resource
UserDetailsService service;

@Override
public void configure(AuthorizationServerEndpointsConfigurer endpoints) {
    endpoints
            .userDetailsService(service)
            .authenticationManager(manager);
}
```

最后再次尝试刷新Token：

![image-20230306235638503](https://s2.loli.net/2023/03/06/QWEwzpiq7FXnv3f.png)

OK，成功刷新Token，返回了一个新的。

### 基于@EnableOAuth2Sso实现

前面我们将验证服务器已经搭建完成了，现在我们就来实现一下单点登陆吧，SpringCloud为我们提供了客户端的直接实现，我们只需要添加一个注解和少量配置即可将我们的服务作为一个单点登陆应用，使用的是第四种授权码模式。

一句话来说就是，这种模式只是将验证方式由原本的默认登录形式改变为了统一在授权服务器登陆的形式。

首先还是依赖：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>

<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-oauth2</artifactId>
    <version>2.2.5.RELEASE</version>
</dependency>
```

我们只需要直接在启动类上添加即可：

```java
@EnableOAuth2Sso
@SpringBootApplication
public class BookApplication {
    public static void main(String[] args) {
        SpringApplication.run(BookApplication.class, args);
    }
}
```

我们不需要进行额外的配置类，因为这个注解已经帮我们做了：

```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@EnableOAuth2Client
@EnableConfigurationProperties({OAuth2SsoProperties.class})
@Import({OAuth2SsoDefaultConfiguration.class, OAuth2SsoCustomConfiguration.class, ResourceServerTokenServicesConfiguration.class})
public @interface EnableOAuth2Sso {
}
```

可以看到它直接注册了OAuth2SsoDefaultConfiguration，而这个类就是帮助我们对Security进行配置的：

```java
@Configuration
@Conditional({NeedsWebSecurityCondition.class})
public class OAuth2SsoDefaultConfiguration extends WebSecurityConfigurerAdapter {
  	//直接继承的WebSecurityConfigurerAdapter，帮我们把验证设置都写好了
    private final ApplicationContext applicationContext;

    public OAuth2SsoDefaultConfiguration(ApplicationContext applicationContext) {
        this.applicationContext = applicationContext;
    }
```

接着我们需要在配置文件中配置我们的验证服务器相关信息：

```yaml
security:
  oauth2:
    client:
      #不多说了
      client-id: web
      client-secret: 654321
      #Token获取地址
      access-token-uri: http://localhost:8500/sso/oauth/token
      #验证页面地址
      user-authorization-uri: http://localhost:8500/sso/oauth/authorize
    resource:
      #Token信息获取和校验地址
      token-info-uri: http://localhost:8500/sso/oauth/check_token
```

现在我们就开启图书服务，调用图书接口：

![image-20230306235651433](https://s2.loli.net/2023/03/06/DrVSZtdKNCoMucx.png)

可以看到在发现没有登录验证时，会直接跳转到授权页面，进行授权登录，之后才可以继续访问图书服务：

![image-20230306235701725](https://s2.loli.net/2023/03/06/nsJGmxcOVYXDUqd.png)

那么用户信息呢？是否也一并保存过来了？我们这里直接获取一下SpringSecurity的Context查看用户信息，获取方式跟我们之前的视频中讲解的是一样的：

```java
@RequestMapping("/book/{bid}")
Book findBookById(@PathVariable("bid") int bid){
  	//通过SecurityContextHolder将用户信息取出
    SecurityContext context = SecurityContextHolder.getContext();
    System.out.println(context.getAuthentication());
    return service.getBookById(bid);
}
```

再次访问图书管理接口，可以看到：

![image-20230306235733777](https://s2.loli.net/2023/03/06/y1VYRC9tmOv854u.png)

这里使用的不是之前的UsernamePasswordAuthenticationToken也不是RememberMeAuthenticationToken，而是新的OAuth2Authentication，它保存了验证服务器的一些信息，以及经过我们之前的登陆流程之后，验证服务器发放给客户端的Token信息，并通过Token信息在验证服务器进行验证获取用户信息，最后保存到Session中，表示用户已验证，所以本质上还是要依赖浏览器存Cookie的。

接下来我们将所有的服务都使用这种方式进行验证，别忘了把重定向地址给所有服务都加上：

```java
@Override
public void configure(ClientDetailsServiceConfigurer clients) throws Exception {
    clients
            .inMemory()
            .withClient("web")
            .secret(encoder.encode("654321"))
            .autoApprove(true)   //这里把自动审批开了，就不用再去手动选同意了
            .scopes("book", "user", "borrow")
            .redirectUris("http://localhost:8101/login", "http://localhost:8201/login", "http://localhost:8301/login")
            .authorizedGrantTypes("client_credentials", "password", "implicit", "authorization_code", "refresh_token");
}
```

这样我们就可以实现只在验证服务器登陆，如果登陆过其他的服务都可以访问了。

但是我们发现一个问题，就是由于SESSION不同步，每次切换不同的服务进行访问都会重新导验证服务器去验证一次：

![image-20230306235744660](https://s2.loli.net/2023/03/06/7zbqlOrSCVRdQ4y.png)

这里有两个方案：

* 像之前一样做SESSION统一存储
* 设置context-path路径，每个服务单独设置，就不会打架了

但是这样依然没法解决服务间调用的问题，所以仅仅依靠单点登陆的模式不太行。

### 基于@EnableResourceServer实现

前面我们讲解了将我们的服务作为单点登陆应用直接实现单点登陆，那么现在我们如果是以第三方应用进行访问呢？这时我们就需要将我们的服务作为资源服务了，作为资源服务就不会再提供验证的过程，而是直接要求请求时携带Token，而验证过程我们这里就继续用Postman来完成，这才是我们常见的模式。

一句话来说，跟上面相比，我们只需要携带Token就能访问这些资源服务器了，客户端被独立了出来，用于携带Token去访问这些服务。

我们也只需要添加一个注解和少量配置即可：

```java
@EnableResourceServer
@SpringBootApplication
public class BookApplication {
    public static void main(String[] args) {
        SpringApplication.run(BookApplication.class, args);
    }
}
```

配置中只需要：

```yaml
security:
  oauth2:
    client:
    	#基操
      client-id: web
      client-secret: 654321
    resource:
    	#因为资源服务器得验证你的Token是否有访问此资源的权限以及用户信息，所以只需要一个验证地址
      token-info-uri: http://localhost:8500/sso/oauth/check_token
```

配置完成后，我们启动服务器，直接访问会发现：

![image-20230306235756676](https://s2.loli.net/2023/03/06/QiZmqznyMxNpETk.png)

这是由于我们的请求头中没有携带Token信息，现在有两种方式可以访问此资源：

* 在URL后面添加`access_token`请求参数，值为Token值
* 在请求头中添加`Authorization`，值为`Bearer +Token值`

我们先来试试看最简的一种：

![image-20230306235804196](https://s2.loli.net/2023/03/06/Np6PKCZD2kAdmtf.png)

另一种我们需要使用Postman来完成：

![image-20230306235814228](https://s2.loli.net/2023/03/06/ypR3G7DxsYicMQI.png)

添加验证信息后，会帮助我们转换成请求头信息：

![image-20230306235826416](https://s2.loli.net/2023/03/06/qPHDU1dXgC7srn3.png)

![image-20230306235833330](https://s2.loli.net/2023/03/06/6IeMvTcCKdfbUlV.png)

这样我们就将资源服务器搭建完成了。

我们接着来看如何对资源服务器进行深度自定义，我们可以为其编写一个配置类，比如我们现在希望用户授权了某个Scope才可以访问此服务：

```java
@Configuration
public class ResourceConfiguration extends ResourceServerConfigurerAdapter { //继承此类进行高度自定义

    @Override
    public void configure(HttpSecurity http) throws Exception {  //这里也有HttpSecurity对象，方便我们配置SpringSecurity
        http
                .authorizeRequests()
                .anyRequest().access("#oauth2.hasScope('lbwnb')");  //添加自定义规则
      					//Token必须要有我们自定义scope授权才可以访问此资源
    }
}
```

可以看到当没有对应的scope授权时，那么会直接返回`insufficient_scope`错误：

![image-20230306235844783](https://s2.loli.net/2023/03/06/5T4d39YkcZIomvD.png)

不知道各位是否有发现，实际上资源服务器完全没有必要将Security的信息保存在Session中了，因为现在只需要将Token告诉资源服务器，那么资源服务器就可以联系验证服务器，得到用户信息，就不需要使用之前的Session存储机制了，所以你会发现HttpSession中没有**SPRING_SECURITY_CONTEXT**，现在Security信息都是通过连接资源服务器获取。

接着我们将所有的服务都

但是还有一个问题没有解决，我们在使用RestTemplate进行服务间的远程调用时，会得到以下错误：

![image-20230306235853397](https://s2.loli.net/2023/03/06/k3LmR9E7UBtVA5x.png)

实际上这是因为在服务调用时没有携带Token信息，我们得想个办法把用户传来的Token信息在进行远程调用时也携带上，因此，我们可以直接使用OAuth2RestTemplate，它会在请求其他服务时携带当前请求的Token信息。它继承自RestTemplate，这里我们直接定义一个Bean：

```java
@Configuration
public class WebConfiguration {

    @Resource
    OAuth2ClientContext context;

    @Bean
    public OAuth2RestTemplate restTemplate(){
        return new OAuth2RestTemplate(new ClientCredentialsResourceDetails(), context);
    }
}
```

接着我们直接替换掉之前的RestTemplate即可：

```java
@Service
public class BorrowServiceImpl implements BorrowService {

    @Resource
    BorrowMapper mapper;

    @Resource
    OAuth2RestTemplate template;

    @Override
    public UserBorrowDetail getUserBorrowDetailByUid(int uid) {
        List<Borrow> borrow = mapper.getBorrowsByUid(uid);
        User user = template.getForObject("http://localhost:8101/user/"+uid, User.class);
        //获取每一本书的详细信息
        List<Book> bookList = borrow
                .stream()
                .map(b -> template.getForObject("http://localhost:8201/book/"+b.getBid(), Book.class))
                .collect(Collectors.toList());
        return new UserBorrowDetail(user, bookList);
    }
}
```

可以看到服务成功调用了：

![image-20230306235903496](https://s2.loli.net/2023/03/06/mvKqyJk7P1FCQSl.png)

现在我们来将Nacos加入，并通过Feign实现远程调用。

依赖还是贴一下，不然找不到：

```xml
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-alibaba-dependencies</artifactId>
    <version>2021.0.1.0</version>
    <type>pom</type>
    <scope>import</scope>
</dependency>
```

```xml
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
</dependency>

<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-loadbalancer</artifactId>
</dependency>
```

所有服务都已经注册成功了：

![image-20230306235926214](https://s2.loli.net/2023/03/06/BqkomFVGK7wv64X.png)

接着我们配置一下借阅服务的负载均衡：

```java
@Configuration
public class WebConfiguration {

    @Resource
    OAuth2ClientContext context;

    @LoadBalanced   //和RestTemplate一样直接添加注解就行了
    @Bean
    public OAuth2RestTemplate restTemplate(){
        return new OAuth2RestTemplate(new ClientCredentialsResourceDetails(), context);
    }
}
```

![image-20230306235934751](https://s2.loli.net/2023/03/06/PZkS8GyU1jhpIrz.png)

现在我们来把它替换为Feign，老样子，两个客户端：

```java
@FeignClient("user-service")
public interface UserClient {
    
    @RequestMapping("/user/{uid}")
    User getUserById(@PathVariable("uid") int uid);
}
```

```java
@FeignClient("book-service")
public interface BookClient {

    @RequestMapping("/book/{bid}")
    Book getBookById(@PathVariable("bid") int bid);
}
```

但是配置完成之后，又出现刚刚的问题了，OpenFeign也没有携带Token进行访问：

![image-20230306235944272](https://s2.loli.net/2023/03/06/EzWuaAJgNLi3sdF.png)

那么怎么配置Feign携带Token访问呢？遇到这种问题直接去官方查：https://docs.spring.io/spring-cloud-openfeign/docs/current/reference/html/#oauth2-support，非常简单，两个配置就搞定：

```yaml
feign:
  oauth2:
  	#开启Oauth支持，这样就会在请求头中携带Token了
    enabled: true
    #同时开启负载均衡支持
    load-balanced: true
```

重启服务器，可以看到结果OK了：

![image-20230306235953555](https://s2.loli.net/2023/03/06/zgEJF7AeHw8iGIZ.png)

这样我们就成功将之前的三个服务作为资源服务器了，注意和我们上面的作为客户端是不同的，将服务直接作为客户端相当于只需要验证通过即可，并且还是要保存Session信息，相当于只是将登录流程换到统一的验证服务器上进行罢了。而将其作为资源服务器，那么就需要另外找客户端（可以是浏览器、小程序、App、第三方服务等）来访问，并且也是需要先进行验证然后再通过携带Token进行访问，这种模式是我们比较常见的模式。

### 使用jwt存储Token

官网：https://jwt.io

JSON Web Token令牌（JWT）是一个开放标准（[RFC 7519](https://tools.ietf.org/html/rfc7519)），它定义了一种紧凑和自成一体的方式，用于在各方之间作为JSON对象安全地传输信息。这些信息可以被验证和信任，因为它是数字签名的。JWT可以使用密钥（使用**HMAC**算法）或使用**RSA**或**ECDSA**进行公钥/私钥对进行签名。

实际上，我们之前都是携带Token向资源服务器发起请求后，资源服务器由于不知道我们Token的用户信息，所以需要向验证服务器询问此Token的认证信息，这样才能得到Token代表的用户信息，但是各位是否考虑过，如果每次用户请求都去查询用户信息，那么在大量请求下，验证服务器的压力可能会非常的大。而使用JWT之后，Token中会直接保存用户信息，这样资源服务器就不再需要询问验证服务器，自行就可以完成解析，我们的目标是不联系验证服务器就能直接完成验证。

JWT令牌的格式如下：

![image-20230307000004710](https://s2.loli.net/2023/03/07/Xu8lxYhKoJNr6it.png)

一个JWT令牌由3部分组成：标头(Header)、有效载荷(Payload)和签名(Signature)。在传输的时候，会将JWT的3部分分别进行Base64编码后用`.`进行连接形成最终需要传输的字符串。

* 标头：包含一些元数据信息，比如JWT签名所使用的加密算法，还有类型，这里统一都是JWT。
* 有效载荷：包括用户名称、令牌发布时间、过期时间、JWT ID等，当然我们也可以自定义添加字段，我们的用户信息一般都在这里存放。
* 签名：首先需要指定一个密钥，该密钥仅仅保存在服务器中，保证不能让其他用户知道。然后使用Header中指定的算法对Header和Payload进行base64加密之后的结果通过密钥计算哈希值，然后就得出一个签名哈希。这个会用于之后验证内容是否被篡改。

这里还是补充一下一些概念，因为很多东西都是我们之前没有接触过的：

* **Base64：**就是包括小写字母a-z、大写字母A-Z、数字0-9、符号"+"、"/"一共64个字符的字符集（末尾还有1个或多个`=`用来凑够字节数），任何的符号都可以转换成这个字符集中的字符，这个转换过程就叫做Base64编码，编码之后会生成只包含上述64个字符的字符串。相反，如果需要原本的内容，我们也可以进行Base64解码，回到原有的样子。

  ```java
  public void test(){
      String str = "你们可能不知道只用20万赢到578万是什么概念";
    	//Base64不只是可以对字符串进行编码，任何byte[]数据都可以，编码结果可以是byte[]，也可以是字符串
      String encodeStr = Base64.getEncoder().encodeToString(str.getBytes());
      System.out.println("Base64编码后的字符串："+encodeStr);
  
      System.out.println("解码后的字符串："+new String(Base64.getDecoder().decode(encodeStr)));
  }
  ```

  注意Base64不是加密算法，只是一种信息的编码方式而已。

* **加密算法：**加密算法分为对称加密和非对称加密，其中**对称加密（Symmetric Cryptography）**比较好理解，就像一把锁配了两把钥匙一样，这两把钥匙你和别人都有一把，然后你们直接传递数据，都会把数据用锁给锁上，就算传递的途中有人把数据窃取了，也没办法解密，因为钥匙只有你和对方有，没有钥匙无法进行解密，但是这样有个问题，既然解密的关键在于钥匙本身，那么如果有人不仅窃取了数据，而且对方那边的治安也不好，于是顺手就偷走了钥匙，那你们之间发的数据不就凉凉了吗。

  因此，**非对称加密（Asymmetric Cryptography）**算法出现了，它并不是直接生成一把钥匙，而是生成一个公钥和一个私钥，私钥只能由你保管，而公钥交给对方或是你要发送的任何人都行，现在你需要把数据传给对方，那么就需要使用私钥进行加密，但是，这个数据只能使用对应的公钥进行解密，相反，如果对方需要给你发送数据，那么就需要用公钥进行加密，而数据只能使用私钥进行解密，这样的话就算对方的公钥被窃取，那么别人发给你的数据也没办法解密出来，因为需要私钥才能解密，而只有你才有私钥。

  因此，非对称加密的安全性会更高一些，包括HTTPS的隐私信息正是使用非对称加密来保障传输数据的安全（当然HTTPS并不是单纯地使用非对称加密完成的，感兴趣的可以去了解一下）

  对称加密和非对称加密都有很多的算法，比如对称加密，就有：DES、IDEA、RC2，非对称加密有：RSA、DAS、ECC

* **不可逆加密算法：**常见的不可逆加密算法有MD5, HMAC, SHA-1, SHA-224, SHA-256, SHA-384, 和SHA-512, 其中SHA-224、SHA-256、SHA-384，和SHA-512我们可以统称为SHA2加密算法，SHA加密算法的安全性要比MD5更高，而SHA2加密算法比SHA1的要高，其中SHA后面的数字表示的是加密后的字符串长度，SHA1默认会产生一个160位的信息摘要。经过不可逆加密算法得到的加密结果，是无法解密回去的，也就是说加密出来是什么就是什么了。本质上，其就是一种哈希函数，用于对一段信息产生摘要，以**防止被篡改**。

  实际上这种算法就常常被用作信息摘要计算，同样的数据通过同样的算法计算得到的结果肯定也一样，而如果数据被修改，那么计算的结果肯定就不一样了。

这里我们就可以利用jwt，将我们的Token采用新的方式进行存储：

![image-20230307000016944](https://s2.loli.net/2023/03/07/W95CFKAmd1wfgSJ.png)

这里我们使用最简单的一种方式，对称密钥，我们需要对验证服务器进行一些修改：

```java
@Bean
public JwtAccessTokenConverter tokenConverter(){  //Token转换器，将其转换为JWT
    JwtAccessTokenConverter converter = new JwtAccessTokenConverter();
    converter.setSigningKey("lbwnb");   //这个是对称密钥，一会资源服务器那边也要指定为这个
    return converter;
}

@Bean
public TokenStore tokenStore(JwtAccessTokenConverter converter){  //Token存储方式现在改为JWT存储
    return new JwtTokenStore(converter);  //传入刚刚定义好的转换器
}
```

```java
@Resource
TokenStore store;

@Resource
JwtAccessTokenConverter converter;

private AuthorizationServerTokenServices serverTokenServices(){  //这里对AuthorizationServerTokenServices进行一下配置
    DefaultTokenServices services = new DefaultTokenServices();
    services.setSupportRefreshToken(true);   //允许Token刷新
    services.setTokenStore(store);   //添加刚刚的TokenStore
    services.setTokenEnhancer(converter);   //添加Token增强，其实就是JwtAccessTokenConverter，增强是添加一些自定义的数据到JWT中
    return services;
}

@Override
public void configure(AuthorizationServerEndpointsConfigurer endpoints) {
    endpoints
            .tokenServices(serverTokenServices())   //设定为刚刚配置好的AuthorizationServerTokenServices
            .userDetailsService(service)
            .authenticationManager(manager);
}
```

然后我们就可以重启验证服务器了：

![image-20230307000026772](https://s2.loli.net/2023/03/07/C6OteoFghrxpYjQ.png)

可以看到成功获取了AccessToken，但是这里的格式跟我们之前的格式就大不相同了，因为现在它是JWT令牌，我们可以对其进行一下Base64解码：

![image-20230307000035581](https://s2.loli.net/2023/03/07/CUMkrRfgOthZKVz.png)

可以看到所有的验证信息包含在内，现在我们对资源服务器进行配置：

```yaml
security:
  oauth2:
    resource:
      jwt:
        key-value: lbwnb #注意这里要跟验证服务器的密钥一致，这样算出来的签名才会一致
```

然后启动资源服务器，请求一下接口试试看：

![image-20230307000043443](https://s2.loli.net/2023/03/07/kOpRlTB7SPtQa4y.png)

请求成功，得到数据：

![image-20230307000053404](https://s2.loli.net/2023/03/07/aicW89KezTSZ7f5.png)

注意如果Token有误，那么会得到：

![image-20230307000101304](https://s2.loli.net/2023/03/07/4wFZx8kNY5WHnvy.png)

***

## Redis与分布式

在SpringBoot阶段，我们学习了Redis，它是一个基于内存的高性能数据库，我们当时已经学习了包括基本操作、常用数据类型、持久化、事务和锁机制以及使用Java与Redis进行交互等，利用它的高性能，我们还使用它来做Mybatis的二级缓存、以及Token的持久化存储。而这一部分，我们将继续深入，探讨Redis在分布式开发场景下的应用。

### 主从复制

在分布式场景下，我们可以考虑让Redis实现主从模式：

![image-20230307000109568](https://s2.loli.net/2023/03/07/bzwPflgBD5O1saN.png)

主从复制，是指将一台Redis服务器的数据，复制到其他的Redis服务器。前者称为主节点(Master)，后者称为从节点(Slave)，数据的复制是单向的，只能由主节点到从节点。Master以写为主，Slave 以读为主。

这样的好处肯定是显而易见的：

* 实现了读写分离，提高了性能。
* 在写少读多的场景下，我们甚至可以安排很多个从节点，这样就能够大幅度的分担压力，并且就算挂掉一个，其他的也能使用。

那么我们现在就来尝试实现一下，这里我们还是在Windows下进行测试，打开Redis文件夹，我们要开启两个Redis服务器，修改配置文件`redis.windows.conf`：

```conf
# Accept connections on the specified port, default is 6379 (IANA #815344).
# If port 0 is specified Redis will not listen on a TCP socket.
port 6001
```

一个服务器的端口设定为6001，复制一份，另一个的端口为6002，接着我们指定配置文件进行启动，打开cmd：

![image-20230307000143566](https://s2.loli.net/2023/03/07/Si54lok9eqtKPf1.png)

现在我们的两个服务器就启动成功了，接着我们可以使用命令查看当前服务器的主从状态，我们打开客户端：

![image-20230307000151282](https://s2.loli.net/2023/03/07/2TbMQeZknSOzFpy.png)

输入`info replication`命令来查看当前的主从状态，可以看到默认的角色为：master，也就是说所有的服务器在启动之后都是主节点的状态。那么现在我们希望让6002作为从节点，通过一个命令即可：

![image-20230307000200296](https://s2.loli.net/2023/03/07/XqpNcihJ5jsZRoI.png)

可以看到，在输入`replicaof 127.0.0.1 6001`命令后，就会将6001服务器作为主节点，而当前节点作为6001的从节点，并且角色也会变成：slave，接着我们来看看6001的情况：

![image-20230307000208687](https://s2.loli.net/2023/03/07/YABKJDsbQkE1UM5.png)

可以看到从节点信息中已经出现了6002服务器，也就是说现在我们的6001和6002就形成了主从关系（还包含一个偏移量，这个偏移量反应的是从节点的同步情况）

> 主服务器和从服务器都会维护一个复制偏移量，主服务器每次向从服务器中传递 N 个字节的时候，会将自己的复制偏移量加上 N。从服务器中收到主服务器的 N 个字节的数据，就会将自己额复制偏移量加上 N，通过主从服务器的偏移量对比可以很清楚的知道主从服务器的数据是否处于一致，如果不一致就需要进行增量同步了。

那么我们现在可以来测试一下，在主节点新增数据，看看是否会同步到从节点：

![image-20230307000216155](https://s2.loli.net/2023/03/07/taxoisA8Tpg2DWM.png)

可以看到，我们在6001服务器插入的`a`，可以在从节点6002读取到，那么，从节点新增的数据在主节点能得到吗？我们来测试一下：

![image-20230307000224291](https://s2.loli.net/2023/03/07/dS2V8xafPj6lKND.png)

可以看到，从节点压根就没办法进行数据插入，节点的模式为只读模式。那么如果我们现在不想让6002作为6001的从节点了呢？

![image-20230307000234718](https://s2.loli.net/2023/03/07/dV7Rxov6pblW2g5.png)

可以看到，通过输入`replicaof no one`，即可变回Master角色。接着我们再来启动一台6003服务器，流程是一样的：

![image-20230307000406674](https://s2.loli.net/2023/03/07/TC7z2mt3EGMPWfq.png)

可以看到，在连接之后，也会直接同步主节点的数据，因此无论是已经处于从节点状态还是刚刚启动完成的服务器，都会从主节点同步数据，实际上整个同步流程为：

1. 从节点执行replicaof ip port命令后，从节点会保存主节点相关的地址信息。
2. 从节点通过每秒运行的定时任务发现配置了新的主节点后，会尝试与该节点建立网络连接，专门用于接收主节点发送的复制命令。
3. 连接成功后，第一次会将主节点的数据进行全量复制，之后采用增量复制，持续将新来的写命令同步给从节点。

当我们的主节点关闭后，从节点依然可以读取数据：

![image-20230307000415001](https://s2.loli.net/2023/03/07/MmNshyQxa2ijSRT.png)

但是从节点会疯狂报错：

![image-20230307000424822](https://s2.loli.net/2023/03/07/pEIo93MQXShrsZD.png)

当然每次都去敲个命令配置主从太麻烦了，我们可以直接在配置文件中配置，添加这样行即可：

```
replicaof 127.0.0.1 6001
```

这里我们给6002和6003服务器都配置一下，现在我们重启三个服务器。

![image-20230307000432434](https://s2.loli.net/2023/03/07/GpAa5kfyC3zVRZK.png)

当然，除了作为Master节点的从节点外，我们还可以将其作为从节点的从节点，比如现在我们让6003作为6002的从节点：

![image-20230307000444990](https://s2.loli.net/2023/03/07/OdAs1weYgkDrQvf.png)

也就是说，现在差不多是这样的的一个情况：

![image-20230307000459161](https://s2.loli.net/2023/03/07/2ADSR8LtpMhCFfK.png)

采用这种方式，优点肯定是显而易见的，但是缺点也很明显，整个传播链路一旦中途出现问题，那么就会导致后面的从节点无法及时同步。

### 哨兵模式

前面我们讲解了Redis实现主从复制的一些基本操作，那么我们接着来看哨兵模式。

经过之前的学习，我们发现，实际上最关键的还是主节点，因为一旦主节点出现问题，那么整个主从系统将无法写入，因此，我们得想一个办法，处理一下主节点故障的情况。实际上我们可以参考之前的服务治理模式，比如Nacos和Eureka，所有的服务都会被实时监控，那么只要出现问题，肯定是可以及时发现的，并且能够采取响应的补救措施，这就是我们即将介绍的哨兵：

![image-20230307000508084](https://s2.loli.net/2023/03/07/YGq8MDZbRK6E7Po.png)

注意这里的哨兵不是我们之前学习SpringCloud Alibaba的那个，是专用于Redis的。哨兵会对所有的节点进行监控，如果发现主节点出现问题，那么会立即让从节点进行投票，选举一个新的主节点出来，这样就不会由于主节点的故障导致整个系统不可写（注意要实现这样的功能最小的系统必须是一主一从，再小的话就没有意义了）

![image-20230307000516875](https://s2.loli.net/2023/03/07/WhkUqfxcHn4CApP.png)

那么怎么启动一个哨兵呢？我们只需要稍微修改一下配置文件即可，这里直接删除全部内容，添加：

```
sentinel monitor lbwnb 127.0.0.1 6001 1
```

其中第一个和第二个是固定，第三个是为监控对象名称，随意，后面就是主节点的相关信息，包括IP地址和端口，最后一个1我们暂时先不说，然后我们使用此配置文件启动服务器，可以看到启动后：

![image-20230307000534399](https://s2.loli.net/2023/03/07/xB78t53RgykXvo9.png)

![image-20230307000542878](https://s2.loli.net/2023/03/07/STh2RgjW7ycPCNB.png)

可以看到以哨兵模式启动后，会自动监控主节点，然后还会显示那些节点是作为从节点存在的。

现在我们直接把主节点关闭，看看会发生什么事情：

![image-20230307000627807](https://s2.loli.net/2023/03/07/97HnwfuNjUK5qx4.png)

可以看到从节点还是正常的在报错，一开始的时候不会直接重新进行选举而是继续尝试重连（因为有可能只是网络小卡一下，没必要这么敏感），但是我们发现，经过一段时间之后，依然无法连接，哨兵输出了以下内容：

![image-20230307000646050](https://s2.loli.net/2023/03/07/GWt8Q6mfSv7TgCb.png)

可以看到哨兵发现主节点已经有一段时间不可用了，那么就会开始进行重新选举，6003节点被选为了新的主节点，并且之前的主节点6001变成了新的主节点的从节点：

![image-20230307000653822](https://s2.loli.net/2023/03/07/4WzTVZ15dMiQ3f8.png)

![image-20230307000703587](https://s2.loli.net/2023/03/07/gGHVvOhBKe9wSEz.png)

当我们再次启动6001时，会发现，它自动变成了6003的从节点，并且会将数据同步过来：

![image-20230307000717002](https://s2.loli.net/2023/03/07/eqLycu8s1rSRtFa.png)

那么，这个选举规则是怎样的呢？是在所有的从节点中随机选取还是遵循某种规则呢？

1. 首先会根据优先级进行选择，可以在配置文件中进行配置，添加`replica-priority`配置项（默认是100），越小表示优先级越高。
2. 如果优先级一样，那就选择偏移量最大的
3. 要是还选不出来，那就选择runid（启动时随机生成的）最小的。

要是哨兵也挂了咋办？没事，咱们可以多安排几个哨兵，只需要把哨兵的配置复制一下，然后修改端口，这样就可以同时启动多个哨兵了，我们启动3个哨兵（一主二从三哨兵），这里我们吧最后一个值改为`2`：

```
sentinel monitor lbwnb 192.168.0.8 6001 2
```

这个值实际上代表的是当有几个哨兵认为主节点挂掉时，就判断主节点真的挂掉了

![image-20230307000731297](https://s2.loli.net/2023/03/07/48MNiLJXqmUtvWc.png)

现在我们把6001节点挂掉，看看这三个哨兵会怎么样：

![image-20230307000741214](https://s2.loli.net/2023/03/07/ajSAhqb5L9Yuorg.png)

可以看到都显示将master切换为6002节点了。

那么，在哨兵重新选举新的主节点之后，我们Java中的Redis的客户端怎么感知到呢？我们来看看，首先还是导入依赖：

```xml
<dependencies>
    <dependency>
        <groupId>redis.clients</groupId>
        <artifactId>jedis</artifactId>
        <version>4.2.1</version>
    </dependency>
</dependencies>
```

```java
public class Main {
    public static void main(String[] args) {
        //这里我们直接使用JedisSentinelPool来获取Master节点
        //需要把三个哨兵的地址都填入
        try (JedisSentinelPool pool = new JedisSentinelPool("lbwnb",
                new HashSet<>(Arrays.asList("192.168.0.8:26741", "192.168.0.8:26740", "192.168.0.8:26739")))) {
            Jedis jedis = pool.getResource();   //直接询问并得到Jedis对象，这就是连接的Master节点
            jedis.set("test", "114514");    //直接写入即可，实际上就是向Master节点写入

            Jedis jedis2 = pool.getResource();   //再次获取
            System.out.println(jedis2.get("test"));   //读取操作
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

这样，Jedis对象就可以通过哨兵来获取，当Master节点更新后，也能得到最新的。

### 集群搭建

如果我们服务器的内存不够用了，但是现在我们的Redis又需要继续存储内容，那么这个时候就可以利用集群来实现扩容。

因为单机的内存容量最大就那么多，已经没办法再继续扩展了，但是现在又需要存储更多的内容，这时我们就可以让N台机器上的Redis来分别存储各个部分的数据（每个Redis可以存储1/N的数据量），这样就实现了容量的横向扩展。同时每台Redis还可以配一个从节点，这样就可以更好地保证数据的安全性。

![image-20230307000754039](https://s2.loli.net/2023/03/07/TjCw8DLi1VqYvpZ.png)

那么问题来，现在用户来了一个写入的请求，数据该写到哪个节点上呢？我们来研究一下集群的机制：

首先，一个Redis集群包含16384个插槽，集群中的每个Redis 实例负责维护一部分插槽以及插槽所映射的键值数据，那么这个插槽是什么意思呢？

实际上，插槽就是键的Hash计算后的一个结果，注意这里出现了`计算机网络`中的CRC循环冗余校验，这里采用CRC16，能得到16个bit位的数据，也就是说算出来之后结果是0-65535之间，再进行取模，得到最终结果：

**Redis key的路由计算公式：slot = CRC16（key） % 16384**

结果的值是多少，就应该存放到对应维护的Redis下，比如Redis节点1负责0-25565的插槽，而这时客户端插入了一个新的数据`a=10`，a在Hash计算后结果为666，那么a就应该存放到1号Redis节点中。简而言之，本质上就是通过哈希算法将插入的数据分摊到各个节点的，所以说哈希算法真的是处处都有用啊。

那么现在我们就来搭建一个简单的Redis集群，这里创建6个配置，注意开启集群模式：

```
# Normal Redis instances can't be part of a Redis Cluster; only nodes that are
# started as cluster nodes can. In order to start a Redis instance as a
# cluster node enable the cluster support uncommenting the following:
#
cluster-enabled yes
```

接着记得把所有的持久化文件全部删除，所有的节点内容必须是空的。

然后输入`redis-cli.exe --cluster create --cluster-replicas 1 127.0.0.1:6001 127.0.0.1:6002 127.0.0.1:6003 127.0.0.1:7001 127.0.0.1:7002 127.0.0.1:7003`，这里的`--cluster-replicas 1`指的是每个节点配一个从节点：

![image-20230307000805968](https://s2.loli.net/2023/03/07/7DikHoxKJPve9qa.png)

输入之后，会为你展示客户端默认分配的方案，并且会询问你当前的方案是否合理。可以看到6001/6002/6003都被选为主节点，其他的为从节点，我们直接输入yes即可：

![image-20230307000814076](https://s2.loli.net/2023/03/07/yxZhjouXB79qfDd.png)

最后分配成功，可以看到插槽的分配情况：

![image-20230307000830886](https://s2.loli.net/2023/03/07/8kg9TbadF2qyHJW.png)

现在我们随便连接一个节点，尝试插入一个值：

![image-20230307000840119](https://s2.loli.net/2023/03/07/YMf8DtlkCsqBpO1.png)

在插入时，出现了一个错误，实际上这就是因为a计算出来的哈希值（插槽），不归当前节点管，我们得去管这个插槽的节点执行，通过上面的分配情况，我们可以得到15495属于节点6003管理：

![image-20230307000849127](https://s2.loli.net/2023/03/07/EZmR2bLFudSskIf.png)

在6003节点插入成功，当然我们也可以使用集群方式连接，这样我们无论在哪个节点都可以插入，只需要添加`-c`表示以集群模式访问：

![image-20230307000907196](https://s2.loli.net/2023/03/07/mVw7EXFJQOcinIb.png)

可以看到，在6001节点成功对a的值进行了更新，只不过还是被重定向到了6003节点进行插入。

我们可以输入`cluster nodes`命令来查看当前所有节点的信息：

![image-20230307000914746](https://s2.loli.net/2023/03/07/pEJdI2UWTcNZFqu.png)

那么现在如果我们让某一个主节点挂掉会怎么样？现在我们把6001挂掉：

![image-20230307000922277](https://s2.loli.net/2023/03/07/zd6L3WosVE8JUqf.png)

可以看到原本的6001从节点7001，晋升为了新的主节点，而之前的6001已经挂了，现在我们将6001重启试试看：

![image-20230307000933812](https://s2.loli.net/2023/03/07/eUfYJS8yhVijvaw.png)

可以看到6001变成了7001的从节点，那么要是6001和7001都挂了呢？

![image-20230307000944827](https://s2.loli.net/2023/03/07/9W2BQtMXrUCnySV.png)

这时我们尝试插入新的数据：

![image-20230307000953377](https://s2.loli.net/2023/03/07/S8r5TE7gJ3M6iDW.png)

可以看到，当存在节点不可用时，会无法插入新的数据，现在我们将6001和7001恢复：

![image-20230307001001435](https://s2.loli.net/2023/03/07/2RL4GNqSWJXFuME.png)

可以看到恢复之后又可以继续正常使用了。

最后我们来看一下如何使用Java连接到集群模式下的Redis，我们需要用到JedisCluster对象：

```java
public class Main {
    public static void main(String[] args) {
        //和客户端一样，随便连一个就行，也可以多写几个，构造方法有很多种可以选择
        try(JedisCluster cluster = new JedisCluster(new HostAndPort("192.168.0.8", 6003))){
            System.out.println("集群实例数量："+cluster.getClusterNodes().size());
            cluster.set("a", "yyds");
            System.out.println(cluster.get("a"));
        }
    }
}
```

操作基本和Jedis对象一样，这里就不多做赘述了。

### 分布式锁

在我们的传统单体应用中，经常会用到锁机制，目的是为了防止多线程竞争导致的并发问题，但是现在我们在分布式环境下，又该如何实现锁机制呢？可能一条链路上有很多的应用，它们都是独立运行的，这时我们就可以借助Redis来实现分布式锁。

还记得我们上一章最后提出的问题吗？

```java
@Override
public boolean doBorrow(int uid, int bid) {
  	//1. 判断图书和用户是否都支持借阅，如果此时来了10个线程，都进来了，那么都能够判断为可以借阅
    if(bookClient.bookRemain(bid) < 1)
        throw new RuntimeException("图书数量不足");
    if(userClient.userRemain(uid) < 1)
        throw new RuntimeException("用户借阅量不足");
  	//2. 首先将图书的数量-1，由于上面10个线程同时进来，同时判断可以借阅，那么这个10个线程就同时将图书数量-1，那库存岂不是直接变成负数了？？？
    if(!bookClient.bookBorrow(bid))
        throw new RuntimeException("在借阅图书时出现错误！");
  	...
}
```

实际上在高并发下，我们看似正常的借阅流程，会出现问题，比如现在同时来了10个同学要借同一本书，但是现在只有3本，而我们的判断规则是，首先看书够不够，如果此时这10个请求都已经走到这里，并且都判定为可以进行借阅，那么问题就出现了，接下来这10个请求都开始进行借阅操作，导致库存直接爆表，形成超借问题（在电商系统中也存在同样的超卖问题）

因此，为了解决这种问题，我们就可以利用分布式锁来实现。那么Redis如何去实现分布式锁呢？

在Redis存在这样一个命令：

```
setnx key value
```

这个命令看起来和`set`命令差不多，但是它有一个机制，就是只有当指定的key不存在的时候，才能进行插入，实际上就是`set if not exists`的缩写。

![image-20230307001013418](https://s2.loli.net/2023/03/07/fNCxEJRX61cPsuk.png)

可以看到，当客户端1设定a之后，客户端2使用`setnx`会直接失败。

![image-20230307001022446](https://s2.loli.net/2023/03/07/wpGutcmxEsWFJVn.png)

当客户端1将a删除之后，客户端2就可以使用`setnx`成功插入了。

利用这种特性，我们就可以在不同的服务中实现分布式锁，那么问题来了，要是某个服务加了锁但是卡顿了呢，或是直接崩溃了，那这把锁岂不是永远无法释放了？因此我们还可以考虑加个过期时间：

```
set a 666 EX 5 NX
```

这里使用`set`命令，最后加一个NX表示是使用`setnx`的模式，和上面是一样的，但是可以通过EX设定过期时间，这里设置为5秒，也就是说如果5秒还没释放，那么就自动删除。

![image-20230307001031311](https://s2.loli.net/2023/03/07/eQEIGKONmkB2u6y.png)

当然，添加了过期时间，带了的好处是显而易见的，但是同时也带来了很多的麻烦，我们来设想一下这种情况：

![image-20230307001038169](https://s2.loli.net/2023/03/07/nStuP75RLOmQWUM.png)

因此，单纯只是添加过期时间，会出现这种把别人加的锁谁卸了的情况，要解决这种问题也很简单，我们现在的目标就是保证任务只能删除自己加的锁，如果是别人加的锁是没有资格删的，所以我们可以吧a的值指定为我们任务专属的值，比如可以使用UUID之类的，如果在主动删除锁的时候发现值不是我们当前任务指定的，那么说明可能是因为超时，其他任务已经加锁了。

![image-20220414113041835](https://s2.loli.net/2023/03/07/4DW1K38UqQJdwkf.jpg)

如果你在学习本篇之前完成了JUC并发编程篇的学习，那么一定会有一个疑惑，如果在超时之前那一刹那进入到释放锁的阶段，获取到值肯定还是自己，但是在即将执行删除之前，由于超时机制导致被删除并且其他任务也加锁了，那么这时再进行删除，仍然会导致删除其他任务加的锁。

![image-20220414113709773](https://s2.loli.net/2023/03/07/8I1Atm7BOZC5ifS.jpg)

实际上本质还是因为锁的超时时间不太好衡量，如果超时时间能够设定地比较恰当，那么就可以避免这种问题了。

要解决这个问题，我们可以借助一下Redisson框架，它是Redis官方推荐的Java版的Redis客户端。它提供的功能非常多，也非常强大，Redisson内部提供了一个监控锁的看门狗，它的作用是在Redisson实例被关闭前，不断的延长锁的有效期，它为我们提供了很多种分布式锁的实现，使用起来也类似我们在JUC中学习的锁，这里我们尝试使用一下它的分布式锁功能。

```xml
<dependency>
    <groupId>org.redisson</groupId>
    <artifactId>redisson</artifactId>
    <version>3.17.0</version>
</dependency>

<dependency>
    <groupId>io.netty</groupId>
    <artifactId>netty-all</artifactId>
    <version>4.1.75.Final</version>
</dependency>
```

首先我们来看看不加锁的情况下：

```java
public static void main(String[] args) {
    for (int i = 0; i < 10; i++) {
        new Thread(() -> {
            try(Jedis jedis = new Jedis("192.168.0.10", 6379)){
                for (int j = 0; j < 100; j++) {   //每个客户端获取a然后增加a的值再写回去，如果不加锁那么肯定会出问题
                    int a = Integer.parseInt(jedis.get("a")) + 1;
                    jedis.set("a", a+"");
                }
            }
        }).start();
    }
}
```

这里没有直接用`incr`而是我们自己进行计算，方便模拟，可以看到运行结束之后a的值并不是我们想要的：

![image-20220414133258227](https://s2.loli.net/2023/03/07/y2Nvi816ut4jpGC.jpg)

现在我们来给它加一把锁，注意这个锁是基于Redis的，不仅仅只可以用于当前应用，是能够垮系统的：

```java
public static void main(String[] args) {
    Config config = new Config();
    config.useSingleServer().setAddress("redis://192.168.0.10:6379");   //配置连接的Redis服务器，也可以指定集群
    RedissonClient client =  Redisson.create(config);   //创建RedissonClient客户端
    for (int i = 0; i < 10; i++) {
        new Thread(() -> {
            try(Jedis jedis = new Jedis("192.168.0.10", 6379)){
                RLock lock = client.getLock("testLock");    //指定锁的名称，拿到锁对象
                for (int j = 0; j < 100; j++) {
                    lock.lock();    //加锁
                    int a = Integer.parseInt(jedis.get("a")) + 1;
                    jedis.set("a", a+"");
                    lock.unlock();   //解锁
                }
            }
            System.out.println("结束！");
        }).start();
    }
}
```

可以看到结果没有问题：

![image-20220414133403403](https://s2.loli.net/2023/03/07/Gyz1Rc7OWhT5NJK.jpg)

注意，如果用于存放锁的Redis服务器挂了，那么肯定是会出问题的，这个时候我们就可以使用RedLock，它的思路是，在多个Redis服务器上保存锁，只需要超过半数的Redis服务器获取到锁，那么就真的获取到锁了，这样就算挂掉一部分节点，也能保证正常运行，这里就不做演示了。

***

## MySQL与分布式

前面我讲解了Redis在分布式场景的下的相关应用，接着我们来看看MySQL数据库在分布式场景下的应用。

### 主从复制

当我们使用MySQL的时候，也可以采取主从复制的策略，它的实现思路基本和Redis相似，也是采用增量复制的方式，MySQL会在运行的过程中，会记录二进制日志，所有的DML和DDL操作都会被记录进日志中，主库只需要将记录的操作复制给从库，让从库也运行一次，那么就可以实现主从复制。但是注意它不会在一开始进行全量复制，所以最好再开始主从之前将数据库的内容保持一致。

和之前一样，一旦我们实现了主从复制，那么就算主库出现故障，从库也能正常提供服务，并且还可以实现读写分离等操作。这里我们就使用两台主机来搭建一主一从的环境，首先确保两台服务器都安装了MySQL数据库并且都已经正常运行了：

![image-20220414162319865](https://s2.loli.net/2023/03/07/95wL8vICYNp61T2.jpg)

接着我们需要创建对应的账号，一会方便从库进行访问的用户：

```sql
CREATE USER test identified with mysql_native_password by '123456';
```

接着我们开启一下外网访问：

```sh
sudo vim /etc/mysql/mysql.conf.d/mysqld.cnf
```

修改配置文件：

```properties
# If MySQL is running as a replication slave, this should be
# changed. Ref https://dev.mysql.com/doc/refman/8.0/en/server-system-variables.html#sysvar_tmpdir
# tmpdir                = /tmp
#
# Instead of skip-networking the default is now to listen only on
# localhost which is more compatible and is not less secure.
# bind-address          = 127.0.0.1    这里注释掉就行
```

现在我们重启一下MySQL服务：

```sh
sudo systemctl restart mysql.service 
```

现在我们首先来配置主库，主库只需要为我们刚刚创建好的用户分配一个主从复制的权限即可：

```sql
grant replication slave on *.* to test;
FLUSH PRIVILEGES;
```

然后我们可以输入命令来查看主库的相关情况：

![image-20220414164943974](https://s2.loli.net/2023/03/07/kqHZoc8xAbNOd3K.jpg)

这样主库就搭建完成了，接着我们需要将从库进行配置，首先是配置文件：

```properties
# The following can be used as easy to replay backup logs or for replication.
# note: if you are setting up a replication slave, see README.Debian about
#       other settings you may need to change.
# 这里需要将server-id配置为其他的值（默认是1）所有Mysql主从实例的id必须唯一，不能打架，不然一会开启会失败
server-id               = 2
```

进入数据库，输入：

```sql
change replication source to SOURCE_HOST='192.168.0.8',SOURCE_USER='test',SOURCE_PASSWORD='123456',SOURCE_LOG_FILE='binlog.000004',SOURCE_LOG_POS=591;
```

注意后面的logfile和pos就是我们上面从主库中显示的信息。

![image-20220414170022303](https://s2.loli.net/2023/03/07/H7BIl9s3kPu2Mnw.jpg)

执行完成后，显示OK表示没有问题，接着输入：

```sql
start replica;
```

现在我们的从机就正式启动了，现在我们输入：

```sql
show replica status\G;
```

来查看当前从机状态，可以看到：

![image-20220414192045320](https://s2.loli.net/2023/03/07/KiCoVP1cGaf94uX.jpg)

最关键的是下面的Replica_IO_Running和Replica_SQL_Running必须同时为Yes才可以，实际上从库会创建两个线程，一个线程负责与主库进行通信，获取二进制日志，暂时存放到一个中间表（Relay_Log）中，而另一个线程则是将中间表保存的二进制日志的信息进行执行，然后插入到从库中。

最后配置完成，我们来看看在主库进行操作会不会同步到从库：

![image-20220414192508849](https://s2.loli.net/2023/03/07/RxNB3QmUYESX5ad.jpg)

可以看到在主库中创建的数据库，被同步到从库中了，我们再来试试看创建表和插入数据：

```sql
use yyds;
create table test  (
  `id` int primary key,
  `name` varchar(255) NULL,
  `passwd` varchar(255) NULL
);
```

![image-20220414192829536](https://s2.loli.net/2023/03/07/qKBwz31P6ySxlZt.jpg)

现在我们随便插入一点数据：

![image-20220414192920277](https://s2.loli.net/2023/03/07/9pqBFXiLhTPc2xO.jpg)

这样，我们的MySQL主从就搭建完成了，那么如果主机此时挂了会怎么样？

![image-20220414200140191](https://s2.loli.net/2023/03/07/s1Q5xt32r6dv9UJ.jpg)

可以看到IO线程是处于重连状态，会等待主库重新恢复运行。

### 分库分表

在大型的互联网系统中，可能单台MySQL的存储容量无法满足业务的需求，这时候就需要进行扩容了。

和之前的问题一样，单台主机的硬件资源是存在瓶颈的，不可能无限制地纵向扩展，这时我们就得通过多台实例来进行容量的横向扩容，我们可以将数据分散存储，让多台主机共同来保存数据。

那么问题来了，怎么个分散法？

* **垂直拆分：**我们的表和数据库都可以进行垂直拆分，所谓垂直拆分，就是将数据库中所有的表，按照业务功能拆分到各个数据库中（是不是感觉跟前面两章的学习的架构对应起来了）而对于一张表，也可以通过外键之类的机制，将其拆分为多个表。

  ![image-20220414204703883](https://s2.loli.net/2023/03/07/mnJO4hBwDAkRcMi.jpg)

* **水平拆分：**水平拆分针对的不是表，而是数据，我们可以让很多个具有相同表的数据库存放一部分数据，相当于是将数据分散存储在各个节点上。

  ![image-20220414205222383](https://s2.loli.net/2023/03/07/AdS5hrH2O1l8iqv.jpg)

那么要实现这样的拆分操作，我们自行去编写代码工作量肯定是比较大的，因此目前实际上已经有一些解决方案了，比如我们可以使用MyCat（也是一个数据库中间件，相当于挂了一层代理，再通过MyCat进行分库分表操作数据库，只需要连接就能使用，类似的还有ShardingSphere-Proxy）或是Sharding JDBC（应用程序中直接对SQL语句进行分析，然后转换成分库分表操作，需要我们自己编写一些逻辑代码），这里我们就讲解一下Sharding JDBC。

### Sharding JDBC

![image-20220414214856875](https://s2.loli.net/2023/03/07/HTlcExgCfZvG9MP.jpg)

**官方文档（中文）：**https://shardingsphere.apache.org/document/5.1.0/cn/overview/#shardingsphere-jdbc

定位为轻量级 Java 框架，在 Java 的 JDBC 层提供的额外服务，它使用客户端直连数据库，以 jar 包形式提供服务，无需额外部署和依赖，可理解为增强版的 JDBC 驱动，完全兼容 JDBC 和各种 ORM 框架。

- 适用于任何基于 JDBC 的 ORM 框架，如：JPA, Hibernate, Mybatis, Spring JDBC Template 或直接使用 JDBC；
- 支持任何第三方的数据库连接池，如：DBCP, C3P0, BoneCP, HikariCP 等；
- 支持任意实现 JDBC 规范的数据库，目前支持 MySQL，PostgreSQL，Oracle，SQLServer 以及任何可使用 JDBC 访问的数据库。

这里我们主要演示一下水平分表方式，我们直接创建一个新的SpringBoot项目即可，依赖如下：

```xml
<dependencies>
  	<!--  ShardingJDBC依赖，那必须安排最新版啊，希望你们看视频的时候还是5.X版本  -->
    <dependency>
        <groupId>org.apache.shardingsphere</groupId>
        <artifactId>shardingsphere-jdbc-core-spring-boot-starter</artifactId>
        <version>5.1.0</version>
    </dependency>
  
    <dependency>
        <groupId>org.mybatis.spring.boot</groupId>
        <artifactId>mybatis-spring-boot-starter</artifactId>
        <version>2.2.2</version>
    </dependency>

    <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
        <optional>true</optional>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
    </dependency>
</dependencies>
```

数据库我们这里直接用上节课的即可，因为只需要两个表结构一样的数据库即可，正好上节课进行了同步，所以我们直接把从库变回正常状态就可以了：

```sql
stop replica;
```

接着我们把两个表的root用户密码改一下，一会用这个用户连接数据库：

```sql
update user set authentication_string='' where user='root';
update user set host = '%' where user = 'root';
alter user root identified with mysql_native_password by '123456';
FLUSH PRIVILEGES;
```

接着我们来看，如果直接尝试开启服务器，那肯定是开不了的，因为我们要配置数据源：

![image-20220414212443482](https://s2.loli.net/2023/03/07/GEfPLSIZyobhtTe.jpg)

那么数据源该怎么配置呢？现在我们是一个分库分表的状态，需要配置两个数据源：

```yaml
spring:
  shardingsphere:
    datasource:
      # 有几个数据就配几个，这里是名称，按照下面的格式，名称+数字的形式
      names: db0,db1
      # 为每个数据源单独进行配置
      db0:
      	# 数据源实现类，这里使用默认的HikariDataSource
        type: com.zaxxer.hikari.HikariDataSource
        # 数据库驱动
        driver-class-name: com.mysql.cj.jdbc.Driver
        # 不用我多说了吧
        jdbc-url: jdbc:mysql://192.168.0.8:3306/yyds
        username: root
        password: 123456
      db1:
        type: com.zaxxer.hikari.HikariDataSource
        driver-class-name: com.mysql.cj.jdbc.Driver
        jdbc-url: jdbc:mysql://192.168.0.13:3306/yyds
        username: root
        password: 123456
```

如果启动没有问题，那么就是配置成功了：

![image-20220414222958901](https://s2.loli.net/2023/03/07/Hvm82dfbEtwBqrA.jpg)

接着我们需要对项目进行一些编写，添加我们的用户实体类和Mapper：

```java
@Data
@AllArgsConstructor
public class User {
    int id;
    String name;
    String passwd;
}
```

```java
@Mapper
public interface UserMapper {

    @Select("select * from test where id = #{id}")
    User getUserById(int id);

    @Insert("insert into test(id, name, passwd) values(#{id}, #{name}, #{passwd})")
    int addUser(User user);
}
```

实际上这些操作都是常规操作，在编写代码时关注点依然放在业务本身上，现在我们就来编写配置文件，我们需要告诉ShardingJDBC要如何进行分片，首先明确：现在是两个数据库都有test表存放用户数据，我们目标是将用户信息分别存放到这两个数据库的表中。

不废话了，直接上配置：

```yaml
spring:
  shardingsphere:
    rules:
      sharding:
        tables:
        	#这里填写表名称，程序中对这张表的所有操作，都会采用下面的路由方案
        	#比如我们上面Mybatis就是对test表进行操作，所以会走下面的路由方案
          test:
          	#这里填写实际的路由节点，比如现在我们要分两个库，那么就可以把两个库都写上，以及对应的表
          	#也可以使用表达式，比如下面的可以简写为 db$->{0..1}.test
            actual-data-nodes: db0.test,db1.test
            #这里是分库策略配置
            database-strategy:
            	#这里选择标准策略，也可以配置复杂策略，基于多个键进行分片
              standard:
              	#参与分片运算的字段，下面的算法会根据这里提供的字段进行运算
                sharding-column: id
                #这里填写我们下面自定义的算法名称
                sharding-algorithm-name: my-alg
        sharding-algorithms:
        	#自定义一个新的算法，名称随意
          my-alg:
          	#算法类型，官方内置了很多种，这里演示最简单的一种
            type: MOD
            props:
              sharding-count: 2
    props:
    	#开启日志，一会方便我们观察
			sql-show: true
```

其中，分片算法有很多内置的，可以在这里查询：https://shardingsphere.apache.org/document/5.1.0/cn/user-manual/shardingsphere-jdbc/builtin-algorithm/sharding/，这里我们使用的是MOD，也就是取模分片算法，它会根据主键的值进行取模运算，比如我们这里填写的是2，那么就表示对主键进行模2运算，根据数据源的名称，比如db0就是取模后为0，db1就是取模后为1（官方文档描述的并不是很清楚），也就是说，最终实现的效果就是单数放在`db1`，双数放在`db0`，当然它还支持一些其他的算法，这里就不多介绍了。

那么现在我们编写一个测试用例来看看，是否能够按照我们上面的规则进行路由：

```java
@SpringBootTest
class ShardingJdbcTestApplicationTests {

    @Resource
    UserMapper mapper;

    @Test
    void contextLoads() {
        for (int i = 0; i < 10; i++) {
            //这里ID自动生成1-10，然后插入数据库
            mapper.addUser(new User(i, "xxx", "ccc"));   
        }
    }

}
```

现在我们可以开始运行了：

![image-20220415104401263](https://s2.loli.net/2023/03/07/7oBrFRwiXQxcumz.jpg)

测试通过，我们来看看数据库里面是不是按照我们的规则进行数据插入的：

![image-20220415104449502](https://s2.loli.net/2023/03/07/kZINi9wmnte3J7g.jpg)

可以看到这两张表，都成功按照我们指定的路由规则进行插入了，我们来看看详细的路由情况，通过控制台输出的SQL就可以看到：

![image-20220415105325917](https://img-blog.csdnimg.cn/img_convert/2e9cd91031d3fc7d2f11a2f59d8841ae.png)

可以看到所有的SQL语句都有一个Logic SQL（这个就是我们在Mybatis里面写的，是什么就是什么）紧接着下面就是Actual SQL，也就是说每个逻辑SQL最终会根据我们的策略转换为实际SQL，比如第一条数据，它的id是0，那么实际转换出来的SQL会在db0这个数据源进行插入。

这样我们就很轻松地实现了分库策略。

分库完成之后，接着我们来看分表，比如现在我们的数据库中有`test_0`和`test_1`两张表，表结构一样，但是我们也是希望能够根据id取模运算的结果分别放到这两个不同的表中，实现思路其实是差不多的，这里首先需要介绍一下两种表概念：

* **逻辑表：**相同结构的水平拆分数据库（表）的逻辑名称，是 SQL 中表的逻辑标识。 例：订单数据根据主键尾数拆分为 10 张表，分别是 `t_order_0` 到 `t_order_9`，他们的逻辑表名为 `t_order`
* **真实表：**在水平拆分的数据库中真实存在的物理表。 即上个示例中的 `t_order_0` 到 `t_order_9`

现在我们就以一号数据库为例，那么我们在里面创建上面提到的两张表，之前的那个`test`表删不删都可以，就当做不存在就行了：

```sql
create table test_0  (
  `id` int primary key,
  `name` varchar(255) NULL,
  `passwd` varchar(255) NULL
);

create table test_1  (
  `id` int primary key,
  `name` varchar(255) NULL,
  `passwd` varchar(255) NULL
);
```

![image-20220415110322981](https://s2.loli.net/2023/03/07/InHsNXA3E8dQBPa.jpg)

接着我们不要去修改任何的业务代码，Mybatis里面写的是什么依然保持原样，即使我们的表名已经变了，我们需要做的是通过路由来修改原有的SQL，配置如下：

```yaml
spring:
  shardingsphere:
    rules:
      sharding:
        tables:
          test:
            actual-data-nodes: db0.test_$->{0..1}
            #现在我们来配置一下分表策略，注意这里是table-strategy上面是database-strategy
            table-strategy:
            	#基本都跟之前是一样的
              standard:
                sharding-column: id
                sharding-algorithm-name: my-alg
        sharding-algorithms:
          my-alg:
          	#这里我们演示一下INLINE方式，我们可以自行编写表达式来决定
            type: INLINE
            props:
            	#比如我们还是希望进行模2计算得到数据该去的表
            	#只需要给一个最终的表名称就行了test_，后面的数字是表达式取模算出的
            	#实际上这样写和MOD模式一模一样
              algorithm-expression: test_$->{id % 2}
              #没错，查询也会根据分片策略来进行，但是如果我们使用的是范围查询，那么依然会进行全量查询
              #这个我们后面紧接着会讲，这里先写上吧
              allow-range-query-with-inline-sharding: false
```

现在我们来测试一下，看看会不会按照我们的策略进行分表插入：

![image-20220415112809843](https://s2.loli.net/2023/03/07/OaRCMTJ1lnIicSd.jpg)

可以看到，根据我们的算法，原本的逻辑表被修改为了最终进行分表计算后的结果，我们来查看一下数据库：

![image-20220415112908760](https://s2.loli.net/2023/03/07/lfvgOjanPZHMNdr.jpg)

插入我们了解完毕了，我们来看看查询呢：

```java
@SpringBootTest
class ShardingJdbcTestApplicationTests {

    @Resource
    UserMapper mapper;

    @Test
    void contextLoads() {
        System.out.println(mapper.getUserById(0));
        System.out.println(mapper.getUserById(1));
    }

}
```

![image-20220415113139917](https://s2.loli.net/2023/03/07/7K1WBk3s8HuMeOI.jpg)

可以看到，根据我们配置的策略，查询也会自动选择对应的表进行，是不是感觉有内味了。

那么如果是范围查询呢？

```java
@Select("select * from test where id between #{start} and #{end}")
List<User> getUsersByIdRange(int start, int end);
```

```java
@SpringBootTest
class ShardingJdbcTestApplicationTests {

    @Resource
    UserMapper mapper;

    @Test
    void contextLoads() {
        System.out.println(mapper.getUsersByIdRange(3, 5));
    }

}
```

我们来看看执行结果会怎么样：

![image-20220415113530971](https://s2.loli.net/2023/03/07/3Hj7s4xqEwiFXJB.jpg)

可以看到INLINE算法默认是不支持进行全量查询的，我们得将上面的配置项改成true：

```yaml
allow-range-query-with-inline-sharding: true
```

再次进行测试：

![image-20220415113652038](https://s2.loli.net/2023/03/07/WoQqNLCXJslBT3D.jpg)

可以看到，最终出来的SQL语句是直接对两个表都进行查询，然后求出一个并集出来作为最后的结果。

当然除了分片之外，还有广播表和绑定表机制，用于多种业务场景下，这里就不多做介绍了，详细请查阅官方文档。

### 分布式序列算法

前面我们讲解了如何进行分库分表，接着我们来看看分布式序列算法。

在复杂分布式系统中，特别是微服构架中，往往需要对大量的数据和消息进行唯一标识。随着系统的复杂，数据的增多，分库分表成为了常见的方案，对数据分库分表后需要有一个唯一ID来标识一条数据或消息（如订单号、交易流水、事件编号等），此时一个能够生成全局唯一ID的系统是非常必要的。

比如我们之前创建过学生信息表、图书借阅表、图书管理表，所有的信息都会有一个ID作为主键，并且这个ID有以下要求：

* 为了区别于其他的数据，这个ID必须是全局唯一的。
* 主键应该尽可能的保持有序，这样会大大提升索引的查询效率。

那么我们在分布式系统下，如何保证ID的生成满足上面的需求呢？

1. **使用UUID：**UUID是由一组32位数的16进制数字随机构成的，我们可以直接使用JDK为我们提供的UUID类来创建：

   ```java
   public static void main(String[] args) {
       String uuid = UUID.randomUUID().toString();
       System.out.println(uuid);
   }
   ```

   结果为`73d5219b-dc0f-4282-ac6e-8df17bcd5860`，生成速度非常快，可以看到确实是能够保证唯一性，因为每次都不一样，而且这么长一串那重复的概率真的是小的可怜。

   但是它并不满足我们上面的第二个要求，也就是说我们需要尽可能的保证有序，而这里我们得到的都是一些无序的ID。

2. **雪花算法（Snowflake）：**

   我们来看雪花算法，它会生成一个一个64bit大小的整型的ID，int肯定是装不下了。

   ![image-20220415150713707](https://s2.loli.net/2023/03/07/lU9A4zjSIKvaxwh.jpg)

   可以看到它主要是三个部分组成，时间+工作机器ID+序列号，时间以毫秒为单位，41个bit位能表示约70年的时间，时间纪元从2016年11月1日零点开始，可以使用到2086年，工作机器ID其实就是节点ID，每个节点的ID都不相同，那么就可以区分出来，10个bit位可以表示最多1024个节点，最后12位就是每个节点下的序列号，因此每台机器每毫秒就可以有4096个系列号。

   这样，它就兼具了上面所说的唯一性和有序性了，但是依然是有缺点的，第一个是时间问题，如果机器时间出现倒退，那么就会导致生成重复的ID，并且节点容量只有1024个，如果是超大规模集群，也是存在隐患的。

ShardingJDBC支持以上两种算法为我们自动生成ID，文档：https://shardingsphere.apache.org/document/5.1.0/cn/user-manual/shardingsphere-jdbc/builtin-algorithm/keygen/

这里，我们就是要ShardingJDBC来让我们的主键ID以雪花算法进行生成，首先是配置数据库，因为我们默认的id是int类型，装不下64位的，改一下：

```sql
ALTER TABLE `yyds`.`test` MODIFY COLUMN `id` bigint NOT NULL FIRST;
```

接着我们需要修改一下Mybatis的插入语句，因为现在id是由ShardingJDBC自动生成，我们就不需要自己加了：

```java
@Insert("insert into test(name, passwd) values(#{name}, #{passwd})")
int addUser(User user);
```

接着我们在配置文件中将我们的算法写上：

```yaml
spring:
  shardingsphere:
    datasource:
      sharding:
        tables:
          test:
            actual-data-nodes: db0.test,db1.test
            #这里还是使用分库策略
            database-strategy:
              standard:
                sharding-column: id
                sharding-algorithm-name: my-alg
            #这里使用自定义的主键生成策略
            key-generate-strategy:
              column: id
              key-generator-name: my-gen
        key-generators:
        	#这里写我们自定义的主键生成算法
          my-gen:
          	#使用雪花算法
            type: SNOWFLAKE
            props:
            	#工作机器ID，保证唯一就行
              worker-id: 666
        sharding-algorithms:
          my-alg:
            type: MOD
            props:
              sharding-count: 2
```

接着我们来编写一下测试用例：

```java
@SpringBootTest
class ShardingJdbcTestApplicationTests {

    @Resource
    UserMapper mapper;

    @Test
    void contextLoads() {
        for (int i = 0; i < 20; i++) {
            mapper.addUser(new User("aaa", "bbb"));
        }
    }

}
```

可以看到日志：

![image-20220415154524545](https://s2.loli.net/2023/03/07/2JBaqnV8k9OWYfw.jpg)

在插入的时候，将我们的SQL语句自行添加了一个id字段，并且使用的是雪花算法生成的值，并且也是根据我们的分库策略在进行插入操作。

### 读写分离

最后我们来看看读写分离，我们之前实现了MySQL的主从，那么我们就可以将主库作为读，从库作为写：

![image-20220415155842834](https://s2.loli.net/2023/03/07/KRBbGXxhkmUHFIr.jpg)

这里我们还是将数据库变回主从状态，直接删除当前的表，我们重新来过：

```sql
drop table test;
```

我们需要将从库开启只读模式，在MySQL配置中进行修改：

```properties
read-only    = 1
```

这样从库就只能读数据了（但是root账号还是可以写数据），接着我们重启服务器：

```sh
sudo systemctl restart mysql.service
```

然后进入主库，看看状态：

![image-20220415160249024](https://s2.loli.net/2023/03/07/8o4YIB5MysaUuFx.jpg)

现在我们配置一下从库：

```sql
change replication source to SOURCE_HOST='192.168.0.13',SOURCE_USER='test',SOURCE_PASSWORD='123456',SOURCE_LOG_FILE='binlog.000007',SOURCE_LOG_POS=19845;
start replica;
```

现在我们在主库创建表：

```sql
create table test  (
  `id` bigint primary key,
  `name` varchar(255) NULL,
  `passwd` varchar(255) NULL
);
```

然后我们就可以配置ShardingJDBC了，打开配置文件：

```yaml
spring:
  shardingsphere:
    rules:
    	#配置读写分离
      readwrite-splitting:
        data-sources:
        	#名称随便写
          user-db:
          	#使用静态类型，动态Dynamic类型可以自动发现auto-aware-data-source-name，这里不演示
            type: Static
            props:
            	#配置写库（只能一个）
              write-data-source-name: db0
              #配置从库（多个，逗号隔开）
              read-data-source-names: db1
              #负载均衡策略，可以自定义
              load-balancer-name: my-load
        load-balancers:
        	#自定义的负载均衡策略
          my-load:
            type: ROUND_ROBIN
```

注意把之前改的用户实体类和Mapper改回去，这里我们就不用自动生成ID的了。所有的负载均衡算法地址：https://shardingsphere.apache.org/document/5.1.0/cn/user-manual/shardingsphere-jdbc/builtin-algorithm/load-balance/

现在我们就来测试一下吧：

```java
@SpringBootTest
class ShardingJdbcTestApplicationTests {

    @Resource
    UserMapper mapper;

    @Test
    void contextLoads() {
        mapper.addUser(new User(10, "aaa", "bbb"));
        System.out.println(mapper.getUserById(10));
    }

}
```

运行看看SQL日志：

![image-20220415162741466](https://s2.loli.net/2023/03/07/zJvqKmfyhVMFLtZ.jpg)

可以看到，当我们执行插入操作时，会直接向db0进行操作，而读取操作是会根据我们的配置，选择db1进行操作。

至此，微服务应用章节到此结束。
