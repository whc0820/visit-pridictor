<template>
  <v-app>
    <v-app-bar app color="primary" dark />
    <Main :source="source" />
  </v-app>
</template>

<script>
import Main from "./components/Main";

import Vue from "vue";
import VueGoogleCharts from "vue-google-charts";
Vue.use(VueGoogleCharts);

import source from "../public/source.json";

export default {
  name: "App",

  components: {
    Main
  },

  data: () => ({
    source: source
  })
};
</script>
