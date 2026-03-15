<template>
  <div class="min-h-screen grid-bg">
    <!-- Header -->
    <header class="border-b border-cyber-blue/20 bg-cyber-darker/80 backdrop-blur-xl sticky top-0 z-50">
      <div class="px-6 py-4 flex items-center justify-between">
        <div class="flex items-center gap-4">
          <div class="flex items-center gap-3">
            <div class="w-10 h-10 rounded-xl bg-gradient-to-br from-cyber-blue to-cyber-purple flex items-center justify-center animate-float">
              <svg class="w-6 h-6 text-white" viewBox="0 0 24 24" fill="currentColor">
                <path d="M8 5v14l11-7z"/>
              </svg>
            </div>
            <div>
              <h1 class="text-xl font-bold text-gradient">OpenClaw Monitor</h1>
              <p class="text-xs text-gray-500">Gateway Dashboard</p>
            </div>
          </div>
        </div>
        
        <div class="flex items-center gap-6">
          <!-- Connection Status -->
          <div class="flex items-center gap-2">
            <div class="status-dot" :class="connected ? 'online' : 'offline'"></div>
            <span class="text-sm text-gray-400">{{ connected ? 'Connected' : 'Disconnected' }}</span>
          </div>
          
          <!-- Time -->
          <div class="text-sm font-mono text-gray-400">
            {{ currentTime }}
          </div>
          
          <!-- Settings -->
          <button @click="showSettings = true" class="p-2 rounded-lg hover:bg-white/5 transition-colors">
            <svg class="w-5 h-5 text-gray-400" fill="none" stroke="currentColor" viewBox="0 0 24 24">
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M10.325 4.317c.426-1.756 2.924-1.756 3.35 0a1.724 1.724 0 002.573 1.066c1.543-.94 3.31.826 2.37 2.37a1.724 1.724 0 001.065 2.572c1.756.426 1.756 2.924 0 3.35a1.724 1.724 0 00-1.066 2.573c.94 1.543-.826 3.31-2.37 2.37a1.724 1.724 0 00-2.572 1.065c-.426 1.756-2.924 1.756-3.35 0a1.724 1.724 0 00-2.573-1.066c-1.543.94-3.31-.826-2.37-2.37a1.724 1.724 0 00-1.065-2.572c-1.756-.426-1.756-2.924 0-3.35a1.724 1.724 0 001.066-2.573c-.94-1.543.826-3.31 2.37-2.37.996.608 2.296.07 2.572-1.065z"/>
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M15 12a3 3 0 11-6 0 3 3 0 016 0z"/>
            </svg>
          </button>
        </div>
      </div>
    </header>

    <!-- Main Dashboard -->
    <main class="p-6">
      <!-- Top Stats Row -->
      <div class="grid grid-cols-5 gap-4 mb-6">
        <!-- CPU -->
        <MetricCard
          title="CPU"
          :value="systemStats.cpu"
          unit="%"
          :color="'blue'"
          icon="cpu"
        />
        <!-- Memory -->
        <MetricCard
          title="Memory"
          :value="systemStats.memory"
          unit="%"
          :color="'purple'"
          icon="memory"
        />
        <!-- Disk -->
        <MetricCard
          title="Disk"
          :value="systemStats.disk"
          unit="%"
          :color="'green'"
          icon="disk"
        />
        <!-- Active Sessions -->
        <MetricCard
          title="Sessions"
          :value="gatewayStats.sessions"
          unit=""
          :color="'orange'"
          icon="users"
        />
        <!-- Uptime -->
        <MetricCard
          title="Uptime"
          :value="formatUptime(gatewayStats.uptime)"
          unit=""
          :color="'blue'"
          icon="clock"
        />
      </div>

      <!-- Main Grid -->
      <div class="grid grid-cols-12 gap-6">
        <!-- Left Column - Charts & Logs -->
        <div class="col-span-8 space-y-6">
          <!-- Charts Row -->
          <div class="grid grid-cols-2 gap-6">
            <!-- CPU Chart -->
            <div class="glow-border rounded-2xl bg-cyber-dark/50 backdrop-blur-sm p-4">
              <h3 class="text-sm font-medium text-gray-400 mb-4">CPU Usage (Real-time)</h3>
              <LineChart :data="cpuHistory" color="#00d4ff" :height="120" />
            </div>
            <!-- Memory Chart -->
            <div class="glow-border-purple rounded-2xl bg-cyber-dark/50 backdrop-blur-sm p-4">
              <h3 class="text-sm font-medium text-gray-400 mb-4">Memory Usage (Real-time)</h3>
              <LineChart :data="memoryHistory" color="#b847ff" :height="120" />
            </div>
          </div>

          <!-- Logs Terminal -->
          <div class="terminal rounded-2xl overflow-hidden">
            <div class="terminal-header px-4 py-3 flex items-center justify-between">
              <div class="flex items-center gap-2">
                <div class="flex gap-1.5">
                  <div class="w-3 h-3 rounded-full bg-red-500"></div>
                  <div class="w-3 h-3 rounded-full bg-yellow-500"></div>
                  <div class="w-3 h-3 rounded-full bg-green-500"></div>
                </div>
                <span class="text-sm text-gray-400 ml-2">System Logs</span>
              </div>
              <div class="flex items-center gap-2">
                <button @click="clearLogs" class="text-xs text-gray-500 hover:text-gray-300 px-2 py-1 rounded hover:bg-white/5">
                  Clear
                </button>
                <div class="text-xs text-gray-500 font-mono">{{ logs.length }} entries</div>
              </div>
            </div>
            <div class="p-4 h-64 overflow-y-auto font-mono text-sm">
              <div v-for="(log, i) in logs" :key="i" class="log-entry py-0.5">
                <span class="log-time">{{ log.time }}</span>
                <span :class="'log-level-' + log.level"> [{{ log.level.toUpperCase() }}] </span>
                <span class="text-gray-300">{{ log.message }}</span>
              </div>
            </div>
          </div>

          <!-- Activity Timeline -->
          <div class="glow-border rounded-2xl bg-cyber-dark/50 backdrop-blur-sm p-4">
            <h3 class="text-sm font-medium text-gray-400 mb-4">Recent Activity</h3>
            <div class="space-y-3">
              <div v-for="(activity, i) in recentActivities" :key="i" 
                   class="flex items-start gap-3 p-3 rounded-lg bg-white/5 hover:bg-white/10 transition-colors">
                <div :class="getActivityColor(activity.type)" class="w-8 h-8 rounded-lg flex items-center justify-center shrink-0">
                  <component :is="getActivityIcon(activity.type)" class="w-4 h-4" />
                </div>
                <div class="flex-1 min-w-0">
                  <div class="text-sm text-gray-200">{{ activity.title }}</div>
                  <div class="text-xs text-gray-500">{{ activity.description }}</div>
                </div>
                <div class="text-xs text-gray-500">{{ activity.time }}</div>
              </div>
            </div>
          </div>
        </div>

        <!-- Right Column - Status Panels -->
        <div class="col-span-4 space-y-6">
          <!-- Gateway Info -->
          <div class="glow-border rounded-2xl bg-cyber-dark/50 backdrop-blur-sm p-4">
            <h3 class="text-sm font-medium text-gray-400 mb-4 flex items-center gap-2">
              <svg class="w-4 h-4 text-cyber-blue" fill="currentColor" viewBox="0 0 20 20">
                <path d="M2 5a2 2 0 012-2h12a2 2 0 012 2v2a2 2 0 01-2 2H4a2 2 0 01-2-2V5zm14 1a1 1 0 11-2 0 1 1 0 012 0zM4 13a2 2 0 012-2h12a2 2 0 012 2v2a2 2 0 01-2 2H6a2 2 0 01-2-2v-2zm14 1a1 1 0 11-2 0 1 1 0 012 0z"/>
              </svg>
              Gateway Status
            </h3>
            <div class="space-y-3">
              <InfoRow label="Version" :value="gatewayStats.version" />
              <InfoRow label="Port" :value="gatewayStats.port" />
              <InfoRow label="Host" :value="gatewayStats.host" />
              <InfoRow label="PID" :value="gatewayStats.pid" />
              <InfoRow label="Node" :value="gatewayStats.nodeVersion" />
            </div>
          </div>

          <!-- Active Channels -->
          <div class="glow-border-green rounded-2xl bg-cyber-dark/50 backdrop-blur-sm p-4">
            <h3 class="text-sm font-medium text-gray-400 mb-4 flex items-center gap-2">
              <svg class="w-4 h-4 text-cyber-green" fill="currentColor" viewBox="0 0 20 20">
                <path d="M2 5a2 2 0 012-2h7a2 2 0 012 2v4a2 2 0 01-2 2H9l-3 3v-3H4a2 2 0 01-2-2V5z"/>
                <path d="M15 7v2a4 4 0 01-4 4H9.828l-1.766 1.767c.28.149.599.233.938.233h2l3 3v-3h2a2 2 0 002-2V9a2 2 0 00-2-2h-1z"/>
              </svg>
              Active Channels
            </h3>
            <div class="space-y-2">
              <div v-for="channel in channels" :key="channel.id" 
                   class="flex items-center justify-between p-2 rounded-lg bg-white/5">
                <div class="flex items-center gap-2">
                  <div class="status-dot online"></div>
                  <span class="text-sm">{{ channel.name }}</span>
                </div>
                <span class="text-xs text-gray-500">{{ channel.type }}</span>
              </div>
            </div>
          </div>

          <!-- Skills Status -->
          <div class="glow-border-purple rounded-2xl bg-cyber-dark/50 backdrop-blur-sm p-4">
            <h3 class="text-sm font-medium text-gray-400 mb-4 flex items-center gap-2">
              <svg class="w-4 h-4 text-cyber-purple" fill="currentColor" viewBox="0 0 20 20">
                <path fill-rule="evenodd" d="M11.3 1.046A1 1 0 0112 2v5h4a1 1 0 01.82 1.573l-7 10A1 1 0 018 18v-5H4a1 1 0 01-.82-1.573l7-10a1 1 0 011.12-.38z" clip-rule="evenodd"/>
              </svg>
              Skills & Agents
            </h3>
            <div class="space-y-2">
              <div v-for="skill in skills" :key="skill.name" class="flex items-center justify-between">
                <span class="text-sm text-gray-300">{{ skill.name }}</span>
                <span :class="skill.status === 'active' ? 'text-cyber-green' : 'text-gray-500'" class="text-xs">
                  {{ skill.status }}
                </span>
              </div>
            </div>
          </div>

          <!-- Cron Jobs -->
          <div class="glow-border rounded-2xl bg-cyber-dark/50 backdrop-blur-sm p-4">
            <h3 class="text-sm font-medium text-gray-400 mb-4 flex items-center gap-2">
              <svg class="w-4 h-4 text-cyber-orange" fill="currentColor" viewBox="0 0 20 20">
                <path fill-rule="evenodd" d="M10 18a8 8 0 100-16 8 8 0 000 16zm1-12a1 1 0 10-2 0v4a1 1 0 00.293.707l2.828 2.829a1 1 0 101.415-1.415L11 9.586V6z" clip-rule="evenodd"/>
              </svg>
              Scheduled Tasks
            </h3>
            <div class="space-y-2">
              <div v-for="cron in cronJobs" :key="cron.name" class="flex items-center justify-between p-2 rounded-lg bg-white/5">
                <div>
                  <div class="text-sm">{{ cron.name }}</div>
                  <div class="text-xs text-gray-500">{{ cron.schedule }}</div>
                </div>
                <div class="status-dot" :class="cron.enabled ? 'online' : 'offline'"></div>
              </div>
            </div>
          </div>
        </div>
      </div>
    </main>

    <!-- Settings Modal -->
    <div v-if="showSettings" class="fixed inset-0 bg-black/50 backdrop-blur-sm z-50 flex items-center justify-center">
      <div class="glow-border rounded-2xl bg-cyber-dark p-6 w-full max-w-md">
        <h2 class="text-lg font-semibold mb-4">Settings</h2>
        <div class="space-y-4">
          <div>
            <label class="block text-sm text-gray-400 mb-2">Gateway URL</label>
            <input v-model="settings.url" type="text" 
                   class="w-full px-4 py-2 rounded-lg bg-white/5 border border-white/10 focus:border-cyber-blue focus:outline-none" />
          </div>
          <div>
            <label class="block text-sm text-gray-400 mb-2">Token</label>
            <input v-model="settings.token" type="password" 
                   class="w-full px-4 py-2 rounded-lg bg-white/5 border border-white/10 focus:border-cyber-blue focus:outline-none" />
          </div>
        </div>
        <div class="flex justify-end gap-3 mt-6">
          <button @click="showSettings = false" class="px-4 py-2 rounded-lg text-gray-400 hover:text-white transition-colors">
            Cancel
          </button>
          <button @click="connect" class="px-4 py-2 rounded-lg bg-gradient-to-r from-cyber-blue to-cyber-purple text-white font-medium">
            Connect
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, onUnmounted, computed, h } from 'vue'
import MetricCard from './components/MetricCard.vue'
import LineChart from './components/LineChart.vue'
import InfoRow from './components/InfoRow.vue'

