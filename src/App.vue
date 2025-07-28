<template>
  <div class="container">
    <h2>Funding Rate Board</h2>
    
    <!-- 設定區域 -->
    <div class="settings-panel">
      <div class="setting-item">
        <label for="threshold">過濾閾值 (%):</label>
        <input 
          type="number" 
          id="threshold" 
          v-model="tempThreshold" 
          step="0.001" 
          min="0.001" 
          max="10"
          class="threshold-input"
          @keyup.enter="confirmThreshold"
        />
        <button @click="confirmThreshold" class="confirm-btn">確認</button>
        <button @click="refreshData" class="refresh-btn">重新整理</button>
        <span class="threshold-info">目前顯示閾值: {{ filterThreshold }}%</span>
      </div>
      
      <!-- 時間資訊區域 -->
      <div class="time-info">
        <div class="time-item">
          <span class="time-label">最新更新時間:</span>
          <span class="time-value">{{ lastUpdateTime || '尚未更新' }}</span>
        </div>
        <div class="time-item">
          <span class="time-label">重新整理時間:</span>
          <span class="time-value">{{ lastRefreshTime || '尚未重新整理' }}</span>
        </div>
      </div>
    </div>
    
    <table>
      <thead>
        <tr>
          <th @click="setSort('symbol')" style="cursor:pointer">
            Symbol <span v-if="sortKey==='symbol'">{{ sortDesc ? '↓' : '↑' }}</span>
          </th>
          <th @click="setSort('btcc')" style="cursor:pointer">
            BTCC <span v-if="sortKey==='btcc'">{{ sortDesc ? '↓' : '↑' }}</span>
          </th>
          <th @click="setSort('mexc')" style="cursor:pointer">
            MEXC <span v-if="sortKey==='mexc'">{{ sortDesc ? '↓' : '↑' }}</span>
          </th>
          <th @click="setSort('bingx')" style="cursor:pointer">
            BINGX <span v-if="sortKey==='bingx'">{{ sortDesc ? '↓' : '↑' }}</span>
          </th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="item in sortedRates" :key="item.symbol">
          <td>{{ formatSymbol(item.symbol) }}</td>
          <td :class="rateClass(item.btcc)">{{ formatRate(item.btcc) }}</td>
          <td :class="rateClass(item.mexc)">{{ formatRate(item.mexc) }}</td>
          <td :class="rateClass(item.bingx)">{{ formatRate(item.bingx) }}</td>
        </tr>
      </tbody>
    </table>
  </div>
</template>

<script setup>
import { ref, computed, onMounted, onUnmounted } from 'vue'
import { io } from 'socket.io-client'
import { API_BASE_URL } from './config.js'

const fundingRates = ref([])
const sortKey = ref('symbol')
const sortDesc = ref(false)
const filterThreshold = ref(0.02)
const tempThreshold = ref(0.02)
const lastUpdateTime = ref('')
const lastRefreshTime = ref('')
let socket = null

function rateClass(rate) {
  if (rate === null || rate === undefined) return ''
  return Math.abs(rate) > 0.5 ? 'high' : Math.abs(rate) > 0.2 ? 'mid' : ''
}

function formatRate(rate) {
  return rate !== null && rate !== undefined ? rate.toFixed(4) + '%' : ''
}

function formatSymbol(symbol) {
  if (!symbol) return ''
  // 後端已經統一處理了符號標準化，直接顯示即可
  return symbol.toUpperCase()
}

function formatDateTime() {
  const now = new Date()
  const year = now.getFullYear()
  const month = String(now.getMonth() + 1).padStart(2, '0')
  const day = String(now.getDate()).padStart(2, '0')
  const hours = String(now.getHours()).padStart(2, '0')
  const minutes = String(now.getMinutes()).padStart(2, '0')
  const seconds = String(now.getSeconds()).padStart(2, '0')
  return `${year}-${month}-${day} ${hours}:${minutes}:${seconds}`
}

async function confirmThreshold() {
  await updateThreshold()
}

async function updateThreshold() {
  try {
    const response = await fetch(`${API_BASE_URL}/api/filter_threshold`, {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
      },
      body: JSON.stringify({ threshold: tempThreshold.value })
    })
    
    const result = await response.json()
    if (result.success) {
      filterThreshold.value = result.threshold
      console.log(`閾值已更新為: ${result.threshold}%`)
      // 重新獲取數據以應用新的過濾條件
      await fetchInitialData()
    } else {
      console.error('更新閾值失敗:', result.error)
      // 重置臨時閾值
      tempThreshold.value = filterThreshold.value
    }
  } catch (error) {
    console.error('更新閾值錯誤:', error)
    tempThreshold.value = filterThreshold.value
  }
}

async function refreshData() {
  try {
    await fetchInitialData()
    lastRefreshTime.value = formatDateTime()
    console.log('數據已重新整理')
  } catch (error) {
    console.error('重新整理數據失敗:', error)
  }
}

async function fetchCurrentThreshold() {
  try {
    const response = await fetch(`${API_BASE_URL}/api/filter_threshold`)
    const result = await response.json()
    filterThreshold.value = result.threshold
    tempThreshold.value = result.threshold
  } catch (error) {
    console.error('獲取閾值錯誤:', error)
  }
}

function setSort(key) {
  if (sortKey.value === key) {
    sortDesc.value = !sortDesc.value
  } else {
    sortKey.value = key
    sortDesc.value = false
  }
}

