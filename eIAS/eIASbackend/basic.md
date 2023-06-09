## 初始化拦截器

主要内容为获取请求头中的token并与Redis中的token值比对是否相同

用于拦截未登录用户访问除登录以外其他功能的操作

```java
    public class LoginInterceptor implements HandlerInterceptor {
    
        @Override
        public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
            if(UserHolder.getUser()==null){
                //没有,需要拦截
                response.setStatus(401);
                return false;
            }
            return true;
        }
    
        @Override
        public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {
            HandlerInterceptor.super.postHandle(request, response, handler, modelAndView);
        }
    
        @Override
        public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
            HandlerInterceptor.super.afterCompletion(request, response, handler, ex);
        }
    }
public class RefreshTokenInterceptor implements HandlerInterceptor {
    private StringRedisTemplate stringRedisTemplate;

    public RefreshTokenInterceptor(StringRedisTemplate stringRedisTemplate) {
        this.stringRedisTemplate=stringRedisTemplate;
    }

    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        //获取token及redis中的用户
        String token = request.getHeader("Authorization");
        if(StringUtils.isEmpty(token)){
            return true;
        }
        Map<Object, Object> userMap = stringRedisTemplate.opsForHash().entries(LOGIN_USER_KEY+token);
        //判断用户是否存在
        if(userMap.isEmpty()){
            return true;
        }
        //存在就将用户信息保存到ThreadLocal中
        UserDto userDto = BeanUtil.fillBeanWithMap(userMap, new UserDto(), false,
                CopyOptions.create().setIgnoreNullValue(true).setFieldValueEditor((fieldName, fieldValue)->fieldValue.toString()));
        UserHolder.saveUser(userDto);
        //刷新token时间
        stringRedisTemplate.expire(LOGIN_USER_KEY+token,LOGIN_USER_TTL, TimeUnit.MINUTES);
        return true;
    }

    @Override
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {
        HandlerInterceptor.super.postHandle(request, response, handler, modelAndView);
    }

    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
        HandlerInterceptor.super.afterCompletion(request, response, handler, ex);
    }
}
```

## 初始化MVC配置文件

主要内容为添加静态资源映射，将上述拦截器加入到springMVC拦截器中

将Redis传递给拦截器，配置MultipartResolver使得文件上传可以接受Multipart类型的文件

```java
@Slf4j
@Configuration
@EnableWebMvc
public class WebMvcConfig implements WebMvcConfigurer {

    @Resource
    private StringRedisTemplate stringRedisTemplate;

    /**
     * 设置静态资源映射
     * @param registry
     */
    @Override
    public void addResourceHandlers(ResourceHandlerRegistry registry) {
        registry.addResourceHandler("/**").addResourceLocations(
                "classpath:/static/");
        registry.addResourceHandler("swagger-ui.html", "doc.html").addResourceLocations(
                "classpath:/META-INF/resources/");
        registry.addResourceHandler("/webjars/**").addResourceLocations(
                "classpath:/META-INF/resources/webjars/");
    }

    /**
     * 扩展MVC框架的消息转换器
     * @param converters MVC原先默认的转换器
     */
    @Override
    public void extendMessageConverters(List<HttpMessageConverter<?>> converters) {
        //创建消息转换器对象
        MappingJackson2HttpMessageConverter converter = new MappingJackson2HttpMessageConverter();
        //设置对象转换器，底层使用Jackson将java对象转为json
        converter.setObjectMapper(new JacksonObjectMapper());
        ArrayList<MediaType> mediaTypes = new ArrayList<>();
        mediaTypes.add(MediaType.ALL);
        mediaTypes.add(MediaType.APPLICATION_OCTET_STREAM);
        converter.setSupportedMediaTypes(mediaTypes);
        //将这个消息转换器追加到默认的转换器中
        converters.add(0,converter);
    }


    @Bean(name = "multipartResolver")
    public MultipartResolver multipartResolver(){
        CommonsMultipartResolver resolver = new CommonsMultipartResolver();
        resolver.setDefaultEncoding("UTF-8");
        //resolveLazily属性启用是为了推迟文件解析，以在在UploadAction中捕获文件大小异常
        resolver.setResolveLazily(true);
        resolver.setMaxInMemorySize(40960);
        //上传文件大小 50M 50*1024*1024
        resolver.setMaxUploadSize(50*1024*1024);
        return resolver;
    }

    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(new LoginInterceptor()).excludePathPatterns("/user/login","/index.html","/js/**","/css/**","/swagger-resources/**"
                ,"/webjars/**"
                ,"/v2/**"
                ,"/swagger-ui.html/**").order(1);
        registry.addInterceptor(new RefreshTokenInterceptor(stringRedisTemplate)).addPathPatterns("/**").order(0);
    }
}
```

