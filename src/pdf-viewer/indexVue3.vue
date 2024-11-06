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
<script setup lang="ts">
import { ref, watch, } from "vue";
import * as pdfjsLib from 'pdfjs-dist';
import * as PdfWorker from 'pdfjs-dist/build/pdf.worker.mjs'
console.log(PdfWorker)
const innerWidth = ref(window.innerWidth)
// let pdfPath = "https://mozilla.github.io/pdf.js/web/compressed.tracemonkey-pldi-09.pdf";
const props = defineProps({
  visable: {
    type: Boolean,
    default: false
  },
  path: {
    type: String,
    default: ""
  }
})
const emit = defineEmits(["update:visable"])
const pdfDocument = ref(); // 保存加载的pdf文件流
const scale = ref(1.5); //放大系数
watch(
  () => props.visable,
  async (val) => {
    if (val) {
      await pdfLoader()
      renderPage(1)
    }
  }
)
const pdfLoader = async () => {
  if (!props.path) {
    console.warn("无pdf地址，请传入path", props.path)
    return
  }
  if (typeof window !== "undefined" && "Worker" in window) {
    console.warn("newworker")
    pdfjsLib.GlobalWorkerOptions.workerPort = new Worker(
      PdfWorker
    );
  }
  // Loading a document.

  const loadingTask = pdfjsLib.getDocument(props.path);
  pdfDocument.value = await loadingTask.promise;
}
const dpr = window.devicePixelRatio || 1;

//渲染pdf文件
const renderPage = async (num) => {
  // Request a first page
  if (!pdfDocument.value?.getPage) {
    console.warn("pdfDocument实例获取失败", pdfDocument.value)
    return
  }
  const pdfPage = await pdfDocument.value.getPage(num);
  // Display page on the existing canvas with 100% scale.
  const viewport = pdfPage.getViewport({ scale: scale.value });
  const canvasId = 'pdf-viewer-canvas-' + num;
  const canvas: any = document.getElementById(canvasId);

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
  if (num < pdfDocument.value.numPages) {
    await renderPage(num + 1);
  }
  return
}
//放大缩小
const scaleChange = (scaleVal) => {
  scale.value = scaleVal;
  renderPage(1)
}
const closePdfViewer = (scaleVal) => {
  emit('update:visable', false)
}
defineExpose({ scaleChange, closePdfViewer, pdfDocument: pdfDocument.value, scale: scale.value })
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