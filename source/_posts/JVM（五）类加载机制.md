---
title: JVM（五）类加载机制
date: 2018-05-10 23:47:24
tags: [JVM]
---
#### Java类加载机制

##### 概述
Class文件是JVM标准文件，其中描述了各种类的信息，最终都需要加载到虚拟机中才能运行和使用。类的加载就是将类的Class文件中二进制数据读入到内存中，将其放到方法区中，然后在堆区创建一个java.lang.class对象，用来封装方法区的数据结构。类加载的最终形式是方法区的信息和堆区的Class对象，Class对象提供访问方法区内数据结构的接口。

加载class文件的方式:
+ 从本地文件加载
+ 通过网络下载 class文件
+ 从zip、jar等归档文件中加载class文件
+ 从数据库中获取二进制数据
+ Java源码编译成class文件

##### 类生命周期
![](classlife.jpg)

###### - 加载

加载阶段的工作：
	- 通过一个类的全限定名获取定义类的二进制字节流
	- 通过这个字节流代表的静态存储结构转换成方法区的运行时数据结构
	- 堆内存中生成代表这个类的java.lang.Class对象，作为方法区数据的访问入口


因为虚拟机规范没有规定二进制字节流的来源，所以可以自定义获取，从而发展出很多：
	- 从zip包获取，发展Jar、war等格式
	- 从网络获取，发展出Applet
	- 运行时计算生成，发展出动态代理
	- 数据库读取，发展出中间件


###### - 验证

主要为了验证Class文件的正确性

###### - 准备

准备阶段是正式为类变量分配内存并设置类变量初始值的阶段，这些内存都在方法区中分配。
	+ 1、这时候进行内存分配的仅仅是类变量static，不包括实例变量，实例变量在对象实例化时随对象分配到堆中
	+ 2、初始值是指数据类型默认零值（0、0L、null、false），而不是Java显示赋的值

假如类变量定义：public static int value=3
在准备阶段后初始值是0，不是3，因为这时候并不执行任何Java代码，···public static···在编译后，放到类构造器*** <clinit>()***方法中，所以value=3到初始化才会执行。

	+ 3、如果是 static final 同时修饰的，会在准备阶段直接设置值


###### - 解析
将常量池符号引用替换成直接引用的过程
符号引用就是一组符号来描述目标，可以是任何字面量
直接引用就是直接指向目标的指针、或者一个间接定位到目标的句柄



###### - 初始化
到了初始化，才真正执行类中定义的Java程序代码
初始化时，为类的静态变量赋予正确的初始值，JVM负责将类进行初始化，主要对类变量进行初始化，是根据程序代码去初始化类变量和其他资源，是执行类构造器```<clinit>（）```方法的过程。
类构造器：

+ 类构造器```<clinit>```方法是由编译器自动收集所有类变量的赋值动作和静态语句块```static{}```中的语句合并产生的，收集的顺序由源文件顺序决定。
+ ```<clinit>()```方法和类构造函数（实例构造器```<init>()```）不一样，它不需要显示调用父类构造器，虚拟机会保证子类的```<clinit>()```执行前，父类的```<clinit>```执行完毕。所以会有父类的静态语句先执行现象出现。

类初始化时机：当类第一次被使用时，才会触发类初始化，如果类不存在，就会抛出ClassNotFound异常
什么时机定义为第一次主动使用呢？
+ 创建类实例 new 关键字，getstatic/putstatic 访问类或接口的静态变量或对其赋值，invokestatic调用类的静态方法
+ 反射调用，如果Class.forName("")
+ 初始化子类，首先会初始化父类
+ JVM启动指定的类，包括```main()```的那个类


##### 类加载器
在类加载阶段，需要应用程序自己决定如何获取所需要的类，实现这个动作的代码模块称为“类加载器”。它的主要作用就是将类加载到JVM。
同时对于任意一个类，都需要由它的类加载器和这个类本身一同确立在JVM中的唯一性；也就是说确定两个类是否相等，只有在同一类加载器的前提下才能确定，即使来自同一份Java文件，但不同的ClassLoader加载，也是不相等的。
###### 双亲委派模型

