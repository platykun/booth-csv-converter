<template>
  <v-app>
    <app-header />
    <v-content>
      <nuxt />
    </v-content>
  </v-app>
</template>

<script>
import appHeader from '@/components/header'
export default {
  components: { appHeader }
}
</script>

<style>
body {
  margin: 0 !important;
}
</style>
