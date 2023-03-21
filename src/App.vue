<script setup>
import * as d3 from "d3";
import { feature } from "topojson-client";
import { onMounted, ref, computed, watch } from "vue";
import _ from "lodash";

let countries = ref(0);
let mode = ref("map");
let obs_countries = ref(0);
let selected_country = ref(null);
let sort = ref(null);
let setMode = str => {
  if (!sort.value) sort.value = 'high-to-low';
  mode.value = str
};

const tooltip_text = computed(() => {
  let country = _.find(countries.value, { Country: selected_country.value });
  let text = "";

  if (country) {
    if (country.URL) {
      text = `<a href="${country.URL}">${selected_country.value}</a><span class="${_.kebabCase(country.Type)} legend-circle"></span>`;
    } else {
      text = `${selected_country.value}<span class="${_.kebabCase(country.Type)} legend-circle"></span>`;
    }

    return text;
  } else {
    return selected_country.value;
  }
});

watch(sort, (val) => {
  countries.value = _.orderBy(countries.value, [val == 'alphabetical' ? 'Country' : 'Type'], [val == 'low-to-high' ? 'desc' : 'asc']);
})

onMounted(() => {
  Promise.all([d3.json("countries-50m.json"), d3.json("https://opensheet.elk.sh/15Fhb7nWSG0WlKzlD96Qy8VfjHuZPa4P0AVIKJqALPtM/countries_db")]).then(([c, d]) => {
    countries.value = d;
    _.remove(c.objects.countries.geometries, (c3) => ["Antarctica"].includes(c3.properties.name));

    const width = d3.select(".countries-map").node().getBoundingClientRect().width;
    const height = d3.select(".countries-map").node().getBoundingClientRect().height;
    let tooltip = d3.select("body").append("div").attr("class", "map-tooltip").style("opacity", 0);
    let svg = d3.select("svg");
    let g = svg.append("g");
    let projection = d3
      .geoNaturalEarth1()
      .translate([width / 2.25, height / 1.8])
      .scale(250);
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
        let c = _.find(d, { Country: country.properties.name });

        if (!_.isUndefined(c)) {
          if (c.Type == "OBS Country") obs_countries.value++;
          return colors[c.Type];
        }

        return "#ccc";
      })
      .attr("class", (country) => [_.kebabCase(country.properties.name), "country"].join(" "))
      .attr("stroke-width", 1)
      .attr("stroke", "#fff")
      .style("cursor", "pointer")
      .on("click", (e, el) => {
        selected_country.value = el.properties.name;

        tooltip
          .html(tooltip_text.value)
          .style("left", e.pageX + "px")
          .style("top", e.pageY + "px")
          .style("opacity", 1)
          .style("pointer-events", "auto");
      });

    let zoom = d3
      .zoom()
      .scaleExtent([1, 8])
      .on("zoom", (e) => {
        tooltip.style("opacity", 0).style("pointer-events", "none");
        g.selectAll("path")
          .attr("transform", e.transform)
          .attr("stroke-width", 1 / e.transform.k);
      });

    svg.call(zoom);

    d3.select("body").on("click", (e) => {
      if (e.target.tagName !== "path") {
        tooltip.style("opacity", 0).style("pointer-events", "none");
      }
    });
  });
});
</script>

<template>
  <div class="tab-bar">
    <div class="tab-buttons"><span @click="setMode('map')" :class="[mode == 'map' ? 'active' : '']">Map View</span><span @click="setMode('list')" :class="[mode == 'list' ? 'active' : '']">List View</span></div>
    <div class="sort-select" v-if="mode == 'list'">
      Sort by
      <select @change="$event => sort = $event.target.value">
        <option value="high-to-low" selected>IBP Engagement High to Low</option>
        <option value="low-to-high">IBP Engagement Low to High</option>
        <option value="alphabetical">Alphabetical</option>
      </select>
    </div>
  </div>

  <div class="countries-map" v-show="mode == 'map'">
    <svg></svg>
    <div class="legend-block">
      <h5>IBP Engagement</h5>
      <div class="map-categories">
        <div class="obs-country">Open Budget Survey Countries</div>
        <div class="multiple-projects">Countries with Multiple Projects</div>
        <div class="country-office">Country Office</div>
      </div>
    </div>
  </div>
  <div class="countries-list" v-show="mode == 'list'">
    <div v-for="country in countries">
      {{ `${country.Country}` }}
      <span :class="_.kebabCase(country.Type)" class="legend-circle"></span>
    </div>
  </div>
</template>

<style lang="scss">
.map-tooltip {
  position: absolute;
  text-align: center;
  padding: 10px 15px;
  font-family: 16px;
  background: white;
  display: flex;
  border: 1px solid var(--ibp-accent, #ff863a);
  align-items: center;

  a {
    color: inherit;
    text-decoration: none;
    border-bottom: 1px solid var(--ibp-accent, #ff863a);
  }

  .legend-circle {
    margin-left: 6px;
  }
}

.map-categories {
  font-size: 13px;
  display: flex;
  justify-content: stretch;
  align-items: stretch;

  &>div {
    border-top-width: 20px;
    border-bottom-width: 0;
    border-left-width: 0;
    border-right-width: 0;
    border-style: solid;
    background: none;
    flex: 1;
    padding-top: 8px;
  }
}

.legend-block {
  width: 50%;
  border: 1px solid #ccc;
  padding: 20px;
  position: absolute;
  bottom: 25px;
  left: 25px;
  background-color: white;

  h5 {
    margin: 0;
    margin-bottom: 20px;
  }
}

.legend-circle {
  width: 9px;
  height: 9px;
  display: inline-flex;
  border-radius: 500px;
  // margin-left: 6px;
}

.obs-country {
  background-color: var(--ibp-teal-light, #52c3c9);
  border-color: var(--ibp-teal-light, #52c3c9);
}

.multiple-projects {
  background-color: var(--ibp-teal, #0083a9);
  border-color: var(--ibp-teal, #0083a9);
}

.country-office {
  background-color: var(--ibp-teal-dark, #034a8a);
  border-color: var(--ibp-teal-dark, #034a8a);
}

.countries-list {
  // column-gap: 0;
  // columns: 4;
  display: grid;
  grid-template-columns: repeat(4, 1fr);
  padding: 25px;

  @media screen and (max-width: 767px) {
    padding: 15px;
    grid-template-columns: repeat(2, 1fr);
    // columns: 2;
  }

  &>div {
    padding-top: 20px;
    padding-bottom: 20px;
    border-bottom: 1px solid #ccc;
    // display: flex;
    align-items: center;
    font-size: 16px;
  }
}

.countries-map {
  min-height: 850px;
  position: relative;
}

// .tab-bar {
//   display: flex;
//   background: var(--ibp-teal-dark, #063d4f);

//   span {
//     background: var(--ibp-teal-dark, #063d4f);
//     cursor: pointer;
//     color: white;
//     padding: 14px 20px;
//     font-weight: 700;

//     &.active {
//       color: var(--ibp-body-text, black);
//       background: white;
//     }
//   }
// }

.tab-bar {
  background: var(--ibp-teal-dark, #063d4f);
  display: flex;
  justify-content: space-between;
  align-items: center;
  color: white;
  font-weight: 700;

  .tab-buttons {
    display: flex;

    span {
      padding: 14px 20px;

      &.active {
        color: var(--ibp-body-text, black);
        background: white;
      }
    }
  }

  .sort-select {
    font-size: 13px;

    select {
      margin-left: 8px;
      padding: 8px 14px;
    }
  }
}
</style>
