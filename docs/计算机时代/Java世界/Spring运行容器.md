# #Spring运行容器

今天看了下`Spring`​针对**WebSocket**的支持. 其中有这么一段判断当前运行环境支持WS升级策略的代码.

> spring 5.3.24
>
> springboot 2.7.6

​`org.springframework.web.socket.server.support.AbstractHandshakeHandler#initRequestUpgradeStrategy`​

```java
private static RequestUpgradeStrategy initRequestUpgradeStrategy() {
	String className;
	if (tomcatWsPresent) {
		className = "org.springframework.web.socket.server.standard.TomcatRequestUpgradeStrategy";
	}
	else if (jettyWsPresent) {
		className = "org.springframework.web.socket.server.jetty.JettyRequestUpgradeStrategy";
	}
	else if (jetty10WsPresent) {
		className = "org.springframework.web.socket.server.jetty.Jetty10RequestUpgradeStrategy";
	}
	else if (undertowWsPresent) {
		className = "org.springframework.web.socket.server.standard.UndertowRequestUpgradeStrategy";
	}
	else if (glassfishWsPresent) {
		className = "org.springframework.web.socket.server.standard.GlassFishRequestUpgradeStrategy";
	}
	else if (weblogicWsPresent) {
		className = "org.springframework.web.socket.server.standard.WebLogicRequestUpgradeStrategy";
	}
	else if (websphereWsPresent) {
		className = "org.springframework.web.socket.server.standard.WebSphereRequestUpgradeStrategy";
	}
	else {
		throw new IllegalStateException("No suitable default RequestUpgradeStrategy found");
	}

	try {
		Class<?> clazz = ClassUtils.forName(className, AbstractHandshakeHandler.class.getClassLoader());
		return (RequestUpgradeStrategy) ReflectionUtils.accessibleConstructor(clazz).newInstance();
	}
	catch (Exception ex) {
		throw new IllegalStateException(
				"Failed to instantiate RequestUpgradeStrategy: " + className, ex);
	}
}
```

通过上述代码可以看出WebSocket支持升级的容器集成了

* tomcat (SpringBoot内嵌Tomcat)
* jetty && jetty10 [https://github.com/eclipse/jetty.project](https://github.com/eclipse/jetty.project)

  > Jetty is a lightweight highly scalable java based web server and servlet engine. Our goal is to support web protocols like HTTP, HTTP/2 and WebSocket in a high volume low latency way that provides maximum performance while retaining the ease of use and compatibility with years of servlet development. Jetty is a modern fully async web server that has a long history as a component oriented technology easily embedded into applications while still offering a solid traditional distribution for webapp deployment.
  >
* undertow [https://undertow.io/](https://undertow.io/)

  > Undertow is a flexible performant web server written in java, providing both blocking and non-blocking API’s based on NIO.
  >
  > Undertow has a composition based architecture that allows you to build a web server by combining small single purpose handlers. The gives you the flexibility to choose between a full Java EE servlet 4.0 container, or a low level non-blocking handler, to anything in between.
  >
  > Undertow is designed to be fully embeddable, with easy to use fluent builder APIs. Undertow’s lifecycle is completely controlled by the embedding application.
  >
  > Undertow is sponsored by JBoss and is the default web server in the [Wildfly Application Server](https://github.com/wildfly/wildfly).
  >
* glassfish [https://javaee.github.io/glassfish/](https://javaee.github.io/glassfish/)

  > GlassFish is an **open-source Jakarta EE platform application server project** started by Sun Microsystems, then sponsored by Oracle Corporation, and now living at the Eclipse Foundation and supported by Payara, Oracle and Red Hat.
  >
* weblogic [https://www.oracle.com/java/weblogic/](https://www.oracle.com/java/weblogic/)

  > WebLogic is **an Application Server that runs on a middle tier, between back-end databases and related applications and browser-based thin clients**. WebLogic Server mediates the exchange of requests from the client tier with responses from the back-end tier.
  >
* websphere [https://www.ibm.com/products/websphere-application-server](https://www.ibm.com/products/websphere-application-server)

  > A flexible, security-rich Java server runtime environment for enterprise applications
  >

‍
