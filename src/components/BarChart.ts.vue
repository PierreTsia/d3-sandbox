<template>
  <v-container class="d-flex flex-column justify-center align-center">
    <v-row>
      <v-col class="col-12 col-md-8 d-flex align-center justify-center">
        <v-card class="pa-6">
          <v-card-title class="text-center d-block">
            {{ title }}
          </v-card-title>
          <v-card-subtitle class="text-center">
            {{ subtitle }}
          </v-card-subtitle>
          <svg
            :id="id"
            :height="outerHeight"
            :width="outerWidth"
            class="d-block"
          />
        </v-card>
      </v-col>
      <v-col class="col-12 col-md-4 d-flex align-center justify-center">
        <v-card class="light-blue pa-2 px-6">
          <v-row>
            <v-col class="col-12 align-center justify-center">
              <v-select
                dark
                hide-details
                :items="formulas"
                outlined
                v-model="activeFormula"
              ></v-select>
            </v-col>
            <v-col class="col-12 d-flex align-center justify-center">
              <v-radio-group
                class="d-flex justify-center"
                v-model="radiusType"
                row
                dark
              >
                <v-radio
                  v-for="(type, index) in radiusTypes"
                  :key="index"
                  :label="type"
                  :value="type"
                ></v-radio>
              </v-radio-group>
            </v-col>
            <v-col class="col-12 align-center justify-center">
              <v-slider
                class="px-4"
                dark
                v-model="radius"
                min="1"
                max="10"
                label="Size"
              ></v-slider>
            </v-col>
            <v-col class="col-12">
              <v-color-picker
                dark
                class="ma-2"
                hide-canvas
                hide-inputs
                v-model="color"
                @update:color="colorObj => (color = colorObj.hex)"
              ></v-color-picker>
            </v-col>
          </v-row>
        </v-card>
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
    id: {},
    name: {
      default: "Default title"
    },
    width: {
      default: 500
    },
    height: {
      default: 300
    },
    dataSet: {
      default: "sine"
    },
    defaultColor: {
      default: "#195693"
    },
    defaultRadiusType: {
      default: "fixed"
    }
  },
  setup(props) {
    const state = reactive({
      title: props.name,
      radiusType: props.defaultRadiusType,
      subtitle: `${props.dataSet} line chart`,
      outerWidth: props.width,
      outerHeight: props.height,
      margin: { left: 30, top: 30, right: 30, bottom: 30 },
      color: props.defaultColor,
      radius: 5,
      formulas: ["sine", "parabola", "hilly"],
      activeFormula: props.dataSet
    });
    const radiusTypes = ["fixed", "scaled"];
    const handleInputColor = (color: any) => {
      state.color = color.hex;
    };
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

    const scaleElement = (el: any, left: number, top: number) =>
      el.attr("transform", `translate(${left},${top})`);

    const radiusByValue = (value: number) =>
      state.radiusType === "scaled"
        ? Math.abs((state.radius * +value * 20) / 100)
        : state.radius;

    const minmax = (data: number[]) =>
      [d3.min(data) ?? 0, d3.max(data) ?? 100] as [number, number];

    const axisValues = (data: number[], type = "x") =>
      data.map(([valueX, valueY]: any) => (type === "x" ? valueX : valueY));

    const draw = async (data: any, parent: any) => {
      parent.selectAll("g").remove();
      const root = parent.append("g");
      scaleElement(root, state.margin.left, state.margin.top);

      const [x, y] = [d3.scaleLinear(), d3.scaleLinear()];
      const xAxis = d3.axisBottom(x).ticks(5);
      const yAxis = d3.axisLeft(y).ticks(5);
      const xAxisG = root.append("g");
      const yAxisG = root.append("g");
      const xMinMax = minmax(axisValues(data, "x"));
      const yMinMax = minmax(axisValues(data, "y"));
      x.domain(xMinMax).range([0, innerWidth.value]);
      y.domain(yMinMax).range([innerHeight.value, 0]);
      xAxisG.attr("transform", `translate(0,${y(0)})`).call(xAxis);
      yAxisG.attr("transform", `translate(${x(0)},0)`).call(yAxis);

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
        .attr("cy", y(0))
        .transition()
        .duration(1000)
        .ease(d3.easeCubicIn, 1)
        .attr("cx", ([xValue]: any) => x(+xValue))
        .attr("cy", ([_, yValue]: any) => y(+yValue));
    };

    onMounted(() => {
      draw(randomData.value, rootSvg());
    });
    const watchedProperties = [
      () => state.activeFormula,
      () => state.radius,
      () => state.color,
      () => state.radiusType
    ];
    watch(
      watchedProperties,
      (newValues, oldValues) => {
        const [newFormula, newRadius, newColor, newRadiusType] = newValues;
        const [oldFormula, oldRadius, oldColor, oldRadiusType] = oldValues;
        if (newFormula !== oldFormula) {
          draw(randomData.value, rootSvg());
        } else if (newRadius !== oldRadius || newRadiusType !== oldRadiusType) {
          dots().attr("r", ([x]: any) => radiusByValue(x));
        } else if (newColor !== oldColor) {
          dots().attr("fill", newColor);
        }
      },
      { lazy: true }
    );

    return { ...toRefs(state), handleInputColor, radiusTypes };
  }
});
export default BarChart;
</script>

<style scoped lang="stylus"></style>
