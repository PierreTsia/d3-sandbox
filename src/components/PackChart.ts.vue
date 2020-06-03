<template>
  <v-container class="d-flex flex-column justify-center">
    <h1 class="title text-center">Circle Pack in D3.js</h1>
    <h2 class="subtitle-2 text-center">{{ message }}</h2>
    <svg :height="height" :width="width" class="d-block mx-auto">
      <g transform="translate(50, 50)">
        <circle
          v-for="c in packChartData"
          :key="c.id"
          :r="c.r"
          :cx="c.x"
          :cy="c.y"
          :fill="c.fill"
          :stroke="c.stroke"
        ></circle>
      </g>
    </svg>
  </v-container>
</template>

<script lang="ts">
import * as d3 from "d3";
const colorScale = d3
  .scaleOrdinal()
  .range(["#5EAFC6", "#FE9922", "#93c464", "#75739F"]);
import {
  defineComponent,
  computed,
  reactive,
  ref,
  Ref,
  toRefs,
  watch
} from "@vue/composition-api";

const PackChart = defineComponent({
  name: "BarChart",
  props: {
    data: {}
  },
  setup(props, ctx) {
    const state = reactive({
      height: 600,
      width: 600,
      message: "Chart Component"
    });
    const chartData: Ref<any> = ref([]);

    const packData = computed(() => {
      const nestedTweets = d3
        .nest()
        .key((d: any) => d.user)
        .entries(chartData.value);

      const packableTweets = { id: "All tweets", values: nestedTweets };
      return d3
        .hierarchy(packableTweets, (d: any) => d.values)
        .sum((d: any) =>
          d.retweets ? d.retweets.length + d.favorites.length + 1 : 1
        );
    });

    const packChartData = computed(() => {
      const packChart = d3
        .pack()
        .size([500, 500])
        .padding(10);
      const output = packChart(packData.value).descendants();
      return output.map((d: any, index: number) => {
        const { r, x, y, depth } = d;

        const fill = colorScale(depth);
        return {
          id: index + 1,
          r,
          x,
          y,
          fill,
          stroke: "grey"
        };
      });
    });

    watch(
      () => props.data,
      newData => {
        chartData.value = newData;
      }
    );

    return { ...toRefs(state), packChartData };
  }
});
export default PackChart;
</script>

<style scoped></style>
