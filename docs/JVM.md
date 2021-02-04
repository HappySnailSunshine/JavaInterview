# JVM

## 目录

[TOC]

> 主要参考自B站尚硅谷JVM课程
>
> 还有《深入理解JVM虚拟机》

# 内存与垃圾回收

- JVM和Java体系结构
- 类加载子系统
- 运行时数据区概述及线程
- 程序计数器
- 虚拟机栈
- 本地方法接口
- 本地方法栈
- 堆
- 方法区
- 直接内存
- 执行引擎
- String Table
- 垃圾回收概述
- 垃圾回收相关算法
- 垃圾回收相关概念
- 垃圾回收器



## JVM和Java体系结构

### JVM文档

https://docs.oracle.com/javase/specs/



### 各种语言的排名

世界上没有最好的语言，只有适合某种场景的最好的语言。

https://www.tiobe.com/tiobe-index/



### JVM跨语言的平台

JVM只关心，字节码文件是不是符合他的要求。

Java不是最强大的语言，但是JVM是最强大的虚拟机。

![image-20201116231542300](../media/pictures/JVM.assets/image-20201116231542300.png)

### JVM作用

JVM就是二进制字节码的运行环境。



### 特点

- 一次编译，到处运行
- 自动内存管理
- 自动垃圾回收



### JVM架构模型  （不会 计算机组成原理 汇编知识）

参考PPT 39

![image-20201119110155131](../media/pictures/JVM.assets/image-20201119110155131.png)



**重点：**

- 栈式架构可移植性好，跨平台，指令集小，指令多，执行性能比寄存器差；
- 寄存器架构完全依赖于硬件，可移植性差。



举例子，然后反编译 反编译命令 `javap  -v StackStruTest.class`

```java
public class StackStruTest {
    public StackStruTest() {
    }

    public static void main(String[] args) {
        int i = 2;
        int j = 3;
        int var10000 = i + j;

        try {
            Thread.sleep(6000L);
        } catch (InterruptedException var5) {
            var5.printStackTrace();
        }

        System.out.println("hello");
    }
}
```



![image-20201119105037144](../media/pictures/JVM.assets/image-20201119105037144.png)



### JVM生命周期

开始 执行 结束

`jps`命令，可以查看当前执行的进程

![image-20201119111519054](../media/pictures/JVM.assets/image-20201119111519054.png)



### JVM发展历程

Java是半解释，半编译形语言。

JIT执行编译器，参考：https://developer.ibm.com/zh/articles/j-lo-just-in-time/

![image-20201119112649613](../media/pictures/JVM.assets/image-20201119112649613.png)

热点代码（经常执行的代码），编译器会编译成字节码文件，缓存起来。

#### Sun Classic VM

最开始的时候，解释器和编译器不能同时运行。



**举例子：**

假如从A到B地点，解释器就是一直步行，得知要去B点以后，马上出发。

编译器就是坐公交，编译器编译的时候，类似等公交，可能要等的时间很长。



总结反思：

其实最好的是解释器，编译器夹杂着用，有些地方走路，有些地方很远坐公交。这也是后续编译器发展的方向。



#### Exact VM

- 热点探测

- 解释器编译器混合工作



#### Sun HotSpot VM 商用 目前属于Oracle

名称上指的是热点探测技术。

- 通过计数器找到最具有编译价值的代码，触发即时编译或者栈上替换
- 通过编译器结解释器协同工作，在最优化的程序响应时间与最佳执行性能中找到平衡。



#### BEA JRockit 商用 目前属于Oracle

专注于服务器应用。

- 所以不太注重启动的速度，所以没有解释器，只有编译器。
- 目前直接上最快的编译器。
- 后来Oracle将JRockit上面的优秀的东西整合到了HotSpot上面，在JDK8完成，在公司中JRockit团队还是占据主导，所以Java创始人，后来去了谷歌。



#### IBM J9 商用 

这三个是目前世界范围内最具有影响力的三大商用虚拟机。

- 这个虚拟机只在自己服务器上面运行速度好，稳定。类似于苹果。



#### KVM 和 CDC/CLDC HotSpot

主要用于Java ME



#### Azul VM

高性能虚拟机中的战斗机。



#### Liquid VM

高性能虚拟机中的战斗机。

- 不需要操作系统就可以用，本身就可以实现专用操作系统的功能。



#### Apache Harmony

Intel和IBM联合开发的开源JVM。但是Sun坚决不让Harmony获得JCP（目前管理Java的组织）的认证。

在Java世界中没用到，结果在Android SDK中用的挺好。



#### MicroSoft JVM

微软未来在浏览器中只是Java Applets ，开发了这个。只能在wins系统用。



#### TaobaoJVM

基于OpenJDK深度定制的

- 将生命周期较长的对象移出堆外，降低GC频率。
- GCIH对象能在虚拟机中共享



#### Dalvik VM

不是遵循Java JVM规范，不能直接执行Java class文件。

主要用于Android。



#### Graal VM 未来虚拟机

2018 Oracle新发布的虚拟机。

Run Programs Faster Anywhere.

跨语言全栈虚拟机，可以作为任何语言的运行平台使用。



## 类加载子系统

![image-20201119141420468](../media/pictures/JVM.assets/image-20201119141420468.png)



- ClassLoader 只负责class文件的加载，至于它是否可以运行，则要Execution Engine决定。（就好像有人给你介绍对象，人我到来了，具体成不成就看你自己啦。你自己就是Engine）



### 类的加载过程

宏观上的加载

![image-20201119143131811](../media/pictures/JVM.assets/image-20201119143131811.png)



微观上的加载，分三个环节

#### 加载 Loading

![image-20201119143645366](../media/pictures/JVM.assets/image-20201119143645366.png)



#### 链接 Linking

##### 验证

保证被加载的正确定，不能危害自己安全。



##### 准备

给变量赋值，开始都是初始值，0值。



##### 解析

将常量池中符号引用转换为直接引用。



#### 初始化 Init

![image-20201119162159712](../media/pictures/JVM.assets/image-20201119162159712.png)



### 类加载器分类

<img src="../media/pictures/JVM.assets/image-20201119162420925.png" alt="image-20201119162420925"  />



注意：

这几个不是继承关系，优先级或者说等级不一样。



实例代码：

```java
public class ClassLoaderTest {
    public static void main(String[] args) {

        //获取系统类加载器
        ClassLoader systemClassLoader = ClassLoader.getSystemClassLoader();
        System.out.println(systemClassLoader);//sun.misc.Launcher$AppClassLoader@18b4aac2

        //获取其上层：扩展类加载器
        ClassLoader extClassLoader = systemClassLoader.getParent();
        System.out.println(extClassLoader);//sun.misc.Launcher$ExtClassLoader@1540e19d

        //获取其上层：获取不到引导类加载器
        ClassLoader bootstrapClassLoader = extClassLoader.getParent();
        System.out.println(bootstrapClassLoader);//null

        //对于用户自定义类来说：默认使用系统类加载器进行加载
        ClassLoader classLoader = ClassLoaderTest.class.getClassLoader();
        System.out.println(classLoader);//sun.misc.Launcher$AppClassLoader@18b4aac2

        //String类使用引导类加载器进行加载的。---> Java的核心类库都是使用引导类加载器进行加载的。
        ClassLoader classLoader1 = String.class.getClassLoader();
        System.out.println(classLoader1);//null
    }
}
```



从上到下结果：

```
sun.misc.Launcher$AppClassLoader@18b4aac2
sun.misc.Launcher$ExtClassLoader@1540e19d
null
sun.misc.Launcher$AppClassLoader@18b4aac2
null
```



总结分析：

- Bootstrap  ClassLoader 相当于女王御用厨师，一般人让做个菜肯定不行啊；
- Java核心类库都是引导类加载的；
- 引导类加载器为什么看不到？是因为这个是C、C++ 写的，所以看不到。



### 虚拟机自带的加载器

![image-20201119164809103](../media/pictures/JVM.assets/image-20201119164809103.png)



![image-20201120093054986](../media/pictures/JVM.assets/image-20201120093054986.png)



![image-20201120093111598](../media/pictures/JVM.assets/image-20201120093111598.png)



### 自定义类加载

![image-20201120093128264](../media/pictures/JVM.assets/image-20201120093128264.png)

```java
/**
 * 自定义用户类加载器
 */
public class CustomClassLoader extends ClassLoader {
    @Override
    protected Class<?> findClass(String name) throws ClassNotFoundException {

        try {
            byte[] result = getClassFromCustomPath(name);
            if(result == null){
                throw new FileNotFoundException();
            }else{
                return defineClass(name,result,0,result.length);
            }
        } catch (FileNotFoundException e) {
            e.printStackTrace();
        }

        throw new ClassNotFoundException(name);
    }

    private byte[] getClassFromCustomPath(String name){
        //从自定义路径中加载指定类:细节略
        //如果指定路径的字节码文件进行了加密，则需要在此方法中进行解密操作。
        return null;
    }

    public static void main(String[] args) {
        CustomClassLoader customClassLoader = new CustomClassLoader();
        try {
            Class<?> clazz = Class.forName("One",true,customClassLoader);
            Object obj = clazz.newInstance();
            System.out.println(obj.getClass().getClassLoader());
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```



### ClassLoader

获取ClassLoader几种方式：

![image-20201120095046610](../media/pictures/JVM.assets/image-20201120095046610.png)



### 双亲委派机制 面试要问

![image-20201120100225986](../media/pictures/JVM.assets/image-20201120100225986.png)

举例子：

```java
//本地java.lang包下面写了个String
public class String {
    //
    static{
        System.out.println("我是自定义的String类的静态代码块");
    }
    //错误: 在类 java.lang.String 中找不到 main 方法
    public static void main(String[] args) {
        System.out.println("hello,String");
    }
}
```



```java
//其他包 引用String  发现并没有引用上面的String
public class StringTest {

    public static void main(String[] args) {
        java.lang.String str = new java.lang.String();
        System.out.println("hello,atguigu.com");

        StringTest test = new StringTest();
        System.out.println(test.getClass().getClassLoader());
    }
}
```



原因是：

系统类加载器加载String类的时候，会向上委派给他的上一级扩展类加载器，扩展类加载器会委派给上一级引导类加载器，引导类加载器加载的时候一看路径是Java下的lang，自己就加载系统核心类库里面的String，所以没有加载本地的String。



**举例子：** （妙不可言）

现在小孩子（系统加载器）有一个苹果，他吃之前问他妈（扩展类加载器），妈你吃吗？他妈看姥姥（引导类加载器）在身边，就问妈你吃吗？

如果姥姥吃了，就相当于直接吃啦。

如果姥姥觉得这个苹果太硬啦，说不吃啦，转手给妈妈，妈妈也觉得太酸啦，不吃。最后回到小孩子手里，只能自己吃啦。（自己加载）



#### 优势

![image-20201120101737457](../media/pictures/JVM.assets/image-20201120101737457.png)



### 沙箱安全机制

类加载器的这种委派模式就是一种沙箱安全机制，会对核心API有保护作用。

在一个隔离的环境中操作，不会污染其他东西，例如360沙箱，支付宝沙箱。



### 其他重要知识

##### 两个class对象是都为同一个类

![image-20201120102653041](../media/pictures/JVM.assets/image-20201120102653041.png)



##### 类的主动使用和被动使用

![image-20201120103044703](../media/pictures/JVM.assets/image-20201120103044703.png)



## 运行时数据区

### 概述

![image-20201120105050051](../media/pictures/JVM.assets/image-20201120105050051.png)



其中红色的是一个进程公用的，灰色的是一个线程公用的。

![image-20201120105212950](../media/pictures/JVM.assets/image-20201120105212950.png)



### 线程

![image-20201120110351418](../media/pictures/JVM.assets/image-20201120110351418.png)



#### JVM系统线程

![image-20201120110526452](../media/pictures/JVM.assets/image-20201120110526452.png)



## 程序计数器 （PC寄存器）

### PC 寄存器介绍

![image-20201120111026034](../media/pictures/JVM.assets/image-20201120111026034.png)



![image-20201120111334108](../media/pictures/JVM.assets/image-20201120111334108.png)

这个地方不会发生OOM，也没有GC。

#### 举例子

![image-20201120132451871](../media/pictures/JVM.assets/image-20201120132451871.png)



![image-20201120132901933](../media/pictures/JVM.assets/image-20201120132901933.png)





### 两个常见问题

![image-20201120140521559](../media/pictures/JVM.assets/image-20201120140521559.png)

**这里用到一个知识，并发还是并行？** 知乎大佬的回答：哈哈

你吃饭吃到一半，电话来了，你一直到吃完了以后才去接，这就说明你不支持并发也不支持并行。
你吃饭吃到一半，电话来了，你停了下来接了电话，接完后继续吃饭，这说明你支持并发。
你吃饭吃到一半，电话来了，你一边打电话一边吃饭，这说明你支持并行。

并发的关键是你有处理多个任务的能力，不一定要同时。
并行的关键是你有同时处理多个任务的能力。

所以我认为它们最关键的点就是：是否是『同时』



![image-20201120141003242](../media/pictures/JVM.assets/image-20201120141003242.png)



### CPU时间片

![image-20201120141452769](../media/pictures/JVM.assets/image-20201120141452769.png)





## 虚拟机栈

### 概述背景概述

![image-20201120142459185](../media/pictures/JVM.assets/image-20201120142459185.png)



#### 内存中的堆和栈

![image-20201120142432235](../media/pictures/JVM.assets/image-20201120142432235.png)



#### 虚拟机栈的基本内容

![image-20201120142550908](../media/pictures/JVM.assets/image-20201120142550908.png)



![image-20201120143604554](../media/pictures/JVM.assets/image-20201120143604554.png)



当然虚拟机栈中还要细分很多东西。



![image-20201120144133967](../media/pictures/JVM.assets/image-20201120144133967.png)



**其实也没有OOM，如果栈满了，溢出。当然也没必要GC。**



![image-20201120144748277](../media/pictures/JVM.assets/image-20201120144748277.png)



#### 设置栈内存大小

-Xss 设置线程的最大栈空间，栈的大小直接决定了函数调用的最大可达深度。



IDEA设置虚拟机栈大小：

<img src="../media/pictures/JVM.assets/image-20201120152833067.png" alt="image-20201120152833067" style="zoom:50%;" />



![image-20201120152929623](../media/pictures/JVM.assets/image-20201120152929623.png)



### 栈的存储单位

<img src="../media/pictures/JVM.assets/image-20201120153247163.png" alt="image-20201120153247163" style="zoom:40%;" />



#### 栈运行原理

![image-20201120153749427](../media/pictures/JVM.assets/image-20201120153749427.png)





Debug的时候，可以看到栈

<img src="../media/pictures/JVM.assets/image-20201120155503215.png" alt="image-20201120155503215" style="zoom: 33%;" />



![image-20201120155627502](../media/pictures/JVM.assets/image-20201120155627502.png)



![image-20201120155932732](../media/pictures/JVM.assets/image-20201120155932732.png)



不管代码中写没写return，反编译中，都有return。



#### 栈帧内部结构

<img src="../media/pictures/JVM.assets/image-20201120161010077.png" alt="image-20201120161010077" style="zoom:50%;" />



### 栈的局部变量表

<img src="../media/pictures/JVM.assets/image-20201120161940943.png" alt="image-20201120161940943" style="zoom: 33%;" />



<img src="../media/pictures/JVM.assets/image-20201120162124503.png" alt="image-20201120162124503" style="zoom:33%;" />



```bash
cd out/production/chapter05/com/atguigu/java1/

#看字节码里面细节
javap -v LocalVariablesTest.class
```

这个里面有堆栈，参数信息等：

<img src="../media/pictures/JVM.assets/image-20201126090909485.png" alt="image-20201126090909485" style="zoom: 50%;" />



#### 用JClassLib看具体一个类的信息

LocalVariablesTest.class

<img src="../media/pictures/JVM.assets/image-20201126092238140.png" alt="image-20201126092238140" style="zoom:50%;" />

<img src="../media/pictures/JVM.assets/image-20201126092646904.png" alt="image-20201126092646904" style="zoom:50%;" />

<img src="../media/pictures/JVM.assets/image-20201126091851710.png" alt="image-20201126091851710" style="zoom:50%;" />

<img src="../media/pictures/JVM.assets/image-20201126092111443.png" alt="image-20201126092111443" style="zoom:50%;" />

分析：

- test变量，其实PC对应行号是14，这是他作用域开始的行；
- 起始PC + 长度 都是16，这个16其实就是第二张图，字节码长度16
- 局部变量最大槽数3，就是说这个方法中有三个变量，分别是args，test，num



#### 关于Slot(槽)的理解

![image-20201126100707224](../media/pictures/JVM.assets/image-20201126100707224.png)

如何理解上面最后那一句话呢？下面代码示例展示。

##### 静态方法和构造方法

代码示例()：

<img src="../media/pictures/JVM.assets/image-20201126100837311.png" alt="image-20201126100837311" style="zoom: 67%;" />

```java
public static void testStatic() {
		LocalVariablesTest test = new LocalVariablesTest();
		Date date = new Date();
		int count = 10;
		System.out.println(count);
		//因为this变量不存在于当前方法的局部变量表中！！
		//System.out.println(this.count);
	}

	//关于Slot的使用的理解
	public LocalVariablesTest() {
		this.count = 1;
	}
```

在testStatic方法中是用不了this关键字的，但是在构造方法中是可以用的。

解析：

构造方法就是初始化方法，局部变量表有他，所以可以用

<img src="../media/pictures/JVM.assets/image-20201126101157921.png" alt="image-20201126101157921" style="zoom:67%;" />

再看testStatic方法，局部变量表中没有this，肯定用不了它喽!

<img src="../media/pictures/JVM.assets/image-20201126101247753.png" alt="image-20201126101247753" style="zoom:67%;" />

##### 非静态方法

