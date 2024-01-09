# #Spring Security Oauth2方案整理

‍

## SpringBoot+Spring Security Oauth2+Redis

### 概述

> 利用SpringBoot+Spring Security Oauth2+Redis方式整合认证服务

### pom信息

#### 父pom

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <packaging>pom</packaging>
    <modules>
        <module>tds-user</module>
        <module>tds-common</module>
        <module>tds-document</module>
    </modules>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.3.8.RELEASE</version>
        <relativePath/>
    </parent>
    <groupId>tds</groupId>
    <artifactId>tds-parent</artifactId>
    <version>1.0-SNAPSHOT</version>

    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
        <java.version>1.8</java.version>
        <spring-cloud.version>Hoxton.SR9</spring-cloud.version>
        <hutool.version>5.4.5</hutool.version>
        <tds-common.version>1.0-SNAPSHOT</tds-common.version>
        <mysql.version>8.0.15</mysql.version>
        <guava.version>30.0-jre</guava.version>
        <log4j.version>1.2.17</log4j.version>
        <springfox.version>3.0.0</springfox.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-validation</artifactId>
        </dependency>
        <!--Lombok -->
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
        </dependency>
        <!--hutool-->
        <dependency>
            <groupId>cn.hutool</groupId>
            <artifactId>hutool-all</artifactId>
            <version>${hutool.version}</version>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-configuration-processor</artifactId>
        </dependency>
    </dependencies>

    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>${spring-cloud.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
            <dependency>
                <groupId>tds</groupId>
                <artifactId>tds-common</artifactId>
                <version>${tds-common.version}</version>
            </dependency>
            <!--MySQL-->
            <dependency>
                <groupId>mysql</groupId>
                <artifactId>mysql-connector-java</artifactId>
                <version>${mysql.version}</version>
            </dependency>
            <!--guava-->
            <dependency>
                <groupId>com.google.guava</groupId>
                <artifactId>guava</artifactId>
                <version>${guava.version}</version>
            </dependency>
            <!--log-->
            <dependency>
                <groupId>log4j</groupId>
                <artifactId>log4j</artifactId>
                <version>${log4j.version}</version>
            </dependency>
            <!--swagger-->
            <dependency>
                <groupId>io.springfox</groupId>
                <artifactId>springfox-boot-starter</artifactId>
                <version>${springfox.version}</version>
            </dependency>
        </dependencies>
    </dependencyManagement>
</project>
```

#### 子模块pom信息

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>tds-parent</artifactId>
        <groupId>tds</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>tds-user</artifactId>


    <dependencies>
        <dependency>
            <groupId>tds</groupId>
            <artifactId>tds-common</artifactId>
        </dependency>
        <!--orm-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-jpa</artifactId>
        </dependency>
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
        </dependency>
        <!--security-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-security</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-oauth2</artifactId>
        </dependency>
        <!--redis-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-redis</artifactId>
        </dependency>
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
        </dependency>
    </dependencies>

    <build>
        <finalName>tds-user</finalName>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
</project>
```

```
# base config
server.port=8000
spring.application.name=tds-user
# switch control
spring.profiles.active=dev
```

### 服务配置

#### 配置WebSecurityConfig

```java
@EnableWebSecurity
public class WebSecurityConfig extends WebSecurityConfigurerAdapter {

    @Bean
    @Override
    public AuthenticationManager authenticationManagerBean() throws Exception {
        return super.authenticationManagerBean();
    }

    // 密码加密
    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.authorizeRequests()
                .antMatchers("/**").permitAll();
    }
}
```

#### 配置Token存储 - TokenStore

```java
@Configuration
public class RedisTokenStoreConfig {
    @Autowired
    private RedisConnectionFactory redisConnectionFactory;

    @Bean
    public TokenStore redisTokenStore() {
        return new RedisTokenStore(redisConnectionFactory);
    }
}
```

#### 实现用户信息获取服务

实现**UserDetailsService**接口

```java
@Slf4j
@Service(value = "tdsUserDetailsService")
public class UserDetailsServiceImpl implements UserDetailsService {
    @Autowired
    private PasswordEncoder passwordEncoder;

    /**
     * 1. 根据账号获取用户信息， 如果未获取到必须抛出指定异常, 例如：UsernameNotFoundException
     * 2. 根据账号获取所有角色及权限信息
     */
    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        log.info("username: {}", username);
        if (!"admin".equals(username)) {
            throw new UsernameNotFoundException("user is not found");
        }
        List<SimpleGrantedAuthority> authorities = new ArrayList<>();
        authorities.add(new SimpleGrantedAuthority("ROLE_ADMIN"));
        return new User("admin", passwordEncoder.encode("123456"), authorities);
    }
}
```

#### 配置认证服务端 - AuthorizationServerConfig

```java
@Configuration
@EnableAuthorizationServer
public class AuthorizationServerConfig extends AuthorizationServerConfigurerAdapter {

    @Autowired
    private PasswordEncoder         passwordEncoder;
    @Qualifier("tdsUserDetailsService")
    @Autowired
    private UserDetailsService      tdsUserDetailsService;
    @Autowired
    private AuthenticationManager   authenticationManager;
    @Autowired
    private TokenStore              redisTokenStore;
    @Autowired
    private DataSource              dataSource;

    @Override
    public void configure(AuthorizationServerEndpointsConfigurer endpoints) {
        endpoints.authenticationManager(authenticationManager)  // 调用此方法支持password模式
                .userDetailsService(tdsUserDetailsService) // 设置用户验证服务
                .tokenStore(redisTokenStore); // 设置token存储方式
    }

    /**
     * authorizedGrantTypes:
     * authorization_code: 授权码类型
     * implicit: 隐式授权类型
     * password: 资源所有者（即用户）密码类型
     * client_credentials: 客户端凭据（客户端id及key）类型
     * refresh_token: 通过以上授权获得的刷新令牌来获取新的令牌
     */
    @Override
    public void configure(ClientDetailsServiceConfigurer clients) throws Exception {
//        clients.inMemory() // 配置保存到内存中，相当于硬编码了。正常做法是保存到数据库中
//                .withClient("order-client") // 客户端clientId
//                .secret(passwordEncoder.encode("order-secret-8888")) // 客户端clientSecret
//                .authorizedGrantTypes("refresh_token", "authorization_code", "password") // 允许的授权类型
//                .accessTokenValiditySeconds(3600) // token有效期
//                .scopes("all") // 限制客户端访问的权限，换取token的时候带上scope参数，只有在scopes定义的才可以正常换取token
//                .and()
//                .withClient("user-client")
//                .secret(passwordEncoder.encode("user-secret-8888"))
//                .authorizedGrantTypes("refresh_token", "authorization_code", "password")
//                .accessTokenValiditySeconds(3600)
//                .scopes("all");
        clients.jdbc(dataSource)
                .passwordEncoder(passwordEncoder);
    }

    @Override
    public void configure(AuthorizationServerSecurityConfigurer security) {
        security.checkTokenAccess("isAuthenticated()") // 允许已授权用户访问 checkToken 接口
                .tokenKeyAccess("isAuthenticated()") // 允许已授权用户获取 token 接口
                .allowFormAuthenticationForClients(); // 允许客户端访问 OAuth2 授权接口，否则请求 token 会返回 401 (url中存在client-id和client-secret的情况)
    }
}
```

因为试用数据存储配置，故需要建`tds`​库，然后在库中建表。

#### 建表SQL参考

```sql
SET NAMES utf8mb4;
SET FOREIGN_KEY_CHECKS = 0;

-- ----------------------------
-- Table structure for oauth_client_details
-- ----------------------------
DROP TABLE IF EXISTS `oauth_client_details`;
CREATE TABLE `oauth_client_details`  (
  `client_id` varchar(256) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NOT NULL,
  `resource_ids` varchar(256) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NULL DEFAULT NULL,
  `client_secret` varchar(256) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NULL DEFAULT NULL,
  `scope` varchar(256) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NULL DEFAULT NULL,
  `authorized_grant_types` varchar(256) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NULL DEFAULT NULL,
  `web_server_redirect_uri` varchar(256) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NULL DEFAULT NULL,
  `authorities` varchar(256) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NULL DEFAULT NULL,
  `access_token_validity` int(11) NULL DEFAULT NULL,
  `refresh_token_validity` int(11) NULL DEFAULT NULL,
  `additional_information` varchar(4096) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NULL DEFAULT NULL,
  `autoapprove` varchar(256) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NULL DEFAULT NULL,
  PRIMARY KEY (`client_id`) USING BTREE
) ENGINE = InnoDB CHARACTER SET = utf8mb4 COLLATE = utf8mb4_general_ci ROW_FORMAT = Dynamic;

-- ----------------------------
-- Records of oauth_client_details
-- ----------------------------
INSERT INTO `oauth_client_details` VALUES ('user-client', NULL, '$2a$10$cy.EbgI0gVMlUqMFSTi43eBQfa00yYHDxkpbZF6w5HrJfUhbSkpbm', 'all', 'authorization_code,refresh_token,password', NULL, NULL, 3600, 36000, NULL, '1');

SET FOREIGN_KEY_CHECKS = 1;
```

同时因为token试用redis存储，故需要启动redis服务，同时应用配置添加，

#### 应用配置参考

```properties
# base config
server.port=8000
spring.application.name=tds-user
# database config
spring.datasource.url=jdbc:mysql://localhost:3306/tds?useUnicode=true&characterEncoding=utf8&characterSetResults=utf8&serverTimezone=UTC&useSSL=false
spring.datasource.username=root
spring.datasource.password=root
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
spring.datasource.hikari.connection-timeout=30000
spring.datasource.hikari.idle-timeout=600000
spring.datasource.hikari.max-lifetime=1800000
spring.datasource.hikari.maximum-pool-size=16
# redis config
spring.redis.host=localhost
spring.redis.port=6379
spring.redis.timeout=3000ms
spring.redis.jedis.pool.max-active=8
spring.redis.jedis.pool.max-idle=8
spring.redis.jedis.pool.min-idle=0
```

至此认证服务配置完毕，启动服务。框架自动提供扩展如下

> /oauth/token 获取 token 接口  
> /oauth/check_token 检查 token 合法性接口  
> /oauth/authorize 授权码模式认证授权接口

### 验证Token获取

#### 通过curl工具或postman工具进行如下请求

```shell
curl --location --request POST 'http://localhost:8000/oauth/token?grant_type=password&username=admin&password=123456&scope=all' \
--header 'Authorization: Basic dXNlci1jbGllbnQ6dXNlci1jbGllbnQtODg4OA==' \
--header 'Cookie: JSESSIONID=BB7B92466123C4AEAFDE635F54450B3C'
```

​![image-20221010135029677](../../images/20221010/认证服务密码登录获取token请求.png)​

#### 接口返回结果

```json
{
    "access_token": "d6219914-3941-4f09-a765-d3fe414f7a3f",
    "token_type": "bearer",
    "refresh_token": "5481c0bb-ddba-4df5-b4e6-61e3f2e17973",
    "expires_in": 3599,
    "scope": "all"
}
```

查看redis中信息，可以发现token信息已经基于redis存储

​![image-20221010135403413](../../images/20221010/认证服务Token缓存Redis.png)​

至此基于redis存储token的认证服务基本验证配置完成！

## Spring+Spring Security Oauth2+JWT

### 概述

> 上面描述了将Token存储在Redis中的方案。Spring Security Oauth2同时提供了针对JWT的支持。
>
> 虾苗描述如何利用SpringBoot+Spring Security Oauth2+JWT的方案实现认证服务

### 第一步：子模块pom中移除redis最后如下

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>tds-parent</artifactId>
        <groupId>tds</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>tds-user</artifactId>


    <dependencies>
        <dependency>
            <groupId>tds</groupId>
            <artifactId>tds-common</artifactId>
        </dependency>
        <!--orm-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-jpa</artifactId>
        </dependency>
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
        </dependency>
        <!--security-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-security</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-oauth2</artifactId>
        </dependency>
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
        </dependency>
    </dependencies>

    <build>
        <finalName>tds-user</finalName>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
</project>
```

### 第二步：配置文件中移除redis配置

```properties
# base config
server.port=8000
spring.application.name=tds-user
# database config
spring.datasource.url=jdbc:mysql://localhost:3306/tds?useUnicode=true&characterEncoding=utf8&characterSetResults=utf8&serverTimezone=UTC&useSSL=false
spring.datasource.username=root
spring.datasource.password=root
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
spring.datasource.hikari.connection-timeout=30000
spring.datasource.hikari.idle-timeout=600000
spring.datasource.hikari.max-lifetime=1800000
spring.datasource.hikari.maximum-pool-size=16
```

### 第三步：新建JWT配置 - JWTTokenConfig

```java
@Configuration
public class JWTTokenConfig {

    @Bean
    public TokenStore jwtTokenStore() {
        return new JwtTokenStore(jwtAccessTokenConverter());
    }

    @Bean
    public JwtAccessTokenConverter jwtAccessTokenConverter() {
        JwtAccessTokenConverter jwtAccessTokenConverter = new JwtAccessTokenConverter();
        jwtAccessTokenConverter.setSigningKey("dev");
        jwtAccessTokenConverter.setVerifierKey("dev");
        return jwtAccessTokenConverter;
    }
}
```

### 第四步：删除RedisTokenStoreConfig文件并修改AuthorizationServerConfig文件

```java
@Configuration
@EnableAuthorizationServer
public class AuthorizationServerConfig extends AuthorizationServerConfigurerAdapter {

    @Autowired
    private PasswordEncoder         passwordEncoder;
    @Qualifier("tdsUserDetailsService")
    @Autowired
    private UserDetailsService      tdsUserDetailsService;
    @Autowired
    private AuthenticationManager   authenticationManager;
    @Autowired
    private TokenStore              jwtTokenStore;
    @Autowired
    private JwtAccessTokenConverter jwtAccessTokenConverter;
    @Autowired
    private DataSource              dataSource;

    @Override
    public void configure(AuthorizationServerEndpointsConfigurer endpoints) {
        endpoints.authenticationManager(authenticationManager)  // 调用此方法支持password模式
                .userDetailsService(tdsUserDetailsService) // 设置用户验证服务
                .tokenStore(jwtTokenStore)
                .accessTokenConverter(jwtAccessTokenConverter);
    }

    /**
     * authorizedGrantTypes:
     * authorization_code: 授权码类型
     * implicit: 隐式授权类型
     * password: 资源所有者（即用户）密码类型
     * client_credentials: 客户端凭据（客户端id及key）类型
     * refresh_token: 通过以上授权获得的刷新令牌来获取新的令牌
     */
    @Override
    public void configure(ClientDetailsServiceConfigurer clients) throws Exception {
//        clients.inMemory() // 配置保存到内存中，相当于硬编码了。正常做法是保存到数据库中
//                .withClient("order-client") // 客户端clientId
//                .secret(passwordEncoder.encode("order-secret-8888")) // 客户端clientSecret
//                .authorizedGrantTypes("refresh_token", "authorization_code", "password") // 允许的授权类型
//                .accessTokenValiditySeconds(3600) // token有效期
//                .scopes("all") // 限制客户端访问的权限，换取token的时候带上scope参数，只有在scopes定义的才可以正常换取token
//                .and()
//                .withClient("user-client")
//                .secret(passwordEncoder.encode("user-secret-8888"))
//                .authorizedGrantTypes("refresh_token", "authorization_code", "password")
//                .accessTokenValiditySeconds(3600)
//                .scopes("all");
        clients.jdbc(dataSource)
                .passwordEncoder(passwordEncoder);
    }

    @Override
    public void configure(AuthorizationServerSecurityConfigurer security) {
        security.checkTokenAccess("isAuthenticated()") // 允许已授权用户访问 checkToken 接口
                .tokenKeyAccess("isAuthenticated()") // 允许已授权用户获取 token 接口
                .allowFormAuthenticationForClients(); // 允许客户端访问 OAuth2 授权接口，否则请求 token 会返回 401 (url中存在client-id和client-secret的情况)
    }
}
```

改造完成，启动认证服务。通过上面的验证方法获取token进行验证。

​![image-20221010141545291](../../images/20221010/认证服务获取jwt-token.png)​

```json
{
    "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJleHAiOjE2NjUzODYxMjgsInVzZXJfbmFtZSI6ImFkbWluIiwiYXV0aG9yaXRpZXMiOlsiUk9MRV9BRE1JTiJdLCJqdGkiOiI4ZGUzODVmOS05Zjk2LTQwMmItOGQzYy0zMzU5MGUwNTZmZTAiLCJjbGllbnRfaWQiOiJ1c2VyLWNsaWVudCIsInNjb3BlIjpbImFsbCJdfQ.8UaGCIG3uhvGDGG5eSsaqU47JjU4FRV_GJNe2GnsKBw",
    "token_type": "bearer",
    "refresh_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VyX25hbWUiOiJhZG1pbiIsInNjb3BlIjpbImFsbCJdLCJhdGkiOiI4ZGUzODVmOS05Zjk2LTQwMmItOGQzYy0zMzU5MGUwNTZmZTAiLCJleHAiOjE2NjU0MTg1MjgsImF1dGhvcml0aWVzIjpbIlJPTEVfQURNSU4iXSwianRpIjoiM2M5ZDBjZTYtYTZkZi00OTRmLWI5MjYtYmVjMWNkN2U1MDVjIiwiY2xpZW50X2lkIjoidXNlci1jbGllbnQifQ.CJey2DdrACbVOT0xwmxQTpLCjOUufyXkvANW20TX728",
    "expires_in": 3599,
    "scope": "all",
    "jti": "8de385f9-9f96-402b-8d3c-33590e056fe0"
}
```

从响应结果可以看出，token已经转换未jwt的形式了，说明改造集成Ok。

将access_token复制到jwt.io网站解析可以得到如下信息

```json
{
    "user_name": "admin",
    "scope": [
        "all"
    ],
    "exp": 1665386128,
    "authorities": [
        "ROLE_ADMIN"
    ],
    "jti": "8de385f9-9f96-402b-8d3c-33590e056fe0",
    "client_id": "user-client"
}
```

可以发现token中存储了该token有效期、用户信息及权限等信息，所以切记不能将密码一类信息存储在token里。

### 第五步：JWT增加扩展信息

通过上述观察，发现接口字段固定，下面介绍如何将未jwt添加新字段返回

#### 新建JWTTokenEnhancer增强实现

```java
@Slf4j
public class JWTTokenEnhancer implements TokenEnhancer {
    @Override
    public OAuth2AccessToken enhance(OAuth2AccessToken accessToken, OAuth2Authentication authentication) {
        log.info("com.tds.user.config.jwt.JWTTokenEnhancer.enhance");
        Map<String, Object> info = new HashMap();
        info.put("jwt-ext", "扩展信息");
        // 可以通过查询redis或数据库获取额外信息写入jwt的额外信息中
        ((DefaultOAuth2AccessToken) accessToken).setAdditionalInformation(info);
        return accessToken;
    }
}
```

#### 修改JWTTokenConfig配置

```java
@Configuration
public class JWTTokenConfig {

    @Bean
    public TokenStore jwtTokenStore() {
        return new JwtTokenStore(jwtAccessTokenConverter());
    }

    @Bean
    public JwtAccessTokenConverter jwtAccessTokenConverter() {
        JwtAccessTokenConverter jwtAccessTokenConverter = new JwtAccessTokenConverter();
        jwtAccessTokenConverter.setSigningKey("dev");
        jwtAccessTokenConverter.setVerifierKey("dev");
        return jwtAccessTokenConverter;
    }

    @Bean
    public TokenEnhancer jwtTokenEnhancer() {
        return new JWTTokenEnhancer();
    }
}
```

#### 修改AuthorizationServerConfig配置

```java
@Configuration
@EnableAuthorizationServer
public class AuthorizationServerConfig extends AuthorizationServerConfigurerAdapter {

    @Autowired
    private PasswordEncoder         passwordEncoder;
    @Autowired
    private UserDetailsService      tdsUserDetailsService;
    @Autowired
    private AuthenticationManager   authenticationManager;
    @Autowired
    private DataSource              dataSource;
    @Autowired
    private TokenStore              jwtTokenStore;
    @Autowired
    private JwtAccessTokenConverter jwtAccessTokenConverter;
    @Autowired
    private TokenEnhancer           jwtTokenEnhancer;

    @Override
    public void configure(AuthorizationServerEndpointsConfigurer endpoints) {
        // token信息增强
        TokenEnhancerChain tokenEnhancerChain = new TokenEnhancerChain();
        List<TokenEnhancer> tokenEnhancers = new ArrayList<>();
        tokenEnhancers.add(jwtTokenEnhancer);
        tokenEnhancers.add(jwtAccessTokenConverter);
        tokenEnhancerChain.setTokenEnhancers(tokenEnhancers);

        endpoints.authenticationManager(authenticationManager)  // 调用此方法支持password模式
                .userDetailsService(tdsUserDetailsService) // 设置用户验证服务
                .tokenStore(jwtTokenStore)
                .tokenEnhancer(tokenEnhancerChain)
                .accessTokenConverter(jwtAccessTokenConverter);
    }

    @Override
    public void configure(ClientDetailsServiceConfigurer clients) throws Exception {
        clients.jdbc(dataSource).passwordEncoder(passwordEncoder);
    }

    @Override
    public void configure(AuthorizationServerSecurityConfigurer security) {
        security.checkTokenAccess("isAuthenticated()") // 允许已授权用户访问 checkToken 接口
                .tokenKeyAccess("isAuthenticated()") // 允许已授权用户获取 token 接口
                .allowFormAuthenticationForClients(); // 允许客户端访问 OAuth2 授权接口，否则请求 token 会返回 401 (url中存在client-id和client-secret的情况)
    }
}
```

然后通过上述验证token获取进行查看

​![image-20221010143905776](../../images/20221010/认证服务token获取添加额外信息.png)​

```json
{
    "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VyX25hbWUiOiJhZG1pbiIsImp3dC1leHQiOiLmianlsZXkv6Hmga8iLCJzY29wZSI6WyJhbGwiXSwiZXhwIjoxNjY1Mzg3NTMzLCJhdXRob3JpdGllcyI6WyJST0xFX0FETUlOIl0sImp0aSI6ImMyZDQ5MWI3LTZmM2YtNGU2Yy05ODI5LTVmYzUyNjYxYTMwNSIsImNsaWVudF9pZCI6InVzZXItY2xpZW50In0.ihpcxP36sderqVp3V1V54ZoqXHtHLrQ7AodZZ7aGqFY",
    "token_type": "bearer",
    "refresh_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VyX25hbWUiOiJhZG1pbiIsImp3dC1leHQiOiLmianlsZXkv6Hmga8iLCJzY29wZSI6WyJhbGwiXSwiYXRpIjoiYzJkNDkxYjctNmYzZi00ZTZjLTk4MjktNWZjNTI2NjFhMzA1IiwiZXhwIjoxNjY1NDE5OTMzLCJhdXRob3JpdGllcyI6WyJST0xFX0FETUlOIl0sImp0aSI6ImJlZmM0MTJmLWZhNzgtNDdhYi04YWQ2LWZmZTBiNDAxNGI3YyIsImNsaWVudF9pZCI6InVzZXItY2xpZW50In0.n-F81CZ2BPznpCqyJk3s5rO_-c_Ix8T0AA-LUVQUpU8",
    "expires_in": 3599,
    "scope": "all",
    "jwt-ext": "扩展信息",
    "jti": "c2d491b7-6f3f-4e6c-9829-5fc52661a305"
}
```

可以看到，在返回中多了我们添加的额外信息字段`jwt-ext`​及对应的内容

## 资源服务(Client)

#### 概述

> 上两节主要讲了如何实现一个认证服务，本节主要描述如何实现配套的客户端。

资源端的pom信息和子模块pom信息一致，不做特殊添加。

#### 资源端配置文件

```properties
# base config
server.port=8001
spring.application.name=tds-xxx
# 资源服务配置config
security.oauth2.client.client-id=user-client
security.oauth2.client.client-secret=user-client-8888
security.oauth2.client.user-authorization-uri=http://localhost:8000/oauth/authorize
security.oauth2.client.access-token-uri=http://localhost:8000/oauth/token
# 非jwt
#security.oauth2.resource.id=user-client
#security.oauth2.resource.user-info-uri=user-info
#security.oauth2.authorization.check-token-access=http://localhost:8000/oauth/check_token
# jwt
security.oauth2.resource.jwt.key-uri=http://localhost:8000/oauth/token_key
# 此处的dev必须和JWTTokenConfig中配置的key一致
security.oauth2.resource.jwt.key-value=dev
```

上述专门标注了使用jwt方案和非jwt方案的配置区别，具体使用请自行甄别.

#### 资源服务配置 - ResourceServerConfig

##### 非jwt版

```java
@Configuration
@EnableResourceServer
@EnableGlobalMethodSecurity(prePostEnabled = true)
public class ResourceServerConfig extends ResourceServerConfigurerAdapter {

    @Value("${security.oauth2.client.client-id}")
    private String clientId;
    @Value("${security.oauth2.client.client-secret}")
    private String clientSecret;
    @Value("${security.oauth2.authorization.check-token-access}")
    private String checkTokenEndpoint;

    @Bean
    public RemoteTokenServices remoteTokenServices() {
        RemoteTokenServices remoteTokenServices = new RemoteTokenServices();
        remoteTokenServices.setClientId(clientId);
        remoteTokenServices.setClientSecret(clientSecret);
        remoteTokenServices.setCheckTokenEndpointUrl(checkTokenEndpoint);
        return remoteTokenServices;
    }

    @Override
    public void configure(ResourceServerSecurityConfigurer resources) {
        resources.tokenServices(remoteTokenServices());
    }
}
```

##### jwt版

```java
@Configuration
@EnableResourceServer
@EnableGlobalMethodSecurity(prePostEnabled = true)
public class ResourceServerConfig extends ResourceServerConfigurerAdapter {

    @Autowired
    private TokenStore jwtTokenStore;

    @Override
    public void configure(ResourceServerSecurityConfigurer resources) {
        resources.tokenStore(jwtTokenStore);
    }
}
```

#### 新建接口验证

```java
@RestController
public class UserController {

    @GetMapping("get")
    @PreAuthorize("hasAnyRole('ROLE_ADMIN')")
    public Object get(Authentication authentication) {
        OAuth2AuthenticationDetails details = (OAuth2AuthenticationDetails) authentication.getDetails();
        return details.getTokenValue();
    }
}
```

##### 无token请求

```shell
curl --location --request GET 'localhost:8001/get'
```

结果如下

```json
{
    "error": "unauthorized",
    "error_description": "Full authentication is required to access this resource"
}
```

##### 携带token请求

```shell
curl --location --request GET 'localhost:8001/get' \
--header 'Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VyX25hbWUiOiJhZG1pbiIsImp3dC1leHQiOiLmianlsZXkv6Hmga8iLCJzY29wZSI6WyJhbGwiXSwiZXhwIjoxNjY1Mzg4NDcxLCJhdXRob3JpdGllcyI6WyJST0xFX0FETUlOIl0sImp0aSI6ImVkNzRkZGQzLWFiZDQtNDgxYy1hZDJhLWQ1NzhkYzcyNDc1YyIsImNsaWVudF9pZCI6InVzZXItY2xpZW50In0.FDZh49aXvaZxGmIPUJwa3jhLH6j6IxobCr6QX4BP1W8'
```

结果如下, 发现接口正确响应

```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VyX25hbWUiOiJhZG1pbiIsImp3dC1leHQiOiLmianlsZXkv6Hmga8iLCJzY29wZSI6WyJhbGwiXSwiZXhwIjoxNjY1Mzg4NDcxLCJhdXRob3JpdGllcyI6WyJST0xFX0FETUlOIl0sImp0aSI6ImVkNzRkZGQzLWFiZDQtNDgxYy1hZDJhLWQ1NzhkYzcyNDc1YyIsImNsaWVudF9pZCI6InVzZXItY2xpZW50In0.FDZh49aXvaZxGmIPUJwa3jhLH6j6IxobCr6QX4BP1W8
```

至此资源服务集成完毕.

## 非对称加密JWT令牌

### 认证服务修改第一步: 密钥证书生成

```shell
keytool -genkeypair -alias oauth2 -keyalg RSA -keypass oauth2 -keystore oauth2.jks -storepass oauth2
# -alias 设置别名
# -keyalg 设置算法
# -keypass 设置密码
# -keystore 设置存储名称
# -storepass 设置存储密码
```

​![image-20221010150501520](../../images/20221010/公私钥生成.png)​

### 认证服务修改第二步: 修改JWTTokenConfig

将**oauth2.jks**文件复制到**resources**目录下

```java
@Configuration
public class JWTTokenConfig {

    @Bean
    public TokenStore jwtTokenStore() {
        return new JwtTokenStore(jwtAccessTokenConverter());
    }

    @Bean
    public JwtAccessTokenConverter jwtAccessTokenConverter() {
        JwtAccessTokenConverter jwtAccessTokenConverter = new JwtAccessTokenConverter();
        KeyStoreKeyFactory factory = new KeyStoreKeyFactory(new ClassPathResource("oauth2.jks"), "oauth2".toCharArray());
//        jwtAccessTokenConverter.setSigningKey("dev");
//        jwtAccessTokenConverter.setVerifierKey("dev");
        jwtAccessTokenConverter.setKeyPair(factory.getKeyPair("oauth2"));
        return jwtAccessTokenConverter;
    }

    @Bean
    public TokenEnhancer jwtTokenEnhancer() {
        return new JWTTokenEnhancer();
    }
}
```

### 验证 - 获取Token

```shell
curl --location --request POST 'http://localhost:8000/oauth/token?grant_type=password&username=admin&password=123456&scope=all' \
--header 'Authorization: Basic dXNlci1jbGllbnQ6dXNlci1jbGllbnQtODg4OA==' \
--header 'Cookie: JSESSIONID=BB7B92466123C4AEAFDE635F54450B3C'
```

请求token接口

```json
{
    "access_token": "eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VyX25hbWUiOiJhZG1pbiIsImp3dC1leHQiOiLmianlsZXkv6Hmga8iLCJzY29wZSI6WyJhbGwiXSwiZXhwIjoxNjY1MzkyMDg0LCJhdXRob3JpdGllcyI6WyJST0xFX0FETUlOIl0sImp0aSI6IjM1MGZiNWYyLTU3ZWEtNDQxZC04ZjhkLWY0MTUzNTJmYWUxZCIsImNsaWVudF9pZCI6InVzZXItY2xpZW50In0.EtG92TbrxzeGzaiYQA6SdfzztwayZZZDgxAFnGduk47wnIS8UUrDupug4-IuHO9hZxvjHIy20Ak4UgB5SEt4QDV01LTuahDOE7e7cHm99UYFHKR772ET6q_ZMH0mVlPDHUAKbv42lddN1zQut1p66gWylEX0zim_NWJQ-oUshCX8s1GgdUgJqoQOAcV7insdXzNB5n7wn1kZZ9IGHXp6ziqxQJEsgcslKL55ktXqQhuwkMVYnAmaVIZpyqWZUfYl596-T7u4kLH2bwldNLfk_2MbA3ftQvlK3G6oXTWT2_oHANyOdtdxUQGuNAaqkBQvJhXuKxtUt1hfrm3HXELEmw",
    "token_type": "bearer",
    "refresh_token": "eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VyX25hbWUiOiJhZG1pbiIsImp3dC1leHQiOiLmianlsZXkv6Hmga8iLCJzY29wZSI6WyJhbGwiXSwiYXRpIjoiMzUwZmI1ZjItNTdlYS00NDFkLThmOGQtZjQxNTM1MmZhZTFkIiwiZXhwIjoxNjY1NDI0NDg0LCJhdXRob3JpdGllcyI6WyJST0xFX0FETUlOIl0sImp0aSI6ImMzNmQ0YTU2LTRiYjYtNGYxNS04ZjM4LTQwZDk0ODI1ZGExMSIsImNsaWVudF9pZCI6InVzZXItY2xpZW50In0.S4uCbkcVAxWcOVp4NSbaBgpYhpEWNySitfegFa-INw4Gd78czSd_a_fpmHSNUl203KBySXlvtNeJO25FInGUzS0-jFzNB5k2_nSjQdju0fhJ7Ujal_LWlyFyETIy6K0Z2lyNaatZ0oumFwXlYKRlily3Fqcg7pxwvnzuSL_XJgI_CyBLPoVfEkh7YMgOFhpUauIlNae_1dzaJK3a0lYcYPggAwS7NFLg1WNufiky9jNeyOK1OSYkXkan9h4JPznyS2kX03dBAESEjvLyHXZzY2xYFcMFtMsxubD_xJezb2f0wRQRxivl86avhzqZXR8u8UiHKewLnfpwT4p3rAlQGA",
    "expires_in": 3599,
    "scope": "all",
    "jwt-ext": "扩展信息",
    "jti": "350fb5f2-57ea-441d-8f8d-f415352fae1d"
}
```

根据返回接口发现长度边长了证明已经基于rsa加密token

### 资源服务修改第一步: 公钥生成

```shell
# 在oauth2.jks同级目录执行下面命令获取公钥信息
keytool -list -rfc --keystore oauth2.jks | openssl x509 -inform pem -pubkey
```

​![image-20221010151611881](../../images/20221010/资源客户端公钥生成.png)​

将上述**public key**部分信息复制到资源客户端的**resources**目录下,新建**public.txt**文件存储

### 资源服务修改第二步: JWTTokenConfig配置修改

```java
@Configuration
public class JwtTokenConfig {

    @Bean
    public TokenStore jwtTokenStore() {
        return new JwtTokenStore(jwtAccessTokenConverter());
    }

    @Bean
    public JwtAccessTokenConverter jwtAccessTokenConverter() {
        JwtAccessTokenConverter jwtAccessTokenConverter = new JwtAccessTokenConverter();
        ClassPathResource classPathResource = new ClassPathResource("public.txt");
        String publicKey;
        try {
            publicKey = IoUtil.read(classPathResource.getInputStream(), "UTF-8");
        } catch (Exception e) {
            throw new RuntimeException("公钥读取失败，请检查public.txt文件信息");
        }
        jwtAccessTokenConverter.setVerifierKey(publicKey);
        return jwtAccessTokenConverter;
    }
}
```

### 资源服务修改第三步: 移除客户端配置下所有properties属性,利用纯代码处理

```java
@Configuration
@EnableResourceServer
@EnableGlobalMethodSecurity(prePostEnabled = true)
public class ResourceServerConfig extends ResourceServerConfigurerAdapter {

    @Autowired
    private TokenStore jwtTokenStore;

    @Override
    public void configure(ResourceServerSecurityConfigurer resources) {
        // 配置当前资源id，会在认证服务验证(客户端表的resources配置了则可以访问这个服务)
        resources.resourceId("tds-document");
        resources.tokenStore(jwtTokenStore);
        resources.tokenServices(tokenServices());
    }


    /**
     * 如果认证服务和资源服务非同一个，则需要配置RemoteTokenServices，通过远程取校验token有效性
     */
    @Bean
    public ResourceServerTokenServices tokenServices() {
        RemoteTokenServices services = new RemoteTokenServices();
        services.setCheckTokenEndpointUrl("http://localhost:8000/oauth/check_token");
        services.setClientId("user-client");
        services.setClientSecret("user-client-8888");
        return services;
    }
}
```

### 验证 - 使用上述Token访问资源

```shell
curl --location --request GET 'localhost:8001/get' \
--header 'Authorization: Bearer eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VyX25hbWUiOiJhZG1pbiIsImp3dC1leHQiOiLmianlsZXkv6Hmga8iLCJzY29wZSI6WyJhbGwiXSwiZXhwIjoxNjY1MzkxMTcxLCJhdXRob3JpdGllcyI6WyJST0xFX0FETUlOIl0sImp0aSI6IjFlNWU3MGI2LWVjODctNGFjMy04MDU3LTg1ZGZiMWYzOTI2MCIsImNsaWVudF9pZCI6InVzZXItY2xpZW50In0.cJBJLsv1mgar-b42Vh_Hp8dyLZpB4Q5WxfmdGjPStv0bWoZlBxzF5yCA5jqOdPiw4AJDk6Ek9HhsiE3D9fJr12FnEfY784u0vCGlfjTaY_Y_TCS04BBxyKp58JKLWn08wLs9_ao2-uBF4ZXXSEinB-fc2E99J5NrYnQBYr8w62pJSJvC849aNHQ97oWhRc3kRZpFh_ZPXoSboNW5NcfriOQHHu7TrKmoxyAuGEyIabZ6kZaklD11FmiqHIY-Pq6wR7i_CQvrs-gshuVCinUKwYB_Jgncq9ij9urXut1nUj7uGdjPYz2dg4uWEns4wDKX4zHDpCb3MWPqWdwRyqad1g'
```

请求接口返回

```shell
eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VyX25hbWUiOiJhZG1pbiIsImp3dC1leHQiOiLmianlsZXkv6Hmga8iLCJzY29wZSI6WyJhbGwiXSwiZXhwIjoxNjY1MzkxMTcxLCJhdXRob3JpdGllcyI6WyJST0xFX0FETUlOIl0sImp0aSI6IjFlNWU3MGI2LWVjODctNGFjMy04MDU3LTg1ZGZiMWYzOTI2MCIsImNsaWVudF9pZCI6InVzZXItY2xpZW50In0.cJBJLsv1mgar-b42Vh_Hp8dyLZpB4Q5WxfmdGjPStv0bWoZlBxzF5yCA5jqOdPiw4AJDk6Ek9HhsiE3D9fJr12FnEfY784u0vCGlfjTaY_Y_TCS04BBxyKp58JKLWn08wLs9_ao2-uBF4ZXXSEinB-fc2E99J5NrYnQBYr8w62pJSJvC849aNHQ97oWhRc3kRZpFh_ZPXoSboNW5NcfriOQHHu7TrKmoxyAuGEyIabZ6kZaklD11FmiqHIY-Pq6wR7i_CQvrs-gshuVCinUKwYB_Jgncq9ij9urXut1nUj7uGdjPYz2dg4uWEns4wDKX4zHDpCb3MWPqWdwRyqad1g
```

可以发现接口正常返回信息. 资源配置完毕.
