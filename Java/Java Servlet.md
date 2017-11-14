# Java Servlet基础

在Java领域开发Web应用程序，Servlet始终是逃不掉的基础，现在的Web开发都是基于各种框架，但是如果要明白原理，还是要有一定的servlet的基础。

## Servlet规范

Java Servlet有接口规范，当前版本是3.1， 定义规范，就可以使你的Web应用可以在不同容器中运行。

## Servlet和Servlet容器 ##

Servlet是一个Java类，用来处理用户请求的业务逻辑，对不同的HTTP方法都有相应的处理方法（doGet，doPost，doPut，doOptions, doHead, doDelete）

Servlet容器顾名思义，servlet运行的环境。

## Servlet目录结构 ##

```
/index.html
/howto.jsp
/feedback.jsp
/images/banner.gif
/images/jumping.gif
/WEB-INF/web.xml
/WEB-INF/lib/jspbean.jar
/WEB-INF/lib/catalog.jar!/METAINF/
resources/catalog/moreOffers/books.html
/WEB-INF/classes/com/mycorp/servlets/MyServlet.class
/WEB-INF/classes/com/mycorp/util/MyUtils.class
```

> WEB-INF除了lib下jar包中METAINF/resources下的静态资源, WEB-INF下的文件都不能被直接访问

## ServleContext ##

Servlet的执行上下文，每一个App都有一个ServletContext，当servlet container启动时会实例化ServletContext，ServletContext提供一系列操作Servlet，过滤器,监听器的方法：
* `addServlet`
* `createServlet`
* `addFilter`
* `createFilter`
* `addListener`
* `createListener`

## 过滤器 ##

可以为url添加多个过滤器，过滤器依次被调用，过滤器可以修改request和response供后面的过滤器或servlet使用。

### 实现一个过滤器 ###

#### 实现`java.servlet.Filter`接口 ####

```java
public class MyFilter implements Filter {
    @Override
    public void init(FilterConfig filterConfig) throws ServletException {
        System.out.println("初始化了！");
		System.out.println(filterConfig.getInitParameter("test"));
    }
 
    @Override
    public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {
        System.out.println("过滤器1，这里处理一些过滤操作！");
        filterChain.doFilter(servletRequest, servletResponse);
        System.out.println("过滤器1响应！");
    }
 
    @Override
    public void destroy() {
        System.out.println("过滤器1被销毁！");
    }
}
```

> doFilter里可以调用doFilter()来继续处理，调用之前修改request，调用之后修改response

#### 配置 ####
```xml
<filter>
       <filter-name>MyFilter1</filter-name>
       <filter-class>com.iteblog.MyFilter</filter-class>
		<init-param>
            <param-name>test</param-name>
            <param-value>Just a test!!</param-value>
        </init-param>
   </filter>
 
   <filter-mapping>
       <filter-name>MyFilter1</filter-name>
       <url-pattern>/*</url-pattern>
   </filter-mapping>
```
> 可以为filter配置初始化参数，调用filterConfig.getInitParameter('xxx')来读取

#### filter的优先规则 ####

1. 以url-pattern方式配置的filter运行时肯定先于以servlet-name方式配置的filter
2. 以url-partern方式配置的filter中，如果有多个与当前请求匹配，则按web.xml中filter-mapping出现的顺序来运行
3. 对于以servlet-name方式配置的filter，如果有多个与当前请求匹配，也是按web.xml中filter-mapping出现的顺序来运行


## Servlet的生命周期 ##

* 实例化 
* init
* service
* destroy

> * 实例化可以有两种，默认会在启动容器时，由ServletContext实例化，如果配置懒加载，那么在第一次请求时实例化
> * 当servlet被第一次调用时会初始化一些东西，他是全局的，凡是走到这个servlet的请求都会受影响

## 监听器 ##

* ServletContext的事件
	- lifecycle
		当ServletContext被创建完成可以接收请求或关闭时
	- Changes to Attribute
		当ServletContext上有属性添加删除或修改时
* Session事件
  - lifecycle
	Session被创建，失效或超时
  - Changes to Attribute
  - 其他

* ServletRequest

