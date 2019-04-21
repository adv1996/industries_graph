<template>
  <v-container fluid grid-list-sm>
    <v-layout row wrap>
      <v-flex
        xs12
        md4
        xl3
        v-for="category in categories"
        :key="category"
        d-flex
      >
        <v-card
          color="white"
          tile
          height="455"
        >
          <v-card-title primary-title>
            <div>
              <h3 v-if="rows[category][0][3].length < 38">{{ rows[category][0][3].replace('and', '&') }}</h3>
              <h4 v-else>{{ rows[category][0][3].slice(0, 38).replace('and', '&') + '...' }}</h4>
            </div>
            <v-tooltip
              bottom
            >
              <template v-slot:activator="{ on }">
                <v-icon
                  small
                  right
                  v-on="on"
                >
                  feedback
                </v-icon>
              </template>
              <span>{{ rows[category][0][3] }}</span>
            </v-tooltip>
          </v-card-title>
          <MapViz :data="rows[category]" :category="category"/>
           <v-divider light></v-divider>
          <v-card-actions class="pa-1">
            <v-spacer></v-spacer>
            <v-btn
              v-on:click="startAnimation(rows[category], category)"
              color="#9ebcda"
              small
            >
              <v-icon>play_arrow</v-icon>
            </v-btn>
            <v-spacer></v-spacer>
          </v-card-actions>
        </v-card>
      </v-flex>
    </v-layout>
  </v-container>
</template>

