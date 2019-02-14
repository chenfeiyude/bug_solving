# Problem

We are deploying two Spring Apps in one same tomcat server, and they are working fine before I integrated the Spring cloud. After we are start
using Spring Feign, and the second App failed on start with the following exception

```
2019-02-14 11:14:53,777 ERROR [localhost-startStop-13] [org.springframework.boot.SpringApplication] Application startup failed
org.springframework.jmx.export.UnableToRegisterMBeanException: Unable to register MBean [org.springframework.cloud.context.environment.EnvironmentManager@43ef563a] with key 'environmentManager'; nested exception is javax.management.InstanceAlreadyExistsException: org.springframework.cloud.context.environment:name=environmentManager,type=EnvironmentManager
        at org.springframework.jmx.export.MBeanExporter.registerBeanNameOrInstance(MBeanExporter.java:628)
        at org.springframework.jmx.export.MBeanExporter.registerBeans(MBeanExporter.java:550)
        at org.springframework.jmx.export.MBeanExporter.afterSingletonsInstantiated(MBeanExporter.java:432)
        at org.springframework.beans.factory.support.DefaultListableBeanFactory.preInstantiateSingletons(DefaultListableBeanFactory.java:781)
        at org.springframework.context.support.AbstractApplicationContext.finishBeanFactoryInitialization(AbstractApplicationContext.java:867)
        at org.springframework.context.support.AbstractApplicationContext.refresh(AbstractApplicationContext.java:543)
        at org.springframework.boot.context.embedded.EmbeddedWebApplicationContext.refresh(EmbeddedWebApplicationContext.java:122)
        at org.springframework.boot.SpringApplication.refresh(SpringApplication.java:693)
        at org.springframework.boot.SpringApplication.refreshContext(SpringApplication.java:360)
        at org.springframework.boot.SpringApplication.run(SpringApplication.java:303)
        at org.springframework.boot.web.support.SpringBootServletInitializer.run(SpringBootServletInitializer.java:154)
        at org.springframework.boot.web.support.SpringBootServletInitializer.createRootApplicationContext(SpringBootServletInitializer.java:134)
        at org.springframework.boot.web.support.SpringBootServletInitializer.onStartup(SpringBootServletInitializer.java:87)
        at com.cartell.spring.Application.onStartup(Application.java:62)
        at org.springframework.web.SpringServletContainerInitializer.onStartup(SpringServletContainerInitializer.java:169)
        at org.apache.catalina.core.StandardContext.startInternal(StandardContext.java:5423)
        at org.apache.catalina.util.LifecycleBase.start(LifecycleBase.java:150)
        at org.apache.catalina.core.ContainerBase.addChildInternal(ContainerBase.java:901)
        at org.apache.catalina.core.ContainerBase.addChild(ContainerBase.java:877)
        at org.apache.catalina.core.StandardHost.addChild(StandardHost.java:633)
        at org.apache.catalina.startup.HostConfig.deployWAR(HostConfig.java:983)
        at org.apache.catalina.startup.HostConfig$DeployWar.run(HostConfig.java:1660)
        at java.util.concurrent.Executors$RunnableAdapter.call(Executors.java:511)
        at java.util.concurrent.FutureTask.run(FutureTask.java:266)
        at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149)
        at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)
        at java.lang.Thread.run(Thread.java:748)
Caused by: javax.management.InstanceAlreadyExistsException: org.springframework.cloud.context.environment:name=environmentManager,type=EnvironmentManager
        at com.sun.jmx.mbeanserver.Repository.addMBean(Repository.java:437)
        at com.sun.jmx.interceptor.DefaultMBeanServerInterceptor.registerWithRepository(DefaultMBeanServerInterceptor.java:1898)
        at com.sun.jmx.interceptor.DefaultMBeanServerInterceptor.registerDynamicMBean(DefaultMBeanServerInterceptor.java:966)
        at com.sun.jmx.interceptor.DefaultMBeanServerInterceptor.registerObject(DefaultMBeanServerInterceptor.java:900)
        at com.sun.jmx.interceptor.DefaultMBeanServerInterceptor.registerMBean(DefaultMBeanServerInterceptor.java:324)
        at com.sun.jmx.mbeanserver.JmxMBeanServer.registerMBean(JmxMBeanServer.java:522)
        at org.springframework.jmx.support.MBeanRegistrationSupport.doRegister(MBeanRegistrationSupport.java:195)
        at org.springframework.jmx.export.MBeanExporter.registerBeanInstance(MBeanExporter.java:682)
        at org.springframework.jmx.export.MBeanExporter.registerBeanNameOrInstance(MBeanExporter.java:618)
        ... 26 more
```

# Solution

The main cause for this bug is JMX trying to register the Spring cloud beans with the same name in two Apps.

```
org.springframework.jmx.export.UnableToRegisterMBeanException: Unable to register MBean 
Caused by: javax.management.InstanceAlreadyExistsException
```

Check online and found the solution is define diff JMX domain name for each App. I was trying to add the following configures in application.properties
but still failed

```
spring.jmx.unique-names=true
spring.jmx.default-domain=appname
spring.application.name=appname
endpoints.jmx.unique-names=true
endpoints.jmx.domain=appname
```

After I added to the annotation to Application.java instead it works. 
```java
@EnableMBeanExport(defaultDomain="cartell")
public class Application extends SpringBootServletInitializer {
```



