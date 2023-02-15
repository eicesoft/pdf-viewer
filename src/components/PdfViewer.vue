<template>
  <div
    :style="{ width: width + 8 + 'px', height: height + 'px' }"
    class="viewerContainer"
  >
    <canvas v-for="num in pageCount" class="pdf-page" :key="num" />
  </div>
</template>

<script>
import * as pdfjsLib from "pdfjs-dist/legacy/build/pdf";
// import * as pdfjsViewer from "pdfjs-dist/legacy/web/pdf_viewer";
// console.log(pdfjsViewer);
import pdfjsWorker from "pdfjs-dist/build/pdf.worker.entry";
pdfjsLib.GlobalWorkerOptions.workerSrc = pdfjsWorker;
export default {
  name: "PdfView",
  components: {},
  props: {
    src: String,
    width: {
      type: Number,
      default: 800
    },
    height: {
      type: Number,
      default: 1024
    }
  },
  data() {
    return {
      pdfDoc: null, // pdfjs 生成的对象
      pageNum: 1, //
      scale: 1, // 放大倍数
      pageCount: 0 // 总页数
    };
  },
  created() {
    const vm = this;
    pdfjsLib.getDocument(this.src).promise.then(pdfDocument => {
      vm.pdfDoc = pdfDocument;
      vm.pageCount = pdfDocument.numPages;
      vm.renderPage(vm.pageNum);
    });
  },
  methods: {
    renderPage(num) {
      const canvas_list = document.getElementsByClassName("pdf-page");
      const vm = this;
      this.pdfDoc.getPage(num).then(page => {
        const canvas = canvas_list[num - 1];
        const defViewport = page.getViewport({ scale: vm.scale });
        let scale = vm.width / defViewport.width;
        const viewport = page.getViewport({ scale: scale });
        const outputScale = window.devicePixelRatio || 1;

        console.log([canvas_list, canvas, viewport, outputScale]);

        canvas.width = Math.floor(viewport.width * outputScale);
        canvas.height = Math.floor(viewport.height * outputScale);
        canvas.style.width = Math.floor(viewport.width) + "px";
        canvas.style.height = Math.floor(viewport.height) + "px";

        var transform =
          outputScale !== 1 ? [outputScale, 0, 0, outputScale, 0, 0] : null;
        const ctx = canvas.getContext("2d");
        var renderContext = {
          canvasContext: ctx,
          viewport: viewport,
          transform: transform
        };
        let renderTask = page.render(renderContext);

        renderTask.promise.then(function () {
          console.log("Render next page:", vm.pageNum, vm.pageCount);
          if (vm.pageNum < vm.pageCount) {
            vm.pageNum++;
            console.log("Render next page:", vm.pageNum);

            vm.renderPage(vm.pageNum);
          }
        });
      });
    }
  }
};
</script>

<style scoped>
.viewerContainer {
  padding: 8px;
  border-radius: 6px;
  overflow: auto;
  border: 1px solid #ececec;
  background: rgba(158, 158, 158, 0.9);
  display: flex;
  height: 700px;
  align-items: center;
  flex-direction: row;
  flex-wrap: wrap;
  justify-content: space-between;
}

.viewerContainer::-webkit-scrollbar {
  width: 7px;
  height: 7px;
}

.viewerContainer::-webkit-scrollbar-thumb {
  border-radius: 7px;
  background: #d6d6d6;
}

.viewerContainer::-webkit-scrollbar-track {
  box-shadow: inset 0 0 3px rgba(197, 197, 197, 0.2);
  border-radius: 7px;
  background: #ededed;
}

.pdf-page {
  display: block;
  margin: 0 0 8px 0;
  /* box-shadow: 1px 1px 3px rgba(197, 197, 197, 0.2); */
}

.pdf-page:last-child {
  margin: 0 0 0 0;
}
</style>
