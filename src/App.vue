<template>
  <div id="app">
    <EditorTui/>
  </div>
</template>

<script>
import EditorTui from './components/EditorTui.vue'

export default {
  name: 'App',
  components: {
    EditorTui
  },
  mounted() {
    document.title = "Generator Sertifikat"
  }
}
</script>


<style>
@import url("https://cdn.jsdelivr.net/npm/tui-image-editor@3.10.0/dist/tui-image-editor.css");
.tui-image-editor-container {
  min-height: 800px;
}
body, html {
  font-family: "Noto Sans", sans-serif;
  margin: 0;
  padding: 0;
}
</style>
