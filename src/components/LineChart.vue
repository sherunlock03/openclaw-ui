<template>
  <canvas ref="canvasRef"></canvas>
</template>

<script setup>
import { ref, onMounted, watch, onUnmounted } from 'vue'
import { Chart, registerables } from 'chart.js'

Chart.register(...registerables)

const props = defineProps({
  cpuData: { type: Array, default: () => [] },
  memData: { type: Array, default: () => [] },
  netData: { type: Array, default: () => [] }
})

const canvasRef = ref(null)
let chart = null

onMounted(() => {
  if (!canvasRef.value) return
  
  const ctx = canvasRef.value.getContext('2d')
  
  // 创建渐变
  const createGradient = (color1, color2) => {
    const gradient = ctx.createLinearGradient(0, 0, 0, 300)
    gradient.addColorStop(0, color1)
    gradient.addColorStop(1, color2)
    return gradient
  }

  chart = new Chart(ctx, {
    type: 'line',
    data: {
      labels: Array(60).fill(''),
      datasets: [
        {
          label: 'CPU',
          data: props.cpuData,
          borderColor: '#00d4ff',
          backgroundColor: createGradient('rgba(0, 212, 255, 0.3)', 'rgba(0, 212, 255, 0)'),
          borderWidth: 2,
          fill: true,
          tension: 0.4,
          pointRadius: 0,
          pointHoverRadius: 4,
          pointHoverBackgroundColor: '#00d4ff',
        },
        {
          label: 'Memory',
          data: props.memData,
          borderColor: '#a855f7',
          backgroundColor: createGradient('rgba(168, 85, 247, 0.3)', 'rgba(168, 85, 247, 0)'),
          borderWidth: 2,
          fill: true,
          tension: 0.4,
          pointRadius: 0,
          pointHoverRadius: 4,
          pointHoverBackgroundColor: '#a855f7',
        },
        {
          label: 'Network',
          data: props.netData,
          borderColor: '#00ff88',
          backgroundColor: createGradient('rgba(0, 255, 136, 0.2)', 'rgba(0, 255, 136, 0)'),
          borderWidth: 2,
          fill: true,
          tension: 0.4,
          pointRadius: 0,
          pointHoverRadius: 4,
          pointHoverBackgroundColor: '#00ff88',
        }
      ]
    },
    options: {
      responsive: true,
      maintainAspectRatio: false,
      interaction: {
        intersect: false,
        mode: 'index'
      },
      plugins: {
        legend: { display: false },
        tooltip: {
          backgroundColor: 'rgba(0, 20, 40, 0.9)',
          titleColor: '#00d4ff',
          bodyColor: '#ffffff',
          borderColor: 'rgba(0, 212, 255, 0.3)',
          borderWidth: 1,
          padding: 12,
          displayColors: true,
          callbacks: {
            label: (ctx) => `${ctx.dataset.label}: ${ctx.raw}%`
          }
        }
      },
      scales: {
        x: { 
          display: false
        },
        y: {
          display: true,
          min: 0,
          max: 100,
          grid: {
            color: 'rgba(255, 255, 255, 0.05)',
            drawBorder: false
          },
          ticks: {
            color: 'rgba(255, 255, 255, 0.4)',
            font: { size: 10 },
            callback: (value) => value + '%',
            stepSize: 25
          }
        }
      },
      animation: {
        duration: 300
      }
    }
  })
})

watch([() => props.cpuData, () => props.memData, () => props.netData], () => {
  if (chart) {
    chart.data.datasets[0].data = props.cpuData
    chart.data.datasets[1].data = props.memData
    chart.data.datasets[2].data = props.netData
    chart.update('none')
  }
}, { deep: true })

onUnmounted(() => {
  if (chart) chart.destroy()
})
</script>