### 主要介绍spring boot web 相关的内容

#### reastful 风格

> @RestController 接口控制器
> @ResponseBody 返回值得内容是json

#### 模板引擎
##### spring boot官方推荐使用Thymeleaf

1. 模板Thymeleaf

    > 引入包
    >
    > [使用方法](#springmv-thymleaf)

    ``` xml

    spring boot 引用的包

    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-thymeleaf</artifactId>
    </dependency>

    单独引包 可集成到spring mvc 中

    <dependency>
        <groupId>org.thymeleaf</groupId>
        <artifactId>thymeleaf</artifactId>
        <version>3.0.7.RELEASE</version>
    </dependency>
    <dependency>
        <groupId>org.thymeleaf</groupId>
        <artifactId>thymeleaf-spring4</artifactId>
        <version>3.0.0.RELEASE</version>
    </dependency>

    ```
    > yml中基本的配置--结合spring-boot 配置文件 使用

    ``` Python
        # Enable template caching.
        spring.thymeleaf.cache=true 
        # Check that the templates location exists.
        spring.thymeleaf.check-template-location=true 
        # Content-Type value.
        spring.thymeleaf.content-type=text/html 
        # Enable MVC Thymeleaf view resolution.
        spring.thymeleaf.enabled=true 
        # Template encoding.
        spring.thymeleaf.encoding=UTF-8 
        # Comma-separated list of view names that should be excluded from resolution.
        spring.thymeleaf.excluded-view-names= 
        # Template mode to be applied to templates. See also StandardTemplateModeHandlers.
        spring.thymeleaf.mode=HTML5 
        # Prefix that gets prepended to view names when building a URL.
        spring.thymeleaf.prefix=classpath:/templates/ 
        # Suffix that gets appended to view names when building a URL.
        spring.thymeleaf.suffix=.html  
        # Order of the template resolver in the chain. 
        spring.thymeleaf.template-resolver-order= 
        #Comma-separated list of view names that can be resolved.
        spring.thymeleaf.view-names= #
    ```

    <span id="springmv-thymleaf">单独使用的配置</span>
    > * 使用的方法

    ``` java
        @Configuration
        @EnableWebMvc
        @ComponentScan("com.thymeleafexamples")
        public class ThymeleafConfig extends WebMvcConfigurerAdapter implements ApplicationContextAware {

        private ApplicationContext applicationContext;

        public void setApplicationContext(ApplicationContext applicationContext) {
            this.applicationContext = applicationContext;
        }

        @Bean
        public ViewResolver viewResolver() {
            ThymeleafViewResolver resolver = new ThymeleafViewResolver();
            resolver.setTemplateEngine(templateEngine());
            resolver.setCharacterEncoding("UTF-8");
            return resolver;
        }

        @Bean
        public TemplateEngine templateEngine() {
            SpringTemplateEngine engine = new SpringTemplateEngine();
            engine.setEnableSpringELCompiler(true);
            engine.setTemplateResolver(templateResolver());
            return engine;
        }

        private ITemplateResolver templateResolver() {
            SpringResourceTemplateResolver resolver = new SpringResourceTemplateResolver();
            resolver.setApplicationContext(applicationContext);
            resolver.setPrefix("/WEB-INF/templates/");
            resolver.setTemplateMode(TemplateMode.HTML);
            return resolver;
        }
        }
    ```




2. 模板FreeMarker
    > 此处不做多余的处理
    > [只引入官方文档](http://freemarker.org/docs/)

    ``` xml

    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-thymeleaf</artifactId>
    </dependency>

    ```

    ##### [相关文档配置可参考](http://blog.didispace.com/springbootweb/) 
    ##### http://blog.didispace.com/springbootweb/
3. 统一异常处理 
    #### 不多说直接上代码

    ``` java
    @ControllerAdvice
    class GlobalExceptionHandler {

        public static final String DEFAULT_ERROR_VIEW = "error";

        // 处理视图的异常
        @ExceptionHandler(value = Exception.class)
        public ModelAndView defaultErrorHandler(HttpServletRequest req, Exception e) throws Exception {
            ModelAndView mav = new ModelAndView();
            mav.addObject("exception", e);
            mav.addObject("url", req.getRequestURL());
            mav.setViewName(DEFAULT_ERROR_VIEW);
            return mav;
        }

        // 处理接口的异常
        @ExceptionHandler(value = MyException.class)
        @ResponseBody
        public ErrorInfo<String> jsonErrorHandler(HttpServletRequest req, MyException e) throws Exception {
            ErrorInfo<String> r = new ErrorInfo<>();
            r.setMessage(e.getMessage());
            r.setCode(ErrorInfo.ERROR);
            r.setData("Some Data");
            r.setUrl(req.getRequestURL().toString());
            return r;
        }

    }
    ```