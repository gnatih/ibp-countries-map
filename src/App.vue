<script setup>
import * as d3 from 'd3'

import { onMounted } from "vue";


const data = [
  { date: "24-Apr-07", amount: 93.24 },
  { date: "25-Apr-07", amount: 95.35 },
  { date: "26-Apr-07", amount: 98.84 },
  { date: "27-Apr-07", amount: 99.92 },
  { date: "30-Apr-07", amount: 99.8 },
  { date: "1-May-07", amount: 99.47 },
  { date: "2-May-07", amount: 100.39 },
  { date: "3-May-07", amount: 100.4 },
  { date: "4-May-07", amount: 100.81 },
  { date: "7-May-07", amount: 103.92 },
  { date: "8-May-07", amount: 105.06 },
  { date: "9-May-07", amount: 106.88 },
  { date: "10-May-07", amount: 107.34 },
];
const parseTime = d3.timeParse("%d-%b-%y");
const width = 800;
const height = 500;
let projection = d3.geoEquirectangular();
let geoGenerator = d3.geoPath()
  .projection(projection);

onMounted(() => {

  d3.json('world-110m.json').then(countries => {
    console.log(countries)
  })


  const svg = d3.select("svg").attr("width", width).attr("height", height);
  const g = svg.append("g");
  const x = d3
    .scaleTime()
    .domain(
      d3.extent(data, function (d) {
        return parseTime(d.date);
      })
    )
    .rangeRound([0, width]);
  const y = d3
    .scaleLinear()
    .domain(
      d3.extent(data, function (d) {
        return d.amount;
      })
    )
    .rangeRound([height, 0]);
  const line = d3
    .line()
    .x(function (d) {
      return x(parseTime(d.date));
    })
    .y(function (d) {
      return y(d.amount);
    });

  g.append("path")
    .datum(data)
    .attr("fill", "none")
    .attr("stroke", "steelblue")
    .attr("stroke-width", 1.5)
    .attr("d", line);
});

</script>

<template>
  <svg></svg>
</template>

<style scoped>
</style>
