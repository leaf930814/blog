---
title: 《白帽子讲WEB安全》读书笔记
subtitle: "互联网本来是安全的，因为有了研究安全的人所以才会不安全。"
cover: https://raw.githubusercontent.com/leaf930814/blog-misc/master/img/cyber_security.png
date: 2019-09-20 22:25:00
categories: Web开发
tags:
  - Web开发
 
author:
    nick: leaf++
    github_name: leaf930814
---
![image](https://raw.githubusercontent.com/leaf930814/blog-misc/master/img/cyber_security_2.png)

#### 第1章 我的安全世界观
安全评估的过程，可以简单分为 4 个阶段：资产等级划分、威胁分析、风险分析、确认解决方案。

Secure By Default 原则、纵深防御原则、数据与代码分离原则、不可预测性原则

#### 第2章 浏览器安全
同源策略是浏览器安全基础

浏览器沙箱：让不可信任代码运行在一定环境中，限制访问隔离区之外的资源

恶意网址拦截

#### 第3章 跨站脚本攻击（XSS）
##### Cross Site Script简介
- 概念：黑客通过“HTML注入”篡改网页，插入恶意脚本，从而在用户浏览网页时，控制用户浏览器。
- 根据效果分类：
  - 反射型XSS：简单地把用户输入的数据反射给浏览器。
  - 存储型XSS：会把用户输入的数据存储在服务器端，这种XSS具有稳定性。
##### XSS攻击进阶
- 构造技巧：
  - 利用字符编码
  - 绕过长度限制
  - 使用<Base>标签
  - Window.name:可以实现跨域，跨页面传递数据
- 防御
  - HttpOnly,解决cookie劫持
  - 浏览器filter
  - 输出检查：编码，转义
 
#### 第4章 跨站点请求伪造（CSRF）
- 通过伪装来自受信任用户的请求来利用受信任的网站
##### CSRF进阶
- 浏览器cookie：一种是Session Cookie，没有指定expire，浏览器关闭，cookie就失效，另一种是Third-party-Cookie，又称为本地cookie，只有expire时间到了后才会失效。
- 如果当前浏览器会拦截本地cookie，那就需要攻击者构造攻击环境，诱使用户在浏览器中先访问目标站点，使session cookie生效，再进行CSRF攻击
- 如果网站返回给浏览器的HTTP头中携带有P3P头，将允许浏览器发送第三方Cookie，即使在ie下也不会拦截。但是由于cookie是以域和path为单位的，p3p头的设置会影响整个域的所有页面。
- 不仅是get，CSRF也可以发起post请求
- 防御：
  - 验证码：最简洁有效的防御手段，但是用户体验差。
  - Referer Check：也可以被用于检查请求是否来自合法的源，但是服务器端并不是任何时候都可以获取到Referer，所以不能作为主要防御手段。
  - Anti Csrf token：使用token是业界一致的做法，token的生成需要具有随机性和保密性
 
#### 第5章 点击劫持（ClickJacking）
- 拖拽劫持 数据窃取 触屏劫持

- 防御：JavaScript禁止iframe嵌套 HTTP头X-Frame-Options

- iframe 可设置 sandbox 参数

#### 第6章 HTML5 安全
- HTML5新标签的XSS：可能带来新的XSS攻击。

#### 注入攻击
- 两个关键条件：第一个是用户能够控制输入，第二个是原本程序要执行的代码，拼接了用户输入的数据。
##### SQL注入
- 发现漏洞方式
  - 攻击者通过web服务器的错误回显构造sql注入语句
  - 没有错误回显一样可以实现注入，就是所谓的盲注，是通过简单的条件判断对比返回结果来发现漏洞。
- 防御
  - 建立数据库账号时要遵循“最小权限”原则，每个应用分配不同的账号，且账号不应该具有创建自定义函数和操作本地文件的权限
  - 使用预编译语句：这是防御sql注入的最佳方式，攻击者无法改变sql语句的结构
  - 使用存储过程：但是存储过程也存在注入风险，所以应避免在存储过程中使用动态语句
  - 检查数据类型：比如限制输入为Intger类型，比如限制输入为日期格式等
##### 其他注入攻击
- XML注入
- 代码注入
- CRLF注入
