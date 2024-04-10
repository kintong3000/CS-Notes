# Springboot项目相关

## 常用依赖

``` xml
<dependencies>
        <!--    邮件发送模块    -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-mail</artifactId>
        </dependency>
        <!--    接口参数校验模块    -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-validation</artifactId>
        </dependency>
        <!--    授权校验模块    -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-security</artifactId>
        </dependency>
        <!--    基础Web模块    -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <!--    Redis交互模块    -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-redis</artifactId>
        </dependency>
        <!--    Mybatis-Plus框架    -->
        <dependency>
            <groupId>com.baomidou</groupId>
            <artifactId>mybatis-plus-boot-starter</artifactId>
            <version>3.5.3.1</version>
        </dependency>
        <!--    MySQL驱动    -->
        <dependency>
            <groupId>com.mysql</groupId>
            <artifactId>mysql-connector-j</artifactId>
            <scope>runtime</scope>
        </dependency>
        <!--    Lombok框架    -->
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>
        <!--    测试模块    -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.springframework.security</groupId>
            <artifactId>spring-security-test</artifactId>
            <scope>test</scope>
        </dependency>
        <!--    消息队列模块    -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-amqp</artifactId>
        </dependency>
        <!--    FastJSON2框架    -->
        <dependency>
            <groupId>com.alibaba.fastjson2</groupId>
            <artifactId>fastjson2</artifactId>
            <version>2.0.25</version>
        </dependency>
        <!--    Jwt令牌生成校验框架    -->
        <dependency>
            <groupId>com.auth0</groupId>
            <artifactId>java-jwt</artifactId>
            <version>4.3.0</version>
        </dependency>
        <!--    Swagger文档生成框架    -->
        <dependency>
            <groupId>org.springdoc</groupId>
            <artifactId>springdoc-openapi-starter-webmvc-ui</artifactId>
            <version>2.1.0</version>
        </dependency>
    </dependencies>
```



## spring security

### Security configuration

