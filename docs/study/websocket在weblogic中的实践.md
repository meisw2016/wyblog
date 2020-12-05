一、问题描述

	项目使用Apache Shiro进行权限管理，同时用了Websocket。开发时部署到tomcat中一切正常。但部署到Weblogic后，Websocket出现异常，后台异常如下：

~~~
异常信息
GET /xxx/websocket.ws?k=123 HTTP/1.1
Connection: Upgrade
Pragma: no-cache
Cache-Control: no-cache
Upgrade: websocket
Origin: http://10.*.*.*:7001
Sec-WebSocket-Version: 13
User-Agent: Mozilla/5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/61.0.3163.79 Safari/537.36
Accept-Encoding: gzip, deflate
Accept-Language: zh-CN,zh;q=0.8
Sec-WebSocket-Key: c4W2n/E2VP5Yoq9CLTwuhw==
Sec-WebSocket-Extensions: permessage-deflate; client_max_window_bits

]] Root cause of ServletException.
java.lang.ClassCastException: org.apache.shiro.web.servlet.ShiroHttpSession cannot be cast to weblogic.servlet.security.internal.SessionSecurityData
        at weblogic.servlet.security.internal.SecurityModule.getCurrentUser(SecurityModule.java:197)
        at weblogic.websocket.tyrus.TyrusServletFilter.doFilter(TyrusServletFilter.java:167)
        at weblogic.servlet.internal.FilterChainImpl.doFilter(FilterChainImpl.java:78)
        at org.springframework.web.filter.CharacterEncodingFilter.doFilterInternal(CharacterEncodingFilter.java:197)
        at org.springframework.web.filter.OncePerRequestFilter.doFilter(OncePerRequestFilter.java:107)
        Truncated. see log file for complete stacktrace
~~~
二、解决办法

	通过分析异常日志，发现问题是由于类型不匹配导致。
~~~
java.lang.ClassCastException: org.apache.shiro.web.servlet.ShiroHttpSession cannot be cast to weblogic.servlet.security.internal.SessionSecurityData

ShiroHttpSession是Shiro里的类，因此怀疑问题是由于Shiro导致，再查看web.xml文件中Shiro的配置：

<filter>  
        <filter-name>shiroFilter</filter-name>  
        <filter-class>org.springframework.web.filter.DelegatingFilterProxy</filter-class>  
        <async-supported>true</async-supported>
        <init-param>  
            <param-name>targetFilterLifecycle</param-name>  
            <param-value>true</param-value>  
        </init-param>
    </filter>  
    <filter-mapping>  
        <filter-name>shiroFilter</filter-name>  
        <url-pattern>/*</url-pattern>
    </filter-mapping>

发现Shiro会对所有请求进行过滤，进一步怀疑是由于ShiroFilter导致异常，因此对filter-mapping进行修改，改为：

<filter-mapping>  
        <filter-name>shiroFilter</filter-name>  
        <url-pattern>*.do</url-pattern>
    </filter-mapping>
    <filter-mapping>
        <filter-name>shiroFilter</filter-name>
        <url-pattern>*.jsp</url-pattern>
    </filter-mapping>

只对后缀为.do和.jsp的请求进行拦截，Websocket监听的URL是.wx结尾的，因此修改后Websocket请求不会被ShiroFilter拦截到。

保存web.xml修改，重新部署后生效后。再次访问，一切正常，Websocket可以使用了，到此问题解决
~~~