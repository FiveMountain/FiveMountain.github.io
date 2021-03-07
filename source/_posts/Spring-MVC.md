---
title: Spring MVC
date: 2021-03-03 06:45:53
tags:
- Spring Learning Notes
---
## Spring Boot 介绍
Spring Boot的核心还是Spring(所以无需单独去管理Spring的Maven依赖)，只是多了一些工程化的方案：
- 如Java Web容器的嵌入集成，Spring Boot默认集成了Tomcat
- Spring Boot还自定义了工程打包格式，通过这个直接把一个Java Web工程转化成普通的Java工程，启动一个main方法就可以把Spring工程启动起来
- Spring Boot默认集成了常用第三方框架和服务，如数据库连接，NoSQL，安全等
- Spring Boot还提供了标准的属性配置文件，支持应用的参数动态配置

创建Spring Boot工程：[https://start.spring.io/](https://start.spring.io/)

## Spring Controller 入门
在Spring Boot方案里，一个网页请求到了服务器后，首先进入的是Java Web服务器，然后进入到Spring Boot应用，最后匹配到某一个Spring Controller(也是一个Spring Bean)，然后路由到具体某一个Bean的方法，执行完后返回结果，输出到客户端。
从这个流程里可以看出，只要掌握Spring Controller就可以自己提供Web服务了

Spring Controller技术有三个核心点：
- Bean的配置：Controller注解运用
- 网络资源的加载：加载网页
- 网址路由的配置：RequestMapping注解运用

### 1. Controller注解
Spring Controller本身也是一个Spring Bean，只是它多提供了Web能力。
只需要在类上提供一个`@Controller`注解就可以了

> Spring大多数是基于接口开发的，但是没有接口也是可以工作的。Spring Controller一般情况下不需要特意实现接口

### 2. 加载网页
在Spring Boot应用中，一般把网页存放在`src/main/resources/static`目录下。
在Controller中，会**自动加载static下**的HTML内容，所以通过Spring Boot建设网站也非常简单

    import org.springframework.stereotype.Controller;

    @Controller
    public class HelloControl {
        public String say() {
            return "hello.html";
        }
    }

在上面代码中，say()方法返回类型为`String`
`return "hello.html";`返回的是HTML文件路径

> 执行这段代码的时候，Spring Boot实际加载的是`src/main/resources/static/hello.html`文件
> 
> 文件路径不需要额外添加`static`

**static子目录**

如果HTML文件路径为`src/.../static/html/hello.html`，那么return语句应为

    return "html/hello.html";

### 3. RequestMapping注解
**路由**：对于Web服务器来说，必须要实现的一个能力就是解析URL，并提供资源内容给调用者。这个过程一般称为路由

Spring MVC完美支持了路由能力，并简化了路由配置，只需要在需要提供Web访问的方法上添加一个`@RequestMapping`注解即可

    import org.springframework.stereotype.Controller;
    import org.springframework.web.bind.annotation.RequestMapping;

    @Controller
    public class HelloControl {
        @RequestMapping("/hello")
        public String say() {
            return "html/hello.html";
        }
    }

一般情况下，会把controller类存放在`control`子包里

## Get Request
每个HTTP URL都可以设定自定义的参数，e.g.`https://www.baidu.com/s?wd=test`中的`wd`
参数要求能索引到唯一一个结果

### 定义参数

在Spring MVC中，定义一个URL参数只需在方法上添加对应的参数和参数注解即可

    @Controller
    public class SongListControl {
        @RequestMapping("/songList")
        public String index(@RequestParam("id") String id) {
            return "html/songList.html";
        }
    }
代码在之前的基础上添加了`@RequestParam("id")`
注解的参数`"id"`这个值必须和URL的param key一致

> `RequestParam`注解的包路径是`org.springframework.web.bind.annotation.RequestParam`

> 默认情况下，URL的参数传递到服务器端都是变成字符串的

### 获取多个参数
添加多个参数即可

    @RequestMapping("songList")
    public String index(@RequestParam("id") String id, @RequestParam("pageNum") int pageNum) {
        ...
    }
> 基础的boolean、int、String数据类型可以直接自动转化

如果在Request方法声明了多个参数，那么URL访问的时候就必须要传递该参数，否则访问不到

### @GetMapping
`@RequestMapping`注解默认支持所有的HTTP Method。放开所有的Http Method不是很安全，一般还是会明确指定method，如get请求
可以使用`@GetMapping`替换`@RequestMapping`

### 非必须传递参数
如果访问URL时不想某个参数必须传递，可以修改一下参数的注解

    (@RequestParam(name="pageNum",required=false) int pageNum, @RequestParam("id") String id)

> 方法定义参数的顺序和URL的参数顺序无关

### 输出JSON数据

    @GetMapping("/api/foos")
    @ResponseBody
    public String getFoos(@RequestParam("id") String id) {
        return "ID: " + id;
    }

返回Java对象的方式

    public class User{
        private String id;
        private Stirng name;
        // 省略了 getter、setter
    }

    public class UserControl{
        //缓存 User 数据
        private static Map<String,User> users = new HashMap();

        /**
         * 初始化数据
         */
        @PostConstruct
        public void init(){
            User user = new User();
            user.setId("100");
            user.setName("ykd");
            users.put(user.getId(),user);
        }

        @GetMapping("/api/user")
        @ResponseBody
        public User getUser(@RequestParam("id") String id) {
            return users.get(id);
        }
    }
Spring MVC会自动把对象转化成JSON字符串输出到网页
一般把这种输出JSON数据的方法称为API
