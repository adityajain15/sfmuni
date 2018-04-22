<template>
  <div id="app">
    <MapContainer :vehicleData="vehicleData" :checkedValues="checkedValues"/>
    <FilterContainer :allRoutes="allRoutes" @change="val => this.checkedValues = val"/>
  </div>
</template>

<script lang="ts">
import Vue from 'vue'
import MapContainer from './components/MapContainer.vue'
import FilterContainer from './components/FilterContainer.vue'

export default Vue.extend({
  name: 'app',
  components: {
    MapContainer,
    FilterContainer
  },
  data () {
    return {
      checkedValues: [],
      vehicleData: []
    }
  },
  async created () {
    this.getvehicleData(true)
    // schedule vehicle data fetching every 3 seconds
    window.setInterval(this.getvehicleData, 3000)
  },
  computed: {
    allRoutes () {
      const allRoutes:String[] = []
      for (let i = 0; i < this.vehicleData.length; i++) {
        if(!allRoutes.includes(this.vehicleData[i].properties.routeTag)){
          allRoutes.push(this.vehicleData[i].properties.routeTag)
        }
      }
      return allRoutes.sort()
    }
  },
  methods: {
    async getvehicleData(firstRun:boolean = false) {
      const vehicleRequest = await fetch(`https://cors-anywhere.herokuapp.com/http://webservices.nextbus.com/service/publicJSONFeed?command=vehicleLocations&a=sf-muni&t=${firstRun ? 0 : Date.now() - 15000}`)
      const vehicleJSON = await vehicleRequest.json()
      const currentIds = this.vehicleData.map(d=>d.properties.id)
      for (let i = 0; i < vehicleJSON.vehicle.length; i++) {
        if(!currentIds.includes(vehicleJSON.vehicle[i].id)) {
          this.vehicleData.push(this.getvehicleObject(vehicleJSON.vehicle[i]))
        } else {
          const updatingIndex = this.vehicleData.findIndex((d) => {
            return d.properties.id === vehicleJSON.vehicle[i].id
          })
          this.updatevehicle(vehicleJSON.vehicle[i], updatingIndex)
        }
      }
    },
    getvehicleObject (vehicle:object) {
      return {
        coordinates: [vehicle.lon, vehicle.lat],
        properties: {
          id: vehicle.id,
          routeTag: vehicle.routeTag,
          secsSinceReport: vehicle.secsSinceReport,
          speedKmHr: vehicle.speedKmHr,
          heading: vehicle.heading
        }
      }
    },
    updatevehicle (vehicle:object, updatingIndex:number) {
      Vue.set(this.vehicleData[updatingIndex], 'coordinates', [vehicle.lon, vehicle.lat])
      Vue.set(this.vehicleData[updatingIndex].properties, 'secsSinceReport', vehicle.secsSinceReport)
      Vue.set(this.vehicleData[updatingIndex].properties, 'speedKmHr', vehicle.speedKmHr)
      Vue.set(this.vehicleData[updatingIndex].properties, 'heading', vehicle.heading)
    }
  }
});
</script>

<style>
@import url('https://fonts.googleapis.com/css?family=Alfa+Slab+One');
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  width: 100vw;
  height: 100vh;
}
</style>