![](http://)

	父加载器并不是通过继承关系来实现的，而是通过组合实现的

从Java虚拟机角度看，只存在两种不同的类加载器：一种是启动类加载器（Bootstrap ClassLoader），这个类加载器由C++实现，是虚拟机自身的一部分，一种是所有其他的类加载器，由Java语句实现独立于虚拟机外部，都继承至java.lang.ClassLoader，这个类加载器也是Java对象，需要由启动类加载器加载到内存之中，才能使用它去加载其他的类。

大部分java程序都会使用以下系统提供的类加载器：

1. 启动类加载器（BootstrapClassLoader）：负责加载```<JAVA_HOME>\Jdk\jre\lib```目录下的类，并且是被虚拟机识别的，名字不符合的放到下面也不会被加载，例如rt.jar下所有```java.```开头的类。启动类加载器不能被Java程序直接引用
2. 扩展类加载器（Extension ClassLoader）：负责加载```<JAVA_HOME>\jdk\jre\ext```目录下的类，开发者可以直接使用这个类加载器
3. 应用程序类加载器（Application ClassLoader）：负责加载用户类路径（ClassPath）所指定的类，如果应用程序没有自定义自己的类加载器，就会使用这个作为默认加载器

**JVM类加载机制**
+ 全盘负责：当一个类加载器负责加载某个Class时，该Class所依赖的和引用的其他Class也由这个加载器负责载入，除非显示使用另一个类加载器
+ 父类委托：当一个类加载器需要加载某个类的时候，首先会把这个请求给自己的父加载器去完成，每个层次的加载器都是这样，因此所有加载请求都会最终到达顶层的启动类加载器，只有当父类加载器反馈自己无法完成这个加载请求，（即在搜索范围内没有找到需要的类），子加载器才会自己去加载
+ 缓存机制：缓存机制会使所有加载过的Class都会被缓存，当程序需要使用Class时，先到缓存区寻找，只有缓存区不存在，才会读取该类对应的二进制数据，并将其转换成Class对象。这就是为什么Java文件修改后，必须重启JVM，才会生效。

##### 类的加载
- 命令行启动应用时JVM初始化加载
- 通过Class.forName()方法动态加载
> 将类Class文件加载器JVM区，还会对类进行解释，执行static块，会进行初始化
> Class.forName(name,initialize,loader)带参数可控制是否加载static
- 通过CLassLoader.loadClass()动态加载
> 只是加载class到JVM中，不会执行static块，即不进行初始化

ClassLoader源码
```language

	public Class<?> loadClass(String name) throws ClassNotFoundException {
        return loadClass(name, false);
    }
     protected Class<?> loadClass(String name, boolean resolve)
        throws ClassNotFoundException
    {
            // First, check if the class has already been loaded
            // 首先判断该类是否已经被加载
            Class c = findLoadedClass(name);
            if (c == null) {
            //如果没被加载
                try {
                    if (parent != null) {
                    	//如果存在父类加载器，就委托给父类加载器
                        c = parent.loadClass(name, false);
                    } else {
                    	//如果父类加载器不存在，就检查给启动类加载器
                        c = findBootstrapClassOrNull(name);
                    }
                } catch (ClassNotFoundException e) {
                    // ClassNotFoundException thrown if class not found
                    // from the non-null parent class loader
                }

                if (c == null) {
                    // If still not found, then invoke findClass in order
                    // to find the class.
                    //如果都不能找到，才调用自身的加载器
                    // 通过findClass方法，所以我们自定义自己的类加载器只要重写这个方法即可，防止破坏了双亲委派模型
                    c = findClass(name);
                }
            }
            if (resolve) {
                resolveClass(c);
            }
            return c;
        }
    }
```
##### 自定义类加载器
```
class myclassLoader extends ClassLoader{
	private String root;
    //最好重写findClass方法，不要重写loadClass，防止破坏双亲委派模型
    protected Class<?> findClass(String name){
        byte[] data=loadClassData(name);
        if data == null{
            throw new ClassNotFoundException();
        }else{
            return defineClass(name,data,0,data.length);
        }
    }
    private byte[] loadClassData(String className){
    	//实现自己的获取class文件，读取字节流，并且操作
    }
}
```