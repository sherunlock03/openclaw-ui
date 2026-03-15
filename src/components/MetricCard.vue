<template>
  <div class="glow-border rounded-2xl bg-cyber-dark/50 backdrop-blur-sm p-4 card-hover"
       :class="'glow-border-' + color">
    <div class="flex items-center justify-between mb-2">
      <span class="text-sm text-gray-400">{{ title }}</span>
      <div :class="iconBgClass" class="w-8 h-8 rounded-lg flex items-center justify-center">
        <component :is="iconComponent" class="w-4 h-4" />
      </div>
    </div>
    <div class="flex items-baseline gap-1">
      <span class="metric-value" :class="textColorClass">{{ value }}</span>
      <span class="text-sm text-gray-500">{{ unit }}</span>
    </div>
    <div class="progress-bar mt-3">
      <div class="progress-bar-fill" :style="{ width: percentage + '%' }"></div>
    </div>
  </div>
</template>

<script setup>
import { computed, h } from 'vue'

const props = defineProps({
  title: String,
  value: [Number, String],
  unit: String,
  color: String,
  icon: String
})

const percentage = computed(() => {
  const num = parseFloat(props.value)
  if (isNaN(num)) return 0
  if (typeof props.value === 'string' && props.value.includes('h')) return 100
  return Math.min(num, 100)
})

const colorClass = computed(() => {
  const colors = {
    blue: 'cyber-blue',
    purple: 'cyber-purple',
    green: 'cyber-green',
    orange: 'cyber-orange',
    red: 'cyber-red'
  }
  return colors[props.color] || 'cyber-blue'
})

const textColorClass = computed(() => `text-${colorClass.value}`)
const iconBgClass = computed(() => `bg-${colorClass.value}/20 text-${colorClass.value}`)

const iconComponent = computed(() => {
  const icons = {
    cpu: h('svg', { class: 'w-4 h-4', fill: 'currentColor', viewBox: '0 0 20 20' }, [
      h('path', { 'fill-rule': 'evenodd', d: 'M5 2a1 1 0 011 1v1h1a1 1 0 010 2H6v1a1 1 0 01-2 0V6H3a1 1 0 010-2h1V3a1 1 0 011-1zm0 10a1 1 0 011 1v1h1a1 1 0 110 2H6v1a1 1 0 11-2 0v-1H3a1 1 0 110-2h1v-1a1 1 0 011-1zM12 2a1 1 0 01.967.744L14.146 7.2 17.5 9.134a1 1 0 010 1.732l-3.354 1.935-1.18 4.455a1 1 0 01-1.933 0L9.854 12.8 6.5 10.866a1 1 0 010-1.732l3.354-1.935 1.18-4.455A1 1 0 0112 2z', 'clip-rule': 'evenodd' })
    ]),
    memory: h('svg', { class: 'w-4 h-4', fill: 'currentColor', viewBox: '0 0 20 20' }, [
      h('path', { d: 'M3 4a1 1 0 011-1h12a1 1 0 011 1v2a1 1 0 01-1 1H4a1 1 0 01-1-1V4zM3 10a1 1 0 011-1h6a1 1 0 011 1v6a1 1 0 01-1 1H4a1 1 0 01-1-1v-6zM14 9a1 1 0 00-1 1v6a1 1 0 001 1h2a1 1 0 001-1v-6a1 1 0 00-1-1h-2z' })
    ]),
    disk: h('svg', { class: 'w-4 h-4', fill: 'currentColor', viewBox: '0 0 20 20' }, [
      h('path', { d: 'M2 6a2 2 0 012-2h5l2 2h5a2 2 0 012 2v6a2 2 0 01-2 2H4a2 2 0 01-2-2V6z' })
    ]),
    users: h('svg', { class: 'w-4 h-4', fill: 'currentColor', viewBox: '0 0 20 20' }, [
      h('path', { d: 'M9 6a3 3 0 11-6 0 3 3 0 016 0zM17 6a3 3 0 11-6 0 3 3 0 016 0zM12.93 17c.046-.327.07-.66.07-1a6.97 6.97 0 00-1.5-4.33A5 5 0 0119 16v1h-6.07zM6 11a5 5 0 015 5v1H1v-1a5 5 0 015-5z' })
    ]),
    clock: h('svg', { class: 'w-4 h-4', fill: 'currentColor', viewBox: '0 0 20 20' }, [
      h('path', { 'fill-rule': 'evenodd', d: 'M10 18a8 8 0 100-16 8 8 0 000 16zm1-12a1 1 0 10-2 0v4a1 1 0 00.293.707l2.828 2.829a1 1 0 101.415-1.415L11 9.586V6z', 'clip-rule': 'evenodd' })
    ])
  }
  return icons[props.icon] || icons.cpu
})
</script>