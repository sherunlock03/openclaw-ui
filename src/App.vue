<template>
  <div class="min-h-screen bg-grid">
    <!-- Header -->
    <header class="header-bar">
      <div class="flex items-center gap-4">
        <div class="w-10 h-10 rounded-lg bg-gradient-to-br from-cyan-400 to-blue-600 flex items-center justify-center">
          <svg class="w-6 h-6 text-white" viewBox="0 0 24 24" fill="currentColor">
            <path d="M8 5v14l11-7z"/>
          </svg>
        </div>
        <h1 class="header-title">OpenClaw 运维监控平台</h1>
      </div>
      <div class="flex items-center gap-8">
        <div class="flex items-center gap-2">
          <div class="status-indicator" :class="connected ? 'online' : 'offline'"></div>
          <span class="text-sm text-gray-400">{{ connected ? '已连接' : '未连接' }}</span>
        </div>
        <div class="header-time">{{ currentTime }}</div>
      </div>
    </header>

    <!-- Main Content -->
    <main class="p-4 grid grid-cols-24 gap-4" style="height: calc(100vh - 80px);">
      
      <!-- Left Column -->
      <div class="col-span-6 flex flex-col gap-4">
        <!-- CPU Gauge -->
        <div class="glow-card">
          <div class="section-title">CPU 使用率</div>
          <div class="gauge-container">
            <svg viewBox="0 0 200 120" class="w-full">
              <!-- 背景弧 -->
              <path d="M 20 100 A 80 80 0 0 1 180 100" fill="none" stroke="rgba(255,255,255,0.1)" stroke-width="12" stroke-linecap="round"/>
              <!-- 值弧 -->
              <path d="M 20 100 A 80 80 0 0 1 180 100" fill="none" :stroke="getGaugeColor(systemStats.cpu)" stroke-width="12" stroke-linecap="round"
                    :stroke-dasharray="`${systemStats.cpu * 2.51} 251`" class="transition-all duration-500"/>
              <!-- 刻度 -->
              <text x="25" y="115" fill="rgba(255,255,255,0.5)" font-size="10">0%</text>
              <text x="165" y="115" fill="rgba(255,255,255,0.5)" font-size="10">100%</text>
            </svg>
            <div class="text-center -mt-8">
              <span class="gauge-value animated-number">{{ systemStats.cpu }}</span>
              <span class="text-lg text-cyan-400">%</span>
            </div>
          </div>
        </div>

        <!-- Memory Gauge -->
        <div class="glow-card">
          <div class="section-title">内存使用率</div>
          <div class="gauge-container">
            <svg viewBox="0 0 200 120" class="w-full">
              <path d="M 20 100 A 80 80 0 0 1 180 100" fill="none" stroke="rgba(255,255,255,0.1)" stroke-width="12" stroke-linecap="round"/>
              <path d="M 20 100 A 80 80 0 0 1 180 100" fill="none" :stroke="getGaugeColor(systemStats.memory)" stroke-width="12" stroke-linecap="round"
                    :stroke-dasharray="`${systemStats.memory * 2.51} 251`" class="transition-all duration-500"/>
              <text x="25" y="115" fill="rgba(255,255,255,0.5)" font-size="10">0%</text>
              <text x="165" y="115" fill="rgba(255,255,255,0.5)" font-size="10">100%</text>
            </svg>
            <div class="text-center -mt-8">
              <span class="gauge-value animated-number">{{ systemStats.memory }}</span>
              <span class="text-lg text-cyan-400">%</span>
            </div>
          </div>
        </div>

        <!-- System Resources -->
        <div class="glow-card flex-1">
          <div class="section-title">系统资源</div>
          <div class="p-4 space-y-4">
            <div>
              <div class="flex justify-between text-sm mb-2">
                <span class="text-gray-400">磁盘使用</span>
                <span class="text-white">{{ systemStats.disk }}%</span>
              </div>
              <div class="progress-bar">
                <div class="progress-fill blue" :style="{ width: systemStats.disk + '%' }"></div>
              </div>
            </div>
            <div>
              <div class="flex justify-between text-sm mb-2">
                <span class="text-gray-400">网络上传</span>
                <span class="text-white">{{ networkStats.upload }} MB/s</span>
              </div>
              <div class="progress-bar">
                <div class="progress-fill green" :style="{ width: networkStats.uploadPct + '%' }"></div>
              </div>
            </div>
            <div>
              <div class="flex justify-between text-sm mb-2">
                <span class="text-gray-400">网络下载</span>
                <span class="text-white">{{ networkStats.download }} MB/s</span>
              </div>
              <div class="progress-bar">
                <div class="progress-fill purple" :style="{ width: networkStats.downloadPct + '%' }"></div>
              </div>
            </div>
          </div>
          
          <div class="border-t border-white/10 mt-2">
            <div class="data-row">
              <span class="data-label">运行时间</span>
              <span class="data-value">{{ formatUptime(gatewayStats.uptime) }}</span>
            </div>
            <div class="data-row">
              <span class="data-label">进程数</span>
              <span class="data-value">{{ systemStats.processes }}</span>
            </div>
            <div class="data-row">
              <span class="data-label">负载均值</span>
              <span class="data-value">{{ systemStats.loadAvg }}</span>
            </div>
          </div>
        </div>
      </div>

      <!-- Center Column -->
      <div class="col-span-12 flex flex-col gap-4">
        <!-- Top Stats -->
        <div class="grid grid-cols-5 gap-4">
          <div class="glow-card p-4 text-center">
            <div class="text-xs text-gray-400 mb-1">在线会话</div>
            <div class="stat-value">{{ gatewayStats.sessions }}</div>
            <div class="text-xs text-green-400 mt-1">↑ 12%</div>
          </div>
          <div class="glow-card p-4 text-center">
            <div class="text-xs text-gray-400 mb-1">消息处理</div>
            <div class="stat-value">{{ gatewayStats.messages }}</div>
            <div class="text-xs text-green-400 mt-1">↑ 8%</div>
          </div>
          <div class="glow-card p-4 text-center">
            <div class="text-xs text-gray-400 mb-1">API调用</div>
            <div class="stat-value">{{ gatewayStats.apiCalls }}</div>
            <div class="text-xs text-green-400 mt-1">↑ 15%</div>
          </div>
          <div class="glow-card p-4 text-center">
            <div class="text-xs text-gray-400 mb-1">技能加载</div>
            <div class="stat-value">{{ gatewayStats.skillsLoaded }}</div>
            <div class="text-xs text-cyan-400 mt-1">正常</div>
          </div>
          <div class="glow-card p-4 text-center">
            <div class="text-xs text-gray-400 mb-1">定时任务</div>
            <div class="stat-value">{{ gatewayStats.cronJobs }}</div>
            <div class="text-xs text-cyan-400 mt-1">运行中</div>
          </div>
        </div>

        <!-- Main Chart -->
        <div class="glow-card flex-1">
          <div class="section-title">
            <span>实时流量监控</span>
            <div class="ml-auto flex gap-4 text-xs">
              <span class="flex items-center gap-1"><span class="w-3 h-0.5 bg-cyan-400 inline-block"></span> CPU</span>
              <span class="flex items-center gap-1"><span class="w-3 h-0.5 bg-purple-400 inline-block"></span> 内存</span>
              <span class="flex items-center gap-1"><span class="w-3 h-0.5 bg-green-400 inline-block"></span> 网络</span>
            </div>
          </div>
          <div class="chart-container" style="height: calc(100% - 50px);">
            <LineChart :cpuData="cpuHistory" :memData="memoryHistory" :netData="networkHistory" />
          </div>
        </div>

        <!-- Services Grid -->
        <div class="grid grid-cols-3 gap-4">
          <!-- Gateway Status -->
          <div class="glow-card">
            <div class="section-title">Gateway 状态</div>
            <div class="p-4 space-y-2">
              <div class="flex items-center gap-3">
                <div class="w-8 h-8 rounded-lg bg-cyan-500/20 flex items-center justify-center">
                  <svg class="w-4 h-4 text-cyan-400" fill="currentColor" viewBox="0 0 20 20">
                    <path d="M2 5a2 2 0 012-2h12a2 2 0 012 2v2a2 2 0 01-2 2H4a2 2 0 01-2-2V5z"/>
                  </svg>
                </div>
                <div>
                  <div class="text-sm text-white">{{ gatewayStats.version }}</div>
                  <div class="text-xs text-gray-500">版本</div>
                </div>
              </div>
              <div class="flex items-center gap-3">
                <div class="w-8 h-8 rounded-lg bg-green-500/20 flex items-center justify-center">
                  <svg class="w-4 h-4 text-green-400" fill="currentColor" viewBox="0 0 20 20">
                    <path fill-rule="evenodd" d="M10 18a8 8 0 100-16 8 8 0 000 16zm1-12a1 1 0 10-2 0v4a1 1 0 00.293.707l2.828 2.829a1 1 0 101.415-1.415L11 9.586V6z"/>
                  </svg>
                </div>
                <div>
                  <div class="text-sm text-white">{{ gatewayStats.port }}</div>
                  <div class="text-xs text-gray-500">端口</div>
                </div>
              </div>
              <div class="flex items-center gap-3">
                <div class="w-8 h-8 rounded-lg bg-purple-500/20 flex items-center justify-center">
                  <svg class="w-4 h-4 text-purple-400" fill="currentColor" viewBox="0 0 20 20">
                    <path d="M9 6a3 3 0 11-6 0 3 3 0 016 0zM17 6a3 3 0 11-6 0 3 3 0 016 0z"/>
                  </svg>
                </div>
                <div>
                  <div class="text-sm text-white">{{ gatewayStats.pid }}</div>
                  <div class="text-xs text-gray-500">PID</div>
                </div>
              </div>
            </div>
          </div>

          <!-- Active Channels -->
          <div class="glow-card">
            <div class="section-title">活动频道</div>
            <div class="p-2">
              <div v-for="channel in channels" :key="channel.id" class="service-item">
                <div class="service-icon" :class="channel.iconBg">
                  {{ channel.icon }}
                </div>
                <div class="service-info">
                  <div class="service-name">{{ channel.name }}</div>
                  <div class="service-desc">{{ channel.messages }} 条消息</div>
                </div>
                <span :class="channel.statusClass" class="tag text-xs">{{ channel.status }}</span>
              </div>
            </div>
          </div>

          <!-- Skills Status -->
          <div class="glow-card">
            <div class="section-title">技能状态</div>
            <div class="p-2 max-h-40 overflow-y-auto">
              <div v-for="skill in skills" :key="skill.name" class="data-row py-2">
                <span class="data-label">{{ skill.name }}</span>
                <div class="flex items-center gap-2">
                  <div class="status-indicator" :class="skill.status === 'active' ? 'online' : 'warning'"></div>
                  <span :class="skill.status === 'active' ? 'text-green-400' : 'text-orange-400'" class="text-xs">{{ skill.status }}</span>
                </div>
              </div>
            </div>
          </div>
        </div>

        <!-- Cron Jobs & Recent Activity -->
        <div class="grid grid-cols-2 gap-4">
          <div class="glow-card">
            <div class="section-title">定时任务</div>
            <div class="p-2">
              <div v-for="cron in cronJobs" :key="cron.name" class="data-row py-2">
                <div>
                  <span class="data-label">{{ cron.name }}</span>
                  <span class="text-xs text-gray-500 ml-2">{{ cron.schedule }}</span>
                </div>
                <div class="status-indicator" :class="cron.enabled ? 'online' : 'offline'"></div>
              </div>
            </div>
          </div>

          <div class="glow-card">
            <div class="section-title">最近活动</div>
            <div class="p-2 max-h-32 overflow-y-auto">
              <div v-for="activity in recentActivities" :key="activity.time" class="flex items-center gap-3 py-2 border-b border-white/5 last:border-0">
                <div class="w-2 h-2 rounded-full" :class="activity.color"></div>
                <div class="flex-1 text-sm text-gray-300">{{ activity.title }}</div>
                <div class="text-xs text-gray-500">{{ activity.time }}</div>
              </div>
            </div>
          </div>
        </div>
      </div>

      <!-- Right Column -->
      <div class="col-span-6 flex flex-col gap-4">
        <!-- System Info -->
        <div class="glow-card">
          <div class="section-title">系统信息</div>
          <div class="p-4">
            <div class="data-row">
              <span class="data-label">主机名</span>
              <span class="data-value">{{ systemInfo.hostname }}</span>
            </div>
            <div class="data-row">
              <span class="data-label">操作系统</span>
              <span class="data-value">{{ systemInfo.os }}</span>
            </div>
            <div class="data-row">
              <span class="data-label">内核版本</span>
              <span class="data-value">{{ systemInfo.kernel }}</span>
            </div>
            <div class="data-row">
              <span class="data-label">Node版本</span>
              <span class="data-value">{{ systemInfo.nodeVersion }}</span>
            </div>
            <div class="data-row">
              <span class="data-label">架构</span>
              <span class="data-value">{{ systemInfo.arch }}</span>
            </div>
            <div class="data-row">
              <span class="data-label">内存总量</span>
              <span class="data-value">{{ systemInfo.totalMemory }}</span>
            </div>
          </div>
        </div>

        <!-- Alerts -->
        <div class="glow-card flex-1">
          <div class="section-title flex justify-between">
            <span>告警信息</span>
            <span class="text-xs text-orange-400">{{ alerts.length }} 条</span>
          </div>
          <div class="p-2 max-h-48 overflow-y-auto">
            <div v-for="alert in alerts" :key="alert.id" class="alert-item">
              <div class="alert-icon" :class="alert.iconBg">
                {{ alert.icon }}
              </div>
              <div class="alert-content">
                <div class="alert-title">{{ alert.title }}</div>
                <div class="alert-time">{{ alert.time }}</div>
              </div>
            </div>
          </div>
        </div>

        <!-- Logs Terminal -->
        <div class="glow-card flex-1">
          <div class="section-title flex justify-between">
            <span>系统日志</span>
            <button @click="clearLogs" class="text-xs text-gray-500 hover:text-cyan-400">清空</button>
          </div>
          <div class="log-terminal m-4 p-3 h-64 overflow-y-auto">
            <div v-for="(log, i) in logs" :key="i" class="log-entry">
              <span class="log-time">{{ log.time }}</span>
              <span :class="'log-level-' + log.level"> [{{ log.level.toUpperCase() }}] </span>
              <span class="text-gray-300">{{ log.message }}</span>
            </div>
          </div>
        </div>

        <!-- Network Status -->
        <div class="glow-card">
          <div class="section-title">网络状态</div>
          <div class="p-4 grid grid-cols-2 gap-4">
            <div class="text-center">
              <div class="text-xs text-gray-400 mb-1">入站流量</div>
              <div class="text-lg text-green-400 font-mono">{{ networkStats.inbound }}</div>
              <div class="text-xs text-gray-500">GB/今日</div>
            </div>
            <div class="text-center">
              <div class="text-xs text-gray-400 mb-1">出站流量</div>
              <div class="text-lg text-purple-400 font-mono">{{ networkStats.outbound }}</div>
              <div class="text-xs text-gray-500">GB/今日</div>
            </div>
          </div>
        </div>
      </div>

    </main>
  </div>
