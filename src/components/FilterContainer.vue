<template>
  <div id="filterContainer">
    <div id="filterWrapper">
      <button @click="resetChecked">Reset</button>
      <span>Filter routes</span>
      <template v-for="(route, index) in allRoutes">
        <div class="checkboxContainer" :key="route">
          <input type="checkbox" :id="idString(index)" :value="route" v-model="checkedValues"/>
          <label :for="idString(index)">{{route}}</label>
        </div>
      </template>
    </div>
  </div>
</template>

<script lang="ts">
import Vue from "vue"

export default Vue.extend({
  name: "FilterContainer",
  props: ['allRoutes'],
  data () {
    return {
      checkedValues: []
    }
  },
  methods: {
    idString (id:number) {
      return `route-${id}`
    },
    resetChecked () {
      this.checkedValues = []
      Array.from(document.querySelectorAll('input[type=checkbox]')).forEach((element:any) => {
        element.checked = false
      });
    }
  },
  watch: {
    checkedValues (newVal, oldVal) {
      this.$emit('change', newVal)
    }
  }
})
</script>

<style scoped>
#filterContainer{
  position: absolute;
  top: 0px;
  left: 0px;
  width: 20%;
  height: 100%;
  background: black;
  overflow: scroll;
}

#filterWrapper{
  padding: 10%;
  font-family: 'Alfa Slab One', cursive;
  margin: auto;
}

button {
  background: none;
  margin-bottom: 20px;
  display: block;
  color: white;
  font-size: 22px;
  border: 1px solid white;
  font-family: 'Alfa Slab One', cursive;
}
button:hover{
  color:black;
  background: white;
  cursor: pointer;
}

span{
  color:white;
  font-size: 22px;
}

label{
  color: white;
  vertical-align: middle;
  position: relative;
  margin-left: 30px;
}
label::before{
  content: "";
  display: inline-block;
  height: 16px;
  width: 16px;
  border: 1px solid;   
  position: absolute;
  top: 3px;
  left: -30px;
}
label::after {
  display: inline-block;
  height: 6px;
  width: 9px;
  border-left: 2px solid;
  border-bottom: 2px solid;
  transform: rotate(-45deg);
  position: absolute;
  left: -26px;
  top: 7px;
}
input[type=checkbox]{
  opacity: 0;
  width: 0px;
}

.checkboxContainer input[type="checkbox"] + label::after {
    content: none;
}
.checkboxContainer input[type="checkbox"]:checked + label::after {
    content: "";
}

.checkboxContainer input[type="checkbox"]:focus + label::before {
    outline: rgb(59, 153, 252) auto 5px;
}

.checkboxContainer{
  margin: 10px 0px;
}
</style>
