# Spring

# Interview

## Spring相关

### 1.什么叫面向对象编程，什么叫面向切面编程？(被三次问到什么叫面向对象)

#### 面向对象：

很早很早以前的编程是面向过程的，比如实现一个算术运算1+1 = 2，通过这个简单的算法就可以解决问题。但是随着时代的进步，人们不满足现有的算法了，因为问题越来越复杂，不是1+1那么单纯了，比如一个班级的学生的数据分析，这样就有了对象这个概念，一切事物皆对象。将现实的事物抽象出来，注意抽象这个词是重点啊，把现实生活的事物以及关系，抽象成类，通过继承，实现，组合的方式把万事万物都给容纳了。实现了对现实世界的抽象和数学建模。这是一次飞跃性的进步。

举个最简单点的例子来区分 面向过程和面向对象

有一天你想吃鱼香肉丝了，怎么办呢？你有两个选择

1、自己买材料，肉，鱼香肉丝调料，蒜苔，胡萝卜等等然后切菜切肉，开炒，盛到盘子里。

2、去饭店，张开嘴：老板！来一份鱼香肉丝！

##### 优缺点：

面向过程：

优点：性能比面向对象好，因为类调用时需要实例化，开销比较大，比较消耗资源。
缺点：不易维护、不易复用、不易扩展.

面向对象：

优点：易维护、易复用、易扩展，由于面向对象有封装、继承、多态性的特性，可以设计出低耦合的系统，使系统 更加灵活、更加易于维护 .
缺点：性能比面向过程差



##### 面向对象的三大特性：

1、封装
 隐藏对象的属性和实现细节，仅对外提供公共访问方式，将变化隔离，便于使用，提高复用性和安全性。
 2、继承
 提高代码复用性；继承是多态的前提。
 3、多态
 父类或接口定义的引用变量可以指向子类或具体实现类的实例对象。提高了程序的拓展性。



##### 五大基本原则：

1、单一职责原则SRP(Single Responsibility Principle)
 类的功能要单一，不能包罗万象，跟杂货铺似的。
 2、开放封闭原则OCP(Open－Close Principle)
 一个模块对于拓展是开放的，对于修改是封闭的，想要增加功能热烈欢迎，想要修改，哼，一万个不乐意。
 3、里式替换原则LSP(the Liskov Substitution Principle LSP)
 子类可以替换父类出现在父类能够出现的任何地方。比如你能代表你爸去你姥姥家干活。哈哈~~
 4、依赖倒置原则DIP(the Dependency Inversion Principle DIP)
 高层次的模块不应该依赖于低层次的模块，他们都应该依赖于抽象。抽象不应该依赖于具体实现，具体实现应该依赖于抽象。就是你出国要说你是中国人，而不能说你是哪个村子的。比如说中国人是抽象的，下面有具体的xx省，xx市，xx县。你要依赖的是抽象的中国人，而不是你是xx村的。
 5、接口分离原则ISP(the Interface Segregation Principle ISP)
 设计时采用多个与特定客户类有关的接口比采用一个通用的接口要好。就比如一个手机拥有打电话，看视频，玩游戏等功能，把这几个功能拆分成不同的接口，比在一个接口里要好的多

面向对象参考：https://www.jianshu.com/p/7a5b0043b035



### 2.IOC相关 

#### 1.IOC是什么？它的使用场景是什么？

AOP它利用一种称为“横切”的技术，剖解开封装的对象内部，并将那些影响了多个类的公共行为封装到一个可重用模块，并将其名为“Aspect”，即方面。所谓“方面”，简单地说，就是将那些与业务无关，却为业务模块所共同调用的逻辑或责任封装起来，便于减少系统的重复代码，降低模块间的耦合度，并有利于未来的可操作性和可维护性。AOP代表的是一个横向的关系，如果说“对象”是一个空心的圆柱体，其中封装的是对象的属性和行为；那么面向方面编程的方法，就仿佛一把利刃，将这些空心圆柱体剖开，以获得其内部的消息。而剖开的切面，也就是所谓的“方面”了。然后它又以巧夺天功的妙手将这些剖开的切面复原，不留痕迹。