this变量就在局部变量表的第一个，代码示例：

```java
public void test1() {
    Date date = new Date();
    String name1 = "atguigu.com";
    test2(date, name1);
    System.out.println(date + name1);
}
```

<img src="../media/pictures/JVM.assets/image-20201126101903918.png" alt="image-20201126101903918" style="zoom:67%;" />



##### 各种类型，含double类型，占两个槽

```java
public String test2(Date dateP, String name2) {
    dateP = null;
    name2 = "songhongkang";
    double weight = 130.5;//占据两个slot
    char gender = '男';
    return dateP + name2;
}
```

这里面其实有五个变量。但是有6个槽

<img src="../media/pictures/JVM.assets/image-20201126102744386.png" alt="image-20201126102744386" style="zoom:67%;" />

体重这个变量占了两个槽，序号是3（第四个位置），序号4（第五个位置）也被他占用啦。

<img src="../media/pictures/JVM.assets/image-20201126102833722.png" alt="image-20201126102833722" style="zoom:67%;" />



#### Slot的重复利用

```java
public void test4() {
    int a = 0;
    {
        int b = 0;
        b = a + 1;
    }
    //变量c使用之前已经销毁的变量b占据的slot的位置
    int c = a + 1;
}
```

<img src="../media/pictures/JVM.assets/image-20201126104819057.png" alt="image-20201126104819057" style="zoom:67%;" />

在这个代码中，变量b为什么和变量c序号一样（就是占用槽其实位置一样）？？？

因为变量b，有效范围就是大括号之内，大括号完结，变量b原来的地方会被c重新利用。（妙啊！）



#### 静态变量和局部变量的对比

<img src="../media/pictures/JVM.assets/image-20201126105753011.png" alt="image-20201126105753011" style="zoom: 33%;" />

补充：

<img src="../media/pictures/JVM.assets/image-20201126110301456.png" alt="image-20201126110301456" style="zoom: 33%;" />

### 操作数栈

Operand Stack

栈：实现可以用数组或者链表来实现。

<img src="../media/pictures/JVM.assets/image-20201126111248667.png" alt="image-20201126111248667" style="zoom: 33%;" />

<img src="../media/pictures/JVM.assets/image-20201126111916720.png" alt="image-20201126111916720" style="zoom: 33%;" />

<img src="../media/pictures/JVM.assets/image-20201126112309105.png" alt="image-20201126112309105" style="zoom:33%;" />

### 代码追踪

具体看PPT，求和运算。



### 栈顶缓存技术

![image-20201126133628192](../media/pictures/JVM.assets/image-20201126133628192.png)



### 动态链接

![image-20201126134628582](../media/pictures/JVM.assets/image-20201126134628582.png)

动态链接是栈帧里面的一个部分。

动态链接指向运行时常量池的方法引用。

![image-20201126150223899](../media/pictures/JVM.assets/image-20201126150223899.png)

![image-20201126135014009](../media/pictures/JVM.assets/image-20201126135014009.png)

示例代码

```java
public class DynamicLinkingTest {
    int num = 10;

    public DynamicLinkingTest() {
    }

    public void methodA() {
        System.out.println("methodA()....");
    }

    public void methodB() {
        System.out.println("methodB()....");
        this.methodA();
        ++this.num;
    }
}
```



#### 字节码文件中的常量池

<img src="../media/pictures/JVM.assets/image-20201126140049958.png" alt="image-20201126140049958" style="zoom: 67%;" />

字节码中的方法

<img src="../media/pictures/JVM.assets/image-20201126140238771.png" alt="image-20201126140238771" style="zoom:67%;" />

这里面的#7，和#2就是指的上面常量池里面的值。



**注意：**

在加载的时候，会把这个类需要用到的东西，都加载进来，以一个符号的形式表示。

例如：#13 表示返回值为空的，下面还有输出流等。



**为什么会用这个动态链接呢？为什么不把内容直接拿过来用呢？**

Java里面的好多其实都是引用，之所以用引用一来节省空间，二来好多内容可以共享。而且好多东西很大，不能都保存在字节码文件中，所以用引用的方式。



### 方法的调用

<img src="../media/pictures/JVM.assets/image-20201126150334306.png" alt="image-20201126150334306" style="zoom: 33%;" />



<img src="../media/pictures/JVM.assets/image-20201126151820734.png" alt="image-20201126151820734" style="zoom: 33%;" />

面向过程的语言，大多数在编译器就可以确定，所以是早期绑定，面向对象语言，早期晚期绑定都有。

<img src="../media/pictures/JVM.assets/image-20201126152941124.png" alt="image-20201126152941124" style="zoom:33%;" />



#### 虚方法和非虚方法

<img src="../media/pictures/JVM.assets/image-20201207090534235.png" alt="image-20201207090534235" style="zoom: 33%;" />

子类对象多态的使用前提：

- 类的继承
- 方法的重写

<img src="../media/pictures/JVM.assets/image-20201207091021163.png" alt="image-20201207091021163" style="zoom:33%;" />



#### invokedynamic指令

<img src="../media/pictures/JVM.assets/image-20201207171617856.png" alt="image-20201207171617856" style="zoom:33%;" />

##### 动态类型语言和静态类型语言

<img src="../media/pictures/JVM.assets/image-20201207171940805.png" alt="image-20201207171940805" style="zoom:33%;" />



Java其实是动静态类型语言（对于类型的检查在编译器）。Js，Python 动态类型语言（对类型的检查在运行期）。

<img src="../media/pictures/JVM.assets/image-20201207172219469.png" alt="image-20201207172219469" style="zoom: 50%;" />



**这个的增加就是为了是某些动态语言，可以运行到JVM虚拟机上面。**



#### 方法重写的本质

<img src="../media/pictures/JVM.assets/image-20201207174642203.png" alt="image-20201207174642203" style="zoom:33%;" />



方法重写本质依次往上找，不太靠谱，所以有下面的需方法表。

#### 虚方法表 

<img src="../media/pictures/JVM.assets/image-20201208090140932.png" alt="image-20201208090140932" style="zoom:33%;" />



<img src="../media/pictures/JVM.assets/image-20201208090205213.png" alt="image-20201208090205213" style="zoom:33%;" />

说明：下面白色的方法是Son类重写父类Father的方法，蓝色的方法是Object类的方法。如果子类Son调用toString方法，那么虚方法表回指向Object类的toString方法，直接调用Object类中的方法，不用再一层一层往上面找。



虚方法表举例：

```java
/**
 * 虚方法表的举例
 *
 * @author shkstart
 * @create 2020 下午 1:11
 */
interface Friendly {
    void sayHello();
    void sayGoodbye();
}
class Dog {
    public void sayHello() {
    }
    public String toString() {
        return "Dog";
    }
}
class Cat implements Friendly {
    public void eat() {
    }
    public void sayHello() {
    }
    public void sayGoodbye() {
    }
    protected void finalize() {
    }
    public String toString(){
        return "Cat";
    }
}

class CockerSpaniel extends Dog implements Friendly {
    public void sayHello() {
        super.sayHello();
    }
    public void sayGoodbye() {
    }
}

public class VirtualMethodTable {
}
```

**Dog类**

<img src="../media/pictures/JVM.assets/image-20201208091312096.png" alt="image-20201208091312096" style="zoom:33%;" />

**CockerSpaniel（可卡犬）类**

<img src="../media/pictures/JVM.assets/image-20201208091419546.png" alt="image-20201208091419546" style="zoom:33%;" />

**Cat 类**

<img src="../media/pictures/JVM.assets/image-20201208091543188.png" alt="image-20201208091543188" style="zoom:33%;" />



### 方法返回值

**返回指令包含ireturn（当返回值是boolean、byte、char、short和int类型时使用）、lreturn、freturn、dreturn以及areturn，另外还有一个return指令供声明为void的方法、实例初始化方法、类和接口的初始化方法使用。**

<img src="../media/pictures/JVM.assets/image-20201208094636332.png" alt="image-20201208094636332" style="zoom:33%;" />



示例

<img src="../media/pictures/JVM.assets/image-20201214204631235.png" alt="image-20201214204631235" style="zoom: 67%;" />

```
  public void method2() {

        methodVoid();

        try {
            method1();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
```

起始 跳转 

<img src="../media/pictures/JVM.assets/image-20201214204703445.png" alt="image-20201214204703445" style="zoom:50%;" />

上面的4，8，11分别对应 72，75，73 行代码。就是catch语句的try包含的方法，结束}，catch所对应的行号。

<img src="../media/pictures/JVM.assets/image-20201214204736164.png" alt="image-20201214204736164" style="zoom:50%;" />



<img src="../media/pictures/JVM.assets/image-20201215084715436.png" alt="image-20201215084715436" style="zoom:50%;" />

### 一些附加信息

![image-20201215085226184](../media/pictures/JVM.assets/image-20201215085226184.png)



![image-20201215085240934](../media/pictures/JVM.assets/image-20201215085240934.png)

### 常见面试题









- 举例说明栈溢出情况？（StockOverFlowError）
  - 通过-Xss设置栈的大小；OOM

- 调整栈大小，就能保证不出现栈溢出的情况吗？ 不能
  - 假如原来栈深度设置为5000，一个递归需要递归6000次，当修改栈的大小以后，修改为7000，调用原来的6000次递归可能不会出现异常，但是如果设置为5500，调用原来的6000次递归还是会出现异常。
  - 生活中通俗的例子，假如你原来每个月500零花钱，你不够花，现在给你5000就一定够花了吗？当然不一定。
- 分配的栈内存越大越好吗？
  - 当然不是？这个问题就好像问每个月给你钱越多越好吗？那肯定不是，多了有很多坏处。
  - 就拿计算机资源来说，总资源是有限的，你这个栈内存分配太大，其他资源就可用的就比较少啦。
- 垃圾回收是否会涉及到虚拟机栈？不会？
  - 为什么？
- 方法中定义的局部变量是否线程安全？ 具体问题具体分析。



详细参见JVMDemo中chapter05.java3.StringBuilderTest

```java
package com.atguigu.java3;

/**
 * 面试题：
 * 方法中定义的局部变量是否线程安全？具体情况具体分析
 * <p>
 * 何为线程安全？
 * 如果只有一个线程才可以操作此数据，则必是线程安全的。
 * 如果有多个线程操作此数据，则此数据是共享数据。如果不考虑同步机制的话，会存在线程安全问题。
 *
 * @author shkstart
 * @create 2020 下午 7:48
 */
public class StringBuilderTest {

	int num = 10;

	//s1的声明方式是线程安全的  个人理解：s1是这个方法内部所有，数据不是共享的
	public static void method1() {
		//StringBuilder:线程不安全        //StringBuffer //线程安全的，源码里面已经有Synchronized 修饰啦
		StringBuilder s1 = new StringBuilder();
		s1.append("a");
		s1.append("b");
		//...
	}

	//sBuilder的操作过程：是线程不安全的
	public static void method2(StringBuilder sBuilder) {
		sBuilder.append("a");
		sBuilder.append("b");
		//...
	}

	//s1的操作：是线程不安全的  个人理解：有返回值，会被其他线程用到
	public static StringBuilder method3() {
		StringBuilder s1 = new StringBuilder();
		s1.append("a");
		s1.append("b");
		return s1;
	}

	// s1的操作：是线程安全的  个人理解：s1在方法中生，结束处死。点开s1的toString（）方法，里面new了一个新的String，
	// 所以返回的并不是原来的s1，而是新的String对象
	public static String method4() {
		StringBuilder s1 = new StringBuilder();
		s1.append("a");
		s1.append("b");
		return s1.toString();
	}

	public static void main(String[] args) {
		StringBuilder s = new StringBuilder();

		new Thread(() -> {
			s.append("a");
			s.append("b");
		}).start();

		method2(s);
	}
}
```



## 本地方法接口

![image-20201215095748855](../media/pictures/JVM.assets/image-20201215095748855.png)



现在学的是右下角这一部分。

### 什么是本地方法

<img src="../media/pictures/JVM.assets/image-20201215100001248.png" alt="image-20201215100001248" style="zoom: 33%;" />



在Thread方法中，搜索native，有很多本地方法，比如setPriority0() 设置线程优先级，其实是本地方法，这些方法并没有方法体，其实方法的具体实现是在其他地方，可能不是用Java代码写的。

<img src="../media/pictures/JVM.assets/image-20201215100800277.png" alt="image-20201215100800277" style="zoom: 67%;" />

Java中的线程都需要转换成操作系统的本地线程来操作。其实虽然是Java多线程，其实还是需要操作系统来支持。



举例：

```java
package com.atguigu.java;

public class IHaveNatives {
    public native void Native1(int x);
    public native static long Native2();
    private native synchronized float Native3(Object o);
    native void Native4(int[] ary) throws Exception;
}
```

注意：native关键字可以和其他的Java标识符连用，但是abstract除外。



### 为什么要使用Native Method？

<img src="../media/pictures/JVM.assets/image-20201215102809488.png" alt="image-20201215102809488" style="zoom: 33%;" />

<img src="../media/pictures/JVM.assets/image-20201215102839386.png" alt="image-20201215102839386" style="zoom:33%;" />

这三条解释：

- 新中国刚成立，不可避免的需要与其他国家进行接触，难免会用到其他国际规范，比如美国在有一些灵玉提供了一些接口，当时我们可以直接用。
- 与操作系统有关的，与性能有关的一些，还是需要用到。比如韩国，什么都不依靠，依靠自己（操作系统）。
- 比如韩国，好多方面就需要美国保护，军事上等等，美国对它的影响潜移默化，不可避免要调用美国接口。



### 现状

<img src="../media/pictures/JVM.assets/image-20201215104155812.png" alt="image-20201215104155812" style="zoom: 33%;" />



## 本地方法栈

<img src="../media/pictures/JVM.assets/image-20201215114833966.png" alt="image-20201215114833966" style="zoom: 67%;" />



<img src="../media/pictures/JVM.assets/image-20201215132222544.png" alt="image-20201215132222544" style="zoom:50%;" />



<img src="../media/pictures/JVM.assets/image-20201215132242970.png" alt="image-20201215132242970" style="zoom: 33%;" />



## 堆

- 堆的核心概述
- 设置堆的大小和OOM
- 年轻代和年老代
- 图解对象分配过程
- Minor GC、Major GC、Full GC
- 堆空间分代思想
- 内存分配策略
- 为对象分配内存：TLAB
- 小结堆空间的参数设置
- 堆是分配对象的唯一原则吗



堆：每个进程是唯一一个堆，好多个线程共享一个进程中的堆空间。

![image-20201215142606441](../media/pictures/JVM.assets/image-20201215142606441.png)

一个JVM实例，代表一个进程，堆和方法区是线程共享的，程序计数器，本地方法栈，虚拟机栈是每个线程独有的。



### 堆的核心概述

<img src="../media/pictures/JVM.assets/image-20201215164025121.png" alt="image-20201215164025121" style="zoom: 33%;" />

**注意**：物理上不连续，逻辑上是连续的。

<img src="../media/pictures/JVM.assets/image-20201215164633662.png" alt="image-20201215164633662" style="zoom: 33%;" />

**注意**：

- Java虚拟机规范中说是all，所有的对象实例都在堆中分配内存空间。实际情况是almost(几乎所有)，有一些是在栈中分配。为什么是这样？具体待证实。

- **方法结束时，栈中保存的引用会断开，堆中的对象不会被马上移除，仅仅在垃圾收集的时候才会被移除。**



对应代码中chapter08中HeapDemo HeapDemo1 

首先配置 每个文件的堆的大小参数：`-Xms10m -Xmx10m`

新版本的Idea配置藏起来啦，需要自己弄出来。

![image-20201215162636639](../media/pictures/JVM.assets/image-20201215162636639.png)

把这两个demo启动以后，然后再JDK安装目录下bin目录，启动监控工具

![image-20201215162832000](../media/pictures/JVM.assets/image-20201215162832000.png)



启动以后，左边找到刚才的Demo，点开，右边回出现详细信息。



注意：刚使用这个监控工具的时候，没有最后面三个插件，需要在工具-->>插件-->>可用插件中，来查看，点击下面安装。

![image-20201215163109795](../media/pictures/JVM.assets/image-20201215163109795.png)

如果这里没有这个列表，且报这个错，说明下载路径有问题。解决方案：重新选一个地址就好。

参考：https://blog.csdn.net/qq_42449963/article/details/110165097

<img src="../media/pictures/JVM.assets/image-20201215163335830.png" alt="image-20201215163335830" style="zoom: 50%;" />



监控软件官网：https://visualvm.github.io/pluginscenters.html 

每个版本，都有对应的下载路径。



#### 栈 堆 方法区关系

<img src="../media/pictures/JVM.assets/image-20201215170732942.png" alt="image-20201215170732942" style="zoom: 50%;" />

测试：数组new放在了堆里头

![image-20201215182813286](../media/pictures/JVM.assets/image-20201215182813286.png)



代码

```java
package com.atguigu.java;

/**
 * @author shkstart  shkstart@126.com
 * @create 2020  17:28
 */
public class SimpleHeap {
    private int id;//属性、成员变量

    public SimpleHeap(int id) {
        this.id = id;
    }

    public void show() {
        System.out.println("My ID is " + id);
    }
    public static void main(String[] args) {
        SimpleHeap sl = new SimpleHeap(1);
        SimpleHeap s2 = new SimpleHeap(2);
        int[] arr = new int[10];
        Object[] arr1 = new Object[10];
    }
}
```



**注意：**

- 垃圾回收主要考虑的是堆空间（方法区也有垃圾回收）。栈是没有垃圾回收的。

- GC会在大内存和频繁的GC中，遇到瓶颈。需要优化。



#### 内存细分

分代垃圾回收算法就是针对堆中分代垃圾来说的。

<img src="../media/pictures/JVM.assets/image-20201215183656953.png" alt="image-20201215183656953" style="zoom: 33%;" />



