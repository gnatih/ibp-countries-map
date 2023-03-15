<script setup>
import * as d3 from "d3";
import * as topojson from "topojson";
import { onMounted } from "vue";
import _ from "lodash";

onMounted(() => {
  Promise.all([
    d3.json("countries-50m.json"),
    d3.json(
      "https://opensheet.elk.sh/15Fhb7nWSG0WlKzlD96Qy8VfjHuZPa4P0AVIKJqALPtM/countries_db"
    ),
  ]).then(([c, d]) => {
    // console.log(c.objects.countries, d);

    _.remove(
      c.objects.countries.geometries,
      (c3) => c3.properties.name == "Antarctica"
    );

    const width = document.body.clientWidth;
    const height = document.body.clientHeight;
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
      .data(topojson.feature(c, c.objects.countries).features)
      .enter()
      .append("path")
      .attr("d", geoGenerator)
      .attr("fill", function (country) {
        let c = _.find(d, { Country: country.properties.name });
        return !_.isUndefined(c) ? colors[c.Type] : "#ccc";
      })
      .attr("class", (country) => _.kebabCase(country.properties.name))
      .attr("stroke-width", 1)
      .attr("stroke", "#fff");

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

  // const x = d3
  //   .scaleTime()
  //   .domain(
  //     d3.extent(data, function (d) {
  //       return parseTime(d.date);
  //     })
  //   )
  //   .rangeRound([0, width]);
  // const y = d3
  //   .scaleLinear()
  //   .domain(
  //     d3.extent(data, function (d) {
  //       return d.amount;
  //     })
  //   )
  //   .rangeRound([height, 0]);
  // const line = d3
  //   .line()
  //   .x(function (d) {
  //     return x(parseTime(d.date));
  //   })
  //   .y(function (d) {
  //     return y(d.amount);
  //   });

  // g.append("path")
  //   .datum(data)
  //   .attr("fill", "none")
  //   .attr("stroke", "steelblue")
  //   .attr("stroke-width", 1.5)
  //   .attr("d", line);
});
</script>

<template>
  <svg></svg>
</template>

<style scoped>
svg {
  outline: 1px solid #ccc;
}
</style>
