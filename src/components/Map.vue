<template>
  <div>
    <Legend v-if="width" :data="density" :category="legendName" :colors="colors" :extent="extent" :jobs="getJobs(data)" :width="width"/>
    <svg :id="category"/>
    <BoxPlot v-if="width" :category="boxName" :data="data" :width="width"/>
  </div>
</template>

<script>
  /* eslint-disable */
  import _ from 'lodash';
  import * as d3 from 'd3';
  import {feature, mesh} from 'topojson';
  import IndustryData from '../data/naics.json';
  import MapData from '../data/us-10m.v1.json';
  import Legend from './Legend';
  import BoxPlot from './BoxPlot';

  export default {
    props: ['data', 'category'],
    components: {
      Legend,
      BoxPlot,
    },
    data() {
      return {
        width: null,
        height: null,
        color: null,
        dataMap: null,
        density: null,
        colors: ['#bfd3e6','#9ebcda','#8c96c6','#8c6bb1','#88419d','#810f7c','#4d004b'],
        extent: [],
      }
    },
    computed: {
      legendName() {
        return this.category + 'Legend'
      },
      boxName() {
        return this.category + 'Box'
      },
    },
    created() {
      this.setData()
    },
    mounted() {
      this.setTooltip()
      this.setDimensions()
      this.initMap()
    },
    methods: {
      setTooltip() {
        let tooltip = d3.select('body')
          .append('div')
          .attr('class', 'tooltip')	
          .style('opacity', 0)
      },
      getJobs(jobs) {
        return _.sum(_.map(jobs, (j) => {
          if (j[5]) {
            return parseInt(j[5])
          }
        }))
      },
      setDimensions() {
        this.width = this.$el.offsetWidth
        this.height = 260
        // d3.select('#' + this.category + 'Box')
        //   .attr('width', this.width)
      },
      setData() {
        this.density = []
        this.dataMap = _.keyBy(_.map(this.data, (state) => {
          let population = parseFloat(state[21].replace(/,/g, ''))
          let jobs = parseFloat(state[5])
          let value = (jobs / population) * 100
          this.density.push(value)
          return {
            id: state[0],
            state: state[1],
            population: population,
            jobs: jobs,
            density: value,
            median: parseFloat(state[18])
          }
        }), 'id')
        this.extent = d3.extent(this.density)
        this.min = this.extent[0]
        this.max = this.extent[1]
      },
      initMap() {
        this.color = d3.scaleQuantize([this.min, this.max], this.colors)
        let svg = d3.select('#' + this.category)
          .attr('width', this.width)
          .attr('height', this.height)
          .attr("viewBox", "0 0 " + this.width + " " + this.height )
          .attr("preserveAspectRatio", "xMinYMin");
        
        const g = svg.append('g')
          .attr("transform", "translate(" + 0 + "," + 0 + ")")
        
        let featureCollection = feature(MapData, MapData.objects.states);
        let projection = d3.geoIdentity()
          .fitExtent([[25, 25],[this.width - 25,this.height - 25]], featureCollection)

        let path = d3.geoPath().projection(projection)
        let tooltip = d3.select('.tooltip')
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
            .on('mouseover', (d) => {
              tooltip
                .style("left", (d3.event.pageX + 10) + "px")	
                .style("top", (d3.event.pageY - 10) + "px")
                .style('opacity', 1)
                .style('height', 50)
                .html(
                  this.dataMap[d.id].state + '<br/>' +
                  this.dataMap[d.id].jobs.toString().replace(/\B(?=(\d{3})+(?!\d))/g, ",") + ' Jobs' + '<br/>' +
                  '$' + this.dataMap[d.id].median.toFixed(0).toString().replace(/\B(?=(\d{3})+(?!\d))/g, ",")
                )
            })
            .on('mouseout', () => {
              tooltip
                .style('opacity', 0)
            })

        g.append('path')
          .attr('class', 'state-borders')
          .attr('d', path(mesh(MapData, MapData.objects.states, (a,b) => {return a != b})))
      }
    }
  }
</script>

<style>
div.tooltip {	
  position: absolute;
  text-align: center;
  width: 90px;			
  padding: 2px;				
  font-size: 12px;		
  font-family: "Helvetica Neue", Helvetica, sans-serif;
  color: black;
  background:white;	
  border: 0px;		
  border-radius: 2px;			
  pointer-events: none;			
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