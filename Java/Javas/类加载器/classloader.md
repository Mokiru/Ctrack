# 类加载器

- 类加载过程：加载->连接->初始化。
- 连接过程又可分为三部：验证->准备->解析

加载是类加载过程的第一步，主要完成下面 $3$ 件事情：
1. 通过全类名获取定义此类的二进制字节流。
2. 将字节流所代表的静态存储结构转换为方法区的运行时数据结构。
3. 在内存中生成一个代表该类的 `Class` 对象，作为方法区这些数据的访问入口。

## 介绍

类加载器从JDK1.0就出现了，最初只是为了满足Java Applet（已经被淘汰）的需要，后来，慢慢成为Java程序中的一个重要组成部分，赋予了Java类可以被动态加载到JVM中并执行的能力。

类加载器是一个负责加载类的对象。`ClassLoader` 是一个抽象类，给定类的二进制名称，类加载器应尝试定位或生成构成类定义的数据。典型的策略是将名称转换为文件名，然后从文件系统中读取该名称的“类文件”。

每个Java类都有一个引用指向加载它的 `ClassLoader`。不过，数组类不是通过 `ClassLoader` 创建的，而是JVM在需要的时候自动创建的，数组类通过 `getClassLoader()` 方法获取 `ClassLoader` 的时候和该数组的元素类型 `ClassLoader` 是一致的。

- 类加载器是一个负责加载类的对象，用于实现类加载过程中的加载这一步；
- 每个Java类都有一个引用指向加载它的 `ClassLoader`；
- 数组类不是通过 `ClassLoader` 创建的（数组类没有对应的二进制字节流），是由JVM直接生成的。

```java
class Class<T> {
    private final ClassLoader classLoader;

    @CallerSensitive
    public ClassLoader getClassLoader() {
        //....
    }
}
```

简单来说，类加载器的主要作用就是加载Java 类的字节码（`.class` 文件）到JVM中（在内存中生成一个代表该类的`Class` 对象）。字节码可以是Java源程序（`.java`文件）经过 `javac` 编译得来，也可以是通过工具动态生成或者通过网络下载的来。

其实除了加载类之外，类加载器还可以加载Java应用所需的资源如文本、图像、配置文件、视频等等文件资源。

## 加载原则

JVM启动的时候，并不会一次性加载所有的类，而是根据需要去动态加载，也就是说，大部分类在具体用到的时候才会去加载。

对于已经加载的类会被放在 `ClassLoader` 中。在类加载的时候，系统会首先判断当前类是否被加载过，已经被加载的类会直接返回，否则才会尝试加载。也就是说，对于一个类加载器来说，相同二进制名称的类只会被加载一次。

```java
public abstract class ClassLoader{

    private final ClassLoader parent;

    private final Vector<Class<?>> classes = new Vector<>();

    // 由VM调用，用此类加载器记录每个已经加载的类
    void addClass(CLass<?> c) {
        classes.addElement(c);
    }
}
```

## 总结

JVM中内置了三个重要的 `ClassLoader`：
1. `BootstrapCLassLoader` （启动类加载器）：最顶层的加载类，由C++实现，通常表示为null，并且没有父级，主要用来加载JDK内部的核心类库（`%JAVA_HOME%/lib` 目录下的 `rt.jar`、`resources.jar`、`charsets.jar` 等jar包和类）以及被 `-Xbootclasspath` 参数指定的路径下的所有类。
2. `ExtensionClassLoader`（扩展类加载器）：主要负责加载`%JRE_HOME%/lib/ext`目录下的jar包和类以及被`java.ext.dirs` 系统变量所指定的路径下的所有类。
3. `AppClassLoader`（应用程序类加载器）：面向用户的加载器，负责加载当前应用classpaht下的所有jar包和类。

- `rt.jar`：rt代表RunTime，`rt.jar`是Java基础类库，包含Java doc里面看到的所有的类的类文件，也就是说，我们常用内置库`java.xxx.*` 都在里面，比如`java.util.*`、`java.io.*`、`java.nio.*`、`java.lang.*`、`java.sql.*`、`java.math.*`。
- Java9引入了模块系统，并且略微更改了上述的类加载器。扩展类加载器被改名为平台类加载器（platform class loader）。JavaSE中除了少数几个关键模块，比如说 `java.base` 是由启动类加载器加载之外，其他的模块均由平台类加载器所加载。

除了这三种类加载器之外，用户还可以加入自定义的类加载器来进行扩展，以满足自己的特殊需求，就比如说，我们可以对Java类的字节码（`.class`文件）进行加密，加载时再利用自定义的类加载器对其解密。