#### 堆空间内部结构 (JDK 7)

<img src="../media/pictures/JVM.assets/image-20201215184836289.png" alt="image-20201215184836289" style="zoom: 50%;" />

#### 堆空间内部结构（JDK 8）

上面的永久区，换成了元空间。（这里没有图片）



#### 打印GC详情日志

首先需要配置一个参数（其他参数参照Guide哥文章里面） 注意参数之前有+

```
-XX:+PrintGCDetails
```

![image-20201216151507069](../media/pictures/JVM.assets/image-20201216151507069.png)

配置了参数以后，运行代码结果（注意这里是JDK1.8  所以下面是元空间（Metaspace），如果是JDK1.7的话，就不是这样的，下面是方法区）：

```java
-Xms : 575M
-Xmx : 575M
Heap
 PSYoungGen      total 179200K, used 12288K [0x00000000f3800000, 0x0000000100000000, 0x0000000100000000)
  eden space 153600K, 8% used [0x00000000f3800000,0x00000000f44001b8,0x00000000fce00000)
  from space 25600K, 0% used [0x00000000fe700000,0x00000000fe700000,0x0000000100000000)
  to   space 25600K, 0% used [0x00000000fce00000,0x00000000fce00000,0x00000000fe700000)
 ParOldGen       total 409600K, used 0K [0x00000000da800000, 0x00000000f3800000, 0x00000000f3800000)
  object space 409600K, 0% used [0x00000000da800000,0x00000000da800000,0x00000000f3800000)
 Metaspace       used 3454K, capacity 4496K, committed 4864K, reserved 1056768K
  class space    used 382K, capacity 388K, committed 512K, reserved 1048576K
```



#### 堆空间内部结构对比

![image-20201215192659341](../media/pictures/JVM.assets/image-20201215192659341.png)



### 设置堆的大小和OOM

<img src="../media/pictures/JVM.assets/image-20201215192853873.png" alt="image-20201215192853873" style="zoom: 40%;" />

#### 设置堆大小 代码测试

查看电脑默认堆的最大最小内存分配

```java
package com.atguigu.java;

/**
 * 1. 设置堆空间大小的参数
 * -Xms 用来设置堆空间（年轻代+老年代）的初始内存大小
 *      -X 是jvm的运行参数
 *      ms 是memory start
 * -Xmx 用来设置堆空间（年轻代+老年代）的最大内存大小
 *
 * 2. 默认堆空间的大小
 *    初始内存大小：物理电脑内存大小 / 64
 *             最大内存大小：物理电脑内存大小 / 4
 * 3. 手动设置：-Xms600m -Xmx600m
 *     开发中建议将初始堆内存和最大的堆内存设置成相同的值。
 *
 * 4. 查看设置的参数：方式一： jps   /  jstat -gc 进程id
 *                  方式二：-XX:+PrintGCDetails
 * @author shkstart  shkstart@126.com
 * @create 2020  20:15
 */
public class HeapSpaceInitial {
    public static void main(String[] args) {

        //返回Java虚拟机中的堆内存总量
        long initialMemory = Runtime.getRuntime().totalMemory() / 1024 / 1024;
        //返回Java虚拟机试图使用的最大堆内存量
        long maxMemory = Runtime.getRuntime().maxMemory() / 1024 / 1024;

        System.out.println("-Xms : " + initialMemory + "M");
        System.out.println("-Xmx : " + maxMemory + "M");

        System.out.println("系统内存大小为：" + initialMemory * 64.0 / 1024 + "G");
        System.out.println("系统内存大小为：" + maxMemory * 4.0 / 1024 + "G");

        try {
            Thread.sleep(1000000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}

```

结果：

```
-Xms : 243M
-Xmx : 3605M
系统内存大小为：15.1875G
系统内存大小为：14.08203125G
```



**注意：**

- 虽然这里有最大堆内存最小堆内存之说，但是在生产环境上面，一般会将最大和最小堆内存设置成一样的值。这样避免GC以后，系统调整堆空间大小，占用系统资源。

- 查看设置的参数：

  - 方式一： jps   /  jstat -gc 进程id（jps 和 jstat 是jdk中的命令）

   *                  方式二：-XX:+PrintGCDetails



内存区域参考图：

<img src="../media/pictures/JVM.assets/image-20201216153244275.png" alt="image-20201216153244275" style="zoom: 33%;" />

同样是上面的代码，实际操作如下（现在JVM参数设置为：-Xms600m -Xmx600m ）：

现在输出结果是：

```
-Xms : 575M
-Xmx : 575M
```

![image-20201216152627313](../media/pictures/JVM.assets/image-20201216152627313.png)



**上面从左到依次是：（前八个）**

1. survivor0总共
2.  survivor1总共 
3. survivor0已使用（U代表use 已使用）
4. survivor1已使用
5. Eden总共的
6. Eden已使用
7. 老年代（O代表old）总共
8. 老年代已使用



**进行计算：**

上面总共的可使用为：（ 25600+25600+153600+409600 ）/ 1024 = 600

实际上输出的明明是575，为什么上面计算出来是600呢？

（ 25600+153600+409600 ）/ 1024 = 575



**总结：**

实际情况是survivor两个区域，同时只有一个区域在用。这里垃圾GC算法是复制算法，所以总有一个是空的。

用监控软件来看的话是很形象的。下面总有一个是空的。

![image-20201216161510607](../media/pictures/JVM.assets/image-20201216161510607.png)

#### OutOfMemoryError举例

代码如下（内存参数设置为：-Xms600m -Xmx600m）：

```java
package com.atguigu.java;

import java.util.ArrayList;
import java.util.Random;

/**
 * -Xms600m -Xmx600m
 * @author shkstart  shkstart@126.com
 * @create 2020  21:12
 */
public class OOMTest {
    public static void main(String[] args) {
        ArrayList<Picture> list = new ArrayList<>();
        while(true){
            try {
                Thread.sleep(20);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            list.add(new Picture(new Random().nextInt(1024 * 1024)));
        }
    }
}

class Picture{
    private byte[] pixels;

    public Picture(int length) {
        this.pixels = new byte[length];
    }
}
```



用监控软件可以动态的看到，首先Eden满了以后，然后Old满了，然后代码就报错啦。

```java
Exception in thread "main" java.lang.OutOfMemoryError: Java heap space
	at com.atguigu.java.Picture.<init>(OOMTest.java:29)
	at com.atguigu.java.OOMTest.main(OOMTest.java:20)
```



### 年轻代和老年代

<img src="../media/pictures/JVM.assets/image-20201216163058798.png" alt="image-20201216163058798" style="zoom:33%;" />

解释：

- 《圣经》中说，上帝创建了两个人，一个叫亚当，一个叫夏娃开始都把他们放在了Eden（伊甸园）区。
- Survivor两个区也叫from 、to区，就是复制算法一边到一边。（并不一定左边就是from，右边就是to）



<img src="../media/pictures/JVM.assets/image-20201216164256349.png" alt="image-20201216164256349" style="zoom: 50%;" />

解释：下面这个参数是个比率值，就是：ratio = 老年代 / 年轻代

```java
-XX:NewRatio=2  //相当于老年代 : 年轻代 = 2 : 1
```



#### 查看老年代和年轻代的比例

```bash
jps -l
jinfo -flag NewRatio 5156(进程号，是通过上面的命令查出来的)
```

<img src="../media/pictures/JVM.assets/image-20201216170943621.png" alt="image-20201216170943621" style="zoom: 50%;" />



年轻代中 Eden和 from to区 默认比例是 8:1:1   

```java
package com.atguigu.java1;

/**
 * -Xms600m -Xmx600m
 *
 * -XX:NewRatio ： 设置新生代与老年代的比例。默认值是2.
 * -XX:SurvivorRatio ：设置新生代中Eden区与Survivor区的比例。默认值是8
 * -XX:-UseAdaptiveSizePolicy ：关闭自适应的内存分配策略  （暂时用不到）
 * -Xmn:设置新生代的空间的大小。 （一般不设置）
 *
 * @author shkstart  shkstart@126.com
 * @create 2020  17:23
 */
public class EdenSurvivorTest {
    public static void main(String[] args) {
        System.out.println("我只是来打个酱油~");
        try {
            Thread.sleep(1000000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```



解释：

- 但是实际测试过程中发现它之间的比例并不是8:1:1  而是6:1:1  

- 有一个**自适应内存分配策略**，但是设置了并没有起作用，需要设置参数`-XX:SurvivorRatio=8`  这样的话，比例才是8:1:1 



设置以后，测试：8:1:1 

<img src="../media/pictures/JVM.assets/image-20201216173622967.png" alt="image-20201216173622967" style="zoom:33%;" />



<img src="../media/pictures/JVM.assets/image-20201217085624656.png" alt="image-20201217085624656" style="zoom:33%;" />

**注意：**

这里新参数 `-Xmn`用来设置新生代最大内存大小，这就有一个问题，如果这里设置啦，同时`-XX:NewRatio`也设置了，但是两个结果不统一，到底以谁的为主呢？

对上述代码设置如下JVM参数：

```jvm
-Xms600m -Xmx600m -XX:NewRatio=5 -Xmn200m
```

实际结果：

总共是600m，按照比例1:5，新生代应该是100m，结果是新生代是200m。说明当两者有矛盾的时候，以`-Xmn`参数为主

<img src="../media/pictures/JVM.assets/image-20201217090850137.png" alt="image-20201217090850137" style="zoom:50%;" />

![image-20201217085903428](../media/pictures/JVM.assets/image-20201217085903428.png)

注意：中间survivor两个区，回循环（垃圾回收的时候，可能会一边复制到另一边。）



### 图解对象分配过程

![image-20201217092731433](../media/pictures/JVM.assets/image-20201217092731433.png)

解释：

- 当Eden区满的时候，才会进行YGC。没用的对象清除，绿色有用的留下放到幸存者0区。下一次如果Eden满了的话，同样的清理没用的，留下，但是有用的这一次绿色的会放到1区，依次循环。
- Eden满了会进行YGC，那么如果幸存者区满了的话，会怎么样呢？ 会直接进入Old区。就像有些人过分优秀，直接一等功或者特等功，直接晋升超级特种兵。
- 那么会不会没有进过Eden区，直接进入老年代呢？当然有，有些人就是含着金钥匙出来的，比如各种二代，确实和普通人不一样。
- 后面Promotion（晋升的意思）。
- 对象从Eden到幸存者0和1区，再到Old区。就像一个新兵蛋子，好好干晋升到排长或者晋升特种兵，后来连长或者超级特种兵，最后十年老兵（Old）基本到了这里就快要凉凉啦。（其实也像程序员，从菜鸟到精英，到35岁，到凋零。。。瞬间难过）



<img src="../media/pictures/JVM.assets/image-20201217102434293.png" alt="image-20201217102434293" style="zoom:33%;" />

<img src="../media/pictures/JVM.assets/image-20201217102509276.png" alt="image-20201217102509276" style="zoom:33%;" />



前辈总结：

<img src="../media/pictures/JVM.assets/image-20201217102654629.png" alt="image-20201217102654629" style="zoom:33%;" />



自己总结：

年轻人人多，竞争激烈，一旦经历过风雨挺过去，到了老年代，公司就不会轻易动他啦，毕竟是公司元老。



<img src="../media/pictures/JVM.assets/image-20201217103558151.png" alt="image-20201217103558151" style="zoom:50%;" />



```java
package com.atguigu.java1;

import java.util.ArrayList;
import java.util.Random;

/**
 * -Xms600m -Xmx600m
 * @author shkstart  shkstart@126.com
 * @create 2020  17:51
 */
public class HeapInstanceTest {
    byte[] buffer = new byte[new Random().nextInt(1024 * 200)];

    public static void main(String[] args) {
        ArrayList<HeapInstanceTest> list = new ArrayList<HeapInstanceTest>();
        while (true) {
            list.add(new HeapInstanceTest());
            try {
                Thread.sleep(10);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
}
```



用监控软件实际测试发现，如果Old区满了以后，会报OutOfMemoryError。



#### 常用调试工具

<img src="../media/pictures/JVM.assets/image-20201217110909683.png" alt="image-20201217110909683" style="zoom:33%;" />

这里用Jprofiler，电脑端用11，IDEA中用Jprofiler插件。

正在安装。。。







### Minor GC、Major GC、Full GC

<img src="../media/pictures/JVM.assets/image-20201217111316786.png" alt="image-20201217111316786" style="zoom:33%;" />

#### 最简单的分布式GC策略的触发条件

<img src="../media/pictures/JVM.assets/image-20201217112958136.png" alt="image-20201217112958136" style="zoom:33%;" />

解释：

最后这个STW，就好像家长在收拾家，你一直在扔垃圾。这样明显不对。这时候，他会让你暂停扔垃圾，他收拾垃圾。



<img src="../media/pictures/JVM.assets/image-20201222135711610.png" alt="image-20201222135711610" style="zoom:33%;" />



<img src="../media/pictures/JVM.assets/image-20201222135801756.png" alt="image-20201222135801756" style="zoom:33%;" />

<img src="../media/pictures/JVM.assets/image-20201222135828721.png" alt="image-20201222135828721" style="zoom:33%;" />



自己测试：

```java
package com.atguigu.java1;

import java.util.ArrayList;
import java.util.List;

/**
 * 测试MinorGC 、 MajorGC、FullGC
 * -Xms9m -Xmx9m -XX:+PrintGCDetails
 * @author shkstart  shkstart@126.com
 * @create 2020  14:19
 */
public class GCTest {
    public static void main(String[] args) {
        int i = 0;
        try {
            List<String> list = new ArrayList<>();
            String a = "atguigu.com";
            while (true) {
                list.add(a);
                a = a + a;
                i++;
            }

        } catch (Throwable t) {
            t.printStackTrace();
            System.out.println("遍历次数为：" + i);
        }
    }
}
```

输出结果：

```
[GC (Allocation Failure) [PSYoungGen: 2039K->504K(2560K)] 2039K->736K(9728K), 0.0026483 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
[GC (Allocation Failure) [PSYoungGen: 2544K->504K(2560K)] 2776K->2122K(9728K), 0.0023130 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
[GC (Allocation Failure) [PSYoungGen: 1981K->504K(2560K)] 3599K->2858K(9728K), 0.0019741 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
[Full GC (Ergonomics) [PSYoungGen: 1256K->0K(2560K)] [ParOldGen: 6578K->4872K(7168K)] 7834K->4872K(9728K), [Metaspace: 3479K->3479K(1056768K)], 0.0201822 secs] [Times: user=0.03 sys=0.02, real=0.02 secs] 
[GC (Allocation Failure) [PSYoungGen: 0K->0K(2560K)] 4872K->4872K(9728K), 0.0008406 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
[Full GC (Allocation Failure) [PSYoungGen: 0K->0K(2560K)] [ParOldGen: 4872K->4854K(7168K)] 4872K->4854K(9728K), [Metaspace: 3479K->3479K(1056768K)], 0.0157749 secs] [Times: user=0.02 sys=0.00, real=0.02 secs] 
遍历次数为：16
Heap
 PSYoungGen      total 2560K, used 100K [0x00000000ffd00000, 0x0000000100000000, 0x0000000100000000)
  eden space 2048K, 4% used [0x00000000ffd00000,0x00000000ffd191f8,0x00000000fff00000)
  from space 512K, 0% used [0x00000000fff80000,0x00000000fff80000,0x0000000100000000)
  to   space 512K, 0% used [0x00000000fff00000,0x00000000fff00000,0x00000000fff80000)
 ParOldGen       total 7168K, used 4854K [0x00000000ff600000, 0x00000000ffd00000, 0x00000000ffd00000)
  object space 7168K, 67% used [0x00000000ff600000,0x00000000ffabd910,0x00000000ffd00000)
 Metaspace       used 3512K, capacity 4502K, committed 4864K, reserved 1056768K
  class space    used 391K, capacity 394K, committed 512K, reserved 1048576K
java.lang.OutOfMemoryError: Java heap space
	at java.util.Arrays.copyOf(Arrays.java:3332)
	at java.lang.AbstractStringBuilder.ensureCapacityInternal(AbstractStringBuilder.java:124)
	at java.lang.AbstractStringBuilder.append(AbstractStringBuilder.java:448)
	at java.lang.StringBuilder.append(StringBuilder.java:136)
	at com.atguigu.java1.GCTest.main(GCTest.java:20)
```



分析：首先进行了三次年轻代的GC，而后做了一次Full GC，而后进行GC，后来又进行一次Full GC。



### 堆空间分代思想

<img src="../media/pictures/JVM.assets/image-20201222142432609.png" alt="image-20201222142432609" style="zoom:33%;" />



<img src="../media/pictures/JVM.assets/image-20201222142504956.png" alt="image-20201222142504956" style="zoom:33%;" />



举例解释说明：

比如一个小区，现在处于疫情监控中，一些已经感染的人，一些和感染者接触过的人，一些没有啥问题的人，就可以分别看成年轻代，老年代，永久代。医护人员吗，每天最需要关注的其实就是年轻代的人，每天测体温等，实时监控。老年代人，相对时间长一点检查一次就好。

同样学校分班，都是便于管理，便于学习。



### 内存分配策略

<img src="../media/pictures/JVM.assets/image-20201222143551823.png" alt="image-20201222143551823" style="zoom:33%;" />



<img src="../media/pictures/JVM.assets/image-20201222143733896.png" alt="image-20201222143733896" style="zoom:33%;" />



大对象：大的数组，或者字符串

代码测试：

```java
package com.atguigu.java1;

/** 测试：大对象直接进入老年代
 * -Xms60m -Xmx60m -XX:NewRatio=2 -XX:SurvivorRatio=8 -XX:+PrintGCDetails
 * @author shkstart  shkstart@126.com
 * @create 2020  21:48
 */
public class YoungOldAreaTest {
    public static void main(String[] args) {
        byte[] buffer = new byte[1024 * 1024 * 20];//20m

    }
}
```



