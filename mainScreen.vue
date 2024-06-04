<template>
  <div class="printscreen" data-html2canvas-ignore>
    <section class="printscreen-box" v-if="show">
      <div class="printscreen-content">
        <div class="printscreen-img">
          <div class="printscreen-mask" @click="onClickImgDown"></div>
        </div>
        <div class="printscreen-tool" v-if="toolShow">
          <button @click="onClickImgDown">下载</button>
        </div>
      </div>
    </section>
  </div>
</template>

<script>
export default {
  data() {
    return {
      show: false,
      toolShow: false,
      printDom: 'body', // 需要截图的dom
      canvasSizeDom: 'body', // 截图的大小尺寸参考dom
      imgSrc: null, // 截图的base64
      allAutoScaleDom: null, // 如果套了全局自动缩放组件填写字符串形式如:'#id','.class','tagname'，没有则填null。 获取全局自动缩放组件的dom方便截图的定位与动画效果，不影响最后下载图片效果
      animationTime: 800, // 动画时间 duration
      animationSpeed: 'linear', // 动画速度类型 timing
      animationEndType: 'forwards', // 动画结束时的状态 fill
      printImgScale: 0.1, // 截图缩放比例
      dieoutTime: 40000, // 多少时间后盒子移出
      dieoutAnimationTime: 500, // 盒子移除动画时间
      toolShowTimer: null,
      dieoutTimer: null,
      clearDomTimer: null,
    };
  },
  mounted() {
    this.addKeyDown();
  },
  methods: {
    addKeyDown() {
      window.addEventListener('keydown', this.keyDown);
    },
    keyDown(event) {
      if (event.altKey && event.shiftKey && event.key.toLowerCase() === 's') {
        event.preventDefault();
        this.printScreen();
      }
    },
    printScreen() {
      this.clear();
      this.show = true;
      const printDom = document.querySelector(this.printDom);
      // getBoundingClientRect 元素实际大小 此方法返回元素的大小及其相对于视口的位置，返回的对象包含了width和height属性，它们表示元素的可见部分的尺寸，不包括边框、外边距和滚动条，但包括内边距。这个方法是获取元素在屏幕上的实际大小的一个好方法。
      // clientWidth clientHeight 盒子的大小缩放之前css样式内的宽高 返回元素内容区的可视宽度和高度，不包括边框、滚动条以及外边距，但包括内边距（padding）。
      const width = document.querySelector(this.canvasSizeDom).clientWidth;
      const height = document.querySelector(this.canvasSizeDom).clientHeight;
      html2canvas(printDom, { allowTaint: true, logging: false, backgroundColor: null }).then(canvas => {
        if (!canvas) {
          console.error('Canvas is not created');
          return;
        }
        const base64Img = canvas.toDataURL('image/png');
        this.src = base64Img;
        const canvasDom = document.querySelector('.printscreen-box');
        if (canvasDom) {
          canvasDom.style.width = width + 'px';
          canvasDom.style.height = height + 'px';
        }
        const imgContentBoxDom = document.querySelector('.printscreen-content');
        // 如果有全局缩放组件则设置位移和中心点的绝对单位px
        if (this.allAutoScaleDom) {
          const allAutoScaleDom = document.querySelector(this.allAutoScaleDom)
            ? document.querySelector(this.allAutoScaleDom)
            : document.querySelector('.printscreen');
          imgContentBoxDom.style.transformOrigin = allAutoScaleDom.clientWidth + 'px' + ' top';
        } else {
          imgContentBoxDom.style.transformOrigin = 'right top';
        }
        // 给图片外层盒子增加动画效果
        if (imgContentBoxDom) {
          imgContentBoxDom.animate(
            [
              {
                border: '12px solid white',
                borderRadius: '8px',
                transform: 'scale(1)',
                offset: 0,
              },
              {
                border: '20px solid white',
                borderRadius: '20px',
                transform: `scale(${(1 - this.printImgScale) / 2})`,
                offset: 0.5,
              },
              {
                border: '40px solid white',
                borderRadius: '40px',
                transform: `scale(${this.printImgScale})`,
                offset: 1,
              },
            ],
            {
              duration: this.animationTime,
              timing: this.animationSpeed,
              fill: this.animationEndType,
            }
          );
          // 给dom设置背景图 截图图片base64
          const imgDom = document.querySelector('.printscreen-img');
          if (imgDom) {
            imgDom.style.backgroundImage = `url(${this.src})`;
          }
          // 给蒙层增加动画效果
          const maskDom = document.querySelector('.printscreen-mask');
          if (maskDom) {
            maskDom.animate(
              [
                {
                  opacity: 0.9,
                  offset: 0,
                },
                {
                  opacity: 0.4,
                  offset: 0.8,
                },
                {
                  opacity: 0,
                  offset: 1,
                },
              ],
              {
                duration: this.animationTime,
                timing: this.animationSpeed,
                fill: this.animationEndType,
              }
            );
          }
          // 将工具栏展示
          this.toolShowTimer = setTimeout(() => {
            this.toolShow = true;
          }, this.animationTime - this.animationTime / 20);
          // 盒子移出并销毁
          this.dieoutTimer = setTimeout(() => {
            // 如果有全局缩放组件则设置位移和中心点的绝对单位px
            let topOffset = '-100%';
            if (this.allAutoScaleDom) {
              const allAutoScaleDom = document.querySelector(this.allAutoScaleDom)
                ? document.querySelector(this.allAutoScaleDom)
                : document.querySelector('.printscreen');
              topOffset = allAutoScaleDom.clientHeight * -1 + 'px';
            }
            imgContentBoxDom.animate(
              [
                {
                  transform: `scale(${this.printImgScale})`,
                  offset: 0,
                },
                {
                  transform: `translate(0,${topOffset}) scale(${this.printImgScale})`,
                  offset: 1,
                },
              ],
              {
                duration: this.dieoutAnimationTime,
                timing: this.animationSpeed,
                fill: this.animationEndType,
              }
            );
            // 销毁清空组件
            this.clearDomTimer = setTimeout(() => {
              this.caler();
            }, this.dieoutAnimationTime);
          }, this.dieoutTime);
        }
      });
    },
    clear() {
      clearTimeout(this.toolShowTimer);
      this.toolShowTimer = null;
      clearTimeout(this.dieoutTimer);
      this.dieoutTimer = null;
      clearTimeout(this.clearDomTimer);
      this.clearDomTimer = null;
      this.show = false;
      this.toolShow = false;
    },
    onClickImgDown() {
      const link = document.createElement('a');
      link.href = this.src;
      link.download = 'screenshot';
      link.style.display = 'none';
      document.body.appendChild(link);
      link.click();
      document.body.removeChild(link);
      this.clear();
    },
  },
  destroyed() {
    this.clear();
    window.removeEventListener('keydown', this.keyDown);
  },
};
</script>