``` java
@Configuration
public class SecurityConfiguration {
    @Resource
    JwtAuthorizeFilter jwtAuthorizeFilter;
    @Resource
    JwtUtils utils;

    @Resource
    AccountService service;

    @Bean
  public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
      return http
              // 配置请求授权规则
              .authorizeHttpRequests(conf -> conf
                      // 对 "/api/auth/**" 路径的请求允许无需身份验证
                      .requestMatchers("/api/auth/**").permitAll()
                      // 其他任何请求需要身份验证
                      .anyRequest().authenticated()
              )
              // 配置表单登录
              .formLogin(conf -> conf
                      // 登录处理的 URL
                      .loginProcessingUrl("/api/auth/login")
                      // 登录成功时的处理器
                      .successHandler(this::onAuthenticationSuccess)
                      // 登录失败时的处理器
                      .failureHandler(this::onAuthenticationFailure)
              )
              // 配置登出
              .logout(conf -> conf
                      // 登出处理的 URL
                      .logoutUrl("/api/auth/logout")
                      // 登出成功时的处理器
                      .logoutSuccessHandler(this::onLogoutSuccess)
              )
              // 配置异常处理
              .exceptionHandling(conf -> conf
                      // 未经授权时的处理器
                      .authenticationEntryPoint(this::onUnauthorized)
                      // 访问被拒绝时的处理器
                      .accessDeniedHandler(this::onAccessDeny)
              )
              // 禁用 CSRF
              .csrf(AbstractHttpConfigurer::disable)
              // 配置会话管理
              .sessionManagement(conf -> conf
                      // 不创建会话，即无状态的身份验证
                      .sessionCreationPolicy(SessionCreationPolicy.STATELESS)
              )
              // 在 UsernamePasswordAuthenticationFilter 之前添加自定义 JWT 鉴权过滤器
              .addFilterBefore(jwtAuthorizeFilter, UsernamePasswordAuthenticationFilter.class)
              .build();
  }

    public void onAuthenticationSuccess(HttpServletRequest request, HttpServletResponse response, Authentication authentication) throws IOException, ServletException {
      // 设置响应内容类型为 JSON
      response.setContentType("application/json");
      // 设置响应字符编码为 UTF-8
      response.setCharacterEncoding("UTF-8");
      // 从身份验证对象中获取用户信息
      User user = (User) authentication.getPrincipal();
      // 根据用户名或邮箱查找对应的账户信息
      Account account = service.findAccountByUsernameOrEamil(user.getUsername());
      // 创建 JWT 令牌并获取相应的令牌字符串
      String token = utils.createJWT(user, account.getUsername(), account.getId());
      // 创建 AuthorizeVO 对象，用于构建授权信息的响应
      AuthorizeVO vo = new AuthorizeVO();
      // 设置令牌过期时间
      vo.setExpire(utils.expireTime());
      // 设置用户角色
      vo.setRole(account.getRole());
      // 设置令牌字符串
      vo.setToken(token);
      // 设置用户名
      vo.setUsername(account.getUsername());
      // 将授权信息对象转换为 JSON 字符串并写入响应
      response.getWriter().write(RestBean.success(vo).asJsonString());
  }

	public void onLogoutSuccess(HttpServletRequest request, HttpServletResponse response, Authentication authentication) throws IOException, ServletException {
          response.setContentType("application/json");
          response.setCharacterEncoding("UTF-8");
          PrintWriter writer = response.getWriter();
          String authorization = request.getHeader("Authorization");
          if (utils.invalidateJwt(authorization)){
              writer.write(RestBean.success("注销成功").asJsonString());
              return;
          }
          writer.write(RestBean.failure(400,"注销失败").asJsonString());
  
  public void onAccessDeny(HttpServletRequest request, HttpServletResponse response, AccessDeniedException exception) throws IOException, ServletException {
          response.setContentType("application/json");
          response.setCharacterEncoding("UTF-8");
          response.getWriter().write(RestBean.failure(403,exception.getMessage()).asJsonString());
      }

      public void onUnauthorized(HttpServletRequest request, HttpServletResponse response, AuthenticationException exception) throws IOException, ServletException {
          response.setContentType("application/json");
          response.setCharacterEncoding("UTF-8");
          response.getWriter().write(RestBean.failure(401,exception.getMessage()).asJsonString());
      }

      public void onAuthenticationFailure(HttpServletRequest request, HttpServletResponse response, AuthenticationException exception) throws IOException, ServletException {
          response.setContentType("application/json");
          response.setCharacterEncoding("UTF-8");
          response.getWriter().write(RestBean.failure(401,exception.getMessage()).asJsonString());
      }
    }

```



### JwtAuthorizeFilter（请求添加filter解析验证JWT）

``` java
@Component
public class JwtAuthorizeFilter extends OncePerRequestFilter {
    // 自定义的过滤器类，继承自 Spring Security 的 OncePerRequestFilter 类
    @Resource
    JwtUtils utils;

    @Override
    protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain filterChain) throws ServletException, IOException {
        // 实现过滤器的核心逻辑，在每个请求上执行一次
        String authorization = request.getHeader("Authorization");
        // 从请求头中获取名为 "Authorization" 的值，通常是包含 JWT 的身份验证标头
        DecodedJWT jwt = utils.resolveJWT(authorization);
        // 使用 JwtUtils 的 resolveJWT 方法解析 JWT，将 JWT 字符串解码为 DecodedJWT 对象
        if (jwt != null) {
            // 如果成功解析 JWT
            UserDetails user = utils.toUser(jwt);
            // 使用 JwtUtils 的 toUser 方法从 JWT 中提取用户信息并创建 UserDetails 对象
            UsernamePasswordAuthenticationToken authenticationToken = new UsernamePasswordAuthenticationToken(user, null, user.getAuthorities());
            // 创建一个 UsernamePasswordAuthenticationToken 对象，将用户信息和权限传递给 Spring Security 进行认证
            authenticationToken.setDetails(new WebAuthenticationDetailsSource().buildDetails(request));
            // 设置认证详细信息，包括 IP 地址和会话 ID 等
            SecurityContextHolder.getContext().setAuthentication(authenticationToken);
            // 将认证对象放入 Spring Security 的安全上下文中，表示用户已通过认证
            request.setAttribute("id", jwt.getClaim("id").asInt());
            // 将 JWT 中的 "id" 声明作为属性添加到当前请求，以便其他组件使用
        }
        filterChain.doFilter(request, response);
        // 继续处理请求链，将请求和响应传递给下一个过滤器或资源
    }
}

```

