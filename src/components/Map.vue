<template>
  <div>
    <Legend v-if="width" :colorScale="color" :category="legendName" :colors="colors" :extent="extent" :jobs="getJobs(data)" :width="width"/>
    <svg :id="category"/>
    <BoxPlot v-if="width" :category="boxName" :data="data" :width="width"/>
  </div>
</template>

<script>
  /* eslint-disable */
  import _ from 'lodash';
  import * as d3 from 'd3';
  import {feature, mesh} from 'topojson';
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
        color: null
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
      this.setDimensions()
      this.initMap()
    },
    methods: {
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
      },
      setData() {
        this.density = []
        let test = []
        this.dataMap = _.keyBy(_.map(this.data, (state) => {
          let population = parseFloat(state[21].replace(/,/g, ''))
          let jobs = parseFloat(state[5])
          let value = (jobs / population) * 100

          // District of Columbia heavily skews data
          if (state[1] !== 'District of Columbia') {
            this.density.push(value)
          }
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
        let that = this;
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
            .attr('class', 'state-areas-' + this.category)
            .on('mouseover', function(d) {
              try {
                let offsetTooltipX = d3.event.pageX - 20
                let offsetTooltipY = d3.event.pageY - 30
                let barrier = d3.mouse(this)
                if (barrier[0] > that.width - 90) {
                  offsetTooltipX = d3.event.pageX - 90
                  offsetTooltipY = d3.event.pageY - 50
                }
                if (that.dataMap[d.id].jobs) {
                  tooltip
                    .style("left", offsetTooltipX + "px")	
                    .style("top", offsetTooltipY + "px")
                    .style('opacity', 1)
                    .style('height', 50)
                    .html(
                      that.dataMap[d.id].state + '<br/>' +
                      that.dataMap[d.id].jobs.toString().replace(/\B(?=(\d{3})+(?!\d))/g, ",") + ' Jobs' + '<br/>' +
                      '$' + that.dataMap[d.id].median.toFixed(0).toString().replace(/\B(?=(\d{3})+(?!\d))/g, ",")
                    )
                }
              }
              catch(err) {
                // handle error on undefined state (possibly Delaware)
              }
            })
            .on('mouseout', () => {
              tooltip
                .style('opacity', 0)
            })

        g.append('path')
          .attr('class', 'state-borders' + this.category)
          .attr('d', path(mesh(MapData, MapData.objects.states, (a,b) => {return a != b})))
          .attr('fill', 'none')
          .attr('stroke', '#fff')
          .style('stroke-width', '0.5px')
          .style('stroke-linecap', 'round')
          .style('stroke-linejoin', 'round')
          .style('pointer-events', 'none')
      }
    }
  }
</script>

<style>
/* .state-borders {
  fill: none;
  stroke: #fff;
  stroke-width: 0.5px;
  stroke-linejoin: round;
  stroke-linecap: round;
  pointer-events: none;
} */
</style>