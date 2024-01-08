# Feign解析

#### pom信息

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-openfeign</artifactId>
    <version>2.1.5.RELEASE</version>
</dependency>
```

#### 容器加载初始化解析

*/org/springframework/cloud/spring-cloud-openfeign-core/2.1.5.RELEASE/spring-cloud-openfeign-core-2.1.5.RELEASE.jar!/META-INF/spring.factories*

```properties
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
org.springframework.cloud.openfeign.ribbon.FeignRibbonClientAutoConfiguration,\
org.springframework.cloud.openfeign.FeignAutoConfiguration,\
org.springframework.cloud.openfeign.encoding.FeignAcceptGzipEncodingAutoConfiguration,\
org.springframework.cloud.openfeign.encoding.FeignContentGzipEncodingAutoConfiguration
```

Spring容器启动扫描包内**spring.factories**文件，将**FeignRibbonClientAutoConfiguration**、**FeignAutoConfiguration**、**FeignAcceptGzipEncodingAutoConfiguration**、**FeignContentGzipEncodingAutoConfiguration**接入Spring中管理

*org.springframework.cloud.openfeign.FeignAutoConfiguration*

```java
@Configuration
@ConditionalOnClass(Feign.class)
@EnableConfigurationProperties({ FeignClientProperties.class,
		FeignHttpClientProperties.class })
public class FeignAutoConfiguration {
    // 根据当前类路径中是否存在对应类，从而选择使用ApacheHttpClient还是OkHttpClient
    // 如果都没有，feign默认使用基础的网络流(openConnection)处理请求
}
```

可以看出读取了**FeignClientProperties**(*可通过自定义同名Bean覆盖*)和**FeignHttpClientProperties**(*feign.httpclient配置前缀*)信息，读取application.yml\application.properties配置文件中配置信息来加载。

*org.springframework.cloud.openfeign.ribbon.FeignRibbonClientAutoConfiguration*

```java
@ConditionalOnClass({ ILoadBalancer.class, Feign.class })
@Configuration
@AutoConfigureBefore(FeignAutoConfiguration.class)
@EnableConfigurationProperties({ FeignHttpClientProperties.class })
// Order is important here, last should be the default, first should be optional
// see
// https://github.com/spring-cloud/spring-cloud-netflix/issues/2086#issuecomment-316281653
@Import({ HttpClientFeignLoadBalancedConfiguration.class,
		OkHttpFeignLoadBalancedConfiguration.class,
		DefaultFeignLoadBalancedConfiguration.class })
