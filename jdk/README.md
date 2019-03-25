# 面向对象编程

## 什么是多态?

多态包括同名不同参数的静态多态，以及父类引用/接口指向子类/实现类的动态多态。

## 抽象类与接口的区别？

相同点是：抽象类的抽象方法和接口的方法都是抽象的，需要被继承或者实现，然后才能被实例化。

不同点是：接口的方法只能是 public 的，而抽象类的方法不能是private。Java8的接口可以拥有默认的方法实现，抽象类没有。

接口的字段只能是 static 和 final 类型的，而抽象类的字段没有这种限制。

## 重写与重载的区别？

重写（Override）是子类完全覆盖父类的方法。里式替换原则要求访问权限可以变大，子类方法的返回类型可以变小。使用 @Override 注解，可以让编译器帮忙检查是否满足上面的两个限制条件。

重载（Overload）是同一个类中的同名方法的参数签名不同，与返回值无关。

## 类的实例化的顺序是什么？

1. 父类（静态变量、静态语句块）

1. 子类（静态变量、静态语句块）

1. 父类（成员变量、普通语句块）

1. 父类（构造函数）

1. 子类（成员变量、普通语句块）

1. 子类（构造函数）

## 类对象的相等判断方法有哪些？

类的equals(), isAssignableFrom(), isInstance()方法和实例对象的instanceof关键字。

# 数据类型

## java的基本类型的默认长度和默认值是什么？

|--|--|--|--|--|--|--|--|--|
|类型|byte|char|short|int|float|long|double|boolean|
|长度|8|16|16|32|32|64|64|1|
|默认值|0|0|0|0|0.0|0|0.0|false|

boolean 只有两个值：true、false，可以使用 1 bit 来存储，但是具体大小没有明确规定。JVM 会在编译时期将 boolean 类型的数据转换为 int，使用 1 来表示 true，0 表示 false。JVM 并不直接支持 boolean 数组，而是使用 byte 数组来表示 int 数组来表示。

## 哪些数据类型有常量池？

|--|--|--|--|--|--|--|
|类型|boolean|byte|short|int|char|String|
|范围|true/false|all byte|[-128,127]|[-128,127]|[\u0000,\u007F]|用过的|

## 常量池在虚拟机的什么位置？

堆区，被静态属性引用，不可回收。

## 对象不可变的好处是什么？

可以缓存hash值，可构成常量池，不会被修改而入参安全，不需要线程同步。

## 如何构造一个不可变对象？

对象的类用final修饰，避免子类重写方法。

对象的成员变量用final修饰，并在构造器中初始化，初始化时保证没有泄漏对象引用。

不提供修改方法，get成员变量时返回对象副本。重写hashCode()方法和equals()方法。

## String内部存储的数据结构是什么？

在 Java 8 中，String 内部使用 char 数组存储数据。
在 Java 9 之后，String 类的实现改用 byte 数组存储字符串，同时使用 `coder` 来标识使用了哪种编码。

## StringBuffer和StringBuilder的区别是什么？

StringBuilder 不是线程安全的。StringBuffer是线程安全的，内部使用 synchronized 进行同步。

## String的intern方法的作用是什么？

字符串常量池在编译时期就确定了。String的intern()方法在运行时期将字符串添加到常量池中，并返回字面量引用。

Java7之前的String Pool被放在运行时常量池中，属于永久代，容易发生OOM。Java7以后的String Pool被移到堆中。

## 字符串相加是怎样被编译器优化的？

final修饰的变量字符串的相加，会被编译器按照StringBuilder方式优化。

字面量字符串的相加，会被编译器优化成一个字符串。

## Java反转字符串的10种方法

[点击查看](http://www.importnew.com/30579.html)

# 运算

## java的参数传递是传值吗？

Java 的参数是以值传递的形式传入方法中，引用参数传递的是引用的副本。

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

# Object的方法有哪些？

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

## 深拷贝和浅拷贝的区别是什么？

浅拷贝的拷贝对象和原始对象的引用类型成员变量引用同一个对象。

深拷贝的拷贝对象和原始对象的引用类型成员变量引用不同对象。

## clone方法的重写，为什么要实现Cloneable接口？

clone() 方法并不是 Cloneable 接口的方法，而是 Object 的一个 protected 方法。Cloneable 接口只是规定，如果一个类没有实现 Cloneable 接口又调用了 clone() 方法，就会抛出 CloneNotSupportedException。

最好不要去使用 clone()，可以使用拷贝构造函数或者拷贝工厂来拷贝一个对象。

## 如何深克隆一个类？

实现Cloneable接口，重写clone()方法。或者使用序列化和反序列化的方式。


## toString方法的返回值是什么意思？

默认返回`类名@无符号十六进制的散列码`。


# 关键字

## final关键字的作用是什么？

修饰类：表示该类不能被继承。

修饰方法：表示方法不能被重写。private 方法隐式地被指定为 final。如果在子类中定义的方法和基类中的一个 private 方法签名相同，此时子类的方法不是重写基类方法，而是在子类中定义了一个新的方法。

修饰变量：表示变量只能在编译时或者初始化后赋值一次。基本类型和引用都不变。但是被引用的对象或者数组可以修改成员变量或者数组元素。

局部内部类定义只能访问被final修饰的局部变量。

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

## Java中是否可以覆盖private或者是static的方法？

都不能。private修饰的方法，不能被继承，所以也不存在重写（覆盖）

static修饰的方法，是静态方法，在编译时就和类名就行了绑定。而重写发生在运行时，动态绑定的。何况static方法，跟类的实例都不相关，所以概念上也适用。

## transient关键字的作用是什么？

禁止序列化和反序列化。

## super关键字的作用是什么？

子类重写了父类的方法,又想用父类该方法的时候。

## switch 语句中的表达式可以是什么类型数据？

String,Integer等不可变对象。

# 反射

## 类加载的方法有哪些？

1. 在命令行启动虚拟机jvm进行加载。

1. 用class.forname()方法进行动态加载，static语句块在加载时执行。调用了NewInstance()方法，用构造函数创建对象的时候可以自己定义是否加载static。

1. 用ClassLoader.loadClass()进行动态加载，static语句块在创建对象时执行。

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

# 异常

## 异常的种类有哪些？

Throwable 可以用来表示任何可以作为异常抛出的类，分为两种： **Error**  和 **Exception**。其中 Error 用来表示 JVM 无法处理的错误，Exception 分为两种：

受检异常，需要用 try...catch... 语句捕获并进行处理，并且可以从异常中恢复；

非受检异常，是程序运行时错误，例如除 0 会引发 Arithmetic Exception，此时程序崩溃并且无法恢复。

## throw和throws的区别是什么？

throws在方法上，标明可能抛出的异常。

throw在方法内部，抛出异常。

# 泛型

## 如何设计泛型接口？


# 注解

# 参考资料

- https://github.com/amlgx/CS-Notes/blob/master/docs/notes/Java\ 基础.md