### JWT 签发验证等相关处理方法

JwtUtils.java

``` java
@Component
public class JwtUtils {
    @Value("${spring.security.jwt.key}")
    String key;

    @Value("${spring.security.jwt.expire}")
    int expire;

    @Resource
    StringRedisTemplate template;

    public boolean invalidateJwt(String headerToken){
        String token = this.convertToken(headerToken);
        if (token == null) {
            return false;
        }
        Algorithm algorithm = Algorithm.HMAC256(key);
        JWTVerifier jwtVerifier = JWT.require(algorithm).build();
        try {
            DecodedJWT jwt = jwtVerifier.verify(token);
            String id = jwt.getId();
            return deleteToken(id, jwt.getExpiresAt());
        }catch (JWTVerificationException e){
            return false;
        }
    }
    private boolean deleteToken(String uuid, Date time){
        if (isInvalidateToken(uuid)) {
            return false;
        }
        long expire = Math.max(time.getTime() - System.currentTimeMillis(), 0);
        template.opsForValue().set(Const.JWT_BLACK_LIST + uuid, "", expire, TimeUnit.MILLISECONDS);
        return true;
    }

    private boolean isInvalidateToken(String uuid){
        return Boolean.TRUE.equals(template.hasKey(Const.JWT_BLACK_LIST + uuid));
    }



    public String createJWT(UserDetails details,String username, int userId){
        Algorithm algorithm = Algorithm.HMAC256(key);
        Date expire = expireTime();
        return JWT.create()
                .withJWTId(UUID.randomUUID().toString())
                .withClaim("id",userId)
                .withClaim("name",username)
                .withClaim("authorities", details.getAuthorities()
                        .stream()
                        .map(GrantedAuthority::getAuthority).toList()
                )
                .withExpiresAt(expire)
                .withIssuedAt(new Date())
                .sign(algorithm);
    }


    public DecodedJWT resolveJWT(String headerToken){
        String token = this.convertToken(headerToken);
        if(token == null) {
            return null;
        }
        Algorithm algorithm = Algorithm.HMAC256(key);
        JWTVerifier jwtVerifier = JWT.require(algorithm).build();
        try {
            DecodedJWT verify = jwtVerifier.verify(token);
            if (isInvalidateToken(verify.getId())) {
                return null;
            }
            Date expireAT = verify.getExpiresAt();
            return new Date().after(expireAT) ? null : verify;

        } catch (JWTVerificationException e){
            return null;
        }

    }

    public UserDetails toUser(DecodedJWT jwt){
        Map<String, Claim> claims = jwt.getClaims();
        return User
                .withUsername(claims.get("name").asString())
                .password("******")
                .authorities(claims.get("authorities").asArray(String.class))
                .build();
    }

    private String convertToken(String headerToken){
        if (headerToken == null || !headerToken.startsWith("Bearer ")) {
            return null;
        }
        return headerToken.replace("Bearer ","");
    }

    public Date expireTime(){
        Calendar calendar = Calendar.getInstance();
        calendar.add(Calendar.HOUR, expire*24);
        return calendar.getTime();
    }
}
```

### WebConfiguration

配置一个加密算法，框架自动使用该算法对前端传过来的密码进行加密，然后与数据库中的已经加密的密码对比

单独写一个配置类是因为防止循环引用，如service类需使用某个配置类的加密算法，而该配置类中又有service

``` java
@Configuration
public class WebConfiguration {

    @Bean
    BCryptPasswordEncoder bCryptPasswordEncoder(){//密码加密算法，框架自动使用
        return new BCryptPasswordEncoder();
    }

}
```

``` java
        System.out.println(new BCryptPasswordEncoder().encode("123456"));
打印加密密码
```



### RestBean(Api模版)