## 初始化web配置文件

主要内容为注册springMVC的配置文件和DispatchServlet

```java
public class WebConfig implements WebApplicationInitializer {
    @Override
    public void onStartup(ServletContext servletContext) throws ServletException {
        AnnotationConfigWebApplicationContext ctx=new AnnotationConfigWebApplicationContext();
        ctx.register(WebMvcConfig.class);//注册SpringMvc的配置类WebMvcConfig
        ctx.setServletContext(servletContext);//和当前ServletContext关联
        /**
         * 注册SpringMvc的DispatcherServlet
         */
        ServletRegistration.Dynamic servlet=servletContext.addServlet("dispatcher",new DispatcherServlet(ctx));
        servlet.addMapping("/");
        servlet.setLoadOnStartup(1);
    }
}
```

## 用户模块

用户类定义

每个用户有自己的id作为唯一标识，有自己的用户名，密码，和状态表示是否被禁用

```java
@Data
@EqualsAndHashCode(callSuper = false)
@Accessors(chain = true)
@TableName("user")
@ApiModel(value="User对象", description="")
public class User implements Serializable {
    private static final long serialVersionUID = 1L;

    @ApiModelProperty(value = "用户id")
    @TableId(value = "id", type = IdType.AUTO)
    private Integer id;

    @ApiModelProperty(value = "用户名")
    private String userName;

    @ApiModelProperty(value = "用户的密码")
    private String password;

    @ApiModelProperty(value = "用户是否禁用 0表示禁用 1表示正常")
    private Integer status;
}
```

### 用户登录模块

接口路径：/user/login

以post方式传入json格式的数据：

```javascript
userName:xxx
password:xxx
```

当数据传入后，首先进入数据库查询是否存在该用户，如果不存在则插入一条用户；如果存在则查看用户是否被禁用，如果未被禁用则登录成功 登录成功后将用户信息和token值存入session中便于后续操作的执行，然后将用户信息和token值都返回给前端

```java
    public User login(@RequestBody Map<String,String> map, HttpSession session){
        String userName = map.get("userName");
        String password = map.get("password");
        //查看该用户是否为新用户
        LambdaQueryWrapper<User> userLambdaQueryWrapper = new LambdaQueryWrapper<>();
        userLambdaQueryWrapper.eq(User::getUserName,userName);
        User user = getOne(userLambdaQueryWrapper);
        if(user==null){
            //是新用户,自动注册
            user = new User();
            user.setUserName(userName);
            user.setPassword(password);
            user.setStatus(1);
            save(user);
            //将用户的信息存到session中，这样可以通过过滤器
            //随机生成token作为登录令牌
            String token = UUID.randomUUID().toString();
            UserDto userDto = new UserDto();
            userDto.setToken(token);
            userDto.setUserName(userName);
            userDto.setPassword(password);
            userDto.setStatus(1);
            session.setAttribute(token,user);
            return userDto;
        }else{
            if(user.getStatus()==0){
                throw new HaveDisabledException("用户已被禁用");
            }else{
                //将用户的信息存到session中，这样可以通过过滤器
                //随机生成token作为登录令牌
                String token = UUID.randomUUID().toString();
                session.setAttribute(token,user);
                UserDto userDto = new UserDto();
                userDto.setToken(token);
                userDto.setUserName(user.getUserName());
                userDto.setPassword(user.getPassword());
                userDto.setStatus(user.getStatus());
                return userDto;
            }
        }
    }
```

## 文件控制模块

### 配置MultipartResolver

#### SpringMvc配置文件

