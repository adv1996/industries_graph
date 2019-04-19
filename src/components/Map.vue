<template>
  <div>
    <!-- <Legend/> -->
    <svg :id="category"/>
  </div>
</template>

<script>
  /* eslint-disable */
  import _ from 'lodash';
  import * as d3 from 'd3';
  import {feature, mesh} from 'topojson';
  import IndustryData from '../data/naics.json';
  import MapData from '../data/us-10m.v1.json';

  export default {
    props: ['data', 'category'],
    data() {
      return {
        width: null,
        height: null,
        color: null,
        dataMap: null
      }
    },
    mounted() {
      this.setDimensions()
      this.setData()
      this.initMap()
    },
    methods: {
      setDimensions() {
        this.width = this.$el.offsetWidth
        this.height = 350
      },
      setData() {
        let density = []
        this.dataMap = _.keyBy(_.map(this.data, (state) => {
          let population = parseFloat(state[21].replace(/,/g, ''))
          let jobs = parseFloat(state[5])
          let value = (jobs / population) * 100
          density.push(value)
          return {
            id: state[0],
            state: state[1],
            population: population,
            jobs: jobs,
            density: value,
          }
        }), 'id')
        let maxMin = d3.extent(density)
        this.min = maxMin[0]
        this.max = maxMin[1]
      },
      initMap() {
        this.color = d3.scaleQuantize([this.min, this.max], ['#e0ecf4','#bfd3e6','#9ebcda','#8c96c6','#8c6bb1','#88419d','#810f7c','#4d004b'])
        let svg = d3.select('#' + this.category)
          .attr('width', this.width)
          .attr('height', this.height)
          .attr("viewBox", "0 0 " + this.width + " " + this.height )
          .attr("preserveAspectRatio", "xMinYMin");
        
        const g = svg.append('g')
          .attr("transform", "translate(" + 0 + "," + -40 + ")")
        
        let featureCollection = feature(MapData, MapData.objects.states);
        let projection = d3.geoIdentity()
          .fitExtent([[25, 25],[this.width - 25,this.height - 25]], featureCollection)

        let path = d3.geoPath().projection(projection)
        g.append('g')
          .attr('class', 'states')
          .selectAll('path')
          .data(feature(MapData, MapData.objects.states).features)
          .enter().append('path')
            .attr('d', path)
            .attr('fill', (d) => {
              if (this.dataMap[d.id]) {
                return this.color(this.dataMap[d.id].density)
              } else {
                return 'white'
              }
            })
            .attr('class', 'state-areas')

        g.append('path')
          .attr('class', 'state-borders')
          .attr('d', path(mesh(MapData, MapData.objects.states, (a,b) => {return a != b})))
      }
    }
  }
</script>

<style>
.state-borders {
  fill: none;
  stroke: #fff;
  stroke-width: 0.5px;
  stroke-linejoin: round;
  stroke-linecap: round;
  pointer-events: none;
}
</style>