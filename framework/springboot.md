

## 什么是 spring boot？

简化了spring的配置。

## Spring Boot有哪些优点？

Spring Boot的优点有：

- 减少开发，测试时间和努力。
- 使用JavaConfig有助于避免使用XML。
- 避免大量的Maven导入和各种版本冲突。
- 提供意见发展方法。
- 通过提供默认值快速开始开发。
- 没有单独的Web服务器需要。这意味着你不再需要启动Tomcat，Glassfish或其他任何东西。
- 需要更少的配置 因为没有web.xml文件。只需添加用@ Configuration注释的类，然后添加用@Bean注释的方法，Spring将自动加载对象并像以前一样对其进行管理。您甚至可以将@Autowired添加到bean方法中，以使Spring自动装入需要的依赖关系中。
- 基于环境的配置 使用这些属性，您可以将您正在使用的环境传递到应用程序：-Dspring.profiles.active = {enviornment}。在加载主应用程序属性文件后，Spring将在（application{environment} .properties）中加载后续的应用程序属性文件。

## 什么是JavaConfig？

Spring JavaConfig是Spring社区的产品，它提供了配置Spring IoC容器的纯Java方法。因此它有助于避免使用XML配置。使用JavaConfig的优点在于：

- 面向对象的配置。由于配置被定义为JavaConfig中的类，因此用户可以充分利用Java中的面向对象功能。一个配置类可以继承另一个，重写它的@Bean方法等。
- 减少或消除XML配置。基于依赖注入原则的外化配置的好处已被证明。但是，许多开发人员不希望在XML和Java之间来回切换。JavaConfig为开发人员提供了一种纯Java方法来配置与XML配置概念相似的Spring容器。从技术角度来讲，只使用JavaConfig配置类来配置容器是可行的，但实际上很多人认为将JavaConfig与XML混合匹配是理想的。
- 类型安全和重构友好。JavaConfig提供了一种类型安全的方法来配置Spring容器。由于Java 5.0对泛型的支持，现在可以按类型而不是按名称检索bean，不需要任何强制转换或基于字符串的查找。

## Spring Boot中的监视器是什么？

Spring boot actuator是spring启动框架中的重要功能之一。Spring boot监视器可帮助您访问生产环境中正在运行的应用程序的当前状态。有几个指标必须在生产环境中进行检查和监控。即使一些外部应用程序可能正在使用这些服务来向相关人员触发警报消息。监视器模块公开了一组可直接作为HTTP URL访问的REST端点来检查状态。

## 如何在Spring Boot中禁用Actuator端点安全性？

默认情况下，所有敏感的HTTP端点都是安全的，只有具有ACTUATOR角色的用户才能访问它们。安全性是使用标准的HttpServletRequest.isUserInRole方法实施的。 我们可以使用
`management.security.enabled = false`
来禁用安全性。只有在执行机构端点在防火墙后访问时，才建议禁用安全性。

## 如何在自定义端口上运行Spring Boot应用程序？

在application.properties中指定端口：`server.port = 8090 `

spring boot 配置文件有哪几种类型？它们有什么区别？

properties文件和yml文件，格式不同。

## spring boot 有哪些方式可以实现热部署？

- 使用maven插件依赖的springloaded，运行命令springboot:run.
- 使用spring-boot-devtools，shift+ctrl+alt+"/" （IDEA中的快捷键） 选择"Registry" 然后勾选 compiler.automake.allow.when.app.running



## jar文件的启动过程

springboot 将项目文件打包为 fat jar(大包)，通过实现了jar in jar(包内包)的加载方式，自定义的LaunchedURLClassLoader类加载器启动，可以从archive归档文件(jar包或者目录)加载class文件，通过约定的Main函数JarLauncher启动，创建一个新的线程，将自己写的启动类反射创建一个实例，然后启动。

springboot通过一个jar包启动的原因是：在多功能的URL构造器传入处理函数的参数，通过自定义的 URLStreamHandler 处理类处理多重jar in jar的逻辑，用SoftReference软引用来缓存jar文件，这样支持多个‘!/’分割符来表示归档文件内的资源，扩展 JarFile 和 JarURLConnection 协议。

IDE里所有的jar都已经在classpath类目录，可以直接启动。如果解压到文件系统，启动流程与jar类似。

如果是嵌入的tomcat/jetty服务器，首先通过检查servlet类和webAC判断是否在web环境，分别创建注解配置的应用上下文。启动时创建临时文件夹。

对于错误页面，Spring boot也是通过创建一个BasicErrorController来统一处理的。避免了传统的web应用来出错时，默认抛出异常而容易泄密。
spring boot应用的maven打包过程

先通过maven-shade-plugin生成一个包含依赖的jar，再通过spring-boot-maven-plugin插件把spring boot loader相关的类，还有MANIFEST.MF打包到jar里。
spring boot里有颜色日志的实现

当在shell里启动spring boot应用时，会发现它的logger输出是有颜色的，这个特性很有意思。

## 如何使用Spring Boot实现异常处理？

Spring提供了一种使用ControllerAdvice处理异常的非常有用的方法。 我们通过实现一个ControlerAdvice类，来处理控制器类抛出的所有异常。

## 我们如何监视所有Spring Boot微服务？

Spring Boot提供监视器端点以监控单个微服务的度量。这些端点对于获取有关应用程序的信息（如它们是否已启动）以及它们的组件（如数据库等）是否正常运行很有帮助。

批量监控将使用[spring-boot-admin](https://github.com/codecentric/spring-boot-admin)的开源项目。 它建立在Spring Boot Actuator之上，它提供了一个Web UI，使我们能够可视化多个应用程序的度量。

## **为什么我们需要 spring-boot-maven-plugin?** 

spring-boot-maven-plugin 提供了一些像 jar 一样打包或者运行应用程序的命令。

- spring-boot:run 运行你的 SpringBooty 应用程序。
- spring-boot：repackage 重新打包你的 jar 包或者是 war 包使其可执行
- spring-boot：start 和 spring-boot：stop 管理 Spring Boot 应用程序的生命周期（也可以说是为了集成测试）。
- spring-boot:build-info 生成执行器可以使用的构造信息。

什么是 spring boot？

\105. 为什么要用 spring boot？

\106. spring boot 核心配置文件是什么？

\107. spring boot 配置文件有哪几种类型？它们有什么区别？

\108. spring boot 有哪些方式可以实现热部署？

\109. jpa 和 hibernate 有什么区别？
