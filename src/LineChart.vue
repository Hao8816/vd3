<!-- 单条线的line chart -->
<template lang="html">
  <div ref="chart" class="line-chart" :style="{ 'width' : width, 'height' : height}">
    <div class="line-chart-title">
      {{chart.name}}
    </div>
    <div class="line-chart-body" v-show="chart.data.length"></div>
  </div>
</template>

<script>
import * as d3 from 'd3'
/*
1. 定义坐标轴，x轴为时间，y轴为值
*/
export default {
  name: 'LineChart',
  data () {
    return {
      width: '1000',
      height: '400',
      svg: {},
      padding: 40
    }
  },
  computed: {
    w () {
      return this.width - 2 * this.padding
    },
    h () {
      return this.height - 2 * this.padding
    }
  },
  props: ['chart'],
  mounted () {
    this.width = this.$el.getBoundingClientRect().width || 1000
    this.initChart()
  },
  updated () {
    this.updateChart()
  },
  methods: {
    initChart () {
      this.svg = d3.select(this.$el).select('.line-chart-body')
        .append('svg')
        .attr('class', 'svg-container')
        .attr('width', this.width)
        .attr('height', this.height)

      const minTime = d3.min(this.chart.data, d => d.unixtime)
      const maxTime = d3.max(this.chart.data, d => d.unixtime)
      const xScale = d3.scaleTime().range([0, this.w]).domain([new Date(minTime), new Date(maxTime)])
      const xAxis = d3.axisBottom(xScale).ticks(10).tickFormat(d3.timeFormat('%H:%M:%S')) // %Y/%m/%d
      this.svg.append('g')
        .attr('class', 'x axis')
        .attr('transform', 'translate(' + this.padding + ', ' + (this.h + this.padding) + ')')
        .call(xAxis)

      const yScale = d3.scaleLinear().range([this.h, 0]).domain([0, 100])
      const yAxis = d3.axisLeft(yScale).ticks(5).tickSize(-this.w)
      this.svg.append('g')
        .attr('class', 'y axis')
        .attr('transform', 'translate(' + this.padding + ', ' + this.padding + ')')
        .call(yAxis)

      const line = d3.line()
        .x(d => xScale(d.unixtime))
        .y(d => yScale(d.value))

      this.svg.append('path')
        .datum(this.chart.data)
        .attr('d', line)
        .attr('class', 'line')
        .attr('transform', 'translate(' + this.padding + ',' + this.padding + ')')
        .attr('stroke', 'rgb(188, 82, 188)')
        .attr('stroke-width', '2px')
        .attr('fill', 'none')

      d3.select(this.$el)
        .append('div')
        .attr('class','hover-tip')
        .style('visibility', 'hidden')
    },
    updateChart () {
      const element = this.$el
      var format = d3.timeFormat("%Y/%m/%d %H:%M:%S");
      const minTime = d3.min(this.chart.data, d => d.unixtime)
      const maxTime = d3.max(this.chart.data, d => d.unixtime)
      const xScale = d3.scaleTime().range([0, this.w]).domain([new Date(minTime), new Date(maxTime)])
      const xAxis = d3.axisBottom(xScale).ticks(5).tickFormat(format) // %Y/%m/%d
      this.svg.select('.x.axis').call(xAxis)

      const yScale = d3.scaleLinear().range([this.h, 0]).domain([0, d3.max(this.chart.data, d => d.value)])
      const yAxis = d3.axisLeft(yScale).ticks(5).tickSize(-this.w)
      this.svg.select('.y.axis').call(yAxis)
      const line = d3.line()
        .x(d => xScale(d.unixtime))
        .y(d => yScale(d.value))
      this.svg.select('.line').datum(this.chart.data).attr('d', line)

      const circles = this.svg.selectAll('circle')
        .data(this.chart.data)
      circles.exit().remove()
      circles.enter()
        .append('circle')
        .attr('class', 'hover-point')
        .attr('r', 3)
        .attr('cx', d => xScale(d.unixtime) + this.padding)
        .attr('cy', d => yScale(d.value) + this.padding)
        .attr('fill', 'rgb(188, 82, 188)')
        .on('mouseover', function(d) {
          const pointX = xScale(d.unixtime)
          const pointY = yScale(d.value)-5
          const unixtime = format(d.unixtime)
          d3.select(element).select('.hover-tip')
          .style('visibility', 'visible')
          .style('left', `${pointX}px`)
          .style('top', `${pointY}px`)
          .html(`<div>时间：${unixtime}</div><div>取值：${d.value}</div>`)
        })
        .on('mouseout', function() {
          d3.select(element).select('.hover-tip').style('visibility', 'hidden')
        });
    }
  }
}
</script>

<style lang="css">
  .axis .tick text{
    fill: #000;
  }
  .axis.y .tick line{
    stroke: rgb(119, 119, 119);
    stroke-dasharray: 2, 2;
  }
  .axis.y .tick line:last-child{
    stroke: transparent;
  }
  .axis.y .domain{
    stroke: transparent;
  }
  .line-chart{
    position: relative;
    text-align: center;
  }
  .line-chart-title{
    font-size: 16px;
    text-align: center;
    color: #333;
  }
  .hover-point{
    cursor: pointer;
    position: absolute;
  }
  .hover-tip{
    width: 180px;
    text-align: left;
    padding: 10px;
    border-radius: 4px;
    position: absolute;
    background-color: rgba(0,0,0,0.8);
    color: #fff;
  }
  .hover-tip::before{
    position: absolute;
    content: "";
    left: 35px;
    bottom:-10px;
    color: red;
    height: 10px;
    width:10px;
    border-left: 5px solid transparent;
    border-right: 5px solid transparent;
    border-top: 5px solid rgba(0,0,0,0.8);

  }
</style>
