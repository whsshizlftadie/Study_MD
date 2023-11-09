# Spring Security学习

## 引入依赖

```xml
	<dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-security</artifactId>
    </dependency>
	<dependency>
         <groupId>cn.hutool</groupId>
         <artifactId>hutool-jwt</artifactId>
     </dependency>
```



## 单体应用解决方案1

### 登录

```java
@RestController
@RequestMapping("/user")
public class AuthController {

    @Autowired
    AuthenticationManager authenticationManager;

...

    @PostMapping("/login")
    public String login(@RequestBody SignInReq req) {

        UsernamePasswordAuthenticationToken authenticationToken =
                new UsernamePasswordAuthenticationToken(req.getUsername(), req.getPassword());
        authenticationManager.authenticate(authenticationToken);

         //上一步没有抛出异常说明认证成功，我们向用户颁发jwt令牌
        String token = JWT.create()
                .setPayload("username", req.getUsername())
                .setKey(MyConstant.JWT_SIGN_KEY.getBytes(StandardCharsets.UTF_8))
                .sign();

        return token;
    }
}
```

这里`AuthenticationManager `注入的是默认实现`ProviderManager`实例。`UsernamePasswordAuthenticationToken `是一个`Authentication`，我们构建一个`Authentication`然后交给`AuthenticationManager `去校验。一定要注意这里使用的是**两个参数的构造方法**，它将认证状态设置为了false，接着就需要让`AuthenticationManager `去校验用户名和秘密。

```java
public UsernamePasswordAuthenticationToken(Object principal, Object credentials) {
        ...
        this.setAuthenticated(false);
    }
```

### 拦截请求，校验token

```java
public class JwtAuthenticationTokenFilter extends OncePerRequestFilter {
    private final static String AUTH_HEADER = "Authorization";
    private final static String AUTH_HEADER_TYPE = "Bearer";

    @Autowired
    private UserDetailsService userDetailsService;

    @Override
    protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain filterChain) throws ServletException, IOException {
        // get token from header:  Authorization: Bearer <token>
        String authHeader = request.getHeader(AUTH_HEADER);
        if (Objects.isNull(authHeader) || !authHeader.startsWith(AUTH_HEADER_TYPE)){
            filterChain.doFilter(request,response);
            return;
        }

        String authToken = authHeader.split(" ")[1];
        log.info("authToken:{}" , authToken);
        //verify token
        if (!JWTUtil.verify(authToken, MyConstant.JWT_SIGN_KEY.getBytes(StandardCharsets.UTF_8))) {
            log.info("invalid token");
            filterChain.doFilter(request,response);
            return;
        }

        final String userName = (String) JWTUtil.parseToken(authToken).getPayload("username");
        UserDetails userDetails = userDetailsService.loadUserByUsername(userName);

        // 注意，这里使用的是3个参数的构造方法，此构造方法将认证状态设置为true
        UsernamePasswordAuthenticationToken authentication =
                new UsernamePasswordAuthenticationToken(userDetails.getUsername(), userDetails.getPassword(), userDetails.getAuthorities());
        authentication.setDetails(new WebAuthenticationDetailsSource().buildDetails(request));

        //将认证过了凭证保存到security的上下文中以便于在程序中使用
        SecurityContextHolder.getContext().setAuthentication(authentication);

        filterChain.doFilter(request, response);
    }
}
```

`JwtAuthenticationTokenFilter`继承 `OncePerRequestFilter`，其会拦截http请求，然后检查其header: `Authorization`携带的 jwt。如果通过了就从jwt中获取用户名，然后到数据库（或者redis缓存等）里查询用户信息，然后生成验证通过的`UsernamePasswordAuthenticationToken `。**一定要注意这次使用的是3个参数的构造函数，其将认证状态设置为了true**。

```java
 public UsernamePasswordAuthenticationToken(Object principal, Object credentials,
            Collection<? extends GrantedAuthority> authorities) {
          ...
        super.setAuthenticated(true); // must use super, as we override
    }
```

### 配置security

```java
@Configuration
@EnableWebSecurity
public class WebSecurityConfig extends WebSecurityConfigurerAdapter{
    ...

    @Bean
    public AuthenticationManager authenticationManager(AuthenticationConfiguration authConfig) throws Exception {
        return authConfig.getAuthenticationManager();
    }

    //我们自定义的拦截器
    @Bean
    public JwtAuthenticationTokenFilter jwtAuthenticationTokenFilter() {
        return new JwtAuthenticationTokenFilter();
    }

...
    @Bean
    public SecurityFilterChain filterChain(HttpSecurity httpSecurity) throws Exception {
        //基于token，所以不需要csrf防护
        httpSecurity.csrf().disable()
                //基于token，所以不需要session
                .sessionManagement().sessionCreationPolicy(SessionCreationPolicy.STATELESS)
                .and()
                .authorizeRequests()
                //登录注册不需要认证
                .antMatchers("/user/login", "/user/register").permitAll()
                //除上面的所有请求全部需要鉴权认证
                .anyRequest()
                .authenticated();
        //禁用缓存
        httpSecurity.headers().cacheControl();
        //将我们的JWT filter添加到UsernamePasswordAuthenticationFilter前面，因为这个Filter是authentication开始的filter，我们要早于它
        httpSecurity.addFilterBefore(jwtAuthenticationTokenFilter(), UsernamePasswordAuthenticationFilter.class);

        ...

        return httpSecurity.build();
    }
}
```

