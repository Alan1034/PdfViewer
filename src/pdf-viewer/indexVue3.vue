<template>
  <div class="mask" v-if="visable">
    <div class="toolBar" :style="{ width: `${innerWidth}px` }">
      <el-row :gutter="20" type="flex" justify="space-between" v-if="componentType === 'Element Plus'">
        <el-col :span="6" class="toolBar-item">
          <span>缩放</span>
          <span>
            <el-slider style="width: 200px;" v-model="scale" :step="scaleParams.step" @change="scaleChange"
              :max="scaleParams.max" :min="scaleParams.min">
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
      <t-row :gutter="20" v-if="componentType === 'Tdesign Mobile Vue'">
        <t-col :span="18" class="toolBar-item">
          <span>缩放</span>
          <span>
            <t-slider style="width: 200px;" v-model="scale" :step="scaleParams.step" @change="scaleChange"
              :max="scaleParams.max" :min="scaleParams.min" theme="capsule">
            </t-slider>
          </span>
        </t-col>
        <t-col :span="6" class="toolBar-item">
          <span> <t-button variant="text" @click="closePdfViewer">x</t-button></span>
        </t-col>
      </t-row>
    </div>
    <div class="pdfViewerCanvasBox">
      <div class="pdfViewer" v-if="numPages">
        <canvas v-for="pageIndex in numPages" :id="`pdf-viewer-canvas-` + pageIndex" :key="pageIndex"
          draggable="true"></canvas>
      </div>

    </div>

  </div>
</template>
<script setup lang="ts">
import { ref, watch } from "vue";
import { MessageQueue } from "network-spanner"
import * as pdfjsLib from 'pdfjs-dist';
import 'pdfjs-dist/build/pdf.worker.mjs'
// import type { ComponentType } from "../types/componentType";
const dpr = window.devicePixelRatio || 1;
const innerWidth = ref(window.innerWidth)
const ifMobile = innerWidth.value < 768
// let pdfPath = "https://mozilla.github.io/pdf.js/web/compressed.tracemonkey-pldi-09.pdf";
const props = defineProps({
  visable: {
    type: Boolean,
    default: false
  },
  path: {
    type: String,
    default: ""
  },
  componentType: {
    type: String,
    default: "Element Plus"
  }
})
const desktopParams = {
  step: 0.1,
  max: 3,
  min: 0.5,
  scale: 1.5
}
const mobileParams = {
  step: 0.01,
  max: 1,
  min: 0.12,
  scale: 2
}
let scaleParams: any = {}

if (ifMobile) {
  // 判断适配屏幕大小
  scaleParams = { ...mobileParams };
} else {
  scaleParams = { ...desktopParams };
}
const emit = defineEmits(["update:visable"])
let pdfDocument: any = ""; // 保存加载的pdf文件流
const numPages = ref(0); // 保存加载的pdf文件流
const scale = ref(scaleParams.scale); //放大系数
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
  const loadingTask = pdfjsLib.getDocument(props.path);
  pdfDocument = await loadingTask.promise;
  numPages.value = pdfDocument.numPages
}

//渲染pdf文件
const renderPage = async (num) => {
  // Request a first page
  if (!pdfDocument.getPage) {
    console.warn("pdfDocument实例获取失败", pdfDocument)
    return
  }

  const pdfPage = await pdfDocument.getPage(num);
  // Display page on the existing canvas with 100% scale.
  const viewport = pdfPage.getViewport({ scale: scale.value });
  const canvasId = 'pdf-viewer-canvas-' + num;
  const canvas: any = document.getElementById(canvasId);
  const ctx = canvas.getContext("2d");
  const bsr = ctx.webkitBackingStorePixelRatio || ctx.mozBackingStorePixelRatio || ctx.msBackingStorePixelRatio || ctx.oBackingStorePixelRatio || ctx.backingStorePixelRatio || 1;
  const ratio = dpr / bsr;
  canvas.width = viewport.width * ratio;
  canvas.height = viewport.height * ratio;
  if (ifMobile) {
    canvas.style.width = `${screen.availWidth}px`
  }

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
  if (num < numPages.value) {
    await renderPage(num + 1);
  }
  return
}
//放大缩小
const scaleChange = (scaleVal) => {
  scale.value = scaleVal;
  MessageQueue({ value: 1, somePromise: renderPage })
}
const closePdfViewer = (scaleVal) => {
  emit('update:visable', false)
}
defineExpose({ scaleChange, closePdfViewer, pdfDocument, scale: scale.value })
</script>
<style lang="scss" scoped>
.mask {

  & {
    @media screen and (min-width: 768px) {
      top: 39px;
    }

    @media screen and (max-width: 768px) {
      top: 0;
    }
  }

  & {
    position: fixed;
    left: 0;
    right: 0;
    bottom: 0;
    background: rgba(0, 0, 0, 0.5);
    z-index: 99999;
    overflow: auto;
  }


  .toolBar {
    & {
      @media screen and (min-width: 768px) {
        top: 0
      }

      @media screen and (max-width: 768px) {
        bottom: 0
      }
    }

    & {
      background-color: white;
      position: fixed;
      left: 0;
      right: 0;
      z-index: 1;
    }



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