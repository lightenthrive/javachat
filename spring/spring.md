# Spring容器

为什么要使用 spring？

\91. 解释一下什么是 aop？

\92. 解释一下什么是 ioc？

\93. spring 有哪些主要模块？

\94. spring 常用的注入方式有哪些？

\95. spring 中的 bean 是线程安全的吗？

\96. spring 支持几种 bean 的作用域？

\97. spring 自动装配 bean 有哪些方式？

\98. spring 事务实现方式有哪些？

\99. 说一下 spring 的事务隔离？

\100. 说一下 spring mvc 运行流程？

\101. spring mvc 有哪些组件？

\102. @RequestMapping 的作用是什么？

\103. @Autowired 的作用是什么？

## Spring容器有几种？

BeanFactory是最简答的容器，提供了基本的DI支持。最常用的BeanFactory实现就是XmlBeanFactory类。

ApplicationContext扩展了BeanFactory的功能，提供面向应用的服务。

通过缓存在map中，实现了类的复用。

## beanfactory和applicationcontext是什么关系，使用有什么区别。

ApplicationContex提供了一种解析文本消息的方法，一种加载文件资源（如图像）的通用方法，它们可以将事件发布到注册为侦听器的bean。此外，可以在应用程序上下文中以声明方式处理容器中的容器或容器上的操作，这些操作必须以编程方式与Bean Factory一起处理。ApplicationContex实现MessageSource，一个用于获取本地化消息的接口，实际的实现是可插入的。

## FactoryBean如何使用？

一般情况下，Spring通过反射机制利用bean的class属性指定实现类来实例化bean. 如果类的配置复杂，那么就可以实现一个FactoryBean<T>接口的工厂类，在getObject()方法中定制实例化逻辑。当使用这个类的时候，Spring通过反射机制发现这个类实现了该工厂接口，就通过getObject()方法返回实例。如果要获得工厂类本身的实例，则需要在beanName前面加"&"。

## DispatchServlet是怎么分发任务的？

# AOP和IOC

## 什么是AOP？

面向方面的编程（AOP）是一种编程技术，它允许程序员模块化横切关注点或行为，这些问题或行为跨越典型的责任分工，例如日志记录和事务管理。

## Spring AOP中的关注点和交叉关注点之间有什么区别？

关注点是我们希望在应用程序模块中拥有的行为。关注点可以定义为我们想要实现的功能。

跨领域的关注点是一个适用于整个应用程序的问题，它会影响整个应用程序。例如，日志记录，安全性和数据传输是应用程序几乎每个模块都需要的问题，因此它们是跨领域的问题。

## AOP和IOC的原理是什么？

**ioc控制反转**是由Spring容器负责创建对象，管理对象（DI），装配对象，配置对象，并且管理这些对象的整个生命周期。

**di依赖注入**是spring容器根据描述配置将被依赖的类通过构造器或者setter方法注入正在被实例化的类。

**aop切面编程**是通过抽取公共的日志、权限、事务等切面逻辑，通过jdk或者cglib的动态代理生成代理类将增强代码织入原代码中。合并类的公共处理逻辑可以减少重复代码和耦合，侵入性小，便于容器测试。

AOP 和 IOC是Spring精华部分，AOP可以看做是对OOP的补充，对代码进行横向的扩展，通过代理模式实现，代理模式有静态代理，动态代理，Spring利用的是动态代理，在程序运行过程中将增强代码织入原代码中。

## spring 常用的注入方式有哪些？

Spring通过DI（依赖注入）实现IOC（控制反转），常用的注入方式主要有三种：

1. 构造方法注入
1. setter注入
1. 基于注解的注入

## 如何创建动态AOP代理？

创建代理的步骤：获取增强方法/增强器，根据增强方法/增强器进行代理。

目标对象如果实现了接口，默认通过JDK代理，也可以强制cglib代理。如果没有实现接口，就必须使用cglib库，Spring会自动实现jdk动态代理和cglib之间的转换。

加载时织入(LTW)是在虚拟机载入字节码文件是动态植入AspectJ切面。LTM参数在虚拟机层面的的设置不够具体，Spring的对LTW的设置可以在类加载器的粒度上打开，通过外部增强实现效果，就不必在工程内部修改代码。

## 如何创建静态AOP代理？

将动态代理改为静态代理，配置文件需要添加："<context:load-time-weaver />"，然后在META-INF文件夹下建立aop.xml：

```
<!DOCTYPE aspectj PUBLIC "-//AspectJ//DTD//EN" "http://www.eclipse.org/aspectj/dtd/aspectj.dtd">
<aspectj>
<weaver><include within="xx.*" /></weaver>
<aspects><aspect name="xx.AspectConfig" /></aspects>
</aspectj>
```

