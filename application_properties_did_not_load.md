# Problem

After migrated with gradle and spring boot, application.properties was not loaded and all settings were not working. 
I have to put some configures aboe application.java instead. Like

```java
@SpringBootApplication(scanBasePackages = {"com.xxxx"}) 
@EnableAutoConfiguration(exclude = {DataSourceAutoConfiguration.class, HibernateJpaAutoConfiguration.class})
@EnableScheduling
@EnableDiscoveryClient
@EnableFeignClients(basePackages = "com.xxxx.feignservices.clients")
@EnableAspectJAutoProxy
@EnableMBeanExport(registration=RegistrationPolicy.IGNORE_EXISTING) // avoid duplicate beans between multiple apps deployed on the same server
@EnableMBeanExport(defaultDomain="xxxx")// added configures spring.jmx.default-domain=xxxx in application.properties but not working
public class Application extends SpringBootServletInitializer {
```

# Bug found

Finally, found the issue is that the project was migrating from old ant build non-spring project, and the packages paths are diff with spring project.
So the configure file is not at the correct path. The solution is configure the resource file in build.gradle like

```gradle
sourceSets {
  	main {
   		java { srcDir 'common/src' }
   		resources { srcDirs "common/resources" }
  	}
}
```
