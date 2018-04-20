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
import Vue from "vue"
import { geoPath, geoMercator } from "d3-geo"
import { setInterval } from "timers";

export default Vue.extend({
  name: "MapContainer",
  data() {
    return {
      geojsonData: {},
      busData: []
    }
  },
  async mounted() {
    const geojsonFile = await fetch("../streets.json")
    this.geojsonData = await geojsonFile.json()
    this.getBusData(true)
    window.setInterval(this.getBusData,1000)
  },
  methods: {
    async getBusData(firstRun:boolean = false) {
      const busRequest = await fetch(`http://webservices.nextbus.com/service/publicJSONFeed?command=vehicleLocations&a=sf-muni&t=${firstRun ? 0 : Date.now() - 15000}`)
      const busJSON = await busRequest.json()
      const currentIds = this.busData.map(d=>d.properties.id)
      for (let i = 0; i < busJSON.vehicle.length; i++) {
        if(!currentIds.includes(busJSON.vehicle[i].id)) {
          this.busData.push(this.getBusObject(busJSON.vehicle[i]))
        } else {
          const updatingIndex = this.busData.findIndex((d) => {
            return d.properties.id === busJSON.vehicle[i].id
          })
          this.updateBus(busJSON.vehicle[i], updatingIndex)
        }
      }
    },
    getBusObject (bus:object) {
      return {
        coordinates: [bus.lon, bus.lat],
        properties: {
          id: bus.id,
          routeTag: bus.routeTag,
          secsSinceReport: bus.secsSinceReport,
          speedKmHr: bus.speedKmHr,
          heading: bus.heading
        }
      }
    },
    updateBus (bus:object, updatingIndex:number) {
      Vue.set(this.busData[updatingIndex], 'coordinates', [bus.lon, bus.lat])
      Vue.set(this.busData[updatingIndex], 'properties.secsSinceReport', bus.secsSinceReport)
      Vue.set(this.busData[updatingIndex], 'properties.speedKmHr', bus.speedKmHr)
      Vue.set(this.busData[updatingIndex], 'properties.heading', bus.heading)
    }
  },
  computed: {
    pathFunction() {
      return geoPath(this.projection)
    },
    projection () {
      return geoMercator()
        .scale(306337.6140209504)
        .center([-122.43595722806097, 37.77071992617303]) //projection center
        .translate([450, 450])
    }
  }
})
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