``` java
public record RestBean<T>(int code, T data, String message){
    public static <T> RestBean<T> success(T data){
        return new RestBean<>(200,data,"success");
    }
    public static <T> RestBean<T> success(){
        return success(null);
    }
    public static <T> RestBean<T> failure(int code, String message){
        return new RestBean<>(code,null,message);
    }
    public static <T> RestBean<T> forbidden(String message){
        return failure(403, message);
    }
    public static <T> RestBean<T> unauthorized(String message){
        return failure(401, message);
    }

  
    public String asJsonString(){
        return JSONObject.toJSONString(this, JSONWriter.Feature.WriteNulls);
    }

}

```

### AccountService（实现数据库数据验证用户）

AccountService

``` java
public interface AccountService extends IService<Account>, UserDetailsService { //继承UserDetailsService接口实现自定义验证
    public Account findAccountByUsernameOrEamil(String text);
}

```

AccountServiceImpl

``` java
@Service
public class AccountServiceImpl extends ServiceImpl<AccountMapper, Account> implements AccountService {
    // 实现 UserDetailsService 接口的方法，通过用户名加载用户信息
    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        // 调用自定义方法查找用户信息
        Account account = this.findAccountByUsernameOrEmail(username);
        // 如果找不到用户，抛出 UsernameNotFoundException 异常
        if (account == null) {
            throw new UsernameNotFoundException("用户名不存在");
        }
        // 构建并返回 Spring Security 的 UserDetails 对象
        return User
                .withUsername(username)
                .password(account.getPassword())
                .roles(account.getRole())
                .build();
    }
    // 自定义方法，通过用户名或邮箱查找用户信息
    public Account findAccountByUsernameOrEmail(String text) {
        // 使用 MyBatis Plus 的 query 方法来构建查询条件
        return this.query()
                .eq("username", text).or()
                .eq("email", text)
                .one(); // 查询单个结果
    }
}
```



### CorsFilter （实现跨域请求）

``` java
@Component
@Order(Const.ORDER_CORS)
public class CorsFilter extends HttpFilter {
    @Override
    protected void doFilter(HttpServletRequest request, HttpServletResponse response, FilterChain chain) throws IOException, ServletException {


        this.addCorsHeader(request,response);
        chain.doFilter(request,response);
    }

    private void addCorsHeader(HttpServletRequest request, HttpServletResponse response){

        response.addHeader("Access-Control-Allow-Origin", request.getHeader("Origin"));
        response.addHeader("Access-Control-Allow-Methods", "GET,POST,PUT,DELETE,OPTIONS");
        response.addHeader("Access-Control-Allow-Headers", "Content-Type, Authorization");


    }


} 
```





## 接口校验框架

配置

``` xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-validation</artifactId>
</dependency>
```

使用注解可以直接校验

```java
@Slf4j
@Validated   //首先在Controller上开启接口校验
@Controller
public class TestController {

    ...

    @ResponseBody
    @PostMapping("/submit")
    public String submit(@Length(min = 3) String username,  //使用@Length注解一步到位
                         @Length(min = 10) String password){
        System.out.println(username.substring(3));
        System.out.println(password.substring(2, 10));
        return "请求成功!";
    }
}
```

写一个异常处理Controller可以处理检验抛出的异常

``` java
@Slf4j
@RestControllerAdvice
public class ValidationController {

    @ExceptionHandler(ValidationException.class)
    public RestBean<Void> validationException(ValidationException exception){
        log.warn("Resolved [{}: {}]", exception.getClass().getName(), exception.getMessage());
        return RestBean.failure(400,"参数错误");
    }

}
```

常用注解

