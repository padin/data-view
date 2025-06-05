<template>
  <div class="hero-bg">
    <div class="container">
      <header class="header-glass">
        <h1>
          <span class="title-gradient neon-flicker">MLP神经网络指标</span>
        </h1>
        <div class="powered">
          <span>powered by</span>
          <a href="mailto:padin@foxmail.com" class="author">padin@foxmail.com</a>
        </div>
      </header>
      <section class="chart-section">
        <div class="chart-card neon-glow">
          <LineChart :chartData="mergedChartData" />
        </div>
      </section>
      <div class="warn-tip">
        <span class="warn-emoji">⚠️</span>
        数据仅供学习交流使用，不构成任何投资建议。
      </div>
    </div>
  </div>
</template>

<script setup>
import LineChart from './components/LineChart.vue'
import { ref, onMounted, onUnmounted, computed } from 'vue'

const WINDOW_SIZE = 1000
const groupedData = ref({})

const initData = []

initData.forEach(item => {
  if (!groupedData.value[item.name]) {
    groupedData.value[item.name] = []
  }
  groupedData.value[item.name].push(item)
})

const mergedChartData = computed(() => {
  return Object.values(groupedData.value).flat()
})

let ws = null

onMounted(() => {
  ws = new WebSocket('wss://ai.wyizd.com/ws/')
  ws.onmessage = (event) => {
    try {
      const arr = JSON.parse(event.data)
      if (Array.isArray(arr)) {
        const newGroup = { ...groupedData.value }
        arr.forEach(data => {
          const item = {
            name: data.name,
            value: data.upRate, // 可自定义展示字段
            time: data.updateTime
          }
          if (!newGroup[item.name]) {
            newGroup[item.name] = []
          }
          newGroup[item.name] = [
            ...newGroup[item.name].slice(-WINDOW_SIZE + 1),
            item
          ]
        })
        groupedData.value = newGroup
      } else if (arr && typeof arr === 'object') {
        // 兼容单对象
        const data = arr
        const item = {
          name: data.name,
          value: data.upRate,
          time: data.updateTime
        }
        const newGroup = { ...groupedData.value }
        if (!newGroup[item.name]) {
          newGroup[item.name] = []
        }
        newGroup[item.name] = [
          ...newGroup[item.name].slice(-WINDOW_SIZE + 1),
          item
        ]
        groupedData.value = newGroup
      }
    } catch (e) {
      console.error('Error parsing message:', e)
    }
  }
  ws.onerror = () => {
    console.error('WebSocket error')
  }
})

onUnmounted(() => {
  ws?.close()
})
</script>

<style scoped>
:global(body) {
  margin: 0;
  padding: 0;
  overflow-x: hidden;
  background: #0f2027;
}