### 自定义Authentication Provider

具体的认证过程就发生在provider中，就是那个比较用户提交的凭证和程序保存凭证的过程。默认使用的是`DaoAuthenticationProvider`，一般情况都够用了。

```java
public class JwtAuthenticationProvider implements AuthenticationProvider {

    @Autowired
    private  PasswordEncoder passwordEncoder;
    @Autowired
    private UserDetailsService userDetailsService;

    @Override
    public Authentication authenticate(Authentication authentication) throws AuthenticationException {
        String username = String.valueOf(authentication.getPrincipal());
        String password = String.valueOf(authentication.getCredentials());

        UserDetails userDetails = userDetailsService.loadUserByUsername(username);
        if(passwordEncoder.matches(password,userDetails.getPassword())){
           return new UsernamePasswordAuthenticationToken(username, password, userDetails.getAuthorities());
        }

        throw new BadCredentialsException("Error!!");
    }

    @Override
    public boolean supports(Class<?> authentication) {
        return UsernamePasswordAuthenticationToken.class.equals(authentication);
    }
}
```

值的注意的是我这里仍然使用了`UserDetailsService `这个Spring Security提供的接口来获取程序保存的用户凭证。我们这里也可以不使用它，而直接使用我们自己定义的类，例如自己写个查询用户信息的service即可。

#### 修改配置

```java
@Configuration
@EnableWebSecurity
public class WebSecurityConfig {

    @Bean
    public AuthenticationManager authenticationManager(AuthenticationConfiguration authConfig) throws Exception {
        return authConfig.getAuthenticationManager();
    }

    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }

    @Bean
    public JwtAuthenticationTokenFilter jwtAuthenticationTokenFilter() {
        return new JwtAuthenticationTokenFilter();
    }

    @Bean
    public JwtAuthenticationProvider jwtAuthenticationProvider(){
        return new JwtAuthenticationProvider();
    }

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity httpSecurity) throws Exception {
        //由于使用的是JWT，这里不需要csrf防护
        httpSecurity.csrf().disable()
                //基于token，所以不需要session
                .sessionManagement().sessionCreationPolicy(SessionCreationPolicy.STATELESS)
                .and()
                .authorizeRequests()
                //对登录注册允许匿名访问
                .antMatchers("/user/login", "/user/register").permitAll()
                .anyRequest()// 除上面外的所有请求全部需要鉴权认证
                .authenticated();
        //禁用缓存
        httpSecurity.headers().cacheControl();
        //使用自定义provider
        httpSecurity.authenticationProvider(jwtAuthenticationProvider());
        //添加JWT filter
        httpSecurity.addFilterBefore(jwtAuthenticationTokenFilter(), UsernamePasswordAuthenticationFilter.class);
        ...

        return httpSecurity.build();
    }
}
```

### 认证失败处理

```java
@Component
public class MyUnauthorizedHandler implements AuthenticationEntryPoint {
    @Override
    public void commence(HttpServletRequest request, HttpServletResponse response, AuthenticationException authException) throws IOException, ServletException {
        response.setCharacterEncoding("UTF-8");
        response.setContentType("application/json");
        response.getWriter().println("认证失败");
        response.getWriter().flush();
    }
}


...
    
将其配置到config中
    
...
public class WebSecurityConfig {

    @Autowired
    private MyUnauthorizedHandler unauthorizedHandler;
    ...

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity httpSecurity) throws Exception {
        ...
        httpSecurity.exceptionHandling()
                .authenticationEntryPoint(unauthorizedHandler);

        return httpSecurity.build();
    }
}
```

### 授权失败处理

```java
@Component
public class MyAccessDeniedHandler implements AccessDeniedHandler {
    @Override
    public void handle(HttpServletRequest request, HttpServletResponse response, AccessDeniedException accessDeniedException) throws IOException, ServletException {
        response.setCharacterEncoding("UTF-8");
        response.setContentType("application/json");
        response.getWriter().println("禁止访问");
        response.getWriter().flush();
    }
}

...
    
将其配置到config中
    
...
   ...
public class WebSecurityConfig {

    @Autowired
    private MyAccessDeniedHandler accessDeniedHandler;

    ...

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity httpSecurity) throws Exception {
        ...
        httpSecurity.exceptionHandling()
                .accessDeniedHandler(accessDeniedHandler);

        return httpSecurity.build();
    }
}
```

### 支持方法级别的授权

在web配置类添加一个注解，如下所示。

```
@EnableGlobalMethodSecurity(prePostEnabled = true)
public class WebSecurityConfig {
    ...
}
```

然后在controller里使用`@PreAuthorize`注解，里面使用spring表达式(SpEL)来设置条件，例如下面的api只允许admin角色方法。

```java
@PreAuthorize("hasRole('admin')")
@GetMapping("/users/{id}")
public String getUserDetail(@PathVariable String id){
    return "用户详情:" + id;
}
```



## 单体方案2



## 微服务方案 Oauth2.0+GateWay



### oauth2.0 对外暴露的重要端口

#### (1)申请token接口/oauth/token

  /oauth/token接口在org.[springframework](https://so.csdn.net/so/search?q=springframework&spm=1001.2101.3001.7020).security.oauth2.provider.endpoint里的TokenEndpoint类里， 该接口会以post请求方式对外提供以表单形式的认证方式，通过了就会返回带期限的token。

通过表单的形式提交, 通过后会返回一个带期限的access_token， 如下用客户端模式去申请token:









