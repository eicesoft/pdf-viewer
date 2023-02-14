<template>
  <div
    :style="{ width: width + 10 + 'px', height: height + 'px' }"
    class="viewerContainer"
    ref="contrainer"
  >
    <canvas
      v-for="num in pageCount"
      ref="canvas"
      name="canvas"
      class="pdf-page"
      :key="num"
    />
  </div>
</template>

<script>
import * as pdfjsLib from "pdfjs-dist/legacy/build/pdf";
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
      default: 800
    }
  },
  data() {
    return {
      pdfDoc: null, // pdfjs 生成的对象
      pageNum: 1, //
      scale: 1.0, // 放大倍数
      pageCount: 0 // 总页数
    };
  },
  // computed: {
  //   ctx() {
  //     const id = document.getElementById("the-canvas");
  //     return id.getContext("2d");
  //   }
  // },
  created() {
    const vm = this;
    pdfjsLib.getDocument(this.src).promise.then(pdfDocument => {
      vm.pdfDoc = pdfDocument;
      vm.pageCount = pdfDocument.numPages;
      vm.renderPage(vm.pageNum);
    });
  },

  mounted() {
    // const vm = this;
    // this.$nextTick(() => {
    //   console.log("nextTick", this.$refs.canvas);
    // });
    // const vm = this;
    // this.$nextTick(() => {
    // pdfjsLib.getDocument(vm.src).promise.then(pdfDocument => {
    //   // console.log(pdfDocument);
    //   vm.pdfDoc = pdfDocument;
    //   vm.pageCount = pdfDocument.numPages;
    //   vm.render(vm.pageNum);
    // });
    // });
  },
  methods: {
    renderPage(num) {
      const canvas_list = document.getElementsByClassName("pdf-page");

      // const canvas = document.getElementById("the-canvas");
      // const canvas = this.$refs.canvas;
      const vm = this;
      this.pdfDoc.getPage(num).then(page => {
        const canvas = canvas_list[num - 1];

        var defViewport = page.getViewport({ scale: vm.scale });
        var scale = vm.width / defViewport.width;
        var viewport = page.getViewport({ scale: scale });
        var outputScale = window.devicePixelRatio || 1;

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
        // Wait for rendering to finish
        renderTask.promise.then(function () {
          // vm.pageRendering = false;
          // vm.pageNumPending =
          // if (vm.pageNumPending !== null) {
          // New page rendering is pending
          console.log("Render next page:", vm.pageNum, vm.pageCount);
          if (vm.pageNum < vm.pageCount) {
            vm.pageNum++;
            console.log("Render next page:", vm.pageNum);

            vm.renderPage(vm.pageNum);
          }
          // }
        });
      });
    }
  }
};
</script>

<style scoped>
.viewerContainer {
  border-radius: 6px;
  overflow-x: auto;
  border: 1px solid #ececec;
}

.viewerContainer::-webkit-scrollbar {
  width: 7px;
  height: 1px;
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
  padding: 10px 0;
  display: block;
  border-bottom: 1px solid #ececec;
  /* box-shadow: 1px 1px 3px rgba(197, 197, 197, 0.2); */
}
.pdf-page:first-child,
.pdf-page:last-child {
  padding: 0;
}
</style>
