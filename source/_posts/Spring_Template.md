---
title: Spring Template
date: 2021-03-03 15:02:16
categories:
- Learning Notes
- Spring
---
## Thymeleaf入门

Thymeleaf是一个模板框架，可以支持多种格式的内容动态渲染

通过模板引擎，可以把**Java对象数据+模板页面**动态的渲染出一个真实的HTML页面来

### 如何初始化Thymeleaf

1. **添加Maven依赖**


    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-thymeleaf</artifactId>
    </dependency>

2. **数据传递**
只需在方法参数里引入一个`Model`对象，就可以通过这个Model对象传递数据到页面中了


    import org.springframework.ui.Model;

    @Controller
    public class SongListControl {
        @Autowired
        private SongListService songListService;

        @RequestMapping("/songlist")
        public String index(@RequestParam("id")String id,Model model){
            SongList songList = songListService.get(id);
            // 传递歌单对象到模板当中
            // 第一个 songList 是模板中使用的变量名
            // 第二个 songList 是当前的对象实例
            model.addAttribute("songList",songList);

            return "songList";
        }
    }

3. **模板文件**
Spring MVC中对模板文件有固定的存放位置`src/main/resources/templates`
所以上面的`return "songList"`其实会去查找`src/main/resources/templates/songList.html`文件
模板文件内容如下


    <!DOCTYPE html>
    <html lang="en">
        <head>
            <meta charset="UTF-8" />
            <meta name="viewport" content="width=device-width, initial-scale=1.0" />
            <meta http-equiv="X-UA-Compatible" content="ie=edge" />
            <link rel="stylesheet" href="/css/songList.css" />
            <title>豆瓣歌单</title>
        </head>
        <body>
            <h1 th:text="${songList.name}"></h1>
        </body>
    </html>

## Thymeleaf变量

上面用到的`th:text="${songList.name}"`就是使用变量技术

为了不破坏HTML结构，Thymeleaf采用自定义HTML属性的方式来生成动态内容

`th:text`这个属性就是Thymeleaf自定义的HTML标签属性，`th`是Thymeleaf的缩写
`th:text`语法的作用就是会动态替换掉HTML标签的内部内容

一般情况下，模板的变量都是会存放在模板上下文中，所以想要调用变量，就需要先设置变量到模板上下文中去

    model.addAttribute("songList", songList);

**对象变量**

模板语言还可以支持复杂对象的输出，可以使用`.`把属性调用出来

    <span th:text="${song.id}"></span>
> 如果对象包含对象，可以用`.`一直调出来，前提是这个对象是POJO类

## Thymeleaf循环语句

    <ul th:each="song : ${songs}">
        <li th:text="${song.name}">歌曲名称</li>
    </ul>

Thymeleaf除了支持List数据类型，还可以支持数组、Map(类似Java的for语句，集合类都支持)

### 打印列表的索引值

    <ul th:each="song, it : ${songs}">
        <li>
            <span th:text="${it.count}"></span>
            <span th:text="${song.name}"></span>
        </li>
    </ul>

`it`为可选参数，如果定义了就可以通过这个it对象获取更多关于统计的需求，具体参数如下
- `it.index`
> 当前迭代对象的index(从0开始计算)，从0开始显示行数
- `it.count`
> 当前迭代对象的index(从1开始计算)，显示行数
- `it.size`
> 被迭代对象的大小，获取列表长度
- `it.current`
> 当前迭代变量，等同于`song`
- `it.even/odd`
> 布尔值，当前循环是否是偶数/奇数(从0开始计算)
- `it.first`
> 布尔值，当前循环是否是第一个
- `it.last`
> 布尔值，当前循环是否是最后一个

模板中的布尔值需要结合条件语句来处理

## Thymeleaf表达式

主要用于两种场景
- 字符串处理
- 数据转化

### 字符串处理

    // 字符串拼接
    <span th:text="${'00:00/' + ${totalTime}}"></span>

    // 拼接优化
    <span th:text="|00:00/${totalTime}|"></span>

### 数据转化

处理`LocalDate`和`LocalDateTime`类的依赖

    <dependency>
        <groupId>org.thymeleaf.extras</groupId>
        <artifactId>thymeleaf-extras-java8time</artifactId>
        <version>3.0.4.RELEASE</version>
    </dependency>
这个库会自动添加一个新的工具类`temporals`

变量使用的是`${变量名}`，工具类使用的是`#{工具类}`

**dates/temporals**

dates和temporals支持的方法是一样的，只是支持的类型不同
dates支持的是Date类，temporals支持的是LocalDate和LocalDateTime

> java.util.Date类和LocalDateTime类功能是一样的，不同的是LocalDateTime是Java8才出现的，一些老的应用还是用Date类

