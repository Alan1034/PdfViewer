<!--
 * @Author: 陈德立*******419287484@qq.com
 * @Date: 2024-11-06 19:09:30
 * @LastEditTime: 2024-11-06 21:04:07
 * @LastEditors: 陈德立*******419287484@qq.com
 * @Github: https://github.com/Alan1034
 * @Description: 
 * @FilePath: \PdfViewer\src\pdf-viewer\index.vue
 * 
-->
<template>
  <div class="mask" v-if="visable">
    <div class="toolBar" :style="`width:${innerWidth}px`">
      <el-row :gutter="20" type="flex" justify="space-between">
        <el-col :span="6" class="toolBar-item">
          <span>缩放</span>
          <span>
            <el-slider style="width: 200px;" v-model="scale" :step="0.1" @change="scaleChange" :max="3" :min="0.5">
            </el-slider>
          </span>
        </el-col>
        <el-col :span="6">
          <div></div>
        </el-col>
        <el-col :span="6">
          <div></div>
        </el-col>
        <el-col :span="6" class="toolBar-item">
          <span> <el-button type="text" icon="el-icon-close" @click="closePdfViewer"></el-button></span>
        </el-col>
      </el-row>
    </div>
    <div class="pdfViewerCanvasBox">
      <div class="pdfViewer" v-if="pdfDocument">
        <canvas v-for="pageIndex in pdfDocument.numPages" :id="`pdf-viewer-canvas-` + pageIndex" :key="pageIndex"
          draggable="true"></canvas>
      </div>

    </div>
  </div>
</template>
<script>
import * as pdfjsLib from 'pdfjs-dist';
import * as PdfWorker from 'pdfjs-dist/build/pdf.worker.mjs'
const dpr = window.devicePixelRatio || 1;

export default {
  data() {
    return {
      innerWidth: window.innerWidth,
      pdfDocument: null,
      scale: 1.5
    }
  },
  props: {
    visable: {
      type: Boolean,
      default: false
    },
    path: {
      type: String,
      default: ""
    }
  },
  watch: {
    async visable(val) {
      if (val) {
        await this.pdfLoader()
        this.renderPage(1)
      }
    }
  },
  methods: {
    async pdfLoader() {
      if (!this.path) {
        console.warn("无pdf地址，请传入path", this.path)
        return
      }
      // Loading a document.
      const loadingTask = pdfjsLib.getDocument(this.path);
      this.pdfDocument = await loadingTask.promise;
    },
    //渲染pdf文件
    async renderPage(num) {
      // Request a first page
      if (!this.pdfDocument?.getPage) {
        console.warn("pdfDocument实例获取失败", this.pdfDocument)
        return
      }
      const pdfPage = await this.pdfDocument.getPage(num);
      // Display page on the existing canvas with 100% scale.
      const viewport = pdfPage.getViewport({ scale: this.scale });
      const canvasId = 'pdf-viewer-canvas-' + num;
      const canvas = document.getElementById(canvasId);

      const ctx = canvas.getContext("2d");
      const bsr = ctx.webkitBackingStorePixelRatio || ctx.mozBackingStorePixelRatio || ctx.msBackingStorePixelRatio || ctx.oBackingStorePixelRatio || ctx.backingStorePixelRatio || 1;
      const ratio = dpr / bsr;
      canvas.width = viewport.width * ratio;
      canvas.height = viewport.height * ratio;
      // canvas.style.width = `${screen.availWidth}px`
      // // 缩放
      ctx.setTransform(ratio, 0, 0, ratio, 0, 0);
      // 【重要】关闭抗锯齿
      ctx.mozImageSmoothingEnabled = false;
      ctx.webkitImageSmoothingEnabled = false;
      ctx.msImageSmoothingEnabled = false;
      ctx.imageSmoothingEnabled = false;
      // const renderTask = 
      pdfPage.render({
        canvasContext: ctx,
        viewport,
      });
      // 异步加载
      // await renderTask.promise;
      // 加载全部
      if (num < this.pdfDocument.numPages) {
        await this.renderPage(num + 1);
      }
      return
    },
    //放大缩小
    scaleChange(scaleVal) {
      this.scale = scaleVal;
      this.renderPage(1)
    },
    closePdfViewer() {
      this.$emit('update:visable', false)
    }
  }
}
</script>
<style lang="scss" scoped>
.mask {
  position: fixed;
  top: 39px;
  left: 0;
  right: 0;
  bottom: 0;
  background: rgba(0, 0, 0, 0.5);
  z-index: 99999;
  overflow: auto;

  .toolBar {
    background-color: white;
    position: fixed;
    top: 0;
    left: 0;
    // right: 0;

    .toolBar-item {
      display: flex;
      justify-content: space-evenly;
      align-items: center;
      gap: 8px;
      font-size: 16px;
    }

  }

  .pdfViewerCanvasBox {
    position: relative;
    // margin: 75px auto 70px;
    width: 100%;
    height: 100%;
    margin-top: 8px;


    .pdfViewer {
      position: absolute;
      left: 50%;
      transform: translateX(-50%); // translateY(-50%);
      text-align: center;

      canvas {
        content-visibility: auto;
        contain-intrinsic-size: 1339px 1900px;
      }
    }
  }

}
</style>