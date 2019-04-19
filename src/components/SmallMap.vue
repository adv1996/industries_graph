<template>
  <v-container>
    <!-- <v-card xs12>
      <v-chip
        v-for="c in categories" :key="c.id"
        v-on:click="changeCategory(c)"
        :color="c.status ? selected : notSelected"
      >
        <strong>{{ c.name }}</strong>&nbsp;
      </v-chip>
    </v-card> -->
    <v-card xs12 md6>
      <v-card-title primary-title>
        <div>
          <h3 class="headline mb-0">{{ first }}</h3>
        </div>
      </v-card-title>
      <svg id="mapid"/>
    </v-card>
    <v-card xs12 md6>
      <svg id="wageBox"/>
    </v-card>
  </v-container>
</template>

<script>
  /* eslint-disable */
  import _ from 'lodash';
  import * as d3 from 'd3';
  import {feature, mesh} from 'topojson';
  import IndustryData from '../data/naics.json';

  export default {
    data() {
      return {
        map: null,
        tileLayer: null,
        layers: [],
        width: 500,
        height: 300,
        jobs: [],
        dataMap: null,
        max: 0,
        min: 0,
        popMap: null,
        categories: [],
        first: 'Agriculture, Forestry, Fishing and Hunting',
        color: null,
        selected: 'green',
        notSelected: 'grey',
        rows: [],
      }
    },
    mounted() {
      // this.setPopulation()
      this.dataMap = new Map();
      this.rows = _.groupBy(IndustryData.rows, (r) => {return r[3]})

      this.setData(this.first)
      this.initMap()

      this.initBoxplot()
    },
    methods: {
      initBoxplot() {
        let sampleData = {
          pct10: 23350,
          pct25: 34020,
          median: 42380,
          pct75: 60330,
          pct90: 77150,
        }
        let xScale = d3.scaleLinear()
          .domain([sampleData['pct10'], sampleData['pct90']])
          .range([10, 250])
        let svg = d3.select('#wageBox')
          .attr('width', 300)
          .attr('height', 100)
          // .style('background', 'gray')
        
        const g = svg.append('g')
          .attr("transform", "translate(" + 10 + "," + 10 + ")")
        
        g.append('line')
          .attr('x1', xScale(sampleData['pct10']))
          .attr('y1', 50)
          .attr('x2', xScale(sampleData['pct90']))
          .attr('y2', 50)
          .attr('stroke', 'black')

        g.append('rect')
          .attr('x', xScale(sampleData['pct25']))
          .attr('y', 25)
          .attr('width', xScale(sampleData['median']) - xScale(sampleData['pct25']))
          .attr('height', 50)
          .attr('fill', 'blue')
        
        g.append('rect')
          .attr('x', xScale(sampleData['median']) )
          .attr('y', 25)
          .attr('width', xScale(sampleData['pct75']) - xScale(sampleData['median']))
          .attr('height', 50)
          .attr('fill', 'orange')

        // g.append('line')
        //   .attr('x1', xScale(sampleData['median']))
        //   .attr('y1', 25)
        //   .attr('x2', xScale(sampleData['median']))
        //   .attr('y2', 75)
        //   .attr('stroke', 'black')
      },
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
      setData(c) {
        this.categories = []
        Object.keys(this.rows).forEach((status, i) => {
          this.categories.push({
            name: status,
            status: false,
            index: i,
          })
        })
        let rows = this.rows[c]
        let ex = []
        let density;
        let pop;
        for (let i = 0; i < rows.length; i++) {
          pop = rows[i][21].replace(/,/g, '')
          density = (parseFloat(rows[i][5]) / parseFloat(pop)) * 100
          this.dataMap.set(rows[i][0], density)
          ex.push(density)
        }
        let maxMin = d3.extent(ex)
        this.min = maxMin[0]
        this.max = maxMin[1]
      },
      initMap() {
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
            .fitExtent([[25, 25],[this.width - 25,this.height - 25]], featureCollection)

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