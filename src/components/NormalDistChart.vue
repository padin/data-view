<template>
  <div ref="chartRef" style="min-width: 1200px; width: 100%; height: 600px;"></div>
</template>

<script setup>
import { onMounted, onUnmounted, ref, nextTick } from 'vue'
import * as echarts from 'echarts'

const chartRef = ref(null)
let chart = null
const chartData = ref([])

let ws = null

const renderChart = () => {
  if (!chart) {
    chart = echarts.init(chartRef.value)
  }
  const xLabels = chartData.value.map(
    d => `${Number(d.from).toFixed(0)}→${Number(d.to).toFixed(0)}`
  )
  const yData = chartData.value.map(d => d.size)
  chart.setOption({
    tooltip: { trigger: 'axis' },
    xAxis: { type: 'category', data: xLabels, axisLabel: { rotate: 35 } },
    yAxis: { type: 'value', name: 'size' },
    series: [{
      name: 'size',
      type: 'bar',
      data: yData,
      itemStyle: { color: '#53d3ff' }
    }]
  })
}

onMounted(() => {
  chart = echarts.init(chartRef.value)
  ws = new WebSocket('wss://ai.wyizd.com/normal-dist-ws/')
  ws.onmessage = (event) => {
    try {
      const resp = JSON.parse(event.data)
      // 你可以根据后端ws实际结构调整
      if (Array.isArray(resp)) {
        chartData.value = resp
      } else if (resp && Array.isArray(resp.data)) {
        chartData.value = resp.data
      }
      nextTick(renderChart)
    } catch (e) {
      console.error('WS数据解析出错:', e)
    }
  }
  ws.onerror = (e) => { console.error('normal-dist ws error', e) }
  ws.onclose = () => { /* 可选：重连逻辑 */ }
  renderChart()
})

onUnmounted(() => {
  ws?.close()
  chart?.dispose()
})
</script>