<script>
  import _ from 'lodash';
  import * as d3 from 'd3';
  import IndustryData from '../data/naics-2018.json';
  import MapViz from './Map.vue';
  
  export default {
    components: {
      MapViz,
    },
    data() {
      return {
        rows: [],
        categories: [],
        orgData: null,
        wages: null,
      }
    },
    created() {
      this.setData();
      this.setLegendTooltip();
      this.setTooltip();
      this.setBoxTooltip();
    },
    methods: {
      startAnimation(data, category) {
        let wageState = this.orgData.get(category)
        let cardWidth = this.$el.children[0].children[0].offsetWidth
        // collision logic
        let wagesList = this.wages.get(category)
        
        let sorted = wagesList[category].sort()
        let bins = []
        let range = [25, cardWidth - 50]
        let rangeBins = []
        let inc = 0
        let dist = 11
        for(let m = 0; m < dist; m++) {
          bins.push(d3.quantile(sorted, inc))
          rangeBins.push(d3.quantile(range, inc))
          inc += .1
        }
        let yPosScale = d3.scaleThreshold()
          .domain(bins)
          .range(rangeBins)

        let yPosMap = new Map()
        let countBucketHeight = new Map()
        Object.keys(wageState).forEach((ws) => {
          let currentState = wageState[ws][0].median
          let band = yPosScale(currentState).toFixed(0).toString()
          if (countBucketHeight.get(band)) {
            let stackValue = countBucketHeight.get(band)
            countBucketHeight.set(band, stackValue + 1)
            yPosMap.set(wageState[ws][0].id, [band, stackValue + 1])
          } else {
            yPosMap.set(wageState[ws][0].id, [band, 1])
            countBucketHeight.set(band, 1)
          }
        })

        let wageExtent = d3.extent(wagesList[category])

        let wageScale = d3.scaleLinear()
          .domain([wageExtent[0], wageExtent[1]])
          .range(range)
        // let that = this
        d3.selectAll('.state-areas-' + category)
          .transition()
          .duration(2500)
          .attr('transform', function(d) {
            if (d.id && wageState[d.id]) {
              let median = wageState[d.id][0].median
              // x movement
              let xs = wageScale(median)
              let transform = d3.select(this).node().getBBox();
              let currentX = transform['x'] / 2
              let currentY = transform['y'] / 2
              let bh = yPosMap.get(d.id)[1]
              return 'translate(' + (-currentX + xs - 5) + ',' + (-currentY + bh * 25 + 40) + ') scale(.50, .50)'
            }
            return 'translate(' + 0 + ', 0)'
          })
          .attr('stroke', 'black')
      

        d3.select('.boxLine' + category + 'Box')
          .transition()
          .duration(1000)
          .attr('transform', 'translate(' + -(cardWidth / 4) + ',0)')
          .attr('x1', wageScale(wageExtent[0]))
          .attr('x2', wageScale(wageExtent[1]))
          .attr('stroke', 'black')
          .attr('opacity', 1)
      
        // box in box plot
        let a10 = d3.quantile(sorted, .1)
        let a25 = d3.quantile(sorted, .25)
        let a50 = d3.quantile(sorted, .50)
        let a75 = d3.quantile(sorted, .75)
        let a90 = d3.quantile(sorted, .90)

        d3.select('.rect' + category + 'Box')
          .transition()
          .duration(1000)
          .attr('transform', 'translate(' + -(cardWidth / 4) + ',0)')
          .attr('x', wageScale(a25))
          .attr("width", wageScale(a75) - wageScale(a25))
        
        let circles = d3.selectAll('.circles' + category + 'Box')
          .data([a10, a25, a50, a75, a90])

        circles.exit().remove()
        circles.enter().append('circles')
          .merge(circles)
            .transition()
            .duration(1000)
            .attr('transform', 'translate(' + -(cardWidth / 4) + ',0)')
            .attr('cx', (d) => {return wageScale(d)})
            
        let labels = d3.selectAll('.bLabels' + category + 'Box')
          .data([a10, a50, a90])

        labels.exit().remove()
        labels.enter().append('text')
          .merge(labels)
            .transition()
            .duration(1000)
            .attr('transform', 'translate(' + -(cardWidth / 4) + ',0)')
            .attr('x', (d) => {return wageScale(d) - 15})
            .text((d) => {return (d / 1000).toFixed(0) + 'K'})
        
        // turning off path borders
        d3.selectAll('.state-borders' + category)
          .style('opacity', 0)
        
        //label box plot
        d3.select('.boxPlotLabel' + category + 'Box')
          .transition()
          .duration(1000)
          .text('Median State Salary Distributed')
      },
      setBoxTooltip() {
        d3.select('body')
          .append('div')
          .attr('class', 'boxTooltip')	
          .style('opacity', 0)
      },
      setTooltip() {
        d3.select('body')
          .append('div')
          .attr('class', 'tooltip')	
          .style('opacity', 0)
      },
      setLegendTooltip() {
        d3.select('body')
          .append('div')
          .attr('class', 'legendTooltip')	
          .style('opacity', 0)
      },
      setData() {
        this.rows = _.groupBy(IndustryData.rows, (r) => {
          return r[3][0] + r[2]
        })
        this.categories = Object.keys(this.rows)
        this.orgData = new Map()
        this.wages = new Map()
        this.categories.forEach((k) => {
          this.orgData.set(k, _.groupBy(_.map(this.rows[k], (d) => {return {id:d[0], state: d[1], median: parseInt(d[18])}}), (d) => {return d.id}))
          this.wages.set(k, _.groupBy(_.map(this.rows[k], (w) => {return parseInt(w[18])}), () => {return k}))
        })
        
      },
    }
  }
</script>

<style>
div.legendTooltip {	
  position: absolute;
  text-align: center;
  width: 60px;			
  padding: 2px;				
  font-size: 12px;		
  font-family: "Helvetica Neue", Helvetica, sans-serif;
  color: black;
  background:white;	
  border: 0px;		
  border-radius: 2px;			
  pointer-events: none;			
}
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
div.boxTooltip {	
  position: absolute;
  text-align: center;
  width: 60px;			
  padding: 2px;				
  font-size: 12px;		
  font-family: "Helvetica Neue", Helvetica, sans-serif;
  color: black;
  background:white;	
  border: 0px;		
  border-radius: 2px;			
  pointer-events: none;			
}
</style>