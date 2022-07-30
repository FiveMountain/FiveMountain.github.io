---
title: Spring Boot 入门
date: 2021-03-05 15:40:29
categories:
- Learning Notes
- Spring
---
## Spring Boot ComponentScan

加了`@SpringBootApplication`注解的类是启动类，是整个系统的启动入口
Spring Boot框架会默认扫描**启动类所在的包及其所有子包**进行解析
在其之外的包则不会自动扫描，也不会自动实例化`Bean`

**解决办法**

因此可以为启动类的注解`@SpringBootApplication`加一个参数，告知系统需要额外扫描的包

    @SpringBootApplication(scanBasePackages={"...", "..."})
    public class AppApplication {
        public static void main(String[] args) {
            SpringApplication.run(AppApplication.class, args);
        }
    }
- 参数名是`scanBasePackages`
- 参数值是一个字符串数组，用于指定所有需要自动扫描的包

**另一种写法**

如果不是`Spring Boot`启动类，可以使用独立的注解`@ComponentScan`，作用也是用于指定多个需要额外自动扫描的包。

    @ComponentScan({"...", "..."})
    public class SpringConfiguration {
        ... ...
    }

**额外知识点**

使用`@RestController`的类，所有方法都不会渲染`Thymeleaf`页面，而是都返回数据。等同于使用`@Controller`的类的方法上添加`@ResponseBody`注解，效果是一样的。但两个注解的包不同

    org.springframework.stereotype.Controller;
    org.springframework.web.bind.annotation.RestController;

## Spring Boot Logger 运用

在`Spring`这种比较复杂的系统中，`System.out.println()`打印的内容会输出到什么地方是不确定的。所以在企业级的项目中，都用日至系统来记录信息

日志系统的两大优势：
1. 可以轻松控制日志是否输出
2. 可以灵活地配置日志的细节，例如输出格式等

### 1. 配置

修改`Spring Boot`系统的标准配置文件`application.properties`(在项目的`src/main/resources`目录下)，增加日志级别配置：

    logging.level.root=info
表示所有日志(`root`)都为`info`级别
也可以为不同的包定义不同的级别

    logging.level.com.test.app=info
就表示`com.test.app`包及其子包的所有类都输出`info`级别的日志

常见的日志级别有

- ERROR (最高)
> 错误信息日志
- WARN  (高)
> 暂时不出错但高风险的警告信息日志
- INFO  (中)
> 一般的提示语、普通数据等不紧要的信息日志
- DEBUG (低)
> 进开发阶段需要关注的调试信息日志

**级别的作用**

`logging,level.root=warn`意味着不输出更低优先级的INFO、DEBUG日志，只输出WARN和更高优先级的ERROR日志。其余类比
在开发阶段配置为`DEBUG`，在项目发布时调整为`INFO`或更高级别，即可做到不改代码而控制只输出关心的日志

### 2. 编码

配置完成后，编码只需要实例化日志对象即可打印日志

    import org.slf4j.Logger;
    import org.slf4j.LoggerFactory;
    import org.springframework.web.bind.annotation.RestController;
    import javax.annotation.PostConstruct;

    @RestController
    public class SongListControl {
        private static final Logger LOG = LoggerFactory.getLogger(SongListControl.class);

        @PostConstruct
        public void init() {
            LOG.info("SongListControl 启动了");
        }
    }

先定义一个类变量`LOG`，然后在`LOG.info()`方法的参数中输入日志内容
注意这里的方法名(`info()`)与日志级别一一对应

定义类变量`LOG`的语句属于固定写法，至于要在不同的类中，修改`getLogger()`方法参数为当前的类名即可。这样就能识别每段日志的来源。

> 修饰为`static final`的目的就是尽量复用，一个类无论多少个实例只需要一个日志对象。日志对象一旦定义也不需要修改

## Spring Boot Properties

`Spring Boot`框架已经免除了大部分手动配置，但对于一些特定的情况，还是需要进行手动配置。
框架提供了`application.properties`配置文件，让我们可以进行自定义配置。这是一个固定位置(在项目`src/main/resources`目录下)、固定名字的文件，框架会自动加载并解析这个配置文件

### 配置文件格式

每一行是一条配置项：配置项名称=配置项值
> 注意：等号两边不要加空格，要写紧凑一些

配置文件书写约定：
- 配置项名称能准确表达作用、含义，以`.`分隔单词
- 相同前缀的配置项写在一起
- 不同前缀的配置项之间空一行

### 配置的意义

配置的主要作用，是把**可变**的内容从代码中剥离出来，做到在不修改代码的情况下，方便的修改这些可变的或常变的内容。这个过程称之为避免硬编码、做到解耦

### 自定义配置项

    song.name=God is a girl

使用：

    import org.springframework.beans.factory.annotation.Value;

    public class SongListControl {
        @Value("${song.name}")
        private String songName;
    }

注意写法，花括号中的配置项名称，与配置文件中保持一致