</template>

<script setup>
import { ref, onMounted, onUnmounted } from 'vue'
import LineChart from './components/LineChart.vue'

// State
const connected = ref(true)
const currentTime = ref('')

const systemStats = ref({
  cpu: 35,
  memory: 58,
  disk: 67,
  processes: 142,
  loadAvg: '0.52, 0.58, 0.61'
})

const networkStats = ref({
  upload: 2.4,
  download: 15.8,
  uploadPct: 24,
  downloadPct: 63,
  inbound: 12.5,
  outbound: 3.2
})

const gatewayStats = ref({
  version: 'v2026.3.12',
  port: 18789,
  host: '127.0.0.1',
  pid: 68832,
  nodeVersion: 'v22.22.1',
  uptime: 0,
  sessions: 3,
  messages: 1247,
  apiCalls: 3582,
  skillsLoaded: 12,
  cronJobs: 4
})

const systemInfo = ref({
  hostname: 'Mac-mini.local',
  os: 'macOS 15.3',
  kernel: 'Darwin 25.3.0',
  nodeVersion: 'v22.22.1',
  arch: 'arm64',
  totalMemory: '16 GB'
})

const cpuHistory = ref([])
const memoryHistory = ref([])
const networkHistory = ref([])

const channels = ref([
  { id: 1, name: 'Feishu', icon: '📨', iconBg: 'bg-blue-500/20', status: '在线', statusClass: 'tag-blue', messages: 523 },
  { id: 2, name: 'Main Session', icon: '💬', iconBg: 'bg-green-500/20', status: '活动', statusClass: 'tag-green', messages: 724 },
  { id: 3, name: 'Telegram', icon: '✈️', iconBg: 'bg-cyan-500/20', status: '就绪', statusClass: 'tag-blue', messages: 0 },
])

