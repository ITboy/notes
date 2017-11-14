# Spring启动过程 #

分析在Web应用以及普通的Java Application中，Spring IoC容器的启动过程。

## Web应用 ##

普通的Java应用程序中，main函数是整个的入口，而在Web应用中，对于Java而言，入口是Servlet规范中的监听器。`ServletContextListener`可以监听ServletContext启动和关闭事件，Spring的启动过程主要是在这个监听器中实现。

另外，当Spring MVC引入后，web应用将使用spring的路由方式而不是原始的在`web.xml`中配置servlet mapping，所以就需要将所有的请求mapping到spring提供的一个servlet中，Spring提供叫`DispatcherServlet`的Servlet来处理。

### `ContextLoaderListener` ###

1. 调用contextInitialized处理ServletContext的启动事件
2. 调用父类ContextLoader的initWebApplicationContext方法创建WebApplicationContext并set到ServletContext中
3. `determineContextClass()`从ServletContext配置的`contextClass`属性读取使用哪个Context来作为WebContext使用，默认使用`XmlWebApplicationContext`，存储在`ContextLoadListener`同目录下的`ContextLoader.properties`中
4. `XmlWebApplicationContext`中配置了默认配置文件地址为`/WEB-INF/applicationContext.xml`，也可以通过`contextConfigLocation`来配置

> Spring使用监听器来实现IoC容器的初始化，初始化参数通过ServletContext来配置，包括`contextClass`和`contextConfigLocation`

### `DispatcherServlet` ###
