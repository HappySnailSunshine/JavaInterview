# Spring



> B站尚硅谷有个很好的课程，注解驱动，看了一点点。

# 注解驱动

![1594740884636](../media/pictures/Spring.assets/1594740884636.png)



> 原文：https://github.com/Snailclimb/JavaGuide.git
>
> guide哥这篇文章写的很好。

### 0.前言

可以毫不夸张地说，这篇文章介绍的 Spring/SpringBoot 常用注解基本已经涵盖你工作中遇到的大部分常用的场景。对于每一个注解我都说了具体用法，掌握搞懂，使用 SpringBoot 来开发项目基本没啥大问题了！

**为什么要写这篇文章？**

最近看到网上有一篇关于 SpringBoot 常用注解的文章被转载的比较多，我看了文章内容之后属实觉得质量有点低，并且有点会误导没有太多实际使用经验的人（这些人又占据了大多数）。所以，自己索性花了大概 两天时间简单总结一下了。

**因为我个人的能力和精力有限，如果有任何不对或者需要完善的地方，请帮忙指出！Guide 哥感激不尽！**

### 1. `@SpringBootApplication`

这里先单独拎出`@SpringBootApplication` 注解说一下，虽然我们一般不会主动去使用它。

_Guide 哥：这个注解是 Spring Boot 项目的基石，创建 SpringBoot 项目之后会默认在主类加上。_

```java
@SpringBootApplication
public class SpringSecurityJwtGuideApplication {
      public static void main(java.lang.String[] args) {
        SpringApplication.run(SpringSecurityJwtGuideApplication.class, args);
    }
}
```

我们可以把 `@SpringBootApplication`看作是 `@Configuration`、`@EnableAutoConfiguration`、`@ComponentScan` 注解的集合。

```java
package org.springframework.boot.autoconfigure;
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan(excludeFilters = {
		@Filter(type = FilterType.CUSTOM, classes = TypeExcludeFilter.class),
		@Filter(type = FilterType.CUSTOM, classes = AutoConfigurationExcludeFilter.class) })
public @interface SpringBootApplication {
   ......
}

package org.springframework.boot;
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Configuration
public @interface SpringBootConfiguration {

}
```

根据 SpringBoot 官网，这三个注解的作用分别是：

- `@EnableAutoConfiguration`：启用 SpringBoot 的自动配置机制
- `@ComponentScan`： 扫描被`@Component` (`@Service`,`@Controller`)注解的 bean，注解默认会扫描该类所在的包下所有的类。
- `@Configuration`：允许在 Spring 上下文中注册额外的 bean 或导入其他配置类

### 2. Spring Bean 相关

#### 2.1. `@Autowired`

自动导入对象到类中，被注入进的类同样要被 Spring 容器管理比如：Service 类注入到 Controller 类中。

```java
@Service
public class UserService {
  ......
}

@RestController
@RequestMapping("/users")
public class UserController {
   @Autowired
   private UserService userService;
   ......
}
```

#### 2.2. `@Component`,`@Repository`,`@Service`, `@Controller`

我们一般使用 `@Autowired` 注解让 Spring 容器帮我们自动装配 bean。要想把类标识成可用于 `@Autowired` 注解自动装配的 bean 的类,可以采用以下注解实现：

- `@Component` ：通用的注解，可标注任意类为 `Spring` 组件。如果一个 Bean 不知道属于哪个层，可以使用`@Component` 注解标注。
- `@Repository` : 对应持久层即 Dao 层，主要用于数据库相关操作。
- `@Service` : 对应服务层，主要涉及一些复杂的逻辑，需要用到 Dao 层。
- `@Controller` : 对应 Spring MVC 控制层，主要用于接受用户请求并调用 Service 层返回数据给前端页面。

#### 2.3. `@RestController`

`@RestController`注解是`@Controller和`@`ResponseBody`的合集,表示这是个控制器 bean,并且是将函数的返回值直 接填入 HTTP 响应体中,是 REST 风格的控制器。

_Guide 哥：现在都是前后端分离，说实话我已经很久没有用过`@Controller`。如果你的项目太老了的话，就当我没说。_

单独使用 `@Controller` 不加 `@ResponseBody`的话一般使用在要返回一个视图的情况，这种情况属于比较传统的 Spring MVC 的应用，对应于前后端不分离的情况。`@Controller` +`@ResponseBody` 返回 JSON 或 XML 形式数据

