---
title: Spring Session
date: 2021-03-05 18:20:39
tags:
- Spring Learning Notes
---
## Cookie

在客户端与服务端通讯的过程中，服务端既要返回`Cookie`给客户端，也要读取客户端提交的`Cookie`

### 读Cookie

为`control`类的方法增加一个`HttpServletRequest`参数，通过`request.getCookies()`取得`cookie`数组。然后再循环遍历数组即可

> 系统会自动传入方法参数所需要的`HttpServletRequest`对象

    import javax.sevlet.http.Cookie;
    import javax.sevlet.http.HttpServletRequest;

    @RequestMapping("/songlist")
    public Map index(HttpServletRequest request) {
        Map returnData = new HashMap();
        returnData.put("result", "this is song list");
        returnData.put("author", songAuthor);

        Cookie[] cookies = request.getCookies();
        returnData.put("cookies", cookies);

        return returnData;
    }

> cookie有很多属性值，各属性值的作用请[点击查看文档](https://fivemountain.github.io/2021/03/07/cookie%E7%9A%84%E9%87%8D%E8%A6%81%E5%B1%9E%E6%80%A7/)

### 使用注解读取cookie

为`control`类的方法增加一个`@CookieValue("xxxx") String xxxx`参数即可，注意使用时要填入正确的`cookie`名字

> 系统会自动解析并传入同名的`cookie`

    import org.springfarmework.web.bind.annotation.CookieValue;

    @RequestMapping("/songlist")
    public Map index(@CookieValue("JSESSIONID") String jSessionId) {
        Map returnData = new HashMap();
        returnData.put("result", "this is song list");
        returnData.put("author", songAuthor);
        returnData.put("JSESSIONID", jSessionId);

        return returnData;
    }

> 如果系统解析不到指定名字的cookie，使用次注解就会报错

### 写Cookie

为`control`类的方法增加一个`HttpServletResponse`参数，调用`response.addCookie()`方法添加`cookie`实例对象即可

> 系统会自动传入方法参数所需要的`HttpServletResponse`对象

    import javax.servlet.http.Cookie;
    import javax.servlet.http.HttpServletResponse;

    @RequestMapping("/songlist")
    public Map index(HttpServletResponse response) {
        Map returnData = new HashMap();
        returnData.put("result", "this is song list");
        returnData.put("name", songName);

        Cookie cookie = new Cookie("sessionId", "CookieTestInfo");
        // 设置的是cookie的域名，就是毁在哪个域名下生成cookie值
        cookie.setDomain("example.com");
        // 是cookie的路径，一般就是写到 / 
        cookie.setPath("/");
        // 设置cookie的最大存活时间，-1代表随浏览器的有效期
        cookie.setMaxAge(-1);
        // 设置是否只能服务器修改，浏览器端不能修改，安全有保障
        cookie.setHttpOnly(false);
        response.addCookie(cookie);

        returnData.put("message", "add cookie successful");
        return returnData;
    }

## Spring Session API

把用户ID、登录状态等重要信息放入`cookie`，会带来安全隐患。采用`Session`会话机制可以解决这个问题
使用会话机制时，`Cookie`作为`session id`的载体与客户端通信。前面代码中，cookie中的`JSESSIONID`就是这个作用

> 名字为JSESSIONID的cookie，是专门用来记录session的。JSESSIONID是标准的、通用的名字

### 读操作

与`cookie`相似，从`HttpServletRequest`对象中取得`HttpSession`对象，使用的语句是`request.getSession()`
但不同的是，返回结果不是数组，是对象。在`attribute`属性中用**key -> value**的形式存储多个数据

    import javax.servlet.http.HttpServletRequest;
    import javax.servlet.http.HttpServletResponse;
    import javax.servlet.http.HttpSession;

    @RequestMapping("/songlist")
    public Map index(HttpServletRequest request, HttpServletResponse response) {
        Map returnData = new HashMap();
        returnData.put("result", "this is song list");

        // 取得 HttpSession 对象
        HttpSession session = request.getSession();
        // 读取登录信息
        UserLoginInfo userLoginInfo = (UserLoginInfo)session.getAttribute("userLoginInfo");
        if (userLoginInfo == null) {
            // 未登录
            returnData.put("loginInfo", "not login");
        } else {
            // 已登录
            returnData.put("loginInfo", "already login");
        }

        return returnData;
    }

### 写操作

写入信息用`setAttribute()`方法

    import javax.servlet.http.HttpServletRequest;
    import javax.servlet.http.HttpServletResponse;
    import javax.servlet.http.HttpSession;

    @RequestMapping("/loginmock")
    public Map loginMock(HttpServletRequest request, HttpServletResponse response) {
        Map returnData = new HashMap();

        // 假设对比用户名和密码成功
        // 仅演示的登录信息对象
        UserLoginInfo userLoginInfo = new UserLoginInfo();
        userLoginInfo.setUserId("12334445576788");
        userLoginInfo.setUserName("ZhangSan");
        // 取得 HttpSession 对象
        HttpSession session = request.getSession();
        // 写入登录信息
        session.setAttribute("userLoginInfo", userLoginInfo);
        returnData.put("message", "login successfule");

        return returnData;
    }


> `Cookie`存放在客户端，一般不能超过4KB，放太多内容会导致出错；而`Session`存放在服务端，没有限制，不过基于服务端的性能考虑也不能放太多内容


## Spring Session 配置

前面关于`Session`的操作中，没有涉及到`Cookie`，系统会自动把默认的`JSESSIONID`放在默认的`cookie`中。但`Cookie`作为`session id`的载体，也可以修改属性

### 前置知识点：配置

前面提到过`application.properties`是`SpringBoot`的标准配置文件，配置一些简单的属性。同时，`SpringBoot`也提供了编程式的配置方式，主要用于配置`Bean`

    import org.springframework.context.annotation.Bean;
    import org.springframework.context.annotation.Configuration;

    @Configuration
    public class SpringHttpSessionConfig {
        @Bean
        public TestBean testBean() {
            return new TestBean();
        }
    }

在类上添加`@Configuration`注解，就表示这是一个配置类，系统会自动扫描并处理
在方法上添加`@Bean`注解，表示把此方法返回的对象实例注册成`Bean`

> 跟`@Service`等写在类上的注解一样，都表示注册Bean


### Session 配置

**依赖库**

先在`pom.xml`文件中增加依赖库

    <!-- spring session 支持 -->
    <dependency>
        <groupId>org.springframework.session</groupId>
        <artifactId>spring-session-core</artifactId>
    </dependency>

**配置类**

在类上额外添加一个注解`@EnableSpringHttpSession`，开启session。然后注册两个bean：
- `CookieSerializer`：读写Cookies中的SessionId信息
- `MapSessionRepository`：Session信息在服务器上的存储仓库


    import org.springframework.session.MapSessionRepository;
    import org.springframework.session.config.annotation.web.http.EnableSpringHttpSession;
    import org.springframework.session.web.http.CookieSerializer;
    import org.springframework.session.web.http.DefaultCookieSerializer;

    import java.util.concurrent.ConcurrentHashMap;

    @Configuration
    @EnableSpringHttpSession
    public class SpringHttpSessionConfig {
        @Bean
        public CookieSerializer cookieSerializer() {
            DefaultCookieSerializer serializer = new DefaultCookieSerializer();
            serializer.setCookieName("JSESSIONID");
            // 用正则表达式配置匹配的域名，可以兼容 localhost、127.0.0.1 等各种场景
            serializer.setDomainNamePattern("^.+?\\.(\\w+\\.[a-z]+)$");
            serializer.setCookiePath("/");
            serializer.setUseHttpOnlyCookie(false);
            // 最大生命周期的单位是分钟
            serializer.setCookieMaxAge(24 * 60 * 60);
            
            return serializer;
        }

        // 当前存在内存中
        @Bean
        public MapSessionRepository sessionRepository() {
            return new MapSessionRepository(new ConcurrentHashMap<>());
        }
    }

> [官方文档](https://docs.spring.io/spring-session/docs/current/reference/html5/guides/java-custom-cookie.html#custom-cookie-sample)


## Spring Request 拦截器

在实际项目中，如果每一个页面都判断是否登录，未登录跳转到登录页，就太繁琐了，也不利于维护，所以需要一种统一处理相同逻辑的机制
Spring提供了`HandlerInterceptor`(拦截器)满足这种需求

### 创建拦截器

拦截器必须实现`HandlerInterceptor`接口。可以在三个点进行拦截：

1. Controller方法执行之前。这是最常用的拦截点。例如是否登录的验证就要在`preHandle()`方法中处理
2. Controller方法执行之后。例如记录日志、统计方法执行时间等，就要在`postHandle()`方法中处理
3. 整个请求完成后。不常用的拦截点。例如统计整个请求的执行时间的时候用，在`afterCompletion`方法中处理


    import javax.servlet.http.HttpServletRequest;
    import javax.servlet.http.HttpServletResponse;
    import org.springframework.web.servlet.HandlerInterceptor;
    import org.springframework.web.servlet.ModelAndView;

    public class InterceptorDemo Implements HandlerInterceptor {
        // Controller方法执行之前
        @Override
        public boolean preHandle(HttpServletRequest request, HttpServletResponse response,
            Object handler) throws Exception {

            // 只有返回true才会继续向下执行，返回false取消当前请求
            return true;
        }

        // Controller方法执行之后
        @Override
        public void postHandle(HttpServletRequest request, HttpServletResponse response,
            Object handler, ModelAndView modelAndView) throws Exception {
            
        }

        // 整个请求完成之后(包括Thymeleaf渲染完毕)
        @Override
        public void afterCompletion(HttpServletRequest request, HttpServletResponse response,
            Object handler, Exception ex) throws Exception {
            
        }
    }

`preHandle()`方法的参数中有`HttpServletRequest`和`HttpServletResponse`，可以像`control`中一样使用`Session`

### 实现WebMvcConfigurer

创建一个类实现`WebMvcConfigurer`，并实现`addInterceptors()`方法。这个步骤用于管理拦截器

> 注意：实现类要加上`@Configuration`注解，让框架能自动扫描并处理

管理拦截器，比较重要的是为拦截器设置**拦截范围**。常用`addPathPatterns("/**")`表示拦截所有的`URL`
当然也可以调用`excludePathPatterns()`方法排除某些`URL`，例如登录页本身就不需要登录，需要排除

    import org.springframework.context.annotation.Configuration;
    import org.springframework.web.servlet.config.annotation.InterceptorRegistry;
    import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;

    @Configuration
    public class WebAppConfigurerDemo implements WebMvcConfigurer {
        @Override
        public void addInterceptors(InterceptorRegistry registry) {
            // 多个拦截器组成一个拦截器链
            // 仅演示，设置所有url都拦截
            registry.addInterceptor(new UserInterceptor()).addPathPatterns("/**");
        }
    }


> 学习拦截器，要注意理解和体会**拦截器**与**管理拦截器**分开的思想

通常，拦截器会放在一个包里(如`interceptor`)，而用于管理拦截器的配置类，会放在另一个包里(如`config`)。这种按功能划分子包的方式，可以比较清晰、直观的了解各个类的作用

> 知识点：`request.getAttribute("xxx")`方法的返回值是`Object`，需转换成实际的对象


