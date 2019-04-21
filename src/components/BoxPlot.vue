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
        height: 50,
      }
    },
    mounted() {
      this.setData()
      this.initBoxPlot()
    },
    methods: {
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
          .attr('class', 'rect' + this.category)
          .attr('x', xScale(sums[1]))
          .attr('y', 5)
          .attr('height', 10)
          .attr('width', 0)
          .attr('fill', '#810f7c')
          .style('opacity', 0.6)
          .transition()
          .delay(1250)
          .duration(1500)
          .attr("width", xScale(sums[3]) - xScale(sums[1]))

        g.append('line')
          .attr('class', 'boxLine' + this.category)
          .attr('x1', xScale(sums[0]))
          .attr('y1', 10)
          .attr('x2', 0)
          .attr('y2', 10)
          .attr('stroke', 'black')
          .transition()
          .delay(1000)
          .duration(1500)
          .attr('x2', xScale(sums[4]))

        g.append('g')
          .attr('class', 'circles')
          .selectAll('nodes')
          .data(sums)
          .enter().append('circle')
          .attr('class', 'circles' + this.category)
          .attr('cx', (d) => {
            return xScale(d)
          })
          .attr('cy', 10)
          .attr('r', 0)
          .on('mouseover', (d) => {
            boxTooltip
              .style("left", (d3.event.pageX - 25) + "px")	
              .style("top", (d3.event.pageY - 25) + "px")
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
          .transition()
          .delay(1000)
          .duration(2000)
          .attr('r', 2)
        
        g.append('g')
          .attr('class', 'labels')
          .selectAll('bLabels')
          .data([sums[0], sums[2], sums[4]])
          .enter().append('text')
          .attr('class', 'bLabels' + this.category)
          .attr('x', (d) => {return xScale(d) - 15})
          .attr('y', 27)
          .text((d) => {return (d / 1000).toFixed(0) + 'K'})
          .style('opacity', 0)
          .transition()
          .delay(1500)
          .duration(2000)
          .style('opacity', 1)
      
        g.append('text')
          .attr('class', 'boxPlotLabel' + this.category)
          .attr('x', 20)
          .attr('y', 42)
          .text('Average Median Salary Across US')
          .style('opacity', 0)
          .style('font-size','10px')
          .transition()
          .delay(1500)
          .duration(2000)
          .style('opacity', 1)
      
      }
    }
  }
</script>

<style>

</style>