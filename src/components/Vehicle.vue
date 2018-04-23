<template>
  <text :x="xPosition" :y="yPosition" :route="vehicle.properties.routeTag" :style="styleObject" :class="Number.parseInt(vehicle.properties.routeTag) ? 'bus' : 'trolley'">{{Number.parseInt(vehicle.properties.routeTag) ? '🚌' : '🚋'}}</text>
</template>

<script lang="ts">
import Vue from "vue"
import {TweenMax, Power1} from 'gsap'

export default Vue.extend({
  name: "MapContainer",
  props: ['checkedValues', 'vehicle', 'projection'],
  data () {
    return {
      xPosition: <Number>0,
      yPosition: <Number>0
    }
  },
  created () {
    this.xPosition = this.xComputed
    this.yPosition = this.yComputed
  },
  watch: {
    xComputed (newVal, oldVal) {
      const update = function () {
        let tweenedVal = (<any>Object).values(tweenFrom)
        tweenedVal.pop()
        this.xPosition = tweenedVal[0]
      }
      let tweenFrom = {}
      let tweenTo = {onUpdate: update, ease: Power1.easeOut, onUpdateScope: this}
      tweenFrom[0] = oldVal
      tweenTo[0] = newVal
      TweenMax.to(tweenFrom, 2, tweenTo)
    },
    yComputed (newVal, oldVal) {
      const update = function () {
        let tweenedVal = (<any>Object).values(tweenFrom)
        tweenedVal.pop()
        this.yPosition = tweenedVal[0]
      }
      let tweenFrom = {}
      let tweenTo = {onUpdate: update, ease: Power1.easeOut, onUpdateScope: this}
      tweenFrom[0] = oldVal
      tweenTo[0] = newVal
      TweenMax.to(tweenFrom, 2, tweenTo)
    }
  },
  computed: {
    styleObject ():object {
      return {
        display: this.checkedValues.length === 0 || this.checkedValues.includes(this.vehicle.properties.routeTag) ? 'block' : 'none'
      }
    },
    xComputed ():Number {
      return this.projection(this.vehicle.coordinates)[0]
    },
    yComputed ():Number {
      return this.projection(this.vehicle.coordinates)[1]
    }
  }
})
</script>

<style>

</style>
