# OOP

## 什么是多态?

多态包括方法重载的静态多态，以及继承父类和实现接口的动态多态。

## 虚拟机是如何实现多态的

动态绑定技术使用了dynamic字节码指令，运行期间判断引用对象的实际类型，根据实际类型调用对应的方法。

## 接口的意义

规范，扩展，回调。

## 抽象类的意义

定义一个用于子类继承的模板类。封装公共变量和抽象方法。

定义抽象方法，子类虽然有不同的实现，但是定义时一致的

## 抽象类能使用 final 修饰吗？

不能，因为抽象类就是为继承而存在，final不允许继承。

## 抽象类必须要有抽象方法吗？

是的。

## 重写与重载的区别？

重写（Override）是子类完全覆盖父类的方法。里式替换原则要求访问权限可以变大，子类方法的返回类型可以变小。使用 @Override 注解，可以让编译器帮忙检查是否满足上面的两个限制条件。

重载（Overload）是同一个类中的同名方法的参数签名不同，与返回值无关。

## 抽象类与接口的区别？

相同点是：抽象类的抽象方法和接口的方法都是抽象的，需要被继承或者实现，然后才能被实例化。

不同点是：接口的方法只能是 public 的，而抽象类的方法不能是private。Java8的接口可以拥有默认的方法实现，抽象类没有。

接口的字段只能是 static 和 final 类型的，而抽象类的字段没有这种限制。

# 数据类型

## java的基本类型的默认长度和默认值是什么？


| 类型 | byte | char | short | int | float | long | double | boolean |
| :---------- | ---- | ---- | ----| ---- | ---- | ---- | ---- | ---- |
| 长度 | 8 | 16 | 16 | 32 | 32 | 64 | 64 | 1 |
| 默认值 | 0 | 0 | 0 | 0 | 0.0 | 0 | 0.0 | false |

boolean 只有两个值：true、false，可以使用 1 bit 来存储，但是具体大小没有明确规定。JVM 会在编译时期将 boolean 类型的数据转换为 int，使用 1 来表示 true，0 表示 false。JVM 并不直接支持 boolean 数组，而是使用 byte 数组来表示 int 数组来表示。

## 哪些数据类型有常量池？

|类型|boolean|byte|short|int|char|String|
|--|--|--|--|--|--|--|
|范围|true/false|all byte|[-128,127]|[-128,127]|[\u0000,\u007F]|用过的|

## 常量池在虚拟机的什么位置？

堆区，被静态属性引用，不可回收。

## java当中使用什么类型表示价格比较好?

如果不是特别关心内存和性能的话，使用BigDecimal，否则使用预定义精度的 double 类型。

## 对象不可变的好处是什么？

1. 可以缓存hash值。
1. 可构成常量池。
1. 不会被修改而入参安全。
1. 具有内存可见性，不需要线程同步。

## 如何构造一个不可变对象？

对象的类用final修饰，避免子类重写方法。

对象的成员变量用final修饰，并在构造器中初始化，初始化时保证没有泄漏对象引用。

不提供修改方法，get成员变量时返回对象副本。重写hashCode()方法和equals()方法。

## String内部存储的数据结构是什么？

在 Java 8 中，String 内部使用 char 数组存储数据。
在 Java 9 之后，String 类的实现改用 byte 数组存储字符串，同时使用 `coder` 来标识使用了哪种编码。

## subString方法是否会引起内存泄漏？

jdk1.6中，当调用substring方法的时候，会创建一个新的string对象，但是这个string的值仍然指向堆中的同一个字符数组。这两个对象中只有count和offset 的值是不同的。可能引起内存泄漏。

如果加上一个空字符串，生成新的对象，就可以解决。

jdk1.7直接生成新的字符串，已经解决。

## StringBuffer和StringBuilder的区别是什么？

StringBuilder 不是线程安全的。StringBuffer是线程安全的，内部使用 synchronized 进行同步。

## String的intern方法的作用是什么？

字符串常量池在编译时期就确定了。intern()方法会首先从常量池中查找，如果不存在就创建，然后直接返回。

Java7之前的String Pool被放在运行时常量池中，属于永久代，容易发生OOM。Java7以后的String Pool被移到堆中。

## 字符串拼接方法有几种？

