



# 参考配置

```java
@EnableGlobalMethodSecurity(prePostEnabled = true, securedEnabled = true)
public class SecurityConfig extends WebSecurityConfigurerAdapter {
		// 自定义用户认证逻辑
    @Autowired
    private UserDetailsService userDetailsService;
		// 认证失败处理类
    @Autowired
    private AuthenticationEntryPointImpl unauthorizedHandler;
		// 退出处理类
    @Autowired
    private LogoutSuccessHandlerImpl logoutSuccessHandler;
		// token认证过滤器
    @Autowired
    private JwtAuthenticationTokenFilter authenticationTokenFilter;
		// 跨域过滤器
    @Autowired
    private CorsFilter corsFilter;
		// 允许匿名访问的地址
    @Autowired
    private PermitAllUrlProperties permitAllUrl;
		// 解决 无法直接注入 AuthenticationManager
    @Bean
    @Override
    public AuthenticationManager authenticationManagerBean() throws Exception {
        return super.authenticationManagerBean();
    }

    /**
     * anyRequest          |   匹配所有请求路径
     * access              |   SpringEl表达式结果为true时可以访问
     * anonymous           |   匿名可以访问
     * denyAll             |   用户不能访问
     * fullyAuthenticated  |   用户完全认证可以访问（非remember-me下自动登录）
     * hasAnyAuthority     |   如果有参数，参数表示权限，则其中任何一个权限可以访问
     * hasAnyRole          |   如果有参数，参数表示角色，则其中任何一个角色可以访问
     * hasAuthority        |   如果有参数，参数表示权限，则其权限可以访问
     * hasIpAddress        |   如果有参数，参数表示IP地址，如果用户IP和参数匹配，则可以访问
     * hasRole             |   如果有参数，参数表示角色，则其角色可以访问
     * permitAll           |   用户可以任意访问
     * rememberMe          |   允许通过remember-me登录的用户访问
     * authenticated       |   用户登录后可访问
     */
    @Override
    protected void configure(HttpSecurity httpSecurity) throws Exception {
        // 注解标记允许匿名访问的url
        ExpressionUrlAuthorizationConfigurer<HttpSecurity>.ExpressionInterceptUrlRegistry registry = httpSecurity.authorizeRequests();
        permitAllUrl.getUrls().forEach(url -> registry.antMatchers(url).permitAll());

        httpSecurity
                // CSRF禁用，因为不使用session
                .csrf().disable()
                // 认证失败处理类
                .exceptionHandling().authenticationEntryPoint(unauthorizedHandler).and()
                // 基于token，所以不需要session
                .sessionManagement().sessionCreationPolicy(SessionCreationPolicy.STATELESS).and()
                // 过滤请求
                .authorizeRequests()
                // 对于登录login 注册register 验证码captchaImage 允许匿名访问
                .antMatchers("/login", "/register", "/captchaImage").anonymous()
                // 静态资源，可匿名访问
                .antMatchers(HttpMethod.GET, "/", "/*.html", "/**/*.html", "/**/*.css", "/**/*.js", "/profile/**").permitAll()
                .antMatchers("/swagger-ui.html", "/swagger-resources/**", "/webjars/**", "/*/api-docs", "/druid/**").permitAll()
                // 除上面外的所有请求全部需要鉴权认证
                .anyRequest().authenticated()
                .and()
                .headers().frameOptions().disable();
        // 添加Logout filter
        httpSecurity.logout().logoutUrl("/logout").logoutSuccessHandler(logoutSuccessHandler);
        // 添加JWT filter
        httpSecurity.addFilterBefore(authenticationTokenFilter, UsernamePasswordAuthenticationFilter.class);
        // 添加CORS filter
        httpSecurity.addFilterBefore(corsFilter, JwtAuthenticationTokenFilter.class);
        httpSecurity.addFilterBefore(corsFilter, LogoutFilter.class);
    }

    /**
     * 强散列哈希加密实现
     */
    @Bean
    public BCryptPasswordEncoder bCryptPasswordEncoder() {
        return new BCryptPasswordEncoder();
    }

    /**
     * 身份认证接口
     */
    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        auth.userDetailsService(userDetailsService).passwordEncoder(bCryptPasswordEncoder());
    }
}
```

## 使用注解权限 @PreAuthorize, 调用容器内bean方法