最后如果用eclipse启动，则加入启动参数
"-javaagent:\path\org.springframework.instrumentjar"
自动选择AspectJ代理的依据就是META-INF/aop.xml文件的存在。

## 动态自定义标签

注册AnnotationAwareAspectJAutoProxyCreator

```
//强制cglib代理，但不能代理final方法，且需要cglib的jar包
<aop:config proxy-target-class="true">
//强制cglib代理和@AspectJ的自动代理配置类的支持
<aop:aspectj-autoproxy proxy-target-class="true">
//如果要求被增强的方法中同时方法内被调用方法的切面，那么就继续添加expose-proxy="true"属性，并把this.b()修改为((AService)AopContext.currentProxy()).b()
//这样就可以对2个方法同时增强
```

# Spring Bean

## Spring bean的定义和作用域是什么？

Spring Beans是构成Spring应用程序主干的Java对象。它们由Spring IoC容器实例化，组装和管理。这些bean是使用提供给容器的配置元数据创建的，例如，以XML定义的形式。

bean的作用域包括单例，原型，请求，回话，全局。

## Bean的生命周期是什么？

1. 创建Bean的实例；
1. 按照配置注入属性；
1. 调用可能实现的BeanNameAware接口方法，传参id。
1. 调用可能实现的BeanFactoryAware接口方法，传参Spring工厂。
1. 调用可能实现的ApplicationContextAware接口方法，传参Spring上下文。
1. 调用可能关联的BeanPostProcessor接口的初始化前方法。
1. 调用可能配置的init-method初始化方法。
1. 调用可能关联的BeanPostProcessor接口的初始化后方法。开始使用。
1. 销毁时，调用可能实现的DisposableBean的destroy方法。
1. 最后，调用可能配置的destroy-method销毁方法。

## Bean的加载过程是什么？

1. 转换对应的beanName。
1. 尝试从缓存中加载单例。
1. bean的实例化。
1. 对原型模式检查类的依赖。
1. 检查父类工厂。
1. 转化bean的定义类。
1. 寻找依赖。
1. 针对不同的scope创建bean。
1. 类型转换。

## Spring中如何让A和B两个bean按顺序加载？

用dependon注解依赖关系。

## 什么是循环依赖，如何解决？

循环依赖就是循环引用，方法之间的环调用，构成有向环。
Spring的构造器循环依赖将正在创建的beanName(id)标志符记录到“当前创建bean池”（构造状态表），构造所需的类的beanName继续添加到表中，如果已有记录，就说明有环结构，抛出循环依赖的异常。

Spring的setter注入的循环依赖是通过提前暴露刚构造完（尚未setter注入）的bean来完成的，且只能解决单例范围的依赖。通过提前暴露一个单例工厂方法，使其他bean能引用到该bean，把beanName标志符加入到“当前创建bean池”中，然后setter注入后续类，后续类因此创建单例，加入自己的标志符。当后续类检测到setter需要的类已经位于池中，就通过该标志符在另一个表中找到对应的ObjectFactory工厂，进而返回工厂类创建的bean. 然后在第二个循环中依次setter注入工厂类创建的bean。

Spring的prototype作用域的bean, Spring容器无法完成依赖注入，因为不缓存该作用域的bean,因此无法提前暴露。

## Spring如何解决循环依赖问题？

什么是循环依赖，怎样检测出循环依赖，Spring循环依赖有几种方式，使用基于setter属性的循环依赖为什么不会出现问题，接下来会问：Bean的生命周期。

## 如何获取单例？

在全局变量加锁的情况下。检查singletonObjects缓存类中是否已加载，没有加载就把beanName记录到加载状态表，通过ObjectFactory的方法得到实例化bean，从加载状态表中移除这个beanName, 缓存实例并删除其他状态表的记录。

## 如何从缓存中获取单例bean？

创建单例的时候会存在依赖注入的情况，而在创建依赖的时候为了避免循环依赖，Spring创建Bean的原则是不等bean创建完成就会将创建bean的ObjectFactory加入到缓存，一旦下一个bean创建时需要依赖上个bean，就直接使用ObjectFactory.

## 如何从bean的实例中获取对象？

加载bean后通过getObjectForBeanInstance方法检测当前bean是否是FactoryBean类型，是则调用该类型的工厂方法返回真正需要的bean实例，并进行后处理。

## Bean 是线程安全的吗？

单例的bean不安全，prototype和request作用域的，安全。

通过无状态的设计实现线程安全。

## 创建bean的实例

初始化前先解析，如果已经创建了代理或者在初始化前的后处理器方法中改变了bean, 则直接返回就可以了。否则需要进行常规bean的创建。

