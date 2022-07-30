---
title: Spring入门
date: 2021-03-02 08:51:57
categories:
- Learning Notes
- Spring
---
## Maven入门
### Maven命令
> 命令要在工程的根目录下执行
1. `mvn clean compile`

编译命令，Maven会自动扫描src/main/java下的代码并完成编译工作，执行完，会在根目录下生成target/classes目录（存放所有的class）。

2. `mvn clean package`

compile和package的集合，会限制性compile命令，然后执行jar打包命令。

3. `mvn clean install`

compile、package和install的集合，执行compile、jar打包命令后，执行install命令安装到本地Maven仓库中，目录是`${user_home}/.m2`

4. `mvn compile exec:java -Dexec.mainClass=${main}`

在compile执行完后，运行Java的命令。具体执行的Java类由`-Dexec.mainClass=${main}`参数指定。

### Maven核心概念
Maven的配置文件是一个强约定的XML格式文件，文件名一定是`pom.xml`。

#### 1、POM(Project Object Model)
一个Java项目所有的配置都放置在POM文件中，大概有如下的行为：

- 定义项目的类型、名字
- 管理依赖关系
- 定制插件 

##### 1.1 Maven坐标

    <groupId> </groupId>
    <artifactId> </artifactId>
    <packaging> </packaging>
    <version> </version>

**groupId**
一般由小写的英文字母和字符`.`组成，如`com.fivemount.test`。一般来说一个公司会设置自己的groupId，避免和其他公司重合，个人开发者也一样。

**artifactId**
在一个groupId内，应该是唯一的。从规范上来说只能使用小写字母、`.`、`-`、`_`。

**packaging**
Maven工程执行完后会把整个工程打包成`packaging`指定的文件格式，如果没有声明这个标签，默认为`jar`
packaging有以下几种格式：

- jar
- war
- ear
- pom 

**version**
基本上遵循了软件工程中对版本号的约定。

- **SNAPSHOT**
快照。代表当前程序还处于不稳定阶段。
- **RELEASE**
代表程序处于稳定版本。 
**版本号的约定：**在软件工程里，一般用三位数字表示版本号：`x.x.x`。
- 第一位代表的是主版本号
> 主版本号一般由团队约定
- 第二位代表的是新增功能
> 表示在此版本中有几次新增功能的行为
- 第三位代表的是bugfix后的版本 
> bugfix是修复代码缺陷、bug的行为

##### 1.2 Maven属性配置
Maven的属性配置就是用来做参数设置的。

    <properties>
        <java.version>1.8</java.version>
        <maven.compiler.source>${java.version}</maven.compiler.source>
        <project.build.sourceEncoding>UTD-8</project.build.sourceEncoding>
        <maven.compiler.target>${java.version}</maven.compiler.target>
    </properties>
首先它的格式是在`properties`标签内，这是固定的格式。`properties`内的标签可以自定义，但一般只能是小写字母+`.`
一些公共参数：
- java.version
代表设置一个参数：`java.version`，它的值是1.8
- maven.compiler.source
指定Maven编译时源代码的JDK版本
- project.build.sourceEncoding
指定工程代码源文件的**文件编码格式**，一般情况下，设置为`UTF-8`
- maven.compiler.target 
这个参数作用是按照这个JDK版本进行编译源代码 

#### 2、依赖管理 dependencies
dependency就是用于指定当前工程依赖其它代码库的，Maven会自动管理jar依赖
> 一旦在pom.xml里声明了dependency信息，会先去本地用户目录下的`.m2`文件夹内查找对应的文件，如果没有找到就会触发从中央仓库下载行为，下载完存在本地的`.m2`文件夹内 

我们只需在pom.xml里添加标签即可。首先声明一个父标签dependencies，然后就可以在这个标签内部添加依赖了。例如添加`fastjson`库的依赖：

    <dependencies>
        <dependency>
	    <groupId>com.alibaba</groupId>
	    <artifactId>fastjson</artifactId>
	    <version>1.2.62</version>
        </dependency>
    </dependencies> 

注意，**一个pom.xml只能存在一个dependencies标签**,可以有多个dependency
dependency标签的内容其实就是Maven坐标
> 一般把别人写的代码库称为**三方库**，自己、团队写的称为**二方库**
##### 2.1 中央仓库
Maven会把所有的jar都存放在中央仓库里，可以通过[https://search.maven.org/](https://search.maven.org/)这个网站来搜索jar。这个网站部署在国外，如果无法访问，可以使用阿里云的镜像服务器[https://maven.aliyun.com/mvn/search](https://maven.aliyun.com/mvn/search)
##### 2.2 间接依赖
如果一个`remote`工程依赖了okhttp库，而当前工程`locale`依赖了`remote`工程，这时`locale`工程也会自动依赖okhttp
#### 3、插件体系 plugins
插件体系让Maven变得高度可定制，可以无缝的支撑工程化能力。

    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compile-plugin</artifactId>
                <version>3.8.1</version>
            </plugin>
        </plugins>
    </build> 
这里声明了一个`maven-compile-plugin`插件用于执行maven compile的，Maven的插件其实也是存放在中央仓库的坐标，也就是一切都是jar。

### Hello Spring
Spring强调的是**面向接口编程**，所以大多数情况下Spring代码都会有接口和实现类

Spring当中，可直接调用接口，而无需关心它的实现类、如何实例化实现类，从而达到真正的解耦合
这也是Spring最大的价值，完全的屏蔽了实现细节，让使用者可以专注于接口的定义和运用。

    public static void main(String[] args) {
        ApplicationContext context = new AnnotationConfigApplicationContext(Application.class);
        MessageService msgService = context.getBean(MessageService.class);
        System.out.println(msgService.getMessage());
    } 