结果：

```
Heap
 PSYoungGen      total 18432K, used 2631K [0x00000000fec00000, 0x0000000100000000, 0x0000000100000000)
  eden space 16384K, 16% used [0x00000000fec00000,0x00000000fee91e00,0x00000000ffc00000)
  from space 2048K, 0% used [0x00000000ffe00000,0x00000000ffe00000,0x0000000100000000)
  to   space 2048K, 0% used [0x00000000ffc00000,0x00000000ffc00000,0x00000000ffe00000)
 ParOldGen       total 40960K, used 20480K [0x00000000fc400000, 0x00000000fec00000, 0x00000000fec00000)
  object space 40960K, 50% used [0x00000000fc400000,0x00000000fd800010,0x00000000fec00000)
 Metaspace       used 3482K, capacity 4498K, committed 4864K, reserved 1056768K
  class space    used 387K, capacity 390K, committed 512K, reserved 1048576K
```



解释：

new 的大数组，20M，直接放到了老年代中，20480K



### 为对象分配内存：TLAB

<img src="../media/pictures/JVM.assets/image-20201222145507769.png" alt="image-20201222145507769" style="zoom:33%;" />

Allocation（分配，配置）



<img src="../media/pictures/JVM.assets/image-20201222145908341.png" alt="image-20201222145908341" style="zoom:33%;" />

面试问题：堆空间一定都是共享的吗？

不是，每个线程有独有的TLAB。

<img src="../media/pictures/JVM.assets/image-20201222150717877.png" alt="image-20201222150717877" style="zoom:33%;" />



<img src="../media/pictures/JVM.assets/image-20201222150827197.png" alt="image-20201222150827197" style="zoom:33%;" />

<img src="../media/pictures/JVM.assets/image-20201222150844153.png" alt="image-20201222150844153" style="zoom:33%;" />



### 堆空间的参数设置小结

<img src="../media/pictures/JVM.assets/image-20201222151552126.png" alt="image-20201222151552126" style="zoom:33%;" />





代码测试：

查看所有的参数  用前两个参数

```java
package com.atguigu.java1;

/**
 * 测试堆空间常用的jvm参数：
 * -XX:+PrintFlagsInitial : 查看所有的参数的默认初始值
 * -XX:+PrintFlagsFinal  ：查看所有的参数的最终值（可能会存在修改，不再是初始值）
 *      具体查看某个参数的指令： jps：查看当前运行中的进程
 *                             jinfo -flag SurvivorRatio 进程id
 *
 * -Xms：初始堆空间内存 （默认为物理内存的1/64）
 * -Xmx：最大堆空间内存（默认为物理内存的1/4）
 * -Xmn：设置新生代的大小。(初始值及最大值)
 * -XX:NewRatio：配置新生代与老年代在堆结构的占比
 * -XX:SurvivorRatio：设置新生代中Eden和S0/S1空间的比例
 * -XX:MaxTenuringThreshold：设置新生代垃圾的最大年龄
 * -XX:+PrintGCDetails：输出详细的GC处理日志
 * 打印gc简要信息：① -XX:+PrintGC   ② -verbose:gc
 * -XX:HandlePromotionFailure：是否设置空间分配担保
 *
 * @author shkstart  shkstart@126.com
 * @create 2020  17:18
 */
public class HeapArgsTest {
    public static void main(String[] args) {

    }
}
```

结果：

![image-20201222151900890](../media/pictures/JVM.assets/image-20201222151900890.png)



解释分析：带`:=` 的，是修改过的，说明不是默认值。



<img src="../media/pictures/JVM.assets/image-20201222153101993.png" alt="image-20201222153101993" style="zoom:33%;" />

<img src="../media/pictures/JVM.assets/image-20201222153736333.png" alt="image-20201222153736333" style="zoom:33%;" />

解释JDK7之后，这个参数相当于写死啦，直接是True啦。这个要是写死的话，感觉会有风险。



### 堆是分配对象的唯一原则吗

<img src="../media/pictures/JVM.assets/image-20201222154443582.png" alt="image-20201222154443582" style="zoom:33%;" />



#### 逃逸分析

<img src="../media/pictures/JVM.assets/image-20201223090906729.png" alt="image-20201223090906729" style="zoom:33%;" />

解析：

对象在方法内部被定义，同时在内部被使用，则没有逃逸。如果对象在方法内部被定义，而在外部被引用，则认为发生逃逸。



<img src="../media/pictures/JVM.assets/image-20201223091614723.png" alt="image-20201223091614723" style="zoom:33%;" />



下面这个，很巧妙的一个设计，可以让对象不逃逸：

<img src="../media/pictures/JVM.assets/image-20201223091635990.png" alt="image-20201223091635990" style="zoom:33%;" />



逃逸分析代码：

```java
package com.atguigu.java2;

/**
 * 逃逸分析
 *
 * 如何快速的判断是否发生了逃逸分析，大家就看new的对象实体是否有可能在方法外被调用。
 * @author shkstart
 * @create 2020 下午 4:00
 */
public class EscapeAnalysis {

    public EscapeAnalysis obj;

    /*
    方法返回EscapeAnalysis对象，发生逃逸
     */
    public EscapeAnalysis getInstance(){
        return obj == null? new EscapeAnalysis() : obj;
    }
    /*
    为成员属性赋值，发生逃逸
     */
    public void setObj(){
        this.obj = new EscapeAnalysis();
    }
    //思考：如果当前的obj引用声明为static的？仍然会发生逃逸。

    /*
    对象的作用域仅在当前方法中有效，没有发生逃逸
     */
    public void useEscapeAnalysis(){
        EscapeAnalysis e = new EscapeAnalysis();
    }
    /*
    引用成员变量的值，发生逃逸
     */
    public void useEscapeAnalysis1(){
        EscapeAnalysis e = getInstance();
        //getInstance().xxx()同样会发生逃逸
    }
}
```



<img src="../media/pictures/JVM.assets/image-20201223092838102.png" alt="image-20201223092838102" style="zoom:33%;" />



**结论：开关中能使用局部变量，就不要使用在方法外定义。**



#### 逃逸分析 代码优化

<img src="../media/pictures/JVM.assets/image-20201223101507000.png" alt="image-20201223101507000" style="zoom:33%;" />



##### 栈上分配

测试代码：

```java
package com.atguigu.java2;

/**
 * 栈上分配测试
 * -Xmx1G -Xms1G -XX:-DoEscapeAnalysis -XX:+PrintGCDetails
 */
public class StackAllocation {
    public static void main(String[] args) {
        long start = System.currentTimeMillis();

        for (int i = 0; i < 10000000; i++) {
            alloc();
        }
        // 查看执行时间
        long end = System.currentTimeMillis();
        System.out.println("花费的时间为： " + (end - start) + " ms");
        // 为了方便查看堆内存中对象个数，线程sleep
        try {
            Thread.sleep(1000000);
        } catch (InterruptedException e1) {
            e1.printStackTrace();
        }
    }

    private static void alloc() {
        User user = new User();//未发生逃逸
    }

    static class User {

    }
}

```



###### 测试（1）

配置参数：（没有开启逃逸分析）

```bash
-Xmx1G -Xms1G -XX:-DoEscapeAnalysis -XX:+PrintGCDetails
```

运行结果：

```
花费的时间为： 97 ms
```

监控软件分析：

![image-20201223095745075](../media/pictures/JVM.assets/image-20201223095745075.png)



###### 测试（2）

配置参数：（开启逃逸分析，和上面的区别是这里是+，上面是-）

```bash
-Xmx1G -Xms1G -XX:+DoEscapeAnalysis -XX:+PrintGCDetails
```

运行结果：

```
花费的时间为： 7 ms
```

监控软件分析：

![image-20201223095641683](../media/pictures/JVM.assets/image-20201223095641683.png)



对比两者同样创建1000万对象，时间差异很大。在内存中占用情况也不一样，开启逃逸分析以后，对象数（实例数量）明显少很多，而上面没有开启逃逸分析的情况，实例数还是1000万个。



##### 同步省略（消除）

<img src="../media/pictures/JVM.assets/image-20201223102245851.png" alt="image-20201223102245851" style="zoom:33%;" />



<img src="../media/pictures/JVM.assets/image-20201223102453857.png" alt="image-20201223102453857" style="zoom:33%;" />

解释：

对于同步（加锁）来说，如果每次调用f（）方法，用到的锁hollis，都不是同一个，因为每次调用都是new的一个hollis对象，那么并没有起到同步的作用。

因为锁只被这个线程使用到，并没有被其他线程使用到，编译器会优化这一部分代码，取消同步。这叫**锁消除**。



代码测试：

```java
package com.atguigu.java2;

/**
 * 同步省略说明
 */
public class SynchronizedTest {
    public void f() {
        Object hollis = new Object();
        synchronized(hollis) {
            System.out.println(hollis);
        }
    }
}
```



![image-20201223103039097](../media/pictures/JVM.assets/image-20201223103039097.png)



##### 标量替换

<img src="../media/pictures/JVM.assets/image-20201223103917115.png" alt="image-20201223103917115" style="zoom:33%;" />



<img src="../media/pictures/JVM.assets/image-20201223103934297.png" alt="image-20201223103934297" style="zoom:33%;" />

解释：上面绿色的代码，经过标量替换以后就变成了下面黄色的部分。



<img src="../media/pictures/JVM.assets/image-20201223105004925.png" alt="image-20201223105004925" style="zoom:33%;" />





###### 代码测试（1）

参数设置：（进行逃逸分析，不进行标量替换，最后一个参数-）

```bash
-Xmx100m -Xms100m -XX:+DoEscapeAnalysis -XX:+PrintGC -XX:-EliminateAllocations
```

代码测试

```java
package com.atguigu.java2;

/**
 * 标量替换测试
 *  -Xmx100m -Xms100m -XX:+DoEscapeAnalysis -XX:+PrintGC -XX:-EliminateAllocations
 */
public class ScalarReplace {
    public static class User {
        public int id;
        public String name;
    }

    public static void alloc() {
        User u = new User();//未发生逃逸
        u.id = 5;
        u.name = "www.atguigu.com";
    }

    public static void main(String[] args) {
        long start = System.currentTimeMillis();
        for (int i = 0; i < 10000000; i++) {
            alloc();
        }
        long end = System.currentTimeMillis();
        System.out.println("花费的时间为： " + (end - start) + " ms");
    }
}
```



结果：

```bash
[GC (Allocation Failure)  25600K->848K(98304K), 0.0027447 secs]
[GC (Allocation Failure)  26448K->784K(98304K), 0.0015434 secs]
[GC (Allocation Failure)  26384K->736K(98304K), 0.0016396 secs]
[GC (Allocation Failure)  26336K->720K(98304K), 0.0012140 secs]
[GC (Allocation Failure)  26320K->720K(98304K), 0.0014937 secs]
[GC (Allocation Failure)  26320K->752K(101376K), 0.0014161 secs]
[GC (Allocation Failure)  32496K->684K(101376K), 0.0011951 secs]
[GC (Allocation Failure)  32428K->684K(100352K), 0.0007323 secs]
花费的时间为： 96 ms
```



###### 代码测试（2）

参数设置：（进行逃逸分析，进行标量替换，最后一个参数+）

```bash
Xmx100m -Xms100m -XX:+DoEscapeAnalysis -XX:+PrintGC -XX:+EliminateAllocations
```

代码：和上面一样

结果：

```
花费的时间为： 10 ms
```



**对比解析：通过对比可以，进行标量替换以后，垃圾回收没有啦，时间缩短很多。**





<img src="../media/pictures/JVM.assets/image-20201223110916461.png" alt="image-20201223110916461" style="zoom:33%;" />



#### 逃逸分析小结

<img src="../media/pictures/JVM.assets/image-20201223111116832.png" alt="image-20201223111116832" style="zoom:33%;" />

最后一句话：

**对象实例都是分配在堆上。**

原因是Oracle Hotspot JVM并没有将这项技术用到JVM中，所以 上面那句话。（本来JVM会在栈上分配那些不会逃逸的对象，但是没用到，那就只有堆存放对象。）



### 本章小结：

<img src="../media/pictures/JVM.assets/image-20201223111742435.png" alt="image-20201223111742435" style="zoom:33%;" />



## 方法区

- 栈、堆、方法区的交互关系
- 方法区的理解
- 设置方法区的大小与OOM
- 方法区的内部结构
- 方法区使用举例
- 方法区的演进细节
- 方法区的垃圾回收
- 总结



### 栈、堆、方法区的交互关系

#### 运行时数据区结构图

![image-20201223112513091](../media/pictures/JVM.assets/image-20201223112513091.png)



从线程共享与否的角度来看

![image-20201223112648579](../media/pictures/JVM.assets/image-20201223112648579.png)



解释：

- 左边线程共享的两个堆和元空间，都会报异常OOM，也会GC。
- 右边线程私有的部分，虚拟机栈和本地方法区会报错StackOverFlow，但是没有GC，后面的**程序计数器不会把报异常，也不存在GC**。
- 中间ThreadLocal可以保证多个线程在并发的时候的安全性。



#### 栈、堆、方法区的关系

![image-20201223133439556](../media/pictures/JVM.assets/image-20201223133439556.png)







reference 叫引用，实体在堆中。



### 方法区的理解

官方英文文档：

https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-2.html#jvms-2.5.4

<img src="../media/pictures/JVM.assets/image-20201223135510172.png" alt="image-20201223135510172" style="zoom:33%;" />



<img src="../media/pictures/JVM.assets/image-20201223140135202.png" alt="image-20201223140135202" style="zoom:33%;" />



代码：

```java
package com.atguigu.java;

/**
 *  测试设置方法区大小参数的默认值
 *
 *  jdk7及以前：
 *  -XX:PermSize=100m -XX:MaxPermSize=100m
 *
 *  jdk8及以后：
 *  -XX:MetaspaceSize=100m  -XX:MaxMetaspaceSize=100m
 */
public class MethodAreaDemo {
    public static void main(String[] args) {
        System.out.println("start...");
        try {
            Thread.sleep(1000000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        System.out.println("end...");
    }
}
```

监控解析：就上面几行代码，实际运行起来加载进来的类就有1600多个，可怕。



<img src="../media/pictures/JVM.assets/image-20201223141353631.png" alt="image-20201223141353631" style="zoom:33%;" />



#### Hotspot中方法区的演进

<img src="../media/pictures/JVM.assets/image-20201223142751020.png" alt="image-20201223142751020" style="zoom:33%;" />



<img src="../media/pictures/JVM.assets/image-20201223142906631.png" alt="image-20201223142906631" style="zoom:33%;" />



#### 方法区的概述

![image-20201223142817690](../media/pictures/JVM.assets/image-20201223142817690.png)



### 设置方法区的大小与OOM

<img src="../media/pictures/JVM.assets/image-20201223144653975.png" alt="image-20201223144653975" style="zoom:33%;" />



<img src="../media/pictures/JVM.assets/image-20201223144820141.png" alt="image-20201223144820141" style="zoom:33%;" />

由于本地是装的JDK8，所以没有PermSize两个参数。

<img src="../media/pictures/JVM.assets/image-20201223144956963.png" alt="image-20201223144956963" style="zoom: 67%;" />

只有下面两个参数：

<img src="../media/pictures/JVM.assets/image-20201223145217692.png" alt="image-20201223145217692" style="zoom: 67%;" />



进过计算器计算：21807104 / 1024 / 1024  =  20.796875 （这就是上面说的大约21M）

下面的数字很大，除三个1024，算出来也很大。很大的G。这里应该不是电脑实际内存 大小。



#### 测试OOM

测试代码：

```java
package com.atguigu.java;

import com.sun.xml.internal.ws.org.objectweb.asm.ClassWriter;
import jdk.internal.org.objectweb.asm.Opcodes;

/**
 * 测试OOM异常
 * jdk6/7中：
 * -XX:PermSize=10m -XX:MaxPermSize=10m
 * <p>
 * jdk8中：
 * -XX:MetaspaceSize=10m -XX:MaxMetaspaceSize=10m
 *
 * @author shkstart  shkstart@126.com
 * @create 2020  22:24
 */
public class OOMTest extends ClassLoader {
	public static void main(String[] args) {
		int j = 0;
		try {
			OOMTest test = new OOMTest();
			for (int i = 0; i < 10000; i++) {
				//创建ClassWriter对象，用于生成类的二进制字节码
				ClassWriter classWriter = new ClassWriter(0);
				//指明版本号，修饰符，类名，包名，父类，接口
				classWriter.visit(Opcodes.V1_6, Opcodes.ACC_PUBLIC, "Class" + i, null, "java/lang/Object", null);
				//返回byte[]
				byte[] code = classWriter.toByteArray();
				//类的加载
				test.defineClass("Class" + i, code, 0, code.length);//Class对象
				j++;
			}
		} finally {
			System.out.println(j);
		}
	}
}
```

测试结果：（JDK8）

```bash
8531
Exception in thread "main" java.lang.OutOfMemoryError: Metaspace
	at java.lang.ClassLoader.defineClass1(Native Method)
	at java.lang.ClassLoader.defineClass(ClassLoader.java:763)
	at java.lang.ClassLoader.defineClass(ClassLoader.java:642)
	at com.atguigu.java.OOMTest.main(OOMTest.java:29)
```



总结

虽然JDK7和JDK8都报错OOM，但是后面显示的详细信息不一样。

- JDK7 是 java.lang.OutOfMemoryError: **PermGen space**
- JDK8 是 java.lang.OutOfMemoryError: **Metaspace**



### 方法区的内部结构

<img src="../media/pictures/JVM.assets/image-20210104091820545.png" alt="image-20210104091820545" style="zoom:40%;" />



<img src="../media/pictures/JVM.assets/image-20210104101636051.png" alt="image-20210104101636051" style="zoom:35%;" />

