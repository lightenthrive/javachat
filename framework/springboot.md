## 作用

简化了spring的jar包配置

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