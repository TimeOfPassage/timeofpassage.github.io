# Hystrix知识综述

com.netflix.hystrix.contrib.javanica.aop.aspectj.HystrixCommandAspect
	com.netflix.hystrix.contrib.javanica.aop.aspectj.HystrixCommandAspect#methodsAnnotatedWithHystrixCommand



1. HystrixCommand和HystrixCollapser两个注解命令才进行解析执行

```java
if (method.isAnnotationPresent(HystrixCommand.class) && method.isAnnotationPresent(HystrixCollapser.class)) {
    throw new IllegalStateException("method cannot be annotated with HystrixCommand and HystrixCollapser " +
                                    "annotations at the same time");
}
```

2. 根据不同注解获取不同类型工厂实现

```java
private static final Map<HystrixPointcutType, MetaHolderFactory> META_HOLDER_FACTORY_MAP;

static {
    META_HOLDER_FACTORY_MAP = ImmutableMap.<HystrixPointcutType, MetaHolderFactory>builder()
        .put(HystrixPointcutType.COMMAND, new CommandMetaHolderFactory())
        .put(HystrixPointcutType.COLLAPSER, new CollapserMetaHolderFactory())
        .build();
}
/***/
MetaHolderFactory metaHolderFactory = META_HOLDER_FACTORY_MAP.get(HystrixPointcutType.of(method));
```

3. 

```java
MetaHolder metaHolder = metaHolderFactory.create(joinPoint);
```
