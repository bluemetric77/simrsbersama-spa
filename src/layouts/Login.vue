<template>
  <q-layout view="hHh Lpr fFf">
    <!-- Be sure to play with the Layout demo on docs -->
    <q-page-container>
      <!-- This is where pages get injected -->
      <router-view />
    </q-page-container>
  </q-layout>
</template>
<script>
import { defineComponent } from 'vue'

export default defineComponent({
  name: 'Auth',
  setup() {
    return {
      leftDrawer: true
    }
  }
})
</script>