一般使用dates/temporals用于处理日期类型到字符串的转化，比如显示`年月日`

    <p th:text="${#dates.format(dateVar, 'yyyy-MM-dd HH:mm:ss')}"></p>
    <p th:text="${#dates.format(dateVar, 'yyyy年MM月dd日 HH时mm分ss秒')}"></p>

如果日期类型是`LocalDate/LocalDateTime`就把`#dates`换成`#temporals`

**strings**

除了日期方法，`#strings`也是使用较多的，支持字符串的数据处理
- `${#strings.toUpperCase(name)}`
> 把字符串改成全大写
- `${#strings.toLowerCase(name)}`
> 把字符串改成全小写
- `${#strings.arrayJoin(array, ',')}`
> 把数组合并成一个字符串，并以`,`连接
- `${#strings.arraySplit(str, ',')}`
> 把字符串分隔成一个数组，并以`,`作为分隔符
- `${#strings.trim(str)}`
> 去掉字符串左右两边空格
- `${#strings.length(str)}`
> 得到字符串的长度，也支持获取集合类的长度
- `${#strings.equals(str1, str2)}`
> 比较两个字符串是否相等
- `${#strings.equalsIgnoreCase(str1, str2)}`
> 忽略大小写后比较两个字符串是否相等

**内联表达式**

    <span>Hello [[${msg}]]</span>
上面的`[[变量]]`这种格式就是内联表达式，支持直接在HTML中调用变量

> `[[]]`是用来替代`th:text`的，不能替代所有`th:`标签

## Thymeleaf条件语句

    // 条件为true 执行渲染
    <span th:if="${user.sex == 'male'}">男</span>

    // 条件为false 执行渲染
    <span th:unless="${user.sex == 'male'}">女</span>

### strings逻辑判断

**isEmpty**

