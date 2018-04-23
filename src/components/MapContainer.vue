<template>
  <svg id="map">
    <g id="panZoomRoot">
      <g id="mapPaths"></g>
      <g id="vehicleRoot" :style="vehicleStyle">
        <template v-for="vehicle in vehicleData">
          <Vehicle :key="vehicle.properties.id" :checkedValues="checkedValues" :vehicle="vehicle" :projection="projection"/>
        </template>
      </g>
    </g>
  </svg>
</template>

<script lang="ts">
import Vue from "vue"
import Vehicle from './Vehicle.vue'
import { geoPath, geoMercator } from 'd3-geo'
import { scaleLog } from 'd3-scale'
import panzoom from 'panzoom'
const _debounce = require('lodash.debounce')
import ndjsonStream from 'can-ndjson-stream'

export default Vue.extend({
  name: "MapContainer",
  props: ['vehicleData', 'checkedValues'],
  components: { Vehicle },
  data() {
    return {
      geojsonData: <any> {},
      zoom: <any> undefined,
      zoomScale: <any> undefined 
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
    
    let result = await reader.read()
    while(!result.done) {
      const newPath = document.createElementNS('http://www.w3.org/2000/svg','path')
      newPath.setAttributeNS(null, 'd', String(this.pathFunction(result.value)))
      newPath.setAttributeNS(null, 'type', result.value.geometry.type)
      if (mapPaths) {
        mapPaths.appendChild(newPath)
      }
      result = await reader.read()
    } 
  },
  mounted() {
    // handle zooming
    const zoomRoot = document.getElementById('panZoomRoot')
    if(zoomRoot) {
      this.zoom = panzoom(zoomRoot, {
        maxZoom: 10,
        minZoom: 1,
        beforeWheel: _debounce(()=>{this.setZoomScale()}, 60)
      })
      this.setZoomScale()
    }
    
    const map = document.getElementById('map')
    if (map) { 
      map.addEventListener('dblclick', this.setZoomScale)
      map.addEventListener('touchstart', _debounce(()=>{this.setZoomScale()}, 60))
      map.addEventListener('keydown', _debounce(()=>{this.setZoomScale()}, 60))
    } 
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
    projection ():Number {
      return geoMercator()
        .scale(306337.6140209504)
        .center([-122.43595722806097, 37.77071992617303]) //projection center
        .translate([this.$el.clientWidth / 2, this.$el.clientHeight / 2])
    },
    emojiSize () {
      return scaleLog().domain([1,10]).range([12,2])
    },
    vehicleStyle ():Object {
      if (this.zoom === undefined) return { 'font-size': '15px' }
      return { 'font-size': `${this.emojiSize(this.zoomScale)}px` }
    }
  }
})
</script>

<style>
svg {
  height: 100%;
  width: 100%;
  box-sizing: border-box;
  background: #a7cdf2;
}
#vehicleRoot {
  cursor: pointer;
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
