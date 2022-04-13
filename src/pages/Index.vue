<template>
  <q-page>

    <div class="relative flex min-h-screen flex-col justify-center overflow-hidden bg-gray-50 py-6 sm:py-12">
      <div v-for="cryptoData in cryptosDatas" :key="cryptoData.id" class="relative bg-white shadow-xl ring-1 mb-10 ring-gray-900/5 mx-auto w-4/5 md:w-3/5 lg:w-2/5 rounded-lg px-4 lg:px-10">
        <div class="py-8 text-base text-gray-600">
          <p class="grid sm:flex items-baseline">{{ cryptoData.name }} ({{ cryptoData.symbol }}) price: <code class="text-xs sm:text-sm font-bold text-gray-800 sm:pl-3 whitespace-nowrap">U$ {{ cryptoData.market_data.current_price.usd }}</code></p>
          <p class="grid sm:flex items-baseline">Last updated: <code class="text-xs sm:text-sm font-bold text-gray-800 sm:pl-3 whitespace-nowrap">{{ cryptoData.last_updated }}</code></p>
        </div>
      </div>
    </div>

  </q-page>
</template>

<script>
import { date } from 'quasar'

export default {
  name: 'PageIndex',
  data () {
    return {
      cryptosDatas: [],
      ticker: ['bitcoin', 'ethereum', 'dacxi' , 'terra-luna', 'cosmos'],
      url: []
    }
  },

  mounted () {
    this.onGetApiData()
  },

  methods: {
    onGetApiData () {
      this.url = []
      this.cryptosDatas = []

      for (var i = 0; i < this.ticker.length; i++) {
        this.url.push(this.$axios.get(`https://api.coingecko.com/api/v3/coins/${this.ticker[i]}?localization=false&tickers=false&community_data=false&developer_data=false&sparkline=false`))
      }

      this.$axios.all([this.url[0], this.url[1], this.url[2], this.url[3], this.url[4]])
        .then((response) => {
          for (var i = 0; i < response.length; i++) {
            this.cryptosDatas.push(response[i].data)
            this.cryptosDatas[i].symbol = this.cryptosDatas[i].symbol.toUpperCase()
            this.cryptosDatas[i].last_updated = date.formatDate(this.cryptosDatas[i].last_updated, 'DD/MM/YYYY - HH:mm:ss')
          }
        })
        .catch((error) => {
          console.log(error)
        })
        // .finally(() => {
        //   setTimeout(() => {
        //     this.onGetApiData()
        //   }, 30000)
        // })
    }
  }
}
</script>