**创建过程**包括：清除单例缓存，创建bean的实例(将BeanDefinition转换为BeanWrapper)，合并类定义的后处理器类解析父类和注解等，依赖处理，属性填充，检查循环依赖，注册DisposableBean, 完成创建并返回。

创建bean的实例，优先使用根定义类的工厂方法实例化，解析构造函数并构造实例。构造实例，要先检查缓存的构造器的唯一解析结果，没解析过的要重新先解析。然后对这个解析结果进行自动注入构造或者默认构造器构造(直接实例化)。

自动注入构造，初始化一个新的类包装器，依次从指定传参、根类定义缓存和配置中获取构造方法参数列表（并转换类型）和参数的个数，从指定传参或者反射获取构造器数组，按照构造器参数从多到少和公开优先的顺序排序，遍历解析构造器并加入缓存，参数类型转换，验证构造函数不是父类重写关系，根据实例化策略和构造函数与参数实例化bean.

实例化策略，如果没有使用replace(覆盖方法)或者lookup(动态替换)配置的方法，直接反射即可实例化。否则就需要使用cglib进行动态代理，将动态的拦截器增强切面方法织入类中，返回代理类。

记录创建bean的ObjectFactory。属性注入，包括根据根据名称、类型注入。

初始化bean，包括激活Aware三个方法，后处理器的前后使用，激活自定义的init方法。

ApplicationContext是对BeanFactory的功能扩展，详见refresh()函数对AC的初始化：
    
准备刷新上下文环境（准备并验证系统属性或者环境变量），初始化FactoryBean并读取xml，填充FB的功能(如自动注入的注解)，开发者定制的子类覆盖方法(postProcessBeanFactory)执行，FB的后处理器执行，注册拦截bean创建的bean处理器（获取bean时调用），为上下文初始化Message国际化语言源，初始化应用消息广播器并AEM中，留给子类来初始化其他的bean, 在注册的bean中查找监听器bean并注册，初始化剩下的单例，完成刷新过程后，通知生命周期处理器刷新过程，同时发出上下文刷新时间通知其他类。

开发者定制的工厂类后处理方法，为类的创建提供了灵活性。

## Bean 的创建过程是什么？

根据class属性或名称解析类，通过override属性的同名方法个数验证合理性或者标记为没有重载方法，然后进行初始化前的预处理，创建bean.

lookup-method和replace-method两个配置功能的加载就是设置到RootBeanDefinition的methodOverrides属性中。实现原理是在bean实例化时如果检测到存在methodOverrides属性，就会动态地为当前bean生成代理并使用对应的拦截器为bean作增强处理。

实例化的前置处理过程的bean经短路判断非空，则直接返回这个bean，忽略后续bean的创建。AOP功能就是基于这种判断。

# 设计

## Spring的设计模式有几种？

9种。

1. 简单工厂：FactoryBean。
1. 工厂方法：xml文件的factory-bean属性指定工厂方法。
1. 单例模式：默认唯一的访问点是BeanFactory访问点。
1. 适配器模式：HanderAdapter，AdvisorAdaptor。
1. 装饰器模式：Wrapper，Decorator。
1. 代理模式：AOP功能的原理就使用代理模式。
1. 观察者模式：监听器。
1. 策略模式：实例化对象的时候使用策略模式。
1. 模板方法模式：JdbcTemplate。

## springmvc的工作过程

servlet收到用户请求，解析URL得到URI；

调用HandlerMapping获得该Handler配置的所有相关的对象（包括Handler对象以及Handler对象对应的拦截器），最后以HandlerExecutionChain对象的形式返回；

选择一个合适的HandlerAdapter，执行

把请求分发到具体的HandlerAdapter处理，把得到的结果数据封装到ModelAndView对象，渲染后返回前台。

## 事务管理类型的Spring支持有哪些？

Spring支持两种类型的事务管理：

程序化事务管理： 这意味着您已经在编程的帮助下管理了事务。这为您提供了极大的灵活性，但很难维护。 

声明式事务管理： 这意味着您将事务管理与业务代码分开。您只能使用注释或基于XML的配置来管理事务。

## Spring Framework的事务管理有哪些好处？

它在不同的事务API（如JTA，JDBC，Hibernate，JPA和JDO）之间提供了一致的编程模型。 与许多复杂的事务API（如JTA）相比，它为程序化事务管理提供了更简单的API。 它支持声明式事务管理。 它与Spring的各种数据访问抽象集成得非常好。

## 哪种交易管理类型更可取？

Spring Framework的大多数用户选择声明式事务管理，因为它是对应用程序代码影响最小的选项，因此最符合非侵入式轻量级容器的理想。声明式事务管理优于程序化事务管理，但它不如程序化事务管理灵活，后者允许您通过代码控制事务。