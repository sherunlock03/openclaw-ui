<template>
  <canvas ref="canvasRef" :height="height"></canvas>
</template>

<script setup>
import { ref, onMounted, watch, onUnmounted } from 'vue'
import { Chart, registerables } from 'chart.js'

Chart.register(...registerables)

const props = defineProps({
  data: {
    type: Array,
    default: () => []
  },
  color: {
    type: String,
    default: '#00d4ff'
  },
  height: {
    type: Number,
    default: 150
  }
})

const canvasRef = ref(null)
let chart = null

const createGradient = (ctx, color) => {
  const gradient = ctx.createLinearGradient(0, 0, 0, props.height)
  gradient.addColorStop(0, color + '40')
  gradient.addColorStop(1, color + '00')
  return gradient
}

onMounted(() => {
  if (!canvasRef.value) return
  
  const ctx = canvasRef.value.getContext('2d')
  const gradient = createGradient(ctx, props.color)
  
  chart = new Chart(ctx, {
    type: 'line',
    data: {
      labels: Array(60).fill(''),
      datasets: [{
        data: props.data,
        borderColor: props.color,
        backgroundColor: gradient,
        borderWidth: 2,
        fill: true,
        tension: 0.4,
        pointRadius: 0,
        pointHoverRadius: 4,
        pointHoverBackgroundColor: props.color,
      }]
    },
    options: {
      responsive: true,
      maintainAspectRatio: false,
      plugins: {
        legend: { display: false },
        tooltip: {
          backgroundColor: 'rgba(0, 0, 0, 0.8)',
          titleColor: '#fff',
          bodyColor: '#fff',
          borderColor: props.color,
          borderWidth: 1,
          displayColors: false,
          callbacks: {
            label: (ctx) => `${ctx.raw}%`
          }
        }
      },
      scales: {
        x: { display: false },
        y: {
          display: true,
          min: 0,
          max: 100,
          grid: {
            color: 'rgba(255, 255, 255, 0.05)',
            drawBorder: false
          },
          ticks: {
            color: 'rgba(255, 255, 255, 0.3)',
            font: { size: 10 },
            callback: (value) => value + '%'
          }
        }
      },
      interaction: {
        intersect: false,
        mode: 'index'
      },
      animation: {
        duration: 300
      }
    }
  })
})

watch(() => props.data, (newData) => {
  if (chart) {
    chart.data.datasets[0].data = newData
    chart.update('none')
  }
}, { deep: true })

onUnmounted(() => {
  if (chart) chart.destroy()
})
</script>