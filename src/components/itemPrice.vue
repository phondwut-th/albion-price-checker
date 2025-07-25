<template>
  <div class="item-price-container">
    <h2>📦 ราคาขาย/รับซื้อ: <span class="highlight">T8_STONEBLOCK</span></h2>

    <button @click="fetchPrice">โหลดข้อมูลราคา</button>

    <div v-if="loading" class="status">⏳ กำลังโหลด...</div>
    <div v-if="error" class="error">❌ {{ error }}</div>

    <table v-if="prices.length">
      <thead>
        <tr>
          <th>📍 เมือง</th>
          <th class="right green">รับซื้อมากสุด (Buy)</th>
          <th class="right red">ขายต่ำสุด (Sell)</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="entry in prices" :key="entry.city">
          <td>{{ entry.city }}</td>
          <td class="right">{{ entry.buy_price_max.toLocaleString() }}</td>
          <td class="right">{{ entry.sell_price_min.toLocaleString() }}</td>
        </tr>
      </tbody>
    </table>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'
import axios from 'axios'

const prices = ref([])
const loading = ref(false)
const error = ref('')
const itemId = 'T8_STONEBLOCK'
const cities = ['Bridgewatch', 'Martlock', 'Thetford', 'Lymhurst', 'FortSterling', 'BlackMarket']

const fetchPrice = async () => {
  loading.value = true
  error.value = ''
  prices.value = []

  try {
    const res = await axios.get(
      `https://east.albion-online-data.com/api/v2/stats/prices/${itemId}?locations=${cities.join(',')}`,
    )

    const raw = res.data
    const grouped = {}

    raw.forEach((entry) => {
      const city = entry.city
      const sell = entry.sell_price_min
      const buy = entry.buy_price_max

      if (!grouped[city]) {
        grouped[city] = { sell_price_min: sell || 0, buy_price_max: buy || 0 }
      } else {
        if (
          sell > 0 &&
          (grouped[city].sell_price_min === 0 || sell < grouped[city].sell_price_min)
        ) {
          grouped[city].sell_price_min = sell
        }
        if (buy > grouped[city].buy_price_max) {
          grouped[city].buy_price_max = buy
        }
      }
    })

    prices.value = Object.entries(grouped).map(([city, data]) => ({
      city,
      sell_price_min: data.sell_price_min,
      buy_price_max: data.buy_price_max,
    }))
  } catch (e) {
    error.value = 'ไม่สามารถโหลดข้อมูลได้'
  } finally {
    loading.value = false
  }
}

onMounted(() => {
  fetchPrice()
})
</script>

<style scoped>
.item-price-container {
  max-width: 640px;
  margin: auto;
  padding: 1rem;
  font-family: 'Segoe UI', sans-serif;
  color: #f3f4f6;
}

h2 {
  font-size: 1.25rem;
  margin-bottom: 1rem;
}

.highlight {
  color: #3b82f6;
  font-weight: bold;
}

button {
  background-color: #2563eb;
  color: white;
  padding: 0.5rem 1rem;
  border: none;
  font-weight: bold;
  border-radius: 6px;
  cursor: pointer;
  margin-bottom: 1rem;
  transition: background-color 0.2s ease;
}

button:hover {
  background-color: #1e40af;
}

.status {
  margin: 0.5rem 0;
  color: #cbd5e1;
}

.error {
  margin: 0.5rem 0;
  color: #f87171;
}

table {
  width: 100%;
  border-collapse: collapse;
  font-size: 0.95rem;
  background-color: #1f2937;
  border: 1px solid #374151;
  border-radius: 6px;
  overflow: hidden;
}

thead {
  background-color: #374151;
  color: #f3f4f6;
}

th,
td {
  padding: 0.75rem 1rem;
  border-bottom: 1px solid #4b5563;
}

tbody tr:nth-child(even) {
  background-color: #2d3748;
}

tbody tr:hover {
  background-color: #4b5563;
}

.right {
  text-align: right;
}

.green {
  color: #10b981;
  font-weight: bold;
}

.red {
  color: #ef4444;
  font-weight: bold;
}
</style>
