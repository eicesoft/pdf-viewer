<template>
  <div id="app">
    <PdfViewer :width="800" :height="700" :src="'./test.pdf'" />
  </div>
</template>

<script>
import PdfViewer from "./components/PdfViewer.vue";

export default {
  name: "App",
  components: {
    PdfViewer
  }
};
</script>

<style>
#app {
}
</style>