<img src="../media/pictures/JVM.assets/image-20210104102301622.png" alt="image-20210104102301622" style="zoom:33%;" />



#### 测试方法区内部构成



源代码：

```java
package com.atguigu.java;

import java.io.Serializable;

/**
 * 测试方法区的内部构成
 * @author shkstart  shkstart@126.com
 * @create 2020  23:39
 */
public class MethodInnerStrucTest extends Object implements Comparable<String>,Serializable {
    //属性
    public int num = 10;
    private static String str = "测试方法的内部结构";
    //构造器
    //方法
    public void test1(){
        int count = 20;
        System.out.println("count = " + count);
    }
    public static int test2(int cal){
        int result = 0;
        try {
            int value = 30;
            result = value / cal;
        } catch (Exception e) {
            e.printStackTrace();
        }
        return result;
    }

    @Override
    public int compareTo(String o) {
        return 0;
    }
}
```



反编译：

终端打开class文件

<img src="../media/pictures/JVM.assets/image-20210104104912944.png" alt="image-20210104104912944" style="zoom: 67%;" />

```
javap -v -p MethodInnerStrucTest.class > test.txt (将结果重定向到text.txt中)
```

-p是将 私有属性也反编译

反编译结果：

```
Classfile /D:/Study/尚硅谷JVM/JVM上篇 内存和垃圾回收/代码/JVMDemo/out/production/chapter09/com/atguigu/java/MethodInnerStrucTest.class
  Last modified 2021-1-4; size 1626 bytes
  MD5 checksum 0d0fcb54854d4ce183063df985141ad0
  Compiled from "MethodInnerStrucTest.java"
public class com.atguigu.java.MethodInnerStrucTest extends java.lang.Object implements java.lang.Comparable<java.lang.String>, java.io.Serializable
  minor version: 0
  major version: 52
  flags: ACC_PUBLIC, ACC_SUPER
Constant pool:
   #1 = Methodref          #18.#52        // java/lang/Object."<init>":()V
   #2 = Fieldref           #17.#53        // com/atguigu/java/MethodInnerStrucTest.num:I
   #3 = Fieldref           #54.#55        // java/lang/System.out:Ljava/io/PrintStream;
   #4 = Class              #56            // java/lang/StringBuilder
   #5 = Methodref          #4.#52         // java/lang/StringBuilder."<init>":()V
   #6 = String             #57            // count =
   #7 = Methodref          #4.#58         // java/lang/StringBuilder.append:(Ljava/lang/String;)Ljava/lang/StringBuilder;
   #8 = Methodref          #4.#59         // java/lang/StringBuilder.append:(I)Ljava/lang/StringBuilder;
   #9 = Methodref          #4.#60         // java/lang/StringBuilder.toString:()Ljava/lang/String;
  #10 = Methodref          #61.#62        // java/io/PrintStream.println:(Ljava/lang/String;)V
  #11 = Class              #63            // java/lang/Exception
  #12 = Methodref          #11.#64        // java/lang/Exception.printStackTrace:()V
  #13 = Class              #65            // java/lang/String
  #14 = Methodref          #17.#66        // com/atguigu/java/MethodInnerStrucTest.compareTo:(Ljava/lang/String;)I
  #15 = String             #67            // 测试方法的内部结构
  #16 = Fieldref           #17.#68        // com/atguigu/java/MethodInnerStrucTest.str:Ljava/lang/String;
  #17 = Class              #69            // com/atguigu/java/MethodInnerStrucTest
  #18 = Class              #70            // java/lang/Object
  #19 = Class              #71            // java/lang/Comparable
  #20 = Class              #72            // java/io/Serializable
  #21 = Utf8               num
  #22 = Utf8               I
  #23 = Utf8               str
  #24 = Utf8               Ljava/lang/String;
  #25 = Utf8               <init>
  #26 = Utf8               ()V
  #27 = Utf8               Code
  #28 = Utf8               LineNumberTable
  #29 = Utf8               LocalVariableTable
  #30 = Utf8               this
  #31 = Utf8               Lcom/atguigu/java/MethodInnerStrucTest;
  #32 = Utf8               test1
  #33 = Utf8               count
  #34 = Utf8               test2
  #35 = Utf8               (I)I
  #36 = Utf8               value
  #37 = Utf8               e
  #38 = Utf8               Ljava/lang/Exception;
  #39 = Utf8               cal
  #40 = Utf8               result
  #41 = Utf8               StackMapTable
  #42 = Class              #63            // java/lang/Exception
  #43 = Utf8               compareTo
  #44 = Utf8               (Ljava/lang/String;)I
  #45 = Utf8               o
  #46 = Utf8               (Ljava/lang/Object;)I
  #47 = Utf8               <clinit>
  #48 = Utf8               Signature
  #49 = Utf8               Ljava/lang/Object;Ljava/lang/Comparable<Ljava/lang/String;>;Ljava/io/Serializable;
  #50 = Utf8               SourceFile
  #51 = Utf8               MethodInnerStrucTest.java
  #52 = NameAndType        #25:#26        // "<init>":()V
  #53 = NameAndType        #21:#22        // num:I
  #54 = Class              #73            // java/lang/System
  #55 = NameAndType        #74:#75        // out:Ljava/io/PrintStream;
  #56 = Utf8               java/lang/StringBuilder
  #57 = Utf8               count =
  #58 = NameAndType        #76:#77        // append:(Ljava/lang/String;)Ljava/lang/StringBuilder;
  #59 = NameAndType        #76:#78        // append:(I)Ljava/lang/StringBuilder;
  #60 = NameAndType        #79:#80        // toString:()Ljava/lang/String;
  #61 = Class              #81            // java/io/PrintStream
  #62 = NameAndType        #82:#83        // println:(Ljava/lang/String;)V
  #63 = Utf8               java/lang/Exception
  #64 = NameAndType        #84:#26        // printStackTrace:()V
  #65 = Utf8               java/lang/String
  #66 = NameAndType        #43:#44        // compareTo:(Ljava/lang/String;)I
  #67 = Utf8               测试方法的内部结构
  #68 = NameAndType        #23:#24        // str:Ljava/lang/String;
  #69 = Utf8               com/atguigu/java/MethodInnerStrucTest
  #70 = Utf8               java/lang/Object
  #71 = Utf8               java/lang/Comparable
  #72 = Utf8               java/io/Serializable
  #73 = Utf8               java/lang/System
  #74 = Utf8               out
  #75 = Utf8               Ljava/io/PrintStream;
  #76 = Utf8               append
  #77 = Utf8               (Ljava/lang/String;)Ljava/lang/StringBuilder;
  #78 = Utf8               (I)Ljava/lang/StringBuilder;
  #79 = Utf8               toString
  #80 = Utf8               ()Ljava/lang/String;
  #81 = Utf8               java/io/PrintStream
  #82 = Utf8               println
  #83 = Utf8               (Ljava/lang/String;)V
  #84 = Utf8               printStackTrace
{
  public int num;
    descriptor: I
    flags: ACC_PUBLIC

  private static java.lang.String str;
    descriptor: Ljava/lang/String;
    flags: ACC_PRIVATE, ACC_STATIC

  public com.atguigu.java.MethodInnerStrucTest();
    descriptor: ()V
    flags: ACC_PUBLIC
    Code:
      stack=2, locals=1, args_size=1
         0: aload_0
         1: invokespecial #1                  // Method java/lang/Object."<init>":()V
         4: aload_0
         5: bipush        10
         7: putfield      #2                  // Field num:I
        10: return
      LineNumberTable:
        line 10: 0
        line 12: 4
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0      11     0  this   Lcom/atguigu/java/MethodInnerStrucTest;

  public void test1();
    descriptor: ()V
    flags: ACC_PUBLIC
    Code:
      stack=3, locals=2, args_size=1
         0: bipush        20
         2: istore_1
         3: getstatic     #3                  // Field java/lang/System.out:Ljava/io/PrintStream;
         6: new           #4                  // class java/lang/StringBuilder
         9: dup
        10: invokespecial #5                  // Method java/lang/StringBuilder."<init>":()V
        13: ldc           #6                  // String count =
        15: invokevirtual #7                  // Method java/lang/StringBuilder.append:(Ljava/lang/String;)Ljava/lang/StringBuilder;
        18: iload_1
        19: invokevirtual #8                  // Method java/lang/StringBuilder.append:(I)Ljava/lang/StringBuilder;
        22: invokevirtual #9                  // Method java/lang/StringBuilder.toString:()Ljava/lang/String;
        25: invokevirtual #10                 // Method java/io/PrintStream.println:(Ljava/lang/String;)V
        28: return
      LineNumberTable:
        line 17: 0
        line 18: 3
        line 19: 28
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0      29     0  this   Lcom/atguigu/java/MethodInnerStrucTest;
            3      26     1 count   I

  public static int test2(int);
    descriptor: (I)I
    flags: ACC_PUBLIC, ACC_STATIC
    Code:
      stack=2, locals=3, args_size=1
         0: iconst_0
         1: istore_1
         2: bipush        30
         4: istore_2
         5: iload_2
         6: iload_0
         7: idiv
         8: istore_1
         9: goto          17
        12: astore_2
        13: aload_2
        14: invokevirtual #12                 // Method java/lang/Exception.printStackTrace:()V
        17: iload_1
        18: ireturn
      Exception table:
         from    to  target type
             2     9    12   Class java/lang/Exception
      LineNumberTable:
        line 21: 0
        line 23: 2
        line 24: 5
        line 27: 9
        line 25: 12
        line 26: 13
        line 28: 17
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            5       4     2 value   I
           13       4     2     e   Ljava/lang/Exception;
            0      19     0   cal   I
            2      17     1 result   I
      StackMapTable: number_of_entries = 2
        frame_type = 255 /* full_frame */
          offset_delta = 12
          locals = [ int, int ]
          stack = [ class java/lang/Exception ]
        frame_type = 4 /* same */

  public int compareTo(java.lang.String);
    descriptor: (Ljava/lang/String;)I
    flags: ACC_PUBLIC
    Code:
      stack=1, locals=2, args_size=2
         0: iconst_0
         1: ireturn
      LineNumberTable:
        line 33: 0
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0       2     0  this   Lcom/atguigu/java/MethodInnerStrucTest;
            0       2     1     o   Ljava/lang/String;

  public int compareTo(java.lang.Object);
    descriptor: (Ljava/lang/Object;)I
    flags: ACC_PUBLIC, ACC_BRIDGE, ACC_SYNTHETIC
    Code:
      stack=2, locals=2, args_size=2
         0: aload_0
         1: aload_1
         2: checkcast     #13                 // class java/lang/String
         5: invokevirtual #14                 // Method compareTo:(Ljava/lang/String;)I
         8: ireturn
      LineNumberTable:
        line 10: 0
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0       9     0  this   Lcom/atguigu/java/MethodInnerStrucTest;

  static {};
    descriptor: ()V
    flags: ACC_STATIC
    Code:
      stack=1, locals=0, args_size=0
         0: ldc           #15                 // String 测试方法的内部结构
         2: putstatic     #16                 // Field str:Ljava/lang/String;
         5: return
      LineNumberTable:
        line 13: 0
}
Signature: #49                          // Ljava/lang/Object;Ljava/lang/Comparable<Ljava/lang/String;>;Ljava/io/Serializable;
SourceFile: "MethodInnerStrucTest.java"

```



### 方法区使用举例







### 方法区的演进细节





### 方法区的垃圾回收







### 总结





# Interview

## 1.新生代包含的区域（牛客题）

**Java** **中的堆是** **JVM** **所管理的最**大的一块内存空间，主要用于存放各种类的实例对象。

在 **Java** 中，堆被划分成两个不同的区域：新生代 ( Young )、老年代 ( Old )。新生代 ( Young ) 又被划分为三个区域：Eden、From Survivor、To Survivor。
这样划分的目的是为了使 **JVM** 能够更好的管理堆内存中的对象，包括内存的分配以及回收。
堆的内存模型大致为：

![1586532188773](../media/pictures/JVM.assets/1586532188773.png)





 从图中可以看出： 堆大小 = 新生代 + 老年代。其中，堆的大小可以通过参数 –Xms、-Xmx 来指定。

## 2.JMM（Java内存模型（Java Memory Model）

JMM就是一种符合内存模型规范的，屏蔽了各种硬件和操作系统的访问差异的，保证了Java程序在各种平台下对内存的访问都能保证效果一致的机制及规范。



Java线程之间的通信由Java内存模型（简称为JMM）控制，JMM决定一个线程对共享变量的写入何时对另一个线程可见。从抽象的角度来看，JMM定义了线程和主内存之间的抽象关系：线程之间的共享变量存储在主内存（main memory）中，每个线程都有一个私有的本地内存（local memory），本地内存中存储了该线程以读/写共享变量的副本。本地内存是JMM的一个抽象概念，并不真实存在。它涵盖了缓存，写缓冲区，寄存器以及其他的硬件和编译器优化

volatile变量的写-读可以实现线程之间的通信。

从内存语义的角度来说，volatile与监视器锁有相同的效果：volatile写和监视器的释放有相同的内存语义；volatile读与监视器的获取有相同的内存语义。



**下面的是博客看到的：**

在Java内存模型中，描述了在多线程代码中，哪些行为是正确的、合法的，以及多线程之间如何进行通信，代码中变量的读写行为如何反应到内存、CPU缓存的底层细节。

在Java中包含了几个关键字：volatile、final和synchronized，帮助程序员把代码中的并发需求描述给编译器。Java内存模型中定义了它们的行为，确保正确同步的Java代码在所有的处理器架构上都能正确执行。



参考：https://www.jianshu.com/p/bf158fbb2432  



简答内存模型的时候，最好再答上内存区域。有些面试官也没分清楚。

## 3.下面有关java内存模型的描述，说法错误的是？（没搞清 然后看看）

正确答案: D   你的答案: B (错误)

```
A、JMM通过控制主内存与每个线程的本地内存之间的交互，来为java程序员提供内存可见性保证
B、“synchronized” — 保证在块开始时都同步主内存的值到工作内存，而块结束时将变量同步回主内存
C、“volatile” — 保证修饰后在对变量读写前都会与主内存更新。
D、如果在一个线程构造了一个不可变对象之后（对象仅包含final字段），就可以保证了这个对象被其他线程正确的查看
```



## 4.垃圾回收算法

![1591857119826](../media/pictures/JVM.assets/1591857119826.png)

### 3.1 标记-清除算法

该算法分为“标记”和“清除”阶段：首先标记出所有需要回收的对象，在标记完成后统一回收所有被标记的对象。它是最基础的收集算法，后续的算法都是对其不足进行改进得到。这种垃圾收集算法会带来两个明显的问题：

1. **效率问题**
2. **空间问题（标记清除后会产生大量不连续的碎片）**
3. ![1591857144703](../media/pictures/JVM.assets/1591857144703.png)

### 3.2 复制算法

为了解决效率问题，“复制”收集算法出现了。它可以将内存分为大小相同的两块，每次使用其中的一块。当这一块的内存使用完后，就将还存活的对象复制到另一块去，然后再把使用的空间一次清理掉。这样就使每次的内存回收都是对内存区间的一半进行回收。

![1591857163981](../media/pictures/JVM.assets/1591857163981.png)

### 3.3 标记-整理算法

根据老年代的特点提出的一种标记算法，标记过程仍然与“标记-清除”算法一样，但后续步骤不是直接对可回收对象回收，而是让所有存活的对象向一端移动，然后直接清理掉端边界以外的内存。

![标记-整理算法 ](../media/pictures/JVM.assets/94057049.jpg)

### 3.4 分代收集算法

当前虚拟机的垃圾收集都采用分代收集算法，这种算法没有什么新的思想，只是根据对象存活周期的不同将内存分为几块。一般将 java 堆分为新生代和老年代，这样我们就可以根据各个年代的特点选择合适的垃圾收集算法。

**比如在新生代中，每次收集都会有大量对象死去，所以可以选择复制算法，只需要付出少量对象的复制成本就可以完成每次垃圾收集。而老年代的对象存活几率是比较高的，而且没有额外的空间对它进行分配担保，所以我们必须选择“标记-清除”或“标记-整理”算法进行垃圾收集。**

**延伸面试问题：** HotSpot 为什么要分为新生代和老年代？

根据上面的对分代收集算法的介绍回答。

## 4 垃圾收集器

![image-20210204220205099](../media/pictures/JVM.assets/image-20210204220205099.png)

**如果说收集算法是内存回收的方法论，那么垃圾收集器就是内存回收的具体实现。**

虽然我们对各个收集器进行比较，但并非要挑选出一个最好的收集器。因为直到现在为止还没有最好的垃圾收集器出现，更加没有万能的垃圾收集器，**我们能做的就是根据具体应用场景选择适合自己的垃圾收集器**。试想一下：如果有一种四海之内、任何场景下都适用的完美收集器存在，那么我们的 HotSpot 虚拟机就不会实现那么多不同的垃圾收集器了。



#### 4.1 Serial 收集器

Serial（串行）收集器收集器是最基本、历史最悠久的垃圾收集器了。大家看名字就知道这个收集器是一个单线程收集器了。它的 **“单线程”** 的意义不仅仅意味着它只会使用一条垃圾收集线程去完成垃圾收集工作，更重要的是它在进行垃圾收集工作的时候必须暂停其他所有的工作线程（ **"Stop The World"** ），直到它收集结束。

 **新生代采用复制算法，老年代采用标记-整理算法。**
![ Serial 收集器 ](../media/pictures/JVM.assets/46873026.jpg)

虚拟机的设计者们当然知道 Stop The World 带来的不良用户体验，所以在后续的垃圾收集器设计中停顿时间在不断缩短（仍然还有停顿，寻找最优秀的垃圾收集器的过程仍然在继续）。

