Title: 配置Tomcat的访问日志格式化输出
Date: 2014-08-02 13:23


在tomcat的server.xml文件中，Host主机配置区域找到类似如下(Value标签元素)即为访问日志的配置：

``` xml
<Host name="localhost"  appBase="webapps"
	unpackWARs="true" autoDeploy="true">
	<!--...部分内容略..-->
	<Valve className="org.apache.catalina.valves.AccessLogValve"
		directory="logs"
		prefix="localhost_access_log." suffix=".txt"
		pattern="%h %l %u %t &quot;%r&quot; %s %b" />
 </Host>
```

其中的directory用于指定日志的存放路径，默认位于tomcat的logs目录中，例如我们可以修改成：`directory="/var/log/tomcat"` 使日志放到/var/log/tomcat目录中去。
其中的prefix和suffic分别用于指定日志文件的前缀和后缀，不用我多说。
现在我们主要来看一下pattern配置段，它用于指定日志的输出格式。有效的日志格式模式可以参见下面内容，如下字符串，其对应的信息由指定的响应内容取代：

``` bash
％a - 远程IP地址％A - 本地IP地址
％b - 发送的字节数，不包括HTTP头，或“ - ”如果没有发送字节
％B - 发送的字节数，不包括HTTP头
％h - 远程主机名
％H - 请求协议
％l (小写的L)- 远程逻辑从identd的用户名（总是返回' - '）
％m - 请求方法
％p - 本地端口
％q - 查询字符串（在前面加上一个“？”如果它存在，否则是一个空字符串
％r - 第一行的要求
％s - 响应的HTTP状态代码
％S - 用户会话ID
％t - 日期和时间，在通用日志格式
％u - 远程用户身份验证
％U - 请求的URL路径
％v - 本地服务器名
％D - 处理请求的时间（以毫秒为单位）
％T - 处理请求的时间（以秒为单位）
％I （大写的i） - 当前请求的线程名称
```

此外，您可以指定以下别名来设置为普遍使用的模式之一：

``` bash
common - %h %l %u %t "%r" %s %b
combined - %h %l %u %t "%r" %s %b "%{Referer}i" "%{User-Agent}i"
```

另外，还可以将request请求的查询参数、session会话变量值、cookie值或HTTP请求/响应头内容的变量值等内容写入到日志文件。
它仿照了apache的语法：

``` bash
％{XXX}i xxx代表传入的头(HTTP Request)
％{XXX}o xxx代表传出的响应头(Http Resonse)
％{XXX}c xxx代表特定的Cookie名
％{XXX}r xxx代表ServletRequest属性名
％{XXX}s xxx代表HttpSession中的属性名
```

举例说明：
例如我们在架设jsp服务器时，采用Nginx+Tomcat这种配置时，将请求由Nginx转发给Tomcat，当需要在tomcat的日志中记录来访者的真实IP地址信息时，我们就需要做一点点有别于其它的特殊匹配了，要不然tomcat记录的访客IP全都是127.0.0.1, 这是因为所有的请求都是由Nginx前端服务器转发而来的，而前端服务器对于tomcat来说就是127.0.0.1。

下面，我们来看一下如何让tomcat记录用户的真实IP地址：
一、配置Nginx转发IP头：
在Nginx的server主机配置段中添加：

``` bash
proxy_set_header      Host $host;
proxy_set_header      X-Real-IP $remote_addr;
```

说明：上面两行用于向tomcat发送真实的远端主机名和IP地址。其中的Host代表主机名， X-Real-IP代表主机IP，对于HTTP头部内容，这些变量是不区分大小写的。

二、配置Tomcat日志记录客户真实IP：
在Tomcat中要记录来访者真实IP，大家参考上面所述的tomcat日志配置语法，只需在日志模式中添加如下模式就行了：%{X-Real-IP}i
如下面完整的Tomcat日志配置段：

``` xml
<Valve className="org.apache.catalina.valves.AccessLogValve"
directory="c:/wwwlogs/" prefix="cluster." suffix=".log"
pattern="%{X-Real-IP}i %u %t %r %s %b" resolveHosts="false" />
```

注意：上两两处修改后，您应该重新启动Nginx和Tomcat服务，以使您的修改生效。这样，当有新的请求过来时，便可以在Tomcat日志文件中记录访客的真实IP地址了。