|   验证注解   |                        验证的数据类型                        |                           说明                           |
| :----------: | :----------------------------------------------------------: | :------------------------------------------------------: |
| @AssertFalse |                       Boolean,boolean                        |                      值必须是false                       |
| @AssertTrue  |                       Boolean,boolean                        |                       值必须是true                       |
|   @NotNull   |                           任意类型                           |                       值不能是null                       |
|    @Null     |                           任意类型                           |                       值必须是null                       |
|     @Min     | BigDecimal、BigInteger、byte、short、int、long、double 以及任何Number或CharSequence子类型 |                   大于等于@Min指定的值                   |
|     @Max     |                             同上                             |                   小于等于@Max指定的值                   |
| @DecimalMin  |                             同上                             |         大于等于@DecimalMin指定的值（超高精度）          |
| @DecimalMax  |                             同上                             |         小于等于@DecimalMax指定的值（超高精度）          |
|   @Digits    |                             同上                             |                限制整数位数和小数位数上限                |
|    @Size     |               字符串、Collection、Map、数组等                |       长度在指定区间之内，如字符串长度、集合大小等       |
|    @Past     |       如 java.util.Date, java.util.Calendar 等日期类型       |                    值必须比当前时间早                    |
|   @Future    |                             同上                             |                    值必须比当前时间晚                    |
|  @NotBlank   |                     CharSequence及其子类                     |         值不为空，在比较时会去除字符串的首位空格         |
|   @Length    |                     CharSequence及其子类                     |                  字符串长度在指定区间内                  |
|  @NotEmpty   |         CharSequence及其子类、Collection、Map、数组          | 值不为null且长度不为空（字符串长度不为0，集合大小不为0） |
|    @Range    | BigDecimal、BigInteger、CharSequence、byte、short、int、long 以及原子类型和包装类型 |                      值在指定区间内                      |
|    @Email    |                     CharSequence及其子类                     |                     值必须是邮件格式                     |
|   @Pattern   |                     CharSequence及其子类                     |               值需要与指定的正则表达式匹配               |
|    @Valid    |                        任何非原子类型                        |                     用于验证对象属性                     |





当参数是对象时

``` java
@ResponseBody
@PostMapping("/submit")  
public String submit(@Valid Account account){//在参数上添加@Valid注解表示需要验证
    System.out.println(account.getUsername().substring(3));
    System.out.println(account.getPassword().substring(2, 10));
    return "请求成功!";
}
```



``` java
@Data
public class Account {
    @Length(min = 3)   //只需要在对应的字段上添加校验的注解即可
    String username;
    @Length(min = 10)
    String password;
}
```



## 邮件发送模块

依赖

``` xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-mail</artifactId>
</dependency>
```

``` yml
spring:
  mail:
      # 163邮箱的地址为smtp.163.com，直接填写即可
    host: smtp.163.com
    # 你申请的163邮箱
    username: javastudy111@163.com
    # 注意密码是在开启smtp/pop3时自动生成的，记得保存一下，不然就找不到了
    password: AZJTOAWZESLMHTNI
```

调用

``` java
@SpringBootTest
class SpringBootTestApplicationTests {

      //JavaMailSender是专门用于发送邮件的对象，自动配置类已经提供了Bean
    @Autowired
    JavaMailSender sender;

    @Test
    void contextLoads() {
          //SimpleMailMessage是一个比较简易的邮件封装，支持设置一些比较简单内容
        SimpleMailMessage message = new SimpleMailMessage();
          //设置邮件标题
        message.setSubject("【电子科技大学教务处】关于近期学校对您的处分决定");
          //设置邮件内容
        message.setText("XXX同学您好，经监控和教务巡查发现，您近期存在旷课、迟到、早退、上课刷抖音行为，" +
                "现已通知相关辅导员，请手写5000字书面检讨，并在2022年4月1日17点前交到辅导员办公室。");
          //设置邮件发送给谁，可以多个，这里就发给你的QQ邮箱
        message.setTo("你的QQ号@qq.com");
          //邮件发送者，这里要与配置文件中的保持一致
        message.setFrom("javastudy111@163.com");
          //OK，万事俱备只欠发送
        sender.send(message);
    }

}
```

项目中使用消息队列来进行邮件发送



## 消息队列

什么是MQ

message queue 意思就是消息队列

什么是AMQP?

AMQP(Advanced Message Queuing Protocol,高级消息队列协议)是进程之间传递异步消息的网络协议。



**RabbitMQ**是消息队列具体实现

依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-amqp</artifactId>
</dependency>
```

配置RabbitMQ信息：

```yaml
spring:
  rabbitmq:
    addresses: 192.168.0.4
    username: admin
    password: admin
    virtual-host: /test