但是 Serial 收集器有没有优于其他垃圾收集器的地方呢？当然有，它**简单而高效（与其他收集器的单线程相比）**。Serial 收集器由于没有线程交互的开销，自然可以获得很高的单线程收集效率。Serial 收集器对于运行在 Client 模式下的虚拟机来说是个不错的选择。

#### 4.2 ParNew 收集器

**ParNew 收集器其实就是 Serial 收集器的多线程版本，除了使用多线程进行垃圾收集外，其余行为（控制参数、收集算法、回收策略等等）和 Serial 收集器完全一样。**

 **新生代采用复制算法，老年代采用标记-整理算法。**
![ParNew 收集器 ](../media/pictures/JVM.assets/22018368.jpg)

它是许多运行在 Server 模式下的虚拟机的首要选择，除了 Serial 收集器外，只有它能与 CMS 收集器（真正意义上的并发收集器，后面会介绍到）配合工作。

**并行和并发概念补充：**

- **并行（Parallel）** ：指多条垃圾收集线程并行工作，但此时用户线程仍然处于等待状态。
- **并发（Concurrent）**：指用户线程与垃圾收集线程同时执行（但不一定是并行，可能会交替执行），用户程序在继续运行，而垃圾收集器运行在另一个 CPU 上。

#### 4.3 Parallel Scavenge 收集器

Parallel Scavenge 收集器也是使用复制算法的多线程收集器，它看上去几乎和ParNew都一样。 **那么它有什么特别之处呢？**

```
-XX:+UseParallelGC 

    使用 Parallel 收集器+ 老年代串行

-XX:+UseParallelOldGC

    使用 Parallel 收集器+ 老年代并行

```

**Parallel Scavenge 收集器关注点是吞吐量（高效率的利用 CPU）。CMS 等垃圾收集器的关注点更多的是用户线程的停顿时间（提高用户体验）。所谓吞吐量就是 CPU 中用于运行用户代码的时间与 CPU 总消耗时间的比值。** Parallel Scavenge 收集器提供了很多参数供用户找到最合适的停顿时间或最大吞吐量，如果对于收集器运作不太了解的话，手工优化存在困难的话可以选择把内存管理优化交给虚拟机去完成也是一个不错的选择。

 **新生代采用复制算法，老年代采用标记-整理算法。**
![ParNew 收集器 ](D:/Code/Typora/media/pictures/JVM.assets/22018368.jpg)

#### 4.4.Serial Old 收集器

**Serial 收集器的老年代版本**，它同样是一个单线程收集器。它主要有两大用途：一种用途是在 JDK1.5 以及以前的版本中与 Parallel Scavenge 收集器搭配使用，另一种用途是作为 CMS 收集器的后备方案。

#### 4.5 Parallel Old 收集器

 **Parallel Scavenge 收集器的老年代版本**。使用多线程和“标记-整理”算法。在注重吞吐量以及 CPU 资源的场合，都可以优先考虑 Parallel Scavenge 收集器和 Parallel Old 收集器。

#### 4.6 CMS 收集器

**CMS（Concurrent Mark Sweep）收集器是一种以获取最短回收停顿时间为目标的收集器。它非常符合在注重用户体验的应用上使用。**

**CMS（Concurrent Mark Sweep）收集器是 HotSpot 虚拟机第一款真正意义上的并发收集器，它第一次实现了让垃圾收集线程与用户线程（基本上）同时工作。**

从名字中的**Mark Sweep**这两个词可以看出，CMS 收集器是一种 **“标记-清除”算法**实现的，它的运作过程相比于前面几种垃圾收集器来说更加复杂一些。整个过程分为四个步骤：

- **初始标记：** 暂停所有的其他线程，并记录下直接与 root 相连的对象，速度很快 ；
- **并发标记：** 同时开启 GC 和用户线程，用一个闭包结构去记录可达对象。但在这个阶段结束，这个闭包结构并不能保证包含当前所有的可达对象。因为用户线程可能会不断的更新引用域，所以 GC 线程无法保证可达性分析的实时性。所以这个算法里会跟踪记录这些发生引用更新的地方。
- **重新标记：** 重新标记阶段就是为了修正并发标记期间因为用户程序继续运行而导致标记产生变动的那一部分对象的标记记录，这个阶段的停顿时间一般会比初始标记阶段的时间稍长，远远比并发标记阶段时间短
- **并发清除：** 开启用户线程，同时 GC 线程开始对为标记的区域做清扫。

![CMS 垃圾收集器 ](../media/pictures/JVM.assets/82825079.jpg)

从它的名字就可以看出它是一款优秀的垃圾收集器，主要优点：**并发收集、低停顿**。但是它有下面三个明显的缺点：

- **对 CPU 资源敏感；**
- **无法处理浮动垃圾；**
- **它使用的回收算法-“标记-清除”算法会导致收集结束时会有大量空间碎片产生。**

#### 4.7 G1 收集器

**G1 (Garbage-First) 是一款面向服务器的垃圾收集器,主要针对配备多颗处理器及大容量内存的机器. 以极高概率满足 GC 停顿时间要求的同时,还具备高吞吐量性能特征.**

被视为 JDK1.7 中 HotSpot 虚拟机的一个重要进化特征。它具备一下特点：

- **并行与并发**：G1 能充分利用 CPU、多核环境下的硬件优势，使用多个 CPU（CPU 或者 CPU 核心）来缩短 Stop-The-World 停顿时间。部分其他收集器原本需要停顿 Java 线程执行的 GC 动作，G1 收集器仍然可以通过并发的方式让 java 程序继续执行。
- **分代收集**：虽然 G1 可以不需要其他收集器配合就能独立管理整个 GC 堆，但是还是保留了分代的概念。
- **空间整合**：与 CMS 的“标记--清理”算法不同，G1 从整体来看是基于“标记整理”算法实现的收集器；从局部上来看是基于“复制”算法实现的。
- **可预测的停顿**：这是 G1 相对于 CMS 的另一个大优势，降低停顿时间是 G1 和 CMS 共同的关注点，但 G1 除了追求低停顿外，还能建立可预测的停顿时间模型，能让使用者明确指定在一个长度为 M 毫秒的时间片段内。

G1 收集器的运作大致分为以下几个步骤：

- **初始标记**
- **并发标记**
- **最终标记**
- **筛选回收**

**G1 收集器在后台维护了一个优先列表，每次根据允许的收集时间，优先选择回收价值最大的 Region(这也就是它的名字 Garbage-First 的由来)**。这种使用 Region 划分内存空间以及有优先级的区域回收方式，保证了 G1 收集器在有限时间内可以尽可能高的收集效率（把内存化整为零）。



## 5.JVM内存区域

Java 虚拟机在执行 Java 程序的过程中会把它管理的内存划分成若干个不同的数据区域。JDK. 1.8 和之前的版本略有不同，下面会介绍到。

**JDK 1.8 之前：**

<div align="center">  
<img src="https://my-blog-to-use.oss-cn-beijing.aliyuncs.com/2019-3/JVM运行时数据区域.png" width="600px"/>
</div>

**JDK 1.8 ：**

<div align="center">  
<img src="https://my-blog-to-use.oss-cn-beijing.aliyuncs.com/2019-3Java运行时数据区域JDK1.8.png" width="600px"/>
</div>

**线程私有的：**

- 程序计数器
- 虚拟机栈
- 本地方法栈

**线程共享的：**

- 堆
- 方法区
- 直接内存 (非运行时数据区的一部分)



#### 2.1 程序计数器

程序计数器是一块较小的内存空间，可以看作是当前线程所执行的字节码的行号指示器。**字节码解释器工作时通过改变这个计数器的值来选取下一条需要执行的字节码指令，分支、循环、跳转、异常处理、线程恢复等功能都需要依赖这个计数器来完成。**

另外，**为了线程切换后能恢复到正确的执行位置，每条线程都需要有一个独立的程序计数器，各线程之间计数器互不影响，独立存储，我们称这类内存区域为“线程私有”的内存。**

**从上面的介绍中我们知道程序计数器主要有两个作用：**

1. 字节码解释器通过改变程序计数器来依次读取指令，从而实现代码的流程控制，如：顺序执行、选择、循环、异常处理。
2. 在多线程的情况下，程序计数器用于记录当前线程执行的位置，从而当线程被切换回来的时候能够知道该线程上次运行到哪儿了。

**注意：程序计数器是唯一一个不会出现 OutOfMemoryError 的内存区域，它的生命周期随着线程的创建而创建，随着线程的结束而死亡。**

#### 2.2 Java 虚拟机栈

**与程序计数器一样，Java 虚拟机栈也是线程私有的，它的生命周期和线程相同，描述的是 Java 方法执行的内存模型，每次方法调用的数据都是通过栈传递的。**

**Java 内存可以粗糙的区分为堆内存（Heap）和栈内存 (Stack),其中栈就是现在说的虚拟机栈，或者说是虚拟机栈中局部变量表部分。** （实际上，Java 虚拟机栈是由一个个栈帧组成，而每个栈帧中都拥有：局部变量表、操作数栈、动态链接、方法出口信息。）

**局部变量表主要存放了编译器可知的各种数据类型**（boolean、byte、char、short、int、float、long、double）、**对象引用**（reference 类型，它不同于对象本身，可能是一个指向对象起始地址的引用指针，也可能是指向一个代表对象的句柄或其他与此对象相关的位置）。

**Java 虚拟机栈会出现两种错误：StackOverFlowError 和 OutOfMemoryError。**

- **StackOverFlowError：** 若 Java 虚拟机栈的内存大小不允许动态扩展，那么当线程请求栈的深度超过当前 Java 虚拟机栈的最大深度的时候，就抛出 StackOverFlowError 错误。
- **OutOfMemoryError：** 若 Java 虚拟机栈的内存大小允许动态扩展，且当线程请求栈时内存用完了，无法再动态扩展了，此时抛出 OutOfMemoryError 错误。

Java 虚拟机栈也是线程私有的，每个线程都有各自的 Java 虚拟机栈，而且随着线程的创建而创建，随着线程的死亡而死亡。

**扩展：那么方法/函数如何调用？**

Java 栈可用类比数据结构中栈，Java 栈中保存的主要内容是栈帧，每一次函数调用都会有一个对应的栈帧被压入 Java 栈，每一个函数调用结束后，都会有一个栈帧被弹出。

Java 方法有两种返回方式：

1. return 语句。
2. 抛出异常。

不管哪种返回方式都会导致栈帧被弹出。



补充：

**内存溢出：**（out of memory）通俗理解就是内存不够，通常在运行大型软件或游戏时，软件或游戏所需要的内存远远超出了你主机内安装的内存所承受大小，就叫内存溢出。

**内存泄漏：**（Memory Leak）是指程序中己动态分配的堆内存由于某种原因程序未释放或无法释放，造成系统内存的浪费，导致程序运行速度减慢甚至系统崩溃等严重后果

参考：https://blog.csdn.net/weixin_42665577/article/details/81944939

#### 2.3 本地方法栈

和虚拟机栈所发挥的作用非常相似，区别是： **虚拟机栈为虚拟机执行 Java 方法 （也就是字节码）服务，而本地方法栈则为虚拟机使用到的 Native 方法服务。** 在 HotSpot 虚拟机中和 Java 虚拟机栈合二为一。

本地方法被执行的时候，在本地方法栈也会创建一个栈帧，用于存放该本地方法的局部变量表、操作数栈、动态链接、出口信息。

方法执行完毕后相应的栈帧也会出栈并释放内存空间，也会出现 StackOverFlowError 和 OutOfMemoryError 两种错误。

#### 2.4 堆

Java 虚拟机所管理的内存中最大的一块，Java 堆是所有线程共享的一块内存区域，在虚拟机启动时创建。**此内存区域的唯一目的就是存放对象实例，几乎所有的对象实例以及数组都在这里分配内存。**

Java 堆是垃圾收集器管理的主要区域，因此也被称作**GC 堆（Garbage Collected Heap）**.从垃圾回收的角度，由于现在收集器基本都采用分代垃圾收集算法，所以 Java 堆还可以细分为：新生代和老年代：再细致一点有：Eden 空间、From Survivor、To Survivor 空间等。**进一步划分的目的是更好地回收内存，或者更快地分配内存。**

在 JDK 7 版本及JDK 7 版本之前，堆内存被通常被分为下面三部分：

1. 新生代内存(Young Generation)

2. 老生代(Old Generation)

3. 永生代(Permanent Generation

   ![1591857285110](../media/pictures/JVM.assets/1591857285110.png)

JDK 8 版本之后方法区（HotSpot 的永久代）被彻底移除了（JDK1.7 就已经开始了），取而代之是元空间，元空间使用的是直接内存。

![1591857330808](../media/pictures/JVM.assets/1591857330808.png)

**上图所示的 Eden 区、两个 Survivor 区都属于新生代（为了区分，这两个 Survivor 区域按照顺序被命名为 from 和 to），中间一层属于老年代。**

大部分情况，对象都会首先在 Eden 区域分配，在一次新生代垃圾回收后，如果对象还存活，则会进入 s0 或者 s1，并且对象的年龄还会加  1(Eden 区->Survivor 区后对象的初始年龄变为 1)，当它的年龄增加到一定程度（默认为 15 岁），就会被晋升到老年代中。对象晋升到老年代的年龄阈值，可以通过参数 `-XX:MaxTenuringThreshold` 来设置。

> 修正（[issue552](https://github.com/Snailclimb/JavaGuide/issues/552)）：“Hotspot遍历所有对象时，按照年龄从小到大对其所占用的大小进行累积，当累积的某个年龄大小超过了survivor区的一半时，取这个年龄和MaxTenuringThreshold中更小的一个值，作为新的晋升年龄阈值”。
>
> **动态年龄计算的代码如下**
>
> ```c++
> uint ageTable::compute_tenuring_threshold(size_t survivor_capacity) {
> 	//survivor_capacity是survivor空间的大小
> size_t desired_survivor_size = (size_t)((((double) survivor_capacity)*TargetSurvivorRatio)/100);
> size_t total = 0;
> uint age = 1;
> while (age < table_size) {
> total += sizes[age];//sizes数组是每个年龄段对象大小
> if (total > desired_survivor_size) break;
> age++;
> }
> uint result = age < MaxTenuringThreshold ? age : MaxTenuringThreshold;
> 	...
> }
> 
> ```
>
> 

堆这里最容易出现的就是  OutOfMemoryError 错误，并且出现这种错误之后的表现形式还会有几种，比如：

1. **`OutOfMemoryError: GC Overhead Limit Exceeded`** ： 当JVM花太多时间执行垃圾回收并且只能回收很少的堆空间时，就会发生此错误。
2. **`java.lang.OutOfMemoryError: Java heap space`** :假如在创建新的对象时, 堆内存中的空间不足以存放新创建的对象, 就会引发`java.lang.OutOfMemoryError: Java heap space` 错误。(和本机物理内存无关，和你配置的内存大小有关！)
3. ......

#### 2.5 方法区

方法区与 Java 堆一样，是各个线程共享的内存区域，它用于**存储已被虚拟机加载的类信息、常量、静态变量、即时编译器编译后的代码等数据**。虽然 **Java 虚拟机规范把方法区描述为堆的一个逻辑部分**，但是它却有一个别名叫做 **Non-Heap（非堆）**，目的应该是与 Java 堆区分开来。

方法区也被称为永久代。很多人都会分不清方法区和永久代的关系，为此我也查阅了文献。

##### 2.5.1 方法区和永久代的关系

> 《Java 虚拟机规范》只是规定了有方法区这么个概念和它的作用，并没有规定如何去实现它。那么，在不同的 JVM 上方法区的实现肯定是不同的了。  **方法区和永久代的关系很像 Java 中接口和类的关系，类实现了接口，而永久代就是 HotSpot 虚拟机对虚拟机规范中方法区的一种实现方式。** 也就是说，永久代是 HotSpot 的概念，方法区是 Java 虚拟机规范中的定义，是一种规范，而永久代是一种实现，一个是标准一个是实现，其他的虚拟机实现并没有永久代这一说法。

##### 2.5.2 常用参数

JDK 1.8 之前永久代还没被彻底移除的时候通常通过下面这些参数来调节方法区大小

```java
-XX:PermSize=N //方法区 (永久代) 初始大小
-XX:MaxPermSize=N //方法区 (永久代) 最大大小,超过这个值将会抛出 OutOfMemoryError 异常:java.lang.OutOfMemoryError: PermGen
```

相对而言，垃圾收集行为在这个区域是比较少出现的，但并非数据进入方法区后就“永久存在”了。

JDK 1.8 的时候，方法区（HotSpot 的永久代）被彻底移除了（JDK1.7 就已经开始了），取而代之是元空间，元空间使用的是直接内存。

下面是一些常用参数：

```java
-XX:MetaspaceSize=N //设置 Metaspace 的初始（和最小大小）
-XX:MaxMetaspaceSize=N //设置 Metaspace 的最大大小
```

与永久代很大的不同就是，如果不指定大小的话，随着更多类的创建，虚拟机会耗尽所有可用的系统内存。

##### 2.5.3 为什么要将永久代 (PermGen) 替换为元空间 (MetaSpace) 呢?

1. 整个永久代有一个 JVM 本身设置固定大小上限，无法进行调整，而元空间使用的是直接内存，受本机可用内存的限制，虽然元空间仍旧可能溢出，但是比原来出现的几率会更小。

> 当你元空间溢出时会得到如下错误： `java.lang.OutOfMemoryError: MetaSpace`

你可以使用 `-XX：MaxMetaspaceSize` 标志设置最大元空间大小，默认值为 unlimited，这意味着它只受系统内存的限制。`-XX：MetaspaceSize` 调整标志定义元空间的初始大小如果未指定此标志，则 Metaspace 将根据运行时的应用程序需求动态地重新调整大小。  

2. 元空间里面存放的是类的元数据，这样加载多少类的元数据就不由 `MaxPermSize` 控制了, 而由系统的实际可用空间来控制，这样能加载的类就更多了。  
3. 在 JDK8，合并 HotSpot 和 JRockit 的代码时, JRockit 从来没有一个叫永久代的东西, 合并之后就没有必要额外的设置这么一个永久代的地方了。

#### 2.6 运行时常量池

运行时常量池是方法区的一部分。Class 文件中除了有类的版本、字段、方法、接口等描述信息外，还有常量池信息（用于存放编译期生成的各种字面量和符号引用）

既然运行时常量池是方法区的一部分，自然受到方法区内存的限制，当常量池无法再申请到内存时会抛出 OutOfMemoryError 错误。

**JDK1.7 及之后版本的 JVM 已经将运行时常量池从方法区中移了出来，在 Java 堆（Heap）中开辟了一块区域存放运行时常量池。** 

![](../media/pictures/JVM.assets/26038433.jpg)
——图片来源：https://blog.csdn.net/wangbiao007/article/details/78545189

#### 2.7 直接内存



**直接内存并不是虚拟机运行时数据区的一部分，也不是虚拟机规范中定义的内存区域，但是这部分内存也被频繁地使用。而且也可能导致 OutOfMemoryError 错误出现。**

JDK1.4 中新加入的 **NIO(New Input/Output) 类**，引入了一种基于**通道（Channel）** 与**缓存区（Buffer）** 的 I/O 方式，它可以直接使用 Native 函数库直接分配堆外内存，然后通过一个存储在 Java 堆中的 DirectByteBuffer 对象作为这块内存的引用进行操作。这样就能在一些场景中显著提高性能，因为**避免了在 Java 堆和 Native 堆之间来回复制数据**。

本机直接内存的分配不会受到 Java 堆的限制，但是，既然是内存就会受到本机总内存大小以及处理器寻址空间的限制。



## 6、JVM调优（GC调优）

首先列举常见参数：

| 参数名称                   | 含义                                                       | 默认值               | 说明                                                         |
| -------------------------- | ---------------------------------------------------------- | -------------------- | ------------------------------------------------------------ |
| -Xms                       | 初始堆大小                                                 | 物理内存的1/64(<1GB) | 默认(MinHeapFreeRatio参数可以调整)空余堆内存小于40%时，JVM就会增大堆直到-Xmx的最大限制. |
| -Xmx                       | 最大堆大小                                                 | 物理内存的1/4(<1GB)  | 默认(MaxHeapFreeRatio参数可以调整)空余堆内存大于70%时，JVM会减少堆直到 -Xms的最小限制 |
| -Xmn                       | 年轻代大小(1.4or lator)                                    |                      | 注意：此处的大小是（eden+ 2 survivor space).与jmap -heap中显示的New gen是不同的。整个堆大小=年轻代大小 + 老年代大小 + 持久代（永久代）大小.增大年轻代后,将会减小年老代大小.此值对系统性能影响较大,Sun官方推荐配置为整个堆的3/8 |
| -XX:NewSize                | 设置年轻代大小(for 1.3/1.4)                                |                      |                                                              |
| -XX:MaxNewSize             | 年轻代最大值(for 1.3/1.4)                                  |                      |                                                              |
| -XX:PermSize               | 设置持久代(perm gen)初始值                                 | 物理内存的1/64       |                                                              |
| -XX:MaxPermSize            | 设置持久代最大值                                           | 物理内存的1/4        |                                                              |
| -Xss                       | 每个线程的堆栈大小                                         |                      | JDK5.0以后每个线程堆栈大小为1M,以前每个线程堆栈大小为256K.更具应用的线程所需内存大小进行 调整.在相同物理内存下,减小这个值能生成更多的线程.但是操作系统对一个进程内的线程数还是有限制的,不能无限生成,经验值在3000~5000左右一般小的应用， 如果栈不是很深， 应该是128k够用的 大的应用建议使用256k。这个选项对性能影响比较大，需要严格的测试。（校长）和threadstacksize选项解释很类似,官方文档似乎没有解释,在论坛中有这样一句话:-Xss is translated in a VM flag named ThreadStackSize”一般设置这个值就可以了 |
| -XX:NewRatio               | 年轻代(包括Eden和两个Survivor区)与年老代的比值(除去持久代) |                      | -XX:NewRatio=4表示年轻代与年老代所占比值为1:4,年轻代占整个堆栈的1/5Xms=Xmx并且设置了Xmn的情况下，该参数不需要进行设置。 |
| -XX:SurvivorRatio          | Eden区与Survivor区的大小比值                               |                      | 设置为8,则两个Survivor区与一个Eden区的比值为2:8,一个Survivor区占整个年轻代的1/10 |
| -XX:+DisableExplicitGC     | 关闭System.gc()                                            |                      | 这个参数需要严格的测试                                       |
| -XX:PretenureSizeThreshold | 对象超过多大是直接在旧生代分配                             | 0                    | 单位字节 新生代采用Parallel ScavengeGC时无效另一种直接在旧生代分配的情况是大的数组对象,且数组中无外部引用对象. |
| -XX:ParallelGCThreads      | 并行收集器的线程数                                         |                      | 此值最好配置与处理器数目相等 同样适用于CMS                   |
| -XX:MaxGCPauseMillis       | 每次年轻代垃圾回收的最长时间(最大暂停时间)                 |                      | 如果无法满足此时间,JVM会自动调整年轻代大小,以满足此值.       |

其实还有一些打印及CMS方面的参数，这里就不以一一列举了

作者：说出你的愿望吧链接：https://juejin.im/post/5e1505d0f265da5d5d744050来源：掘金著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。

参考JavaGuide大白话带你认识JVM，讲的比较实际。

#### （1）调整最大堆内存和最小堆内存

-Xmx –Xms：指定java堆最大值（默认值是物理内存的1/4(<1GB)）和初始java堆最小值（默认值是物理内存的1/64(<1GB))

