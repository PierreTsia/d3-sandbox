<template>
  <v-container class="d-flex flex-column justify-center">
    <h1 class="title text-center">{{ title }}</h1>
    <h2 class="subtitle-2 text-center">select formula</h2>
    <svg :id="id" :height="outerHeight" :width="outerWidth"></svg>
    <v-row>
      <v-col class="col-2">
        <v-select :items="formulas" outlined v-model="activeFormula"></v-select>
      </v-col>
      <v-col class="col-3">
        <v-slider
          v-model="radius"
          min="1"
          max="10"
          label="Size"
          inverse-label
        ></v-slider>
      </v-col>
      <v-col class="col-5">
        <v-color-picker
          class="ma-2"
          hide-canvas
          @input="colorObj => (color = colorObj.hex)"
        ></v-color-picker>
      </v-col>
    </v-row>
  </v-container>
</template>

<script lang="ts">
import * as d3 from "d3";
import {
  computed,
  defineComponent,
  onMounted,
  watch,
  reactive,
  toRefs
} from "@vue/composition-api";

const BarChart = defineComponent({
  name: "BarChart",
  props: {
    id: {}
  },
  setup(props) {
    const state = reactive({
      title: "Line Dots Component",
      outerWidth: 400,
      outerHeight: 200,
      color: "red",
      radius: 5,
      margin: { left: 30, top: 30, right: 30, bottom: 30 },
      formulas: ["sine", "parabola", "hilly"],
      activeFormula: "sine"
    });
    const generateData = (arg = "sine") => {
      switch (arg) {
        case "sine":
          return d3.range(0, 5, 0.05).map(x => [x, Math.sin(x)]);
        case "parabola":
          return d3.range(-10, 10, 0.2).map(x => [x, x * x]);
        case "hilly":
          return d3
            .range(-15, 15, 0.3)
            .map(x => [
              x,
              Math.abs(
                Math.sin(x / 4) + 0.5 * Math.cos(x / 2) + 0.3 * Math.sin(x)
              )
            ]);
      }
      return [];
    };

    const innerWidth = computed(
      () => state.outerWidth - state.margin.left - state.margin.right
    );
    const innerHeight = computed(
      () => state.outerHeight - state.margin.top - state.margin.bottom
    );

    const randomData = computed(() => {
      return generateData(state.activeFormula);
    });

    const rootSvg = () => d3.select(`#${props.id}`);
    const dots = () => rootSvg().selectAll("circle");

    const radiusByValue = (value: number) =>
      Math.abs((state.radius * +value * 20) / 100);

    const minmax = (data: number[]) =>
      [d3.min(data) ?? 0, d3.max(data) ?? 100] as [number, number];

    const draw = async (data: any, parent: any) => {
      parent.selectAll("g").remove();
      const root = parent.append("g");

      const [x, y] = [d3.scaleLinear(), d3.scaleLinear()];
      const xAxis = d3.axisBottom(x).ticks(5);
      const yAxis = d3.axisLeft(y).ticks(5);
      const xAxisG = root.append("g");
      const yAxisG = root.append("g");
      const xMinMax = minmax(data.map(([x]: any) => x));
      const yMinMax = minmax(data.map(([_, y]: any) => y));
      x.domain(xMinMax).range([0, innerWidth.value]);
      y.domain(yMinMax).range([innerHeight.value, 0]);
      xAxisG.attr("transform", `translate(0,${y(0)})`).call(xAxis);
      yAxisG.attr("transform", `translate(${x(0)},0)`).call(yAxis);

      root.attr(
        "transform",
        `translate(${state.margin.left},${state.margin.top})`
      );
      const line = root.append("g");

      const dots = line.selectAll("circle");
      dots
        .data(data)
        .enter()
        .append("circle")
        .attr("stroke", "white")
        .attr("stroke-width", "0.5")
        .attr("r", ([xValue]: any) => radiusByValue(xValue))
        .attr("fill", state.color)
        .attr("cx", ([xValue]: any) => x(+xValue))
        .attr("cy", ([_, yValue]: any) => y(+yValue));
    };

    onMounted(() => {
      draw(randomData.value, rootSvg());
    });
    watch(
      [() => state.activeFormula, () => state.radius, () => state.color],
      (newValues, oldValues) => {
        const [newFormula, newRadius, newColor] = newValues;
        const [oldFormula, oldRadius, oldColor] = oldValues;
        if (newFormula && newFormula !== oldFormula) {
          draw(randomData.value, rootSvg());
        } else if (newRadius && newRadius !== oldRadius) {
          dots().attr("r", ([x]: any) => radiusByValue(x));
        } else if (newColor && newColor !== oldColor) {
          dots().attr("fill", newColor);
        }
      },
      { lazy: true }
    );

    return { ...toRefs(state) };
  }
});
export default BarChart;
</script>

<style scoped></style>