<template>
  <svg :id="category"/>
</template>

<script>
  import * as d3 from 'd3';
  export default {
    props: ['colorScale', 'category', 'colors', 'extent', 'jobs', 'width'],
    data() {
      return {
        height: 30,
        lWidth: 100,
        lHeight: 50
      }
    },
    mounted() {
      this.initLegend()
    },
    methods: {
      initLegend() {
        let legendTooltip = d3.select('.legendTooltip')

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
            .attr('y', 16)
            .attr('width', 10)
            .attr('height', 10)
            .attr('fill', (d) => {return d})
            .on('mouseover', (d) => {
              let ie = this.colorScale.invertExtent(d)
              legendTooltip
                .style("left", (d3.event.pageX - 30) + "px")	
                .style("top", (d3.event.pageY - 27.5) + "px")
                .style('opacity', 1)
                .style('height', 50)
                .html(
                  (((ie[0] + ie[1]) / 2)).toFixed(2)
                )
            })
            .on('mouseout', () => {
              legendTooltip
                .style('opacity', 0)
            })

        g.append('g')
          .append('text')
            .attr('x', xScale(0) - 32)
            .attr('y', 25)
            .text((this.extent[0]).toFixed(3))
            .style('font-size','11px')
        
        g.append('g')
          .append('text')
            .attr('x', xScale(this.colors.length - 1) + 13)
            .attr('y', 25)
            .text((this.extent[1]).toFixed(3))
            .style('font-size','11px')
        
        g.append('g')
          .append('text')
            .attr('x', xScale(this.colors.length / 2) - 65)
            .attr('y', 10)
            .text('% of Population in Industry')
            .style('font-size','11px')

        g.append('g')
          .append('text')
            .attr('x', xScale(this.colors.length - 1) + this.width * .25)
            .attr('y', 19)
            .text('# of Jobs: ' + this.jobs.toString().replace(/\B(?=(\d{3})+(?!\d))/g, ","))
            .style('font-size','13px')
      }
    }
  }
</script>

<style lang="scss" scoped>

</style>