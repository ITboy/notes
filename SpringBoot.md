# Spring boot 学习记录
## 第一个demo

通常情况下，可以直接通过http://start.spring.io/生成一个简单的初始工程，在之上进行修改简单也不容易出错。

### pom.xml

有几点需要注意：

1. spring-boot-starter-parent提供了maven的一些有用的默认配置，比如`dependency-management`.
2. mvn dependency:tree可以查看jar包的依赖关系
3. spring-boot-starter-web依赖指定这是一个web项目，提供跟web相关的jar包
4. `Spring Boot Maven plugin`提供了如下功能：
	1. 收集在classpath下的jar包，打包成一个单独的、可运行的"über-jar"
	2. 搜索`public static void main()`来标记可运行的class
	3. 提供内置的spring boot包依赖，当设置了依赖的spring-boot的版本，他所依赖的jar包版本就有了默认值
5. `mvn spring-boot:run`启动项目，在内存完成打包，不会重新生成"über-jar"
6. 如果要生成"über-jar"执行`mvn clean package`

完整内容如下：

```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>org.springframework</groupId>
    <artifactId>gs-spring-boot</artifactId>
    <version>0.1.0</version>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>1.5.2.RELEASE</version>
    </parent>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
    </dependencies>

    <properties>
        <java.version>1.8</java.version>
    </properties>


    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

</project>
```

### Code

关键是掌握下面几个注解，明白spring boot的启动原理：
* `@Controller` 配置在类上, 来标识一个类是Controller，使用spring MVC出来http请求
* `@RestController` 配置在类上, 把`@Controller`和`@ResponseBody`组合在一起，意味着web请求的响应是数据而不是视图
* `@EnableAutoConfiguration` 配置在类上，告诉Spring Boot基于当前配置的jar包依赖如何配置spring，比如现在配置的`spring-boot-starter-web`会添加Tomcat和Spring MVC，自动配置会假设你在配置web应用程序而采取相应的配置
* `@RequestMapping` 配置在方法上，提供路由信息，如果不配置method，默认把`GET`, `PUT`, `POST`所有请求都由某方法来处理