```

声明一个消息队列，我们只需要一个配置类就行了：

```java
@Configuration
public class RabbitConfiguration {
    @Bean("directExchange")  //定义交换机Bean，可以很多个
    public Exchange exchange(){
        return ExchangeBuilder.directExchange("amq.direct").build();
    }

    @Bean("yydsQueue")     //定义消息队列
    public Queue queue(){
        return QueueBuilder
          				.nonDurable("yyds")   //非持久化类型
          				.build();
    }

    @Bean("binding")
    public Binding binding(@Qualifier("directExchange") Exchange exchange,
                           @Qualifier("yydsQueue") Queue queue){
      	//将我们刚刚定义的交换机和队列进行绑定
        return BindingBuilder
                .bind(queue)   //绑定队列
                .to(exchange)  //到交换机
                .with("my-yyds")   //使用自定义的routingKey
                .noargs();
    }
}
```

接着我们来创建一个生产者，这里我们直接编写在测试用例中：

```java
@SpringBootTest
class SpringCloudMqApplicationTests {

  	//RabbitTemplate为我们封装了大量的RabbitMQ操作，已经由Starter提供，因此直接注入使用即可
    @Resource
    RabbitTemplate template;												//AmqpTemplate template也可以

    @Test
    void publisher() {
      	//使用convertAndSend方法一步到位，参数基本和之前是一样的
      	//最后一个消息本体可以是Object类型，真是大大的方便
        template.convertAndSend("amq.direct", "my-yyds", "Hello World!");//Exchange交换机，routingKey，消息
    }

}
```

现在我们再来看看如何创建一个消费者，因为消费者实际上就是一直等待消息然后进行处理的角色，这里我们只需要创建一个监听器就行了，它会一直等待消息到来然后再进行处理：

```java
@Component  //注册为Bean
@RabbitListener(queues = "yyds")
public class TestListener {

    @RabbitHandler  //用于定义一个方法，该方法会在接收到特定队列的消息时被调用
    public void test(Message message){
        System.out.println(new String(message.getBody()));
    }
}
```



## 限流

FlowLimitingFilter

```java
@Slf4j
@Component
@Order(Const.ORDER_FLOW_LIMIT)
public class FlowLimitingFilter extends HttpFilter {
    @Resource
    StringRedisTemplate template;

    @Override
    protected void doFilter(HttpServletRequest request, HttpServletResponse response, FilterChain chain) throws IOException, ServletException {
        // 获取请求源地址（IP）
        String address = request.getRemoteAddr();
        
        // 尝试对请求源地址进行计数
        if (tryCount(address))
            chain.doFilter(request, response); // 放行请求
        else
            this.writeBlockMessage(response); // 返回请求过于频繁的错误消息
    }

    // 返回请求过于频繁的错误消息
    private void writeBlockMessage(HttpServletResponse response) throws IOException {
        response.setContentType("application/json;charset=UTF-8");
        response.setStatus(HttpServletResponse.SC_FORBIDDEN); // 设置HTTP响应状态为403 Forbidden
        response.getWriter().write(RestBean.forbidden("请求过于频繁，请稍后再试").asJsonString());
    }

    // 尝试对指定IP地址进行计数
    private boolean tryCount(String ip) {
        synchronized (ip.intern()) {
            // 检查是否已经被阻塞
            if (Boolean.TRUE.equals(template.hasKey(Const.Flow_LIMIT_BLOCK + ip))) {
                return false;
            }
            
            // 进行时间段内的频率限制检查
            return this.limitPeriodCheck(ip);
        }
    }
    
    // 在一定时间段内对指定IP地址的请求进行频率限制检查
    private boolean limitPeriodCheck(String ip) {
        if (Boolean.TRUE.equals(template.hasKey(Const.Flow_LIMIT_COUNTER + ip))) {
            // 检查计数器是否存在，若存在则进行递增操作
            long increment = Optional.ofNullable(template.opsForValue().increment(Const.Flow_LIMIT_COUNTER + ip, 1)).orElse(0L);
            if (increment > 10) {
                // 如果超过限制次数，则阻塞IP一段时间
                template.opsForValue().set(Const.Flow_LIMIT_BLOCK + ip, "", 30, TimeUnit.SECONDS);
                return false;
            }
        } else {
            // 如果计数器不存在，则初始化为1，并设置过期时间
            template.opsForValue().set(Const.Flow_LIMIT_COUNTER + ip, "1", 30, TimeUnit.SECONDS);
        }
        return true;
    }
}