.hero-bg {
  min-height: 100vh;
  width: 100%;
  background: linear-gradient(135deg, #0f2027 0%, #2c5364 100%);
  display: flex;
  align-items: flex-start;
  justify-content: center;
  padding: 0;
  overflow-x: hidden;
}

.container {
  width: 100%;
  max-width: 1280px;
  margin: 0 auto;
  /* 顶部留白减少 */
  padding: 16px 20px 20px 20px;
  box-sizing: border-box;
  display: flex;
  flex-direction: column;
  align-items: center;
}

.header-glass {
  background: rgba(255,255,255,0.08);
  box-shadow: 0 8px 32px 0 rgba(31,38,135,0.18);
  backdrop-filter: blur(8px);
  border-radius: 24px;
  padding: 18px 18px 12px 18px;
  margin-bottom: 14px;
  text-align: center;
  border: 1.5px solid rgba(255,255,255,0.15);
  width: 100%;
  max-width: 900px;
  transition: box-shadow 0.2s;
}

.header-glass:hover {
  box-shadow: 0 8px 54px 0 rgba(0,0,0,0.26);
}

.title-gradient {
  background: linear-gradient(90deg, #6ef195 20%, #53d3ff 60%, #f3c1ff 100%);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  font-size: 2.1rem;
  font-weight: 800;
  letter-spacing: 1px;
  line-height: 1.2;
  display: inline-block;
  text-shadow: 0 4px 24px rgba(122,245,255,0.35);
  filter: brightness(1.08) drop-shadow(0 0 10px #6ef19566);
}

.neon-flicker {
  animation: neon-flicker 1.8s infinite alternate;
}
@keyframes neon-flicker {
  0%, 100% {filter: brightness(1.08) drop-shadow(0 0 10px #6ef19566);}
  50% {filter: brightness(1.4) drop-shadow(0 0 18px #53d3ff99);}
}

.powered {
  display: flex;
  justify-content: center;
  align-items: center;
  margin-top: 10px;
  font-size: 1.05rem;
  color: #b0eaff;
  gap: 10px;
  font-weight: 500;
}
.author {
  color: #ffe3a3;
  font-weight: bold;
  text-shadow: 0 2px 6px rgba(0,0,0,0.18);
}

.chart-section {
  width: 100%;
  display: flex;
  justify-content: center;
}

.chart-card {
  width: 100%;
  max-width: 1100px;
  min-width: 260px;
  height: 60vh;
  min-height: 260px;
  border-radius: 28px;
  background: rgba(255,255,255,0.14);
  box-shadow: 0 8px 40px 0 rgba(64,255,255,0.22), 0 2.5px 8px 0 rgba(0,0,0,0.07);
  padding: 8px 8px 4px 8px;
  margin-bottom: 16px;
  margin-top: 6px;
  transition: box-shadow 0.2s, background 0.4s;
  /* 彻底移除滚动条 */
  overflow: hidden;
  display: flex;
  align-items: center;
  justify-content: center;
  position: relative;
}

.chart-card:hover {
  box-shadow: 0 12px 80px 0 rgba(80,230,255,0.33);
  background: rgba(255,255,255,0.21);
}

.neon-glow {
  box-shadow:
    0 0 28px 0 #53d3ff88,
    0 0 60px 10px #6ef19520,
    0 0 12px 0 #f3c1ff66;
  animation: neon-glow 2.7s infinite alternate;
}
@keyframes neon-glow {
  0% {box-shadow:
    0 0 28px 0 #53d3ff88,
    0 0 60px 10px #6ef19520,
    0 0 12px 0 #f3c1ff66;}
  100% {box-shadow:
    0 0 48px 0 #53d3ff,
    0 0 120px 20px #6ef19544,
    0 0 28px 0 #f3c1ffbb;}
}

.warn-tip {
  margin-top: 16px;
  text-align: center;
  color: #ff5566;
  font-weight: 700;
  font-size: 1.08rem;
  background: rgba(255,255,255,0.12);
  padding: 13px 7px;
  border-radius: 14px;
  letter-spacing: 1px;
  box-shadow: 0 1.5px 4px 0 rgba(0,0,0,0.07);
  display: flex;
  justify-content: center;
  align-items: center;
  gap: 8px;
  max-width: 700px;
  margin-left: auto;
  margin-right: auto;
}

.warn-emoji {
  font-size: 1.25em;
  margin-right: 4px;
}

/* 响应式布局 */
@media (max-width: 1200px) {
  .container { max-width: 99vw; }
  .chart-card { max-width: 97vw; }
}
@media (max-width: 900px) {
  .chart-card {
    max-width: 99vw;
    min-width: 0;
    padding: 4px;
    height: 46vw;
    min-height: 170px;
  }
  .container {
    padding: 10px 3vw;
  }
  .header-glass {
    padding: 10px 5vw;
    max-width: 98vw;
  }
  .title-gradient {
    font-size: 1.2rem;
  }
}
@media (max-width: 600px) {
  .header-glass {
    padding: 5px 2vw;
  }
  .container {
    padding: 3vw 0 0 0;
  }
  .title-gradient {
    font-size: 0.94rem;
  }
  .warn-tip {
    font-size: 0.83rem;
    padding: 8px 2px;
    border-radius: 9px;
  }
  .chart-card {
    height: 54vw;
    min-height: 120px;
    max-width: 100vw;
    padding: 1px;
  }
}
</style>