```java
@Slf4j
@Configuration
@EnableWebMvc
public class WebMvcConfig implements WebMvcConfigurer {

    @Resource
    private StringRedisTemplate stringRedisTemplate;

    /**
     * 设置静态资源映射
     * @param registry
     */
    @Override
    public void addResourceHandlers(ResourceHandlerRegistry registry) {
        registry.addResourceHandler("/**").addResourceLocations(
                "classpath:/static/");
        registry.addResourceHandler("swagger-ui.html", "doc.html").addResourceLocations(
                "classpath:/META-INF/resources/");
        registry.addResourceHandler("/webjars/**").addResourceLocations(
                "classpath:/META-INF/resources/webjars/");
    }

    /**
     * 扩展MVC框架的消息转换器
     * @param converters MVC原先默认的转换器
     */
    @Override
    public void extendMessageConverters(List<HttpMessageConverter<?>> converters) {
        //创建消息转换器对象
        MappingJackson2HttpMessageConverter converter = new MappingJackson2HttpMessageConverter();
        //设置对象转换器，底层使用Jackson将java对象转为json
        converter.setObjectMapper(new JacksonObjectMapper());
        ArrayList<MediaType> mediaTypes = new ArrayList<>();
        mediaTypes.add(MediaType.ALL);
        mediaTypes.add(MediaType.APPLICATION_OCTET_STREAM);
        converter.setSupportedMediaTypes(mediaTypes);
        //将这个消息转换器追加到默认的转换器中
        converters.add(0,converter);
    }


    @Bean(name = "multipartResolver")
    public MultipartResolver multipartResolver(){
        CommonsMultipartResolver resolver = new CommonsMultipartResolver();
        resolver.setDefaultEncoding("UTF-8");
        //resolveLazily属性启用是为了推迟文件解析，以在在UploadAction中捕获文件大小异常
        resolver.setResolveLazily(true);
        resolver.setMaxInMemorySize(40960);
        //上传文件大小 50M 50*1024*1024
        resolver.setMaxUploadSize(50*1024*1024);
        return resolver;
    }

    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(new LoginInterceptor()).excludePathPatterns("/user/login","/index.html","/js/**","/css/**","/swagger-resources/**"
                ,"/webjars/**"
                ,"/v2/**"
                ,"/swagger-ui.html/**").order(1);
        registry.addInterceptor(new RefreshTokenInterceptor(stringRedisTemplate)).addPathPatterns("/**").order(0);
    }
}
```

#### Web配置文件

```java
public class WebConfig implements WebApplicationInitializer {
    @Override
    public void onStartup(ServletContext servletContext) throws ServletException {
        AnnotationConfigWebApplicationContext ctx=new AnnotationConfigWebApplicationContext();
        ctx.register(WebMvcConfig.class);//注册SpringMvc的配置类WebMvcConfig
        ctx.setServletContext(servletContext);//和当前ServletContext关联
        /**
         * 注册SpringMvc的DispatcherServlet
         */
        ServletRegistration.Dynamic servlet=servletContext.addServlet("dispatcher",new DispatcherServlet(ctx));
        servlet.addMapping("/");
        servlet.setLoadOnStartup(1);
    }
}
```

### 文件上传

接口路径：/file/upload

前端表单

```html
<form method="post" action="/file/upload" enctype="multipart/form-data">
    <input name="file" type="file"  />
    <input type="submit" value="提交" /> 
</form>
    @Override
    public String upload(MultipartFile file,String basePath) {
        // 1.获取当前上传的文件名
        String originalFilename = file.getOriginalFilename();
        // 2.截取当前文件名的格式后缀
        String suffix = originalFilename.substring(originalFilename.lastIndexOf("."));
        // 3.判断要存储文件的路径是否存在，不存在则创建
        File dir = new File(basePath);
        if (!dir.exists()){
            dir.mkdirs();
        }
        // 4.将上传的文件保存到指定的路径
        try {
            file.transferTo(new File(basePath + originalFilename));
        } catch (IOException e) {
            e.printStackTrace();
        }
        // 5.返回数据给前端
        return originalFilename;
    }
```

### 文件下载

接口路径：/file/download

以get方式传入fileName

```javascript
filename:xxx
    @Override
    public ResponseEntity<byte[]> download(HttpSession session, String basePath, String fileName) throws IOException {
        //获取服务器中文件的真实路径
        String realPath = basePath+fileName;
        //创建输入流
        InputStream is = Files.newInputStream(Paths.get(realPath));
        //创建字节数组
        byte[] bytes = new byte[is.available()];
        //将流读到字节数组中
        is.read(bytes);
        //创建HttpHeaders对象设置响应头信息
        MultiValueMap<String, String> headers = new HttpHeaders();
        //设置要下载方式以及下载文件的名字
        headers.add("Content-Disposition", "attachment;filename="+ URLEncoder.encode(fileName,"UTF-8"));
        //设置响应状态码
        HttpStatus statusCode = HttpStatus.OK;
        //创建ResponseEntity对象
        ResponseEntity<byte[]> responseEntity = new ResponseEntity<>(bytes, headers,
                statusCode);
        //关闭输入流
        is.close();
        return responseEntity;
    }
```