---
title: Spring Template
date: 2021-03-03 15:02:16
tags:
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

