<template>
  <div class="e-nuxt-container">
    <CsvReader />
  </div>
</template>

<script>
import { remote } from 'electron'
import CsvReader from '@/components/CsvReader.vue'

export default {
  components: {
    CsvReader
  },
  data () {
    return {
      externalContent: ''
    }
  },
  methods: {
    openURL (url) {
      remote.shell.openExternal(url)
    }
  }
}
</script>

<style>
.e-nuxt-container {
  min-height: calc(100vh - 50px);
  background: linear-gradient(to right, #ece9e6, #ffffff);
  font-family: Helvetica, sans-serif;
}
</style>