> 1、`@PreAuthorize` 注解支持丰富的权限表达式，可以根据具体的需求进行配置。例如，可以使用 `hasRole('ROLE_ADMIN')` 来验证用户是否具备指定角色，或者使用 `hasAuthority('PERMISSION_ADD_USER')` 来验证用户是否具备指定权限，继承`UserDetails`，实现接口`GrantedAuthority`权限和角色；
>
> `org.springframework.security.access.expression.SecurityExpressionOperations`
>
> > ```java
> > GlobalMethodSecurityConfiguration#methodSecurityInterceptor
> > 	AuthorizationManagerBeforeMethodInterceptor#attemptAuthorization
> > 	AuthorizationManager#check
> >   	// 以下是接口类实现
> >   	AuthenticatedAuthorizationManager //
> >   	AuthorityAuthorizationManager // 
> >   	Jsr250AuthorizationManager
> >   	PostAuthorizeAuthorizationManager
> >   	PreAuthorizeAuthorizationManager
> >   	SecuredAuthorizationManager
> > ```
>
> 2、spring bean 校验：` ss` 是一个注册在 Spring容器中的BEAN，对应的类是`PermissionService`；`hasPermi `是`PermissionService`类中定义的方法；当Spring EL 表达式返回TRUE，则权限校验通过

```java
// 如果传入hasRole("ADMIN")或hasRole("ROLE_ADMIN")，则当defaultRolePrefix为“ROLE_”（默认）时，将使用角色 ROLE_ADMIN
// hasRole('ROLE_角色名称')、hasAnyRole 
// hasAuthority('PERMISSION_权限编码code')、hasAnyAuthority
// 源码解析：org.springframework.security.authorization.AuthorityAuthorizationManager

// 在controller的方法上增加
// @PreAuthorize("hasRole('ROLE_ADMIN')") 或 @PreAuthorize("hasAuthority('system_role_list')") 
@PreAuthorize("@ss.hasPermi('system:role:list')")
@GetMapping("/list")
public R list(SysRoleEntity role) {
  PageResult<SysRoleEntity> list = roleService.page(role);
  return R.ok().put(list);
}

@Service("ss")
public class PermissionService {
  
  /**
   * 验证用户是否具备某权限
   *
   * @param permission 权限字符串
   * @return 用户是否具备某权限
   */
   public boolean hasPermi(String permission) {
        if (StringUtils.isEmpty(permission)) {
            return false;
        }
        LoginUser loginUser = SecurityUtils.getLoginUser();
        if (StringUtils.isNull(loginUser) || CollectionUtils.isEmpty(loginUser.getPermissions())) {
            return false;
        }
        PermissionContextHolder.setContext(permission);
        return hasPermissions(loginUser.getPermissions(), permission);
    }
}
```

## 自定义认证实现，允许匿名访问

```java
// 注解标记允许匿名访问的url
ExpressionUrlAuthorizationConfigurer<HttpSecurity>.ExpressionInterceptUrlRegistry registry = httpSecurity.authorizeRequests();
// 自定义类注入
PermitAllUrlProperties permitAllUrl = SpringUtils.getBean(PermitAllUrlProperties.class);
permitAllUrl.getUrls().forEach(url -> registry.antMatchers(url).permitAll());

// 设置Anonymous注解允许匿名访问的url
@Data
@Component
public class PermitAllUrlProperties implements InitializingBean {
    private static final Pattern PATTERN = Pattern.compile("\\{(.*?)\\}");

    private List<String> urls = new ArrayList<>();

    public String ASTERISK = "*";

    @Override
    public void afterPropertiesSet() {
        RequestMappingHandlerMapping mapping = SpringUtil.getBean("requestMappingHandlerMapping", RequestMappingHandlerMapping.class);
        Map<RequestMappingInfo, HandlerMethod> map = mapping.getHandlerMethods();

        map.keySet().forEach(info -> {
            HandlerMethod handlerMethod = map.get(info);

            // 获取方法上边的注解 替代path variable 为 *
            Anonymous method = AnnotationUtils.findAnnotation(handlerMethod.getMethod(), Anonymous.class);
            Optional.ofNullable(method).ifPresent(anonymous -> info.getPatternsCondition().getPatterns()
                    .forEach(url -> urls.add(RegExUtils.replaceAll(url, PATTERN, ASTERISK))));

            // 获取类上边的注解, 替代path variable 为 *
            Anonymous controller = AnnotationUtils.findAnnotation(handlerMethod.getBeanType(), Anonymous.class);
            Optional.ofNullable(controller).ifPresent(anonymous -> info.getPatternsCondition().getPatterns()
                    .forEach(url -> urls.add(RegExUtils.replaceAll(url, PATTERN, ASTERISK))));
        });
    }
}
```

