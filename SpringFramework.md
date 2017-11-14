# Spring Framework学习
## 使用注解还是XML配置

### 官网的观点

看情况，各有优缺点，annotation在声明时就提供了大量的上下文，使得配置短并且简洁。但是XML方式可以在不碰代码的情况下重组组件的连接方式。有些开发者喜欢在写代码的同时写配置，但有些人却认为带注解的类不再是POJO，而且去中心化的配置难于控制。

### 自己的分析
#### XML的优点

* 修改无需重新编译
* 对象的注入关系与代码本身彻底解耦
* 所有的配置在一起，可以根据项目的层次分类在多个配置文件中
* 对于第三方的包，由于无法修改他的代码，只能使用XML配置

#### XML的缺点

* 配置和修改麻烦，需要改动较多的内容
* 与代码分离，当阅读代码的时候需要频繁的在代码和配置之间切换视角

#### 注解的优点

* 简洁
* 读代码很方便，容易维护

#### 缺点

* 任意改动，需要重新编译打包

#### 取舍

1. 个人更喜欢注解，因为简洁和可维护性，我觉得这是最重要的，另外我也无需考虑如何组织XML文件（需要组织，否则XML的配置会过于臃肿，难于理解和维护，用<import />导入），
2. 对于类似数据源的配置之类的，一定还是要在XML中配置，或在properties文件中配置，否则修改数据源要重新编译，实在无法接收。
3. 当然对于第三方库也只能使用XML配置

## Bean ##

### 命名 ###

bean可以有name，id来作为标识符，还可以为bean起别名

### Bean的定制化 ###

spring可以为单独某个bean提供初始化后和销毁前的hook，方式有如下三种：
1. 让bean实现`InitializingBean `或`DisposableBean`接口
2. XML方式： 为bean配置`init-method`或`destroy-method`属性
3. annotation方式，在相应方法上添加@PostConstruct和@PreDestroy注解

### 实例化 ###

* 构造函数
* 类的静态工厂方法
* 对象的工厂方法

### 依赖注入 ###

* 构造函数传参 `<constructor-arg>`
* setter方法 `<property>`

### 什么时候使用`constructor-arg`，什么时候使用`property`？

正常情况下，对象的必须属性使用`constructor-arg`, 可选属性使用`property`，然而Spring团队推荐总是使用`constructor-arg`, 因为这样初始化出来的对象就没有null的问题，不需要检查null，但是如果参数很多的时候，构造函数的参数就会很多，这样的代码会很糟糕，所以如果使用`property`，也尽量都给默认值，不要有null。

**这个点子非常好，在对象初始化的时候，为所有的属性赋值，彻底解决空指针的讨厌问题。**

### 如何解决对象的循环依赖

A引用B，B也引用A，如果都使用`constructor-arg`来配置，就会报循环异常，可以使用setter注入的方法解决这个问题。
使用`<property>`就是用使用setter方法注入。

### 自动装载 ###

无需手动配置依赖的实际对象，可配置根据类型，名字等自动装载。

极大的简化配置

我能想到的是真正做到可插拔式装载，比如把service对dao的依赖配置为autowire，当切换成另一套dao时，只需要替换掉已有的dao的class即可，只要他们都实现了同样的接口。

#### 自动装载的缺点和限制

* 精确的装载配置`<property>`, `<constructor-arg>`会覆盖自动装载
* 基础类型属性不支持自动装载，如`String`， `int`等
* 不如显示的配置清晰
* 不利于有些工具生成文档
* 如果存在多个可用的类型或名字，会导致自动装载失败

#### 什么情况下使用自动装载

如果大量使用显式装载，而个别使用自动装载会令人感到费解，但是反过来却不会。

我想大部分项目应该都可以主要使用自动装载，简化配置，个别显示指定。

## 可以实现ApplicationContextAware来得到ApplicationContext ##

一般不要这样做，会使你的代码和spring framework耦合

## 注解使用 ##

### @Required ###

作用在bean属性的setter方法上，表示该属性在被IoC容器构造时是必须设置的（显示设置property或autowire，否则就报错。
这保证当前对象被初始化后，对象的方法在操作对象时，不需要检查某些首先是否为空。
但是仍然需要在这些操作的方法中加入断言，这样就可以保证在容器之外使用这个对象时也不用检查空指针的属性。

