<template>
  <v-app>
    <v-app-bar app flat color="grey lighten-3">
      <v-toolbar-title class="app-title">Morse Code Editor</v-toolbar-title>
    </v-app-bar>
    <InputDisplay />
  </v-app>
</template>

<script lang="ts">
import Vue from "vue";
import InputDisplay from "./components/InputDisplay.vue";

export default Vue.extend({
  name: "App",

  components: {
    InputDisplay
  },

  data: () => ({
    //
  })
});
</script>

<style lang="scss">
// Web Fontのちらつき対策
html {
  visibility: hidden;
}
html.wf-active,
html.loading-delay {
  visibility: visible;
}
.app-title {
  font-family: "Candal", sans-serif;
  font-size: 2.25rem !important;
}
</style>