// State
const connected = ref(false)
const showSettings = ref(false)
const currentTime = ref('')

const settings = ref({
  url: 'ws://127.0.0.1:18789',
  token: ''
})

const systemStats = ref({
  cpu: 0,
  memory: 0,
  disk: 0
})

const gatewayStats = ref({
  version: 'v2026.3.12',
  port: 18789,
  host: '127.0.0.1',
  pid: 68832,
  nodeVersion: 'v22.22.1',
  uptime: 0,
  sessions: 1
})

const cpuHistory = ref([])
const memoryHistory = ref([])

const logs = ref([
  { time: '19:23:45', level: 'info', message: 'Gateway started on port 18789' },
  { time: '19:23:46', level: 'success', message: 'WebSocket connection established' },
  { time: '19:24:01', level: 'info', message: 'Session created: main' },
  { time: '19:25:33', level: 'info', message: 'Heartbeat received' },
  { time: '19:30:00', level: 'info', message: 'Health check passed' },
])

const channels = ref([
  { id: 1, name: 'feishu', type: 'messaging', status: 'connected' },
  { id: 2, name: 'main', type: 'session', status: 'active' },
])

const skills = ref([
  { name: 'feishu-doc', status: 'active' },
  { name: 'feishu-bitable', status: 'active' },
  { name: 'feishu-im', status: 'active' },
  { name: 'weather', status: 'idle' },
  { name: 'github', status: 'active' },
])

