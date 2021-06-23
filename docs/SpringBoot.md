# SpringBoot



> 参考Guide哥文章：https://snailclimb.gitee.io/springboot-guide

# Interview



## SpringBoot相关

### 1.介绍一下SpringBoot，为什么会有它的出现？解决了什么问题？他的出现有什么优点或者缺点？



### 1.SpringBoot注解相关

#### SpringBoot的常用注解有哪些？

虽然在平时开发中用到的注解有很多，但是这里问的是SpringBoot中的注解，范围相对来说很小：

参考知乎（SpringBoot中的注解）：https://zhuanlan.zhihu.com/p/57689422

其他Spring常用的注解（参考JavaGuide注解文章）：https://snailclimb.gitee.io/javaguide/#/



#### Spring boot controller 层的常用注解

@RequsMapping (@GetMapping @PostMapping) @Responsebody @RestController @Reference @RequestBody



#### SpringBoot最核心，最特性的注解是什么（启动项中的）？

**@SpringBootApplication**  最主要的一个注解

这是 Spring Boot 最最最核心的注解，用在 Spring Boot 主类上，标识这是一个 Spring Boot 应用，用来开启 Spring Boot 的各项能力。

其实这个注解就是 `@SpringBootConfiguration`、`@EnableAutoConfiguration`、`@ComponentScan` 这三个注解的组合，也可以用这三个注解来代替 `@SpringBootApplication` 注解。



**2、@EnableAutoConfiguration**

允许 Spring Boot 自动配置注解，开启这个注解之后，Spring Boot 就能根据当前类路径下的包或者类来配置 Spring Bean。

如：当前类路径下有 Mybatis 这个 JAR 包，`MybatisAutoConfiguration` 注解就能根据相关参数来配置 Mybatis 的各个 Spring Bean。



**4、@SpringBootConfiguration**

这个注解就是 @Configuration 注解的变体，只是用来修饰是 Spring Boot 配置而已，或者可利于 Spring Boot 后续的扩展。



**5、@ComponentScan**

这是 Spring 3.1 添加的一个注解，用来代替配置文件中的 component-scan 配置，开启组件扫描，即自动扫描包路径下的 @Component 注解进行注册 bean 实例到 context 中。

参考知乎：https://zhuanlan.zhihu.com/p/57689422



### 2.SpringBoot启动相关

#### Springboot的启动流程？

下面这是常见的SpringBoot启动的主函数：

```java
@SpringBootApplication
public class SpringBootWebApplication {
    public static void main(String[] args) {
        SpringApplication application = new SpringApplication(SpringBootWebApplication.class);
        application.run(args);
    }
}
```

要回答SpringBoot启动流程主要从两个方面来回答：

- @SpringBootApplication注解
- application.run()

这两部分分别是SpringBoot**加载配置**和**启动**。



1、首先介绍@SpringBootApplication 背后是三个注解，每个注解的意义，干了什么事情，参考上面的内容。

2、然后介绍application.run() ，这个实例化过程干了三件事：

- 根据classpath下是否存在(ConfigurableWebApplicationContext)判断是否要启动一个web applicationContext。
- SpringFactoriesInstances加载classpath下所有可用的ApplicationContextInitializer
- SpringFactoriesInstances加载classpath下所有可用的ApplicationListener



然后还有一些后续收尾工作。

参考思否：https://segmentfault.com/a/1190000014525138



### 3.SpringBoot对微服务的支持

SpringCloud是一套微服务的解决方案，SpringCloud底层是SpringBoot。

而SpringCloud是一套微服务解决方案，里面包含大多数微服务用到的各个组件。



### 4.SpringBoot框架的优势、特点

#### 特点

首先要知道什么是SpringBoot，先要知道Spring，Spring在做Java EE开发需要写大量的XML，代码非常多，非常繁杂。

SpringBoot有一句话叫约定大于配置，好多都是不需要自己配置（人家提前配置好了）。

Spring Boot 并不是用来替代 Spring 的解决方案，而是和 Spring 框架紧密结合用于提升 Spring 开发者体验的工具。同时它集成了大量常用的第三方库配置，Spring Boot应用中这些第三方库几乎可以是零配置的开箱即用（out-of-the-box），大部分的 Spring Boot 应用都只需要非常少量的配置代码（基于 Java 的配置），开发者能够更加专注于业务逻辑。



#### 优点

- **有良好的基因**    基于Spring 4.0   
- **简化编码**  引入的依赖少了很多（本质上是对配置依赖包进行了合并）
- **简化配置**  不需要像Spring一样，使用前配置各种xml，SpringBoot更多的是Java Config类型的配置方式，使用@Configuration 和@Bean两个注解即可
- **简化部署**   以前Spring发布，需要将war包放在Tomcat中，然后启动Tomcat才可以访问项目，现在SpringBoot内部有Tomcat，只需要将项目打包成jar包，然后java -jar **.jar  就可以运行项目。
- **简化监控**    我们可以引入 spring-boot-start-actuator 依赖，直接使用 REST 方式来获取进程的运行期性能参数，从而达到监控的目的，比较方便。但是 Spring Boot 只是个微框架，没有提供相应的服务发现与注册的配套功能，没有外围监控集成方案，没有外围安全管理方案，所以在微服务架构中，还需要 Spring Cloud 来配合一起使用。



参考：https://blog.csdn.net/qq_32595453/article/details/81141643



#### 修改SpringBoot内置Tomcat的配置

常用几种方式：

- 改配置

  ```yml
  #Tomcat
  server:
    tomcat:
      uri-encoding: UTF-8
      #最小线程数
      min-spare-threads: 500
      #最大线程数
      max-threads: 2500
      #最大链接数
      max-connections: 6500
      #最大等待队列长度
      accept-count: 1000
      #请求头最大长度kb
      max-http-header-size: 1048576
      #请请求体最大长度kb
      #max-http-post-size: 2097152
    #服务http端口
    port: 8080
    #链接建立超时时间
    connection-timeout: 12000
    servlet:
      #访问根路径
      context-path: /song
  ```

  

- 自定义类修改

  - 参考下面CSDN文章。



参考博客园：https://www.cnblogs.com/lys_013/p/13185940.html

参考CSDN：https://blog.csdn.net/keitho00/article/details/85003430?utm_medium=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-1.control&dist_request_id=11289e51-22e1-48fb-ac44-4d80664dcc46&depth_1-utm_source=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-1.control 

### 5.SpringBoot和其他的对比

#### Springboot和SSM的最大区别是什么？

简而言之就是：

SSM （Spring + Spring MVC + Mybatis）里面包含的三种是固定的，而SpringBoot，他没有和这些框架集成，不一定非要使用这三种框架，你可以使用其他的框架，相当于你需要自己去组装里面的东西。



参考知乎：https://www.zhihu.com/question/284488830



#### SpringBoot如何实现自动化配置

参考博客园：https://www.cnblogs.com/bluemilk/p/10569720.html  （这个没有细看，没有看懂）



#### Spring和springboot区别





### 6.SpringBoot处理事务、连接池

Springboot处理事务工作原理，连接池工作原理



### Springboot的核心是什么？









### Springboot中如何导入springMVC自己构建的模块，就是用什么注解可以一键导入？

### springboot的pom文件看过吗，里面有什么东西？

### 创建一个新的springboot应用要做哪些事情？框架已经搭建好了

### springboot里一个service接口，如何将只要是他的实现类都注册到容器里去



### SpringBoot自动装配原理（重点看一下）



参考Guide哥收录的篇文章：https://www.cnblogs.com/javaguide/p/springboot-auto-config.html



##  



##  