<template>
  <v-card>
    <svg id="mapid"/>
  </v-card>
</template>

<script>
  /* eslint-disable */
  import * as d3 from 'd3';
  import {feature, mesh} from 'topojson';
  export default {
    data() {
        return {
            map: null,
            tileLayer: null,
            layers: [],
            width: 960,
            height: 500,
        }
    },
    mounted() {
        this.initMap()
    },
    methods: {
      initMap() {
        // this.map = L.map('mapid').setView([37.8, -96], 4);
        // this.tileLayer = L.tileLayer('https://api.tiles.mapbox.com/v4/{id}/{z}/{x}/{y}.png?access_token={accessToken}', {
        //   attribution: 'Map data &copy; <a href="https://www.openstreetmap.org/">OpenStreetMap</a> contributors, <a href="https://creativecommons.org/licenses/by-sa/2.0/">CC-BY-SA</a>, Imagery © <a href="https://www.mapbox.com/">Mapbox</a>',
        //   id: 'mapbox.light',
        //   accessToken: 'pk.eyJ1IjoiYWR2YWl0aHYxIiwiYSI6ImNqb3JoYmlkczBkeGMzcnNjMWNxazU0ejQifQ.BC2xnuKO8ZxdMt-4kJkw6g'
        // })
        // this.tileLayer.addTo(this.map);
        let svg = d3.select('#mapid')
          .attr('width', 960)
          .attr('height', 600)
        let path = d3.geoPath();
        const url = "https://d3js.org/us-10m.v1.json"
        
        d3.json(url).then((data) => {
          svg.append('g')
            .attr('class', 'states')
            .selectAll('path')
            .data(feature(data, data.objects.states).features)
            .enter().append('path')
              .attr('d', path)

          svg.append('path')
            .attr('class', 'state-borders')
            .attr('d', path(mesh(data, data.objects.states, (a,b) => {return a != b})))
        })
      }
    }
  }
</script>

<style>
.states :hover {
  fill: red;
}

.state-borders {
  fill: none;
  stroke: #fff;
  stroke-width: 0.5px;
  stroke-linejoin: round;
  stroke-linecap: round;
  pointer-events: none;
}
</style>