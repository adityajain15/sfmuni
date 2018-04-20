<template>
  <svg height="900" width="900">
    <g id="mapPaths">
      <template v-for="feature in geojsonData.features">
        <path :d="pathFunction(feature)" :class="feature.type"/>
      </template>
    </g>
    <template v-for="bus in busData">
      <text :x="projection(bus.coordinates)[0]" :y="projection(bus.coordinates)[1]">🚌</text>
    </template>
  </svg>
</template>

<script lang="ts">
import Vue from "vue";
import { geoPath, geoMercator } from "d3-geo";

export default Vue.extend({
  name: "MapContainer",
  data() {
    return {
      geojsonData: {},
      busData: []
    };
  },
  async mounted() {
    const geojsonFile = await fetch("../streets.json");
    this.geojsonData = await geojsonFile.json();
    this.getBusData();
  },
  methods: {
    async getBusData() {
      const busRequest = await fetch(
        `http://webservices.nextbus.com/service/publicJSONFeed?command=vehicleLocations&a=sf-muni&t=${Date.now() -
          15000}`
      );
      const busJSON = await busRequest.json();
      const currentIds = this.busData.map(d=>d.properties.id)
      for (let i = 0; i < busJSON.vehicle.length; i++) {
        console.log(busJSON.vehicle[i].id)
        if(!currentIds.includes(busJSON.vehicle[i].id)) {
          this.busData.push({
            coordinates: [busJSON.vehicle[i].lon, busJSON.vehicle[i].lat],
            properties: {
              id: busJSON.vehicle[i].id,
              routeTag: busJSON.vehicle[i].routeTag,
              secsSinceReport: busJSON.vehicle[i].secsSinceReport,
              speedKmHr: busJSON.vehicle[i].speedKmHr,
              heading: busJSON.vehicle[i].heading
            }
          })
        }
      }
    }
  },
  computed: {
    pathFunction() {
      return geoPath(this.projection);
    },
    projection () {
      return geoMercator()
        .scale(306337.6140209504)
        .center([-122.43595722806097, 37.77071992617303]) //projection center
        .translate([450, 450]);
    }
  }
});
</script>

<style scoped>
span {
  display: block;
}
path {
  stroke-width: 0.3px;
  stroke: black;
  fill: none;
}
</style>