默认(MinHeapFreeRatio参数可以调整)空余堆内存小于40%时，JVM就会增大堆直到-Xmx的最大限制.，默认(MaxHeapFreeRatio参数可以调整)空余堆内存大于70%时，JVM会减少堆直到 -Xms的最小限制。简单点来说，你不停地往堆内存里面丢数据，等它剩余大小小于40%了，JVM就会动态申请内存空间不过会小于-Xmx，如果剩余大小大于70%，又会动态缩小不过不会小于–Xms。就这么简单

开发过程中，**通常会将 -Xms 与 -Xmx两个参数的配置相同的值**，其目的是为了能够在java垃圾回收机制清理完堆区后不需要重新分隔计算堆区的大小而浪费资源。

#### （2）调整Survivor区和Eden区的比值

最优解当然就是官方的Eden和Survivor的占比为8:1:1

#### （3）设置新生代，老年代大小

堆，有新生代和老年代组成，新生代有Eden，from，to区组成。

根据实际事情调整新生代和幸存代的大小，官方推荐**新生代占java堆的3/8**，**幸存代占新生代的1/10**。

#### （4）永久区的设置

```
-XX:PermSize -XX:MaxPermSize
```

初始空间（默认为物理内存的1/64）和最大空间（默认为物理内存的1/4）。也就是说，jvm启动时，永久区一开始就占用了PermSize大小的空间，如果空间还不够，可以继续扩展，但是不能超过MaxPermSize，否则会OOM。

tips：如果堆空间没有用完也抛出了OOM，有可能是永久区导致的。堆空间实际占用非常少，但是永久区溢出 一样抛出OOM。

#### （5）JVM的栈参数调优

##### 4.7.1 调整每个线程栈空间的大小

可以通过-Xss：调整每个线程栈空间的大小

JDK5.0以后每个线程堆栈大小为1M，以前每个线程堆栈大小为256K。在相同物理内存下,减小这个值能生成更多的线程。但是操作系统对一个进程内的线程数还是有限制的，不能无限生成，经验值在3000~5000左右

##### 4.7.2 设置线程栈的大小

```
-XXThreadStackSize：
    设置线程栈的大小(0 means use default stack size)
```



下面这个也是JavaGuide里面的：

#### GC 调优策略

**策略 1：**将新对象预留在新生代，由于 Full GC 的成本远高于 Minor GC，因此尽可能将对象分配在新生代是明智的做法，实际项目中根据 GC 日志分析新生代空间大小分配是否合理，适当通过“-Xmn”命令调节新生代大小，最大限度降低新对象直接进入老年代的情况。

**策略 2：**大对象进入老年代，虽然大部分情况下，将对象分配在新生代是合理的。但是对于大对象这种做法却值得商榷，大对象如果首次在新生代分配可能会出现空间不足导致很多年龄不够的小对象被分配的老年代，破坏新生代的对象结构，可能会出现频繁的 full gc。因此，对于大对象，可以设置直接进入老年代（当然短命的大对象对于垃圾回收来说简直就是噩梦）。`-XX:PretenureSizeThreshold` 可以设置直接进入老年代的对象大小。

**策略 3：**合理设置进入老年代对象的年龄，`-XX:MaxTenuringThreshold` 设置对象进入老年代的年龄大小，减少老年代的内存占用，降低 full gc 发生的频率。

**策略 4：**设置稳定的堆大小，堆大小设置有两个参数：`-Xms` 初始化堆大小，`-Xmx` 最大堆大小。

**策略5：**注意： 如果满足下面的指标，**则一般不需要进行 GC 优化：**

> MinorGC 执行时间不到50ms； Minor GC 执行不频繁，约10秒一次； Full GC 执行时间不到1s； Full GC 执行频率不算频繁，不低于10分钟1次。



参考：JavaGuide

参考：https://blog.csdn.net/weixin_42447959/article/details/81637909?depth_1-utm_source=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-1&utm_source=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-1（感觉这篇文章写得很好，但是好多不懂）

https://blog.csdn.net/Javazhoumou/article/details/99298624





## 7.新生代怎么变成年轻代

也叫新生代，顾名思义，主要是用来存放新生的对象。新生代又细分为 Eden区、SurvivorFrom区、SurvivorTo区。

新创建的对象都会被分配到Eden区(如果该对象占用内存非常大，则直接分配到老年代区), 当Eden区内存不够的时候就会触发MinorGC（Survivor满不会引发MinorGC，而是将对象移动到老年代中），

在Minor GC开始的时候，对象只会存在于Eden区和Survivor from区，Survivor to区是空的。

Minor GC操作后，Eden区如果仍然存活（判断的标准是被引用了，通过GC root进行可达性判断）的对象，将会被移到Survivor To区。而From区中，**对象在Survivor区中每熬过一次Minor GC，年龄就会+1岁，当年龄达到一定值(年龄阈值，可以通过-XX:MaxTenuringThreshold来设置，默认是15)的对象会被移动到年老代中**，否则对象会被复制到“To”区。经过这次GC后，Eden区和From区已经被清空。

“From”区和“To”区互换角色，原Survivor To成为下一次GC时的Survivor From区, 总之，GC后，都会保证Survivor To区是空的。

奇怪为什么有 From和To，2块区域？
这就要说到新生代Minor GC的算法了：复制算法

把内存区域分为两块，每次使用一块，GC的时候把一块中的内容移动到另一块中，原始内存中的对象就可以被回收了，

优点是避免内存碎片。

参考：https://blog.csdn.net/yujianping_123/article/details/99545138 （感觉写的挺好）



## 8.Java会出现内存泄漏吗，什么情况下会出现？

当然会出现内存泄露。

**内存溢出：**（out of memory）通俗理解就是内存不够，通常在运行大型软件或游戏时，软件或游戏所需要的内存远远超出了你主机内安装的内存所承受大小，就叫内存溢出。

**内存泄漏：**（Memory Leak）是指程序中己动态分配的堆内存由于某种原因程序未释放或无法释放，造成系统内存的浪费，导致程序运行速度减慢甚至系统崩溃等严重后果



内存泄露的根本原因：

Java内存泄漏的根本原因是什么呢？长生命周期的对象持有短生命周期对象的引用就很可能发生内存泄漏，尽管短生命周期对象已经不再需要，但是因为长生命周期持有它的引用而导致不能被回收，这就是Java中内存泄漏的发生场景。

![1586859813149](../media/pictures/JVM.assets/1586859813149.png)

总结起来就比如上图，A的生命周期很长，而B的生命周期很短，而A一直没有释放，导致B也一直没有被回收。



**以下情况会出现内存泄露**：

**1、静态集合类引起内存泄漏：** （如果static 修饰一个很大的对象，也可能会发生内存泄露）

像HashMap、Vector等的使用最容易出现内存泄露，这些静态变量的生命周期和应用程序一致，他们所引用的所有的对象Object也不能被释放，因为他们也将一直被Vector等引用着。

例如：

```java
Static Vector v = new Vector(10);
for (int i = 1; i<100; i++)
{
Object o = new Object();
v.add(o);
o = null;
}
```

在这个例子中，循环申请Object 对象，并将所申请的对象放入一个Vector 中，如果仅仅释放引用本身（o=null），那么Vector 仍然引用该对象，所以这个对象对GC 来说是不可回收的。因此，如果对象加入到Vector 后，还必须从Vector 中删除，最简单的方法就是将Vector对象设置为null。

**2、当集合里面的对象属性被修改后，再调用remove()方法时不起作用。**

例如：

```java
public static void main(String[] args)
{
Set<Person> set = new HashSet<Person>();
Person p1 = new Person("唐僧","pwd1",25);
Person p2 = new Person("孙悟空","pwd2",26);
Person p3 = new Person("猪八戒","pwd3",27);
set.add(p1);
set.add(p2);
set.add(p3);
System.out.println("总共有:"+set.size()+" 个元素!"); //结果：总共有:3 个元素!
p3.setAge(2); //修改p3的年龄,此时p3元素对应的hashcode值发生改变
set.remove(p3); //此时remove不掉，造成内存泄漏
set.add(p3); //重新添加，居然添加成功
System.out.println("总共有:"+set.size()+" 个元素!"); //结果：总共有:4 个元素!
for (Person person : set)
{
System.out.println(person);
}
}
```

**3、监听器**

在java 编程中，我们都需要和监听器打交道，通常一个应用当中会用到很多监听器，我们会调用一个控件的诸如addXXXListener()等方法来增加监听器，但往往在释放对象的时候却没有记住去删除这些监听器，从而增加了内存泄漏的机会。

**4、各种连接**

比如数据库连接（dataSourse.getConnection()），网络连接(socket)和io连接，除非其显式的调用了其close（）方法将其连接关闭，否则是不会自动被GC 回收的。对于Resultset 和Statement 对象可以不进行显式回收，但Connection 一定要显式回收，因为Connection 在任何时候都无法自动回收，而Connection一旦回收，Resultset 和Statement 对象就会立即为NULL。但是如果使用连接池，情况就不一样了，除了要显式地关闭连接，还必须显式地关闭Resultset Statement 对象（关闭其中一个，另外一个也会关闭），否则就会造成大量的Statement 对象无法释放，从而引起内存泄漏。这种情况下一般都会在try里面去的连接，在finally里面释放连接。

**5、内部类和外部模块的引用**

内部类的引用是比较容易遗忘的一种，而且一旦没释放可能导致一系列的后继类对象没有释放。此外程序员还要小心外部模块不经意的引用，例如程序员A 负责A 模块，调用了B 模块的一个方法如：

public void registerMsg(Object b);

这种调用就要非常小心了，传入了一个对象，很可能模块B就保持了对该对象的引用，这时候就需要注意模块B 是否提供相应的操作去除引用。

**6、单例模式**

不正确使用单例模式是引起内存泄漏的一个常见问题，单例对象在初始化后将在JVM的整个生命周期中存在（以静态变量的方式），如果单例对象持有外部的引用，那么这个对象将不能被JVM正常回收，导致内存泄漏，考虑下面的例子：

```java
class A{
public A(){
B.getInstance().setA(this);
}
....
}
//B类采用单例模式
class B{
private A a;
private static B instance=new B();
public B(){}
public static B getInstance(){
return instance;
}
public void setA(A a){
this.a=a;
}
//getter...
}
```

显然B采用singleton模式，它持有一个A对象的引用，而这个A类的对象将不能被回收。想象下如果A是个比较复杂的对象或者集合类型会发生什么情况



参考：https://blog.csdn.net/u012516166/article/details/77014910 



## 9.GC如何搜索到需要回收的对象？如何判断哪些对象需要回收？

![](../media/pictures/JVM.assets/11034259.jpg)

#### 引用计数法：

给对象中添加一个引用计数器，每当有一个地方引用它，计数器就加 1；当引用失效，计数器就减 1；任何时候计数器为 0 的对象就是不可能再被使用的。

**这个方法实现简单，效率高，但是目前主流的虚拟机中并没有选择这个算法来管理内存，其最主要的原因是它很难解决对象之间相互循环引用的问题。**

#### 可达性分析算法：（有些地方叫根搜索算法？）

这个算法的基本思想就是通过一系列的称为 **“GC Roots”** 的对象作为起点，从这些节点开始向下搜索，节点所走过的路径称为引用链，当一个对象到 GC Roots 没有任何引用链相连的话，则证明此对象是不可用的。

![可达性分析算法 ](../media/pictures/JVM.assets/72762049.jpg)



当然还有：参考JavaGuide JVM垃圾回收 对象已经死亡



## 10.JAVA内存区域 堆和栈上存储什么

堆内存是用来存放由new创建的对象实例和数组。**此内存区域的唯一目的就是存放对象实例，几乎所有的对象实例以及数组都在这里分配内存。**

栈内存中保存的是堆内存空间的访问地址，或者说栈中的变量指向堆内存中的变量（java中的“指针”）



**堆和栈队列定义和差别**

堆与栈的区别：

​            1.栈内存存储的是局部变量而堆内存存储的是实体；

​            2.栈内存的更新速度要快于堆内存，因为局部变量的生命周期很短；

​            3.栈内存存放的变量生命周期一旦结束就会被释放，而堆内存存放的实体会被垃圾回收机制不定时的回收。

堆和栈区别，参考：https://www.cnblogs.com/cchHers/p/10010275.html



