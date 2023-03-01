# Spring-Bean注入使用

## 1. hashmap注入

### a) xml配置

```xml
<!-- 1.定义service impl实现类,这些类都实现都一个接口或者同一个抽象类 -->
<bean id="xxxxAServiceImpl" class="com.xxx.xxx.XXXXAServiceImpl"></bean>
<bean id="xxxxBServiceImpl" class="com.xxx.xxxx.XXXXBServiceImpl"></bean>
<!-- 2.配置构造注入 -->
<bean id="servicesMap" class="java.util.HashMap" scope="singleton">
  <constructor-arg>
    <map>
      <entry>
        <key>
          <value type="java.lang.String">1</value>
        </key>
        <ref bean="xxxxAServiceImpl"/>
      </entry>
      <entry>
        <key>
          <value type="java.lang.String">2</value>
        </key>
        <ref bean="xxxxBServiceImpl"/>
      </entry>
    </map>
  </constructor-arg>
</bean>
<!-- 3. 配置具体类使用,通过set注入 -->
<bean id="definiteService" class="com.xxx.xxx.DefiniteServiceImpl">
  <property name="definiteServicesMap" ref="servicesMap"/>
</bean>
```

### b) 代码使用

```java
// class
public class DefiniteServiceImpl implements DefiniteService {
    private Map<String, XXXXService> definiteServicesMap;
		public void setDefiniteService(Map<String, XXXXService> definiteServicesMap) {
        this.definiteServicesMap = definiteServicesMap;
    }
  
    public void test() {
      definiteServicesMap.get("2").xxxx();
    }
}

```

## 2.线程池托管

### a) xml配置

```xml
<!-- spring thread pool executor -->
<bean id="taskExecutor" class="org.springframework.scheduling.concurrent.ThreadPoolTaskExecutor">
  <!-- 线程池维护线程的最少数量 -->
  <property name="corePoolSize" value="20"/>
  <!-- 允许的空闲时间 -->
  <property name="keepAliveSeconds" value="60"/>
  <!-- 线程池维护线程的最大数量 -->
  <property name="maxPoolSize" value="60"/>
  <!-- 缓存队列 -->
  <property name="queueCapacity" value="60"/>
  <!-- 对拒绝task的处理策略 -->
  <property name="rejectedExecutionHandler">
    <bean class="java.util.concurrent.ThreadPoolExecutor$CallerRunsPolicy"/>
  </property>
</bean>
```