const cronJobs = ref([
  { name: 'Heartbeat', schedule: '*/5 * * * *', enabled: true },
  { name: 'Health Check', schedule: '*/1 * * * *', enabled: true },
])

const recentActivities = ref([
  { type: 'message', title: 'New message from 知何似', description: '红米k50u如何刷入Windows...', time: '2m ago' },
  { type: 'tool', title: 'Tool executed: web_search', description: 'Search for Windows on ARM...', time: '5m ago' },
  { type: 'session', title: 'Session created', description: 'New session initialized', time: '10m ago' },
  { type: 'system', title: 'Gateway health check', description: 'All systems operational', time: '15m ago' },
])

let ws = null
let timeInterval = null
let statsInterval = null

// Methods
const updateTime = () => {
  const now = new Date()
  currentTime.value = now.toLocaleTimeString('zh-CN', { hour12: false })
}

const simulateStats = () => {
  // Simulate real-time stats (would be replaced with actual API calls)
  systemStats.value.cpu = Math.floor(Math.random() * 30 + 10)
  systemStats.value.memory = Math.floor(Math.random() * 20 + 40)
  systemStats.value.disk = 67

  // Update history
  cpuHistory.value.push(systemStats.value.cpu)
  memoryHistory.value.push(systemStats.value.memory)

  if (cpuHistory.value.length > 60) cpuHistory.value.shift()
  if (memoryHistory.value.length > 60) memoryHistory.value.shift()

  gatewayStats.value.uptime++
}

