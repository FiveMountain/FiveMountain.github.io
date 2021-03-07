---
title: cookie的重要属性
date: 2021-03-07 21:26:14
tags:
- Pages
---
### name

名称

### value

值

### domain

表示cookie生效的域
为null表示此cookie只对当前域名有效
如果设置了域名，表示当前域名和子域名都有效

### path

表示cookie生效的目录
为null表示此cookie只对当前请求的URL路径有效
如果设置了域名，表示当前URL路径和所有下级路径都有效
`/`表示整个网站都生效

### maxAge

有效时间，默认值为-1
负数表示关闭浏览器就删除cookie
0表示浏览器立即删除此cookie(cookie机制没有提供删除cookie的方法，因此通过设置该cookie即时失效实现删除cookie的效果。失效的cookie会被浏览器从cookie文件或内存中删除)
整数表示多少**秒**后过期自动失效

### httpOnly

安全性设置
值为true表示通过js脚本将无法读取到cookie信息
false表示不限制