使用“横切”技术，AOP把软件系统分为两个部分：核心关注点和横切关注点。业务处理的主要流程是核心关注点，与之关系不大的部分是横切关注点。横切关注点的一个特点是，他们经常发生在核心关注点的多处，而各处都基本相似。比**如权限认证、日志、事务处理**。Aop 的作用在于分离系统中的各种关注点，将核心关注点和横切关注点分离开来。AOP的核心思想就是“将应用程序中的商业逻辑同对其提供支持的通用服务进行分离。”

实现AOP的技术，主要分为两大类：一是采用动态代理技术，利用截取消息的方式，对该消息进行装饰，以取代原有对象行为的执行；二是采用静态织入的方式，引入特定的语法创建“方面”，从而使得编译器可以在编译期间织入有关“方面”的代码。



#### Spring的IOC是为了解决什么问题？对IOC的理解。

https://www.cnblogs.com/superjt/p/4311577.html

对IOC的和依赖注入理解，参考知乎 https://www.zhihu.com/question/23277575/answer/169698662



#### Spring IOC用了什么技术？

反射   https://www.cnblogs.com/superjt/p/4311577.html



#### Spring IOC里面怎样使用bean？

https://www.cnblogs.com/xiaoma000deblog/p/10288371.html

参考：https://www.cnblogs.com/micrari/p/7354650.html



#### Spring的IOC和DI的区别？

IOC控制反转  DI依赖注入



#### 如果让你实现一个IOC，需要怎么实现？

https://juejin.cn/post/6844903585830944782

代码在 JavaHub/Spring/simpleioc

#### IOC容器的顶层接口是什么？怎么启用一个IOC容器？

Spring 源码分析 ：https://github.com/seaswalker/spring-analysis





https://happysnail.cn/#/

http://happysnail.cn/#/?id=javainterview	

### 3.AOP相关

#### 1.AOP是什么 它的使用场景是什么？

#### 2.说一下AOP的面向切面？为什么AOP里有切点，为什么叫面向切面不叫面向切点？

#### 3.Aop 怎么使用 什么使用场景 日志?

#### 4..AOP用来封装横切关注点，具体可以在下面的场景中使用

- Authentication 权限

- Caching 缓存

- Context passing 内容传递

- Error handling 错误处理

- Debugging　　调试

- logging, tracing, profiling and monitoring　记录跟踪　优化　校准

- Performance optimization　性能优化

- Persistence　　持久化

- Resource pooling　资源池

- Synchronization　同步

- Transactions 事务

#### 5.设计一个代码，通过AOP实现禁止一个用户在60秒内搜索相同的内容

#### Aop是啥？是怎样实现的？实现原理？

#### 介绍一下Spring AOP？AOP是怎样做到的？

#### 说一下AOP底层是如何实现的 ？

#### 具体在你的这个项目(影院)中是如何使用AOP的？



### 4.说一下自己对IOC和AOP的理解？



### 5.Spring的IOC，DI和AOP的原理了解吗？原理？



### 7.注解相关 (重点)

#### 除了用@Autowerid注入实例还可以用什么注解？  看一下注解驱动，总结一下其他常用的注解

#### @autowired的用法？--回答放在成员变量上，但好像说这样子是会报错

#### Spring常用注解的异同？



### 8.Spring都有哪些模块



### 9.事务相关

#### 事务的传播形式？



#### 平常spring的事务是怎么控制的？如果外层事务和内层事务 内存事务回滚 外层事务怎么回滚 或者说不回滚 ？这个事务性消息是怎么操作的？



#### Spring里A方法调用B方法有用到事务吗？为什么（假设A方法开了事务，B方法上也有事务注解）



#### Spring的事务你是怎么样理解的，Spring的事务是怎样传播的，默认的传播行为是哪一个？

#### Spring事务传播机制？

#### 写出spring事务隔离级别与传播行为？

#### 对事务了解多吗？能不能介绍一下你们项目中哪里用到了事务？怎么开启事务？事务隔离级别还记得吗？

