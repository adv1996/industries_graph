<template>
  <svg ref="box" :id="category"/>
</template>

<script>
  import _ from 'lodash';
  import * as d3 from 'd3';

  export default {
    props: ['category', 'data', 'width'],
    data() {
      return {
        boxValues: [],
        height: 40,
      }
    },
    mounted() {
      this.setTooltip()
      this.setData()
      this.initBoxPlot()
    },
    methods: {
      setTooltip() {
        d3.select('body')
          .append('div')
          .attr('class', 'boxTooltip')	
          .style('opacity', 0)
      },
      setData() {
        for (let k = 0; k < 5; k++) {
          this.boxValues.push({
            sum: 0,
            count: 0,
          })
        }
        this.data.forEach((d) => {
          for (let j = 0; j < 5; j++) {
            if (d[j + 16].length > 0) {
              this.boxValues[j].sum = this.boxValues[j].sum + parseInt(d[j + 16])
              this.boxValues[j].count = this.boxValues[j].count + 1
            }
          }
        })
      },
      initBoxPlot() {
        let boxTooltip = d3.select('.boxTooltip')
        // debugger;
        // this.width = this.$el.offsetWidth;

        let svg = d3.select('#' + this.category)
          .attr('height', this.height)
          .attr('width', this.width)

        let center = this.width * .50
        let quarter = this.width * .25

        const g = svg.append('g')
          .attr("transform", "translate(" + quarter + "," + 0 + ")")
        
        let sums = _.map(this.boxValues, (b) => {return b.sum / b.count})
        let xScale = d3.scaleLinear()
          .domain([sums[0], sums[4]])
          .range([0, center])

        g.append('rect')
          .attr('x', xScale(sums[1]))
          .attr('y', 5)
          .attr('height', 10)
          .attr('width', xScale(sums[3]) - xScale(sums[1]))
          .attr('fill', '#810f7c')
          .style('opacity', 0.6)

        g.append('line')
          .attr('x1', xScale(sums[0]))
          .attr('y1', 10)
          .attr('x2', xScale(sums[4]))
          .attr('y2', 10)
          .attr('stroke', 'black')

        g.append('g')
          .attr('class', 'circles')
          .selectAll('nodes')
          .data(sums)
          .enter().append('circle')
          .attr('cx', (d) => {
            return xScale(d)
          })
          .attr('cy', 10)
          .attr('r', 2)
          .on('mouseover', (d) => {
            boxTooltip
              .style("left", (d3.event.pageX + 10) + "px")	
              .style("top", (d3.event.pageY - 10) + "px")
              .style('opacity', 1)
              .style('height', 50)
              .html(
                '$' + d.toFixed(0).toString().replace(/\B(?=(\d{3})+(?!\d))/g, ",")
              )
          })
          .on('mouseout', () => {
            boxTooltip
              .style('opacity', 0)
          })
        
        g.selectAll('bLabels')
          .data([sums[0], sums[2], sums[4]])
          .enter().append('text')
          .attr('x', (d) => {return xScale(d) - 15})
          .attr('y', 27)
          .text((d) => {return (d / 1000).toFixed(0) + 'K'})
      
      }
    }
  }
</script>

<style>
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