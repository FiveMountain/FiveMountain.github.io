---
title: 豆瓣FM
date: 2021-03-14 19:36:39
categories:
- Projects
---
## 开始

### 功能模块

- 歌手
- 歌曲
- 歌单
- 播放器
- 爬虫
- 兆赫(MHz)
- 搜索
- 用户注册登录
- 红心
- 我的
- 分享

### 搭建本地开发环境

- JDK
- maven
- IntelliJ IDEA(根据个人喜好选择)

### 创建新工程

地址：[Spring Initializr](https://start.spring.io/)

表单各项为：

|  表单项标题    |         值                           |
|   :------:     |      :------:                        |
|  Group         |     `fm.douban`                      |
|  Artifact      |     `practice`                       |
|  Name          |     `practice`                       |
|  Description   |     `simulate fm.douban.com site`    |
|  Package name  |     `fm.douban`                      |

Packaging默认值Jar即可，Java根据自己电脑上的版本来选择。依赖可以选上Spring Web、Thymeleaf、MongoDB等常用的

### 配置工程

在新工程中配置好MongoDB

### 注意事项

- 保证启动类在`fm.douban`包中，不要放在其他包(如`fm.douban.app`)中
- 保留`src/test/java/fm/douban`目录，但清空目录下的所有文件和文件夹，保持此目录干净


## 歌手服务