## 11.GC Roots可以是哪些对象？

（能够直接从外部访问的对象）可以和Java内存区域合起来记忆。

虚拟机栈中的引用对象

方法区中的静态属性引用的对象

方法区中的常量引用的对象

本地方法栈中native方法的引用对象



## 12.JVM内存区域，哪些是线程私有？

虚拟机栈，本地方法栈，程序计数器。



## 13.Java 对象的创建过程（五步，建议能默写出来并且要知道每一步虚拟机做了什么）

下图便是 Java 对象的创建过程，我建议最好是能默写出来，并且要掌握每一步在做什么。

![1591857359349](../media/pictures/JVM.assets/1591857359349.png)

#### Step1:类加载检查

 虚拟机遇到一条 new 指令时，首先将去检查这个指令的参数是否能在常量池中定位到这个类的符号引用，并且检查这个符号引用代表的类是否已被加载过、解析和初始化过。如果没有，那必须先执行相应的类加载过程。

#### Step2:分配内存

在**类加载检查**通过后，接下来虚拟机将为新生对象**分配内存**。对象所需的内存大小在类加载完成后便可确定，为对象分配空间的任务等同于把一块确定大小的内存从 Java 堆中划分出来。**分配方式**有 **“指针碰撞”** 和 **“空闲列表”** 两种，**选择那种分配方式由 Java 堆是否规整决定，而 Java 堆是否规整又由所采用的垃圾收集器是否带有压缩整理功能决定**。

**内存分配的两种方式：（补充内容，需要掌握）**

选择以上两种方式中的哪一种，取决于 Java 堆内存是否规整。而 Java 堆内存是否规整，取决于 GC 收集器的算法是"标记-清除"，还是"标记-整理"（也称作"标记-压缩"），值得注意的是，复制算法内存也是规整的

![1591857371019](../media/pictures/JVM.assets/1591857371019.png)

**内存分配并发问题（补充内容，需要掌握）**

在创建对象的时候有一个很重要的问题，就是线程安全，因为在实际开发过程中，创建对象是很频繁的事情，作为虚拟机来说，必须要保证线程是安全的，通常来讲，虚拟机采用两种方式来保证线程安全：

- **CAS+失败重试：** CAS 是乐观锁的一种实现方式。所谓乐观锁就是，每次不加锁而是假设没有冲突而去完成某项操作，如果因为冲突失败就重试，直到成功为止。**虚拟机采用 CAS 配上失败重试的方式保证更新操作的原子性。**
- **TLAB：** 为每一个线程预先在 Eden 区分配一块儿内存，JVM 在给线程中的对象分配内存时，首先在 TLAB 分配，当对象大于 TLAB 中的剩余内存或 TLAB 的内存已用尽时，再采用上述的 CAS 进行内存分配

#### Step3:初始化零值

内存分配完成后，虚拟机需要将分配到的内存空间都初始化为零值（不包括对象头），这一步操作保证了对象的实例字段在 Java 代码中可以不赋初始值就直接使用，程序能访问到这些字段的数据类型所对应的零值。

#### Step4:设置对象头

初始化零值完成之后，**虚拟机要对对象进行必要的设置**，例如这个对象是那个类的实例、如何才能找到类的元数据信息、对象的哈希码、对象的 GC 分代年龄等信息。 **这些信息存放在对象头中。** 另外，根据虚拟机当前运行状态的不同，如是否启用偏向锁等，对象头会有不同的设置方式。

#### Step5:执行 init 方法

 在上面工作都完成之后，从虚拟机的视角来看，一个新的对象已经产生了，但从 Java 程序的视角来看，对象创建才刚开始，`<init>` 方法还没有执行，所有的字段都还为零。所以一般来说，执行 new 指令之后会接着执行 `<init>` 方法，把对象按照程序员的意愿进行初始化，这样一个真正可用的对象才算完全产生出来。



## 14.对象的访问定位：

句柄，直接指针。



## 15.String类和常量池

**String 对象的两种创建方式：**

```java
String str1 = "abcd";//先检查字符串常量池中有没有"abcd"，如果字符串常量池中没有，则创建一个，然后 str1 指向字符串常量池中的对象，如果有，则直接将 str1 指向"abcd""；
String str2 = new String("abcd");//堆中创建一个新的对象
String str3 = new String("abcd");//堆中创建一个新的对象
System.out.println(str1==str2);//false
System.out.println(str2==str3);//false
```

这两种不同的创建方法是有差别的。

- 第一种方式是在常量池中拿对象；
- 第二种方式是直接在堆内存空间创建一个新的对象。

记住一点：**只要使用 new 方法，便需要创建新的对象。**

再给大家一个图应该更容易理解，图片来源：<https://www.journaldev.com/797/what-is-java-string-pool>：

![String-Pool-Java](../media/pictures/JVM.assets/2019-3String-Pool-Java1-450x249.png)

**String 类型的常量池比较特殊。它的主要使用方法有两种：**

- 直接使用双引号声明出来的 String 对象会直接存储在常量池中。
- 如果不是用双引号声明的 String 对象，可以使用 String 提供的 intern 方法。String.intern() 是一个 Native 方法，它的作用是：如果运行时常量池中已经包含一个等于此 String 对象内容的字符串，则返回常量池中该字符串的引用；如果没有，JDK1.7之前（不包含1.7）的处理方式是在常量池中创建与此 String 内容相同的字符串，并返回常量池中创建的字符串的引用，JDK1.7以及之后的处理方式是在常量池中记录此字符串的引用，并返回该引用。

```java
	      String s1 = new String("计算机");
	      String s2 = s1.intern();
	      String s3 = "计算机";
	      System.out.println(s2);//计算机
	      System.out.println(s1 == s2);//false，因为一个是堆内存中的 String 对象一个是常量池中的 String 对象，
	      System.out.println(s3 == s2);//true，因为两个都是常量池中的 String 对象
```

**字符串拼接:**

```java
		  String str1 = "str";
		  String str2 = "ing";
		 
		  String str3 = "str" + "ing";//常量池中的对象
		  String str4 = str1 + str2; //在堆上创建的新的对象	  
		  String str5 = "string";//常量池中的对象
		  System.out.println(str3 == str4);//false
		  System.out.println(str3 == str5);//true
		  System.out.println(str4 == str5);//false
```

![1591857393445](../media/pictures/JVM.assets/1591857393445.png)

尽量避免多个字符串拼接，因为这样会重新创建对象。如果需要改变字符串的话，可以使用 StringBuilder 或者 StringBuffer。

#### String s1 = new String("abc");这句话创建了几个字符串对象？

**将创建 1 或 2 个字符串。如果池中已存在字符串常量“abc”，则只会在堆空间创建一个字符串常量“abc”。如果池中没有字符串常量“abc”，那么它将首先在池中创建，然后在堆空间中创建，因此将创建总共 2 个字符串对象。**

**验证：**

```java
		String s1 = new String("abc");// 堆内存的地址值
		String s2 = "abc";
		System.out.println(s1 == s2);// 输出 false,因为一个是堆内存，一个是常量池的内存，故两者是不同的。
		System.out.println(s1.equals(s2));// 输出 true
```

**结果：**

```
false
true
```



## 16.如何实现小内存，排序大文件？只用2GB内存在20亿个整数中找到出现次数最多的数？

用哈希函数分小，再找。

参考：https://blog.csdn.net/u010456903/article/details/48805985?depth_1-utm_source=distribute.pc_relevant.none-task-blog-BlogCommendFromBaidu-3&utm_source=distribute.pc_relevant.none-task-blog-BlogCommendFromBaidu-3

分成多个满足内存限制的小文件，再操作。



## 17.如何设置jvm创建的堆内存的大小？

-Xmx：最大堆大小

-Xms：初始堆大小（有的地方也叫最小堆内存）



## 18.八种基本类型包装类

**Java 基本类型的包装类的大部分都实现了常量池技术，即 Byte,Short,Integer,Long,Character,Boolean；前面 4 种包装类默认创建了数值[-128，127] 的相应类型的缓存数据，Character创建了数值在[0,127]范围的缓存数据，Boolean 直接返回True Or False。如果超出对应范围仍然会去创建新的对象。** 为啥把缓存设置为[-128，127]区间？（[参见issue/461](https://github.com/Snailclimb/JavaGuide/issues/461)）性能和资源之间的权衡。

```java
public static Boolean valueOf(boolean b) {
    return (b ? TRUE : FALSE);
}
```

```java
private static class CharacterCache {         
    private CharacterCache(){}
          
    static final Character cache[] = new Character[127 + 1];          
    static {             
        for (int i = 0; i < cache.length; i++)                 
            cache[i] = new Character((char)i);         
    }   
}

```

两种浮点数类型的包装类 Float,Double 并没有实现常量池技术。**

```java
		Integer i1 = 33;
		Integer i2 = 33;
		System.out.println(i1 == i2);// 输出 true
		Integer i11 = 333;
		Integer i22 = 333;
		System.out.println(i11 == i22);// 输出 false
		Double i3 = 1.2;
		Double i4 = 1.2;
		System.out.println(i3 == i4);// 输出 false

```

**Integer 缓存源代码：** 

```java
/**
*此方法将始终缓存-128 到 127（包括端点）范围内的值，并可以缓存此范围之外的其他值。
*/
    public static Integer valueOf(int i) {
        if (i >= IntegerCache.low && i <= IntegerCache.high)
            return IntegerCache.cache[i + (-IntegerCache.low)];
        return new Integer(i);
    }


```

**应用场景：**

1. Integer i1=40；Java 在编译的时候会直接将代码封装成 Integer i1=Integer.valueOf(40);，从而使用常量池中的对象。
2. Integer i1 = new Integer(40);这种情况下会创建新的对象。

```java
  Integer i1 = 40;
  Integer i2 = new Integer(40);
  System.out.println(i1==i2);//输出 false

```

**Integer 比较更丰富的一个例子:**

```java
  Integer i1 = 40;
  Integer i2 = 40;
  Integer i3 = 0;
  Integer i4 = new Integer(40);
  Integer i5 = new Integer(40);
  Integer i6 = new Integer(0);
  
  System.out.println("i1=i2   " + (i1 == i2));
  System.out.println("i1=i2+i3   " + (i1 == i2 + i3));
  System.out.println("i1=i4   " + (i1 == i4));
  System.out.println("i4=i5   " + (i4 == i5));
  System.out.println("i4=i5+i6   " + (i4 == i5 + i6));   
  System.out.println("40=i5+i6   " + (40 == i5 + i6));     

```

结果：

```
i1=i2   true
i1=i2+i3   true
i1=i4   false
i4=i5   false
i4=i5+i6   true
40=i5+i6   true

```

解释：

语句 i4 == i5 + i6，因为+这个操作符不适用于 Integer 对象，首先 i5 和 i6 进行自动拆箱操作，进行数值相加，即 i4 == 40。然后 Integer 对象无法与数值进行直接比较，所以 i4 自动拆箱转为 int 值 40，最终这条语句转为 40 == 40 进行数值比较。



## 19.堆 （垃圾回收器管理的主要区域）

Java 虚拟机所管理的内存中最大的一块，Java 堆是所有线程共享的一块内存区域，在虚拟机启动时创建。**此内存区域的唯一目的就是存放对象实例，几乎所有的对象实例以及数组都在这里分配内存。**

Java 堆是垃圾收集器管理的主要区域，因此也被称作**GC 堆（Garbage Collected Heap）**.从垃圾回收的角度，由于现在收集器基本都采用分代垃圾收集算法，所以 Java 堆还可以细分为：新生代和老年代：再细致一点有：Eden 空间、From Survivor、To Survivor 空间等。**进一步划分的目的是更好地回收内存，或者更快地分配内存。**

在 JDK 7 版本及JDK 7 版本之前，堆内存被通常被分为下面三部分：

1. 新生代内存(Young Generation)
2. 老生代(Old Generation)
3. 永生代(Permanent Generation)

![1591857411492](../media/pictures/JVM.assets/1591857411492.png)

JDK 8 版本之后方法区（HotSpot 的永久代）被彻底移除了（JDK1.7 就已经开始了），取而代之是元空间，元空间使用的是直接内存。

![1591857426096](../media/pictures/JVM.assets/1591857426096.png)

**上图所示的 Eden 区、两个 Survivor 区都属于新生代（为了区分，这两个 Survivor 区域按照顺序被命名为 from 和 to），中间一层属于老年代。**



## 19.商用垃圾回收算法中的分代收集算法中如何确定一个对象属于新生代还是老年代？

#### 对象优先在 eden 区分配

目前主流的垃圾收集器都会采用分代回收算法，因此需要将堆内存分为新生代和老年代，这样我们就可以根据各个年代的特点选择合适的垃圾收集算法。

大多数情况下，对象在新生代中 eden 区分配。当 eden 区没有足够空间进行分配时，虚拟机将发起一次 Minor GC.下面我们来进行实际测试以下。

在测试之前我们先来看看 **Minor GC 和 Full GC 有什么不同呢？**

- **新生代 GC（Minor GC）**:指发生新生代的的垃圾收集动作，Minor GC 非常频繁，回收速度一般也比较快。
- **老年代 GC（Major GC/Full GC）**:指发生在老年代的 GC，出现了 Major GC 经常会伴随至少一次的 Minor GC（并非绝对），Major GC 的速度一般会比 Minor GC 的慢 10 倍以上。

#### 大对象直接进入老年代

大对象就是需要大量连续内存空间的对象（比如：字符串、数组）。

**为什么要这样呢？**

为了避免为大对象分配内存时由于分配担保机制带来的复制而降低效率。

#### 长期存活的对象将进入老年代

既然虚拟机采用了分代收集的思想来管理内存，那么内存回收时就必须能识别哪些对象应放在新生代，哪些对象应放在老年代中。为了做到这一点，虚拟机给每个对象一个对象年龄（Age）计数器。

如果对象在 Eden 出生并经过第一次 Minor GC 后仍然能够存活，并且能被 Survivor 容纳的话，将被移动到 Survivor 空间中，并将对象年龄设为 1.对象在 Survivor 中每熬过一次 MinorGC,年龄就增加 1 岁，当它的年龄增加到一定程度（默认为 15 岁），就会被晋升到老年代中。对象晋升到老年代的年龄阈值，可以通过参数 `-XX:MaxTenuringThreshold` 来设置。

#### 动态对象年龄判定

大部分情况，对象都会首先在 Eden 区域分配，在一次新生代垃圾回收后，如果对象还存活，则会进入 s0 或者 s1，并且对象的年龄还会加  1(Eden 区->Survivor 区后对象的初始年龄变为 1)，当它的年龄增加到一定程度（默认为 15 岁），就会被晋升到老年代中。对象晋升到老年代的年龄阈值，可以通过参数 `-XX:MaxTenuringThreshold` 来设置。



个人博客写过这个。（是参考一篇博客写的）



## 20.描述一下JVM加载class文件的原理机制



## 21.String对象可以改变吗，为什么？



牛客20201221选择题

错误的是（C）

```java
A 程序计数器是一个比较小的内存区域，用于指示当前线程所执行的字节码执行到了第几行，是线程隔离的
B 虚拟机栈描述的是Java方法执行的内存模型，用于存储局部变量，操作数栈，动态链接，方法出口等信息，是线程隔离的
C 方法区用于存储JVM加载的类信息、常量、静态变量、以及编译器编译后的代码等数据，是线程隔离的
D 原则上讲，所有的对象都在堆区上分配内存，是线程之间共享的
```

解释：方法区和堆，都是线程共享的。

大多数 JVM 将内存区域划分为 **Method Area（Non-Heap）（方法区）** ,**Heap（堆）** , **Program Counter Register（程序计数器）** ,  **VM Stack（虚拟机栈，也有翻译成JAVA 方法栈的）,Native Method Stack** （ **本地方法栈** ），其中**Method Area** 和 **Heap** 是线程共享的 **VM Stack，Native Method Stack 和\*\*Program Counter Register** 是非线程共享的。为什么分为 线程共享和非线程共享的呢?请继续往下看。

首先我们熟悉一下一个一般性的 Java 程序的工作过程。一个 Java 源程序文件，会被编译为字节码文件（以 class 为扩展名），每个java程序都需要运行在自己的JVM上，然后告知 JVM 程序的运行入口，再被 JVM 通过字节码解释器加载运行。那么程序开始运行后，都是如何涉及到各内存区域的呢？

概括地说来，JVM初始运行的时候都会分配好 **Method Area（方法区）** 和**Heap（堆）** ，而JVM 每遇到一个线程，就为其分配一个 **Program Counter Register（程序计数器）** ,  **VM Stack（虚拟机栈）和Native Method Stack （本地方法栈），** 当线程终止时，三者（虚拟机栈，本地方法栈和程序计数器）所占用的内存空间也会被释放掉。这也是为什么我把内存区域分为线程共享和非线程共享的原因，非线程共享的那三个区域的生命周期与所属线程相同，而线程共享的区域与JAVA程序运行的生命周期相同，所以这也是系统垃圾回收的场所只发生在线程共享的区域（实际上对大部分虚拟机来说知发生在Heap上）的原因。



## 22.类加载相关

### **类加载的过程**

类从被加载到JVM中开始，到卸载为止，整个生命周期包括：加载、验证、准备、解析、初始化、使用和卸载七个阶段。其中**类加载过程包括加载、验证、准备、解析和初始化五个阶段**。

![img](../media/pictures/JVM.assets/105211344671.png)

\4. 从类加载到方法执行的执行顺序？

\5. 为什么类对象能避免重复加载？(双亲委派)加载类对象是先从自己的类中进行加载，还是从父类中进行加载？(父类)

解释java内存中栈（stack），堆（heap）及方法区（method area）的用法？



# 网上面试题

参考：https://www.jianshu.com/p/573b5c6b8e89

https://blog.csdn.net/qq_41701956/article/details/100074023

