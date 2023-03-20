<script setup>
import * as d3 from "d3";
import { feature } from "topojson";
import { onMounted, ref } from "vue";
import _ from "lodash";

let mode = ref("map");
let obs_countries = ref(0);
let total_countries = ref(0);

let setMode = function (str) {
  mode.value = str;
};

onMounted(() => {
  Promise.all([
    d3.json("countries-50m.json"),
    d3.json(
      "https://opensheet.elk.sh/15Fhb7nWSG0WlKzlD96Qy8VfjHuZPa4P0AVIKJqALPtM/countries_db"
    ),
  ]).then(([c, d]) => {
    _.remove(c.objects.countries.geometries, (c3) =>
      ["Antarctica"].includes(c3.properties.name)
    );

    const width = document.body.clientWidth;
    const height = document.body.clientHeight;
    const tooltip = d3.select("#app").append("div").attr("class", "tooltip");
    let svg = d3.select("svg");
    let g = svg.append("g");
    let projection = d3
      .geoNaturalEarth1()
      .translate([width / 2, height / 2])
      .scale((width - 1) / 2 / Math.PI);
    let geoGenerator = d3.geoPath().projection(projection);

    let colors = {
      "OBS Country": "#52C3C9",
      "Multiple Projects": "#0083A9",
      "Country Office": "#034A8A",
    };

    svg = d3.select("svg").attr("width", width).attr("height", height);
    g.selectAll("path")
      .data(feature(c, c.objects.countries).features)
      .enter()
      .append("path")
      .attr("d", geoGenerator)
      .attr("fill", function (country) {
        total_countries.value++;
        let c = _.find(d, { Country: country.properties.name });

        if (!_.isUndefined(c)) {
          if (c.Type == "OBS Country") obs_countries.value++;
          return colors[c.Type];
        }

        return "#ccc";
      })
      .attr("class", (country) =>
        [_.kebabCase(country.properties.name), "country"].join(" ")
      )
      .attr("stroke-width", 1)
      .attr("stroke", "#fff")
      .on("mouseover", (e) => d3.select(this).style("fill", "red"))
      .on("click", (e, el) => {
        tooltip
          .html(`${el.properties.name}`)
          .style("left", e.pageX + "px")
          .style("top", e.pageY + "px")
          .style("opacity", 1);
      });

    let zoom = d3
      .zoom()
      .scaleExtent([1, 8])
      .on("zoom", (e) => {
        g.selectAll("path")
          .attr("transform", e.transform)
          .attr("stroke-width", 1 / e.transform.k);
      });

    svg.call(zoom);
  });
});
</script>

<template>
  <div class="tab">
    <span @click="setMode('map')">Map</span
    ><span @click="setMode('list')">List</span>
  </div>

  <svg></svg>
</template>

<style lang="scss">
.tooltip {
  position: absolute;
  padding: 2px;
  height: auto;
  background: lightsteelblue;
  border: 0px;
  pointer-events: none;
  display: flex;
  justify-content: center;
  align-items: center;
}

.tab {
  display: flex;

  span {
    cursor: pointer;
    background: white;
    color: green;
    padding: 20px;
  }
}
</style>