### @Autowired ###

可以作用在属性，属性setter，构造函数，普通函数上，以及方法的参数上。

#### 属性或属性setter ####

`@Required`是指明属性必须被注入，而`@Autowired`就是指属性必须使用自动注入的方式注入。

#### 构造函数 ####

构造函数的几个参数都使用自动注入的方式来得到。

#### 普通方法 ####

普通方法注入有什么用呢？当然是在指定使用普通方法来产生bean被IoC管理时（@Bean注解），如果这个方法注解`@Autowired`，就会对方法的参数自动注入。

#### 方法参数 ####

其实`@Autowired`注解本质上就是应用在参数上，标识这个参数使用自动注入方式寻找他的值，其他位置的注解都是变种。

* 在属性上 等价于在属性的setter方法的参数上
* 在属性setter方法上 等价于在属性的setter方法的参数上
* 在构造函数上 等价于在构造函数每个参数上
* 普通方法上 等价于普通方法的每个参数上

### @Primary ###

当自动注入有多个备用bean时，指定哪个bean具有更高的优先级。

```java
@Configuration
public class MovieConfiguration {

    @Bean
    @Primary
    public MovieCatalog firstMovieCatalog() { ... }

    @Bean
    public MovieCatalog secondMovieCatalog() { ... }

    // ...

}
```

``` java
public class MovieRecommender {

    @Autowired
    private MovieCatalog movieCatalog;

    // ...

}
```

### @Qualifier ###

类似于profile的概念。
用法：
1. 每个bean都有一个qualifier值，默认是bean的name
2. 使用`@Qualifier`和`@Bean`或`@Component`配合使用来设置qualifier的值。XML方式使用`<qualifier value="action"/>`
3. `@Qualifier`和`@Autowired`使用里选择使用哪个bean作为注入对象
   
### @Resource ###

通过名字注入对象，如果省略名字，使用属性名字或参数名字

### @Autowire vs. @Inject vs. @Resource ###
@Autowire是spring注解，@Inject和@Resource是javax注解, @Inject基于JSR-330，@Resource基于JSR-250

@Autowired可以配置required=false来在查找失败时使用null来作为值，而@Inject没有这种配置
@Autowire和@Inject的查找路径是Type，Qualifier最后是Name，而@Resource是先Name，然后Type最后Qualifier。

### @Component, @Service, @Controller, @Repository ###

@Configuration, @Bean， @Component，@Service, @Controller用来注册基础的bean，而@Autowired，@Resource用来完成依赖注入，@Qualifier和@Primary用来支持依赖注入

在这些组建标识的类中，也可以使用`@Bean`来从工厂方法中取得对象

### @Configuration @Bean ###

我理解是用普通函数（不是构造函数和setter）来注册对象的方式。

### @Configuration @ComponentScan ###

配置自动扫描包，从这些包的类中解析spring注解

``` java
@Configuration
@ComponentScan(basePackages = "org.example")
```

### @SpringBootApplication ###

`@SpringBootApplication` = `@Configuration` + `@ComponentScan` + `@EnableAutoConfiguration`

### @RestController ###

`@RestController` = `@Controller` + `@ResponseBody`

### @RequestMapping ###

配置路由信息

## Sample ##

### ApplicationContext ###
ApplicationConetxt实现一个只包含spring最基础的IoC功能，帮助明白ApplicationContext如何使用，并且是最简单的application而不是web的应用，他的实现分为基于XML的spring配置和基于anotation的配置两种。

#### 基于XML的spring例子 ####
基于XML的`spring`配置使用`ApplicationContext`的具体实现`ClassPathXmlApplicationContext`.

```java
ApplicationContext applicationContext = new ClassPathXmlApplicationContext("SpringBeans.xml");
Hello hello = applicationContext.getBean("Hello", Hello.class);
hello.sayHello();
((ClassPathXmlApplicationContext)applicationContext).close();
```

标准的JavaBean Hello类实现：
```java
public class Hello {

	private String name;
	
	public Hello() {
		this.name = "world";
	}
	
	public void sayHello() {
		System.out.println("Hello, " + name);
	}

	public String getName() {
		return name;
	}

	public void setName(String name) {
		this.name = name;
	}
}
```

