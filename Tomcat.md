# tomcat是一个Servlet容器
## Context

```java
class Tomcat {
    Connector connector; // 连接处理器
    List<Servlet> sers;
}
```

顾名思义，Servlet容器就是用来装载存储Servlet的

一个Servlet表示一个运行在服务端的程序（servlet = server + applet）。用户想要使用这种程序，需要向该程序发送请求以及获取该程序的响应，也就是Servlet规范中的ServletRequest、ServletResponse。

所以Servlet其实就是Java中用来处理请求的一种规范，所以我们的项目中通常都会有一个或多个Servlet，由它来负责接收请求，或者将请求转交给其他业务逻辑。

所以我们的Spring MVC、Spring Boot都存在一个DispatcherServlet（类似功能的一个Servlet，负责接收请求）。

所以，通常Servlet是属于一个应用程序（项目）的，换句话说，我们的 ***一个应用包含多个Servlet，所以这是第二层Servlet容器--应用***，也就是Tomcat中的Context（应用上下文）。那么第一层Servlet容器呢？
## Wrapper
Wrapper就是第一层Servlet容器，Wrapper表示Servlet的包装者，所以它是最接近Servlet的，那么为什么需要Wrapper呢？

```java
class Wrapper {
    Liat<Servlet> servlet;//
}
```

一个Wrapper对应一个Servlet，这么来想的话，确实不需要Wrapper，但是我们还要考虑一些其他的情况：
+ 比如Filter，一个Filter是可以对应一个Servlet的。
+ 比如ServletPool，通常的Servlet是所有请求线程公用的，但是在Servlet中支持每一个请求线程单独使用独立的Servlet实例。

所以在Wrapper中，不仅仅只包括一个Servlet，还包括过滤器和Servlet池，所以Wrapper是第一层

## host
在我们现实生活中，一个应用都是部署在一个主机上的，所以，一个主机可以包含多个应用，一个应用包含多个Servlet，所以，Host是第三层容器。

在Tomcat中，Host表示虚拟主机，Tomcat在处理请求时，可以根据请求的域名进入到相应的Host中进行处理。
## Engine
Host管理Context，Context管理Wrapper，Wrapper管理Servlet，而Engine就是用来管理Host的。所以Engine是第四层容器。

## 知识点
1. tcp的链接时封装在socket中的， 一个socket实际就是new了一个tcp链接
2. 代码层面的socket最终调用的时操作系统中的socket方法去建立连接
3. tomcat处理请求得步骤
    1. 数据从另一台机器发送数据过来，当前操作系统接收socket
    2. tomcat从socket中获取数据， 解析数据，解析成request
    3. request将数据传递给容器engine
    4. request -> engine->host-> context -> wrapper-> servlet