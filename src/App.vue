<template>
  <v-app>
    <v-app-bar app color="primary" dark>
      <div class="d-flex align-center">
        <v-img
          alt="Vuetify Logo"
          class="shrink mr-2"
          contain
          src="https://cdn.vuetifyjs.com/images/logos/vuetify-logo-dark.png"
          transition="scale-transition"
          width="40"
        />

        <v-img
          alt="Vuetify Name"
          class="shrink mt-1 hidden-sm-and-down"
          contain
          min-width="100"
          src="https://cdn.vuetifyjs.com/images/logos/vuetify-name-dark.png"
          width="100"
        />
      </div>

      <v-spacer></v-spacer>

      <v-btn
        href="https://github.com/vuetifyjs/vuetify/releases/latest"
        target="_blank"
        text
      >
        <span class="mr-2">Latest Release</span>
        <v-icon>mdi-open-in-new</v-icon>
      </v-btn>
    </v-app-bar>

    <v-content>
      <BarChart id="first" />
      <BarChart id="second" />
      <PackChart :data="loadData" />
    </v-content>
  </v-app>
</template>

<script lang="ts">
import Vue from "vue";
import PackChart from "@/components/PackChart.ts.vue";
import BarChart from "@/components/BarChart.ts.vue";
import tweets from "@/data/tweets.json";

export default Vue.extend({
  name: "App",

  components: {
    PackChart,
    BarChart
  },

  data: () => ({
    loadData: {}
  }),
  methods: {
    async fetchData() {
      this.loadData = tweets.tweets;
    }
  },
  async mounted() {
    await this.fetchData();
  }
});
</script>
