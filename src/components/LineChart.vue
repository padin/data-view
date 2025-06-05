
  <template>
    <div ref="chartRef" style="min-width: 1200px; width: 100%; height: 600px;"></div>
  </template>
  
  <script setup>
  import { onMounted, onUnmounted, watch, ref } from 'vue'
  import * as echarts from 'echarts'
  
  const props = defineProps({
    chartData: {
      type: Array,
      required: true
    }
  })
  
  const chartRef = ref(null)
  let chart = null
  
  // 渲染图表
  const renderChart = () => {
  if (!chart) {
    chart = echarts.init(chartRef.value)
  }

  // 1. 分组：name => [data]
  const group = {}
  props.chartData.forEach(d => {
    if (!group[d.name]) group[d.name] = []
    group[d.name].push(d)
  })

  // 2. 统一时间轴（所有时间点，排序）
  const allTimes = Array.from(new Set(props.chartData.map(d => d.time))).sort()

  // 3. 构建 series，每条线用 allTimes 对齐（无数据时间填 null）
  const series = Object.entries(group).map(([name, arr]) => {
    const map = new Map(arr.map(d => [d.time, d.value]))
    return {
      name,
      type: 'line',
      smooth: true,
      showSymbol: false,
      data: allTimes.map(time => map.get(time) ?? null)
    }
  })

  // 4. 设置图表
  chart.setOption({
    tooltip: { trigger: 'axis' },
    legend: { data: Object.keys(group) },
    xAxis: {
      type: 'category',
      data: allTimes,
      axisLabel: {
        formatter: val => val.split(' ')[1] // 只显示时间部分
      }
    },
    yAxis: { type: 'value' },
    series
  })
}

  
  onMounted(() => {
    chart = echarts.init(chartRef.value)
    renderChart()
  })
  
  watch(() => props.chartData, () => {
    renderChart()
  }, { deep: true })
  
  onUnmounted(() => {
    chart?.dispose()
  })
  </script>
  