<template>
  <div :class="className" :style="{height:height,width:width}" />
</template>

<script>
import echarts from 'echarts'
require('echarts/theme/macarons') // echarts theme
import resize from './mixins/resize'

const animationDuration = 3000

export default {
  mixins: [resize],
  props: {
    className: {
      type: String,
      default: 'chart'
    },
    width: {
      type: String,
      default: '100%'
    },
    height: {
      type: String,
      default: '300px'
    }
  },
  data() {
    return {
      chart: null
    }
  },
  mounted() {
    this.$nextTick(() => {
      this.initChart()
    })
  },
  beforeDestroy() {
    if (!this.chart) {
      return
    }
    this.chart.dispose()
    this.chart = null
  },
  methods: {
    initChart() {
      this.chart = echarts.init(this.$el, 'macarons')
      let option = {
          tooltip: {
              trigger: 'axis',
              axisPointer: {            // Use axis to trigger tooltip
                  type: 'shadow'        // 'shadow' as default; can also be 'line' or 'shadow'
              }
          },
          legend: {
              data: ['电子文档', '文本数据']
          },
          grid: {
              left: '3%',
              right: '4%',
              bottom: '3%',
              containLabel: true
          },
          xAxis: {
              type: 'value'
          },
          yAxis: {
              type: 'category',
              data: [ '占比']
          },
          series: [
              {
                  name: '电子文档',
                  type: 'bar',
                  stack: 'total',
                  label: {
                      show: true
                  },
                  emphasis: {
                      focus: 'series'
                  },
                  data: [3]
              },
              {
                  name: '文本数据',
                  type: 'bar',
                  stack: 'total',
                  label: {
                      show: true
                  },
                  emphasis: {
                      focus: 'series'
                  },
                  data: [1]
              },
              
          ]
      };

      this.chart.setOption(option)
    }
  }
}
</script>
