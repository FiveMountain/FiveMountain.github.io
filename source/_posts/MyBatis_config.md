---
title: MyBatis配置
date: 2021-04-29 14:41:36
categories:
- Pages
---
## resources

**mybatis-config.xml:**

    <?xml version="1.0" encoding="UTF-8" ?>
    <!DOCTYPE configuration
            PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
            "http://mybatis.org/dtd/mybatis-3-config.dtd">
    <configuration>
        <properties resource="db.properties" />
        
        <settings>
            <setting name="logImpl" value="LOG4J" />
        </settings>

        <typeAliases>
            <package name="com.example.model"/>
        </typeAliases>

        <environments default="development">
            <environment id="development">
                <transactionManager type="JDBC"/>
                <dataSource type="POOLED">
                    <property name="driver" value="${driver}"/>
                    <property name="url" value="${url}"/>
                    <property name="username" value="${username}"/>
                    <property name="password" value="${password}"/>
                </dataSource>
            </environment>
        </environments>

        <mappers>
            <mapper resource="com/example/mapper/UserMapper.xml"/>
        </mappers>
    </configuration>

**db.properties:**

    driver=com.mysql.cj.jdbc.Driver
    url=jdbc:mysql://localhost:3306/test?userSSL=false&useUnicode=true&characterEncoding=UTF-8&serverTimezone=GMT%2B8
    username=root
    password=123456

**log4j.properties:**

    # 将等级为DEBUG的日志信息输出到console和File这两个目的地，console和file的定义在下面的代码
    log4j.rootLogger=DEBUG,console,file

    # 控制台输出的相关配置
    log4j.appender.console=org.apache.log4j.ConsoleAppender
    log4j.appender.console.Target=System.out
    log4j.appender.console.Threshold=DEBUG
    log4j.appender.console.layout=org.apache.log4j.PatternLayout
    log4j.appender.console.layout.ConversionPattern=[%c]-%m%n

    # 文件输出的相关配置
    log4j.appender.file=org.apache.log4j.RollingFileAppender
    log4j.appender.file.File=./log/wue.log
    log4j.appender.file.MaxFileSize=10mb
    log4j.appender.file.Threshold=DEBUG
    log4j.appender.file.layout=org.apache.log4j.PatternLayout
    log4j.appender.file.layout.ConversionPattern=[%p][%d{yy-MM-dd}][%c]%m%n

    # 日志输出级别
    log4j.logger.org.mybatis=DEBUG
    log4j.logger.java.sql=DEBUG
    log4j.logger.java.sql.Statement=DEBUG
    log4j.logger.java.sql.ResultSet=DEBUG
    log4j.logger.java.sql.PreparedStatement=DEBUG


## utils

**MyBatisUtils:**

    import org.apache.ibatis.io.Resources;
    import org.apache.ibatis.session.SqlSession;
    import org.apache.ibatis.session.SqlSessionFactory;
    import org.apache.ibatis.session.SqlSessionFactoryBuilder;

    import java.io.IOException;
    import java.io.InputStream;

    /**
     * @author FiveMountain
     * @date 2021/4/26 12:39
     */
    public class MyBatisUtils {
        private static SqlSessionFactory sqlSessionFactory;
        static {
            try {
                String resource = "mybatis-config.xml";
                InputStream inputStream = Resources.getResourceAsStream(resource);
                sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
            } catch (IOException e) {
                e.printStackTrace();
            }
        }

        public static SqlSession getSqlSession() {
            return sqlSessionFactory.openSession();
        }
    }

## 资源文件过滤

**pom.xml:**

    <build>
        <resources>
            <resource>
                <directory>src/main/resources</directory>
                <includes>
                    <include>**/*.properties</include>
                    <include>**/*.xml</include>
                </includes>
                <filtering>true</filtering>
            </resource>
            <resource>
                <directory>src/main/java</directory>
                <includes>
                    <include>**/*.properties</include>
                    <include>**/*.xml</include>
                </includes>
                <filtering>true</filtering>
            </resource>
        </resources>
    </build>

