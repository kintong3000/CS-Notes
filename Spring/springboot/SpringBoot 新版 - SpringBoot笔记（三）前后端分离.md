![image-20230722114659728](https://s2.loli.net/2023/07/22/z7sUmDCOBxvi3a5.png)

# 走进SpringBoot前后端分离

前后端分离是一种软件架构模式，它将前端和后端的开发职责分开，使得前端和后端可以独立进行开发、测试和部署。在之前，我们都是编写Web应用程序，但是随着时代发展，各种桌面App、手机端App还有小程序层出不穷，这都完全脱离我们之前的开发模式，客户端和服务端的划分越来越明显，前后端分离开发势在必行。

在前后端分离架构中，前端主要负责展示层的开发，包括用户界面的设计、用户交互的实现等。前端使用一些技术栈，如Vue、React等技术来实现用户界面，同时通过Ajax、Axios等技术与后端进行数据的交互，这样前端无论使用什么技术进行开发，都与后端无关，受到的限制会小很多。

后端主要负责业务逻辑的处理和数据的存储，包括用户认证、数据验证、数据处理、数据库访问等，我们在SSM阶段就已经给各位小伙伴介绍过了前后端开发的相关思路了，实际上后端只需要返回前端需要的数据即可，我们一般使用JSON格式进行返回。

前后端分离架构的优势包括：

- 前后端可以同时独立进行开发，提高开发效率。
- 前端可以灵活选择技术栈和框架，提供更好的用户体验。
- 后端可以专注于业务逻辑的实现，提高代码的可维护性。
- 前后端通过接口进行通信，使得前端和后端可以分别进行部署，提高系统的可扩展性和灵活性。

![image-20230722122002573](https://s2.loli.net/2023/07/22/8Zxp5PVjN7zfn6b.png)

然而，前后端分离架构也存在一些挑战，包括接口设计的复杂性、前后端协作的沟通成本等。因此，在选择前后端分离架构时，需要综合考虑项目的特点和团队成员的技能，以及开发周期等因素。

本章我们将介绍两种实现前后端分离的方案。

## 基于Session的分离（有状态）

基于Cookie的前后端分离是最简单的一种，也是更接近我们之前学习的一种。在之前，我们都是使用SpringSecurity提供的默认登录流程完成验证。

我们发现，实际上SpringSecurity在登录之后，会利用Session机制记录用户的登录状态，这就要求我们每次请求的时候都需要携带Cookie才可以，因为Cookie中存储了用于识别的JSESSIONID数据。因此，要实现前后端分离，我们只需要稍微修改一下就可以实现了，这对于小型的单端应用程序非常友好。

### 学习环境搭建

考虑到各位小伙伴没有学习过Vue等前端框架，这里我们依然使用前端模版进行魔改。只不过现在我们的前端页面需要单独进行部署，而不是和后端揉在一起，这里我们需要先创建一个前端项目，依赖只需勾选SpringWeb即可，主要用作反向代理前端页面：

![image-20230722151228110](https://s2.loli.net/2023/07/22/A7gTxwv6r89tKh3.png)

如果各位小伙伴学习了Nginx代理，使用Nginx代理前端项目会更好一些。

接着我们将所有的前端模版文件全部丢进对应的目录中，创建一个`web`目录到resource目录下，然后放入我们前端模版的全部文件：

![image-20230722154349756](https://s2.loli.net/2023/07/22/DtLF21ue7RVMQPY.png)

然后配置一下静态资源代理，现在我们希望的是页面直接被代理，不用我们手动去写Controller来解析视图：

```yaml
spring:
  web:
    resources:
      static-locations: classpath:/web
```

然后启动服务器就行了：

![image-20230722154452928](https://s2.loli.net/2023/07/22/65snkmhyjFENTxt.png)

接着我们就可以随便访问我们的网站了：

![image-20230722154659328](https://s2.loli.net/2023/07/22/GEWekp2IwMZhx5c.png)

这样前端页面就部署完成了，接着我们还需要创建一个后端项目，用于去编写我们的后端，选上我们需要的一些依赖：

![image-20230722155049948](https://s2.loli.net/2023/07/22/vt52ogbLp8YN1Im.png)

接着我们需要修改一下后端服务器的端口，因为现在我们要同时开两个服务器，一个是负责部署前端的，一个是负责部署后端的，这样就是标准的前后端分离了，所以说为了防止端口打架，我们就把端口开放在8081上：

```yml
server:
  port: 8081
```

现在启动这两个服务器，我们的学习环境就搭建好了。

### 实现登录授权和跨域处理

在之前，我们的登录操作以及登录之后的页面跳转都是由SpringSecurity来完成，但是现在前后端分离之后，整个流程发生了变化，现在前端仅仅是调用登录接口进行一次校验即可，而后端只需要返回本次校验的结果，由前端来判断是否校验成功并跳转页面：

![image-20230722164431249](https://s2.loli.net/2023/07/22/yZpHd4wcikVxhta.png)

因此，现在我们只需要让登录模块响应一个JSON数据告诉前端登录成功与否即可，当然，前端在发起请求的时候依然需要携带Cookie信息，否则后端不认识是谁。

现在我们就来尝试实现一下这种模式，首先我们配置一下SpringSecurity的相关接口：

```java
@Configuration
public class SecurityConfiguration {

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        return http
                .authorizeHttpRequests(conf -> {
                    conf.anyRequest().authenticated();
                })
                .formLogin(conf -> {
                  	//一般分离之后，为了统一规范接口，使用 /api/模块/功能 的形式命名接口
                    conf.loginProcessingUrl("/api/auth/login");
                    conf.permitAll();
                })
                .csrf(AbstractHttpConfigurer::disable)
                .build();
    }
}
```

虽然这样成功定义了登录接口相关内容，但是怎么才能让SpringSecurity在登录成功之后返回一个JSON数据给前端而不是默认的重定向呢？这时我们可以手动设置SuccessHandler和FailureHandler来实现：

```java
		@Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        return http
                ...
                .formLogin(conf -> {
                    conf.loginProcessingUrl("/api/auth/login");
                  	//使用自定义的成功失败处理器
                    conf.failureHandler(this::onAuthenticationFailure);
                    conf.successHandler(this::onAuthenticationSuccess);
                    conf.permitAll();
                })
                ...
    }

		//自定义成功失败处理器
    void onAuthenticationFailure(HttpServletRequest request,
                                 HttpServletResponse response,
                                 AuthenticationException exception) {

    }

    void onAuthenticationSuccess(HttpServletRequest request, 
                                 HttpServletResponse response, 
                                 Authentication authentication) {
        
    }
```

现在我们需要返回一个标准的JSON格式数据作为响应，这里我们根据Rest API标准来进行编写：

> REST API是遵循REST（Representational State Transfer, 表述性状态转移）原则的Web服务接口，下面简单介绍一下REST接口规范以及对应的响应数据该如何编写:
>
> ### 1. REST接口规范
>
> - **使用HTTP方法**：GET（检索资源）、POST（创建资源）、PUT（更新资源）、DELETE（删除资源）。
> - **无状态**: REST接口要求实现无状态从而使其独立于之前的请求。
> - **使用正确的HTTP状态码**：在HTTP响应中反馈操作的结果（例如，200表示成功，404表示资源不存在等）。
> - **URI 应该清晰易懂**：URI应能清晰地指示出所引用资源的类型和编号，并能易于理解和使用。
>
> ### 2. 响应数据格式
>
> REST应答一般使用的格式为JSON，以下是一个标准的JSON响应数据样例：
>
> ```json
> {
>   "code": 200,
>   "data": {
>     "id": 1,
>     "name": "Tom",
>     "age": 18
>   },
>   "message": "查询成功"
> }
> ```
>
> 字段的含义分别为：
>
> - **code**：HTTP状态码，表示请求的结果。常见的有200（成功）、400（客户端错误）、500（服务器错误）等。
> - **data**：响应的真实数据。在上例中，是一个包含用户信息的对象。
> - **message**：请求响应信息，常用于描述请求处理结果。
>
> 上述都是建议的最佳实践，实际应用中可以根据具体的业务需求进行适当的调整。

这里我们创建一个实体类来装载响应数据，可以使用记录类型：

```java
public record RestBean<T> (int code, T data, String message) {
		//写几个工具方法，用于快速创建RestBean对象
    public static <T> RestBean<T> success(T data){
        return new RestBean<>(200, data, "请求成功");
    }

    public static <T> RestBean<T> failure(int code, String message){
        return new RestBean<>(code, null, message);
    }

    public static <T> RestBean<T> failure(int code){
        return failure(code, "请求失败");
    }
		//将当前对象转换为JSON格式的字符串用于返回
    public String asJsonString() {
        return JSONObject.toJSONString(this, JSONWriter.Feature.WriteNulls);
    }
}
```

接着我们稍微设置一下对应的Handler即可：

```java
		void onAuthenticationFailure(HttpServletRequest request,
                                 HttpServletResponse response,
                                 AuthenticationException exception) throws IOException {
      	response.setContentType("application/json;charset=utf-8");
        PrintWriter writer = response.getWriter();
        writer.write(RestBean.failure(401, exception.getMessage()).asJsonString());
    }

    void onAuthenticationSuccess(HttpServletRequest request,
                                 HttpServletResponse response,
                                 Authentication authentication) throws IOException {
      	response.setContentType("application/json;charset=utf-8");
        PrintWriter writer = response.getWriter();
        writer.write(RestBean.success(authentication.getName()).asJsonString());
    }
```

现在我们就可以使用API测试工具来调试一下了：

![image-20230723193442527](https://s2.loli.net/2023/07/23/EiMUuCjcKpnOmRb.png)

可以看到响应的结果是标准的JSON格式数据，而不是像之前那样重定向到一个页面，这样前端发起的异步请求就可以进行快速判断了。

我们来尝试写一个简单的前端逻辑试试看，这里依然引入Axios框架来发起异步请求：

```html
<script src="https://unpkg.com/axios@1.1.2/dist/axios.min.js"></script>
<script>
    function getInfo() {
        axios.post('http://localhost:8081/api/auth/login', {
            username: document.getElementById('username').value,
            password: document.getElementById('password').value
        }, {
            headers: {
                'Content-Type': 'application/x-www-form-urlencoded'
            },
          	withCredentials: true
        }).then(({data}) => {
            if(data.code === 200) {  //通过状态码进行判断
                window.location.href = '/index.html'  //登录成功进入主页
            } else {
                alert('登录失败：'+data.message)   //登录失败返回弹窗
            }
        })
    }
</script>
```

可能会有小伙伴好奇，这个前端不是每个页面都能随便访问吗，这登录跟不登录有啥区别？实际上我们的前端开发者会在前端做相应的路由以及拦截来控制页面的跳转，我们后端开发者无需担心，我们只需要保证自己返回的数据是准确无误的即可，其他的交给前端小姐姐就好，这里我们只是做个样子。

当点击按钮时就能发起请求了，但是我们现在遇到了一个新的问题：

![image-20230723190406008](https://s2.loli.net/2023/07/23/KYULQNoFsHbm3zg.png)

我们在发起登录请求时，前端得到了一个跨域请求错误，这是因为我们前端的站点和后端站点不一致导致的，浏览器为了用户的安全，防止网页中一些恶意脚本跨站请求数据，会对未经许可的跨域请求发起拦截。那么，我们怎么才能让这个请求变成我们许可的呢？对于跨域问题，是属于我们后端需要处理的问题，跟前端无关，我们需要在响应的时候，在响应头中添加一些跨域属性，来告诉浏览器从哪个站点发来的跨域请求是安全的，这样浏览器就不会拦截了。

那么如何进行配置呢，我们现在使用了SpringSecurity框架，可以直接进行跨域配置：

```java
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        return http
                ...
                .cors(conf -> {
                    CorsConfiguration cors = new CorsConfiguration();
                  	//添加前端站点地址，这样就可以告诉浏览器信任了
                  	cors.addAllowedOrigin("http://localhost:8080");
                    //虽然也可以像这样允许所有 cors.addAllowedOriginPattern("*");
                  	//但是这样并不安全，我们应该只许可给我们信任的站点
                    cors.setAllowCredentials(true);  //允许跨域请求中携带Cookie
                    cors.addAllowedHeader("*");   //其他的也可以配置，为了方便这里就 * 了
                    cors.addAllowedMethod("*");
                    cors.addExposedHeader("*");
                    UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
                    source.registerCorsConfiguration("/**", cors);  //直接针对于所有地址生效
                    conf.configurationSource(source);
                })
                ...
                .build();
    }
```

这样，当我们再次重启服务器，返回的响应头中都会携带跨域相关的信息，这样浏览器就不会进行拦截了：

![image-20230723192217101](https://s2.loli.net/2023/07/23/QVFEWknMdujomqi.png)

这样就可以实现前后端分离的登录模式了：

![image-20230723194030641](https://s2.loli.net/2023/07/23/1GpZuQUawM48eVq.png)

由于记住我功能和退出登录操作跟之前是一样的配置，这里我们就不进行演示了。

### 响应JSON化

前面我们完成了前后端分离的登录模式，我们来看看一般的业务接口该如何去实现，比如这里我们写一个非常简单的的用户名称获取接口：

```java
@RestController   //为了方便，我们一律使用RestController，这样每个请求默认都返回JSON对象
@RequestMapping("/api/user")   //用户相关的接口，路径可以设置为/api/user/xxxx
public class UserController {

    @GetMapping("/name")
    public RestBean<String> username() {
        User user = (User) SecurityContextHolder.getContext().getAuthentication().getPrincipal();
        return RestBean.success(user.getUsername());
    }
}
```

这样前端就可以在登录之后获取到这个接口的结果了，注意一定要在请求时携带Cookie，否则服务端无法识别身份，会直接被拦截并重定向：

```html
<script>
    axios.get('http://localhost:8081/api/user/name', {
        withCredentials: true  //携带Cookie访问，不然服务器不认识我们
    }).then(({data}) => {
        document.getElementById('username').innerText = data.data
    })
</script>
```

注意一定要登录之后再请求，成功的请求结果如下：

![image-20230724000237828](https://s2.loli.net/2023/07/24/L4PcVKpO2nmHG7e.png)

不过我们发现，我们的一些响应还是不完善，比如用户没有登录，默认还是会302重定向，但是实际上我们只需要告诉前端没有登录就行了，所以说我们修改一下未登录状态下返回的结果：

```java
    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        return http
                ...
                .exceptionHandling(conf -> {
                  	//配置授权相关异常处理器
                    conf.accessDeniedHandler(this::onAccessDeny);
                  	//配置验证相关异常的处理器
                    conf.authenticationEntryPoint(this::onAuthenticationFailure);
                })
                .build();
    }
```

现在有三个方法，但是实际上功能都是一样的，我们可以把它们整合为同一个方法：

```java
		private void handleProcess(HttpServletRequest request,
                       HttpServletResponse response,
                       Object exceptionOrAuthentication) throws IOException {
        response.setContentType("application/json;charset=utf-8");
        PrintWriter writer = response.getWriter();
        if(exceptionOrAuthentication instanceof AccessDeniedException exception) {
            writer.write(RestBean.failure(403, exception.getMessage()).asJsonString());
        } else if(exceptionOrAuthentication instanceof Exception exception) {
            writer.write(RestBean.failure(401, exception.getMessage()).asJsonString());
        } else if(exceptionOrAuthentication instanceof Authentication authentication){
            writer.write(RestBean.success(authentication.getName()).asJsonString());
        }
    }
```

这样，用户在没有登录的情况下，请求接口就会返回我们的自定义JSON信息了：

![image-20230724002459523](https://s2.loli.net/2023/07/24/Rf9BSVLvih1lOE2.png)

对于我们页面中的一些常见的异常，我们也可以编写异常处理器来将其规范化返回，比如404页面，我们可以直接配置让其抛出异常：

```yml
spring:
	mvc:
    throw-exception-if-no-handler-found: true
  web:
    resources:
      add-mappings: false
```

然后编写异常处理器：

```java
@RestController
@ControllerAdvice
public class ExceptionController {

    @ExceptionHandler(Exception.class)
    public RestBean<String> error(Exception e){
        if(e instanceof NoHandlerFoundException exception)  //这里就大概处理一下404就行
            return RestBean.failure(404, e.getMessage());  
        else if (e instanceof ServletException exception)  //其他的Servlet异常就返回400状态码
            return RestBean.failure(400, e.getMessage());
        else
            return RestBean.failure(500, e.getMessage());  //其他异常直接返回500
    }
}
```

这样我们的后端就返回的是非常统一的JSON格式数据了，前端开发人员只需要根据我们返回的数据编写统一的处理即可，基于Session的前后端分离实现起来也是最简单的，几乎没有多少的学习成本，跟我们之前的使用是一样的，只是现在前端单独编写了而已。

***

## 基于Token的分离（无状态）

基于Token的前后端分离主打无状态，无状态服务是指在处理每个请求时，服务本身不会维持任何与请求相关的状态信息。每个请求被视为独立的、自包含的操作，服务只关注处理请求本身，而不关心前后请求之间的状态变化。也就是说，用户在发起请求时，服务器不会记录其信息，而是通过用户携带的Token信息来判断是哪一个用户：

* 有状态：用户请求接口 ->  从Session中读取用户信息  ->   根据当前的用户来处理业务   ->  返回
* 无状态：用户携带Token请求接口    ->   从请求中获取用户信息   ->   根据当前的用户来处理业务   ->  返回

无状态服务的优点包括：

1. 服务端无需存储会话信息：传统的会话管理方式需要服务端存储用户的会话信息，包括用户的身份认证信息和会话状态。而使用Token，服务端无需存储任何会话信息，所有的认证信息都包含在Token中，使得服务端变得无状态，减轻了服务器的负担，同时也方便了服务的水平扩展。
2. 减少网络延迟：传统的会话管理方式需要在每次请求中都携带会话标识，即使是无状态的RESTful API也需要携带身份认证信息。而使用Token，身份认证信息已经包含在Token中，只需要在请求的Authorization头部携带Token即可，减少了每次请求的数据量，减少了网络延迟。
3. 客户端无需存储会话信息：传统的会话管理方式中，客户端需要存储会话标识，以便在每次请求中携带。而使用Token，客户端只需要保存Token即可，方便了客户端的存储和管理。
4. 跨域支持：Token可以在各个不同的域名之间进行传递和使用，因为Token是通过签名来验证和保护数据完整性的，可以防止未经授权的修改。

这一部分，我们将深入学习目前比较主流的基于Token的前后端分离方案。

### 认识JWT令牌

在认识Token前后端分离之前，我们需要先学习最常见的JWT令牌，官网：https://jwt.io

JSON Web Token令牌（JWT）是一个开放标准（[RFC 7519](https://tools.ietf.org/html/rfc7519)），它定义了一种紧凑和自成一体的方式，用于在各方之间作为JSON对象安全地传输信息。这些信息可以被验证和信任，因为它是数字签名的。JWT可以使用密钥（使用**HMAC**算法）或使用**RSA**或**ECDSA**进行公钥/私钥对进行签名。

JWT令牌的格式如下：

![image-20230307000004710](https://s2.loli.net/2023/03/07/Xu8lxYhKoJNr6it.png)

一个JWT令牌由3部分组成：标头(Header)、有效载荷(Payload)和签名(Signature)。在传输的时候，会将JWT的钱2部分分别进行Base64编码后用`.`进行连接形成最终需要传输的字符串。

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

因此，JWT令牌实际上是一种经过加密的JSON数据，其中包含了用户名字、用户ID等信息，我们可以直接解密JWT令牌得到用户的信息，我们可以写一个小测试来看看，导入JWT支持库依赖：

```xml
<dependency>
     <groupId>com.auth0</groupId>
     <artifactId>java-jwt</artifactId>
     <version>4.3.0</version>
</dependency>
```

要生成一个JWT令牌非常简单：

```java
public class Main {
    public static void main(String[] args) {
        String jwtKey = "abcdefghijklmn";                 //使用一个JWT秘钥进行加密
        Algorithm algorithm = Algorithm.HMAC256(jwtKey);  //创建HMAC256加密算法对象
        String jwtToken = JWT.create()
                .withClaim("id", 1)   //向令牌中塞入自定义的数据
                .withClaim("name", "lbw")
                .withClaim("role", "nb")
                .withExpiresAt(new Date(2024, Calendar.FEBRUARY, 1))  //JWT令牌的失效时间
                .withIssuedAt(new Date())   //JWT令牌的签发时间
                .sign(algorithm);    //使用上面的加密算法进行加密，完成签名
        System.out.println(jwtToken);   //得到最终的JWT令牌
    }
}
```

可以看到最后得到的JWT令牌就长这样：

```
eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJyb2xlIjoibmIiLCJuYW1lIjoibGJ3IiwiaWQiOjEsImV4cCI6NjE2NjQ4NjA4MDAsImlhdCI6MTY5MDEzMTQ3OH0.KUuGKM0OynL_DEUnRIETDBlmGjoqbt_5dP2r21ZDE1s
```

我们可以使用Base64将其还原为原本的样子：

```java
public static void main(String[] args) {
        String jwtToken = "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJyb2xlIjoibmIiLCJuYW1lIjoibGJ3IiwiaWQiOjEsImV4cCI6NjE2NjQ4NjA4MDAsImlhdCI6MTY5MDEzMTQ3OH0.KUuGKM0OynL_DEUnRIETDBlmGjoqbt_5dP2r21ZDE1s";
        String[] split = jwtToken.split("\\.");
        for (int i = 0; i < split.length - 1; i++) {
            String s = split[i];
            byte[] decode = Base64.getDecoder().decode(s);
            System.out.println(new String(decode));
        }
}
```

解析前面两个部分得到：

```json
{"typ":"JWT","alg":"HS256"}
{"role":"nb","name":"lbw","id":1,"exp":61664860800,"iat":1690131478}
```

可以看到确实是经过Base64加密的JSON格式数据，包括我们的自定义数据也在其中，而我们可以直接使用JWT令牌来作为我们权限校验的核心，我们可以像这样设计我们的系统：

![image-20230724010807761](https://s2.loli.net/2023/07/24/4bThtMwA9XsP5uc.png)

首先用户还是按照正常流程进行登录，只不过用户在登录成功之后，服务端会返回一个JWT令牌用于后续请求使用，由于JWT令牌具有时效性，所以说当过期之后又需要重新登录。就像我们进入游乐园需要一张门票一样，只有持有游乐园门票才能进入游乐园游玩，如果没有门票就会被拒之门外，而游乐园门票也有时间限制，如果已经过期，我们也是没有办法进入游乐园的。

所以，我们只需要在后续请求中携带这个Token即可（可以放在Cookie中，也可以放在请求头中）这样服务器就可以直接从Token中解密读取到我们用户的相关信息以及判断用户是否登录过期了。

不过这个时候会有小伙伴疑问，既然现在用户信息都在JWT中，那要是用户随便修改里面的内容，岂不是可以以任意身份访问服务器了？这会不会存在安全隐患？对于这个问题，前面我们已经说的很清楚了，JWT实际上最后会有一个加密的签名，这个是根据秘钥+JWT本体内容计算得到的，用户在没有持有秘钥的情况下，是不可能计算得到正确的签名的，所以说服务器会在收到JWT时对签名进行重新计算，比较是否一致，来验证JWT是否被用户恶意修改，如果被修改肯定也是不能通过的。

![image-20230724011814993](https://s2.loli.net/2023/07/24/17dmiHXEG4rLO6W.png)

### SpringSecurity实现JWT校验

前面我们介绍了JWT的基本原理以及后端的基本校验流程，那么我们现在就来看看如何实现这样的流程。

SpringSecurity中并没有为我们提供预设的JWT校验模块（只有OAuth2模块才有，但是知识太超前了，我们暂时不讲解）这里我们只能手动进行整合，JWT可以存放在Cookie或是请求头中，不过不管哪种方式，我们都可以通过Request获取到对应的JWT令牌，这里我们使用比较常见的请求头携带JWT的方案，客户端发起的请求中会携带这样的的特殊请求头：

```
Authorization: Bearer eyJhbGciOiJSUzI1NiJ9.eyJpc3MiOiJzZWxmIiwic3ViIjoidXNlciIsImV4cCI6MTY5MDIxODE2NCwiaWF0IjoxNjkwMTgyMTY0LCJzY29wZSI6ImFwcCJ9.Z5-WMeulZyx60WeNxrQg2z2GiVquEHrsBl9V4dixbRkAD6rFp-6gCrcAXWkebs0i-we4xTQ7TZW0ltuhGYZ1GmEaj4F6BP9VN8fLq2aT7GhCJDgjikaTs-w5BbbOD2PN_vTAK_KeVGvYhWU4_l81cvilJWVXAhzMtwgPsz1Dkd04cWTCpI7ZZi-RQaBGYlullXtUrehYcjprla8N-bSpmeb3CBVM3kpAdehzfRpAGWXotN27PIKyAbtiJ0rqdvRmvlSztNY0_1IoO4TprMTUr-wjilGbJ5QTQaYUKRHcK3OJrProz9m8ztClSq0GRvFIB7HuMlYWNYwf7lkKpGvKDg
```

这里的Authorization请求头就是携带JWT的专用属性，值的格式为"Bearer Token"，前面的Bearer代表身份验证方式，默认情况下有两种：

> Basic 和 Bearer 是两种不同的身份验证方式。
>
> Basic 是一种基本的身份验证方式，它将用户名和密码进行base64编码后，放在 Authorization 请求头中，用于向服务器验证用户身份。这种方式不够安全，因为它将密码以明文的形式传输，容易受到中间人攻击。
>
> Bearer 是一种更安全的身份验证方式，它基于令牌（Token）来验证用户身份。Bearer 令牌是由身份验证服务器颁发给客户端的，客户端在每个请求中将令牌放在 Authorization 请求头的 Bearer 字段中。服务器会验证令牌的有效性和权限，以确定用户的身份。Bearer 令牌通常使用 JSON Web Token (JWT) 的形式进行传递和验证。

一会我们会自行编写JWT校验拦截器来处理这些信息。

首先我们先把用于处理JWT令牌的工具类完成一下：

```java
public class JwtUtils {
  	//Jwt秘钥
    private static final String key = "abcdefghijklmn";

  	//根据用户信息创建Jwt令牌
    public static String createJwt(UserDetails user){
        Algorithm algorithm = Algorithm.HMAC256(key);
        Calendar calendar = Calendar.getInstance();
        Date now = calendar.getTime();
        calendar.add(Calendar.SECOND, 3600 * 24 * 7);
        return JWT.create()
                .withClaim("name", user.getUsername())  //配置JWT自定义信息
                .withClaim("authorities", user.getAuthorities().stream().map(GrantedAuthority::getAuthority).toList())
                .withExpiresAt(calendar.getTime())  //设置过期时间
                .withIssuedAt(now)    //设置创建创建时间
                .sign(algorithm);   //最终签名
    }

  	//根据Jwt验证并解析用户信息
    public static UserDetails resolveJwt(String token){
        Algorithm algorithm = Algorithm.HMAC256(key);
        JWTVerifier jwtVerifier = JWT.require(algorithm).build();
        try {
            DecodedJWT verify = jwtVerifier.verify(token);  //对JWT令牌进行验证，看看是否被修改
            Map<String, Claim> claims = verify.getClaims();  //获取令牌中内容
            if(new Date().after(claims.get("exp").asDate())) //如果是过期令牌则返回null
                return null;
            else
              	//重新组装为UserDetails对象，包括用户名、授权信息等
                return User
                        .withUsername(claims.get("name").asString())
                        .password("")
                        .authorities(claims.get("authorities").asArray(String.class))
                        .build();
        } catch (JWTVerificationException e) {
            return null;
        }
    }
}
```

接着我们需要自行实现一个JwtAuthenticationFilter加入到SpringSecurity默认提供的过滤器链（有关SpringSecurity的实现原理介绍，我们在SSM中已经详细讲解过，各位小伙伴可以回顾一下）中，用于处理请求头中携带的JWT令牌，并配置登录状态：

```java
public class JwtAuthenticationFilter extends OncePerRequestFilter {  
//继承OncePerRequestFilter表示每次请求过滤一次，用于快速编写JWT校验规则

    @Override
    protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain filterChain) throws ServletException, IOException {
      	//首先从Header中取出JWT
        String authorization = request.getHeader("Authorization");
      	//判断是否包含JWT且格式正确
        if (authorization != null && authorization.startsWith("Bearer ")) {
            String token = authorization.substring(7);	
          	//开始解析成UserDetails对象，如果得到的是null说明解析失败，JWT有问题
            UserDetails user = JwtUtils.resolveJwt(token);
            if(user != null) {
              	//验证没有问题，那么就可以开始创建Authentication了，这里我们跟默认情况保持一致
              	//使用UsernamePasswordAuthenticationToken作为实体，填写相关用户信息进去
                UsernamePasswordAuthenticationToken authentication =
                        new UsernamePasswordAuthenticationToken(user, null, user.getAuthorities());
                authentication.setDetails(new WebAuthenticationDetailsSource().buildDetails(request));
              	//然后直接把配置好的Authentication塞给SecurityContext表示已经完成验证
                SecurityContextHolder.getContext().setAuthentication(authentication);
            }
        }
      	//最后放行，继续下一个过滤器
      	//可能各位小伙伴会好奇，要是没验证成功不是应该拦截吗？这个其实没有关系的
      	//因为如果没有验证失败上面是不会给SecurityContext设置Authentication的，后面直接就被拦截掉了
      	//而且有可能用户发起的是用户名密码登录请求，这种情况也要放行的，不然怎么登录，所以说直接放行就好
        filterChain.doFilter(request, response);
    }
}
```

最后我们来配置一下SecurityConfiguration配置类，其实配置方法跟之前还是差不多，用户依然可以使用表单进行登录，并且登录方式也是一样的，就是有两个新增的部分需要我们注意一下：

```java
@Configuration
public class SecurityConfiguration {

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        return http
          			//其他跟之前一样，就省略掉了
                ...  
                //将Session管理创建策略改成无状态，这样SpringSecurity就不会创建会话了，也不会采用之前那套机制记录用户，因为现在我们可以直接从JWT中获取信息
                .sessionManagement(conf -> {
                    conf.sessionCreationPolicy(SessionCreationPolicy.STATELESS);
                })
          			//添加我们用于处理JWT的过滤器到Security过滤器链中，注意要放在UsernamePasswordAuthenticationFilter之前
                .addFilterBefore(new JwtAuthenticationFilter(), UsernamePasswordAuthenticationFilter.class)
                .build();
    }

  	//这个跟之前一样的写法，整合到一起处理，统一返回JSON格式
    private void handleProcess(HttpServletRequest request,
                               HttpServletResponse response,
                               Object exceptionOrAuthentication) throws IOException {
        response.setContentType("application/json;charset=utf-8");
        PrintWriter writer = response.getWriter();
        if(exceptionOrAuthentication instanceof AccessDeniedException exception) {
            writer.write(RestBean.failure(403, exception.getMessage()).asJsonString());
        } else if(exceptionOrAuthentication instanceof AuthenticationException exception) {
            writer.write(RestBean.failure(401, exception.getMessage()).asJsonString());
        } else if(exceptionOrAuthentication instanceof Authentication authentication){
          	//不过这里需要注意，在登录成功的时候需要返回我们生成的JWT令牌，这样客户端下次访问就可以携带这个令牌了，令牌过期之后就需要重新登录才可以
            writer.write(RestBean.success(JwtUtils.createJwt((User) authentication.getPrincipal())).asJsonString());
        }
    }
}
```

最后我们创建一个测试使用的Controller来看看效果：

```java
@RestController
public class TestController {

    @GetMapping("/test")
    public String test(){
        return "HelloWorld";
    }
}
```

那么现在采用JWT之后，我们要怎么使用呢？首先我们还是使用工具来测试一下：

![image-20230724200235358](https://s2.loli.net/2023/07/24/L1O8m6auYc2IFWR.png)

登录成功之后，可以看到现在返回给我们了一个JWT令牌，接着我们就可以使用这个令牌了。比如现在我们要访问某个接口获取数据，那么就可以携带这个令牌进行访问：

![image-20230724200341917](https://s2.loli.net/2023/07/24/Hn7X5qeDf9htk6P.png)

注意需要在请求头中添加：

```
Authorization: Bearer 刚刚获取的Token
```

如果以后没有登录或者携带一个错误的JWT访问服务器，都会返回401错误：

![image-20230724200533964](https://s2.loli.net/2023/07/24/ID96yY7lkr5VsPS.png)

我们现在来模拟一下前端操作：

```html
<script>
  	//其他都是跟之前一样的
    function getInfo() {
        axios.post('http://localhost:8081/api/auth/login', {
            username: document.getElementById('username').value,
            password: document.getElementById('password').value
        }, {
            headers: {
                'Content-Type': 'application/x-www-form-urlencoded'
            }
        }).then(({data}) => {
            if(data.code === 200) {
                //将得到的JWT令牌存到sessionStorage用于本次会话
                sessionStorage.setItem("access_token", data.data)
                window.location.href = '/index.html'
            } else {
                alert('登录失败：'+data.message)
            }
        })
    }
</script>
```

接着是首页，获取信息的时候携带上JWT即可，不需要依赖Cookie了：

```html
<script>
    axios.get('http://localhost:8081/api/user/name', {
        headers: {
            'Authorization': "Bearer "+sessionStorage.getItem("access_token")
        }
    }).then(({data}) => {
        document.getElementById('username').innerText = data.data
    })
</script>
```

这样我们就实现了基于SpringSecurity的JWT校验，整个流程还是非常清晰的。

### 退出登录JWT处理

虽然我们使用JWT已经很方便了，但是有一个很严重的问题就是，我们没办法像Session那样去踢用户下线，什么意思呢？我们之前可以使用退出登录接口直接退出，用户Session中的验证信息也会被销毁，但是现在是无状态的，用户来管理Token令牌，服务端只认Token是否合法，那这个时候该怎么让用户正确退出登录呢？

首先我们从最简单的方案开始，我们可以直接让客户端删除自己的JWT令牌，这样不就相当于退出登录了吗，这样甚至不需要请求服务器，直接就退了：

```html
<script>
		...
  
    function logout() {
        //直接删除存在sessionStorage中的JWT令牌
        sessionStorage.removeItem("access_token")
        //然后回到登录界面
        window.location.href = '/login.html'
    }
</script>
```

这样虽然是最简单粗暴的，但是存在一个问题，用户可以自行保存这个Token拿来使用。虽然客户端已经删除掉了，但是这个令牌仍然是可用的，如果用户私自保存过，那么依然可以正常使用这个令牌，这显然是有问题的。

目前有两种比较好的方案：

* 黑名单方案：所有黑名单中的JWT将不可使用。
* 白名单方案：不在白名单中的JWT将不可使用。

这里我们以黑名单机制为例，让用户退出登录之后，无法再次使用之前的JWT进行操作，首先我们需要给JWT额外添加一个用于判断的唯一标识符，这里就用UUID好了：

```java
public class JwtUtils {
    private static final String key = "abcdefghijklmn";

    public static String createJwt(UserDetails user){
        Algorithm algorithm = Algorithm.HMAC256(key);
        Calendar calendar = Calendar.getInstance();
        Date now = calendar.getTime();
        calendar.add(Calendar.SECOND, 3600 * 24 * 7);
        return JWT.create()
          			//额外添加一个UUID用于记录黑名单，将其作为JWT的ID属性jti
          			.withJWTId(UUID.randomUUID().toString())
                .withClaim("name", user.getUsername())
                .withClaim("authorities", user.getAuthorities().stream().map(GrantedAuthority::getAuthority).toList())
                .withExpiresAt(calendar.getTime())
                .withIssuedAt(now)
                .sign(algorithm);
    }
  
		...
}
```

这样我们发出去的所有令牌都会携带一个UUID作为唯一凭据，接着我们可以创建一个专属的表用于存储黑名单：

```java
public class JwtUtils {	
  
  private static final HashSet<String> blackList = new HashSet<>();
  //加入黑名单方法
  public static boolean invalidate(String token){
        Algorithm algorithm = Algorithm.HMAC256(key);
        JWTVerifier jwtVerifier = JWT.require(algorithm).build();
        try {
            DecodedJWT verify = jwtVerifier.verify(token);
            Map<String, Claim> claims = verify.getClaims();
          	//取出UUID丢进黑名单中
            return blackList.add(verify.getId());
        } catch (JWTVerificationException e) {
            return false;
        }
  }
  
  ...
  
	public static UserDetails resolveJwt(String token){
        Algorithm algorithm = Algorithm.HMAC256(key);
        JWTVerifier jwtVerifier = JWT.require(algorithm).build();
        try {
            DecodedJWT verify = jwtVerifier.verify(token);
            //判断是否存在于黑名单中，如果存在，则返回null表示失效
            if(blackList.contains(verify.getId()))
                return null;
            Map<String, Claim> claims = verify.getClaims();
            if(new Date().after(claims.get("exp").asDate()))
                return null;
            return User
                    .withUsername(claims.get("name").asString())
                    .password("")
                    .authorities(claims.get("authorities").asArray(String.class))
                    .build();
        } catch (JWTVerificationException e) {
            return null;
        }
    }
}
```

接着我们来SecurityConfiguration中配置一下退出登录操作：

```java
private void onLogoutSuccess(HttpServletRequest request,
                                 HttpServletResponse response,
                                 Authentication authentication) throws IOException {
        response.setContentType("application/json;charset=utf-8");
        PrintWriter writer = response.getWriter();
        String authorization = request.getHeader("Authorization");
        if(authorization != null && authorization.startsWith("Bearer ")) {
            String token = authorization.substring(7);
          	//将Token加入黑名单
            if(JwtUtils.invalidate(token)) {
              	//只有成功加入黑名单才会退出成功
                writer.write(RestBean.success("退出登录成功").asJsonString());
                return;
            }
        }
        writer.write(RestBean.failure(400, "退出登录失败").asJsonString());
}
```

这样，我们就成功安排上了黑名单机制，即使用户提前保存，这个Token依然是失效的：

![image-20230724214624046](https://s2.loli.net/2023/07/24/4o76q5yNHkabuip.png)

虽然这种黑名单机制很方便，但是如果到了后面的微服务阶段，可能多个服务器都需要共享这个黑名单，这个时候我们再将黑名单存储在单个应用中就不太行了，后续我们可以考虑使用Redis服务器来存放黑名单列表，这样就可以实现多个服务器共享，并且根据JWT的过期时间合理设定黑名单中UUID的过期时间，自动清理。

### 自动续签JWT令牌

在有些时候，我们可能希望用户能够一直使用我们的网站，而不是JWT令牌到期之后就需要重新登录，这种情况下前端就可以配置JWT自动续签，在发起请求时如果令牌即将到期，那么就向后端发起续签请求得到一个新的JWT令牌。

这里我们写一个接口专门用于令牌刷新：

```java
@RestController
@RequestMapping("/api/auth")
public class AuthorizeController {

    @GetMapping("/refresh")
    public RestBean<String> refreshToken(){
        User user = (User) SecurityContextHolder.getContext().getAuthentication().getPrincipal();
        String jwt = JwtUtils.createJwt(user);
        return RestBean.success(jwt);
    }
}
```

这样，前端在发现令牌可用时间不足时，就会先发起一个请求自动完成续期，得到一个新的Token：

![image-20230724232152613](https://s2.loli.net/2023/07/24/cqEgnQOZtFp1w7o.png)

我们可能还需要配置一下这种方案的请求频率，不然用户疯狂请求刷新Token就不太好了，我们同样可以借助Redis进行限流等操作，防止频繁请求，这里就不详细编写了，各位小伙伴可以自行实现。

我们最后可以来对比一下两种前后端分离方式的优缺点如何：

**JWT校验方案的优点：**

1. 无状态: JWT是无状态的，服务器不需要在后端维护用户的会话信息，可以在分布式系统中进行水平扩展，减轻服务器的负担。
2. 基于Token: JWT使用token作为身份认证信息，该token可以存储用户相关的信息和权限。这样可以减少与数据库的频繁交互，提高性能。
3. 安全性: JWT使用数字签名或加密算法保证token的完整性和安全性。每次请求都会验证token的合法性，防止伪造或篡改。
4. 跨域支持: JWT可以在不同域之间进行数据传输，适合前后端分离的架构。

**JWT校验方案的缺点：**

1. 无法做到即时失效: JWT中的token通常具有较长的有效期，一旦签发，就无法立即失效。如果需要即时失效，需要在服务端进行额外的处理。
2. 信息无法撤销: JWT中的token一旦签发，除非到期或者客户端清除，无法撤销。无法中途取消和修改权限。
3. Token增大的问题: JWT中包含了用户信息和权限等，token的体积较大，每次请求都需要携带，增加了网络传输的开销。
4. 动态权限管理问题: JWT无法处理动态权限管理，一旦签发的token权限发生变化，仍然有效，需要其他手段进行处理。

**传统Session校验方案的优点：**

1. 即时失效: Session在服务器端管理，可以通过设置过期时间或手动删除实现即时失效，保护会话的安全性。
2. 信息即时撤销: 服务器端可以随时撤销或修改Session的信息和权限。
3. 灵活的权限管理: Session方案可以更灵活地处理动态权限管理，可以根据具体场景进行即时调整。

**传统Session校验方案的缺点：**

1. 状态维护: 传统Session需要在服务器端维护会话状态信息，增加了服务器的负担，不利于系统的横向扩展。
2. 性能开销: 每次请求都需要在服务器端进行会话状态的校验和读写操作，增加了性能开销。
3. 跨域问题: Session方案在跨域时存在一些问题，需要进行额外的处理。
4. 无法分布式共享: 传统Session方案不适用于多个服务器之间共享会话信息的场景，需要额外的管理和同步机制。

综上所述，JWT校验方案适用于无状态、分布式系统，几乎所有常见的前后端分离的架构都可以采用这种方案。而传统Session校验方案适用于需要即时失效、即时撤销和灵活权限管理的场景，适合传统的服务器端渲染应用，以及客户端支持Cookie功能的前后端分离架构。在选择校验方案时，需要根据具体的业务需求和技术场景进行选择。
