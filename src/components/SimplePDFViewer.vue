<template>
  <div class="simple-pdf-viewer">
    <slot v-if="error" name="error" />
    <slot v-if="loading" name="loading" />
    <slot v-if="placeholder" name="placeholder" />
    <canvas
      v-for="page in pdf.numPages"
      :key="page"
      :id="`canvas-${page}`"
    />
  </div>
</template>

<script>
import PDFJS from 'pdfjs-dist'

export default {
  name: 'SimplePDFViewer',
  props: {
    scale: Number,
    pdfReference: Object,
    externalPdfWorker: String
  },
  data: () => ({
    pdf: {},
    error: false,
    loading: false,
    placeholder: true
  }),
  async mounted () {
    PDFJS.GlobalWorkerOptions.workerSrc = this.externalPdfWorker
    if (this.pdfReference) {
      this.createDocument()
    } else {
      console.warn('No pdfReference found!', this.pdfReference)
    }
  },
  methods: {
    initialize ({ placeholder = false, loading = true } = { }) {
      this.loading = loading
      this.placeholder = placeholder
      if (loading) this.$emit('loading')
    },
    finalize ({ loading = false, isError = false, _error } = { }) {
      this.error = isError
      this.loading = loading
      if (loading) this.$emit('loading')
      if (isError) this.$emit('error', _error)
    },
    async createDocument () {
      try {
        this.initialize()
        this.pdf = await PDFJS.getDocument(this.pdfReference).promise
        for (let page = 1; page <= this.pdf.numPages; page++) {
          await this.loadPage(this.pdf, page)
        }
        this.finalize()
      } catch (error) {
        this.finalize({ isError: true, _error: error })
      }
    },
    async loadPage (pdf, pageNumber) {
      try {
        const page = await pdf.getPage(pageNumber)
        const idCanvas = `canvas-${pageNumber}`
        const viewport = page.getViewport({ scale: this.scale })
        const canvas = document.getElementById(idCanvas)
        canvas.width = viewport.width
        canvas.height = viewport.height
        const canvasContext = canvas.getContext('2d')
        await page.render({ canvasContext, viewport }).promise
      } catch (error) {
        this.finalize({ isError: true, _error: error })
      }
    }
  },
  watch: {
    pdfReference (value) {
      this.createDocument()
    },
    scale (value) {
      if (value > 0 && this.pdf) {
        for (let page = 1; page <= this.pdf.numPages; page++) this.loadPage(this.pdf, page)
      }
    }
  }
}
</script>

<style scoped>
.simple-pdf-viewer {
  width: 100vw;
  overflow: auto;
}
</style>
