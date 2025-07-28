<template>
  <div>
    <h2>MEXC Funding Rate</h2>
    <table>
      <thead>
        <tr>
          <th @click="sortBy('symbol')">Symbol</th>
          <th @click="sortBy('rate')">Funding Rate</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="item in sortedFundingRates" :key="item.symbol">
          <td>{{ item.symbol }}</td>
          <td>{{ formatRate(item.rate) }}</td>
        </tr>
      </tbody>
    </table>
  </div>
</template>

<script>
import axios from 'axios';

export default {
  data() {
    return {
      fundingRates: {},
      sortKey: 'symbol',
      sortDirection: 'asc',
      updateInterval: null,
    };
  },
  computed: {
    sortedFundingRates() {
      const ratesArray = Object.entries(this.fundingRates).map(([symbol, rate]) => ({ symbol, rate }));
      if (this.sortKey === 'symbol') {
        ratesArray.sort((a, b) => {
          if (this.sortDirection === 'asc') {
            return a.symbol.localeCompare(b.symbol);
          } else {
            return b.symbol.localeCompare(a.symbol);
          }
        });
      } else if (this.sortKey === 'rate') {
        ratesArray.sort((a, b) => {
          if (this.sortDirection === 'asc') {
            return a.rate - b.rate;
          } else {
            return b.rate - a.rate;
          }
        });
      }
      return ratesArray;
    },
  },
  methods: {
    async fetchFundingRates() {
      try {
        const response = await axios.get('http://localhost:5000/api/funding_rates');
        this.fundingRates = response.data;
      } catch (error) {
        console.error('Failed to fetch funding rates', error);
      }
    },
    sortBy(key) {
      if (this.sortKey === key) {
        this.sortDirection = this.sortDirection === 'asc' ? 'desc' : 'asc';
      } else {
        this.sortKey = key;
        this.sortDirection = 'asc';
      }
    },
    formatRate(rate) {
      return (rate ) + '%';
    },
  },
  mounted() {
    this.fetchFundingRates();
    this.updateInterval = setInterval(this.fetchFundingRates, 3000); // 每 3 秒更新一次
  },
  beforeUnmount() {
    clearInterval(this.updateInterval);
  },
};
</script>

<style scoped>
table {
  width: 100%;
  border-collapse: collapse;
  margin-top: 20px;
}

th, td {
  border: 1px solid #ccc;
  padding: 8px;
  text-align: left;
}

th {
  background-color: #f0f0f0;
  cursor: pointer;
}
</style>