---
title: Spring依赖注入
date: 2021-03-02 16:11:02
tags:
- Spring Learning Notes
---
## Java注解(Annotation)
本质上来说Annotation(注解)是Java推出的一种注释机制。和普通注释的显著区别就是，Annotation可以在编译、运行阶段读取。因此，可以借助它来实现一些增强功能。
> Annotation类里可以继续引用其他的Annotation类

### 1、Target
`java.lang.annotation.Target`自身也是一个注解，它只有一个数组属性，用预设定该注解的目标范围，如可以作用于类或方法等。由于是数组，所以可以设定多个范围。
具体可以作用的类型配置在`java.lang.annotation.ElementType`枚举类中，常用的有：
- ElementType.TYPE
> 可以作用于类、接口类、枚举类上
- ElementType.FIELD
> 可以作用于类的属性上
- ElementType.METHOD
> 可以作用于类的方法上
- ElementType.PARAMETER
> 可以作用于类的参数

如果想同时作用于类和方法上，可以

    @Target({ElementType.TYPE, ElementType.METHOD})

### 2、Retention
`java.lang.annotation.Retention`自身也是一个注解，它用于声明该注解的生命周期，简单来说就是在Java编译、运行的哪个环节有效。它的值定义在`java.lang.annotation.RetentionPolicy`枚举类中，有三个值可选
1. SOURCE： 纯注释
2. CLASS：  在编译阶段有效
3. RUNTIME：在运行阶段有效 


    @Retention(RetentionPolicy.RUNTIME)
表示在运行时有效
> 自定义的Annotation，一般都设置成`RUNTIME` 

### 3、Documented
`java.lang.annotation.Documented`自身也是一个注解，它的作用是将注解中的元素包含到JavaDoc文档中，一般情况下都会添加这个注解。

### 4、@interface
`@interface`就是声明当前的Java类型是Annotation，固定语法。

### 5、Annotation属性

    String value() default "";
Annotation的属性有点像类的属性一样，它约定了属性的类型（这个类型是基础类型：String、boolean、int、long），和属性名称（默认名称是value，在饮用时可省略），default代表默认值。
有了这个属性，就可以正式引用一个Annotation了。例如：

    import org.springframework.stereotype.Service;

        @Service
        public class Demo {

    }

> Annotation也是Java类，所以一样需要import

上面的`@Service`也可以写成`@Service(value="Demo")`
因为value是默认属性名，可缩写，故等价于`@Service("Demo")`

## Spring Bean
IoC(Inversion of Control, 控制反转)容器是Spring框架最最核心的组件。

> IoC是面向对象编程中的一种设计原则，可以用来减低计算机代码之间的耦合度

在Spring框架中，主要通过依赖注入(Dependency Injection, DI)来实现IoC

在Spring的世界中，所有的Java对象都会通过IoC容器转变为Bean(Spring对象的一种称呼)，构成应用程序主干和由Spring IoC容器管理的对象成为beans。beans和它们之间的依赖关系反映在容器使用的配置元数据中。基本上所有的Bean都是有接口+实现类完成的，用户想要获取Bean的实例直接从IoC容器获取即可，无需关心实现类

Spring主要有两种配置元数据的方式，一种是基于XML，一种是基于Annotation

`org.springframework.context.ApplicationContext`接口类定义容器的对外服务，通过这个接口可以从IoC容器中得到Bean对象。在启动Java程序的时候必须先启动IoC容器

Annotation类型的IoC容器对应的类是

    org.springframework.context.annotation.AnnotationConfigApplicationContext

如果要启动IoC容器，可以运行下面的代码

    ApplicationContext context = new AnnotationConfigApplicationContext("fm.douban");
这段代码的含义就是启动IoC容器，并且会自动加载包`fm.douban`下的Bean(引用了Spring注解的类)

`AnnotationConfigApplicationContext`类的构造函数有两种
- `AnnotationConfigApplicationContext(String ... basePackages)`根据包名实例化
- `AnnotationConfigApplicationContext(Class clszz)`根据自定义包扫描行为实例化

