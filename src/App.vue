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
      <BarChart id="first" name="Chart One" :height="400" :width="400" />
      <BarChart
        id="second"
        name="Chart Two"
        :height="400"
        :width="400"
        data-set="hilly"
        default-color="#d1741d"
      />
      <BarChart
        id="third"
        name="Chart Three"
        :height="200"
        :width="300"
        data-set="parabola"
        default-color="#9D1CD1"
      />
      <BarChart
        id="fourth"
        name="Chart Four"
        :height="1200"
        :width="300"
        data-set="hilly"
        default-color="#ea19d5"
      />
      <!-- <PackChart :data="loadData" /-->>
    </v-content>
  </v-app>
</template>

<script lang="ts">
import Vue from "vue";
//import PackChart from "@/components/PackChart.ts.vue";
import BarChart from "@/components/BarChart.ts.vue";
import tweets from "@/data/tweets.json";

export default Vue.extend({
  name: "App",

  components: {
    /*    PackChart,*/
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