关于`@RestController` 和 `@Controller`的对比，请看这篇文章：[@RestController vs @Controller](https://mp.weixin.qq.com/s?__biz=Mzg2OTA0Njk0OA==&mid=2247485544&idx=1&sn=3cc95b88979e28fe3bfe539eb421c6d8&chksm=cea247a3f9d5ceb5e324ff4b8697adc3e828ecf71a3468445e70221cce768d1e722085359907&token=1725092312&lang=zh_CN#rd)。

#### 2.4. `@Scope`

声明 Spring Bean 的作用域，使用方法:

```java
@Bean
@Scope("singleton")
public Person personSingleton() {
    return new Person();
}
```

**四种常见的 Spring Bean 的作用域：**

- singleton : 唯一 bean 实例，Spring 中的 bean 默认都是单例的。
- prototype : 每次请求都会创建一个新的 bean 实例。
- request : 每一次 HTTP 请求都会产生一个新的 bean，该 bean 仅在当前 HTTP request 内有效。
- session : 每一次 HTTP 请求都会产生一个新的 bean，该 bean 仅在当前 HTTP session 内有效。

#### 2.5. `@Configuration`

一般用来声明配置类，可以使用 `@Component`注解替代，不过使用`@Configuration`注解声明配置类更加语义化。

```java
@Configuration
public class AppConfig {
    @Bean
    public TransferService transferService() {
        return new TransferServiceImpl();
    }

}
```

### 3. 处理常见的 HTTP 请求类型

**5 种常见的请求类型:**

- **GET** ：请求从服务器获取特定资源。举个例子：`GET /users`（获取所有学生）
- **POST** ：在服务器上创建一个新的资源。举个例子：`POST /users`（创建学生）
- **PUT** ：更新服务器上的资源（客户端提供更新后的整个资源）。举个例子：`PUT /users/12`（更新编号为 12 的学生）
- **DELETE** ：从服务器删除特定的资源。举个例子：`DELETE /users/12`（删除编号为 12 的学生）
- **PATCH** ：更新服务器上的资源（客户端提供更改的属性，可以看做作是部分更新），使用的比较少，这里就不举例子了。

#### 3.1. GET 请求

`@GetMapping("users")` 等价于`@RequestMapping(value="/users",method=RequestMethod.GET)`

```java
@GetMapping("/users")
public ResponseEntity<List<User>> getAllUsers() {
 return userRepository.findAll();
}
```

#### 3.2. POST 请求

`@PostMapping("users")` 等价于`@RequestMapping(value="/users",method=RequestMethod.POST)`

关于`@RequestBody`注解的使用，在下面的“前后端传值”这块会讲到。

```java
@PostMapping("/users")
public ResponseEntity<User> createUser(@Valid @RequestBody UserCreateRequest userCreateRequest) {
 return userRespository.save(user);
}
```

#### 3.3. PUT 请求

`@PutMapping("/users/{userId}")` 等价于`@RequestMapping(value="/users/{userId}",method=RequestMethod.PUT)`

```java
@PutMapping("/users/{userId}")
public ResponseEntity<User> updateUser(@PathVariable(value = "userId") Long userId,
  @Valid @RequestBody UserUpdateRequest userUpdateRequest) {
  ......
}
```

#### 3.4. **DELETE 请求**

`@DeleteMapping("/users/{userId}")`等价于`@RequestMapping(value="/users/{userId}",method=RequestMethod.DELETE)`

```java
@DeleteMapping("/users/{userId}")
public ResponseEntity deleteUser(@PathVariable(value = "userId") Long userId){
  ......
}
```

#### 3.5. **PATCH 请求**

一般实际项目中，我们都是 PUT 不够用了之后才用 PATCH 请求去更新数据。

```java
  @PatchMapping("/profile")
  public ResponseEntity updateStudent(@RequestBody StudentUpdateRequest studentUpdateRequest) {
        studentRepository.updateDetail(studentUpdateRequest);
        return ResponseEntity.ok().build();
    }
```

### 4. 前后端传值

**掌握前后端传值的正确姿势，是你开始 CRUD 的第一步！**

#### 4.1. `@PathVariable` 和 `@RequestParam`

`@PathVariable`用于获取路径参数，`@RequestParam`用于获取查询参数。

举个简单的例子：

```java
@GetMapping("/klasses/{klassId}/teachers")
public List<Teacher> getKlassRelatedTeachers(
         @PathVariable("klassId") Long klassId,
         @RequestParam(value = "type", required = false) String type ) {
...
}
```

如果我们请求的 url 是：`/klasses/{123456}/teachers?type=web`

那么我们服务获取到的数据就是：`klassId=123456,type=web`。

#### 4.2. `@RequestBody`

用于读取 Request 请求（可能是 POST,PUT,DELETE,GET 请求）的 body 部分并且**Content-Type 为 application/json** 格式的数据，接收到数据之后会自动将数据绑定到 Java 对象上去。系统会使用`HttpMessageConverter`或者自定义的`HttpMessageConverter`将请求的 body 中的 json 字符串转换为 java 对象。

我用一个简单的例子来给演示一下基本使用！

我们有一个注册的接口：

```java
@PostMapping("/sign-up")
public ResponseEntity signUp(@RequestBody @Valid UserRegisterRequest userRegisterRequest) {
  userService.save(userRegisterRequest);
  return ResponseEntity.ok().build();
}
```

`UserRegisterRequest`对象：

```java
@Data
@AllArgsConstructor
@NoArgsConstructor
public class UserRegisterRequest {
    @NotBlank
    private String userName;
    @NotBlank
    private String password;
    @NotBlank
    private String fullName;
}
```

我们发送 post 请求到这个接口，并且 body 携带 JSON 数据：

```json
{"userName":"coder","fullName":"shuangkou","password":"123456"}
```

这样我们的后端就可以直接把 json 格式的数据映射到我们的 `UserRegisterRequest` 类上。

![](../media/pictures/Spring.assets/663d1ec1-7ebc-41ab-8431-159dc1ec6589.png)

👉 需要注意的是：**一个请求方法只可以有一个`@RequestBody`，但是可以有多个`@RequestParam`和`@PathVariable`**。 如果你的方法必须要用两个 `@RequestBody`来接受数据的话，大概率是你的数据库设计或者系统设计出问题了！

### 5. 读取配置信息

**很多时候我们需要将一些常用的配置信息比如阿里云 oss、发送短信、微信认证的相关配置信息等等放到配置文件中。**

**下面我们来看一下 Spring 为我们提供了哪些方式帮助我们从配置文件中读取这些配置信息。**

我们的数据源`application.yml`内容如下：：

```yaml
wuhan2020: 2020年初武汉爆发了新型冠状病毒，疫情严重，但是，我相信一切都会过去！武汉加油！中国加油！

my-profile:
  name: Guide哥
  email: koushuangbwcx@163.com

library:
  location: 湖北武汉加油中国加油
  books:
    - name: 天才基本法
      description: 二十二岁的林朝夕在父亲确诊阿尔茨海默病这天，得知自己暗恋多年的校园男神裴之即将出国深造的消息——对方考取的学校，恰是父亲当年为她放弃的那所。
    - name: 时间的秩序
      description: 为什么我们记得过去，而非未来？时间“流逝”意味着什么？是我们存在于时间之内，还是时间存在于我们之中？卡洛·罗韦利用诗意的文字，邀请我们思考这一亘古难题——时间的本质。
    - name: 了不起的我
      description: 如何养成一个新习惯？如何让心智变得更成熟？如何拥有高质量的关系？ 如何走出人生的艰难时刻？
```

#### 5.1. `@value`(常用)

使用 `@Value("${property}")` 读取比较简单的配置信息：

```java
@Value("${wuhan2020}")
String wuhan2020;
```

#### 5.2. `@ConfigurationProperties`(常用)

通过`@ConfigurationProperties`读取配置信息并与 bean 绑定。

```java
@Component
@ConfigurationProperties(prefix = "library")
class LibraryProperties {
    @NotEmpty
    private String location;
    private List<Book> books;

    @Setter
    @Getter
    @ToString
    static class Book {
        String name;
        String description;
    }
  省略getter/setter
  ......
}
```

你可以像使用普通的 Spring bean 一样，将其注入到类中使用。

#### 5.3. `PropertySource`（不常用）

`@PropertySource`读取指定 properties 文件

```java
@Component
@PropertySource("classpath:website.properties")

class WebSite {
    @Value("${url}")
    private String url;

  省略getter/setter
  ......
}
```

更多内容请查看我的这篇文章：《[10 分钟搞定 SpringBoot 如何优雅读取配置文件？](https://mp.weixin.qq.com/s?__biz=Mzg2OTA0Njk0OA==&mid=2247486181&idx=2&sn=10db0ae64ef501f96a5b0dbc4bd78786&chksm=cea2452ef9d5cc384678e456427328600971180a77e40c13936b19369672ca3e342c26e92b50&token=816772476&lang=zh_CN#rd)》 。

### 6. 参数校验

**数据的校验的重要性就不用说了，即使在前端对数据进行校验的情况下，我们还是要对传入后端的数据再进行一遍校验，避免用户绕过浏览器直接通过一些 HTTP 工具直接向后端请求一些违法数据。**

**JSR(Java Specification Requests）** 是一套 JavaBean 参数校验的标准，它定义了很多常用的校验注解，我们可以直接将这些注解加在我们 JavaBean 的属性上面，这样就可以在需要校验的时候进行校验了，非常方便！

校验的时候我们实际用的是 **Hibernate Validator** 框架。Hibernate Validator 是 Hibernate 团队最初的数据校验框架，Hibernate Validator 4.x 是 Bean Validation 1.0（JSR 303）的参考实现，Hibernate Validator 5.x 是 Bean Validation 1.1（JSR 349）的参考实现，目前最新版的 Hibernate Validator 6.x 是 Bean Validation 2.0（JSR 380）的参考实现。

SpringBoot 项目的 spring-boot-starter-web 依赖中已经有 hibernate-validator 包，不需要引用相关依赖。如下图所示（通过 idea 插件—Maven Helper 生成）：

![](../media/pictures/Spring.assets/c7bacd12-1c1a-4e41-aaaf-4cad840fc073.png)

非 SpringBoot 项目需要自行引入相关依赖包，这里不多做讲解，具体可以查看我的这篇文章：《[如何在 Spring/Spring Boot 中做参数校验？你需要了解的都在这里！](https://mp.weixin.qq.com/s?__biz=Mzg2OTA0Njk0OA==&mid=2247485783&idx=1&sn=a407f3b75efa17c643407daa7fb2acd6&chksm=cea2469cf9d5cf8afbcd0a8a1c9cc4294d6805b8e01bee6f76bb2884c5bc15478e91459def49&token=292197051&lang=zh_CN#rd)》。

👉 需要注意的是： **所有的注解，推荐使用 JSR 注解，即`javax.validation.constraints`，而不是`org.hibernate.validator.constraints`**

#### 6.1. 一些常用的字段验证的注解

- `@NotEmpty` 被注释的字符串的不能为 null 也不能为空
- `@NotBlank` 被注释的字符串非 null，并且必须包含一个非空白字符
- `@Null` 被注释的元素必须为 null
- `@NotNull` 被注释的元素必须不为 null
- `@AssertTrue` 被注释的元素必须为 true
- `@AssertFalse` 被注释的元素必须为 false
- `@Pattern(regex=,flag=)`被注释的元素必须符合指定的正则表达式
- `@Email` 被注释的元素必须是 Email 格式。
- `@Min(value)`被注释的元素必须是一个数字，其值必须大于等于指定的最小值
- `@Max(value)`被注释的元素必须是一个数字，其值必须小于等于指定的最大值
- `@DecimalMin(value)`被注释的元素必须是一个数字，其值必须大于等于指定的最小值
- `@DecimalMax(value)` 被注释的元素必须是一个数字，其值必须小于等于指定的最大值
- `@Size(max=, min=)`被注释的元素的大小必须在指定的范围内
- `@Digits (integer, fraction)`被注释的元素必须是一个数字，其值必须在可接受的范围内
- `@Past`被注释的元素必须是一个过去的日期
- `@Future` 被注释的元素必须是一个将来的日期
- ......

#### 6.2. 验证请求体(RequestBody)

```java
@Data
@AllArgsConstructor
@NoArgsConstructor
public class Person {

    @NotNull(message = "classId 不能为空")
    private String classId;

    @Size(max = 33)
    @NotNull(message = "name 不能为空")
    private String name;

    @Pattern(regexp = "((^Man$|^Woman$|^UGM$))", message = "sex 值不在可选范围")
    @NotNull(message = "sex 不能为空")
    private String sex;

    @Email(message = "email 格式不正确")
    @NotNull(message = "email 不能为空")
    private String email;

}
```

我们在需要验证的参数上加上了`@Valid`注解，如果验证失败，它将抛出`MethodArgumentNotValidException`。

```java
@RestController
@RequestMapping("/api")
public class PersonController {

    @PostMapping("/person")
    public ResponseEntity<Person> getPerson(@RequestBody @Valid Person person) {
        return ResponseEntity.ok().body(person);
    }
}
```

#### 6.3. 验证请求参数(Path Variables 和 Request Parameters)

**一定一定不要忘记在类上加上 `Validated` 注解了，这个参数可以告诉 Spring 去校验方法参数。**

```java
@RestController
@RequestMapping("/api")
@Validated
public class PersonController {

    @GetMapping("/person/{id}")
    public ResponseEntity<Integer> getPersonByID(@Valid @PathVariable("id") @Max(value = 5,message = "超过 id 的范围了") Integer id) {
        return ResponseEntity.ok().body(id);
    }
}
```

更多关于如何在 Spring 项目中进行参数校验的内容，请看《[如何在 Spring/Spring Boot 中做参数校验？你需要了解的都在这里！](https://mp.weixin.qq.com/s?__biz=Mzg2OTA0Njk0OA==&mid=2247485783&idx=1&sn=a407f3b75efa17c643407daa7fb2acd6&chksm=cea2469cf9d5cf8afbcd0a8a1c9cc4294d6805b8e01bee6f76bb2884c5bc15478e91459def49&token=292197051&lang=zh_CN#rd)》这篇文章。

### 7. 全局处理 Controller 层异常

介绍一下我们 Spring 项目必备的全局处理 Controller 层异常。

**相关注解：**

1. `@ControllerAdvice` :注解定义全局异常处理类
2. `@ExceptionHandler` :注解声明异常处理方法

如何使用呢？拿我们在第 5 节参数校验这块来举例子。如果方法参数不对的话就会抛出`MethodArgumentNotValidException`，我们来处理这个异常。

```java
@ControllerAdvice
@ResponseBody
public class GlobalExceptionHandler {

    /**
     * 请求参数异常处理
     */
    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<?> handleMethodArgumentNotValidException(MethodArgumentNotValidException ex, HttpServletRequest request) {
       ......
    }
}
```

更多关于 Spring Boot 异常处理的内容，请看我的这两篇文章：

1. [SpringBoot 处理异常的几种常见姿势](https://mp.weixin.qq.com/s?__biz=Mzg2OTA0Njk0OA==&mid=2247485568&idx=2&sn=c5ba880fd0c5d82e39531fa42cb036ac&chksm=cea2474bf9d5ce5dcbc6a5f6580198fdce4bc92ef577579183a729cb5d1430e4994720d59b34&token=2133161636&lang=zh_CN#rd)
2. [使用枚举简单封装一个优雅的 Spring Boot 全局异常处理！](https://mp.weixin.qq.com/s?__biz=Mzg2OTA0Njk0OA==&mid=2247486379&idx=2&sn=48c29ae65b3ed874749f0803f0e4d90e&chksm=cea24460f9d5cd769ed53ad7e17c97a7963a89f5350e370be633db0ae8d783c3a3dbd58c70f8&token=1054498516&lang=zh_CN#rd)

### 8. JPA 相关

#### 8.1. 创建表

`@Entity`声明一个类对应一个数据库实体。

`@Table` 设置表名

```java
@Entity
@Table(name = "role")
public class Role {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String name;
    private String description;
    省略getter/setter......
}
```

#### 8.2. 创建主键

`@Id` ：声明一个字段为主键。

使用`@Id`声明之后，我们还需要定义主键的生成策略。我们可以使用 `@GeneratedValue` 指定主键生成策略。

**1.通过 `@GeneratedValue`直接使用 JPA 内置提供的四种主键生成策略来指定主键生成策略。**

```java
@Id
@GeneratedValue(strategy = GenerationType.IDENTITY)
private Long id;
```

JPA 使用枚举定义了 4 中常见的主键生成策略，如下：

_Guide 哥：枚举替代常量的一种用法_

```java
public enum GenerationType {

    /**
     * 使用一个特定的数据库表格来保存主键
     * 持久化引擎通过关系数据库的一张特定的表格来生成主键,
     */
    TABLE,

    /**
     *在某些数据库中,不支持主键自增长,比如Oracle、PostgreSQL其提供了一种叫做"序列(sequence)"的机制生成主键
     */
    SEQUENCE,

    /**
     * 主键自增长
     */
    IDENTITY,

    /**
     *把主键生成策略交给持久化引擎(persistence engine),
     *持久化引擎会根据数据库在以上三种主键生成 策略中选择其中一种
     */
    AUTO
}

```

`@GeneratedValue`注解默认使用的策略是`GenerationType.AUTO`

```java
public @interface GeneratedValue {

    GenerationType strategy() default AUTO;
    String generator() default "";
}
```

一般使用 MySQL 数据库的话，使用`GenerationType.IDENTITY`策略比较普遍一点（分布式系统的话需要另外考虑使用分布式 ID）。

**2.通过 `@GenericGenerator`声明一个主键策略，然后 `@GeneratedValue`使用这个策略**

```java
@Id
@GeneratedValue(generator = "IdentityIdGenerator")
@GenericGenerator(name = "IdentityIdGenerator", strategy = "identity")
private Long id;
```

等价于：

```java
@Id
@GeneratedValue(strategy = GenerationType.IDENTITY)
private Long id;
```

jpa 提供的主键生成策略有如下几种：

```java
public class DefaultIdentifierGeneratorFactory
		implements MutableIdentifierGeneratorFactory, Serializable, ServiceRegistryAwareService {

	@SuppressWarnings("deprecation")
	public DefaultIdentifierGeneratorFactory() {
		register( "uuid2", UUIDGenerator.class );
		register( "guid", GUIDGenerator.class );			// can be done with UUIDGenerator + strategy
		register( "uuid", UUIDHexGenerator.class );			// "deprecated" for new use
		register( "uuid.hex", UUIDHexGenerator.class ); 	// uuid.hex is deprecated
		register( "assigned", Assigned.class );
		register( "identity", IdentityGenerator.class );
		register( "select", SelectGenerator.class );
		register( "sequence", SequenceStyleGenerator.class );
		register( "seqhilo", SequenceHiLoGenerator.class );
		register( "increment", IncrementGenerator.class );
		register( "foreign", ForeignGenerator.class );
		register( "sequence-identity", SequenceIdentityGenerator.class );
		register( "enhanced-sequence", SequenceStyleGenerator.class );
		register( "enhanced-table", TableGenerator.class );
	}

	public void register(String strategy, Class generatorClass) {
		LOG.debugf( "Registering IdentifierGenerator strategy [%s] -> [%s]", strategy, generatorClass.getName() );
		final Class previous = generatorStrategyToClassNameMap.put( strategy, generatorClass );
		if ( previous != null ) {
			LOG.debugf( "    - overriding [%s]", previous.getName() );
		}
	}

}
```

#### 8.3. 设置字段类型

`@Column` 声明字段。

**示例：**

设置属性 userName 对应的数据库字段名为 user_name，长度为 32，非空

```java
@Column(name = "user_name", nullable = false, length=32)
private String userName;
```

设置字段类型并且加默认值，这个还是挺常用的。

```java
Column(columnDefinition = "tinyint(1) default 1")
private Boolean enabled;
```

#### 8.4. 指定不持久化特定字段

`@Transient` ：声明不需要与数据库映射的字段，在保存的时候不需要保存进数据库 。

如果我们想让`secrect` 这个字段不被持久化，可以使用 `@Transient`关键字声明。

```java
Entity(name="USER")
public class User {

    ......
    @Transient
    private String secrect; // not persistent because of @Transient

}
```

除了 `@Transient`关键字声明， 还可以采用下面几种方法：

```java
static String secrect; // not persistent because of static
final String secrect = “Satish”; // not persistent because of final
transient String secrect; // not persistent because of transient
```

一般使用注解的方式比较多。

#### 8.5. 声明大字段

`@Lob`:声明某个字段为大字段。

```java
@Lob
private String content;
```

更详细的声明：

```java
@Lob
//指定 Lob 类型数据的获取策略， FetchType.EAGER 表示非延迟 加载，而 FetchType. LAZY 表示延迟加载 ；
@Basic(fetch = FetchType.EAGER)
//columnDefinition 属性指定数据表对应的 Lob 字段类型
@Column(name = "content", columnDefinition = "LONGTEXT NOT NULL")
private String content;
```

#### 8.6. 创建枚举类型的字段

可以使用枚举类型的字段，不过枚举字段要用`@Enumerated`注解修饰。

```java
public enum Gender {
    MALE("男性"),
    FEMALE("女性");

    private String value;
    Gender(String str){
        value=str;
    }
}
```

```java
@Entity
@Table(name = "role")
public class Role {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String name;
    private String description;
    @Enumerated(EnumType.STRING)
    private Gender gender;
    省略getter/setter......
}
```

数据库里面对应存储的是 MAIL/FEMAIL。

#### 8.7. 增加审计功能

只要继承了 `AbstractAuditBase`的类都会默认加上下面四个字段。

```java
@Data
@AllArgsConstructor
@NoArgsConstructor
@MappedSuperclass
@EntityListeners(value = AuditingEntityListener.class)
public abstract class AbstractAuditBase {

    @CreatedDate
    @Column(updatable = false)
    @JsonIgnore
    private Instant createdAt;

    @LastModifiedDate
    @JsonIgnore
    private Instant updatedAt;

    @CreatedBy
    @Column(updatable = false)
    @JsonIgnore
    private String createdBy;

    @LastModifiedBy
    @JsonIgnore
    private String updatedBy;
}

```

我们对应的审计功能对应地配置类可能是下面这样的（Spring Security 项目）:

```java
@Configuration
@EnableJpaAuditing
public class AuditSecurityConfiguration {
    @Bean
    AuditorAware<String> auditorAware() {
        return () -> Optional.ofNullable(SecurityContextHolder.getContext())
                .map(SecurityContext::getAuthentication)
                .filter(Authentication::isAuthenticated)
                .map(Authentication::getName);
    }
}
```

简单介绍一下上面设计到的一些注解：

1. `@CreatedDate`: 表示该字段为创建时间时间字段，在这个实体被 insert 的时候，会设置值

2. `@CreatedBy` :表示该字段为创建人，在这个实体被 insert 的时候，会设置值

   `@LastModifiedDate`、`@LastModifiedBy`同理。

`@EnableJpaAuditing`：开启 JPA 审计功能。

#### 8.8. 删除/修改数据

`@Modifying` 注解提示 JPA 该操作是修改操作,注意还要配合`@Transactional`注解使用。

```java
@Repository
public interface UserRepository extends JpaRepository<User, Integer> {

    @Modifying
    @Transactional(rollbackFor = Exception.class)
    void deleteByUserName(String userName);
}
```

#### 8.9. 关联关系

- `@OneToOne` 声明一对一关系
- `@OneToMany` 声明一对多关系
- `@ManyToOne`声明多对一关系
- `MangToMang`声明多对多关系

更多关于 Spring Boot JPA 的文章请看我的这篇文章：[一文搞懂如何在 Spring Boot 正确中使用 JPA](https://mp.weixin.qq.com/s?__biz=Mzg2OTA0Njk0OA==&mid=2247485689&idx=1&sn=061b32c2222869932be5631fb0bb5260&chksm=cea24732f9d5ce24a356fb3675170e7843addbfcc79ee267cfdb45c83fc7e90babf0f20d22e1&token=292197051&lang=zh_CN#rd) 。

### 9. 事务 `@Transactional`

在要开启事务的方法上使用`@Transactional`注解即可!

```java
@Transactional(rollbackFor = Exception.class)
public void save() {
  ......
}

```

我们知道 Exception 分为运行时异常 RuntimeException 和非运行时异常。在`@Transactional`注解中如果不配置`rollbackFor`属性,那么事物只会在遇到`RuntimeException`的时候才会回滚,加上`rollbackFor=Exception.class`,可以让事物在遇到非运行时异常时也回滚。

`@Transactional` 注解一般用在可以作用在`类`或者`方法`上。

- **作用于类**：当把`@Transactional 注解放在类上时，表示所有该类的`public 方法都配置相同的事务属性信息。
- **作用于方法**：当类配置了`@Transactional`，方法也配置了`@Transactional`，方法的事务会覆盖类的事务配置信息。

更多关于关于 Spring 事务的内容请查看：

1. [可能是最漂亮的 Spring 事务管理详解](https://mp.weixin.qq.com/s?__biz=Mzg2OTA0Njk0OA==&mid=2247484943&idx=1&sn=46b9082af4ec223137df7d1c8303ca24&chksm=cea249c4f9d5c0d2b8212a17252cbfb74e5fbe5488b76d829827421c53332326d1ec360f5d63&token=1082669959&lang=zh_CN#rd)
2. [一口气说出 6 种 @Transactional 注解失效场景](https://mp.weixin.qq.com/s?__biz=Mzg2OTA0Njk0OA==&mid=2247486483&idx=2&sn=77be488e206186803531ea5d7164ec53&chksm=cea243d8f9d5cacecaa5c5daae4cde4c697b9b5b21f96dfc6cce428cfcb62b88b3970c26b9c2&token=816772476&lang=zh_CN#rd)

### 10. json 数据处理

#### 10.1. 过滤 json 数据

**`@JsonIgnoreProperties` 作用在类上用于过滤掉特定字段不返回或者不解析。**

```java
//生成json时将userRoles属性过滤
@JsonIgnoreProperties({"userRoles"})
public class User {

    private String userName;
    private String fullName;
    private String password;
    @JsonIgnore
    private List<UserRole> userRoles = new ArrayList<>();
}
```

**`@JsonIgnore`一般用于类的属性上，作用和上面的`@JsonIgnoreProperties` 一样。**

```java
public class User {

    private String userName;
    private String fullName;
    private String password;
   //生成json时将userRoles属性过滤
    @JsonIgnore
    private List<UserRole> userRoles = new ArrayList<>();
}
```

#### 10.2. 格式化 json 数据

`@JsonFormat`一般用来格式化 json 数据。：

比如：

```java
@JsonFormat(shape=JsonFormat.Shape.STRING, pattern="yyyy-MM-dd'T'HH:mm:ss.SSS'Z'", timezone="GMT")
private Date date;
```

#### 10.3. 扁平化对象

```java
@Getter
@Setter
@ToString
public class Account {
    @JsonUnwrapped
    private Location location;
    @JsonUnwrapped
    private PersonInfo personInfo;

  @Getter
  @Setter
  @ToString
  public static class Location {
     private String provinceName;
     private String countyName;
  }
  @Getter
  @Setter
  @ToString
  public static class PersonInfo {
    private String userName;
    private String fullName;
  }
}

```

未扁平化之前：

```json
{
    "location": {
        "provinceName":"湖北",
        "countyName":"武汉"
    },
    "personInfo": {
        "userName": "coder1234",
        "fullName": "shaungkou"
    }
}
```

使用`@JsonUnwrapped` 扁平对象之后：

```java
@Getter
@Setter
@ToString
public class Account {
    @JsonUnwrapped
    private Location location;
    @JsonUnwrapped
    private PersonInfo personInfo;
    ......
}
```

```json
{
  "provinceName":"湖北",
  "countyName":"武汉",
  "userName": "coder1234",
  "fullName": "shaungkou"
}
```

### 11. 测试相关

**`@ActiveProfiles`一般作用于测试类上， 用于声明生效的 Spring 配置文件。**

```java
@SpringBootTest(webEnvironment = RANDOM_PORT)
@ActiveProfiles("test")
@Slf4j
public abstract class TestBase {
  ......
}
```

**`@Test`声明一个方法为测试方法**

**`@Transactional`被声明的测试方法的数据会回滚，避免污染测试数据。**

**`@WithMockUser` Spring Security 提供的，用来模拟一个真实用户，并且可以赋予权限。**

```java
    @Test
    @Transactional
    @WithMockUser(username = "user-id-18163138155", authorities = "ROLE_TEACHER")
    void should_import_student_success() throws Exception {
        ......
    }
```

_暂时总结到这里吧！虽然花了挺长时间才写完，不过可能还是会一些常用的注解的被漏掉，所以，我将文章也同步到了 Github 上去，Github 地址： 欢迎完善！_

本文已经收录进我的 75K Star 的 Java 开源项目 JavaGuide：[https://github.com/Snailclimb/JavaGuide](https://github.com/Snailclimb/JavaGuide)。











# IOC 控制反转 

**Inverse of Controll**

**实例的生成权交给Spring就叫控制反转**

![](../media/pictures/Spring.assets/91825874bdbdb7e0e74835b58dea5ef3.png)

## DI 依赖注入

**Dependency Injection**

```java
UserDao userDao = new UserDaoImpl（）
UserService userService = new UserServiceImpl（）；
```

应用程序 Spring之间的关系

谁依赖谁

为什么需要依赖

谁注入谁

注入了什么

## 入门案例

### 入门案例1

#### 依赖

5+1 aop beans core context expression + jcl

只导入spring-context去导入5+1

![](../media/pictures/Spring.assets/3ba6bba201fc14e960c8541e50efd59e.png)

### 入门案例2

#### 导包

同上

#### 类

Dao层

![](../media/pictures/Spring.assets/6f2ed1569a68f2dc333f1351807c6929.png)

Service层

![](../media/pictures/Spring.assets/4e7cc09ed817ef996051cb2094e4eb4f.png)

#### 配置文件

![](../media/pictures/Spring.assets/a00d36b76477c5495b507d421b5dd341.png)

#### 单元测试

1. 查看service中是否已经维护了和dao之间的关系

![](../media/pictures/Spring.assets/b76c06984ee3960ea08cb26f76d39322.png)

1. 查看service中的dao实例和直接从spring容器中取出来的实例是同一个实例

![](../media/pictures/Spring.assets/9aced9e587822cc362a6f21614208ea4.png)

1. 通过类型从spring容器中取出组件，前提是该类型在spring容器中只包含一个实例

![](../media/pictures/Spring.assets/643b2328c87a1af064410b6a7cf6329b.png)

## ApplicationContext

![](../media/pictures/Spring.assets/2607e1505727a0f8ca80b12bc162dea3.png)

## Bean的实例化方式

### 构造方法

#### 无参构造（主要使用）

![](../media/pictures/Spring.assets/315daa8f8ab39a68bec1da1252e09ed5.png)

#### 有参构造

![](../media/pictures/Spring.assets/b67de1f3e167218af76823c39207d603.png)

### 工厂

#### 静态工厂

![](../media/pictures/Spring.assets/de849d11cbd2e1158e6485c3a7b5e51f.png)

#### 实例工厂

![](../media/pictures/Spring.assets/cfa7eba45968a029ae95647560b432cd.png)

## Scope

Prototype

Singleton

## Bean的lifeCycle

参考ppt

## Collection

![](../media/pictures/Spring.assets/c64d722b53f733cf1e51d9a7ab1814aa.png)

## 通过注解使用spring（重中之重）

### 准备工作，扫描注解

![](../media/pictures/Spring.assets/9d99a67494e071fcf7125bb2da48bc97.png)

### 组件注册

\@Component

\@Component（“组件id”）

Dao：\@Repository

Service：\@Service

Controller：\@Controller

### 属性注入

1. \@Autowired单独使用，根据类型(使用最多的)
2. \@Autowired和\@Qualifier(“bean的id”)
3. \@Resource(name=””)

\@Value（“值”）

不再需要set方法了

### Scope

在类上直接写注解\@Scope

![](../media/pictures/Spring.assets/7eab014974b1bc4de5754915b083183b.png)

### Init-method和destroy-method

![](../media/pictures/Spring.assets/2e9fe7aa9d133bac8a9b23137f9e92ff.png)

### Spring对junit的支持

![](../media/pictures/Spring.assets/c0f9f156d31d4c886a464b8364e41753.png)

![](../media/pictures/Spring.assets/ca470f8239e3e98afbef45749b81dd76.png)

## 了解一下先

ORM

Object Relationship Mapping

DispatcherServlet 特殊Springmvc的核心

Aop Aspect Oriented Program

## Cglib动态代理

通过继承

![](../media/pictures/Spring.assets/3affad49b93247dbff2346d464d3e16a.png)



## 代码部分!

resources里面 application.xml

```java
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context" xsi:schemaLocation="
        http://www.springframework.org/schema/beans https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context https://www.springframework.org/schema/context/spring-context.xsd">

	//下面这里是新建bean对象的
    <!-- bean definitions here -->
    <bean id="userServiceBean" class="com.cskaoyan.service.impl.UserServiceImpl"></bean>
</beans>

```

Test方法里要测试spring构建出来的对象,需要下面这几句:

```java
ApplicationContext applicationContext = new ClassPathXmlApplicationContext("application.xml");

        UserService userService = (UserService) applicationContext.getBean("userServiceBean");

        int steve = userService.queryAgeByName("steve");

```

### Bean的实例化方式

#### 构造方法

##### 无参构造(主要使用)

通过无参构造方法和set方法注册

```xml
<bean id="user2" class="com.cskaoyan.bean.User2">  
    <property name="username" value="lisi"/>  
    <property name="password" value="123456"/>
</bean>

```

##### 有参构造

```java
public User(String usernamez, String password) {
    this.username = usernamez;
    this.password = password;
}

```

```xml
<bean id="user" class="com.cskaoyan.bean.User">
     <constructor-arg name="usernamez" value="zhanngsan"/>
     <constructor-arg name="password" value="112345"/>
</bean>

```

#### 工厂

##### 静态工厂

```java
public class MoneyFactory {
    public static Money createMoney(){
        return new Money();
    }
}

```

```xml
<bean id="moneyBeanFromStaticFactory" class="com.cskaoyan.factory.MoneyFactory" factory-method="createMoney"/>

```

##### 实例工厂

```java
public class MoneyInstanceFactory {
    public Money createMoney(){
        return new Money();
    }
}

```

```xml
<!--实例工厂-->
   <!--首先先去实例一个工厂-->
 	 <bean id="instanceFactory" class="com.cskaoyan.factory.MoneyInstanceFactory"/>
   <!--接着根据实例的工厂去生产实例-->
 	 <bean id="moneyBeanFromInstanceFactory" factory-bean="instanceFactory" factory-method="createMoney"/>

```

##### Scope

```xml
<bean id="singleton" class="com.cskaoyan.bean.Singleton" scope="singleton"/><!--一个对象-->
<bean id="prototype" class="com.cskaoyan.bean.Singleton" scope="prototype"/><!--不是一个对象-->

```

### Bean的生命周期

```java
public class LifeCycleBean implements BeanNameAware, BeanFactoryAware {

    String parameter;

    public String getParameter() {
        return parameter;
    }

    public void setParameter(String parameter) {
        System.out.println("2、setter方法");
        this.parameter = parameter;
    }

    public LifeCycleBean() {
        System.out.println("1、构造方法");
    }

    @Override
    public void setBeanName(String s) {
        System.out.println("3、beanNameAware" + s);
    }

    @Override
    public void setBeanFactory(BeanFactory beanFactory) throws BeansException {
        System.out.println("4、beanFactoryAware" + beanFactory);
    }

    public void myinit() {
        System.out.println("7、myinit");
    }

    public void mydestory() {
        System.out.println("10、mydestory");
    }
}

```

下面这个是和初始化有关的一个接口,和一个方法!

```java
public class MyBeanPostProcessor implements BeanPostProcessor {

    @Override
    public Object postProcessBeforeInitialization(Object bean, String beanName) throws 			BeansException {
        System.out.println("before");
        return bean;
    }

    @Override
    public Object postProcessAfterInitialization(Object bean, String beanName) throws 			BeansException {
        System.out.println("after");
        return bean;
    }
}

```

application.xml里面的要写的初始化和要销毁的方法:

init-method="myinit" ,destroy-method="mydestory"只有这里进行了说明以后,才会在实现了BeanPostProcessor接口以后有所表现!

也就是下面的:

before
7、myinit
after

```xml
<bean class="com.cskaoyan.bean.LifeCycleBean" init-method="myinit" destroy-					method="mydestory">   
    <property name="parameter" value="12345"/>
</bean>
<bean class="com.cskaoyan.bean.MyBeanPostProcessor"/>

```

上面的程序运行出来的结果是:

1、构造方法
2、setter方法
3、beanNameAwarecom.cskaoyan.bean.LifeCycleBean#0
4、beanFactoryAwareorg.springframework.beans.factory.support.DefaultListableBeanFactory@60c6f5b: defining beans [com.cskaoyan.bean.LifeCycleBean#0,com.cskaoyan.bean.MyBeanPostProcessor#0]; root of factory hierarchy
before
7、myinit
after
com.cskaoyan.bean.LifeCycleBean@ed9d034
10、mydestory

### collection注册

bean中的有好多集合属性

```java
List<String> stringList;
List<User> userList;
String[] array;
Set<String> setData;
Map<String,String> mapData;
Properties properties;

```

application.xml文件中,注册集合和bean的集合!

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context" xsi:schemaLocation="
        http://www.springframework.org/schema/beans https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context https://www.springframework.org/schema/context/spring-context.xsd">

    <bean id="collectionBean" class="com.cskaoyan.bean.CollectionBean">
       
        <property name="array">
            <array>
                <value>arrayData1</value>
                <value>arrayData2</value>
                <value>arrayData3</value>
            </array>
        </property>
        <property name="stringList">
            <list>
                <value>listData1</value>
                <value>listData2</value>
                <value>listData3</value>
            </list>
        </property>
        <property name="setData">
            <set>
                <value>setData1</value>
                <value>setData2</value>
                <value>setData3</value>
            </set>
        </property>

        <property name="mapData">
            <map>
                <entry key="key1" value="value1"/>
                <entry key="key2" value="value2"/>
                <entry key="key3" value="value3"/>
            </map>
        </property>
      
        <!--这里是Properties属性-->
        <property name="properties">
            <props>
                <prop key="key1">propValue1</prop>
                <prop key="key2">propValue2</prop>
                <prop key="key3">propValue3</prop>
            </props>
        </property>
        
        <property name="userList">
            <list>
            <bean class="com.cskaoyan.bean.User"> <!--bean里面有属性,要放在property里面-->
                <property name="username" value="user1"></property>
                <property name="password" value="password1"></property>
            </bean>
                <!--ref的bean属性写的是容器当中其他组件id-->
                <ref bean="user2"></ref>
            </list>
        </property>
    </bean>
    <bean id="user2" class="com.cskaoyan.bean.User">
        <property name="username" value="user2"/>
        <property name="password" value="password2"/>
    </bean>
</beans>
```

### 注解(最重要的部分)

#### 1.使用注解之前要先扫描注解注解

```java
 <!--打开注解的扫描开关-->
    <!--aop tx mvc transaction-->
    <!--这个包以及这个包下的所有的目录-->
    <context:component-scan base-package="com.cskaoyan"/>


```

#### 2.组件注册

```java
@Component
@Component（“组件id”）

Dao：@Repository
Service：@Service
Controller：@Controller


```

#### 3.属性注入

1、@Autowired单独使用，根据类型(使用最多的)
2、@Autowired和@Qualifier(“bean的id”)
3、@Resource(name=””)



@Value（“值”）

不再需要set方法了

```java
  @Autowired
  @Qualifier("userDao2")
  @Resource(name = "userDao2")
  UserDao userDao;
  @Value("123456")
  String abc;

  @Override
  public void register(String username, String password) {
      System.out.println("service的abc = " + abc);
      userDao.addUser(username, password);
}


```

#### 4.scope域

```java
@Component("user2")
@Scope("prototype")
public class User2 {
}


```

#### 5.Init-method和destroy-method

初始化和销毁,必须实现这两个注解,才会在测试的时候有所表现!下面的是用close方法关闭,才可以用!

```$java
@PostConstruct
public void init(){
	System.out.println("user的init");
}
@PreDestroy
public void destroy(){
	System.out.println("user的destory");
}


```

#### 6.Spring对junit的支持

依赖得写!

```xml
<dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.12</version>
            <scope>test</scope>
</dependency>


```

加了这两句注解以后就不用写以前的一句导入application.xml一句

```java
//以前是这样写的,引入application.xml
ApplicationContext applicationContext = new ClassPathXmlApplicationContext("application.xml");
        CollectionBean bean = applicationContext.getBean(CollectionBean.class);
        System.out.println(bean);


```



```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration("classpath:application.xml")
public class AnnotationTest2 {
    @Resource(name = "userDao2")
    UserDao userDao;
    @Test
    public void mytest(){
        //UserServiceImpl userService = new UserServiceImpl();
        System.out.println(111);
    }

}


```

### 动态代理

两种代理比较长问。

参考：https://www.cnblogs.com/whirly/p/10154887.html



#### jdk动态代理

jdk动态代理代理的是接口,其中TransferService是接口,后面是其实现类!

```java
public class MainTest2 {
    @Test
    public void mytest(){
        TransferService transferService = new TransferServiceImpl();
        TransferService transferServiceProxy = (TransferService) 										Proxy.newProxyInstance(transferService.getClass().getClassLoader(),
                transferService.getClass().getInterfaces(), new InvocationHandler() {
            
            @Override
            public Object invoke(Object proxy, Method method, Object[] args) throws 					Throwable {
                Object invoke = method.invoke(transferService, args);
                return invoke;
            }
        });
        //这里的transfer是接口的实现类的一个方法(转账案例!)
        boolean transfer = transferServiceProxy.transfer(1, 2, 1000);
    }
}


```

#### cglib动态代理

bean对象

```java
public class HouseOwner {

    public boolean rentHouse(int money){
        System.out.println("rentHOuse :" + money);
        if (money >= 2000){
            return true;
        }
        return false;
    }
}


```

cglib动态代理的Test

```java
public class CglibTest {
    @Test
    public void mytest(){
        HouseOwner houseOwner = new HouseOwner();

        HouseOwner houseOwnerProxy = (HouseOwner) Enhancer.create(HouseOwner.class,
                new InvocationHandler() {
            @Override
            public Object invoke(Object proxy, Method method, Object[] args) throws 						Throwable {
                System.out.println("before ");
                Object invoke = method.invoke(houseOwner, args);
                System.out.println("after");
                return invoke;
            }
        });
        boolean b = houseOwnerProxy.rentHouse(2000);
    }
}


```

# 二.AOP

## OOP和AOP区别，各自的优缺点

![1585976658903](../media/pictures/Spring.assets/1585976658903.png)

OOP：面向对象编程   比较强调对象的**封装 继承 多态**

### OOP优点：

**OOP 的优点：**使人们的编程与实际的世界更加接近，所有的对象被赋予属性和方法，结果编程就更加富有人性化

而且维护方便 ，阅读方便 

**OOP缺点：**方便的同时牺牲了性能 在有些场合性能还是很重要的

#### OOP六大设计原则

单一职责原则，里氏替换原则，依赖倒置原则，接口隔离原则，迪米特原则，开闭原则（对扩展开放，对修改关闭）

https://blog.csdn.net/slx3320612540/article/details/71662775

### AOP优点：

AOP（Aspect Oriented Programming）意为：面向切面编程，通过预编译方式和运行期动态代理实现程序功能的统一维护的一种技术。AOP是OOP的延续，是软件开发中的一个热点，也是Spring框架中的一个重要内容（Spring核心之一），是函数式编程的一种衍生范型。**利用AOP可以对业务逻辑的各个部分进行隔离**，从而使得业务逻辑各部分之间的**耦合度降低**，**提高程序的可重用性，同时提高了开发的效率**。



#### JavaGuide里面推荐的一篇文章： 下面这一段可以面试的时候说 

[Spring AOP 基于AspectJ注解如何实现AOP](https://juejin.im/post/5a55af9e518825734d14813f) ： **AspectJ是一个AOP框架，它能够对java代码进行AOP编译（一般在编译期进行），让java代码具有AspectJ的AOP功能（当然需要特殊的编译器）**，可以这样说AspectJ是目前实现AOP框架中最成熟，功能最丰富的语言，更幸运的是，AspectJ与java程序完全兼容，几乎是无缝关联，因此对于有java编程基础的工程师，上手和使用都非常容易。Spring注意到AspectJ在AOP的实现方式上依赖于特殊编译器(ajc编译器)，因此Spring很机智回避了这点，转向采用动态代理技术的实现原理来构建Spring AOP的内部机制（动态织入），这是与AspectJ（静态织入）最根本的区别。**Spring 只是使用了与 AspectJ 5 一样的注解，但仍然没有使用 AspectJ 的编译器，底层依是动态代理技术的实现，因此并不依赖于 AspectJ 的编译器**。 Spring AOP虽然是使用了那一套注解，其实实现AOP的底层是使用了动态代理(JDK或者CGLib)来动态植入。至于AspectJ的静态植入，不是本文重点，所以只提一提。

## 名词

Target：目标类，委托类，举例：房东

Pointcut：切入点，通过切入点 指定要等增强：谁

Advice：通知：在什么时机 做什么事情

切面：切入点+通知。通知 谁 在什么时机 做什么事情

## SpringAop

![](../media/pictures/Spring.assets/9556a2b1cf95435cd8b31c43ccbb5c8d.png)

### 导包

Spring 5+1

Aop alliance

### Class

UserService和CustomAdvice

![](../media/pictures/Spring.assets/bceb85570fe625ec7f852fe0806fc759.png)

### 组件的注册

3个组件

UserService的实例

Advice通知的实例（实现MethodInterceptor的接口，通过invoke方法通知在什么之间做什么事情）

UserServiceProxy代理对象的实例（通过目标对象和interceptorName→容器中通知）

![](../media/pictures/Spring.assets/6fa5ca615a2cac0b82b9bc5b74f51c52.png)

### 使用

从容器中取出的是：代理对象的实例

![](../media/pictures/Spring.assets/d60732cd73635d37497e9b940c6310ab.png)

## AspectJ

理解 参考 https://blog.csdn.net/wenyingzhi/article/details/83989707

### 导包

Spring

Aspectjweaver

![](../media/pictures/Spring.assets/28c2bbc8db3a6c409921116f91bf7cfc.png)

### 业务逻辑模块组件注册

注册UserService的实例

![](../media/pictures/Spring.assets/3ede0dd4bac17f63e6278d2a62fe5495.png)

### 使用

#### Advisor

注册通知类，并且配置

![](../media/pictures/Spring.assets/0cd140b4cbf873859244db13bf170f07.png)

单元测试：直接获得service实例

#### Aspect（最重要的）

切面类：切入点和通知

##### 通知

![](../media/pictures/Spring.assets/2700b8bae4afb1565446f4d3845d31c7.png)

##### JointPoint

作为通知方法的形参，可以通过joinPoint拿到代理对象、目标对象、方法、参数

![](../media/pictures/Spring.assets/1db5e7c0b261d259a5c47df567d83f5d.png)

##### Before

在委托类（目标类）方法执行之前

##### After

在委托类（目标类）方法执行之后，不管发生什么，都会执行到，类似于finally

![](../media/pictures/Spring.assets/9f5fa561991b5021aa67e8838d47ceae.png)

##### Around

环绕 目标类的执行方法

方法返回值是Object 参数是ProceedingJoinPoint

joinPoint的proceed方法类似于动态代理中的method.invoke方法

![](../media/pictures/Spring.assets/e220e2c83d8ad38470253ee6bee6f57b.png)

##### After-throwing

传入参数Throwable 和aop通知标签的throwing属性对应

![](../media/pictures/Spring.assets/34e14c73ab15a4ed11699d43d801d626.png)

##### After-returing

![](../media/pictures/Spring.assets/1c78058ebed263d7db2a867cfda7310e.png)

#### 通过注解去实现切面AspectJ

##### Aspectj的注解开关

![](../media/pictures/Spring.assets/47b1452adc84b90259fb047dd40cd821.png)

##### 通过注解去配置切面类

![](../media/pictures/Spring.assets/ec5962d2727b504fbf6da88854f8a2cd.png)

其他通知：

![](../media/pictures/Spring.assets/63d50b9b81491ca3936f191dbd4e8ece.png)

##### 通过自定义注解对指定方法进行增强

自定义注解

![](../media/pictures/Spring.assets/fb6aaf7f8eed937d16e324fc8a789d6d.png)

切入点表达式：

![](../media/pictures/Spring.assets/f4f580dae6d0750de89c861b9dd054e3.png)

容器中组件的方法上增加注解：

![](../media/pictures/Spring.assets/25da9944f88fdfc4fa3c8b3ce0e40c1f.png)

### 切入点表达式(非常核心的内容)

指定要被增强的方法

Execution（修饰符 返回值 包名.类名.方法名（参数））

#### 修饰符

不写 == 通配

通常我们也不写修饰符

#### 返回值

能不能省略？ 不能省略

能不能通配？ 可以通配，使用\*代表任意类型的返回值

特殊情况：返回值要写类的全类名。除了java.lang包下的类和基本类型可以直接写。

#### 包名+类名+方法名

能不能省略？ 可以省略（有条件），除了头和尾，使用..省略中间的部分

能不能通配？
可以通配，使用\*通配，代表一个单词或者一个单词的一部分，最多通配一级目录。
头和尾都可以进行通配

#### 参数

能不能省略？（）对应的是无参的方法

能不能写一个东西代表任意类型任意个数的参数呢？ (..)

能不能通配？\* 进行通配，一个\*通配一个任意类型的参数，如果多个参数就多个\*

特殊情况：特定的参数类型需要写类的全类名。除了java.lang包下的类和基本类型可以直接写

#### Throws

## JdbcTemplate

### Se代码的实现

![](../media/pictures/Spring.assets/0eac9f3584cf791469c897d23f55ceaf.png)



## 代码部分

### 面向切面编程

#### 经典的应用

事务管理、性能监视、安全检查、缓存 、日志等

#### AOP编程术语



1.Target目标类(需要被代理的类,委托类)

2.JoinPoint 链接点,指被代理对象里那些可能被增强的点(方法),如所有方法(候选的可能被增强的候选点)

**3.PointCut切入点  已经被增强的连接点**

**4.Advice 通知(具体增强的代码). 代理对象1执行到JoinPoint所做的事情**

5.Weaver 织入(植入)是指把advice应用到目标对象来创建新的dialing对象的过程

6.Proxy 代理类 (动态代理生成的)

**7.Aspect切面是切入点和通知的结合(一个线是一条特殊的面  一个切入点和一个通知组成一个特殊的面)**



### SpringAOP的使用

**问题**:SpringAOP的本质就是动态代理，那么Spring到底使用的是JDK代理，还是cglib代理呢？

**答：**混合使用。如果被代理对象实现了接口，就优先使用JDK代理，如果没有实现接口，就用用cglib代理。

![1568548911645](../media/pictures/Spring.assets/1568548911645.png)

本质上就是动态代理!动态代理有两种:jdk,cglib.

#### 1.导包

Spring 5 + 1

#### 2.Class

```java
@Component("myadvice")
public class CustomAdvice implements MethodInterceptor {
    @Override
    public Object invoke(MethodInvocation methodInvocation) throws Throwable {
        System.out.println("before");
        //这一行执行的是委托类的方法，类似于动态代理中的method。invoke
        Object proceed = methodInvocation.proceed();
        System.out.println("after");
        return proceed;
    }
}


```

```java
@Service("userService")
public class UserServiceImpl implements UserService {
    @Override
    public int register(String username, String password) {
        System.out.println("username = " + username + "|| password = " + password);
        return 0;
    }
}


```

#### 3.组件注册

里面有三个实例

userService的实例

advice通知的实例(实现MethodInterceptor的接口,通知invoke方法在什么直接做什么事情)!

userServiceProxy代理对象的实例(通过目标对象和interceptorName 容器中通知)

```xml
<bean id="userServiceProxy" class="org.springframework.aop.framework.ProxyFactoryBean">
        <property name="target" ref="userService"/>   容器中的目标对象
        <!--容器中这个通知的名称（id）-->
        <property name="interceptorNames" value="myadvice"/>  容器中通知的id
</bean>


```

#### 4.使用

正在起作用的是代理对象的实例,而不是原来对象的实例

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration("classpath:application.xml")
public class AopTest {
    @Resource(name = "userService")     //这个没起到作用
    UserService userService;
    @Resource(name = "userServiceProxy")   //代理对象起到作用
    UserService userServiceProxy;
    
    @Test
    public void mytest(){
        userService.register("songge","123456");
    }
    @Test
    public void mytest2(){
        userServiceProxy.register("songge","123456");
    }
}


```

### AspectJ

#### 1.要织入包

```xml
<!--织入包-->
<dependency>
    <groupId>org.aspectj</groupId>
    <artifactId>aspectjweaver</artifactId>
    <version>1.6.12</version>
</dependency>


```

#### 2.业务逻辑模块组件注册

```java
@Service("userService")
public class UserServiceImpl implements UserService {
    @Override
    public User register(String username, String password) {
        System.out.println("username = " + username + "|| password = " + password);
        return new User();
    }
}


```

#### 3.使用

##### Advisor

注册通知类,并且配置 

```xml
<!--使用aspectJ-->
<aop:config>
   <!--pointcut-->
   <!--切入点的表达式：指定的是方法→容器中组件的方法-->          这里是切入点表达式
	<aop:pointcut id="mypointcut" expression="execution(com.cskaoyan.bean.User
			*.cs*..*(..))"/>
   <!--advisor-->
   <aop:advisor advice-ref="myadvice" pointcut-ref="mypointcut" />
</aop:config>


```

```java
@Component("myadvice")
public class CustomAdvice implements MethodInterceptor {
    @Override
    public Object invoke(MethodInvocation methodInvocation) throws Throwable {
        System.out.println("before");
        //这一行执行的是委托类的方法，类似于动态代理中的method。invoke
        Object proceed = methodInvocation.proceed();
        System.out.println("after");
        return proceed;
    }
}


```

# 三.Spring-tx(事务)

## JdbcTemplate

//使用

![](../media/pictures/Spring.assets/8fa6cd1febe19b5a8edbb98006cf52a5.png)

//spring整合其他的框架

//Datasource JdbcTemplate

### 组件注册

![](../media/pictures/Spring.assets/ed99ae51b75712bf8b50a3a1c78efeb4.png)

### 使用组件

![](../media/pictures/Spring.assets/7dae6d939605fb2edd4bfd70df814ce8.png)

### 拓展使用（通过JdbcDaoSupport）

继承jdbcDaoSupport，JdbcDaoSupport提供了jdbcTemplate的成员变量和setDatasource的方法，注意：这个setDatasource做的是初始化JdbcTemplate的事情

![](../media/pictures/Spring.assets/3f42f45a744753b5920ada83e4d8f5f9.png)

如果是xml形式，则使用property标签调用到setDatasource方法

![](../media/pictures/Spring.assets/681b8696631818bcf8434dcb655542c6.png)

如果使用注解，则需使用\@Autowired注解，通过它调用父类的setDatasource方法

![](../media/pictures/Spring.assets/d676a9921a77a07b7089f9566ae8f98a.png)

## JavaConfig

使用纯java代码的形式 配置spring → 为springboot做铺垫

配置类：写一个类，这类中注册spring的组件

![](../media/pictures/Spring.assets/e81c56e93d418210d1a741bcecc70e19.png)

![](../media/pictures/Spring.assets/9968deb40f28b05ddcbd73ceeb9b3bd9.png)

![](../media/pictures/Spring.assets/3f029f0e9ae09228b70f1032095d1751.png)

![](../media/pictures/Spring.assets/e250b40c212c1392c94e850ccc2a4b49.png)

## Spring tx=transaction

### 事务的特点

A：原子性

C：一致性

I：隔离性

D：持久化

### 事务并发可能引起的问题

脏读、不可重复读、虚读

![](../media/pictures/Spring.assets/1313717269305cea1fb43b9bce74a3c7.png)

![](../media/pictures/Spring.assets/416840be4f6abf922b5115858ab66d20.png)

![](../media/pictures/Spring.assets/fbd56401101e30026d25dd798197028f.png)

### 事务的隔离级别

Read-uncommitted

Read-commited

Read-repeatable

serialize

### Spring事务核心名词

#### PlatFormTransactionManager

平台事务管理器：接口，提供规范

实现类：DatasourceTransactionManager

#### TransactionStatus

事务状态，Manager根据Status决定执行何种操作

Status从何而来，根据Definition来的

![](../media/pictures/Spring.assets/471a71bf6aa0d4acec2cfabb1b6597c4.png)

#### TransactionDefinition

![](../media/pictures/Spring.assets/c6000a840a98fc6a97151e49cb5eb047.png)

##### 事务的传播行为

多个业务之间共享事务

![](../media/pictures/Spring.assets/4b47eb0bc094d6dd6c415fbdaef6c85d.png)

**REQUIRED**: 支持当前事务，如果当前没有事务，就新建一个事务。

SUPPORTS：支持当前事务，如果当前没有事务，就以非事务方式执行。

MANDATORY： 支持当前事务，如果当前没有事务，就抛出异常

**REQUIRES_NEW**：新建事务，如果当前存在事务，把当前事务挂起

NOT_SUPPORTED: 以非事务方式执行操作，如果当前存在事务，就把当前事务挂起

NEVER: 以非事务方式执行，如果当前存在事务，则抛出异常

NESTED:
支持当前事务，如果当前事务存在，则执行一个嵌套（父子）事务，如果当前没有事务，就新建一个事务

#### 案例

##### TransactionTemplate

在容器中注册TransactionTemplate的组件

![](../media/pictures/Spring.assets/d908e3f6bbb878f6d5b5ed0bb3108e69.png)

![](../media/pictures/Spring.assets/ab993ae952c5393343ffd88a55684326.png)

##### Demo2 基于动态代理（aop）

通过TransactionProxyFactoryBean生成目标对象的代理对象

Xml：

![](../media/pictures/Spring.assets/c6eecc2ab4b2332f32c9c71455aa3926.png)

JavaConfig：

![](../media/pictures/Spring.assets/26a954db68c37182a43efd8c2b759201.png)

##### aspectJ实现

需要额外导入aspectjweaver织入包

![](../media/pictures/Spring.assets/328eed5017952751259b15cdb2a89c80.png)

通过aop:advisor和tx:advise实现

![](../media/pictures/Spring.assets/985ce70453efea9d6f81cd1753fdf368.png)

##### 注解驱动（一定要会）

通过平台事务管理器和事务注解驱动

![](../media/pictures/Spring.assets/a7cc4daf4b9937c3d8a395e34dfb830f.png)

或者注解驱动换成注解的形式

![](../media/pictures/Spring.assets/853516f8839dc6a901eaea1a8d3e8cbe.png)

使用：在类上或方法上增加\@Transactional注解

![](../media/pictures/Spring.assets/0853ef8adcac1d52b0720811decde55a.png)

![](../media/pictures/Spring.assets/4c66c0066b6386bc4ba76c65f5cca6ff.png)

# 四.SpringMVC

## Springmvc流程

![](../media/pictures/Spring.assets/8fa547296d1b7598e9298ac831aed6f2.png)

## Springmvc

### 入门案例1

DispatcherServlet开始加载spring容器

Spring-web spring-webmvc

5+2+1

#### 导包

![](../media/pictures/Spring.assets/5eebf7a443be80565c2cf44f9b833877.png)

#### Web.xml

![](../media/pictures/Spring.assets/dadab53140c8586f1cf0ff6f6f6e6620.png)

#### 注册组件

![](../media/pictures/Spring.assets/952477ae57bf4faacdbb31d4ac319d09.png)

#### Handler代码

![](../media/pictures/Spring.assets/9ed8e59108fcec8a206fee4b3fe89e93.png)

### 入门案例2

#### Mvc注解驱动

修改映射器和适配器为\<mvc:annotation-driven/\>

![](../media/pictures/Spring.assets/e9b09d443acb26a1c69e2dd39729c4c6.png)

![](../media/pictures/Spring.assets/7adcac54252d257eac80c046a68b7d5c.png)

#### 使用

直接在类上增加\@Controller注解，则该类为Handler类

通过\@RequestMapping将请求url和应用中的方法建立映射关系

![](../media/pictures/Spring.assets/e0f0b17b7bd441f6897947b104f9d1e5.png)

### RequestMapping的使用

#### url路径映射

#### 窄化请求

User/query

User/insert

User/delete

通过在类上增加\@RequestMapping的注解进行请求的窄化

![](../media/pictures/Spring.assets/4b00dcaaf20c4aa29f9ede0fc49975c6.png)

#### 请求方法限定（405）

通过\@RequestMapping中的method属性进行请求方法的限定

![](../media/pictures/Spring.assets/146392f8049af101c54acd42738f182c.png)

由RequestMapping延伸出的注解

![](../media/pictures/Spring.assets/ed156f9adbcdc1fa613b693bd00f5978.png)

#### 请求参数的限定（400）

Params属性

![](../media/pictures/Spring.assets/0a87775546221dd427e0b6cf8c53acda.png)

#### 请求的Content-type的限定（415）

![](../media/pictures/Spring.assets/07d9956cb986ebc858e5e166c7a13985.png)

#### 请求Accept的限定（406）

![](../media/pictures/Spring.assets/59594091435528190ce8f0f94f34ad60.png)

### Controller对应方法的返回值(处理ModelAndView的时候)

#### Void

在形参中可以增加HttpServletRequest和HttpServletResponse

![](../media/pictures/Spring.assets/90630ac97e769f09a3132ea059ff09da.png)

#### ModelAndView

略

#### String

处理的还是ModelAndView

##### 物理视图

下面的没有/的里面,实际上是路径里面去掉noGang,然后把后面的进行拼接

上面加了/,访问的是绝对路径,下面的没有加/访问的是相对路径,会出现404错误!注意!

![](../media/pictures/Spring.assets/dafcbd2e753aacae67a6d5423b6a52bf.png)

##### 逻辑视图

做的是viewName的拼接

/WEB-INF/view/stirng.jsp

![](../media/pictures/Spring.assets/8b4755aff510dcc8016ec28d4cd92470.png)

注意的点：如果新增了逻辑视图的配置，原先物理视图的返回值和modelAndView中的viewName需要调整

##### 请求

Forward redirect

对应的是请求之间的转发和重定向，也就是RequestMapping所对应方法之间的跳转

![](../media/pictures/Spring.assets/217385c75e9e985f86b6e886edef2812.png)

下面这块自己测试加的!

![](../media/pictures/Spring.assets/65686bab38dcf22e439d02042222cd7c.png)

### 请求参数的封装

#### Request（不推荐）

![](../media/pictures/Spring.assets/9785ac24470aeac2e1382d08c979c934.png)

#### 直接进行参数的封装

![](../media/pictures/Spring.assets/7299c7df6161d9e39d880475e7abb030.png)

#### 通过javabean进行封装

![](../media/pictures/Spring.assets/053a6bb1c7913dca4bdc85e8de273da7.png)

#### 嵌套javabean

![](../media/pictures/Spring.assets/d6e637ca36abdcd7524132e16e795081.png)

<http://localhost/parameter/direct?username=yanglei&password=123456&age=1&married=true&value=2&userDetail.email=abc@qq.com>

中间加了一个 “.”

#### 数组参数的接收

##### 直接接收

![](../media/pictures/Spring.assets/0e485e166beffe8a7b45c0b99d5ec2d9.png)

##### 通过作为javabean的成员变量

![](../media/pictures/Spring.assets/07f0d63cfdbdb4f02d96d32bffcedc25.png)

#### List

![](../media/pictures/Spring.assets/a46f6eaee5f4bf25613f63ddd8b613c0.png)

#### 字符编码（中文乱码）

![](../media/pictures/Spring.assets/df2f53fe711a3e86d843bbabe9835938.png)

#### 日期类型转换→转换器

![](../media/pictures/Spring.assets/81c0a2fce413bbec11f3de2dc7dfd1b9.png)

配置过程

![](../media/pictures/Spring.assets/cf7aad23b4a3d29faaac0d539633d381.png)

#### Fileupload

##### 导包

![](../media/pictures/Spring.assets/c127da00e97896c36753a75e5dcb1c76.png)

##### 请求

![](../media/pictures/Spring.assets/64fc537977fbee6e3c19f208ac2cffe0.png)

MultipartFile通过transferTo方法存储到我们的文件上

##### 组件的注册

![](../media/pictures/Spring.assets/d348acd21121833e43be2b28170c6700.png)

![](../media/pictures/Spring.assets/ab17c78e0d4eeb37a00bb948ed933188.png)





## 代码部分

#### SpringMVC流程

![1568807404918](C:\Users\Steve\Desktop\CS\Typora\images\1568807404918.png)

#### SpringMVC

##### 入门案例1

###### 导包

```xml
	//首先最上面需要写war包
	<packaging>war</packaging>
	
	/下面是所需要的两个依赖 servlet和webmvc
    <dependencies>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-webmvc</artifactId>
            <version>5.1.9.RELEASE</version>
        </dependency>
        <dependency>
            <groupId>javax.servlet</groupId>
            <artifactId>servlet-api</artifactId>
            <version>3.0-alpha-1</version>
            <scope>provided</scope>
        </dependency>
    </dependencies>


```

###### web.xml

```xml
<servlet>
        <servlet-name>dispatcherServlet</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <init-param>									这一句是加载spring配置文件
            <param-name>contextConfigLocation</param-name>
            <param-value>classpath:application.xml</param-value>
        </init-param>
</servlet>
<servlet-mapping>
        <servlet-name>dispatcherServlet</servlet-name>
        <url-pattern>/</url-pattern>  下面这里必须是/
</servlet-mapping>


```

###### Handler代码

```java
import org.springframework.web.servlet.ModelAndView;
import org.springframework.web.servlet.mvc.Controller;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
//这里要注意的是Controller是上面的包里面的,不是那个注解包
public class HelloController implements Controller {
    @Override
    public ModelAndView handleRequest(HttpServletRequest httpServletRequest, 						HttpServletResponse httpServletResponse) throws Exception {
        ModelAndView modelAndView = new ModelAndView();
        //model和数据相关
        modelAndView.addObject("content"," springmvc" );
        
        //跳转到哪一个视图,给这一个视图什么样的数据
        
        //相对于webroot(webapp)下的路径
        modelAndView.setViewName("/hello.jsp");   

        return modelAndView;
    }
}


```

##### 入门案例2

###### MVC注解驱动

application.xml里面要加的注册的映射器和适配器

```xml
<context:component-scan base-package="com.cskaoyan"/>
<mvc:annotation-driven/>


```

下面的是几个用到的映射器和适配器

![1568808158919](../media/pictures/Spring.assets/1568808158919.png)

###### 使用

直接在类上增加@Controller注解，则该类为Handler类

方法通过RequestMapping中的值和请求建立映射关系

```java
@Controller
public class Hello2Controller {
    @RequestMapping("/hello2")
    public ModelAndView hello2z(){
        ModelAndView modelAndView = new ModelAndView();
        //model和数据相关
        modelAndView.addObject("content","springmvc hello2");
        //相当于webroot(webapp)下的路径
        modelAndView.setViewName("WEB_INF/hello.jsp");

        return modelAndView;
    }

    @RequestMapping("/hello3")
    public ModelAndView hello2a(){
        ModelAndView modelAndView = new ModelAndView();
        //model和数据相关
        modelAndView.addObject("content"," springmvc hello3" );
        //相对于webroot(webapp)下的路径
        modelAndView.setViewName("/WEB-INF/hello.jsp");

        return modelAndView;
    }
}


```

##### RequestMapping的使用

下面有几种是用post来测试的,详细见老师上课讲义!

###### url路径映射

###### 窄化请求

路径映射和窄化请求是一个!窄化请求指示把公共路径user放到了外面,其实下面的RequestMapping的路径是

/user/insert

```java
@Controller
@RequestMapping("user")
public class UserController {   
    //窄化请求   
    @RequestMapping("/insert")   
    public ModelAndView insertUser(){ 
        System.out.println("insert User");
        ModelAndView modelAndView = new ModelAndView(); 
        //setViewName 
        modelAndView.setViewName("/WEB-INF/view/insert.jsp");
        return modelAndView;   
    }
}


```

###### 请求方法限定（405）

只能用Post方法

```java
@RequestMapping(value = "method/limit", method = RequestMethod.POST)
    public ModelAndView methodLimit(){

        ModelAndView modelAndView = new ModelAndView();
		//setViewName
        modelAndView.setViewName("/WEB-INF/view/limit.jsp");

        return modelAndView;
    }


```

可以是POST或者是GET, or的关系

```java
/*请求方法的限定 or*/
/*method:多个method之间的关系是or*/
@RequestMapping(value = "method/limit/multi", method = 					   			         					{RequestMethod.POST,RequestMethod.GET})
public ModelAndView methodLimitMulti(){
    ModelAndView modelAndView = new ModelAndView();
    modelAndView.setViewName("/WEB-INF/view/limit.jsp");

    return modelAndView;
}


```

下面这种虽然用的是get或者post,但是源码里面其实还是上面的情况

```java
 /*请求方法的限定 or*/  
 //当然这里只能是get方法,可以用postman测试POST,会报405错误   
 @GetMapping(value = "method/limit/get")   
 //@PostMapping(value = "method/limit/get")  
 public ModelAndView methodLimitGet(){       
 	ModelAndView modelAndView = new ModelAndView();   
	modelAndView.setViewName("/WEB-INF/view/limit.jsp");     
 	return modelAndView;   
 }


```

###### 请求参数的限定（400）

```java
/*参数中要包含*/
/*多个参数之间的关系是and*/  //这里的参数,可以指定必须等于多少,或者不等于多少,很灵活
//http://localhost/user/param/limit?username=yang&password=123456
@RequestMapping(value = "param/limit",params = {"username!=songge","password"})
public ModelAndView paramLimit(){
    ModelAndView modelAndView = new ModelAndView();
    modelAndView.setViewName("/WEB-INF/view/limit.jsp");
    modelAndView.addObject("value", "param limit");

    return modelAndView;
}


```

###### 请求的Content-type的限定（415）

```java
 /**
* 请求的Content-type限定  他的值只能是consumes后面的值 其他的都会报415错误
* @return
*/
@RequestMapping(value = "consume/limit",consumes = "application/json")
public ModelAndView cusumeLimit(){
    ModelAndView modelAndView = new ModelAndView();

    modelAndView.setViewName("/WEB-INF/view/limit.jsp");
    modelAndView.addObject("value", "consume limit");

    return modelAndView;
}

```

###### 请求Accept的限定（406）

```java
/*限定的是Accept*/  //Accept如果不是application/json,会报406错误
@RequestMapping(value = "produces/limit",produces = "application/json")
public ModelAndView producesLimit(){
    ModelAndView modelAndView = new ModelAndView();

    modelAndView.setViewName("/WEB-INF/view/limit.jsp");
    modelAndView.addObject("value", "produces limit");
    return modelAndView;
}

```

##### Controller对应方法的返回值(处理ModelAndView的时候)

###### Void

在形参中可以增加HttpServletRequest和HttpServletResponse

```java
//返回值为void所对应的方法
@RequestMapping("return/void")
public void returnVoid(HttpServletRequest request, HttpServletResponse response)
    throws ServletException, IOException {
    request.getRequestDispatcher("/WEB-INF/view/void.jsp").forward(request, response);
}

```

###### ModelAndView

以前的都是这种!略(^-^)

###### String(下属还有三个)

###### **物理视图:**

上面这个是加/

```java
@RequestMapping("return/string")
public String returnString(Model model){
    //返回值变为ModelAndView中的ViewName
    model.addAttribute("value","string");
    //viewName开头包含/的时候对应的是相对webroot（webapp）下的文件
    return "/WEB-INF/view/string.jsp";

}


```

下面的这个是没有加/   ,  就是WEB_INF前面的/

```java
@RequestMapping("return/string/noGang")
public String returnStringNoGang(Model model){
    //返回值变为ModelAndView中的ViewName
    model.addAttribute("value","string");
    //viewName开头包含/的时候对应的是相对webroot（webapp）下的文件
    return "WEB-INF/view/string.jsp";
    
    //noGang：相对于我们的请求：将我们请求的最后一级去掉 ，在和viewName进行拼接
    //在这个demo中对应的就是：return/string/WEB-INF/view/string.jsp
}


```

###### **逻辑视图**:

application.xml配置

```xml
<context:component-scan base-package="com.cskaoyan"/>
<mvc:annotation-driven/><bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">  
    <!--这两句只是给逻辑视图用的(LogicViewController,RequestStringController里面也用了),其他的方法如果也拼接,出不来结果-->    
    <property name="prefix" value="/WEB-INF/view/"/>   
    <property name="suffix" value=".jsp"/></bean>


```

```java
@Controller
public class LogicViewController {
    /**
     * 逻辑视图
     * 这里实实在在用到了application.xml里面的东西  里面的东西对现在的返回值做了一个拼接  打开以后会影		响其他的操作
     * 要有组件的注册 InternalResourceViewResolver
     * @return
     */
    @RequestMapping("return/logic")
    public String returnLogic(){

        //prefix+返回值+suffix
        //其实拼接完以后是这个 /WEB-INF/view/ stirng .jsp

        return "string";
    }
}


```

###### **请求**:

转发

```java
//这三个里面也用了前面的拼接
//地址栏没有发生变化
//转发
@RequestMapping("request/forward")
    public String requestForward(){
        System.out.println("forward");
        return "forward:/receive";
}


```

重定向

```java
//地址栏发生了变化,跳转到了下面的方法写的页面
//重定向
@RequestMapping("request/redirect")
public String requestRedirect(){
    System.out.println("redirect");
    return "redirect:/receive";
}

@RequestMapping("receive")
public String receive(){
    System.out.println("receive");
    return "receive";
}


```

下面是自己测试做的结果!

![1568810681031](../media/pictures/Spring.assets/1568810681031.png)

##### 请求参数的封装

详细的测试结果看老师上课讲义!

```java
//下面用到的bean对象,有数组,有集合
public class User {    
    String username;  
    String password;
    int age;  
    boolean married;  
    float value;   
    UserDetail userDetail; 
    String[] hobbys; 
    List<Car> carList; 
    Date date;
}


```

###### Request（不推荐）

```java
//通过Request进行请求参数的封装
@RequestMapping("parameter/request")
public String getRequestParameter(HttpServletRequest request, Model model){
    String username = request.getParameter("username");
    String password = request.getParameter("password");
    model.addAttribute("username", username);
    model.addAttribute("password",password);
    return "/WEB-INF/view/request.jsp";
}


```

###### 直接进行参数的封装

```java
@RequestMapping("parameter/direct")
public String getDirectParameter(String username,String password,int age,boolean 		     		married,float value,Model model){
    model.addAttribute("username", username);
    model.addAttribute("password",password);
    return "/WEB-INF/view/request.jsp";
}


```

###### 通过javabean进行封装

```java
@RequestMapping("parameter/javabean")  
public String getBeanParameter(User user, Model model){   
    System.out.println(user);     
    return "/WEB-INF/view/request.jsp";    
}


```

###### 嵌套javabean

上面!老师讲义有详细测试过程!

![1568811384245](../media/pictures/Spring.assets/1568811384245.png)

###### 数组参数的接收

###### 直接接收

```java
@RequestMapping("parameter/array")
public String getArrayParameter(String[] hobbys, Model model){
	for (String hobby : hobbys) {
	System.out.println(hobby);
}
	return "/WEB-INF/view/request.jsp";
}


```

###### ![1568811442895](../media/pictures/Spring.assets/1568811442895.png)通过作为javabean的成员变量

```java
//这个是用bean来接收数组
@RequestMapping("/parameter/javabean/array")
public String getArrayInBeanParameter(User userz){ //这里userz这个名字 可以随便起   
	return "/WEB-INF/view/request.jsp";
}


```

###### List

看上面的bean,只要在bean中加一个属性,这里就可以接收好多东西!

```java
@RequestMapping("/parameter/javabean/list")
public String getListInBeanParameter(User userz){
    return "/WEB-INF/view/request.jsp";
}


```

![1568811742904](../media/pictures/Spring.assets/1568811742904.png)

###### 字符编码（中文乱码）

```xml
<filter>
    <filter-name>characterEncodingFilter</filter-name>
    <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
        <init-param>
            <param-name>encoding</param-name>
            <param-value>utf-8</param-value>
        </init-param>
</filter>
<filter-mapping>
    <filter-name>characterEncodingFilter</filter-name>
    <url-pattern>/*</url-pattern>
</filter-mapping>


```

![1568811753301](../media/pictures/Spring.assets/1568811753301.png)

###### 日期类型转换→转换器

![1568811846143](../media/pictures/Spring.assets/1568811846143.png)

```xml
<!--这个日期类  -->
<bean id="customConversionService" class="org.springframework.format.support.FormattingConversionServiceFactoryBean">
    <property name="converters">
        <set>
            <bean class="com.cskaoyan.converter.String2DateConverter"/>
        </set>
    </property>
</bean>


```

```java
@RequestMapping("parameter/javabean/date")
public String getDateInJavabeanParameter(User user){
    return "/WEB-INF/view/request.jsp";
}


```

配置过程

![1568811861475](../media/pictures/Spring.assets/1568811861475.png)

```java
public class String2DateConverter implements  Converter<String,Date>{

    @Override
    public Date convert(String s) {
        //转换成date类型的业务逻辑2019-9-18
        SimpleDateFormat simpleDateFormat = new SimpleDateFormat("yyyy-MM-dd");
        try{
            Date parse = simpleDateFormat.parse(s);
            return parse;
        }catch (ParseException e){
            e.printStackTrace();
        }
        return null;
    }
}


```

###### Fileupload

###### 导包

文件上传所需要的依赖

```xml
<!--文件上传的支持-->
<dependency>
    <groupId>commons-io</groupId>
    <artifactId>commons-io</artifactId>
    <version>2.6</version>
</dependency>
<dependency>
    <groupId>commons-fileupload</groupId>
    <artifactId>commons-fileupload</artifactId>
    <version>1.3.3</version>
</dependency>


```

###### 请求

jsp中代码,前面的${}里面的路径是tomcat的配置路径,现在用的是/,所以写不写都行

```jsp
<form action="${pageContext.request.contextPath}/file/upload" enctype="multipart/form-data" method="post">
    <input type="file" name="myfile"><input type="submit">
</form>


```

```java
@Controller
public class FileUploadController {
    @RequestMapping("file/upload")
    public String fileUpload(MultipartFile myfile, User user) throws IOException {
        //myfile.getOriginalFilename()
        File file = new File("d://myfile.jpg");
        myfile.transferTo(file);
        return "/WEB-INF/view/success.jsp";
    }
}


```

![1568812178352](../media/pictures/Spring.assets/1568812178352.png)

MultipartFile通过transferTo方法存储到我们的文件上

###### 组件的注册

上面的id是固定的,只写这个,下面的限制文件大小的!

```xml
<!--id一定要写multipartResolver-->
<!--下面的value是限制上传文件的大小的 5*1024*1024-->
<bean id="multipartResolver" class="org.springframework.web.multipart.commons.CommonsMultipartResolver">
    <property name="maxUploadSize" value="5242880"/>
</bean>


```



# 五.Rest,Interceptor,Json

## REST

### 获得请求url中的参数

![](../media/pictures/Spring.assets/53344fa94acd409bfdc0e0ad374d950d.png)

### 获得请求头（RequestHeaders）中的值

![](../media/pictures/Spring.assets/d8b8563974b63c743ba4d16e34f471de.png)

### 限定请求参数

![](../media/pictures/Spring.assets/05575d482a60630c5d5764b8ff69ac58.png)

## 静态资源的处理

### Default-servlet

让默认的servlet去处理静态资源

![](../media/pictures/Spring.assets/6a9a065058dc0d399cadbfcd24477269.png)

### Default servlet handler

Springmvc 提供了一种方式让默认的servlet去处理静态资源

![](../media/pictures/Spring.assets/257e0c0904c02b4814082e53ab6e4887.png)

### Springmvc的静态资源处理

Webroot：

Classpath：

File：

![](../media/pictures/Spring.assets/a4e95e89e86cb5c25528b9827f6d6d9f.png)

![](../media/pictures/Spring.assets/69773bb47a0cbf96ef2a9cbff994f0bc.png)

## Interceptor

![](../media/pictures/Spring.assets/4797c45b373decb5a75843d814a431c7.png)

### HandlerInterceptor接口

#### preHandle

返回值是boolean，当为true可以执行到Handler（RequestMapping所对应的的犯法），如果为false则执行不到。

当前类的prehandle返回值为true，一定可以执行到它所对应的afterCompletion

![](../media/pictures/Spring.assets/6efe79ba8d68c34d4a5ff8a5607aaddb.png)

#### PostHandle

执行是在Handler返回ModelAndView给dispatcherServlet之间执行

形参中包含一个ModelAndView，来源于\@RequestMapping所对应的的Handler，并且可以对ModelAndView做进一步的处理

![](../media/pictures/Spring.assets/148d218aad7b9f013fd6fdcbd9183ca2.png)

#### afterCompletion

在dispatcherServlet处理完ModelAndView之后执行，通常做一些流的关闭

![](../media/pictures/Spring.assets/7051cb3defb5cb71f979819d2640d1e5.png)

### 自定义的Handler

![](../media/pictures/Spring.assets/010d506e0170af76e4ad67048ac891fe.png)

### 配置interceptor

![](../media/pictures/Spring.assets/9060345451ae82124a888805d7ac18f5.png)

### 多个interceptor的执行关系

#### 全部通过

![](../media/pictures/Spring.assets/502caa4a2ae0eb99bf15956896f8c60c.png)

#### 多个interceptor，当第一个preHandle返回false

下面所有的流程都走不到

![](../media/pictures/Spring.assets/2b49b8898f642d9a2e6c4eb7260c21da.png)

#### 多个interceptor第一个为true，第二个为false

![](../media/pictures/Spring.assets/e6699fb3ad3dca533f95674eef3b871d.png)

#### 结论

当prehandle返回值为true，嵌套结构这部分可以继续往下走，

并且当prehandle返回值为true，一定可以指定到其对应的afterCompletion

### Interceptor的作用范围

拦截的请求

![](../media/pictures/Spring.assets/06a49fa8509360c10980e5347377a06b.png)

## 异常处理器

### 自定义的异常的实现

通过继承异常类实现自定义的异常

![](../media/pictures/Spring.assets/e5fdca7143d2fb7beed8fac2e757117b.png)

### 注册自定义的异常处理器

![](../media/pictures/Spring.assets/c2e50abe000c8c1acb45ed3d5ed72912.png)

## Json

接收前端发送的json数据

返回json数据给到前端

### 导包

一拖三

Jackson core databind annotation

![](../media/pictures/Spring.assets/d9440abe4d4c5d53ee58f460a23d4761.png)

### 使用json

\@ResponseBody 以json数据的形式返回值

\@RequestBody 接收json数据

\@RestController = \@Controller + \@ResponseBody 当前类下全部方法都返回json数据

![](../media/pictures/Spring.assets/2ce90f43d45dccc2c10b7f0476d6fbb9.png)

### 构造一个json请求

![](../media/pictures/Spring.assets/f78fbce4a4ddaf627184595c80939dc3.png)

### 补充

#### 同一个url映射到不同的方法

通过限定请求方法

![](../media/pictures/Spring.assets/08162cecbd4bafa5e0c4f897bf12ad21.png)

#### 多个url映射到同一个方法上

![](../media/pictures/Spring.assets/7a158d5c1e9b5313271c171c5f86d7c8.png)

# 六.FreeMarker

## 异常处理器（Json）

![](../media/pictures/Spring.assets/ad91655fe31a621d8ffa9c06fc3bc328.png)

## Freemarker

### 入门案例

![](../media/pictures/Spring.assets/e303c6cba4cdf55f816f5e87ddd2ef9a.png)

### 表达式

通过向我们的数据集（map）中放不同类型的数据，使用模板生成这些数据

#### 访问map中的key

#### 访问pojo中的属性

![](../media/pictures/Spring.assets/132c5d116450f3d6654dc54a94ddea91.png)

#### 取集合中的数据

#### 取循环中的下标

![](../media/pictures/Spring.assets/8a78e1544b3c0ca7c187ad31c016b56b.png)

#### 判断

![](../media/pictures/Spring.assets/bbc380a861a3944e3afb3f49983b92be.png)

#### 特殊值的处理

![](../media/pictures/Spring.assets/efbce7ec2365a3e07954d41a7a88b7e0.png)

日期

![](../media/pictures/Spring.assets/44d0a0a7605d4ec5f24abb0695f3a135.png)

#### include

![](../media/pictures/Spring.assets/17e49e14401cbb50bce8b3a0f54f6b31.png)

### springmvc中使用Freemarker

#### 导包

Freemark

![](../media/pictures/Spring.assets/7e04e3c3df49c07b024a5881e9691c81.png)

#### 组件注册

![](../media/pictures/Spring.assets/1a04115496e8f6a9604bd013454ea275.png)

#### 使用

和写jsp是一样的

![](../media/pictures/Spring.assets/605dd4af398e08e3e425b7e0ac91a4c2.png)

### 多个视图解析器共存

#### 通过解析顺序

通过注册顺序，一个是bean标签写在前面，一种通过order属性

![](../media/pictures/Spring.assets/f12a1e932f31df9069640be9814b9424.png)

![](../media/pictures/Spring.assets/15f0d65987fa819396d8231f9e1d38f2.png)

#### 通过viewName进行分发

通过配置viewNames属性，将不同的viewName所对应的ModelAndView交给不同的处理器

![](../media/pictures/Spring.assets/f27d90266d4abad64559c540aa28a64a.png)

![](../media/pictures/Spring.assets/67f40c17f08d36c490bcbd52caa23cf4.png)

# 七:SSM

## SSM整合

## Spring和mybatis的整合

## SSM（XML）

### 导包

Spring-webmvc 5+2+1

Spring-jdbc（tx）

Mybatis-spring

Mybatis

Mysql-connector-java

Druid

Javax.servlet（provided）

Json：jackson-databind

![](../media/pictures/Spring.assets/d0384c2d459a1c89d5c6f3d14ad9522d.png)

### 配置组件

#### Spring配置文件

Mybatis：3个组件→sqlSessionFactoryBean、datasource、MapperScannerConfigurer

![](../media/pictures/Spring.assets/5229d9532fa1df74342e344e974f8f7a.png)

事务：datasourceTransactionManager、tx:annotation-driven

![](../media/pictures/Spring.assets/4e2069e2f589f95afd3afe9690fa1636.png)

Spring：context：component-scan （exclude-filter）

![](../media/pictures/Spring.assets/4de0e03f5b52e9744731557057f2c296.png)

#### Springmvc配置文件

Springmvc：context:component-scan、mvc:annotation-driven

![](../media/pictures/Spring.assets/b92c685f81aea5128194a548fec25630.png)

#### Web.xml

共同通过contextConfigLocation

contextLoaderListener通过context-param加载spring配置文件

dispatcherServlet通过init-param加载springmvc的配置文件

![](../media/pictures/Spring.assets/63254e4e2181891cf19636d2a9b75c1f.png)

## SSM（java config）

### 导包

同上

### 启动类AACDSI

Spring配置类、Springmvc配置类、url-pattern

![](../media/pictures/Spring.assets/b7a68c920e4912ae1c3d2adefa66a395.png)

### Spring配置类

注解：\@ComponentScan、\@EnableTransactionManagement、\@Configuration

![](../media/pictures/Spring.assets/6f7fcba728a04eb3c164768987a00e5b.png)

组件注册：\@Bean→
SqlSessionFactoryBean、MapperScannerConfigurer、datasource、DataSourceTransactionManager

![](../media/pictures/Spring.assets/ac1a9e0f58a8fa1c838cb09406916e63.png)

### Springmvc配置类

注解：\@EnableWebmvc

\@ComponentScan

Implements WebmvcConfigurer

![](../media/pictures/Spring.assets/04de2df6793a1acb903d06dba3bd7cc0.png)

# 八:SpringBoot

## Springboot

Start.spring.io

Idea

## Springboot使用

### 简单配置项

Serverport

Context-path    

​		这个配置是在application.properties中的配置! 后来常用的还是Yml的配置 

![](../media/pictures/Spring.assets/1c0a6cff765ab7e94bf46dd52aeb76f2.png)

​		这个SpringBoot在配置完使用的时候,**controller层**,在创建项目会自动创建!不用自动创建,目录级别可能会和以前的spring的目录有一点不一样!



​		配置项目的时候,不要多选,需要啥拿啥!

### Yml

Yaml

datasource.username=root

datasource.password=123456

datasource.driverClassname=com.mysql.jdbc.Driver

datasource.url=

datasource:

​	username: root

​	password: 123456

url:

properties配置文件的“.”.对应yml的    **:和换行**

properties配置文件的”=”对应yml的    **:和空格**

![](../media/pictures/Spring.assets/cdfff16e967a2457f06d7385390be1f9.png)

### ConfigurationProperties(非常重要的一个)

#### @value

![](../media/pictures/Spring.assets/9d6927bba9abd0d35aa3e58865d50fbe.png)

#### ConfigurationProperties

提示需要的依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-configuration-processor</artifactId>
    <optional>true</optional>
</dependency>

```

**导包以后,在yml中如果没有提示,手动编译一下,然后就会有提示!**

![](../media/pictures/Spring.assets/c0dad0308e764215cd9646fd2fcb1b9f.png)

​		左边之所以小写,是因为把右边的代码大写字母换成了  **-加小写字母**!

### PropertySource

引入额外的springboot配置文件，properties格式的

```java
//要在使用这个配置文件的上面加这个注解,然后就可以使用了!
@propertySource("classpath:abc.properties")
public class Demo1{
    
}

```

### ImportSource

引入额外的spring配置文件，xml格式的

![](../media/pictures/Spring.assets/9e832fc0a104a0cfa2166d31a008abda.png)

### 占位符

使用的好处是,是需要修改上面的file-path,下面占位符的内容也会变!

![](../media/pictures/Spring.assets/8a55d9c283b68cfe9220eb66f9344bc4.png)

## 补充ConfigurationProperties赋值

String、int、boolean、List、Map

Yml：

![](../media/pictures/Spring.assets/67629e94d62a802773ac2690b9085d86.png)

其中list中间用,隔开! 

Properties：

![](../media/pictures/Spring.assets/884e813e32f8e40c4e944b7b4c2cf580.png)

## Profiles

多配置文件

**不同环境、不同组件的配置**

### Properties

![](../media/pictures/Spring.assets/e50406459038ba71d07b2ef6a4d5df84.png)

通过文件命名的形式区分多个配置文件

激活使用**spring.profiles.active**进行**激活1个或多个配置**文件

![](../media/pictures/Spring.assets/cac98f228b2c348668699f4391607454.png)

### Yml

同样可以使用文件命名的方式区分多个配置文件

![](../media/pictures/Spring.assets/31573ecd9250362728a9476c7e4e1ed9.png)

另外yml提供了一种额外的方式分隔多个配置文件

在配置文件中**新增三个杠“---”**，并且在分隔符中通过spring.profiles=“名字”的形式表明自己的配置文件名

![](../media/pictures/Spring.assets/280f8c5fc239435e7919d23aba7f1a48.png)

## 配置文件的加载顺序（主要是运维）

打包项目,点击项目旁边的package

![1569641029147](../media/pictures/Spring.assets/1569641029147.png)

启动jar包:在moni(模拟环境下),cmd进去,然后输入java -jar demo3,jar  就可以启动jar包!

jar包的启动1,实际上是相当于启动jar包里面的main方法!

![1569641437688](../media/pictures/Spring.assets/1569641437688.png)



1:**File:/Config**

2:**File:/**

3:**Classpath:/Config**

4:**Classpath:/**

![](../media/pictures/Spring.assets/7300a256e1ec0596f364502ce3adef9d.png)



## 命令行启动

**在命令行中配置的启动参数优先级是最高的，凌驾于上面的配置文件**



其实上面的四个优先级,不是全部的优先级!老笔记里面有,总共的优先级有十几种! 

其中最高级别的优先级是命令行,二进制的形式! 比上面是四种的有限级别都高的多!

## Web(freemarker)

### Servlet

1. webServlet注解和ServletComponentScan搭配使用

![](../media/pictures/Spring.assets/d16f75499ed872485ae65cbbaa9e787c.png)

1. 注册ServletRegistrationBean组件

![](../media/pictures/Spring.assets/7f71340e5d0a21e0dfcc933f3f32fc95.png)

### Filter    

filter也是两种形式,一种是写注解的形式,另外一种是注册bean的形式

![](../media/pictures/Spring.assets/97bb9f2c0ac4faee472f61955ca6e16c.png)



这里出现一个问题,只要加了filter,就出错 

### Listener

![](../media/pictures/Spring.assets/972206c8e54b8225eb4e96b89c5c820f.png)

## Mybatis

​	mybatis在sptingBoot自动生成mysql的版本是8.几的版本,所以这里要自己将对应的版本改到5.1.47!

![1569670346731](../media/pictures/Spring.assets/1569670346731.png)

这是修改以后的版本!

```xml
<dependency>
     <groupId>mysql</groupId>
     <artifactId>mysql-connector-java</artifactId>
     <version>5.1.47</version>
     <scope>runtime</scope>
</dependency>

```

springboot默认会有一个数据库连接池,不导入的话也是可以用的!

如果想用阿里的druid,需要加一句话:

```yml
#数据源
#扫描包
spring:
  datasource:
    driver-class-name: com.mysql.jdbc.Driver
    url: jdbc:mysql://localhost:3306/mybatis?useUnicode=true&characterEncoding=utf-8
    password: 123456
    username: root
    #springboot默认有一个连接池  这里想用druid,所以,要单独导入
    type: com.alibaba.druid.pool.DruidDataSource

```



## 自动配置的原理

约定大于配置

自动配置类

![](../media/pictures/Spring.assets/f8f909ae1d4585a1090965ea4353ade2.png)

这是在看源码中找的! 没听懂!

去这个文件下找自动配置类

Conditional

![](../media/pictures/Spring.assets/18227822563cee586d9e83989cc83d9c.png)

按需导入

![](../media/pictures/Spring.assets/1d1c610f08e4f5873924fe926c46b613.png)

这两个注解是看源码的入口!

![1569672925718](../media/pictures/Spring.assets/1569672925718.png)

![1569672739907](../media/pictures/Spring.assets/1569672739907.png)

凡是能自动导入的,在原码中都是可以找到的!

## SpringBoot中使用log

参考springboot-日志配置文档

老师缺了这块笔记!

https://www.cnblogs.com/bigdataZJ/p/springboot-log.html

网上的博客有好多!



## 彩蛋

banner  

https://www.cnblogs.com/huanzi-qch/p/9916784.html

文章里面好多可以用图片或者其他字体转换成的字符!

http://patorjk.com/software/taag

http://www.network-science.de/ascii/

http://www.degraeve.com/img2txt.php

三个常用的网站!可以生成想要的东西!





# 程序羊

## Spring 经典面试题汇总

[CodeSheep](javascript:void(0);) *今天*



![img](../media/pictures/Spring.assets/640.webp)



## **1、基础概念**

**1.1. 不同版本的 Spring Framework 有哪些主要功能？**

Version                    Feature

| Spring 2.5 | 发布于 2007 年。这是第一个支持注解的版本。                   |
| :--------- | :----------------------------------------------------------- |
| Spring 3.0 | 发布于 2009 年。它完全利用了 Java5 中的改进，并为 JEE6 提供了支持。 |
| Spring 4.0 | 发布于 2013 年。这是第一个完全支持 JAVA8 的版本。            |

**1.2. 什么是 Spring Framework？**

Spring 是一个开源应用框架，旨在降低应用程序开发的复杂度。

它是轻量级、松散耦合的。

它具有分层体系结构，允许用户选择组件，同时还为 J2EE 应用程序开发提供了一个有凝聚力的框架。

它可以集成其他框架，如 Structs、Hibernate、EJB 等，所以又称为框架的框架。

**1.3. 列举 Spring Framework 的优点。**

由于 Spring Frameworks 的分层架构，用户可以自由选择自己需要的组件。

Spring Framework 支持 POJO(Plain Old Java Object) 编程，从而具备持续集成和可测试性。

由于依赖注入和控制反转，JDBC 得以简化。

它是开源免费的。

**1.4. Spring Framework 有哪些不同的功能？**

轻量级 - Spring 在代码量和透明度方面都很轻便。

IOC - 控制反转

AOP - 面向切面编程可以将应用业务逻辑和系统服务分离，以实现高内聚。

容器 - Spring 负责创建和管理对象（Bean）的生命周期和配置。

MVC - 对 web 应用提供了高度可配置性，其他框架的集成也十分方便。

事务管理 - 提供了用于事务管理的通用抽象层。Spring 的事务支持也可用于容器较少的环境。

JDBC 异常 - Spring 的 JDBC 抽象层提供了一个异常层次结构，简化了错误处理策略。

**1.5. Spring Framework 中有多少个模块，它们分别是什么？**

![img](D:/Code/Typora/media/pictures/Spring.assets/640.webp)

Spring 核心容器 – 该层基本上是 Spring Framework 的核心。它包含以下模块：

- Spring Core
- Spring Bean
- SpEL (Spring Expression Language)
- Spring Context

数据访问/集成 – 该层提供与数据库交互的支持。它包含以下模块：

- JDBC (Java DataBase Connectivity)
- ORM (Object Relational Mapping)
- OXM (Object XML Mappers)
- JMS (Java Messaging Service)
- Transaction

Web – 该层提供了创建 Web 应用程序的支持。它包含以下模块：

- Web
- Web – Servlet
- Web – Socket
- Web – Portlet

**AOP** – 该层支持面向切面编程

**Instrumentation** – 该层为类检测和类加载器实现提供支持。

**Test** – 该层为使用 JUnit 和 TestNG 进行测试提供支持。

几个杂项模块:

- Messaging – 该模块为 STOMP 提供支持。它还支持注解编程模型，该模型用于从 WebSocket 客户端路由和处理 STOMP 消息
- Aspects – 该模块为与 AspectJ 的集成提供支持。

**1.6. 什么是 Spring 配置文件？**

Spring 配置文件是 XML 文件。该文件主要包含类信息。它描述了这些类是如何配置以及相互引入的。但是，XML 配置文件冗长且更加干净。如果没有正确规划和编写，那么在大项目中管理变得非常困难。

**1.7. Spring 应用程序有哪些不同组件？**

Spring 应用一般有以下组件：

**接口** - 定义功能。

**Bean 类** - 它包含属性，setter 和 getter 方法，函数等。

**Spring 面向切面编程（AOP）** - 提供面向切面编程的功能。

**Bean 配置文件** - 包含类的信息以及如何配置它们。

**用户程序** - 它使用接口。

**1.8. 使用 Spring 有哪些方式？**

使用 Spring 有以下方式：

- 作为一个成熟的 Spring Web 应用程序。
- 作为第三方 Web 框架，使用 Spring Frameworks 中间层。
- 用于远程使用。
- 作为企业级 Java Bean，它可以包装现有的 POJO（Plain Old Java Objects）。

## **2、依赖注入（Ioc）**

**2.1. 什么是 Spring IOC 容器？**

Spring 框架的核心是 Spring 容器。容器创建对象，将它们装配在一起，配置它们并管理它们的完整生命周期。Spring 容器使用依赖注入来管理组成应用程序的组件。

容器通过读取提供的配置元数据来接收对象进行实例化，配置和组装的指令。该元数据可以通过 XML，Java 注解或 Java 代码提供。

![img](D:/Code/Typora/media/pictures/Spring.assets/640.webp)img

**2.2. 什么是依赖注入？**

在依赖注入中，您不必创建对象，但必须描述如何创建它们。您不是直接在代码中将组件和服务连接在一起，而是描述配置文件中哪些组件需要哪些服务。由 IoC 容器将它们装配在一起。

**2.3. 可以通过多少种方式完成依赖注入？**

通常，依赖注入可以通过三种方式完成，即：

- 构造函数注入
- setter 注入
- 接口注入

在 Spring Framework 中，仅使用构造函数和 setter 注入。

**2.4. 区分构造函数注入和 setter 注入。**

| 构造函数注入               | setter 注入                |
| :------------------------- | :------------------------- |
| 没有部分注入               | 有部分注入                 |
| 不会覆盖 setter 属性       | 会覆盖 setter 属性         |
| 任意修改都会创建一个新实例 | 任意修改不会创建一个新实例 |
| 适用于设置很多属性         | 适用于设置少量属性         |

**2.5. spring 中有多少种 IOC 容器？**

- BeanFactory - BeanFactory 就像一个包含 bean 集合的工厂类。它会在客户端要求时实例化 bean。
- ApplicationContext - ApplicationContext 接口扩展了 BeanFactory 接口。它在 BeanFactory 基础上提供了一些额外的功能。

**2.6. 区分 BeanFactory 和 ApplicationContext。**

| BeanFactory                | ApplicationContext       |
| :------------------------- | :----------------------- |
| 它使用懒加载               | 它使用即时加载           |
| 它使用语法显式提供资源对象 | 它自己创建和管理资源对象 |
| 不支持国际化               | 支持国际化               |
| 不支持基于依赖的注解       | 支持基于依赖的注解       |

**2.7. 列举 IoC 的一些好处。**

IoC 的一些好处是：

- 它将最小化应用程序中的代码量。
- 它将使您的应用程序易于测试，因为它不需要单元测试用例中的任何单例或 JNDI 查找机制。
- 它以最小的影响和最少的侵入机制促进松耦合。
- 它支持即时的实例化和延迟加载服务。

**2.8. Spring IoC 的实现机制。**

Spring 中的 IoC 的实现原理就是工厂模式加反射机制。

示例：

```java
interface Fruit {
     public abstract void eat();
}
class Apple implements Fruit {
    public void eat(){
        System.out.println("Apple");
    }
}
class Orange implements Fruit {
    public void eat(){
        System.out.println("Orange");
    }
}
class Factory {
    public static Fruit getInstance(String ClassName) {
        Fruit f=null;
        try {
            f=(Fruit)Class.forName(ClassName).newInstance();
        } catch (Exception e) {
            e.printStackTrace();
        }
        return f;
    }
}
class Client {
    public static void main(String[] a) {
        Fruit f=Factory.getInstance("io.github.dunwu.spring.Apple");
        if(f!=null){
            f.eat();
        }
    }
}

```

## **3、Beans**

**3.1. 什么是 spring bean？**

- 它们是构成用户应用程序主干的对象。
- Bean 由 Spring IoC 容器管理。
- 它们由 Spring IoC 容器实例化，配置，装配和管理。
- Bean 是基于用户提供给容器的配置元数据创建。

**3.2. spring 提供了哪些配置方式？**

- **基于 xml 配置**

bean 所需的依赖项和服务在 XML 格式的配置文件中指定。这些配置文件通常包含许多 bean 定义和特定于应用程序的配置选项。它们通常以 bean 标签开头。例如：

```xml
<bean id="studentbean" class="org.edureka.firstSpring.StudentBean"> <property name="name" value="Edureka"></property></bean>

```

- **基于注解配置**

您可以通过在相关的类，方法或字段声明上使用注解，将 bean 配置为组件类本身，而不是使用 XML 来描述 bean 装配。默认情况下，Spring 容器中未打开注解装配。因此，您需要在使用它之前在 Spring 配置文件中启用它。例如：

```xml
<bean id="studentbean" class="org.edureka.firstSpring.StudentBean">
 <property name="name" value="Edureka"></property>
</bean>

```

- **基于 Java API 配置**

Spring 的 Java 配置是通过使用 @Bean 和 @Configuration 来实现。

1. @Bean 注解扮演与 <bean /> 元素相同的角色。
2. @Configuration 类允许通过简单地调用同一个类中的其他 @Bean 方法来定义 bean 间依赖关系。

例如：

```java
@Configuration
public class StudentConfig {
    @Bean
    public StudentBean myStudent() {
        return new StudentBean();
    }
}

```

**3.3. spring 支持集中 bean scope？**

Spring bean 支持 5 种 scope：

- Singleton - 每个 Spring IoC 容器仅有一个单实例。
- Prototype - 每次请求都会产生一个新的实例。
- Request - 每一次 HTTP 请求都会产生一个新的实例，并且该 bean 仅在当前 HTTP 请求内有效。
- Session - 每一次 HTTP 请求都会产生一个新的 bean，同时该 bean 仅在当前 HTTP session 内有效。
- Global-session - 类似于标准的 HTTP Session 作用域，不过它仅仅在基于 portlet 的 web 应用中才有意义。Portlet 规范定义了全局 Session 的概念，它被所有构成某个 portlet web 应用的各种不同的 portlet 所共享。在 global session 作用域中定义的 bean 被限定于全局 portlet Session 的生命周期范围内。如果你在 web 中使用 global session 作用域来标识 bean，那么 web 会自动当成 session 类型来使用。

仅当用户使用支持 Web 的 ApplicationContext 时，最后三个才可用。更多spring内容

**3.4. spring bean 容器的生命周期是什么样的？**

spring bean 容器的生命周期流程如下：

1. Spring 容器根据配置中的 bean 定义中实例化 bean
2. Spring 使用依赖注入填充所有属性，如 bean 中所定义的配置。
3. 如果 bean 实现 BeanNameAware 接口，则工厂通过传递 bean 的 ID 来调用 setBeanName()。
4. 如果 bean 实现 BeanFactoryAware 接口，工厂通过传递自身的实例来调用 setBeanFactory()。
5. 如果存在与 bean 关联的任何 BeanPostProcessors，则调用 preProcessBeforeInitialization() 方法。
6. 如果为 bean 指定了 init 方法（ <bean>的 init-method 属性），那么将调用它。
7. 最后，如果存在与 bean 关联的任何 BeanPostProcessors，则将调用 postProcessAfterInitialization() 方法。
8. 如果 bean 实现 DisposableBean 接口，当 spring 容器关闭时，会调用 destory()。
9. 如果为 bean 指定了 destroy 方法（ <bean>的 destroy-method 属性），那么将调用它。

![img](../media/pictures/Spring.assets/640.png)img

**3.5. 什么是 spring 的内部 bean？**

只有将 bean 用作另一个 bean 的属性时，才能将 bean 声明为内部 bean。为了定义 bean，Spring 的基于 XML 的配置元数据在<property>或<constructor-arg> 中提供了<bean>元素的使用。内部 bean 总是匿名的，它们总是作为原型。

例如，假设我们有一个 Student 类，其中引用了 Person 类。这里我们将只创建一个 Person 类实例并在 Student 中使用它。

Student.java

```java
public class Student {
    private Person person;
    //Setters and Getters
}
public class Person {
    private String name;
    private String address;
    //Setters and Getters
}

```

bean.xml

```xml
<bean id=“StudentBean" class="com.edureka.Student">
    <property name="person">
        <!--This is inner bean -->
        <bean class="com.edureka.Person">
            <property name="name" value=“Scott"></property>
            <property name="address" value=“Bangalore"></property>
        </bean>
    </property>
</bean>

```

**3.6. 什么是 spring 装配**

当 bean 在 Spring 容器中组合在一起时，它被称为装配或 bean 装配。Spring 容器需要知道需要什么 bean 以及容器应该如何使用依赖注入来将 bean 绑定在一起，同时装配 bean。

**3.7. 自动装配有哪些方式？**

Spring 容器能够自动装配 bean。也就是说，可以通过检查 BeanFactory 的内容让 Spring 自动解析 bean 的协作者。

自动装配的不同模式：

- no - 这是默认设置，表示没有自动装配。应使用显式 bean 引用进行装配。
- byName - 它根据 bean 的名称注入对象依赖项。它匹配并装配其属性与 XML 文件中由相同名称定义的 bean。
- byType - 它根据类型注入对象依赖项。如果属性的类型与 XML 文件中的一个 bean 名称匹配，则匹配并装配属性。
- 构造函数 - 它通过调用类的构造函数来注入依赖项。它有大量的参数。
- autodetect - 首先容器尝试通过构造函数使用 autowire 装配，如果不能，则尝试通过 byType 自动装配。

**3.8. 自动装配有什么局限？**

- 覆盖的可能性 - 您始终可以使用<constructor-arg> 和 <property>设置指定依赖项，这将覆盖自动装配。
- 基本元数据类型 - 简单属性（如原数据类型，字符串和类）无法自动装配。
- 令人困惑的性质 - 总是喜欢使用明确的装配，因为自动装配不太精确。

## **4、注 解**

**4.1. 你用过哪些重要的 Spring 注解？**

- @Controller - 用于 Spring MVC 项目中的控制器类。
- @Service - 用于服务类。
- @RequestMapping - 用于在控制器处理程序方法中配置 URI 映射。
- @ResponseBody - 用于发送 Object 作为响应，通常用于发送 XML 或 JSON 数据作为响应。
- @PathVariable - 用于将动态值从 URI 映射到处理程序方法参数。
- @Autowired - 用于在 spring bean 中自动装配依赖项。
- @Qualifier - 使用 @Autowired 注解，以避免在存在多个 bean 类型实例时出现混淆。
- @Scope - 用于配置 spring bean 的范围。
- @Configuration，@ComponentScan 和 @Bean - 用于基于 java 的配置。
- @Aspect，@Before，@After，@Around，@Pointcut - 用于切面编程（AOP）。

**4.2. 如何在 spring 中启动注解装配？**

默认情况下，Spring 容器中未打开注解装配。因此，要使用基于注解装配，我们必须通过配置<context：annotation-config /> 元素在 Spring 配置文件中启用它。

**4.3. @Component, @Controller, @Repository, @Service 有何区别？**

- @Component：这将 java 类标记为 bean。它是任何 Spring 管理组件的通用构造型。spring 的组件扫描机制现在可以将其拾取并将其拉入应用程序环境中。
- @Controller：这将一个类标记为 Spring Web MVC 控制器。标有它的 Bean 会自动导入到 IoC 容器中。
- @Service：此注解是组件注解的特化。它不会对 @Component 注解提供任何其他行为。您可以在服务层类中使用 @Service 而不是 @Component，因为它以更好的方式指定了意图。
- @Repository：这个注解是具有类似用途和功能的 @Component 注解的特化。它为 DAO 提供了额外的好处。它将 DAO 导入 IoC 容器，并使未经检查的异常有资格转换为 Spring DataAccessException。

**4.4. @Required 注解有什么用？**

@Required 应用于 bean 属性 setter 方法。此注解仅指示必须在配置时使用 bean 定义中的显式属性值或使用自动装配填充受影响的 bean 属性。如果尚未填充受影响的 bean 属性，则容器将抛出 BeanInitializationException。

示例：

```java
public class Employee {
    private String name;
    @Required
    public void setName(String name){
        this.name=name;
    }


    public string getName(){
        return name;
    }
}

```

**4.5. @Autowired 注解有什么用？**

@Autowired 可以更准确地控制应该在何处以及如何进行自动装配。此注解用于在 setter 方法，构造函数，具有任意名称或多个参数的属性或方法上自动装配 bean。默认情况下，它是类型驱动的注入。



```java
    private String name;
    @Autowired
    public void setName(String name) {
        this.name=name;
    }
    public string getName(){
        return name;
    }
}

```

**4.6. @Qualifier 注解有什么用？**

当您创建多个相同类型的 bean 并希望仅使用属性装配其中一个 bean 时，您可以使用@Qualifier 注解和 @Autowired 通过指定应该装配哪个确切的 bean 来消除歧义。

例如，这里我们分别有两个类，Employee 和 EmpAccount。在 EmpAccount 中，使用@Qualifier 指定了必须装配 id 为 emp1 的 bean。

```java
public class Employee {
    private String name;
    @Autowired
    public void setName(String name) {
        this.name=name;
    }
    public string getName() {
        return name;
    }
}

```

EmpAccount.java

```java
public class EmpAccount {
    private Employee emp;

    @Autowired
    @Qualifier(emp1)
    public void showName() {
        System.out.println(“Employee name : ”+emp.getName);
    }
}

```

**4.7. @RequestMapping 注解有什么用？**

@RequestMapping 注解用于将特定 HTTP 请求方法映射到将处理相应请求的控制器中的特定类/方法。此注解可应用于两个级别：

- 类级别：映射请求的 URL
- 方法级别：映射 URL 以及 HTTP 请求方法

## **5、数据访问**

**5.1. spring DAO 有什么用？**

Spring DAO 使得 JDBC，Hibernate 或 JDO 这样的数据访问技术更容易以一种统一的方式工作。这使得用户容易在持久性技术之间切换。它还允许您在编写代码时，无需考虑捕获每种技术不同的异常。

**5.2. 列举 Spring DAO 抛出的异常。**

![img](../media/pictures/Spring.assets/640-1589988153077.webp)img

**5.3. spring JDBC API 中存在哪些类？**

- JdbcTemplate
- SimpleJdbcTemplate
- NamedParameterJdbcTemplate
- SimpleJdbcInsert
- SimpleJdbcCall

**5.4. 使用 Spring 访问 Hibernate 的方法有哪些？**

我们可以通过两种方式使用 Spring 访问 Hibernate：

1. 使用 Hibernate 模板和回调进行控制反转
2. 扩展 HibernateDAOSupport 并应用 AOP 拦截器节点

**5.5. 列举 spring 支持的事务管理类型**

Spring 支持两种类型的事务管理：

1. 程序化事务管理：在此过程中，在编程的帮助下管理事务。它为您提供极大的灵活性，但维护起来非常困难。
2. 声明式事务管理：在此，事务管理与业务代码分离。仅使用注解或基于 XML 的配置来管理事务。

**5.6. Spring 支持哪些 ORM 框架**

- Hibernate
- iBatis
- JPA
- JDO
- OJB

## **6、AOP**

**6.1. 什么是 AOP？**

AOP(Aspect-Oriented Programming), 即 面向切面编程, 它与 OOP( Object-Oriented Programming, 面向对象编程) 相辅相成, 提供了与 OOP 不同的抽象软件结构的视角.

在 OOP 中, 我们以类(class)作为我们的基本单元, 而 AOP 中的基本单元是 Aspect(切面)

**6.2. AOP 中的 Aspect、Advice、Pointcut、JointPoint 和 Advice 参数分别是什么？**

![img](D:/Code/Typora/media/pictures/Spring.assets/640.png)img

1. Aspect - Aspect 是一个实现交叉问题的类，例如事务管理。方面可以是配置的普通类，然后在 Spring Bean 配置文件中配置，或者我们可以使用 Spring AspectJ 支持使用 @Aspect 注解将类声明为 Aspect。
2. Advice - Advice 是针对特定 JoinPoint 采取的操作。在编程方面，它们是在应用程序中达到具有匹配切入点的特定 JoinPoint 时执行的方法。您可以将 Advice 视为 Spring 拦截器（Interceptor）或 Servlet 过滤器（filter）。
3. Advice Arguments - 我们可以在 advice 方法中传递参数。我们可以在切入点中使用 args() 表达式来应用于与参数模式匹配的任何方法。如果我们使用它，那么我们需要在确定参数类型的 advice 方法中使用相同的名称。
4. Pointcut - Pointcut 是与 JoinPoint 匹配的正则表达式，用于确定是否需要执行 Advice。Pointcut 使用与 JoinPoint 匹配的不同类型的表达式。Spring 框架使用 AspectJ Pointcut 表达式语言来确定将应用通知方法的 JoinPoint。
5. JoinPoint - JoinPoint 是应用程序中的特定点，例如方法执行，异常处理，更改对象变量值等。在 Spring AOP 中，JoinPoint 始终是方法的执行器。

**6.3. 什么是通知（Advice）？**

特定 JoinPoint 处的 Aspect 所采取的动作称为 Advice。Spring AOP 使用一个 Advice 作为拦截器，在 JoinPoint “周围”维护一系列的拦截器。

**6.4. 有哪些类型的通知（Advice）？**

- Before - 这些类型的 Advice 在 joinpoint 方法之前执行，并使用 @Before 注解标记进行配置。
- After Returning - 这些类型的 Advice 在连接点方法正常执行后执行，并使用@AfterReturning 注解标记进行配置。
- After Throwing - 这些类型的 Advice 仅在 joinpoint 方法通过抛出异常退出并使用 @AfterThrowing 注解标记配置时执行。
- After (finally) - 这些类型的 Advice 在连接点方法之后执行，无论方法退出是正常还是异常返回，并使用 @After 注解标记进行配置。
- Around - 这些类型的 Advice 在连接点之前和之后执行，并使用 @Around 注解标记进行配置。

**6.5. 指出在 spring aop 中 concern 和 cross-cutting concern 的不同之处。**

concern 是我们想要在应用程序的特定模块中定义的行为。它可以定义为我们想要实现的功能。

cross-cutting concern 是一个适用于整个应用的行为，这会影响整个应用程序。例如，日志记录，安全性和数据传输是应用程序几乎每个模块都需要关注的问题，因此它们是跨领域的问题。

**6.6. AOP 有哪些实现方式？**

实现 AOP 的技术，主要分为两大类：

- 静态代理 - 指使用 AOP 框架提供的命令进行编译，从而在编译阶段就可生成 AOP 代理类，因此也称为编译时增强；
- 编译时编织（特殊编译器实现）
- 类加载时编织（特殊的类加载器实现）。
- 动态代理 - 在运行时在内存中“临时”生成 AOP 动态代理类，因此也被称为运行时增强。
- JDK 动态代理
- CGLIB

**6.7. Spring AOP and AspectJ AOP 有什么区别？**

Spring AOP 基于动态代理方式实现；AspectJ 基于静态代理方式实现。

Spring AOP 仅支持方法级别的 PointCut；提供了完全的 AOP 支持，它还支持属性级别的 PointCut。

**6.8. 如何理解 Spring 中的代理？**

将 Advice 应用于目标对象后创建的对象称为代理。在客户端对象的情况下，目标对象和代理对象是相同的。

- 

```
Advice + Target Object = Proxy

```

**6.9. 什么是编织（Weaving）？**

为了创建一个 advice 对象而链接一个 aspect 和其它应用类型或对象，称为编织（Weaving）。在 Spring AOP 中，编织在运行时执行。请参考下图：

![img](D:/Code/Typora/media/pictures/Spring.assets/640.png)img

## **7、MVC**

**7.1. Spring MVC 框架有什么用？**

Spring Web MVC 框架提供 模型-视图-控制器 架构和随时可用的组件，用于开发灵活且松散耦合的 Web 应用程序。MVC 模式有助于分离应用程序的不同方面，如输入逻辑，业务逻辑和 UI 逻辑，同时在所有这些元素之间提供松散耦合。

**7.2. 描述一下 DispatcherServlet 的工作流程**

DispatcherServlet 的工作流程可以用一幅图来说明：

![img](D:/Code/Typora/media/pictures/Spring.assets/640.png)img

1. 向服务器发送 HTTP 请求，请求被前端控制器 DispatcherServlet 捕获。
2. DispatcherServlet 根据 -servlet.xml 中的配置对请求的 URL 进行解析，得到请求资源标识符（URI）。然后根据该 URI，调用 HandlerMapping 获得该 Handler 配置的所有相关的对象（包括 Handler 对象以及 Handler 对象对应的拦截器），最后以HandlerExecutionChain 对象的形式返回。
3. DispatcherServlet 根据获得的Handler，选择一个合适的 HandlerAdapter。（附注：如果成功获得HandlerAdapter后，此时将开始执行拦截器的 preHandler(…)方法）。
4. 提取Request中的模型数据，填充Handler入参，开始执行Handler（Controller)。在填充Handler的入参过程中，根据你的配置，Spring 将帮你做一些额外的工作：

- HttpMessageConveter：将请求消息（如 Json、xml 等数据）转换成一个对象，将对象转换为指定的响应信息。
- 数据转换：对请求消息进行数据转换。如`String`转换成`Integer`、`Double`等。
- 数据根式化：对请求消息进行数据格式化。如将字符串转换成格式化数字或格式化日期等。
- 数据验证：验证数据的有效性（长度、格式等），验证结果存储到`BindingResult`或`Error`中。

\5. Handler(Controller)执行完成后，向 DispatcherServlet 返回一个 ModelAndView 对象；

\6. 根据返回的ModelAndView，选择一个适合的 ViewResolver（必须是已经注册到 Spring 容器中的ViewResolver)返回给DispatcherServlet。

\7. ViewResolver 结合Model和View，来渲染视图。

\8. 视图负责将渲染结果返回给客户端。

**7.3. 介绍一下 WebApplicationContext**

WebApplicationContext 是 ApplicationContext 的扩展。它具有 Web 应用程序所需的一些额外功能。它与普通的 ApplicationContext 在解析主题和决定与哪个 servlet 关联的能力方面有所不同。



# Spring中的事务

## @Transaction必知必会



##### 1. Spring事务的基本原理

事务管理是应用系统开发中必不可少的一部分。Spring 为事务管理提供了丰富的功能支持。Spring 事务管理分为编码式和声明式的两种方式。编程式事务指的是通过编码方式实现事务；声明式事务基于 AOP,将具体业务逻辑与事务处理解耦。声明式事务管理使业务代码逻辑不受污染, 因此在实际使用中声明式事务用的比较多。声明式事务有两种方式，一种是在配置文件（xml）中做相关的事务规则声明，另一种是基于@Transactional 注解的方式。注释配置是目前流行的使用方式，因此本文将着重介绍基于@Transactional 注解的事务管理。
使用@Transactional的相比传统的我们需要手动开启事务，然后提交事务来说。它提供如下方便

- 根据你的配置，设置是否自动开启事务
- 自动提交事务或者遇到异常自动回滚

声明式事务（@Transactional)基本原理如下：

1. 配置文件开启注解驱动，在相关的类和方法上通过注解@Transactional标识。
2. spring 在启动的时候会去解析生成相关的bean，这时候会查看拥有相关注解的类和方法，并且为这些类和方法生成代理，并根据@Transaction的相关参数进行相关配置注入，这样就在代理中为我们把相关的事务处理掉了（开启正常提交事务，异常回滚事务）。
3. 真正的数据库层的事务提交和回滚是通过binlog或者redo log实现的。

##### 2. @Transactional基本配置解析



```java
@Transactional
    public void saveUser(){

        User user = new User();
        user.setAge(22);
        user.setName("maskwang");
        logger.info("save the user{}",user);
        userRepository.save(user);
    }

```

如上面这个例子一样，很轻松的就能应用事务。只需要在方法上加入@Transactional注解。当@Transactional加在方法上，表示对该方法应用事务。当加在类上，表示对该类里面所有的方法都应用相同配置的事务。接下来对@Transactional的参数解析。



```java
public @interface Transactional {
    @AliasFor("transactionManager")
    String value() default "";

    @AliasFor("value")
    String transactionManager() default "";

    Propagation propagation() default Propagation.REQUIRED;

    Isolation isolation() default Isolation.DEFAULT;

    int timeout() default -1;

    boolean readOnly() default false;

    Class<? extends Throwable>[] rollbackFor() default {};

    String[] rollbackForClassName() default {};

    Class<? extends Throwable>[] noRollbackFor() default {};

    String[] noRollbackForClassName() default {};
}

```

- `transactionManager()`表示应用那个应用那个`TransactionManager`.常用的有如下的事务管理器



![img](../media/pictures/Spring.assets/5281821-51bcc971dd9f9374.webp)

image.png

- `isolation()`表示隔离级别



![img](../media/pictures/Spring.assets/5281821-753c25c5309dec2d.webp)

image.png

> 脏读：一事务对数据进行了增删改，但未提交，另一事务可以读取到未提交的数据。如果第一个事务这时候回滚了，那么第二个事务就读到了脏数据。
> 不可重复读：一个事务中发生了两次读操作，第一次读操作和第二次操作之间，另外一个事务对数据进行了修改，这时候两次读取的数据是不一致的。
> 幻读：第一个事务对一定范围的数据进行批量修改，第二个事务在这个范围增加一条数据，这时候第一个事务就会丢失对新增数据的修改。

**不可重复读的重点是修改 :同样的条件, 你读取过的数据,再次读取出来发现值不一样了幻读的重点在于新增或者删除:同样的条件, 第 1 次和第 2 次读出来的记录数不一样**

- `propagation()`表示事务的传播属性
  事务的传播是指事务的嵌套的时候，它们的事务属性。常见的传播属性有如下几个

1. `PROPAGATION_REQUIRED`(spring 默认)
   假设外层事务 Service A 的 Method A() 调用 内层Service B 的 Method B()。如果ServiceB.methodB() 的事务级别定义为 PROPAGATION_REQUIRED，那么执行 ServiceA.methodA() 的时候spring已经起了事务，这时调用 ServiceB.methodB()，ServiceB.methodB() 看到自己已经运行在 ServiceA.methodA() 的事务内部，就不再起新的事务。假如 ServiceB.methodB() 运行的时候发现自己没有在事务中，他就会为自己分配一个事务。不管如何，ServiceB.methodB()都会在事务中。
2. `PROPAGATION_REQUIRES_NEW`
   比如我们设计 ServiceA.methodA() 的事务级别为 PROPAGATION_REQUIRED，ServiceB.methodB() 的事务级别为 PROPAGATION_REQUIRES_NEW。那么当执行到 ServiceB.methodB() 的时候，ServiceA.methodA() 所在的事务就会挂起，ServiceB.methodB() 会起一个新的事务，等待 ServiceB.methodB() 的事务完成以后，它才继续执行。它与1中的区别在于ServiceB.methodB() 新起了一个事务。如过ServiceA.methodA() 发生异常，ServiceB.methodB() 已经提交的事务是不会回滚的。
3. `PROPAGATION_SUPPORTS`
   假设ServiceB.methodB() 的事务级别为 PROPAGATION_SUPPORTS，那么当执行到ServiceB.methodB()时，如果发现ServiceA.methodA()已经开启了一个事务，则加入当前的事务，如果发现ServiceA.methodA()没有开启事务，则自己也不开启事务。这种时候，内部方法的事务性完全依赖于最外层的事务。
   剩下几种就不多介绍，可以参考这篇文章。https://www.jianshu.com/p/249f2cd42692

- `readOnly()` 事务超时设置.超过这个时间，发生回滚
- `readOnly()` 只读事务
  从这一点设置的时间点开始（时间点a）到这个事务结束的过程中，其他事务所提交的数据，该事务将看不见！（查询中不会出现别人在时间点a之后提交的数据）。
  **注意是一次执行多次查询来统计某些信息，这时为了保证数据整体的一致性，要用只读事务**
- `rollbackFor()`导致事务回滚的异常类数组.
- `rollbackForClassName()` 导致事务回滚的异常类名字数组
- `noRollbackFor` 不会导致事务回滚的异常类数组
- `noRollbackForClassName` 不会导致事务回滚的异常类名字数组

##### 3. `@Transactional` 使用应该注意的地方

###### 3.1 默认情况下，如果在事务中抛出了未检查异常（继承自 RuntimeException 的异常）或者 Error，则 Spring 将回滚事务；除此之外，Spring 不会回滚事务。你如果想要在特定的异常回滚可以考虑`rollbackFor()`等属性

###### 3.2 `@Transactional` 只能应用到 public 方法才有效。

这是因为在使用 Spring AOP 代理时，Spring 会调用 `TransactionInterceptor`在目标方法执行前后进行拦截之前，`DynamicAdvisedInterceptor`（`CglibAopProxy`的内部类）的的 `intercept`方法或 JdkDynamicAopProxy 的 invoke 方法会间接调用 `AbstractFallbackTransactionAttributeSource`（Spring 通过这个类获取`@Transactional` 注解的事务属性配置属性信息）的 `computeTransactionAttribute` 方法。



```java
@Nullable
    protected TransactionAttribute computeTransactionAttribute(Method method, @Nullable Class<?> targetClass) {
       //这里判断是否是public方法
        if(this.allowPublicMethodsOnly() && !Modifier.isPublic(method.getModifiers())) {
            return null;
        } 
//省略其他代码

```

若不是 public，就不会获取@Transactional 的属性配置信息，最终会造成不会用 TransactionInterceptor 来拦截该目标方法进行事务管理。整个事务执行的时序图如下。





![img](../media/pictures/Spring.assets/5281821-fecd22fffdc8d181.webp)

image.png

###### 3.3 Spring 的 AOP 的自调用问题

在 Spring 的 AOP 代理下，只有目标方法由外部调用，目标方法才由 Spring 生成的代理对象来管理，这会造成自调用问题。若同一类中的其他没有`@Transactional`注解的方法内部调用有`@Transactional`注解的方法，有`@Transactional`注解的方法的事务被忽略，不会发生回滚。这个问题是由于Spring AOP 代理造成的(如下面代码所示）。之所以没有应用事务，是因为在内部调用，而代理后的类(把目标类作为成员变量静态代理)只是调用成员变量中的对应方法，自然也就没有aop中的advice，造成只能调用父类的方法。另外一个问题是只能应用在public方法上。为解决这两个问题，使用 AspectJ 取代 Spring AOP 代理。



```java
@Transactional
 public void saveUser(){
        User user = new User();
        user.setAge(22);
        user.setName("mask");
        logger.info("save the user{}",user);
        userRepository.save(user);
       // throw new RuntimeException("exception");
    }
 public void saveUserBack(){
        saveUser();   //自调用发生
    }
```

###### 3.4 自注入解决办法



```java
@Service
public class UserService {

    Logger logger = LoggerFactory.getLogger(UserService.class);

    @Autowired
    UserRepository userRepository;
    
    @Autowired 
    UserService userService; //自注入来解决

    @Transactional
    public void saveUser(){

        User user = new User();
        user.setAge(22);
        user.setName("mask");
        logger.info("save the user{}",user);
        userRepository.save(user);
       // throw new RuntimeException("exception");
    }
 public void saveUserBack(){
        saveUser();
    }
}

```

另外也可以把注解加到类上来解决。

##### 4. 总结

`@Transactional`用起来是方便，但是我们需要明白它背后的原理，避免入坑。另外`@Transactional`不建议用在处理时间过长的事务。因为，它会一直持有数据库线程池的连接，造成不能及时返回。就是尽量是的事务的处理时间短。

参考文章：

[Spring @Transactional的使用及原理](https://link.jianshu.com/?t=https%3A%2F%2Fblog.csdn.net%2Faichuanwendang%2Farticle%2Fdetails%2F53306351)
[深入理解 Spring 事务原理](https://www.jianshu.com/p/99f8787f9eaa)
[透彻的掌握 Spring 中@transactional 的使用](https://link.jianshu.com/?t=https%3A%2F%2Fwww.ibm.com%2Fdeveloperworks%2Fcn%2Fjava%2Fj-master-spring-transactional-use%2Findex.html)
[在同一个类中，一个方法调用另外一个有注解（比如@Async，@Transational）的方法，注解失效的原因和解决方法](https://link.jianshu.com/?t=https%3A%2F%2Fblog.csdn.net%2Fclementad%2Farticle%2Fdetails%2F47339519)



参考：https://www.jianshu.com/p/5687e2a38fbc

事务的传播属性：https://blog.csdn.net/piaoslowly/article/details/81743658 



在成都自来水中，政府咨询列表清单中，是这样使用的：

```java
@Transactional(propagation = Propagation.REQUIRED, rollbackFor = Exception.class)
	public List<CombiMeterPO> bathUploadFile(HttpServletRequest request) throws BizException {
	
}

```

其中这里的propagation指的是事务传播属性，rollbackFor导致事务回滚的异常数组。





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



### 









### 

















































































