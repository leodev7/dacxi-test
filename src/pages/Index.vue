<template>
  <q-page>

    <div class="relative flex min-h-screen flex-col justify-center overflow-hidden bg-gray-50">
      <q-expansion-item group="somegroup" v-for="cryptoData in cryptosDatas" :key="cryptoData.id" @show="clearWhenExpand()" class="relative bg-white shadow-xl ring-1 ring-gray-900/5 my-5 mx-auto w-4/5 md:w-3/5 lg:w-2/5 rounded-lg px-4 lg:px-10 lg:py-5">

        <template v-slot:header>
          <q-item-section class="text-base text-gray-600">
            <p>{{ cryptoData.name }} ({{ cryptoData.symbol }}) price: <code class="text-xs sm:text-sm font-bold text-gray-800 sm:pl-3 whitespace-nowrap">U$ {{ cryptoData.market_data.current_price.usd }}</code></p>
            <p>Last updated: <code class="text-xs sm:text-sm font-bold text-gray-800 sm:pl-3 whitespace-nowrap">{{ cryptoData.last_updated }}</code></p>
          </q-item-section>
        </template>

        <q-card>
          <q-card-section>
            <p>Insira uma data</p>
            <div class="grid sm:flex sm:flex-nowrap">
              <q-input dense autofocus outlined clearable color="black" text-color="black" class="full-width" placeholder="31-12-2021" v-model="cryptosDatas.historyPrice" mask="##-##-####" @update:model-value="changeToNull()" />
              <q-btn :disable="!cryptosDatas.historyPrice || cryptosDatas.historyPrice.length < 10" label="Pesquisar" class="mt-4 ml-0 sm:mt-0 sm:ml-4 bg-gray-800 text-white text-capitalize" @click="onGetHistoryCryptoPrice(cryptoData.name, cryptosDatas.historyPrice)" />
            </div>
            <p class="mt-4" v-if="price">O valor do(a) <code class="text-xs sm:text-sm font-bold text-gray-800">{{ cryptoData.name }}</code> na data <code class="text-xs sm:text-sm font-bold text-gray-800 whitespace-nowrap">{{ cryptosDatas.historyPrice }}</code> era de <code class="text-xs sm:text-sm font-bold text-gray-800 whitespace-nowrap">U$ {{ price }}</code></p>
          </q-card-section>
        </q-card>

      </q-expansion-item>
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
      url: [],
      price: null
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
          this.$q.notify({ type: 'negative', message: `Erro na API. Código: ${error}`})
        })
        // .finally(() => {
        //   setTimeout(() => {
        //     this.onGetApiData()
        //   }, 15000)
        // })
    },

    onGetHistoryCryptoPrice (cryptoName, historyPrice) {
      this.$axios({ method: 'get', url: `https://api.coingecko.com/api/v3/coins/${cryptoName.toLowerCase()}/history?date=${historyPrice}&localization=false` })
        .then((response) => {
          if (!response || !response.data || !response.data.market_data || (response && response.data && !response.data.market_data)) {
            this.$q.notify({ type: 'negative', message: 'Data inválida, favor verificar!'})
            this.changeToNull()
            return
          }
          this.price = response.data.market_data.current_price.usd.toFixed(2)
        })
        .catch((error) => {
          this.$q.notify({ type: 'negative', message: `Erro na API. Código: ${error}`})
        })
    },

    changeToNull () {
      this.price = null
    },

    clearWhenExpand () {
      this.changeToNull()
      this.cryptosDatas.historyPrice = null
    }
  }
}
</script>
