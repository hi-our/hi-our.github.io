---
title: 小程序与Web端的深色、浅色模式适配技巧
---

最近，深色模式与浅色模式相当火，微信 App 增加了深色模式后，各大 App 都在跟进中。而我早在 19 年 8 月底就开始做深色模式的页面了。当时是 CCtalk 的视频播放页面。

![图片](https://n1image.hjfile.cn/res7/2020/04/04/784c03e42122586c3552b259368c1aa3.png)

当时首要问题就是想，深色、浅色模式在前端中的表现是什么呢？

## 色值与情感色彩

在《[紧跟潮流学设计：深色模式设计的 8 个小技巧](https://36kr.com/p/5231320)》中详细阐述了颜色设置上的技巧。

比如在 CCtalk 这边的浅色模式与深色模式的文字颜色

|   级别   | 浅色模式 |         深色模式         |
| :------: | :------: | :----------------------: |
| 重要文字 | #2f3742  |  rgba(255, 255, 255, 1)  |
| 普通文字 | #687583  | rgba(255, 255, 255, 0.8) |
| 辅助文字 | #95a1af  | rgba(255, 255, 255, 0.6) |

**为何什么深色模式下使用透明颜色?**
因为深色模式下的背景色不一定为特定的颜色，而文字使用透明颜色，背景色就会透过文字透明来渗出来，效果会更好。

**为何不直接使用`opacity`，这样更加直接**
因为文字 RGBA 颜色，不会对元素造成影响。而`opacity`是改变整个元素的透明度

```css
.button {
  color: rgba(255, 255, 255, 0.8);
  background: #325ef6;
}

.button {
  color: #fff;
  opacity: 0.8;
  background: #325ef6;
}
```

## CSS 设置方法

深色、浅色模式有两种方向来考虑：

- **主题色**，即定义`theme-dark`与`theme-light`两套主题
- **全局统一色彩定义**，就是一种颜色修改后，全局全部变化，比如蓝色主题下链接颜色为`#325ef6`，而粉红色主题下链接为`#ff3271`，最好能统一定义。

常规的方案是用 [BEM 命名规范](https://www.cnblogs.com/jianxian/p/11084305.html) + `wrapper class`的方式。

```css
.theme-blue .button {
  color: #325ef6;
}

.theme-pink .button {
  color: #ff3271;
}
```

那如果一个页面有几十个 button，那这个颜色是不是要写几十遍呢？当然不是。

这里有这几种方案：

- 使用 CSS 预处理器
- 使用 CSS 变量
- 使用媒体查询 `@media(prefers-color-scheme: dark)`

### CSS 预处理器

使用 `Less`、`Scss`、`Stylus` 等 CSS 预处理器，可以快速实现定制主题色的方案。这里以 `Stylus` 为例。

> Stylus：富于表现力、健壮、功能丰富的 CSS 预处理器

```stylus
// 第一种写法
.theme-blue
  $button-color=#325ef6

  .button
    color $button-color

.theme-pink
  $button-color=#ff3271

  .button
    color $button-color
```

```stylus
// 第二种写法
// const-blue.styl
$button-color=#325ef6

// theme-blue.styl
@import "../../const-blue.styl"
.button
  color $button-color

// const-pink.styl
$button-color=#ff3271

// theme-pink.styl
@import "../../const-pink.styl"
.button
  color $button-color
```

也就是说，CSS 预处器在预先定义好颜色变量后，后续可以随意引用。当然，这里更加可以定义更多的功能，比如公共函数等。

使用了 CSS 预处理器的典型框架就是`BootStrap`，定制网址为：https://v3.bootcss.com/customize/

`Ant Design`已支持暗黑主题的定制，其配置方案与 `Bootstrap` 类似。https://ant.design/docs/react/customize-theme-cn

> **Ant Design of React** 是基于 Ant Design 设计体系的 React UI 组件库，主要用于研发企业级中后台产品。

### CSS 变量

那么，在不借助 CSS 预处理器的情况下，有没有原生的方法呢？答案就是 CSS 变量。

> 自定义属性（有时候也被称作 CSS 变量或者级联变量）是由 CSS 作者定义，它包含的值可以在整个文档中重复使用。由自定义属性标记设定值（比如： `**--main-color: black;**`），由 var() 函数来获取值（比如： `color: var(--main-color)`;）
> 复杂的网站都会有大量的 CSS 代码，通常也会有许多重复的值。举个例子，同样一个颜色值可能在成千上百个地方被使用到，如果这个值发生了变化，需要全局搜索并且一个一个替换（很麻烦哎～）。自定义属性在某个地方存储一个值，然后在其他许多地方引用它。另一个好处是语义化的标识。比如，`--main-text-color` 会比 `#00ff00` 更易理解，尤其是这个颜色值在其他上下文中也被使用到。  
> 自定义属性受级联的约束，并从其父级继承其值。

声明一个自定义属性：

```css
.element {
  --main-bg-color: brown;
}
```

使用一个局部变量：

```css
.element {
  background-color: var(--main-bg-color);
}
```

使用 CSS 变量的好处是动态改变。当元素被被优先级更高的 CSS 选择器定义时，就会被这上面的 CSS 变量所影响。

```css
.parent .element {
  background-color: var(--main-bg-color);
}
```

那么，CSS 兼容性如何呢？可以看张鑫旭早在 2016 年写的文章《[小 tips:了解 CSS 变量 var](https://www.zhangxinxu.com/wordpress/2016/11/css-css3-variables-var/)》。

![](https://n1image.hjfile.cn/res7/2020/04/04/8f538a4b0a343d46e7a9992e0f717d58.png)
也就是说，IE11 不支持。当然也有简单的 hack 的方式，但其实也挺麻烦的。就目前国内 Windows 7 系统占有比例还这么高的情况下，使用 CSS 变量的成本还是挺高的。

使用 CSS 变量的网站为苹果官网。
![](https://n1image.hjfile.cn/res7/2020/04/04/9fe803a21031baf6493cc70ee8797309.png)

## 自动取色技巧

### 图片主体色彩

**方法一：通过图片存储商提供的方法来取色**

- 优点：速度快，性能好
- 缺点：图片存储上得提供这个方法才行

腾讯云数据万象通过 `imageAve` 接口获取图片主色调信息。目前支持大小在 20M 以内、长宽小于 9999 像素的图片处理。

![](https://n1image.hjfile.cn/res7/2020/04/04/269e02acf1314ea62737e069d932e72e.jpg)

访问 `https://cc.hjfile.cn/cc/img/20200325/2020032503285796778812.png?imageAve`即可获得以下结果

```json
{
  "RGB":"0xf68a61"
}
```

然后将 RGB 的颜色转换为 RGBA 的，这样的颜色略微浅一些。
给页面大容器设置背景时，背景色（`background-color`）为取色结果，而用了一层`background-image`的渐变色来作为遮罩。

```css
{
  background-color: rgba(155, 173, 185, 0.65);
  background-image: linear-gradient(rgba(28, 37, 50, 0.45) 0%, rgba(28, 37, 50, 0.7) 30%, rgba(28, 37, 50, 0.7) 100%);
}
```

> 这里也有一个小坑。`https://cc.hjfile.cn/cc/img/20200325/2020032503285796778812.png`这个地址正常为图片的地址，而如果当做接口来请求时，要保证网站与图片地址使用同样的`HTTPS`协议或`HTTP`协议。因为 HTTPS 是无法访问 HTTP 的接口的。

**方法二：借助 canvas 来获取主要色**

- 优点：不借助第三方服务
- 缺点：页面上同时获取多个 canvas 的色彩，会导致页面直接卡住，影响了正常浏览

### 图片高斯模糊

除了使用取色方案外，我们也可以使用高斯模糊的方案，其原理是用了 CSS 滤镜。CSS 滤镜还支持灰度、亮度等变化。

![](https://n1image.hjfile.cn/res7/2020/04/02/5d72b1a7c2de158c4638d7ece1c26549.png)

```html
<div class="bg" style="background-image: url(https://cc.hjfile.cn/cc/img/20180723/2018072309274407617842.jpg);">
  <div class="mask"></div>
</div>
```

```css
.bg {
  background-repeat: no-repeat;
  background-position: center;
  background-size: cover;
  filter: blur(40px);
  height: 200%;
  width: 200%;
}

.bg .mask {
  background: rgba(0,0,0,.35);
  height: 100%;
  width: 100%;
}
```

关于 CSS 滤镜，这里有两篇文章。

- MDN-[filter](https://developer.mozilla.org/zh-CN/docs/Web/CSS/filter)
- [CSS3 filter(滤镜)属性及小程序 高斯模糊和 Web 的使用](https://blog.csdn.net/Ruffaim/article/details/84430792)中提及，CSS3 滤镜同样在小程序中也适用。

---

# 草稿中

而我的目标是当用户选择头像图片后，这个小程序的色彩跟着头像图片的主要色彩来变化

提供使用 canvas 来获取主要色彩和云函数获取主要色彩的方案

在小程序上具体布局方法

主题色
苹果官方
adobe 网站
cctalk 网站？
深色模式颜色搭配？
8 个技巧
配色方案网站 https://mp.weixin.qq.com/s/iuavg_w3uhcjDov4tL-B9Q

我想写的是响应式设计对应的降级方案自适应布局，以及暗黑模式的实现。

## 变量如何传递

react props,context
css 变量传递

### 传递技巧

### 巧妙写法

取色模式
canvas 取色
七牛云取色
腾讯云取色获取图片主色调
坑：https 站点访问 http 的图片地址的限制

高瑟模糊
除了利用提取图片的主要颜色以外，我们还可以借助高斯模糊来进行带有情感类色彩的颜色模式。
比如下图中，将图片高斯模糊后作为背景（`.bg`），上面再放上一层遮罩（`.mask`）。

小程序里面要完成深色模式需要关注

- 顶部 statusBar 颜色
- 底部 tabBar 颜色
- 下拉加载的颜色
- 触底上拉的颜色
- 自定义 statusBar 的投入色？

### 自适应布局，增加暗黑模式效果

我想写的是响应式设计对应的降级方案自适应布局，以及暗黑模式的实现。

模式设定
笔记本电脑模式
iPad pro 模式
iPad mini
纯手机模式

iPad 下横竖屏监听，想办法只在大于 1024 下才开启？

宽度设定
1040-pc 模式下，两侧有留白
768-mobile 下，两侧本身带有留白

主题色设定
黑白
纯白
纯黑

参考文章

- 移动端开发的屏幕、图像、字体与布局的兼容适配 https://www.cnblogs.com/coco1s/p/11463599.html
- 紧跟潮流学设计：深色模式设计的 8 个小技巧 https://36kr.com/p/5231320

判断条件

- 响应式布局
- 自适应布局
- 完整版本
  - 自适应布局
  - js 断点
  - 扩展大屏显示

模式：触屏(mobile)、大屏(pc)
主题色：暗黑色、浅白、上黑下白（视频播放模式）
操作方式：touch、click
课程模块，从 touch 左右滑动改为局部滚动

**横屏宽度大于 1040 时，为 PC 模式，其余为 mobile**

| 设备                  | 网页宽高  |        模式         |         主题色          |
| :-------------------- | :-------: | :-----------------: | :---------------------: |
| 手机 iPhone11 pro max |  414x896  |       mobile        |          暗黑           |
| iPad 7.9 寸           | 1024x768  |       mobile        |          暗黑           |
| iPad Pro 11 寸        | 1194x834  | 横屏 pc 竖屏 mobile | 横屏-上黑下白 竖屏-暗黑 |
| iPad Pro 12.9 寸      | 1366x1024 | 横屏 pc 竖屏 mobile | 横屏-上黑下白 竖屏-暗黑 |
| pc 客户端             | 1366x1024 |       横屏 pc       |          浅白           |
| 笔记本                | 1280x800  |         pc          |        上黑下白         |