const connect = () => {
  showSettings.value = false
  connected.value = true
  addLog('info', `Connecting to ${settings.value.url}...`)
  
  // Simulate connection (would be replaced with actual WebSocket)
  setTimeout(() => {
    addLog('success', 'Connected to Gateway')
  }, 500)
}

const addLog = (level, message) => {
  const now = new Date()
  logs.value.unshift({
    time: now.toLocaleTimeString('zh-CN', { hour12: false }),
    level,
    message
  })
  if (logs.value.length > 100) logs.value.pop()
}

const clearLogs = () => {
  logs.value = []
}

const formatUptime = (seconds) => {
  const h = Math.floor(seconds / 3600)
  const m = Math.floor((seconds % 3600) / 60)
  const s = seconds % 60
  return `${h}h ${m}m ${s}s`
}

const getActivityColor = (type) => {
  const colors = {
    message: 'bg-cyber-blue/20 text-cyber-blue',
    tool: 'bg-cyber-purple/20 text-cyber-purple',
    session: 'bg-cyber-green/20 text-cyber-green',
    system: 'bg-cyber-orange/20 text-cyber-orange'
  }
  return colors[type] || 'bg-gray-500/20 text-gray-500'
}

const getActivityIcon = (type) => {
  const icons = {
    message: h('svg', { class: 'w-4 h-4', fill: 'currentColor', viewBox: '0 0 20 20' }, 
      h('path', { d: 'M2 5a2 2 0 012-2h7a2 2 0 012 2v4a2 2 0 01-2 2H9l-3 3v-3H4a2 2 0 01-2-2V5z' })),
    tool: h('svg', { class: 'w-4 h-4', fill: 'currentColor', viewBox: '0 0 20 20' },
      h('path', { 'fill-rule': 'evenodd', d: 'M11.3 1.046A1 1 0 0112 2v5h4a1 1 0 01.82 1.573l-7 10A1 1 0 018 18v-5H4a1 1 0 01-.82-1.573l7-10a1 1 0 011.12-.38z', 'clip-rule': 'evenodd' })),
    session: h('svg', { class: 'w-4 h-4', fill: 'currentColor', viewBox: '0 0 20 20' },
      h('path', { d: 'M9 6a3 3 0 11-6 0 3 3 0 016 0zM17 6a3 3 0 11-6 0 3 3 0 016 0zM12.93 17c.046-.327.07-.66.07-1a6.97 6.97 0 00-1.5-4.33A5 5 0 0119 16v1h-6.07zM6 11a5 5 0 015 5v1H1v-1a5 5 0 015-5z' })),
    system: h('svg', { class: 'w-4 h-4', fill: 'currentColor', viewBox: '0 0 20 20' },
      h('path', { 'fill-rule': 'evenodd', d: 'M11.49 3.17c-.38-1.56-2.6-1.56-2.98 0a1.532 1.532 0 01-2.286.948c-1.372-.836-2.942.734-2.106 2.106.54.886.061 2.042-.947 2.287-1.561.379-1.561 2.6 0 2.978a1.532 1.532 0 01.947 2.287c-.836 1.372.734 2.942 2.106 2.106a1.532 1.532 0 012.287.947c.379 1.561 2.6 1.561 2.978 0a1.533 1.533 0 012.287-.947c1.372.836 2.942-.734 2.106-2.106a1.533 1.533 0 01.947-2.287c1.561-.379 1.561-2.6 0-2.978a1.532 1.532 0 01-.947-2.287c.836-1.372-.734-2.942-2.106-2.106a1.532 1.532 0 01-2.287-.947zM10 13a3 3 0 100-6 3 3 0 000 6z', 'clip-rule': 'evenodd' }))
  }
  return icons[type] || icons.system
}

// Lifecycle
onMounted(() => {
  updateTime()
  timeInterval = setInterval(updateTime, 1000)
  statsInterval = setInterval(simulateStats, 1000)
  
  // Auto-connect with token from URL hash
  const hash = window.location.hash
  const tokenMatch = hash.match(/token=([^&]+)/)
  if (tokenMatch) {
    settings.value.token = tokenMatch[1]
    connect()
  }
})

onUnmounted(() => {
  if (timeInterval) clearInterval(timeInterval)
  if (statsInterval) clearInterval(statsInterval)
  if (ws) ws.close()
})
</script>