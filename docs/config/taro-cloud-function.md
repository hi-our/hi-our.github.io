---
title: 项目搭建-Taro小程序开启云开发模式
---

## 本文阅读步骤

* 了解Taro、小程序云开发、腾讯云环境配置等基本概念
* 小程序配置`project.config.json`从普通模式切换成云开发模式
* 小程序启动时，设置云开发所需环境
* 在云函数中，可以灵活地调用腾讯云的服务，在下一篇文章中作具体介绍


## 概念解析
### 小程序云开发
[官方文档](https://developers.weixin.qq.com/miniprogram/dev/wxcloud/basis/getting-started.html)

开发者可以使用云开发开发微信小程序、小游戏，无需搭建服务器，即可使用云端能力。
云开发为开发者提供完整的原生云端支持和微信服务支持，弱化后端和运维概念，无需搭建服务器，使用平台提供的 API 进行核心业务开发，即可实现快速上线和迭代，同时这一能力，同开发者已经使用的云服务相互兼容，并不互斥。 

### Taro介绍
[官方文档](https://taro-docs.jd.com/taro/docs/README.html)

Taro 是一套遵循 `React` 语法规范的 **多端开发** 解决方案。使用 Taro，我们可以只书写一套代码，再通过 Taro 的编译工具，将源代码分别编译出可以在不同端（微信/百度/支付宝/字节跳动/QQ/京东小程序、快应用、H5、React-Native 等）运行的代码。

	
### 腾讯云人脸识别简介
[官方文档](https://cloud.tencent.com/product/facerecognition)

腾讯云神图·人脸识别（Face Recognition）基于腾讯优图强大的面部分析技术，提供包括人脸检测与分析、五官定位、人脸搜索、人脸比对、人脸验证、人员查重、活体检测等多种功能，为开发者和企业提供高性能高可用的人脸识别服务。 可应用于智慧零售、智慧社区、在线娱乐、智慧楼宇、在线身份认证等多种应用场景，充分满足各行业客户的人脸属性识别及用户身份确认等需求。 


## 小程序项目配置 project.config.json

### 普通模式

* `miniprogramRoot` 是小程序代码目录，此处写的是Taro编译后的dist目录
* `appid` 微信小程序的appid


```json
{
  "miniprogramRoot": "taro/dist/",
  "appid": "testappid"
}
```

### 云开发模式

**开通云环境**

* 在微信开发者工具控制台上，点击“云开发”按钮，就可以新家云开发环境。最多支持两个
* 建好之后，就可以看到你的云环境的名称，如下图

![](https://n1image.hjfile.cn/res7/2020/03/29/97f0a08f4779c07add38f10fb7c4f526.png)

**云环境配置说明**
* `cloudfunctionRoot` 云开发的云函数所在的目录
* `cloudfunctionTemplateRoot`云开发的云函数测试数据模板所在的目录
* `qqappid` QQ小程序的appid。即腾讯云云开发目标支持跨端使用

```json
{
  "miniprogramRoot": "taro/dist/",
  "cloudfunctionRoot": "cloud/functions/",
  "cloudfunctionTemplateRoot": "cloud/template/",
  "appid": "testappid",
  "qqappid": "testqqappid"
}
```

这里我用的是Taro跨端框架，可以使用其`taro-cli`中的云开发模板来初始化一个新的小程序项目，观察里面的文件目录设置来以作为参考。



## 云开发模式配置及使用

1. 小程序启动时，初始化云环境
2. 在需要调用云函数或云存储的地方，调用对应的`callFunction`等方法即可

```js
// taro/src/pages/app.js
Taro.cloud.init({
  env: 'name-id' // 云开发环境名
})

// taro/src/pages/wear-a-mask/wear-a-mask.js
Taro.cloud.callFunction({
  name: 'analyze-face',
  data: {
    fileID: '12345'
  }
}).then(res => console.log(res))
```


## 常见问题

**接口请求体太大，还未接收到请求结果，云函数就返回失败？**

将接口请求的返回数据调大一些，如20秒

```json
"networkTimeout": {
  "request": 20000,
  "downloadFile": 10000
}
```

**微信开发者工具上使用云函数本地调试的时候，函数还未发出就报错，并且返回的`result`为`null`**

我在做“人脸五官分析”功能的时候，曾经调用了约 1Mb 大小的 base64 图片数据，就遇到了这个问题。后来努力将 base64 图片数据降为 150Kb（在图片识别加速的文章中有讲解），或者将这个图片先上传到云存储上，再在云函数环境中再下来下来，这样也达到多次调用，减少网络间数据传递体积。