检查字符串变量是否为空(or null)，在检查之前会先执行trim()操作，去掉空格

    ${#strings.isEmpty(name)}

    ${#strings.arrayIsEmpty(name)}  // 数组

    ${#strings.listIsEmpty(name)}   // 集合

**contains**
检查字符串变量是否包含片段

    ${strings.contains(name, 'abc')}

其它常用的`#strings`判断语法
- `${#strings.containsIgnoreCase(name, ''abc)}`
> 忽略大小的情况下，判断是否包含指定的字符串
- `${#strings.startsWith(name, 'abc')}`
> 判断字符串是否以abc开头
- `${#strings.endsWith(name, 'abc')}`
> 判断字符串是否以abc结束

## Spring Validation

处理表单数据的验证

### JSR 380

JSR是Java Specification Requests的缩写，意思是Java规范提案。指向JCP(Java Community Process)提出新增一个标准化技术规范的正式请求。
JSR 380就是Bean Validation 2.0，这个就是Bean验证的规范，这里的Bean就是实例化后的POJO类。
JSR 380提案的规范可以通过下面的依赖添加到工程里

    <dependency>
        <groupId>jakarta.validation</groupId>
        <artifactId>jakarta.validation-api</artifactId>
        <version>2.0.1</version>
    </dependency>

Spring Validation也是JSR 380提案的一个实现方案

### Validation注解

JSR 380定义了一些注解用于数据校验，这些注解可以直接设置在Bean属性上
- `@NotNull`
> 不允许为null对象
- `@AssertTrue`
> 是否为true
- `@Size`
> 约定字符串的长度
- `@Min`
> 字符串的最小长度
- `@Max`
> 字符串的最大长度
- `@Email`
> 是否是邮箱格式
- `@NotEmpty`
>  不允许为null或为空，可用于判断字符串、集合，如Map、数组、List
- `@NotBlank`
> 不允许为null和空格


    import javax.validation.constraints.*;

    public class User {
        @NotEmpty(message = "名称不能为null")
        private String name;

        @Min(message = "年龄必须大于等于18岁")
        @Max(message = "年龄必须小于等于120岁")
        private int age;

        @NotEmpty(message = "邮箱必须输入")
        @Email(message = "邮箱不正确")
        private String email;

        // standard setters and getters
    }
> 大多数情况下，建议使用NotEmpty替代NotNull、NotBlank

校验的注解是可以累加的，系统会按顺序执行校验

### 执行校验

    @Controller
    public class UserControl {
        @GetMapping("/user/add.html")
        public String addUser() {
            return "user/addUser";
        }

        @PostMapping("/user/save")
        public String saveUser(@Valid User user, BindingResult errors) {
            if(errors.hasErrors()) {
                // 校验不通过，返回用户编辑界面
                return "user/addUser";
            }

            // 校验通过
            return "user/addUserSuccess";
        }
    }
> @Valid 完整包路径`javax.validation.Valid;`
> BindingResult 类的路径是`org.springframework.validation.BindingResult`


> 当数据存储成功后，跳转至某页面，这个逻辑一般借助页面跳转`redirect`实现

    return "redirect:/user/list";

**th:object**

为了让表单验证状态生效，需要在`form`标签里增加一个`th:object="${user}"`属性

> `th:object`用于替换对象，使用了这个就不需要每次都编写`user.xxx`，可以直接操作`xxx`了

**th:classappend**

现在需要动态的管理表单的样式。如果有错误就在该标签上增加表示错误的`css class`，否则不处理

    <div th:classappend="${#fields.hasErrors('name')} ? 'error' : ''">
    </div>
如果错误信息里有name字段，上面的代码就会生成

    <div class="error">
    </div>
注意`${#fields.hasErrors('key')}`这个语法是专门为验证场景提供的，这里的`key`就是对象的属性名称，比如User对象的name、age等

**th:errors**

如果还想要显示出错误信息，可以使用`th:errors="*{age}"`属性，这个会自动取出错误信息

    <p th:if="${#fields.hasErrors('age')}" th:errors="*{age}"></p>

> 注意这里的`*`由于在form中使用了`th:object="${user}"`，所以就可以通过`*{age}`来获取具体的属性值

**th:field**

一般错误的时候，希望能显示上一次输入的内容，此时可以使用th:field语法

    <div th:classappend="${#fields.hasErrors('age')} ? 'error' : ''">
        <label>年龄：</label>
        <input type="text" th:field="*{age}" />
        <p th:if="${#fields.hasErrors('age')}" th:errors="{age}"></p>
    </div>

## Thymeleaf布局(Layout)
这里布局指网站的页面架构。layout解决的是模板复用的问题

推荐使用`th:include + th:replace`方案完成布局的开发

### layout.html

    <!DOCTYPE html>
    <html xmlns:th="http://www.thymeleaf.org">
        <head>
            <meta charset="UTF-8">
            <meta name="viewpoint" content="width=device-width, initial-scale=1.0" />
            <title>布局</title>
            <style>
                .header {background-color: #f5f5f5; padding: 20px;}
                .header a {padding: 0 20px;}
                .container {padding: 20px; margin: 20px auto;}
                .footer {height: 40px; background-color: #f5f5f5; border-top:1px solid #ddd; padding: 20px;}
            </style>
        </head>
        <body>
            <header class="header">
                <div>
                    <a href="/book/list.html">图书管理</a>
                    <a href="/user/list.html">用户管理</a>
                </div>
            </header>
            <div class="container" th:include="::content">页面正文内容</div>
            <footer>
                <div>
                    <p style="float: left">&copy; youkeda.com 2017</p>
                    <p style="float: right">Powered by 优课达</p>
                </div>
            </footer>
        </body>
    </html>
重点在container节点上，使用了`th:include="::content"`语法

**th:include="::content"**

`::content`指的是选择器，这个选择器指的就是加载当前页面的`th:fragment`的值
当页面渲染的时候，布局会合并content这个fragment内容一起渲染，下面配置fragment

### user/list.html

    <!DOCTYPE html>
    <html xmlns:th="http://www.thymeleaf.org" th:replace="layout">
        <div th:fragment="content">
            <h2>用户列表</h2>
            <div>
                <a href="/user/add.html">添加用户</a>
            </div>
            <table>
                <thead>
                    <tr>
                        <th>
                            用户名称
                        </th>
                        <th>
                            用户年龄
                        </th>
                        <th>
                            用户邮箱
                        </th>
                    </tr>
                </thead>
                <tbody>
                    <tr th:each="user : ${users}">
                        <td th:text="${user.name}"></td>
                        <td th:text="${user.age}"></td>
                        <td th:text="${user.email}"></td>
                    </tr>
                </tbody>
            </table>
        </div>
    </html>

**th:replace="layout"**

指定了布局的名称，这个一旦声明后，页面会被替换成layout的内容。注意不要指定布局名称错误，这里`"layout"`指的是`templates/layout.html`

**th:fragment="content"**

fragment是片段的意思，当页面渲染的时候，可以通过选择器指定使用这个片段。在`layout.html`文件的`th:include="::content"`指定的就是这个值


执行JS操作
    <a href="javascript:;" th:onclick="delBook([[${book.id}]])">删除</a>

其中，Thymeleaf对HTML属性的动态赋值的操作方法
    th:onclick="delBook([[${book.id}]])"
> 由于超链接有默认的打开链接行为，所以对于想要执行onclick的时候，一般设置`href="javascript:;"`