const skills = ref([
  { name: 'feishu-doc', status: 'active' },
  { name: 'feishu-bitable', status: 'active' },
  { name: 'feishu-im', status: 'active' },
  { name: 'weather', status: 'idle' },
  { name: 'github', status: 'active' },
  { name: 'agent-browser', status: 'active' },
])

const cronJobs = ref([
  { name: 'Heartbeat', schedule: '*/5 * * * *', enabled: true },
  { name: 'Health Check', schedule: '*/1 * * * *', enabled: true },
  { name: 'Session Cleanup', schedule: '0 0 * * *', enabled: true },
  { name: 'Log Rotation', schedule: '0 0 * * 0', enabled: false },
])

const recentActivities = ref([
  { title: '收到新消息：红米K50u如何刷入Windows', time: '2分钟前', color: 'bg-cyan-400' },
  { title: '工具执行：web_search', time: '5分钟前', color: 'bg-purple-400' },
  { title: '会话创建：main', time: '10分钟前', color: 'bg-green-400' },
  { title: '技能加载：feishu-bitable', time: '15分钟前', color: 'bg-blue-400' },
  { title: '健康检查通过', time: '20分钟前', color: 'bg-green-400' },
])

const alerts = ref([
  { id: 1, icon: '⚠️', iconBg: 'bg-orange-500/20', title: '内存使用率超过70%', time: '5分钟前' },
  { id: 2, icon: 'ℹ️', iconBg: 'bg-blue-500/20', title: '新设备连接：iPhone 15', time: '15分钟前' },
  { id: 3, icon: '✓', iconBg: 'bg-green-500/20', title: '技能更新完成', time: '1小时前' },
])

