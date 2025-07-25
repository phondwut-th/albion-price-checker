<template>
  <div class="buy-order-container">
    <h2>🧱 ราคารับซื้อ/ขาย หินขัดเกลา Tier 7–8 (Enchanted)</h2>

    <button @click="fetchPrices">โหลดข้อมูลราคา</button>

    <div v-if="loading" class="status">⏳ กำลังโหลดข้อมูล...</div>
    <div v-if="error" class="error">❌ {{ error }}</div>

    <div v-if="Object.keys(pricesByItem).length" class="tables-wrapper">
      <div v-for="(entries, itemId) in pricesByItem" :key="itemId" class="table-block">
        <h3 class="item-title">🪨 {{ itemId }}</h3>
        <table>
          <thead>
            <tr>
              <th>📍 เมือง</th>
              <th class="right green">รับซื้อมากสุด (Buy)</th>
              <th class="right red">ขายต่ำสุด (Sell)</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="entry in entries" :key="entry.city">
              <td>{{ entry.city }}</td>
              <td class="right">{{ entry.buy_price_max.toLocaleString() }}</td>
              <td class="right">{{ entry.sell_price_min.toLocaleString() }}</td>
            </tr>
          </tbody>
        </table>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue'
import axios from 'axios'

const refinedRockIds = [
  'T7_ROCK',
  'T7_ROCK_LEVEL1@1',
  'T7_ROCK_LEVEL2@2',
  'T7_ROCK_LEVEL3@3',
  'T8_ROCK',
  'T8_ROCK_LEVEL1@1',
  'T8_ROCK_LEVEL2@2',
  'T8_ROCK_LEVEL3@3',
]

const cities = ['Bridgewatch', 'Martlock', 'Thetford', 'Lymhurst', 'FortSterling']

const pricesByItem = ref({})
const loading = ref(false)
const error = ref('')

const fetchPrices = async () => {
  loading.value = true
  error.value = ''
  pricesByItem.value = {}

  try {
    for (const itemId of refinedRockIds) {
      const res = await axios.get(
        `https://east.albion-online-data.com/api/v2/stats/prices/${itemId}?locations=${cities.join(',')}`,
      )

      const grouped = {}

      for (const entry of res.data) {
        const { city, buy_price_max, sell_price_min } = entry
        if (!grouped[city]) {
          grouped[city] = {
            city,
            buy_price_max: buy_price_max || 0,
            sell_price_min: sell_price_min || 0,
          }
        } else {
          // Keep the best buy and lowest sell
          if (buy_price_max > grouped[city].buy_price_max) {
            grouped[city].buy_price_max = buy_price_max
          }
          if (
            sell_price_min > 0 &&
            (grouped[city].sell_price_min === 0 || sell_price_min < grouped[city].sell_price_min)
          ) {
            grouped[city].sell_price_min = sell_price_min
          }
        }
      }

      pricesByItem.value[itemId] = Object.values(grouped)
    }
  } catch (e) {
    error.value = 'เกิดข้อผิดพลาดในการโหลดข้อมูล'
  } finally {
    loading.value = false
  }
}
</script>

<style scoped>
.buy-order-container {
  max-width: 700px;
  margin: auto;
  padding: 1rem;
  font-family: 'Segoe UI', sans-serif;
  color: #f3f4f6;
}

h2 {
  font-size: 1.25rem;
  margin-bottom: 1rem;
}

button {
  background-color: #16a34a;
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
  background-color: #15803d;
}

.status {
  margin: 0.5rem 0;
  color: #cbd5e1;
}

.error {
  margin: 0.5rem 0;
  color: #f87171;
}

.tables-wrapper {
  display: flex;
  flex-direction: column;
  gap: 2rem;
}

.table-block {
  background-color: #1f2937;
  padding: 1rem;
  border-radius: 8px;
  border: 1px solid #374151;
}

.item-title {
  font-weight: bold;
  margin-bottom: 0.5rem;
  color: #3b82f6;
}

table {
  width: 100%;
  border-collapse: collapse;
  font-size: 0.95rem;
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
