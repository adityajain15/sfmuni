<template>
  <svg id="map">
    <g id="panZoomRoot">
      <g id="mapPaths"></g>
      <g :style="vehicleStyle">
        <template v-for="vehicle in vehicleData">
          <template v-if="checkedValues.length === 0 || checkedValues.includes(vehicle.properties.routeTag)">
            <text :x="projection(vehicle.coordinates)[0]" :y="projection(vehicle.coordinates)[1]" :route="vehicle.properties.routeTag" :class="Number.parseInt(vehicle.properties.routeTag) ? 'bus' : 'trolley'">{{Number.parseInt(vehicle.properties.routeTag) ? '🚌' : '🚋'}}</text>
          </template>
        </template>
      </g>
    </g>
  </svg>
</template>

<script lang="ts">
import Vue from "vue"
import { geoPath, geoMercator } from 'd3-geo'
import { scaleLog } from 'd3-scale'
import panzoom from 'panzoom'
const _debounce = require('lodash.debounce')
import ndjsonStream from 'can-ndjson-stream';

export default Vue.extend({
  name: "MapContainer",
  props: ['vehicleData', 'checkedValues'],
  data() {
    return {
      geojsonData: {},
      zoom: undefined,
      zoomScale: undefined
    }
  },
  async created () {
    // get the map file
    this.geojsonData.features = []
    this.geojsonData.type = "FeatureCollection"
    const geojsonFile = await fetch("./all.ndjson")
    const fileStream = await ndjsonStream(geojsonFile.body)
    const reader = fileStream.getReader();
    const mapPaths = document.getElementById('mapPaths')
    while(true){
      const result = await reader.read()
      if (result.done) { break }
      const newPath = document.createElementNS('http://www.w3.org/2000/svg','path')
      newPath.setAttributeNS(null, 'd', String(this.pathFunction(result.value)))
      newPath.setAttributeNS(null, 'type', result.value.geometry.type)
      mapPaths.appendChild(newPath)
    }
  },
  mounted() {
    // handle zooming
    this.zoom = panzoom(document.getElementById('panZoomRoot'), {
      maxZoom: 10,
      minZoom: 1,
      beforeWheel: _debounce(()=>{this.setZoomScale()}, 60)
    })
    this.setZoomScale()
    const map = document.getElementById('map')
    map.addEventListener('dblclick', this.setZoomScale)
    map.addEventListener('touchstart', _debounce(()=>{this.setZoomScale()}, 60))
    map.addEventListener('keydown', _debounce(()=>{this.setZoomScale()}, 60))
  },
  methods: {
    setZoomScale () {
      this.zoomScale = this.zoom.getTransform().scale
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
        .translate([this.$el.clientWidth / 2, this.$el.clientHeight / 2])
    },
    emojiSize () {
      return scaleLog().domain([1,10]).range([12,2])
    },
    vehicleStyle () {
      if (this.zoom === undefined) return '15px'
      return {
        'font-size': `${this.emojiSize(this.zoomScale)}px` 
      }
    }
  }
})
</script>

<style>
svg {
  height: 100%;
  width: 100%;
  box-sizing: border-box;
  background:rgb(70, 152, 199);
}
path {
  stroke-width: 0.3px;
  stroke: white;
  fill: none;
}
path[type=polygon], path[type=MultiPolygon]{
  fill: black;
  stroke: none;
}
</style>
