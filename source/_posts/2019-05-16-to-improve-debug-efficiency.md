title: Sentry - 及时发现Bug，提高Debug效率
subtitle: "Sentry是一款异常监控开源工具，名字翻译过来就是“哨兵”，在Github上面的简介是：“跨平台应用监控，关注错误报告”。"
cover: https://blockfe.github.io/misc/xlogic/post_to_improve_debug_efficiency.png
date: 2019-05-16 12:43:35
categories: 基础设施
tags:
  - Sentry
  - 前端监控
  - Bug追踪
author:
    nick: xLogic
    github_name: xLogic92
---

## Sentry简介

Sentry是一款异常监控开源工具，名字翻译过来就是“哨兵”，在Github上面的简介是：“跨平台应用监控，关注错误报告”。支持各种语言，例如 python、oc、java、node、javascript 等。也可以应用到各种不同的框架上面，如前端框架中的vue 、angular 、react 等最流行的前端框架。Sentry分为社区开源版和在线Saas版，这里我们已经在自己服务器部署了一套服务。

官网： [https://sentry.io](https://sentry.io/)

文档： https://docs.sentry.io/clients/javascript/install/

Github仓库： https://github.com/getsentry/sentry

社区除了github issue外还可以关注 https://forum.sentry.io/

## 使用指南

> PS: 进入Dashbord点击左上角头像，选择User settinng，然后在PREFEREBCES中，Language切换成简体中文，且Timezone调整至本地时区，不然后面看到监控的bug创建时间会有差别。

### 创建项目

进入我们可以通过右上角 **Add new… > 项目** 来创建，然后选择相应的项目，这里以vue为例子，如下图:

![sentry_01](<https://raw.githubusercontent.com/blockfe/misc/gh-pages/xlogic/sentry_01.png>)

接下来会进入到*介绍页面*了，到这里第一步就算完成，**请详细阅读该页面**。

### 前端项目部署

切回本地项目,通过npm安装Sentry’s browser JavaScript SDK

```bash
npm install @sentry/browser
npm install @sentry/integrations
```

```javascript
import Vue from 'vue'
import * as Sentry from '@sentry/browser';
import * as Integrations from '@sentry/integrations';

Sentry.init({
  dsn: 'https://xxxxxx/DSN', // DSN密钥 - 应对后台项目
  environment: 'prod',  // 项目环境 - 区分测试环境的和生产环境的异常
  release: '1.0.0', // Release版本控制 - 下文将会介绍
  integrations: [
    new Integrations.Vue({
      Vue,
      attachProps: true,
    }),
  ],
});
```

记得把DSN换成自己的，在**介绍页面**中可以找到，如果已经离开该页面，可以在**项目 > 设置 > 客户端密钥(DSN)**中找到它。

### 自动捕捉异常+查看

ok，部署操作已经完成，接下来我们主动上报一个bug试试水。

在App.vue的mounted中写一个bug：

```javascript
console.log(window.aaa.bbb());
```

然后刷新页面触发bug，这时可以通过chrome调试工具查看上报异常的网络请求。

![sentry_02](<https://raw.githubusercontent.com/blockfe/misc/gh-pages/xlogic/sentry_02.png>)

回到Sentry中，不出意外此时就可以看到相应的错误信息提示。

![sentry_03](<https://raw.githubusercontent.com/blockfe/misc/gh-pages/xlogic/sentry_03.png>)

点进去后就能看到更多的错误信息还有用户信息，包括浏览器、版本、ip等

### 主动捕捉异常

通过上面的操作我们已经能成功监控到vue中的错误、异常，但是还不能捕捉到异步操作、接口请求中的错误，比如接口返回404、500等信息，此时我们可以通过Sentry.caputureException()进行主动上报。

##### 接口异常

由于项目中常采用的axios进行接口请求，axios提供了请求响应的拦截器 axios.interceptors.response，示例：

```javascript
axios.interceptors.response.use(data => {
    return data;
}, error => {
    Sentry.captureException(error);
})
```

##### 异步操作

在异步操作中的异常也不能被自动捕捉，我们需要手动处理：

```javascript
setTimeout(()=>{
  try {
    // do some
  } catch (err) {
    Sentry.captureException(err);
  }
}, 1000)`
```

另外，请在主动抛出的异常时使用new error进行创建，这样能更好的定位异常所在位置。

```javascript
// good 
throw new error()

// bad
throw "error"
```

### context 上下文信息

上下文信息包括：user、tags、level、fingerprint、extra data，这些信息我们可以通过在 scope 上面设置来定义。
其中可以通过两种方法得到 scope ：

```javascript
// 将 scope 配置到 context 上面
Sentry.configureScope((scope) => { });
// 创建一个零时到 scope ，配置到 context 上面
Sentry.withScope((scope) => { });
```

##### User

```javascript
scope.setUser({
  id:'1',
  username:'xLogic',
  ip_address:'127.0.0.1',
  email: 'xlogic@example.com'
});
```

其中user可以设置的信息包括id、username、ip_address、email。

##### Tags

tags是给事件定义不同的键/值对，可以在查找的时候更容易。
后台查找的时候，查找选项会多出来一个选项，就是通过tags来设置的。
	
```javascript
scope.setTag("page_local", "de-at");
```

通过setTag来设置了一个page_local的标签, 后台会多一个page_local选项, 包括de-at。

##### Extra Data

传入额外的信息, 并不会创建索引(也就是不可以提供来检索)。

```javascript
scope.setExtra("character_name", "Mighty Fighter");
```

## 拓展功能

### 准备工作

需要安装Sentry对应的命令行管理工具 sentry-cli，方式如下：

```bash
npm i -g @sentry/cli
```

##### 生成token

点击Sentry页面左下角头像，选择API后就可以生成token，记得勾选 project:read, project:releases, project:write, project:admin 权限。

##### 登录

```bash
$ sentry-cli --url https://myserver/ login
```

回车后输入上一步获得的 token 即可

### Release控制

创建Release来进行“异常”的版本控制，此外下文提到的SoureceMap需通过Realease来标注版本，以匹配相应版本的源码，以便定位压缩前实际的错误信息。

##### 创建Release

```
sentry-cli releases -o 组织 -p 项目 new 1.0.0
```

-o : 组织，可以在我们的 **组织设置** 中找到 (sentry)

-p : 项目名称 ， 可以在 **项目** 中找到 (blokcfe)

这里的 1.0.0 就是我们指定的版本号，现在我们可以通过创建多个版本号来进行异常分类。
同时，也可以通过页面中"Releases"查看是否创建成功。

##### 本地应用Release

回到前端项目中，在Sentry.init添加对应的release，指定版本后，每次上报的异常就会分类到该版本下。

##### 删除Release

```bash
sentry-cli releases -o 组织 -p 项目 delete 1.0.0
```

*注意:* 删除某个release时需要将其下的异常处理掉,并将该版本的SourceMap文件清空,完成上面两步可能还要等待2小时才能删除，不然会报错。

### SourceMap管理

目前来说，前端项目基本都会压缩混淆代码，这样导致Sentry捕捉到的异常堆栈无法理解。

我们希望在Sentry直接看到异常代码的源码时就需要上传对应的source和map。

##### 上传 SourceMap

*手动上传*

```bash
sentry-cli releases -o 组织 -p 项目 files 1.0.0 upload-sourcemaps jspath文件所在目录 --url-prefix 线上资源URI
```

-o , -p : 和上文一样

jspath : js 文件的位置

uri : js 文件相对于域名的位置

*Webpack Plugin上传*

```bash
npm i -D @sentry/webpack-plugin
```

```javascript
const SentryCliPlugin = require('@sentry/webpack-plugin');
webpackConfig.plugins.push(
  new SentryCliPlugin({
    release: 1.0.0,
    include: path.join(__dirname, '../dist/static/js/'), // js 文件的位置
    urlPrefix: '~/static/js' // js 文件相对于域名的位置
  })
);
```

> PS: 需配置.sentryclirc，下文将详细介绍

##### 清空 SourceMap 文件

```bash
sentry-cli releases files 1.0.0 delete --all
```

也可以选择在 版本>工件 里点击一个个辣鸡桶进行删除

##### 重要的url-prefix

这里的url-prefix可以通过线上看js文件的完整路径，有可能static不在根目录下

举例说明，项目线上资源URI如下：

```
https://www.baidu.com/static/js/test.js
```

我们上传时文件的url-prefix就应该设置为 '~/static/js/'

```
https://www.baidu.com/assets/aaa/js/test.js
```

我们上传时文件的url-prefix就应该设置为 '~/assets/aaa/js/'

*SourceMap上传完毕，就能在Sentry上直接看到出错源码位置了：*

![sentry_04](<https://raw.githubusercontent.com/blockfe/misc/gh-pages/xlogic/sentry_04.png>)

### 报警邮件发送规则

Sentry默认会将所有采集到的异常发送警报邮件，有时我们可能希望只收到特定规则下的警报邮件，这时候就需要删除默认的警报规则，然后新建自定义规则。

一个比较常规的规则引擎，自己配置一下就可以搞定，还是比较简单。

### 修改sentry-cli默认设置

在上面的操作中，大家应该发现每次命令都需要重复输入一长串 -o xxx -p xxxx 来指定我们的项目，一点不DRY。

只需要找到当前用户文件夹下的 .sentryclirc 文件添加默认组织和项目即可，修改内容为如下：

```bash
[auth]
token=YOUR API TOKEN

[defaults]
url=服务器
org=组织
project=项目
```

## 结语

以上是我自己目前测试过功能，基本涵盖了常见场景。

当然，小伙们遇到问题或者新发现也可以跟我交流，一起挖掘更多使用场景。