<style lang="less" scoped>
.printscreen {
  width: 100%;
  height: 100%;
  position: fixed;
  top: 0;
  left: 0;
  pointer-events: none;
  z-index: 999 !important;
  .printscreen-box {
    pointer-events: none;
    .printscreen-content {
      width: 100%;
      height: 100%;
      pointer-events: none;
      position: relative;
      .printscreen-img {
        width: 100%;
        height: 100%;
        pointer-events: initial;
        cursor: pointer;
        pointer-events: none;
        background-size: 100% 100%;
        background-repeat: no-repeat;
        background-position: center center;
        pointer-events: initial;
        cursor: pointer;
        .printscreen-mask {
          width: 100%;
          height: 100%;
          position: absolute;
          top: 0;
          left: 0;
          background-color: #ccc;
        }
      }
      .printscreen-tool {
        width: 100%;
        height: 300px;
        position: absolute;
        bottom: -400px;
        display: flex;
        justify-content: center;
        align-items: center;
        pointer-events: initial;
        button {
          position: relative;
          cursor: pointer;
          width: 80%;
          height: 300px;
          font-size: 200px;
          color: #2440b3;
          border-radius: 200px;
          box-shadow: 0px 0px 200px 0px rgba(0, 0, 0, 0.2);
          border: none;
          background-color: #fff;
          display: flex;
          align-items: center;
          justify-content: center;
        }
      }
    }
  }
}
</style>
