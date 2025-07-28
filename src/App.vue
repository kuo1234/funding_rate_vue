<template>
  <div class="container">
    <h2>Funding Rate Board</h2>
    <button class="refresh-btn" @click="fetchData">Refresh</button>
    <table>
      <thead>
        <tr>
          <th @click="setSort('symbol')" style="cursor:pointer">
            Symbol <span v-if="sortKey==='symbol'">{{ sortDesc ? '▼' : '▲' }}</span>
          </th>
          <th @click="setSort('btcc')" style="cursor:pointer">
            BTCC <span v-if="sortKey==='btcc'">{{ sortDesc ? '▼' : '▲' }}</span>
          </th>
          <th @click="setSort('mexc')" style="cursor:pointer">
            MEXC <span v-if="sortKey==='mexc'">{{ sortDesc ? '▼' : '▲' }}</span>
          </th>
          <th @click="setSort('bingx')" style="cursor:pointer">
            BINGX <span v-if="sortKey==='bingx'">{{ sortDesc ? '▼' : '▲' }}</span>
          </th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="item in sortedRates" :key="item.symbol">
          <td>{{ item.symbol.toUpperCase() }}</td>
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

const fundingRates = ref([])
let fetchTimer = null

const sortKey = ref('symbol')
const sortDesc = ref(false)

// --- localStorage cache ---
function saveCache(data) {
  localStorage.setItem('fundingRatesCache', JSON.stringify({
    data,
    ts: Date.now()
  }))
}

function loadCache() {
  const cache = localStorage.getItem('fundingRatesCache')
  if (!cache) return null
  try {
    const obj = JSON.parse(cache)
    // 只用10分鐘內的快取
    if (Date.now() - obj.ts < 10 * 60 * 1000) {
      return obj.data
    }
  } catch {
    // ignore cache parse error
  }
  return null
}

function rateClass(rate) {
  if (rate === null || rate === undefined) return ''
  return Math.abs(rate) > 0.5 ? 'high' : Math.abs(rate) > 0.2 ? 'mid' : ''
}

function formatRate(rate) {
  return rate !== null && rate !== undefined ? rate.toFixed(4) + '%' : ''
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

async function fetchData() {
  try {
    const res = await fetch('http://localhost:5000/api/funding_rates')
    const data = await res.json()
    fundingRates.value = data
    saveCache(data)
  } catch {
    // 如果失敗，使用快取
    const cache = loadCache()
    if (cache) fundingRates.value = cache
  }
}

onMounted(() => {
  // 頁面載入先用快取
  const cache = loadCache()
  if (cache) fundingRates.value = cache
  fetchData()
  fetchTimer = setInterval(fetchData, 30000)
})
onUnmounted(() => {
  if (fetchTimer) clearInterval(fetchTimer)
})
</script>

<style>
body { background: #181818; color: #eee; font-family: 'Segoe UI', Arial, sans-serif; }
.container { max-width: 900px; margin: 40px auto; background: #222; padding: 24px; border-radius: 12px; box-shadow: 0 2px 16px #0008; }
h2 { text-align: center; margin-bottom: 24px; color: #00bfff; }
.refresh-btn {
  display: block;
  margin: 0 auto 16px auto;
  padding: 8px 24px;
  background: #00bfff;
  color: #fff;
  border: none;
  border-radius: 6px;
  font-size: 1em;
  cursor: pointer;
  transition: background 0.2s;
}
.refresh-btn:hover {
  background: #0099cc;
}
table { border-collapse: collapse; width: 100%; background: #222; }
th, td { border: 1px solid #333; padding: 12px 8px; text-align: center; }
th { background: #333; color: #fff; font-size: 1.1em; }
tr:nth-child(even) { background: #282828; }
.high { color: #e74c3c; font-weight: bold; background: #2c0a0a; }
.mid { color: #f39c12; font-weight: bold; background: #2c1a0a; }
</style>