const logs = ref([
  { time: '21:42:15', level: 'info', message: 'Gateway started on port 18789' },
  { time: '21:42:16', level: 'success', message: 'WebSocket connection established' },
  { time: '21:42:20', level: 'info', message: 'Session created: main' },
  { time: '21:43:00', level: 'info', message: 'Heartbeat received' },
  { time: '21:44:30', level: 'warn', message: 'Memory usage above 60%' },
  { time: '21:45:00', level: 'info', message: 'Health check passed' },
  { time: '21:46:15', level: 'info', message: 'API call: web_search' },
  { time: '21:47:00', level: 'success', message: 'Report generated successfully' },
])

// Methods
const updateTime = () => {
  const now = new Date()
  currentTime.value = now.toLocaleString('zh-CN', { 
    year: 'numeric',
    month: '2-digit',
    day: '2-digit',
    hour: '2-digit',
    minute: '2-digit',
    second: '2-digit',
    hour12: false 
  })
}

const simulateStats = () => {
  systemStats.value.cpu = Math.floor(Math.random() * 30 + 25)
  systemStats.value.memory = Math.floor(Math.random() * 15 + 50)
  systemStats.value.processes = Math.floor(Math.random() * 20 + 135)
  networkStats.value.upload = parseFloat((Math.random() * 5 + 1).toFixed(1))
  networkStats.value.download = parseFloat((Math.random() * 20 + 10).toFixed(1))
  networkStats.value.uploadPct = Math.floor(networkStats.value.upload * 10)
  networkStats.value.downloadPct = Math.floor(networkStats.value.download * 3)

  cpuHistory.value.push(systemStats.value.cpu)
  memoryHistory.value.push(systemStats.value.memory)
  networkHistory.value.push(networkStats.value.download)

  if (cpuHistory.value.length > 60) cpuHistory.value.shift()
  if (memoryHistory.value.length > 60) memoryHistory.value.shift()
  if (networkHistory.value.length > 60) networkHistory.value.shift()

  gatewayStats.value.uptime++
  gatewayStats.value.messages += Math.floor(Math.random() * 3)
  gatewayStats.value.apiCalls += Math.floor(Math.random() * 5)
}

const clearLogs = () => {
  logs.value = []
}

const formatUptime = (seconds) => {
  const h = Math.floor(seconds / 3600)
  const m = Math.floor((seconds % 3600) / 60)
  const s = seconds % 60
  return `${h.toString().padStart(2, '0')}:${m.toString().padStart(2, '0')}:${s.toString().padStart(2, '0')}`
}

const getGaugeColor = (value) => {
  if (value < 50) return '#00d4ff'
  if (value < 75) return '#ffaa00'
  return '#ff4757'
}

let timeInterval = null
let statsInterval = null

onMounted(() => {
  updateTime()
  timeInterval = setInterval(updateTime, 1000)
  statsInterval = setInterval(simulateStats, 1000)
})

onUnmounted(() => {
  if (timeInterval) clearInterval(timeInterval)
  if (statsInterval) clearInterval(statsInterval)
})
</script>

<style scoped>
.grid-cols-24 {
  grid-template-columns: repeat(24, minmax(0, 1fr));
}
</style>