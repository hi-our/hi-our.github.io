---
title: 扩展能力说明
---

## 图像缩放及裁剪

图像缩放及裁剪，有两种方法进行，一种是小程序侧借助图片裁剪插件让用户自己手动裁剪，而另一种就是今天介绍的基于云开发扩展能力的[图像处理](https://cloud.tencent.com/document/product/876/42103#.E4.BD.BF.E7.94.A8.E6.89.A9.E5.B1.95)来自动完成图片裁剪。

如，原图为2160x2880的1Mb大小的图片，而在小程序显示时只需要宽高为600x600的图片即可（图片大小会降为70Kb）。

![](https://n1image.hjfile.cn/res7/2020/04/12/56ed9b6e34415129deefc3ba6463c865.jpg)

```js
// 获取图片临时链接
const getImageUrl = async (fileID) => {
  const { fileList } = await tcb.getTempFileURL({
    fileList: [fileID]
  })
  return fileList[0].tempFileURL
}
```

```js
let originImageUrl = await getImageUrl(fileID)

let rule = `imageMogr2/thumbnail/!${width}x${height}r|imageMogr2/scrop/${width}x${height}/`

// 人脸智能裁剪后的图片
return cutImageUrl = originImageUrl + '?' + rule
    
```

图片缩放及裁剪的核心就是`rule`，这里用了管道操作符（`|`），对独特平进行了2次处理

1. `imageMogr2/thumbnail/!${width}x${height}r`，将图片缩放为宽高中的小边为600px（`限定缩略图的宽度和高度的最小值分别为 Width 和 Height，进行等比缩放`）
2. `|imageMogr2/scrop/${width}x${height}/`，将图片的人脸部分裁剪出来（`基于图片中的人脸位置进行缩放裁剪。目标图片的宽度为 Width、高度为 Height`）


## 图像安全审核

> 图像安全审核提供鉴黄、鉴政、鉴暴恐等多种类型的敏感内容审核服务，有效识别违禁图片，规避违规风险。

我们这里对比了云开发扩展能力的`图片安全审核`与开放能力的`图片安全审核`的差异。


| 名称                  | 额度    |    数据类型         |
| :-------------------- | :-------: | :-------: |
| 图片安全审核<br />云开发扩展能力 | 2000张/日 |      云存储fileId中图片绝对地址       |
| imgSecCheck<br />开放能力 | 免费 |      Buffer       |

**图片安全审核，来自云开发扩展能力**

接口文档：https://cloud.tencent.com/document/product/876/42096#.E4.BD.BF.E7.94.A8.E6.89.A9.E5.B1.95

**使用步骤：**
1. 上传图片到云存储
2. 云函数调用`图片安全审核，来自云开发扩展能力`进行校验
3. 如果是违规图片，可能还需要从云存储上删除，以免占用不必要的空间

**开放能力 imgSecCheck**

接口文档 https://developers.weixin.qq.com/miniprogram/dev/api-backend/open-api/sec-check/security.imgSecCheck.html

在实际使用中发现的大问题，图片Buffer大小超过100k，可能就报图片太大的错误了。有的小伙伴说，10k都可能会报错。

### 使用方法
```js
async function demo() {
  try {
    const opts = {
      type: ["porn", "terrorist", "politics"]
    }
    const res = await tcb.invokeExtension('CloudInfinite',{
      action:'DetectType',
      cloudPath: "ab.png", // 需要分析的图像的绝对路径，与tcb.uploadFile中一致
      operations: opts
    })
    console.log(JSON.stringify(res.data, null, 4));
  } catch (err) {
    console.log(JSON.stringify(err, null, 4));
  }
}
```

完整的使用示例，请参照 https://github.com/hi-our/hi-face/blob/master/cloud/functions/image-safe-check/index.js