concat方法，符号"+"的语法糖（不是运算符重载），StringBuffer，StringBuilder，StringUtils.join。

## 字符串相加是怎样被编译器优化的？

final修饰的变量字符串的相加，会被编译器按照StringBuilder方式优化。

字面量字符串的相加，会被编译器优化成一个字符串。

## 为什么不建议在循环体中使用`+`进行字符串拼接？

因为每次相加都会new一个StringBuild对象，浪费

## 字符串相加的对象与字面量对象是不是同一个对象？

String a = "ab"; String b = "a" + "b"; a == b 不相等，因为前者是字面量对象，后者是新的对象，值指向常量池中的字面量。

## String.valueOf和Integer.toString的区别

功能上分别是字符串到包装对象，以及从包装对象到字符串。

## 如何将byte转为String

可以使用 String 接收 byte[] 参数的构造器来进行转换，需要注意的点是要使用的正确的编码，否则会使用平台默认编码，这个编码可能跟原来的编码相同，也可能不同。

## 可以将int强转为byte类型么?会产生什么问题?

我们可以做强制转换，但是Java中int是32位的而byte是8 位的，所以,如果强制转化int类型的高24位将会被丢弃，byte 类型的范围是从-128到128。

## Java反转字符串的10种方法

[点击查看](http://www.importnew.com/30579.html)

# 运算

## == 和 equals 的区别是什么？

==是运算符，equals是对象方法。

用于对象相等的判断时，==表示同一个对象。equals表示对象内容相同。

## java的参数传递是传值吗？

Java 的参数是以值传递的形式传入方法中，引用参数传递的是引用的副本。

## 这个运行结果是什么？

```java
public class Test { 
    public static void main(String[] args){
        // 实数比较
        System.out.println(Math.min(Double.MIN_VALUE, 0.0d)); 
        // 字符串编码
        char[] chars = new char[] {'\u0097'}; 
        String str = new String(chars);
        byte[] bytes = str.getBytes(); 
        System.out.println(Arrays.toString(bytes)); 
        // 实数除零运算
        System.out.println(1.0 / 0.0); 
    } 
}
```

输出`0`。因为Double.MIN_VALUE是大于0的最小实数。

输出可能为`[-62, -105]`。因为默认编码方案与操作系统和区域设置相关。

输出`Infinity`。因为不会抛出ArithmeticExcpetion并返回Double.INFINITY。

## 这个比较方法有什么问题？

```java
public int compareTo(Object o){ 
    Employee e = (Employee) o; 
    return this.id - e.id; 
}
```

要保证id是整数，且不会溢出。

## float与double能不能相等？

3*0.1 == 0.3 将会返回false，因为前者是float类型，后者是double类型。

## 这两种加法有没有隐式类型转换？

short s1 = 1; s1 = s1 + 1; 是错的，因为s1+1被转换类型为double, 不能向精度更低的short赋值。

short s1 = 1; s1 += 1; 是对的，因为1被隐式转换为short类型，直接加到s1上。

## &操作符和&&操作符有什么区别?](&操作符和&&操作符有什么区别?)

&操作符是位运算，&&操作符是短路与运算。

涉及到重写时，方法调用的优先级为：

- this.show(O)
- super.show(O)
- this.show((super)O)
- super.show((super)O)

## 如何正确的退出多层嵌套循环

1. 使用标号和break。
1. 通过在外层循环中添加标识符。


# 关键字

## final关键字的作用是什么？

修饰类：表示该类不能被继承。

修饰方法：表示方法不能被重写，虚拟机内联优化。private 方法隐式地被指定为 final。如果在子类中定义的方法和基类中的一个 private 方法签名相同，此时子类的方法不是重写基类方法，而是在子类中定义了一个新的方法。

修饰变量：表示变量只能在编译时或者初始化后赋值一次。基本类型和引用都不变。但是被引用的对象或者数组可以修改成员变量或者数组元素。

修饰常量：编译阶段存入常量池。

局部内部类定义只能访问被final修饰的局部变量。

## final关键字的两个重排序规则是什么？

1. 先在构造函数内初始化final变量,然后把这个对象赋值给一个引用变量,这两个操作之间不能重排序. 
1. 先读取包含final域的对象的引用,然后读取这个final域,这两个操作之间不能重排序.

## final,finally,finalize的区别是什么？

final是关键字。

finally是try块后必须执行的统一出口，用于资源清理。

finalize方法在对象回收时被调用。

## 能否在运行时向'static final'修饰的变量赋值？

不能，被static final修饰的变量只能在被定义的时候或者类的静态代码块中初始化，赋值后就不能改变引用。

## static关键字的作用是什么？

1. 静态变量：又称为类变量，也就是说这个变量属于类的，类所有的实例都共享静态变量，可以直接通过类名来访问它。静态变量在内存中只存在一份。

1. 静态方法：在类加载的时候就存在了，它不依赖于任何实例。所以静态方法必须有实现，也就是说它不能是抽象方法。只能访问所属类的静态字段和静态方法，方法中不能有 this 和 super 关键字。

1. 静态代码块：在类初始化时运行一次。

1. 静态内部类：不需要依赖于外部类的实例。不能访问外部类的非静态的变量和方法。

1. 静态导包：在使用静态变量和方法时不用再指明 ClassName，从而简化代码，但可读性大大降低。

1. 初始化顺序：静态变量和静态代码块同级，优先于实例变量和普通语句块。

## 静态类型有什么特点？

1.随着类的加载而加载，和类绑定到一起的。静态变量引用的对象在堆区，不回收，直到程序结束后销毁，生命周期最长。
2.早于实例对象的存在。
3.被所有对象共享。
4.通常通过类名来访问，静态方法不作用于实例对象，也不需要创建任何的类实例。
5.静态方法本身就是final的，因为重写只会发生在类实例上。没有声明为final的父类的静态方法会被子类的静态方法屏蔽，但这不是重写。

## 为什么静态方法不能访问成员变量？

静态方法会随着类的加载而被加载，调用时含有成员变量的对象可能还没有创建。

## 静态变量是在编译期还是运行期加载？

静态变量的加载创建，在类加载器将类加载到虚拟机方法区的时候，属于编译期和运行期中间的过渡时期。

## java中是否可以覆盖private或者是static的方法？

不能在Java中覆盖私有或静态方法，如果你在子类中创建一个具有相同返回类型和相同方法参数的类似方法，那么它将隐藏超类方法，这称为方法隐藏。

private修饰的方法，不能被继承，所以也不存在重写（覆盖）。

static修饰的方法，是静态方法，在编译时就和类名就行了绑定。而重写发生在运行时，动态绑定的。何况static方法，跟类的实例都不相关，所以概念上也适用。

## private方法可以通过反射访问，那么private的意义是什么？

private的意义在于遵守面向对象原则的友好提示。按照规矩写代码，可以保证不出事故。

## transient关键字的作用是什么？

禁止序列化和反序列化。

## super关键字的作用是什么？

子类重写了父类的方法,又想用父类该方法的时候。

## switch 语句中的表达式可以是什么类型数据？

String,Integer等不可变对象。

# 类

## 类加载的方法有哪些？

1. 在命令行启动虚拟机进行加载。
1. 用class.forname()方法进行动态加载，static语句块在加载时执行。调用了NewInstance()方法，用构造函数创建对象的时候可以自己定义是否加载static。
1. 用ClassLoader.loadClass()进行动态加载，static语句块在创建对象时执行。

## 一个java文件有3个接口或类，编译后有几个class文件？

3个。

## 什么是编译期常量?使用它有什么风险?

公共静态不可变（public static final ）变量也就是我们所说的编译期常量，这里的 public 可选的。实际上这些变量在编译时会被替换掉，因为编译器知道这些变量的值，并且知道这些变量在运行时不能改变。

这种方式存在的一个问题是你使用了一个内部的或第三方库中的公有编译时常量，但是这个值后面被其他人改变了，但是你的客户端仍然在使用老的值，甚至你已经部署了一个新的jar。为了避免这种情况，当你在更新依赖 JAR 文件时，确保重新编译你的程序。

## 类的初始化顺序是什么？

1. 父类（静态变量、静态语句块）
1. 子类（静态变量、静态语句块）
1. 父类（成员变量、普通语句块）
1. 父类（构造函数）
1. 子类（成员变量、普通语句块）
1. 子类（构造函数）

## 内部类的作用是什么？

内部类可以有多个实例,每个实例都有自己的状态信息,并且与其他外围对象的信息相互独立.在单个外围类当中,可以让多个内部类以不同的方式实现同一接口,或者继承同一个类.创建内部类对象的时刻不依赖于外部类对象的创建.内部类并没有令人疑惑的”is-a”关系,它就像是一个独立的实体.

内部类提供了更好的封装,除了该外围类,其他类都不能访问

## 类对象的相等判断方法有哪些？

类的equals(), isAssignableFrom(), isInstance()方法和实例对象的instanceof关键字。

## 深拷贝和浅拷贝的区别是什么？

浅拷贝的对象引用指向同一个对象。

深拷贝的对象引用指向不同的对象。

## clone方法的重写，为什么要实现Cloneable接口？

clone() 方法并不是 Cloneable 接口的方法，而是 Object 的一个 protected 方法。Cloneable 接口只是规定，如果一个类没有实现 Cloneable 接口又调用了 clone() 方法，就会抛出 CloneNotSupportedException。

最好不要去使用 clone()，可以使用拷贝构造函数或者拷贝工厂来拷贝一个对象。

## 如何深克隆一个类？

实现Cloneable接口，重写clone()方法。或者使用序列化和反序列化的方式。

## 静态变量和实例变量的区别?

静态变量存储在方法区，属于类所有。实例变量存储在堆当中，其引用存在当前线程栈。

## 局部变量使用前需要显式赋值，否则不能通过编译，为什么？

局部变量的作用域限制在方法体中，赋值和取值的访问顺序由happens-before原则确定。使用前显式赋值可以避免人们因为忘记赋值而造成事故。

而成员变量赋值和取值访问的顺序在编译期不确定，需要虚拟机在运行时决定，因此不赋值也不会报错。

## Object的9个方法是什么？

```java
public native int hashCode()

public boolean equals(Object obj)

protected native Object clone() throws CloneNotSupportedException

public String toString()

public final native Class<?> getClass()

protected void finalize() throws Throwable {}

public final native void notify()

public final native void notifyAll()

public final native void wait(long timeout) throws InterruptedException

public final void wait(long timeout, int nanos) throws InterruptedException

public final void wait() throws InterruptedException
```

## equals方法和instanceof关键字的区别是什么？

## hashCode方法有什么用途？

hashCode()方法返回对象的散列值，与equals()方法一起确认对象是否相等。

## 为什么hash计算经常与31相乘？

31是中等大小的质数，可以散列均匀。与 31 相乘可以被编译器优化成移位和减法：`31*x == (x<<5)-x`。

## toString方法的返回值是什么意思？

默认返回`类名@无符号十六进制的散列码`。

## 线程类的构造方法、静态块是被哪个线程调用的

线程类的构造方法、静态块是被new这个线程类所在的线程所调用的，而run方法里面的代码才是被线程自身所调用的。

## API、API和SPI的关系和区别

## 如何定义SPI、SPI的实现原理

# 时间处理

## 时区、冬令时和夏令时、时间戳、Java中时间API

## 格林威治时间、CET,UTC,GMT,CST几种常见时间的含义和关系

## SimpleDateFormat的线程安全性问题

## Java 8中的时间处理

## 如何在东八区的计算机上获取美国时间

# 反射

什么是反射？

\58. 什么是 java 序列化？什么情况下需要序列化？

\59. 动态代理是什么？有哪些应用？

\60. 怎么实现动态代理？

## 反射的功能和优缺点是什么？

反射可以提供运行时的类信息，并且这个类可以在运行时才加载进来，甚至在编译时期该类的 .class 不存在也可以加载进来。动态修改Field，Method，Constructor。

反射的优点：

可以利用全限定名创建可扩展对象的实例，来使用来自外部的用户自定义类。

类浏览器和IDE需要利用反射枚举类的成员和类型。

调试器需要能够检查一个类里的私有成员。测试工具可以利用反射来自动地调用类里定义的可被发现的 API 定义，以确保一组测试中有较高的代码覆盖率。

反射的缺点：

性能开销：反射涉及动态类型的解析，所以 JVM 无法对这些代码进行优化。

安全限制：使用反射技术要求程序必须在一个没有安全限制的环境中运行。

暴露对象内部的私有的熟悉和方法，破坏封装性，可能导致代码功能失调并破坏可移植性。

## 谈谈 Java 反射机制，动态代理是基于什么原理？

反射机制是 Java 语言提供的一种基础功能，赋予程序在运行时自省（introspect，官方用语）的能力。通过反射我们可以直接操作类或者对象，比如获取某个对象的类定义，获取类声明的属性和方法，调用方法或者构造对象，甚至可以运行时修改类定义。

动态代理是一种方便运行时动态构建代理、动态处理代理方法调用的机制，很多场景都是利用类似机制做到的，比如用来包装 RPC 调用、面向切面的编程（AOP）。

实现动态代理的方式很多，比如 JDK 自身提供的动态代理，就是主要利用了上面提到的反射机制。还有其他的实现方式，比如利用传说中更高性能的字节码操作机制，类似 ASM、cglib（基于 ASM）、Javassist 等。

# 异常

## 异常的种类有哪些？

![img](https://mmbiz.qpic.cn/mmbiz_png/YriaiaJPb26VMsSY4bkABCq0Ticr2AgMT1mcfIBPNRSHiaM05RXMZNvr10ANib6EMqVzOab5GmhsnNtiaYCQBfGWcn3A/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)Exception 和 Error 都是继承了 Throwable 类，在 Java 中只有 Throwable 类的实例才可以被抛出（throw）或者捕获（catch），它是异常处理机制的基本组成类型。

Exception 和 Error 体现了 Java 平台设计者对不同异常情况的分类。Exception 是程序正常运行中，可以预料的意外情况，可能并且应该被捕获，进行相应处理。

Error 是指在正常情况下，不大可能出现的情况，绝大部分的 Error 都会导致程序（比如 JVM 自身）处于非正常的、不可恢复状态。既然是非正常情况，所以不便于也不需要捕获，常见的比如 OutOfMemoryError 之类，都是 Error 的子类。

Exception 又分为可检查（checked）异常和不检查（unchecked）异常，可检查异常在源代码里必须显式地进行捕获处理，这是编译期检查的一部分。前面我介绍的不可查的 Error，是 Throwable 不是 Exception。

不检查异常就是所谓的运行时异常，类似 NullPointerException、ArrayIndexOutOfBoundsException 之类，通常是可以编码避免的逻辑错误，具体根据需要来判断是否需要捕获，并不会在编译期强制要求。

## throw和throws的区别是什么？

throws在方法上，标明可能抛出的异常。

throw在方法内部，抛出异常。

## finally和return的执行顺序

先执行finally，最后return.

## 如果在try或catch块上放置return语句或System.exit()会退出吗？

try或catch块的return之前，要执行finally块。如果finally里return了，方法就结束了。

在任何代码块执行System.exit()都会导致程序退出。

# 泛型

## 同类型不同泛型的实例对象是否属于同一个类？

```java
ArrayList<Integer> list1=new ArrayList<>();
ArrayList<Integer> list1=new ArrayList<>();
System.out.print(list1.getClass()==list2.getClass());
```

属于同一个类，因为泛型会在编译器被擦除。

## 如何设计泛型接口？

# 枚举

## 枚举的用法、枚举的实现、枚举与单例、Enum类

## Java枚举如何比较

## switch对枚举的支持

## [枚举的序列化如何实现](https://github.com/amlgx/toBeTopJavaer/blob/master/basics/java-basic/enum-serializable.md)

## 枚举的线程安全性问题

# 注解

## 注解的原理是什么？

# 编码方式

## Unicode、有了Unicode为啥还需要UTF-8

## GBK、GB2312、GB18030之间的区别

## UTF8、UTF16、UTF32区别

## URL编解码、Big Endian和Little Endian

## 如何解决乱码问题

# 新技术

## Java 8

lambda表达式、Stream API、时间API

## Java 9

Jigsaw、Jshell、Reactive Streams

## Java 10

局部变量类型推断、G1的并行Full GC、ThreadLocal握手机制

## Java 11

ZGC、Epsilon、增强var、

## Spring 5

响应式编程
Spring Boot 2.0
http/2
http/3

## 性能优化

使用单例、使用Future模式、使用线程池、选择就绪、减少上下文切换、减少锁粒度、数据压缩、结果缓存





