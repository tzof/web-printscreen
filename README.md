# web-printscreen

> **基于 html2canvas 实现 web 端的截图动画 vue 组件**

## 注意事项：

1. 首先将 html2canvas.min.js 文件在 html 模版中导入 js，或者使用 npm 或 yarn 下载 html2canvas 并导入到 printscreen.vue 中：`import html2canvas from 'html2canvas'`。
2. 在 main.vue 全局引入或者相应模块引入该组件。
3. 如果项目中有使用全局缩放自适应不同分辨率则需要在 printscreen.vue 中修改`allAutoScaleDom`变量的值为缩放组件的 dom 表示，字符串形式如:'#id','.class','tagname'。获取全局自动缩放组件的 dom 方便与截图的定位与动画效果，不影响最后下载图片效果。
4. 同时按下 alt+shift+s 即可截图，如快捷键冲突请在 printscreen.vue 中`keyDown()`函数中修改按键监听。

## 参数意义如下：

```js
show: false,
toolShow: false,
printDom: 'body', // 需要截图的dom
canvasSizeDom: 'body', // 截图的大小尺寸参考dom
imgSrc: null, // 截图的base64
allAutoScaleDom: '#autoScale', // 如果套了全局自动缩放组件填写字符串形式如:'#id',- ass','tagname'，没有则填null。 获取全局自动缩放组件的dom方便截图的定位与动画效果- 影响最载图片效果
animationTime: 800, // 动画时间 duration
animationSpeed: 'linear', // 动画速度类型 timing
animationEndType: 'forwards', // 动画结束时的状态 fill
printImgScale: 0.1, // 截图缩放比例
dieoutTime: 40000, // 多少时间后盒子移出
dieoutAnimationTime: 500, // 盒子移除动画时间
toolShowTimer: null,
dieoutTimer: null,
clearDomTimer: null,
```
