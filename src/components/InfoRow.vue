<template>
  <div class="flex items-center justify-between py-2 border-b border-white/5 last:border-0">
    <span class="text-sm text-gray-500">{{ label }}</span>
    <span class="text-sm font-mono text-gray-200">{{ value }}</span>
  </div>
</template>

<script setup>
defineProps({
  label: String,
  value: [String, Number]
})
</script>