```



## redis

依赖

``` xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```

配置

``` yml
spring:
  redis:
  	#Redis服务器地址
    host: 192.168.10.3
    #端口
    port: 6379
    #使用几号数据库
    database: 0
```

使用

``` java
@SpringBootTest
class SpringBootTestApplicationTests {

    @Autowired
    StringRedisTemplate template;

    @Test
    void contextLoads() {
        ValueOperations<String, String> operations = template.opsForValue();
        operations.set("c", "xxxxx");   //设置值
        System.out.println(operations.get("c"));   //获取值
      	
        template.delete("c");    //删除键
        System.out.println(template.hasKey("c"));   //判断是否包含键
    }

}
```



## mybatis-plus

依赖

``` xml
<dependency>
     <groupId>com.baomidou</groupId>
     <artifactId>mybatis-plus-boot-starter</artifactId>
     <version>3.5.3.1</version>
</dependency>
<dependency>
     <groupId>com.mysql</groupId>
     <artifactId>mysql-connector-j</artifactId>
</dependency>
```

配置

``` yml
spring:
  datasource:
    url: jdbc:mysql://localhost:3306/test
    username: root
    password: 123456
    driver-class-name: com.mysql.cj.jdbc.Driver
```

### 实体类 映射表

``` java
@Data //lombok注解，生成一个javabean
@TableName("user")  //对应的表名
public class User {
    @TableId(type = IdType.AUTO)   //对应的主键
    int id;
    @TableField("name")   //对应的字段
    String name;
    @TableField("email")
    String email;
    @TableField("password")
    String password;
}
```



### CRUD操作的两个接口

#### BaseMapper<T>

mapper层（DAO）

``` java
@Mapper
public interface UserMapper extends BaseMapper<User> {
  	//使用方式与JPA极其相似，同样是继承一个基础的模版Mapper
  	//这个模版里面提供了预设的大量方法直接使用，跟JPA如出一辙
}
```

Service层

```java
@SpringBootTest
class DemoApplicationTests {

    @Resource
    UserMapper mapper;

    @Test
    void contextLoads() {
        System.out.println(mapper.selectById(1));  //同样可以直接selectById，非常快速方便
    }
}
```

对于一些复杂查询的情况，MybatisPlus支持我们自己构造QueryWrapper（条件构造器）用于复杂条件查询：

``` java
@Test
void contextLoads() {
    QueryWrapper<User> wrapper = new QueryWrapper<>();    //复杂查询可以使用QueryWrapper来完成
  	wrapper
            .select("id", "name", "email", "password")    //可以自定义选择哪些字段
            .ge("id", 2)     			//选择判断id大于等于1的所有数据
            .orderByDesc("id");   //根据id字段进行降序排序
    System.out.println(mapper.selectList(wrapper));   //Mapper同样支持使用QueryWrapper进行查询
}

```



#### IService<T>

mapper层（DAO）

``` java
@Mapper
public interface UserMapper extends BaseMapper<User> {
  	//使用方式与JPA极其相似，同样是继承一个基础的模版Mapper
  	//这个模版里面提供了预设的大量方法直接使用，跟JPA如出一辙
}
```



写一个接口继承IService<T>

``` java
@Service
public interface UserService extends IService<User> {
  	//除了继承模版，我们也可以把它当成普通Service添加自己需要的方法
}
```

接着我们还需要编写一个实现类，这个实现类就是UserService的实现：

``` java
@Service   //需要继承ServiceImpl才能实现那些默认的CRUD方法
public class UserServiceImpl extends ServiceImpl<UserMapper, User> implements UserService {
  //可以使用模版的方法，例：
  
  
  //public Account findAccountByUsernameOrEamil(String text) {
  //      return this.query() //链式查询
  //              .eq("username", text).or()
  //              .eq("email", text).one();

    }
}
```

其他地方调用service

``` java
@Resource
AccountService service;