const sortedRates = computed(() => {
  const arr = [...fundingRates.value]
  arr.sort((a, b) => {
    let v1 = a[sortKey.value]
    let v2 = b[sortKey.value]
    if (sortKey.value === 'symbol') {
      v1 = v1?.toUpperCase() || ''
      v2 = v2?.toUpperCase() || ''
      return sortDesc.value ? v2.localeCompare(v1) : v1.localeCompare(v2)
    } else {
      v1 = v1 ?? -Infinity
      v2 = v2 ?? -Infinity
      return sortDesc.value ? v2 - v1 : v1 - v2
    }
  })
  return arr
})

async function fetchInitialData() {
  const res = await fetch(`${API_BASE_URL}/api/funding_rates`)
  const data = await res.json()
  fundingRates.value = data
  // 初次加載時設置時間
  if (!lastRefreshTime.value) {
    lastRefreshTime.value = formatDateTime()
  }
}

onMounted(() => {
  fetchInitialData()
  fetchCurrentThreshold()
  
  socket = io(API_BASE_URL)
  
  // 處理資金費率更新
  socket.on('funding_rate_update', (update) => {
    const idx = fundingRates.value.findIndex(i => i.symbol === update.symbol)
    if (idx >= 0) {
      // 更新現有項目
      fundingRates.value[idx][update.exchange.toLowerCase()] = update.rate
    } else {
      // 創建新項目
      const newItem = {
        symbol: update.symbol,
        btcc: null,
        mexc: null,
        bingx: null
      }
      newItem[update.exchange.toLowerCase()] = update.rate
      fundingRates.value.push(newItem)
    }
    // 更新最新更新時間
    lastUpdateTime.value = formatDateTime()
  })
  
  // 處理資金費率刪除
  socket.on('funding_rate_delete', (deleteInfo) => {
    const idx = fundingRates.value.findIndex(i => i.symbol === deleteInfo.symbol)
    if (idx >= 0) {
      // 將對應交易所的費率設為 null
      fundingRates.value[idx][deleteInfo.exchange.toLowerCase()] = null
      
      // 如果所有交易所的費率都為 null，則刪除整個項目
      const item = fundingRates.value[idx]
      if (item.btcc === null && item.mexc === null && item.bingx === null) {
        fundingRates.value.splice(idx, 1)
      }
      // 更新最新更新時間
      lastUpdateTime.value = formatDateTime()
    }
  })
  
  // 處理閾值更新通知
  socket.on('threshold_updated', (data) => {
    filterThreshold.value = data.threshold
    tempThreshold.value = data.threshold
    console.log(`閾值已更新為: ${data.threshold}%`)
    // 重新獲取數據以應用新的過濾條件
    fetchInitialData()
  })
})

onUnmounted(() => {
  if (socket) socket.disconnect()
})
</script>

<style>
body { background: #181818; color: #eee; font-family: 'Segoe UI', Arial, sans-serif; }
.container { max-width: 900px; margin: 40px auto; background: #222; padding: 24px; border-radius: 12px; box-shadow: 0 2px 16px #0008; }
h2 { text-align: center; margin-bottom: 24px; color: #00bfff; }

.settings-panel { 
  background: #333; 
  padding: 16px; 
  border-radius: 8px; 
  margin-bottom: 24px; 
  border: 1px solid #444;
}

.setting-item { 
  display: flex; 
  align-items: center; 
  gap: 12px; 
  flex-wrap: wrap;
  margin-bottom: 12px;
}

.time-info {
  display: flex;
  gap: 24px;
  flex-wrap: wrap;
  padding-top: 12px;
  border-top: 1px solid #444;
}

.time-item {
  display: flex;
  align-items: center;
  gap: 8px;
}

.time-label {
  color: #00bfff;
  font-weight: bold;
  font-size: 13px;
}

.time-value {
  color: #ccc;
  font-size: 13px;
  font-family: 'Courier New', monospace;
}

.setting-item label { 
  color: #00bfff; 
  font-weight: bold; 
  min-width: 120px;
}

.threshold-input { 
  background: #444; 
  border: 1px solid #555; 
  color: #eee; 
  padding: 8px 12px; 
  border-radius: 4px; 
  width: 120px;
  font-size: 14px;
}

.threshold-input:focus { 
  outline: none; 
  border-color: #00bfff; 
  box-shadow: 0 0 0 2px rgba(0, 191, 255, 0.2);
}

.threshold-info { 
  color: #999; 
  font-size: 12px; 
  font-style: italic;
  margin-left: 8px;
}

.confirm-btn, .refresh-btn {
  background: #00bfff;
  border: none;
  color: white;
  padding: 8px 16px;
  border-radius: 4px;
  cursor: pointer;
  font-size: 14px;
  margin-left: 8px;
  transition: background-color 0.2s;
}

.confirm-btn:hover {
  background: #0099cc;
}

.refresh-btn {
  background: #28a745;
}

.refresh-btn:hover {
  background: #218838;
}

table { border-collapse: collapse; width: 100%; background: #222; }
th, td { border: 1px solid #333; padding: 12px 8px; text-align: center; }
th { background: #333; color: #fff; font-size: 1.1em; }
tr:nth-child(even) { background: #282828; }
.high { color: #e74c3c; font-weight: bold; background: #2c0a0a; }
.mid { color: #f39c12; font-weight: bold; background: #2c1a0a; }
</style>
