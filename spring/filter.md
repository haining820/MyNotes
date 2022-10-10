---
title: Spring中的过滤器
date: 2022-10-10
tags: [SpringBoot,SpringMVC]
toc: true
---

# 1、FilterRegistrationBean 

FilterRegistrationBean ，SpringBoot 中的过滤器；通过 FilterRegistrationBean 实例注册，该方法能够设置过滤器之间的优先级。

<!--more-->

# 2、SpringMVC 的过滤器

实现 `javax.servlet.Filter` 接口，在 web.xml 中配置；

Filter 接口有三个方法：实现类必须必须重写这三个方法

- 初始化时调用 init：对 filtername 和 filterclass 进行处理，此方法只执行一次；

- 拦截请求时调用 doFilter，每次有满足条件的请求都会调用此方法；
- 销毁时调用 destroy：用户与服务器断开，或者 session 过时调用，此方法只执行一次。

## 2.1、乱码问题

<font size=4 style="font-weight:bold;background:yellow;">问题导入</font>

添加 form.jsp

```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>Title</title>
</head>
<body>

<form action="/e/t1" method="post">
    <input type="text" name="name">
    <input type="submit">
</form>

</body>
</html>
```

编写对应的处理类

```java
@Controller
public class EncodingController {
    @PostMapping("/e/t1")
    public String test1(String name, Model model){
        System.out.println(name);
        model.addAttribute("msg",name);  // 获取表单提交的值
        return "test";  // 跳转到test页面显示输入的值
    }
}
```

运行后在表单中输入中文后提交，会出现乱码。

![springmvc-filter-乱码](https://haining820-bucket.oss-cn-beijing.aliyuncs.com/typora_img/springmvc-filter-%E4%B9%B1%E7%A0%81.png)

<font size=4 style="font-weight:bold;background:yellow;">解决方法</font>

用 SpringMVC 提供的过滤器，可以解决乱码问题！

在 web.xml 中进行配置

```xml
    <!--配置SpringMVC的乱码过滤-->
    <filter>
        <filter-name>encoding</filter-name>
        <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
        <init-param>
            <param-name>encoding</param-name>
            <param-value>utf-8</param-value>
        </init-param>
    </filter>

    <filter-mapping>
        <filter-name>encoding</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>
```

也可以实现 Filter 接口，自己编写过滤器，同样在 web.xml 中进行配置

```java
public class EncodingFilter implements Filter {
    public void init(FilterConfig filterConfig) throws ServletException {}

    public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {
        servletRequest.setCharacterEncoding("utf-8");
        servletResponse.setCharacterEncoding("utf-8");
        servletResponse.setContentType("text/html;charset=utf-8");

        filterChain.doFilter(servletRequest,servletResponse);
    }

    public void destroy() {}
}
```

```xml
	<filter>
        <filter-name>EncoginFilter</filter-name>
        <filter-class>com.haining820.filter.EncodingFilter</filter-class>
    </filter>
    <filter-mapping>
        <filter-name>EncoginFilter</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>
```

**运行结果：**输入汉字后提交表单，也可以正常显示。

**注意：filter-mapping 设置时的范围必须是 `/*`**

