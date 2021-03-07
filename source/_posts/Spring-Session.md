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


