<!-- 多条线的line chart -->
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
  name: 'LineGroupChart',
  data () {
    return {
      width: '1000',
      height: '400',
      svg: {},
      padding: 40,
      chart: {
        name: '折线分组图',
        labels: ['周一','周二','周三','周四','周五','周六','周日'],
        data: [
          {
            name:'邮件营销',
            type:'line',
            data:[120, 132, 101, 134, 0, 230, 210]
          },
          {
            name:'联盟广告',
            type:'line',
            data:[220, 182, 191, 234, 290, 330, 310]
          },
          {
            name:'视频广告',
            type:'line',
            data:[150, 232, 201, 154, 190, 330, 410]
          },
          {
            name:'直接访问',
            type:'line',
            data:[320, 332, 301, 334, 390, 330, 320]
          },
          {
            name:'搜索引擎',
            type:'line',
            data:[820, 932, 901, 934, 1290, 1330, 1320]
          }
        ]
      }
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
  //props: ['chart'],
  mounted () {
    this.width = this.$el.getBoundingClientRect().width || 1000
    this.initChart()
    // TODO test
    //this.updateChart()
  },
  updated () {
    //this.updateChart()
  },
  methods: {
    initChart () {
      this.svg = d3.select(this.$el).select('.line-chart-body')
        .append('svg')
        .attr('class', 'svg-container')
        .attr('width', this.width)
        .attr('height', this.height)

      // 计算最大值
      const maxValues = this.chart.data.map(d => {
        return d3.max(d.data)
      })

      const xScale = d3.scaleBand().range([0, this.w]).domain(this.chart.labels)
      const xAxis = d3.axisBottom(xScale)
      this.svg.append('g')
        .attr('class', 'x axis')
        .attr('transform', 'translate(' + this.padding + ', ' + (this.h + this.padding) + ')')
        .call(xAxis)

      const yScale = d3.scaleLinear().range([this.h, 0]).domain([0, d3.max(maxValues)])
      console.log('最大值', d3.max(maxValues))
      const yAxis = d3.axisLeft(yScale).ticks(5).tickSize(-this.w)
      this.svg.append('g')
        .attr('class', 'y axis')
        .attr('transform', 'translate(' + this.padding + ', ' + this.padding + ')')
        .call(yAxis)

      const line = d3.line()
        .x((d, i) => xScale(this.chart.labels[i]))
        .y(d => yScale(d))

      // 绘制多条path
      let dataList = []
      const colors = d3.schemeSet3
      this.chart.data.forEach( (item, index) => {
        const bandWidth = (this.w/maxValues.length)/2
        this.svg.append('path')
          .datum(item.data)
          .attr('d', line)
          .attr('class', 'line')
          .attr('transform', 'translate(' + bandWidth + ',' + this.padding + ')')
          .attr('stroke', colors[index])
          .attr('stroke-width', '2px')
          .attr('fill', 'none')
          dataList.push(item.data)
      })

      const datasets = d3.merge(dataList)
      console.log('所有数据',datasets)

      // 添加图例的显示


      // 添加图例的显示
      d3.select(this.$el)
        .append('div')
        .attr('class','hover-tip')
        .style('visibility', 'hidden')

      // const circles = this.svg.selectAll('circle')
      //   .data(datasets)
      // circles.exit().remove()
      // circles.enter()
      //   .append('circle')
      //   .attr('class', 'hover-point')
      //   .attr('r', 3)
      //   .attr('cx', (d,i) => xScale(this.chart.labels[i]) + this.padding)
      //   .attr('cy', d => yScale(d) + this.padding)
      //   .attr('fill', (d, i) => colors[i])
      //   .on('mouseover', function(d, i) {
      //     const pointX = xScale(this.chart.labels[i])
      //     const pointY = yScale(d)-5
      //     const label = this.chart.labels[i]
      //     d3.select(element).select('.hover-tip')
      //     .style('visibility', 'visible')
      //     .style('left', `${pointX}px`)
      //     .style('top', `${pointY}px`)
      //     .html(`<div>时间：${label}</div><div>取值：${d.value}</div>`)
      //   })
      //   .on('mouseout', function() {
      //     d3.select(element).select('.hover-tip').style('visibility', 'hidden')
      //   });
    },
    updateChart () {
      const element = this.$el
      var format = d3.timeFormat("%Y/%m/%d %H:%M:%S");
      const minTime = d3.min(this.chart.data, d => d.label)
      const maxTime = d3.max(this.chart.data, d => d.label)
      const xScale = d3.scaleTime().range([0, this.w]).domain([new Date(minTime), new Date(maxTime)])
      const xAxis = d3.axisBottom(xScale).ticks(5).tickFormat(format) // %Y/%m/%d
      this.svg.select('.x.axis').call(xAxis)

      const yScale = d3.scaleLinear().range([this.h, 0]).domain([0, d3.max(this.chart.data, d => d.value)])
      const yAxis = d3.axisLeft(yScale).ticks(5).tickSize(-this.w)
      this.svg.select('.y.axis').call(yAxis)
      const line = d3.line()
        .x(d => xScale(d.label))
        .y(d => yScale(d.value))
      this.svg.select('.line').datum(this.chart.data).attr('d', line)

      const circles = this.svg.selectAll('circle')
        .data(this.chart.data)
      circles.exit().remove()
      circles.enter()
        .append('circle')
        .attr('class', 'hover-point')
        .attr('r', 3)
        .attr('cx', d => xScale(d.label) + this.padding)
        .attr('cy', d => yScale(d.value) + this.padding)
        .attr('fill', 'rgb(188, 82, 188)')
        .on('mouseover', function(d) {
          const pointX = xScale(d.label)
          const pointY = yScale(d.value)-5
          const label = format(d.label)
          d3.select(element).select('.hover-tip')
          .style('visibility', 'visible')
          .style('left', `${pointX}px`)
          .style('top', `${pointY}px`)
          .html(`<div>时间：${label}</div><div>取值：${d.value}</div>`)
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
  .axis.y .tick:first-child line{
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
