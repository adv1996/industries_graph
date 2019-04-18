<template>
  <v-container>
    <v-card xs12>
      <v-chip
        v-for="c in categories" :key="c.id"
        v-on:click="changeCategory(c)"
        :color="c.status ? selected : notSelected"
      >
        <strong>{{ c.name }}</strong>&nbsp;
      </v-chip>
    </v-card>
    <v-card xs12 md6>
      <v-card-title primary-title>
        <div>
          <h3 class="headline mb-0">{{ first }}</h3>
        </div>
      </v-card-title>
      <svg id="mapid"/>
    </v-card>
  </v-container>
</template>

<script>
  /* eslint-disable */
  import _ from 'lodash';
  import * as d3 from 'd3';
  import {feature, mesh} from 'topojson';
  import IndustryData from '../data/Industry.json';
  import Population from '../data/population.json';

  export default {
    data() {
      return {
        map: null,
        tileLayer: null,
        layers: [],
        width: 800,
        height: 400,
        jobs: [],
        dataMap: null,
        max: 0,
        min: 0,
        popMap: null,
        categories: [],
        first: 'Agriculture, Forestry, Fishing and Hunting',
        color: null,
        selected: 'green',
        notSelected: 'grey'
      }
    },
    mounted() {
      this.setPopulation()
      this.setData(this.first)
      this.initMap()
    },
    methods: {
      changeCategory(c) {
        this.setData(c.name)
        this.first = c.name
        this.categories[c.index].status = true;
        this.color = d3.scaleQuantize([this.min, this.max], ['#e0ecf4','#bfd3e6','#9ebcda','#8c96c6','#8c6bb1','#88419d','#810f7c','#4d004b'])
        d3.selectAll('.state-areas')
          .attr('fill', (d) => {
            if (this.dataMap.get(d.id)) {
              return this.color(this.dataMap.get(d.id))
            } else {
              return 'gray'
            }
          })
      },
      setPopulation() {
        this.popMap = new Map();
        let population = Population.states;
        population.forEach((p) => {
          this.popMap.set(p[0], p[1])
        })
      },
      setData(c) {
        this.dataMap = new Map();
        let rows = IndustryData.rows;
        rows = _.groupBy(rows, (r) => {return r[3]})
        this.categories = []
        Object.keys(rows).forEach((status, i) => {
          this.categories.push({
            name: status,
            status: false,
            index: i,
          })
        })
        rows = rows[c]
        let ex = []
        let density;
        let pop;
        for (let i = 0; i < rows.length; i++) {
          if (this.popMap.get(rows[i][1])) {
            pop = this.popMap.get(rows[i][1]).replace(/,/g, '')
            density = (parseFloat(rows[i][5]) / parseFloat(pop)) * 100
            this.dataMap.set(rows[i][0], density)
            ex.push(density)
          }
        }
        let maxMin = d3.extent(ex)
        this.min = maxMin[0]
        this.max = maxMin[1]
      },
      initMap() {
        // let clearSvg = d3.select('#mapid')
        // clearSvg.selectAll("*").remove();
        this.categories[0].status = true;
        this.color = d3.scaleQuantize([this.min, this.max], ['#e0ecf4','#bfd3e6','#9ebcda','#8c96c6','#8c6bb1','#88419d','#810f7c','#4d004b'])
        let svg = d3.select('#mapid')
          .attr('width', this.width)
          .attr('height', this.height)
          .attr("viewBox", "0 0 " + this.width + " " + this.height )
          .attr("preserveAspectRatio", "xMinYMin");
        // let path = d3.geoPath();
        const url = "https://d3js.org/us-10m.v1.json"
        
        d3.json(url).then((data) => {
          let featureCollection = feature(data, data.objects.states);
          let projection = d3.geoIdentity()
            .fitExtent([[50,50],[this.width-50,this.height-50]], featureCollection)

          let path = d3.geoPath().projection(projection)
          svg.append('g')
            .attr('class', 'states')
            .selectAll('path')
            .data(feature(data, data.objects.states).features)
            .enter().append('path')
              .attr('d', path)
              .attr('fill', (d) => {
                if (this.dataMap.get(d.id)) {
                  return this.color(this.dataMap.get(d.id))
                } else {
                  console.log(d.id)
                  return 'gray'
                }
              })
              .style('opacity', 0.75)
              .attr('class', 'state-areas')
              .on('mouseover', function(d,i) {
                d3.select(this)
                  .style('opacity', 1)
              })
              .on('mouseout', function(d, i) {
                d3.select(this)
                  .style('opacity', 0.75)
              })

          svg.append('path')
            .attr('class', 'state-borders')
            .attr('d', path(mesh(data, data.objects.states, (a,b) => {return a != b})))
        })
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