xml配置要放到`src/main/resource`下，因为`maven` build时会默认把这个路径下的文件放到classpath下。
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:p="http://www.springframework.org/schema/p"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:util="http://www.springframework.org/schema/util"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
                           http://www.springframework.org/schema/beans/spring-beans-4.0.xsd
                           http://www.springframework.org/schema/context
                           http://www.springframework.org/schema/context/spring-context.xsd
                           http://www.springframework.org/schema/util
                           http://www.springframework.org/schema/util/spring-util.xsd">
	<bean class="com.prepper.spring.ApplicationContext.Hello" name="Hello">
		<property name="name" value="leo"></property>
	</bean>
</beans>
```

`pom.xml`只需要依赖最基础的`spring-context`。
```
<dependency>
	<groupId>org.springframework</groupId>
	<artifactId>spring-context</artifactId>
	<version>${spring.version}</version>
</dependency>
```
#### 基于注解的spring例子 ####
基于注解的`spring`配置使用`ApplicationContext`的具体实现`AnnotationConfigApplicationContext`.

```java
public static void main( String[] args )
{
    ApplicationContext applicationContext = new AnnotationConfigApplicationContext(Hello.class);
    Hello hello = applicationContext.getBean("Hello", Hello.class);
    hello.sayHello();
    ((AnnotationConfigApplicationContext)applicationContext).close();
}
```
`AnnotationConfigApplicationContext`的参数也可以是包名，也可以无参，然后使用`register`来添加类：
```java
public static void main(String[] args) {
  AnnotationConfigApplicationContext ctx = new AnnotationConfigApplicationContext();
  ctx.register(AppConfig.class, OtherConfig.class);
  ctx.register(AdditionalConfig.class);
  ctx.refresh();
  MyService myService = ctx.getBean(MyService.class);
  myService.doStuff();
}
```

标准的JavaBean Hello类实现需要使用`@Component`来标识是被IoC管理的一个组件。
```java
@Component("Hello")
public class Hello {

	private String name;
	
	public Hello() {
		this.name = "world";
	}
	
	public void sayHello() {
		System.out.println("Hello, " + name);
	}

	public String getName() {
		return name;
	}

	public void setName(String name) {
		this.name = name;
	}
}
```

`pom.xml`只需要依赖最基础的`spring-context`。
```
<dependency>
	<groupId>org.springframework</groupId>
	<artifactId>spring-context</artifactId>
	<version>${spring.version}</version>
</dependency>
```
> spring的`ApplicationContext`目前只支持xml和注解两种方式，本质上会使用`ClassPathXmlApplicationContext`和`AnnotationConfigApplicationContext`，这两个类分别代表了xml和anotation注解，是应用程序进入使用spring IoC的总入口，对于xml方式需要指定xml文件路径，如果是anotation需要指定入口的class或者一个base package，然后扫描所有包内部的带注解的类。这是在入口时指定，在spring启动过程中，仍然可以指定别的xml文件或者扫描别的包，这时可以用

## 容器的扩展点 ##
### 使用BeanPostProcessor扩展bean ###
bean的生命周期如下：
实例化 -> setProperties -> call init-method(@PostConstruct) -> use -> call destroy-method(@PreDestroy) -> destroy
BeanPostProcessor提供的两个方法是在初始化之前和初始化之后，也就是分别在调用`init-method`之前和之后
但是他是对当前容器中的每一个bean都做处理，而且只需要把实现`BeanPostProcessor`接口的bean交由ApplicationContext管理，就自动生效。

如果要调整多个实现`BeanPostProcessor`接口的执行顺序，可以实现`Order`接口。

很多注解或配置都是基于`BeanPostProcessor`接口实现的。

### `@Required` ###

在每个bean初始化后判断标记为`required`的属性是不是被设置了，如果没有则抛出异常。

### `<context:annotation-config/>` ###

如果xml中加入这行配置，会让容器启动时扫描注解，但他没有配扫描哪些包的注解，这里他会扫描在当前`ApplicationContext`中配置的bean中的注解，比如可以在这些bean中配置的`@autowire`注解。

这个功能也可以在初始化bean之后来做，但是此时注解的配置在后会覆盖xml的配置。