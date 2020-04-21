---
title: 多种云开发扩展能力，让人脸识别飞起来
---

最近，腾讯云云开发增加了几项非常实用的[扩展能力](https://console.cloud.tencent.com/tcb/add)，包含图像标签、图像安全审核、图像处理、图片盲水印。

本次，我就将借助云开发扩展能力来完成人脸识别的小程序。

![](https://n1image.hjfile.cn/res7/2020/04/12/9cd41ecbf9d287661512af1060bf2039.jpg)



## 人脸识别

借助图像处理来优化人脸识别流程。

<!-- 缺少一个流程图 -->
* 选择图片
* 上传到云存储
* 云开发
  * 图片：缩放、人脸智能裁剪
  * 图像安全审核（后文中介绍）
  * 腾讯云-人脸识别
* 返回到小程序上进行显示
  * 显示裁剪后的图片
  * 显示人脸魅力框

云函数超时时间调整为15秒

### 图像缩放及裁剪

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

let cutImageUrl = originImageUrl + '?' + rule

return cutImageUrl
    
```

图片缩放及裁剪的核心就是`rule`，这里我执行了两项操作

1. `/thumbnail/!600x600r/`，将图片缩放为宽高中的小边为600px（`限定缩略图的宽度和高度的最小值分别为 Width 和 Height，进行等比缩放`）
2. `/scrop/600x600`，将图片的人脸部分裁剪出来（`基于图片中的人脸位置进行缩放裁剪。目标图片的宽度为 Width、高度为 Height`）

> 如果人脸智能裁剪不奏效的话，也可以用`/crop/600x600/center`，将图片居中裁剪。

### 人脸识别
腾讯云的[人脸识别](https://cloud.tencent.com/document/api/867/32800)服务支持使用图片链接（`Url`）或者图片base64数据（`Image`）来完成人脸识别。

这里，我通过 fileID 获取图片临时链接，以提供给人脸识别使用。

![](https://n1other.hjfile.cn/res7/2020/04/12/ae020750c69cc3c4938542ea98d64e48.PNG)


```js
// 人脸识别示例代码
const detectFace = (Image) => {

  let faceReq = new models.DetectFaceRequest()

  // 支持图片链接或图片base64数据
  let query_string = JSON.stringify(Image.includes('http') ? {
    Url: Image,
    MaxFaceNum: 5,
    NeedFaceAttributes: 1
  } : {
      Image,
      MaxFaceNum: 5,
      NeedFaceAttributes: 1
    })
  
  // 传入json参数
  faceReq.from_json_string(query_string);

  // 进行人脸识别
  return new Promise((resolve, reject) => {
    // 通过client对象调用想要访问的接口，需要传入请求对象以及响应回调函数
    client.DetectFace(faceReq, function (error, response) {
      // 请求异常返回，打印异常信息
      if (error) {
        const { code = '' } = error
        console.log('code :', code);

        resolve({
          data: {},
          time: new Date(),
          status: -10086,
          message: 'DetectFace ' + status.DETECT_CODE[code] || code || '图片解析失败'
        })
        return
      }
      // 请求正常返回，打印response对象
      resolve({
        data: response,
        time: new Date(),
        status: 0,
        message: ''
      })
    })
  });
}
```

**注意点**

> 接口文档：最多返回面积最大的 5 张人脸属性信息，超过 5 张人脸（第 6 张及以后的人脸）的 FaceAttributesInfo 不具备参考意义。

* `MaxFaceNum`：最大识别人数建议限定为5个
* `NeedFaceAttributes`：设置为1时，才会返回人脸属性值（魅力值）




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

## 多图裁剪
> 需要补充多图裁剪的功能

## 图片主色调
在`图像处理`中其实有一项十分好用的功能，那就是获取图片主色调，这个能力是基于云存储的数据万象来做的。

![](https://n1other.hjfile.cn/res7/2020/04/12/969c1959a0b48b23f202b70235dda365.JPG)

其实用方法为将云存储的图片链接后拼接`?imageAve`。而我这里还打算将颜色转换放在里面，因为可能需要的颜色是带透明度的RGBA颜色。

```js
exports.main = async (event) => {
  const { fileID = '', opacity = 1, colorType = 'default' } = event

  console.log('fileID :', fileID);

  if (fileID) {
    try {
      const imgUrl = await getImageUrl(fileID)

      // 云存储的图片临时链接拼接`?imageAve`
      const res = await fetch.get(imgUrl + '?imageAve')
      const { RGB } = getResCode(res)
  
      const colorHex = '#' + RGB.substring(2)
      const colorRgbaObj = hexToRgba(colorHex, opacity)
      const colorRgba = colorRgbaObj.rgba
      const colorRgb = `rgb(${colorRgbaObj.red}, ${colorRgbaObj.green}, ${colorRgbaObj.blue})`
  
      // 支持多种颜色获取方式
      let mainColor = opacity === 1 ? colorHex : colorRgba
      if (colorType === 'hex') {
        mainColor = colorHex
      } else if (colorType === 'rgb') {
        mainColor = colorRgb
      }

      return {
        data: {
          mainColor
        },
        time: new Date(),
        status: 0,
        message: ''
      }
      
    } catch (error) {
      return {
        data: {},
        time: new Date(),
        status: -10087,
        message: JSON.stringify(error)
      }
    }

  }

  let errorString = '请设置 fileID'
  console.log(errorString)
  return {
    data: {},
    time: new Date(),
    status: -10086,
    message: errorString
  }
}
```

完整的使用示例，请参照 https://github.com/hi-our/hi-face/blob/master/cloud/functions/get-main-color/index.js

## 图像标签

> 图像标签对云存储中存量数据的图片标签识别，返回图片中置信度较高的主题标签，帮助开发者分析图像。

![](https://n1image.hjfile.cn/res7/2020/04/12/9cd41ecbf9d287661512af1060bf2039.jpg)

> 上图中，人体、婚纱等文字就是图像标签。

```js
let imgID = fileID.replace('cloud://', '')
let index = let .indexOf('/')
let cloudPath = imgID.substr(index)

const res = await tcb.invokeExtension("CloudInfinite", {
  action: "DetectLabel",
  cloudPath: cloudPath // 需要分析的图像的绝对路径，与tcb.uploadFile中一致
})

const { Labels } = getResCode(res)
// 兼容只有标签时为对象的情况
const tmpLabels = Labels.length > 1 ? Labels : [Labels]

const list = tmpLabels.map(item => ({
  confidence: item.Confidence,
  name: item.Name
}))
```

完整的使用示例，请参照 https://github.com/hi-our/hi-face/blob/master/cloud/functions/get-main-color/index.js



## 文章相关内容：


我是 **盛瀚钦**，沪江 CCtalk 前端开发工程师，Taro 框架的 issue 维护志愿者，腾讯云云开发布道师，主要侧重于前端 UI 编写和团队文档建设

源码地址：
[https://github.com/hi-our/hi-face](https://github.com/hi-our/hi-face "https://github.com/hi-our/hi-face")


## 文章标题

### `多种云开发扩展能力，让人脸识别飞起来-图像处理篇`

人脸识别、多图裁剪

### `多种云开发扩展能力，让人脸识别飞起来-图像安全篇`

对比三种图像安全审核功能，突出`图像安全审核`扩展能力好


### `多种云开发扩展能力，让人脸识别飞起来-小创意篇`
图像主色调、人脸图片标签


