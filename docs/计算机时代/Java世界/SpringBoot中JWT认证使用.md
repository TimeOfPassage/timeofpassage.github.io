# SpringBoot-JWT认证使用

1. 导入pom

   ```xml
   <dependency>
       <groupId>com.auth0</groupId>
       <artifactId>java-jwt</artifactId>
       <version>3.8.2</version>
   </dependency>
   ```

2. 编写工具类

   ```java
   public class JWTUtil {
   
       //1h
       private static final long EXPIRE_TIME = 60 * 60 * 1000;
   
       /**
        * 校验 token是否正确
        *
        * @param token  密钥
        * @param secret 用户的密码
        * @return 是否正确
        */
       public static boolean verify(String token, String username, String secret) {
           try {
               Algorithm algorithm = Algorithm.HMAC256(secret);
               JWTVerifier verifier = JWT.require(algorithm)
                       .withClaim("username", username)
                       .build();
               verifier.verify(token);
               return true;
           } catch (JWTVerificationException e) {
               throw e;
           }
       }
   
       /**
        * 从 token中获取用户名
        *
        * @return token中包含的用户名
        */
       public static String getUsername(String token) {
           try {
               DecodedJWT jwt = JWT.decode(token);
               return jwt.getClaim("username").asString();
           } catch (JWTDecodeException e) {
               e.printStackTrace();
               return null;
           }
       }
   
       /**
        * 生成 token
        *
        * @param username 用户名
        * @param secret   用户的密码
        * @return token
        */
       public static String sign(String username, String secret) {
           try {
               username = username.toLowerCase();
               Date date = new Date(System.currentTimeMillis() + EXPIRE_TIME);
               Algorithm algorithm = Algorithm.HMAC256(secret);
               return JWT.create()
                       .withClaim("username", username)
                       .withExpiresAt(date)
                       .sign(algorithm);
           } catch (Exception e) {
               e.printStackTrace();
               return null;
           }
       }
   }
   ```

3. 编写登录请求

   ```java
   @RestController
   @RequestMapping("/api")
   public class UserController {
   
       @PostMapping("/login.do")
       public MResponse login(@RequestBody User user) {
           Map<String, Object> retMap = Maps.newHashMap();
           if ("root".equals(user.getUsername()) && "root".equals(user.getPassword())) {
               retMap.put("token", JWTUtil.sign(user.getUsername(), user.getPassword()));
               return MRespHandler.success(retMap);
           }
           return MRespHandler.failed();
       }
   
       @GetMapping("/logout.do")
       public MResponse logout(String username) {
           return MRespHandler.success(username);
       }
   
       @RequireToken
       @GetMapping("/userinfo.do")
       public MResponse userinfo(String username) {
           return MRespHandler.success(new User(1l, "root", "root"));
       }
   }
   ```

4. 编写拦截请求

   ```java
   @Component
   public class TokenInterceptor implements HandlerInterceptor {
   
       @Override
       public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object object) throws Exception {
           //从header中获取前端传入的token,可以添加token过期失效设置
           String token = request.getHeader("token");
           if (StringUtils.isEmpty(token)) {
               throw new RuntimeException("无token，请重新登录");
           }
           // 如果不是映射到方法直接通过
           if (!(object instanceof HandlerMethod)) {
               return true;
           }
           HandlerMethod handlerMethod = (HandlerMethod) object;
           Method method = handlerMethod.getMethod();
           //检查有没有需要用户权限的注解
           if (method.isAnnotationPresent(RequireToken.class)) {
               RequireToken userLoginToken = method.getAnnotation(RequireToken.class);
               if (!userLoginToken.required()) {
                   return true;
               }
               // 执行认证
               String userName;
               try {
                   userName = JWT.decode(token).getClaim("username").asString();
               } catch (JWTDecodeException j) {
                   throw new RuntimeException("401");
               }
               if (!"root".equals(userName)) {
                   throw new RuntimeException("用户不存在，请重新登录");
               }
               //临时模拟
               User user = new User(1l, "root", "root");
               // 验证 token
               try {
                   JWTUtil.verify(token, userName, user.getPassword());
                   return true;
               } catch (JWTVerificationException e) {
                   throw new RuntimeException("401");
               }
           }
           return false;
       }
   }
   ```

   注解支持：

   ```java
   @Target({ElementType.TYPE, ElementType.METHOD})
   @Retention(RetentionPolicy.RUNTIME)
   public @interface RequireToken {
   
       boolean required() default true;
   }
   ```

   

5. 配置拦截请求生效

   ```java
   @Configuration
   public class InterceptorConfig implements WebMvcConfigurer {
   
       @Autowired
       private TokenInterceptor interceptor;
   
       @Override
       public void addInterceptors(InterceptorRegistry registry) {
           registry.addInterceptor(interceptor).addPathPatterns("/api/**")
                                               .excludePathPatterns("/api/login.do");
       }
   }
   ```

   