public class FeignRibbonClientAutoConfiguration {
}
```

可以看出**FeignRibbonClientAutoConfiguration**主要通过**ILoadBalancer**实现请求负载均衡。

**FeignAcceptGzipEncodingAutoConfiguration**和**FeignContentGzipEncodingAutoConfiguration**主要和压缩传输相关，通过配置开启，不做详细阐述。

|Name|Default|Description|
| :---------------------------------------------| :--------| :-----------------------------------------------------------------------------------------------------------------|
|feign.autoconfiguration.jackson.enabled|​`false`​|If true, PageJacksonModule and SortJacksonModule bean will be provided for Jackson page decoding.|
|feign.circuitbreaker.enabled|​`false`​|If true, an OpenFeign client will be wrapped with a Spring Cloud CircuitBreaker circuit breaker.|
|feign.circuitbreaker.group.enabled|​`false`​|If true, an OpenFeign client will be wrapped with a Spring Cloud CircuitBreaker circuit breaker with with group.|
|feign.client.config|||
|feign.client.decode-slash|​`true`​|Feign clients do not encode slash `/`​ characters by default. To change this behavior, set the `decodeSlash`​ to `false`​.|
|feign.client.default-config|​`default`​||
|feign.client.default-to-properties|​`true`​||
|feign.client.refresh-enabled|​`false`​|Enables options value refresh capability for Feign.|
|feign.compression.request.enabled|​`false`​|Enables the request sent by Feign to be compressed.|
|feign.compression.request.mime-types|​`[text/xml, application/xml, application/json]`​|The list of supported mime types.|
|feign.compression.request.min-request-size|​`2048`​|The minimum threshold content size.|
|feign.compression.response.enabled|​`false`​|Enables the response from Feign to be compressed.|
|feign.compression.response.useGzipDecoder|​`false`​|Enables the default gzip decoder to be used.|
|feign.encoder.charset-from-content-type|​`false`​|Indicates whether the charset should be derived from the {@code Content-Type} header.|
|feign.httpclient.connection-timeout|​`2000`​||
|feign.httpclient.connection-timer-repeat|​`3000`​||
|feign.httpclient.disable-ssl-validation|​`false`​||
|feign.httpclient.enabled|​`true`​|Enables the use of the Apache HTTP Client by Feign.|
|feign.httpclient.follow-redirects|​`true`​||
|feign.httpclient.hc5.enabled|​`false`​|Enables the use of the Apache HTTP Client 5 by Feign.|
|feign.httpclient.hc5.pool-concurrency-policy||Pool concurrency policies.|
|feign.httpclient.hc5.pool-reuse-policy||Pool connection re-use policies.|
|feign.httpclient.hc5.socket-timeout|​`5`​|Default value for socket timeout.|
|feign.httpclient.hc5.socket-timeout-unit||Default value for socket timeout unit.|
|feign.httpclient.max-connections|​`200`​||
|feign.httpclient.max-connections-per-route|​`50`​||
|feign.httpclient.time-to-live|​`900`​||
|feign.httpclient.time-to-live-unit|||
|feign.metrics.enabled|​`true`​|Enables metrics capability for Feign.|
|feign.okhttp.enabled|​`false`​|Enables the use of the OK HTTP Client by Feign.|

#### 核心注解解析

**@EnableFeignClients**和**@FeignClient**

@EnableFeignClients注解时SpringBoot应用启动类上注解，标志开启Feign相关定义接口扫面，启用Feign组件。

```java
/**
 * Scans for interfaces that declare they are feign clients (via
 * {@link org.springframework.cloud.openfeign.FeignClient} <code>@FeignClient</code>).
 * Configures component scanning directives for use with
 * {@link org.springframework.context.annotation.Configuration}
 * <code>@Configuration</code> classes.
 * 扫描接口(@FeignClient注解修饰的接口或者通过@Configuration配置的组件扫描指令)
 */
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
@Documented
@Import(FeignClientsRegistrar.class)
public @interface EnableFeignClients {
	// 其他字段
}
```

可以看出通过Import方式导入了*org.springframework.cloud.openfeign.FeignClientsRegistrar*，我们主要看**registerBeanDefinitions**这个核心方法

```java
public void registerBeanDefinitions(AnnotationMetadata metadata,
			BeanDefinitionRegistry registry) {
    registerDefaultConfiguration(metadata, registry);
    registerFeignClients(metadata, registry);
}
```

*org.springframework.cloud.openfeign.FeignClientsRegistrar#registerFeignClient*

```java
private void registerFeignClient(BeanDefinitionRegistry registry,AnnotationMetadata annotationMetadata, Map<String, Object> attributes) {
    // 解析被FeignClient注解修饰的类
    // 看FeignClientFactoryBean的创建及实例化
    String className = annotationMetadata.getClassName();
    // 这一步
    BeanDefinitionBuilder definition = BeanDefinitionBuilder
        .genericBeanDefinition(FeignClientFactoryBean.class);
    validate(attributes);
    definition.addPropertyValue("url", getUrl(attributes));
    definition.addPropertyValue("path", getPath(attributes));
    String name = getName(attributes);
    definition.addPropertyValue("name", name);
    String contextId = getContextId(attributes);
    definition.addPropertyValue("contextId", contextId);
    definition.addPropertyValue("type", className);
    definition.addPropertyValue("decode404", attributes.get("decode404"));
    definition.addPropertyValue("fallback", attributes.get("fallback"));
    definition.addPropertyValue("fallbackFactory", attributes.get("fallbackFactory"));
    definition.setAutowireMode(AbstractBeanDefinition.AUTOWIRE_BY_TYPE);

    String alias = contextId + "FeignClient";
    AbstractBeanDefinition beanDefinition = definition.getBeanDefinition();

    boolean primary = (Boolean) attributes.get("primary"); // has a default, won't be
    // null

    beanDefinition.setPrimary(primary);

    String qualifier = getQualifier(attributes);
    if (StringUtils.hasText(qualifier)) {
        alias = qualifier;
    }

    BeanDefinitionHolder holder = new BeanDefinitionHolder(beanDefinition, className,
                                                           new String[] { alias });
    BeanDefinitionReaderUtils.registerBeanDefinition(holder, registry);
}
```

@FeignClient主要是接口定义注解

```java
@FeignClient(name = "serverName", url = "serverUrl", fallback = XXXServiceHystrix.class)
public interface XXXServiceClient {
    // 定义接口,支持SpringMVC接口方式定义
}
```

主要注解信息，可以参考上述**registerFeignClient**解析查看对应注解元信息

```java
/**
 * Annotation for interfaces declaring that a REST client with that interface should be
 * created (e.g. for autowiring into another component). If ribbon is available it will be
 * used to load balance the backend requests, and the load balancer can be configured
 * using a <code>@RibbonClient</code> with the same name (i.e. value) as the feign client.
 *
 * @author Spencer Gibb
 * @author Venil Noronha
 */
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface FeignClient {

	/**
	 * The name of the service with optional protocol prefix. Synonym for {@link #name()
	 * name}. A name must be specified for all clients, whether or not a url is provided.
	 * Can be specified as property key, eg: ${propertyKey}.
	 * @return the name of the service with optional protocol prefix
	 */
	@AliasFor("name")
	String value() default "";

	/**
	 * The service id with optional protocol prefix. Synonym for {@link #value() value}.
	 * @deprecated use {@link #name() name} instead
	 * @return the service id with optional protocol prefix
	 */
	@Deprecated
	String serviceId() default "";

	/**
	 * This will be used as the bean name instead of name if present, but will not be used
	 * as a service id.
	 * @return bean name instead of name if present
	 */
	String contextId() default "";

	/**
	 * @return The service id with optional protocol prefix. Synonym for {@link #value()
	 * value}.
	 */
	@AliasFor("value")
	String name() default "";

	/**
	 * @return the <code>@Qualifier</code> value for the feign client.
	 */
	String qualifier() default "";

	/**
	 * @return an absolute URL or resolvable hostname (the protocol is optional).
	 */
	String url() default "";

	/**
	 * @return whether 404s should be decoded instead of throwing FeignExceptions
	 */
	boolean decode404() default false;

	/**
	 * A custom <code>@Configuration</code> for the feign client. Can contain override
	 * <code>@Bean</code> definition for the pieces that make up the client, for instance
	 * {@link feign.codec.Decoder}, {@link feign.codec.Encoder}, {@link feign.Contract}.
	 *
	 * @see FeignClientsConfiguration for the defaults
	 * @return list of configurations for feign client
	 */
	Class<?>[] configuration() default {};

	/**
	 * Fallback class for the specified Feign client interface. The fallback class must
	 * implement the interface annotated by this annotation and be a valid spring bean.
	 * @return fallback class for the specified Feign client interface
	 */
	Class<?> fallback() default void.class;

	/**
	 * Define a fallback factory for the specified Feign client interface. The fallback
	 * factory must produce instances of fallback classes that implement the interface
	 * annotated by {@link FeignClient}. The fallback factory must be a valid spring bean.
	 *
	 * @see feign.hystrix.FallbackFactory for details.
	 * @return fallback factory for the specified Feign client interface
	 */
	Class<?> fallbackFactory() default void.class;

	/**
	 * @return path prefix to be used by all method-level mappings. Can be used with or
	 * without <code>@RibbonClient</code>.
	 */
	String path() default "";

	/**
	 * @return whether to mark the feign proxy as a primary bean. Defaults to true.
	 */
	boolean primary() default true;
}
```
