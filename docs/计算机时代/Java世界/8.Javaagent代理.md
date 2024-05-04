# Java-Javaagent代理

## 1. 介绍

一种字节码"插桩"技术

市面相关应用：阿里arthas

## 2. 使用

a. 实现如下方法作为入口：

```java
public static void premain(String arg, Instrumentation instrumentation);
```

b. 实现业务逻辑

```java
public static void premain(String arg, Instrumentation instrumentation) {
  System.err.println("agent startup , args is " + arg);
  // 注册我们的文件下载函数
  instrumentation.addTransformer(new DumpClassesService());
}

public class DumpClassesService implements ClassFileTransformer {
    @Override
    public byte[] transform(ClassLoader loader, String className, Class<?> classBeingRedefined, ProtectionDomain protectionDomain, byte[] classfileBuffer) throws IllegalClassFormatException {
        System.out.println("premain load class, className:" + className);
        return classfileBuffer;
    }
}
```

c. 配置pre-main启动（可借助maven插件辅助生成)

```xml
<build>
  <plugins>
    <plugin>
      <artifactId>maven-compiler-plugin</artifactId>
      <version>3.1</version>
      <configuration>
        <source>1.8</source>
        <target>1.8</target>
      </configuration>
    </plugin>
    <plugin>
      <groupId>org.apache.maven.plugins</groupId>
      <artifactId>maven-jar-plugin</artifactId>
      <configuration>
        <archive>
          <manifestEntries>
            <Premain-Class>[包含premain的类]</Premain-Class>
            <Agent-Class>[包含premain的类]</Agent-Class>
            <Can-Redefine-Classes>true</Can-Redefine-Classes>
            <Can-Retransform-Classes>true</Can-Retransform-Classes>
          </manifestEntries>
        </archive>
      </configuration>
    </plugin>
  </plugins>
</build>
```

MAINIFEST.MF

```
Manifest-Version: 1.0
Implementation-Title: micro-agent
Premain-Class: com.vivo.micro.AgentApplication
Implementation-Version: 1.0-SNAPSHOT
Agent-Class: com.vivo.micro.AgentApplication
Can-Redefine-Classes: true
Can-Retransform-Classes: true
Build-Jdk-Spec: 1.8
Created-By: Maven Archiver 3.4.0
```

d: 生成jar包【agent.jar 名字跟着工程走】

```text
通过maven的package生成jar包
mvn clearn package
```

e: 应用agent(启动应用的时候指定javaagent参数)

```bash
java -javaagent:/[dirctory]/agent.jar 
# etc: example
java -javaagent:/[dirctory]/agent.jar -jar app.jar #[应用jar包]
```



## 3.场景

应用全链路调用分析系统

etc: 对应用做拦截，记录请求时间、调用耗时、执行成功数

