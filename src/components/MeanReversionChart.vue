<template>
  <div ref="chartRef" style="min-width: 1200px; width: 100%; height: 600px;"></div>
</template>


<script setup>
import { ref, onMounted, onUnmounted } from 'vue'
import * as echarts from 'echarts'

const vwa = ref([])
const price = ref([])
const time = ref([])
let ws = null
const chartRef = ref()
let chart = null

const renderChart = () => {
  if (!chart) chart = echarts.init(chartRef.value)
  chart.setOption({
    tooltip: { trigger: 'axis' },
    legend: { data: ['VWAP', 'PRICE'] },
    xAxis: { type: 'category', data: time.value, axisLabel: { rotate: 45, color: '#aaa' } },
    yAxis: { type: 'value',min: function (value) {
      return Math.floor(value.min)
    }, name: '价格', axisLabel: { color: '#5cfaff' } },
    series: [
      { name: 'VWAP', type: 'line', data: vwa.value, smooth: true, symbol: 'none', lineStyle: { width: 2, color: '#4fbcff' } },
      { name: 'PRICE', type: 'line', data: price.value, smooth: true, symbol: 'none', lineStyle: { width: 2, color: '#ffb958' } }
    ]
  })
}

onMounted(() => {
  ws = new WebSocket('wss://ai.wyizd.com/mean-reversion-ws/')
  ws.onmessage = e => {
    try {
      const d = JSON.parse(e.data)
      if (Array.isArray(d.vwa) && Array.isArray(d.price) && Array.isArray(d.time)) {
        vwa.value = d.vwa.map(Number)
        price.value = d.price.map(Number)
        time.value = d.time
        renderChart()
      }
    } catch (err) { console.error(err) }
  }
})
onUnmounted(() => {
  ws?.close()
  chart?.dispose()
})
</script>