#### Spring事务的注解都有哪些

#### Spring中事务的隔离级别



### 10.Spring思想相关、整体情况

#### Spring的核心思想是什么？

#### 说一下你对Spring的理解？ 

#### Spring原理（我答的控制反转，依赖注入 问还有呢？ 回答AOP?

#### Spring全家桶的实现过程？





### 11.Spring容器

（bean factory）

#### spring 容器创建的过程中做了几件事





### 12.SpringBean相关

#### Spring Bean的生命周期，默认的scope是什么，为什么？

#### spring的scope 以及是否创建

#### Spring Bean的scope（单例 多例）？

#### @Bean的使用场景一般在哪？

#### spring的scope 以及是否创建

#### Spring的bean不用注解 如何取出bean



### 13.拦截器过滤器

#### 说一下Filter和Interceptor的区别？JDK动态代理和Cglib动态代理的区别？

#### Spring的filter和interceptor，interceptor里有哪些方法



### 14.Spring日志

#### 你常用的日志框架是什么，Spring自带的日志框架是哪个？



### 15.Spring代理(归属IOC)

####  Java的动态代理和Spring的CGLIB动态代理底层有什么区别，Spring框架用的是哪一种，为什么？

#### 动态代理有什么局限性，什么时候动态代理会失效？

#### private的方法可以使用Spring的动态代理吗？



### 16.Spring 设计模式

#### 单例是在容器初始化的时候创建 而多例是在getBean()的时候创建？

#### singleton单例：每一次取出来是同一个，并且只在加载的时候实例化一次

#### prototype：加载appllicationContext时没有实例化，在每一次getBean的时候都实例化一个新的bean



### 17.Spring 定时任务

#### Spring的定时任务用过吗？(@Schedule注解)？



### 18.HandleAdapter的作用



### 19.开放性问题

#### pagehelp这类似的插件，如果叫你写一个这个插件知道怎们写吗



### 20.逆向工程是用的插件还是自己写的？

### 21.BeanFactory 和factoryBean的差别



## SpringMVC相关

这里可以参考 3y电子书

### SpringMVC中如何返回一个JSON对象？

### SpringMvc  的架构图？

### SpringMvc 的大概工作流程？具体工作流程？

### SpringMvc的底层原理？

### 说一说你对SpringMVC的理解？

### 以及SpringMVC的运行过程？

### 上次面试碰到一个dispatchServlet 什么的？看看

### SpringMVC常用获取传递参数的方法？

### SpringMVC常用的注解有哪些？

### SpringMVC的架构

### springmvc的设计模式



### 你们的项目MVC层是怎么分的？Dao层？事务你们是在哪层做的？（报装项目好像在service层做）

### SpringMVC的核心是什么？请求的流程？控制反转怎么实现？

### Spring MVC和Struts的区别















## Spring SpringBoot SpringCloud SpringMVC 之间的关系和区别相关



### 1.Spring和SpringBoot的区别，有能力的可以看看SpringBoot的一键启动的源码，到时候也可以吹吹

1) Spring Boot可以建立独立的Spring应用程序；
2) 内嵌了如Tomcat，Jetty和Undertow这样的容器，也就是说可以直接跑起来，用不着再做部署工作了。
3) 无需再像Spring那样搞一堆繁琐的xml文件的配置；
4) 可以自动配置Spring；
5) 提供了一些现有的功能，如量度工具，表单数据验证以及一些外部配置这样的一些第三方功能；
6) 提供的POM可以简化Maven的配置；

本篇，讲述了Spring boot和Spring之间的区别。用一句话概况就是：**Spring Boot只是Spring本身的扩展，使开发，测试和部署更加方便。**



比较详细的参考：https://mp.weixin.qq.com/s/0qk2kaCKLdAViVzsw401sg



### 2.spring 与 springmvc的区别

### 3.springMVC和springboot有什么区别，springboot和springmvc相比最大的优点是什么？

### 4.spring和springboot的区别？

### 5.spring,springmvc,springboot区别？



















































































