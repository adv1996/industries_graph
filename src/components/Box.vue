<template>
  <div>

  </div>
</template>

<script>
  export default {
    mounted() {
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
      },
    }
  }
</script>

<style lang="scss" scoped>

</style>