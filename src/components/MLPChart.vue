<template>
  <div ref="chartRef" style="min-width: 1200px; width: 100%; height: 600px;"></div>
</template>

<script setup>
import { onMounted, onUnmounted, ref } from 'vue'
import * as echarts from 'echarts'

const chartRef = ref(null)
let chart = null
const chartData = ref([])

let ws = null
const WINDOW_SIZE = 1000

function mergeData(existing, incoming) {
  const map = new Map()
  ;[...existing, ...incoming].forEach(item => {
    const key = `${item.name}__${item.time}`
    map.set(key, item)
  })
  // 按时间排序，只保留最新 WINDOW_SIZE*5 条
  return Array.from(map.values())
    .sort((a, b) => new Date(a.time) - new Date(b.time))
    .slice(-WINDOW_SIZE * 5)
}

const renderChart = () => {
  if (!chart) {
    chart = echarts.init(chartRef.value)
  }
  const group = {}
  chartData.value.forEach(d => {
    if (!group[d.name]) group[d.name] = []
    group[d.name].push(d)
  })
  const allTimes = Array.from(new Set(chartData.value.map(d => d.time))).sort()
  const series = Object.entries(group).map(([name, arr]) => {
    const arrSorted = arr.sort((a, b) => new Date(a.time) - new Date(b.time)).slice(-WINDOW_SIZE)
    const map = new Map(arrSorted.map(d => [d.time, d.value]))
    return {
      name,
      type: 'line',
      smooth: true,
      showSymbol: false,
      data: allTimes.map(time => map.get(time) ?? null)
    }
  })
  chart.setOption({
    tooltip: { trigger: 'axis' },
    legend: { data: Object.keys(group) },
    xAxis: {
      type: 'category',
      data: allTimes,
      axisLabel: {
        formatter: val => val.split(' ')[1]
      }
    },
    yAxis: { type: 'value' },
    series,
    grid: { left: 60, right: 40, top: 60, bottom: 60 }
  })
}

onMounted(() => {
  chart = echarts.init(chartRef.value)
  renderChart()
  ws = new WebSocket('wss://ai.wyizd.com/mlp-analysis-ws/')
  ws.onmessage = (event) => {
    try {
      const arr = JSON.parse(event.data)
      let flatArr = []
      if (Array.isArray(arr) && arr.length > 0 && Array.isArray(arr[0])) {
        flatArr = arr.flat()
      } else if (Array.isArray(arr)) {
        flatArr = arr
      } else if (arr && typeof arr === 'object') {
        flatArr = [arr]
      }
      const incoming = flatArr.map(data => ({
        name: data.name,
        value: data.upRate ?? data.value,
        time: data.updateTime ?? data.time
      }))
      chartData.value = mergeData(chartData.value, incoming)
      renderChart()
    } catch (e) {
      console.error('Error parsing message:', e)
    }
  }
  ws.onerror = () => { console.error('WebSocket error') }
})

onUnmounted(() => {
  chart?.dispose()
  ws?.close()
})
</script>