<template>
  <svg :id="category"/>
</template>

<script>
  import * as d3 from 'd3';
  export default {
    props: ['data', 'category', 'colors', 'extent', 'jobs', 'width'],
    data() {
      return {
        // width: 400,
        height: 20,
        lWidth: 100,
        lHeight: 50
      }
    },
    mounted() {
      this.initLegend()
    },
    methods: {
      initLegend() {
        let svg = d3.select('#' + this.category)
          .attr('height', this.height)
          .attr('width', this.width)
        
        const g = svg.append('g')
          .attr("transform", "translate(" + 40 + "," + 0 + ")")
        
        let xScale = d3.scaleLinear()
          .domain([0, this.colors.length])
          .range([0, this.lWidth])

        g.append('g')
          .attr('class', 'legends')
          .selectAll('bars')
          .data(this.colors)
          .enter().append('rect')
            .attr('x', (d, i) => {
              return xScale(i)
            })
            .attr('y', 10)
            .attr('width', 10)
            .attr('height', 10)
            .attr('fill', (d) => {return d})

        g.append('g')
          .append('text')
            .attr('x', xScale(0) - 32)
            .attr('y', 19)
            .text((this.extent[0] * 100).toFixed(2))
            .style('font-size','11px')
            .style('font-family', 'Arial, Helvetica, sans-serif')
        
        g.append('g')
          .append('text')
            .attr('x', xScale(this.colors.length - 1) + 13)
            .attr('y', 19)
            .text((this.extent[1] * 100).toFixed(2))
            .style('font-size','11px')
            .style('font-family', 'Arial, Helvetica, sans-serif')

        g.append('g')
          .append('text')
            .attr('x', this.width * .55)
            .attr('y', 19)
            .text('Total Jobs: ' + this.jobs.toString().replace(/\B(?=(\d{3})+(?!\d))/g, ","))
            .style('font-size','14px')
            .style('font-family', 'Arial, Helvetica, sans-serif')
      }
    }
  }
</script>

<style lang="scss" scoped>

</style>