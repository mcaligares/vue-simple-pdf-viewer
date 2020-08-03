<template>
  <div id="app">
    <button @click="zoomIn">ZoomIn</button>
    <button @click="zoomOut">ZoomOut</button>
    <SimplePDFViewer
      :scale="scale"
      :pdfURL="pdfURL"
      :externalPdfWorker="externalPdfWorker"
    >
      <template slot="placeholder">
        PDF Here
      </template>
      <template slot="loading">
        loading...
      </template>
      <template slot="error">
        error!
      </template>
    </SimplePDFViewer>
  </div>
</template>

<script>
import SimplePDFViewer from './components/SimplePDFViewer.vue'

export default {
  name: 'App',
  components: {
    SimplePDFViewer
  },
  mounted () {
    setTimeout(() => {
      console.log('change document')
      this.pdfURL = 'http://localhost:5000/sample.pdf'
    }, 2000)
  },
  data: () => ({
    scale: 1,
    pdfURL: null,
    externalPdfWorker: 'https://cdn.jsdelivr.net/npm/pdfjs-dist@2.4.456/build/pdf.worker.js'
  }),
  methods: {
    zoomIn () {
      this.scale += 0.25
    },
    zoomOut () {
      this.scale -= 0.25
    }
  }
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
