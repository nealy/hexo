title: Tomcat 配置文件 server.xml 详解
date: 2016-04-07 14:59:56
updated: 2016-04-07 14:59:56
tags: [j2ee, tomcat]
categories: java
---

`conf/server.xml` 是 Servlet/JSP 容器 Tomcat 的主要配置文件，使用 XML 的方式描述了 WebApp 的运行方式。
<!-- more -->

在 server.xml 中的元素（element）和属性（properties）是区分大小写的，并且支持 Apache Ant 方式的变量替换（Apache Ant-style variable substitution）。在配置文件中还支持以 `${propname}` 的方式读取系统变量 propname。支持 系统环境变量（包括 `-D` 语法）、JVM 变量和配置在 `$CATALINA_BASE/conf/catalina.properties file` 文件中的变量。

server.xml 元素包含四大类：
>1、顶层类元素 \- `<Server>` 是根元素，`<Service>` 表示一组与一个 `<Engine>` 有关的连接器。

>2、连接器类元素 \- `<Connector>`，客户端和容器类元素的通讯接口。

>3、容器类元素 \- `<Engine>`、`<Host>` 和 `<Context>`，处理客户请求并且生成响应结果。

>4、嵌套类元素 \- `<logger>`、`<Value>` 和 `<Realm>`，可以加入到容器中的元素。

文件结构：
```
<Server>
    <Service>
        <Connector/>
        <Connector/>
        <Engine>
            <Host>
                <Context>
                <Context/>
            </Host>
            <Host>
                <Context/>
                <Context/>
            </Host>
        </Engine>
    </Service>
<Server>
```

### Server 元素

`<Server>` 是 Tomcat 实例的顶层元素，由 org.apache.catalina.Server 接口定义，它可以包含一个或多个 `<Service>` 元素，并且不能做为任何元素的子元素。一个 `<Server>` 是一个提供完整JVM的独立组件，它可以代表整个容器，但它本身不是一个容器，不可以定义 `<value>` 或 `<loggers>` 之类的子组件。

属性说明：

属性|说明
---|---
port|指定一个端口，这个端口负责监听关闭 Tomcat 的请求
shut down|向以上端口发送的关闭服务器的命令字符串，通常为 `SHUTDOWN`

对于一个已经开启的 Tomcat 服务器，可以在 cmd 下使用 `telnet localhost 8005` 命令进行连接，然后输入 `SHUTDOWN` 命令就可以关闭服务器。

### Service 元素

`<Service>` 是一个集合，它由一个或者多个 `<Connector>` 以及一个 `<Engine>` 组成，这个 `<Engine>` 负责处理所有 `<Connector>` 所获得的客户请求。每个 `<Service>` 元素只能有一个 `<Engine>` 元素。 `<Service>` 本身也不是容器。

属性说明：

属性|说明
---|---
name|`<Service>` 的名称

### Connector 元素

`<Connector>` 是直接与用户交互的组件，负责接受用户请求和向客户返回响应结果。 

属性说明：

属性|说明
---|---
port|`<Connector>` 所监听的端口。在浏览器中可以通过输入 `url:port` 的方式提交给对应的 `<Connector>`。因为浏览器的默认端口是 80，所以如果把 `<Connector>` 的 port 设成 80 的话，可以直接使用 url 进行访问，不用在后边再跟一个端口号。
protocol|设定 Http 协议，默认是 HTTP/1.1。
minThreads|服务器启动时创建的处理用户请求的线程数。
maxThreads|可以创建的最大的处理用户请求的线程数。
minSpareThreads|最小备用线程数。
maxSpareThreads|最大备用线程数。
acceptCount|当所有可以使用的处理请求的线程都被用光时,可以放到处理队列中的请求数,超过这个数的请求将不予处理，而返回 Connection refused 错误。
redirectPort|服务器正在处理 http 请求时收到了一个 SSL 传输请求后重定向的端口号。（即当请求是 https 时，将它转发到的端口）。
enableLookups|如果为 true，表示支持域名解析，则可以在 web 应用中通过调用 request.getRemoteHost() 进行 DNS 查询来得到远程客户端的实际主机名；若为 false 则不进行 DNS 查询，而是返回其 ip 地址。默认值为 true。
connectionTimeout|等待超时的时间数（以毫秒为单位），如果为 -1 表示不限制客户连接的时间。

### Engine 元素

它处理在同一个 `<Service>` 中所有 `<Connector>` 元素接收到的客户请求。它匹配请求和自己的虚拟主机，并将请求发给对应的 `<Host>` 处理，默认的主机是 localhost。

属性说明：

属性|说明
---|---
name|engine的名称，对应目录 `/conf/Catalina`。
defaultHost|默认的处理请求的虚拟主机，至少与下面一个Host的name属性一样。对应 `/conf/Catalina/localhost`。
Debug|日志等级。

### Host 元素

一个 `<Engine>` 元素可以包含多个 `<Host>` 元素，每个 `<Host>` 元素定义一个虚拟主机，它包含一个或多个web应用。

属性说明：

属性|说明
---|---
name|虚拟主机名，对应目录 `/conf/Catalina/localhost`。
appBase|指定虚拟主机的目录，默认为 `/webapps`。它将请求 url 与该虚拟主机的 `<Context>` 进行匹配，并把请求转给对应的 `<Context>` 来处理。
Debug|日志等级。
autoDeploy|默认为 true，表示如果有新的 Web 应用放入 appBase 并且 Tomcat 在运行的情况下，自动载入应用。
unpackWARs|如果设置为 true，表示把war文件先展开再运行。如果为 false则直接运行 war 文件。

### Context 元素

代表运行在虚拟主机上的单个 web 应用。一个 Host> 可以包含多个 `<Context>` 元素。每个 web 应用有唯一个相对应的 `<Context>` 代表 web 应用自身。

属性说明：

属性|说明
---|---
path|Web应用名，在使用 url 访问 `<Host>` 下的web应用时，通过 `http://localhst/website` 的形式。其中 localhost 为上文所说的 `<Host>` 的 name，而 website 就是这里的 path。也就是说当一具请求到来时，`<Engine>` 先根据 host name = localhost 来确定所要求的主机，再根据 context path = website 确定用户所请求的 web 应用。
docBase|Web应用的具体存放路径
Debug|日志等级。
autoDeploy|默认为 true，表示如果有新的 web 应用放入 appBase 并且 Tomcat 在运行的情况下，自动载入应用。
unpackWARs|如果设置为 true，表示把 war 文件先展开再运行。如果为 false 则直接运行 war 文件。

[Reference](https://tomcat.apache.org/tomcat-8.0-doc/config/index.html)
