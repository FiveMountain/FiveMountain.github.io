---
title: Apollo
date: 2021-07-09 13:36:49
categories:
- Learning Notes
---
# 简介

Apollo（阿波罗）是携程框架部门研发的**分布式配置中心**，能够**集中化管理**应用不同环境、不同集群的配置，配置修改后能够**实时推送**到应用端，并且具备规范的权限、流程治理等特性，适用于微服务配置管理场景。

服务端基于Spring Boot和Spring Cloud开发，打包后可以直接运行，不需要额外安装Tomcat等应用容器。

Java客户端不依赖任何框架，能够运行于所有Java运行时环境，同时对Spring/Spring Boot环境也有较好的支持。

.Net客户端不依赖任何框架，能够运行于所有.Net运行时环境。

Apollo基础模型：

1. 用户在配置中心对配置进行修改并发布
2. 配置中心通知Apollo客户端有配置更新
3. Apollo客户端从配置中心拉取最新的配置、更新本地配置并通知到应用

## 各模块概要

### Config Service

- 提供配置获取接口
- 提供配置更新推送接口（基于HTTP long polling）
    - 服务端使用Spring DeferredResult实现异步化，从而打打增加长连接数量
    - 目前使用的tomcat embed默认配置是最多10000个连接（可以调整），使用了4C8G的虚拟机实测可以支撑10000个连接，所以满足需求（一个应用实例只会发起一个长连接）。
- 接口服务对象为Apollo客户端

### Admin Service

- 提供配置管理接口
- 提供配置修改、发布等接口
- 接口服务对象为Portal

### Meta Server

- Portal通过域名访问Meta Server获取Admin Service服务列表（IP+Port）
- Client通过域名访问Meta Server获取Config Service服务列表（IP+Port）
- Meta Server从Eureka获取Config Service和Admin Service的服务信息，相当于是一个Eureka Client
- 增设一个Meta Server的角色主要是为了封装服务发现的细节，对Portal和Client而言，永远通过一个Http接口获取Admin Service和Config Service的服务信息，而不需要关心背后实际的服务注册和发现组件
- Meta Server只是一个逻辑角色，在部署时和Config Service是在一个JVM进程中的，所以IP、端口和Config Service一致

### Eureka

- 基于Eureka和Spring Cloud Netflix提供服务注册和发现
- Config Service和Admin Service会向Eureka注册服务，并保持心跳
- 为了简单起见，目前Eureka在部署时和Config Service是在一个JVM进程中的（通过Spring Cloud Netflix）

### Portal

- 提供Web界面供用户管理配置
- 通过Meta Server获取Admin Service服务列表（IP+Port），通过IP+Port访问服务
- 在Client侧做load balance、错误重试

## 核心概念

Apollo支持4个维度管理Key-Value格式的配置：

1. application（应用）
2. environment（环境）
3. cluster    （集群）
4. namespace  （命名空间）

### application（应用）

- 实际使用配置的应用，Apollo客户端在运行时需要知道当前应用是谁，从而获取对应配置
- 每个应用都需要有唯一的身份标识--appid，应用身份是跟着代码走的，所以需要在代码中配置

### environment（环境）

- 配置对应的环境，Apollo客户端在运行时需要知道当前应用处于哪个环境，从而获取应用的配置
- 环境和代码无关，同一份代码部署在不同的环境就应该能够获取到不同环境的配置
- 环境默认是通过读取机器上的配置（server.properties中的env属性）指定的，不过为了开发方便，也支持运行时通过System Property指定

### cluster（集群）

- 一个应用下不同实例的分组，比如典型的可以按照数据中心分
- 对不同的cluster，同一个配置可以有不一样的值，如zookeeper地址
- 集群默认是通过读取机器上的配置（server.properties中的idc属性）指定的，不过也支持运行时通过System Property指定

### namespace（命名空间）

- 一个应用下不同配置的分组，可以简单地把namespace类比为文件，不同类型的配置存放在不同的文件中
- 应用可以直接读取到公共组件的配置namespace，如DAL、RPC等
- 应用也可以通过集成公共组件的配置namespace来对公共组件的配置做调整

# Quick Start

## 准备工作

- Java
    - Apollo服务端：1.8+
    - Apollo客户端：1.7+
    
- MySQL
    - 版本要求：5.6.5+
    
    Apollo的表结构对`timestamp`使用了多个default声明，所以需要5.6.5以上版本。
    
- 下载Quick Start安装包

    checkout或下载[apollo-build-scripts项目](https://github.com/apolloconfig/apollo-build-scripts)

    - 手动打包Quick Start安装包
        1. 修改apollo-configservice, apollo-adminservice和apollo-portal的pom.xml，注释掉spring-boot-maven-plugin和maven-assembly-plugin
        2. 在根目录下执行`mvn clean package -pl apollo-assembly -am -DskipTests=true`
        3. 复制apollo-assembly/target下的jar包，rename为apollo-all-in-one.jar

## 安装步骤

### 创建数据库

Apollo服务端共需要两个数据库：`ApolloPortalDB`和`ApolloConfigDB`

> 如果本地已经创建过Apollo数据库，需注意备份数据

**创建ApolloPortalDB**

导入[sql/apolloportaldb.sql](https://github.com/apolloconfig/apollo-build-scripts/blob/master/sql/apolloportaldb.sql)

**创建ApolloConfigDB**

导入[sql/apolloconfigdb.sql](https://github.com/apolloconfig/apollo-build-scripts/blob/master/sql/apolloconfigdb.sql)

### 配置数据库连接信息

Apolo服务端需要知道如何连接到前面创建的数据库，所以需要编辑demo.sh，修改ApolloPortalDB和ApolloConfigDB相关的数据库连接串信息。

> 填入的用户需要具备对ApolloPortalDB和ApolloConfigDB数据的读写权限

```sh
#apollo config db info
apollo_config_db_url="jdbc:mysql://localhost:3306/ApolloConfigDB?characterEncoding=utf8&serverTimezone=Asia/Shanghai"
apollo_config_db_username=用户名
apollo_config_db_password=密码（如果没有密码，留空即可）

# apollo portal db info
apollo_portal_db_url="jdbc:mysql://localhost:3306/ApolloPortalDB?characterEncoding=utf8&serverTimezone=Asia/Shanghai"
apollo_portal_db_username=用户名
apollo_portal_db_password=密码（如果没有密码，留空即可）
```

## 启动Apollo配置中心

### 确保端口未被占用

Quick Start脚本会在本地启动三个服务，分别使用8070、8080、8090

### 执行启动脚本

```sh
./demo.sh start
```

当看到如下输出后，就说明启动成功了

```
==== starting service ====
Service logging file is ./service/apollo-service.log
Started [10768]
Waiting for config service startup.......
Config service started. You may visit http://localhost:8080 for service status now!
Waiting for admin service startup....
Admin service started
==== starting portal ====
Portal logging file is ./portal/apollo-portal.log
Started [10846]
Waiting for portal startup......
Portal started. You can visit http://localhost:8070 now!
```

### 异常排查

如果启动遇到了异常，可以分别查看service和portal目录下的log文件排查问题

>在启动apollo-configservice的过程中会在日志中输出eureka注册失败的信息，如`com.sun.jersey.api.client.ClientHandlerException: java.net.ConnectException: Connection refused`。需要注意的是，这个是预期的情况，因为apollo-configservice需要向Meta Server(它自己)注册服务，但是因为在启动过程中，自己还没起来，所以会报这个错。后面会进行重试的动作，所以等自己服务起来后就会注册正常了。

### 注意

Quick Start只是用来帮助快速体验Apollo项目，不支持增加环境。