Spring官方声明为Spring Bean的注解有以下几种：
- `org.springframework.stereotype.Component`
`@Component`注解是通用的Bean注解，其余三个都是扩展自Component
- `org.springframework.stereotype.Service`
`@Service`代表的是Service Bean
- `org.springframework.stereotype.Controller`
`@Controller`作用于Web Bean
- `org.springframework.stereotype.Repository`
`@Repsitory`作用于持久化相关Bean

**依赖注入的第一步是完成容器的启动，第二步就是真正的完成依赖注入行为了**

依赖注入使得到其他Bean实例相当简单，只需在属性上添加注解，例如

    @Autowired
    private SongService songService;
> Autowired完整的类路径是`org.springframework.beans.factory.annotation.Autowired`

当然还有一个前提条件，就是当前的类是Spring Bean(在类前添加合适注解，如`@Service`)

## Spring Resource
**文件处理方案**

当文件在电脑某个位置(e.g.`d:\temp\c.txt`)，或在工程目录下(e.g.`work/c.txt`)时，使用File对象就可以读写

    File file = new File("work/readme.md");

但当文件在工程的`src/main/resources`目录下时，由于Maven执行package会把resources目录下的文件一起打包进jar包里，此时显然用File对象是读取不到的。

#### classpath
在Java内部，一般把文件路径称为classpath，classpath指定的文件不能解析成File对象，但是可以解析成InputStream，借助Java IO就可以读取了

> 借助第三方类库，commons-io


    // 读取classpath的内容
    InputStream in = Test.class.getClassLoader().getResourceAsStream("c.txt");
    // 使用 commons-io库读取文本
    try{
        String content = IOUtils.toString(in, "utf-8");
        System.out.println(content);
    } catch (IOException e) {
        // IOUtils.toString有可能会抛出异常
        e.printStackTrace();
    }
这段代码含义就是从Java运行的类加载器(ClassLoader)实例中查找文件，`Test.class`指当前的Test.java编译后的class文件。


在Spring中定义了一个`org.springframework.io.Resource`类来封装文件，这个类既支持普通的File，也支持classpath文件。
并且在Spring中通过`org.springframework.core.io.ResourceLoader`服务来提供任意文件的读写。

    import fm.douban.service.FileService;
    import org.apache.commons.io.IOUtils;
    import org.springframework.beans.factory.annotation.Autowired;
    import org.springframework.core.io.Resource;
    import org.springframework.core.io.ResourceLoader;
    import org.springframework.stereotype.Service;

    import java.io.IOException;
    import java.io.InputStream;

    @Service
    public class FileServiceImpl implements FileService {
        @Autowired
        private ResourceLoader loader;

        @Override
        public String getContent(String name) {
            try{
                InputStream in = loader.getResource(name).getInputStream();
                return IOUtils.toString(in, "utf-8");
            } catch (IOException e) {
                return null;
            }
        }
    }

服务的调用

    FileService fileService = context.getBean(FileService.class);
    String content1 = fileService.getContent("classpath:data/urls.txt");
    System.out.println(content1);

    String content2 = fileService.getContent("file:mywork/readme.md");
    System.out.println(content2);

    String content3 = fileService.getContent("https://www.zhihu.com/question/34786516/answer/822686390");
    System.out.println(content3);

在Spring Resource中，把本地文件、classpath文件、远程文件都封装成Resource对象来统一加载

## Spring Bean的生命周期(Lifecycle)
实例化Bean -> 完成依赖注入 -> afterPropertiesSet -> Bean init -> destroy

大部分时候只需掌握`init`方法即可。`init`方法名称可以使任意的，因为`init`是通过注解来声明的

    @PostConstruct
    public void init() {
        System.out.println("服务启动");
    }
只需在方法上添加`@PostConstruct`注解，就代表该方法在Spring Bean启动后会自动执行

> PostConstruct的完整包路径是`javax.annotation.PostConstruct`


