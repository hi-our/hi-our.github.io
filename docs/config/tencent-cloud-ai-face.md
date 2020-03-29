---
title: 项目搭建-Taro小程序开启云开发模式
---

## 更多人工智能的介绍，以及sdk的使用
配图


```js
// config.js
module.exports = {
  SecretId: '',
  SecretKey: ''
}
```

```js
// cloud/functions/analyze-face/index.js 腾讯云人脸识别效果
return new Promise((resolve, reject) => {
  // 通过client对象调用想要访问的接口，需要传入请求对象以及响应回调函数
  client.AnalyzeFace(faceReq, function (error, response) {
    // 请求异常返回，打印异常信息
    if (error) {
      const { code = '' } = error

      resolve({
        data: {},
        time: new Date(),
        status: -10086,
        message: status.FACE_CODE[code] || '图片解析失败'
      })
      return
    }
    console.log('AnalyzeFace response :', response)

    // 请求正常返回，打印response对象
    resolve({
      data: response,
      time: new Date(),
      status: 0,
      message: ''
    })
  })
});
```

### 腾讯云五官识别

其实，启发我做这个小程序的是这两个文章，《[「圣诞特辑」纯前端实现人脸识别自动佩戴圣诞帽](https://juejin.im/post/5e02b73fe51d455807699b1f "「圣诞特辑」纯前端实现人脸识别自动佩戴圣诞帽")》和《[我要戴口罩 – 为微信、微博等社交网络头像戴口罩](https://www.appinn.com/woyaodaikouzhao-wechat-miniapp/ "我要戴口罩 – 为微信、微博等社交网络头像戴口罩")》。

因为新冠病毒疫情蔓延，而戴口罩就是一个必备的预防措施啦。那怎样才能创新呢，我在使用“我要戴口罩”小程序过程中发现，口罩的位置是手动移动的，我就想如何自动戴过去呢，正好先前看到的“自动识别戴圣诞帽”，那我来一个戴口罩就好了。

在“自动佩戴圣诞帽”中，使用的方案是纯前端的 face-api，想放到小程序中就会有如下几个小问题：

- face-api 的识别模型有 5M 大小还多，即使纯前端加载，也显得比较大。而小程序的 canvas 与 web 网页中的还是有差异的，没法直接用 face-api。
- face-api 放在 nodejs 上加载，还需要配合`tensorflow`和`canvas`模拟。实际实现后发现，图片识别过程还是比较慢的（图片上传后、获取图片内容、识别五官位置、返回五官数据），容易让接口请求发生超时的情况。

在使用腾讯云的过程中，我就发现，腾讯云的人工智能大类目下居然有人脸识别功能，细致推究发现里面有“[五官分析](https://cloud.tencent.com/document/api/867/32779 "五官分析")”，其返回的数据跟`face-api`返回的数据格式还是非常像的，“人脸识别”的每月免费额度 10000 次，当时就让我开心了一大把。

当然，使用过程中非常大的坑就是，我的实现过程是需要上传 1M 以上大小的图片，而“五官分析”签名方法需要`TC3-HMAC-SHA256`，官方提供 npm 版本`tencentcloud-sdk-nodejs`是不支持这个签名方法的，需要从[官方 GitHub](https://github.com/TencentCloud/tencentcloud-sdk-nodejs/tree/signature3 "官方 GitHub")库的`signature3`分支上下载对应的代码。