@Test
void contextLoads() {
    User one = service.query().eq("id", 1).one();
    System.out.println(one);
}
```



使用IService接口，可以封装了更复杂的业务逻辑，可以将数据库操作与业务逻辑解耦，使代码更具可读性和可维护性。效果一样。。

使用 `IService` 接口相比直接使用 `BaseMapper` 有以下优点：

- **代码抽象层次更高**：在 `UserService` 中可以定义更高层次的业务逻辑，使控制器中的代码更加简洁。
- **统一管理**：通过 `IService` 统一管理 CRUD 操作，使得业务代码更加规范化。
- **易于维护和扩展**：当需要添加新的业务逻辑时，只需在服务层进行扩展，而无需修改现有的数据访问代码，提高了代码的可维护性和扩展性。

这样的设计使得业务逻辑与数据访问逻辑分离，提高了代码的可读性和维护性，同时也使得单元测试更加方便。





### 分页插件

#### 配置方法

```java
@Configuration
@MapperScan("scan.your.mapper.package")
public class MybatisPlusConfig {

    /**
     * 添加分页插件
     */
    @Bean
    public MybatisPlusInterceptor mybatisPlusInterceptor() {
        MybatisPlusInterceptor interceptor = new MybatisPlusInterceptor();
        interceptor.addInnerInterceptor(new PaginationInnerInterceptor(DbType.MYSQL));//如果配置多个插件,切记分页最后添加
        //interceptor.addInnerInterceptor(new PaginationInnerInterceptor()); 如果有多数据源可以不配具体类型 否则都建议配上具体的DbType
        return interceptor;
    }
}
```

#### 使用

``` java
public IPage<Article> getFrontArticles(Integer current, Integer limit) {

        Page<Article> page = new Page<>(current, limit);//创建page对象，第一个参数为当前页数，第二个参数为每页的数量
        LambdaQueryWrapper<Article> wrapper = new QueryWrapper<Article>().lambda()
                .eq(Article::getStatus, PostStatusEnum.PUBLISHED)
                .eq(Article::getType, Types.POST)
                .orderByDesc(Article::getPriority, Article::getCreateTime);
        IPage<Article> result = articleMapper.selectPage(page, wrapper);//得到ipage对象

        result.getRecords().forEach(article -> {
            String content = DiceUtil.contentTransform(article.getContent(), true, true);
            article.setContent(content);
        });

        return result;
    }
```



#### Page对象

> 该类继承了 `IPage` 类，实现了 `简单分页模型` 如果你要实现自己的分页模型可以继承 `Page` 类或者实现 `IPage` 类

|         属性名         |  类型   |  默认值   |                             描述                             |
| :--------------------: | :-----: | :-------: | :----------------------------------------------------------: |
|        records         |  List   | emptyList |                         查询数据列表                         |
|         total          |  Long   |     0     |                       查询列表总记录数                       |
|          size          |  Long   |    10     |                   每页显示条数，默认 `10`                    |
|        current         |  Long   |     1     |                            当前页                            |
|         orders         |  List   | emptyList | 排序字段信息，允许前端传入的时候，注意 SQL 注入问题，可以使用 `SqlInjectionUtils.check(...)` 检查文本 |
|    optimizeCountSql    | boolean |   true    | 自动优化 COUNT SQL 如果遇到 `jSqlParser` 无法解析情况，设置该参数为 `false` |
| optimizeJoinOfCountSql | boolean |   true    |         自动优化 COUNT SQL 是否把 join 查询部分移除          |
|      searchCount       | boolean |   true    | 是否进行 count 查询，如果只想查询到列表不要查询总记录数，设置该参数为 `false` |
|        maxLimit        |  Long   |           |                       单页分页条数限制                       |
|        countId         | String  |           | `xml` 自定义 `count` 查询的 `statementId` 也可以不用指定在分页 `statementId` 后面加上 `_mpCount` 例如分页 `selectPageById` 指定 count 的查询 `statementId` 设置为 `selectPageById_mpCount` 即可默认找到该 `SQL` 执行 |
