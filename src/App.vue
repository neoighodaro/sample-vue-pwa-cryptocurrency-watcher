<template>
  <div>
    <intro></intro>
    <div id="body">
      <div id="current">
        <current v-bind:currentCurrency="currentCurrency"></current>
      </div>
      <div id="previous">
        <previous v-bind:previousCurrency="previousCurrency"></previous>
      </div>
    </div>
  </div>
</template>

<script>
import Pusher from 'pusher-js';
import Intro from './components/Intro.vue';
import Current from './components/Current.vue';
import Previous from './components/Previous.vue';

export default {
  name: 'app',
  components: {Intro, Current, Previous},
  data() {
    return {
      currentCurrency: {BTC: '', ETH: '', LTC: ''},
      previousCurrency: {
        yesterday: {}, twoDays: {}, threeDays: {}, fourDays: {}, fiveDays: {}
      }
    }
  },
  methods: {
    _fetchDataForToday: function () {
      this.axios.get('https://min-api.cryptocompare.com/data/pricemulti?fsyms=BTC,ETH,LTC&tsyms=USD').then(res => {
          localStorage.setItem('BTC', this.currentCurrency.BTC = res.data.BTC.USD),
          localStorage.setItem('ETH', this.currentCurrency.ETH = res.data.ETH.USD),
          localStorage.setItem('LTC', this.currentCurrency.LTC = res.data.LTC.USD)
      })
    },
    _fetchDataFor: function(key, daysAgo) {
      var self = this
      var date = this.$moment().subtract(daysAgo, 'days').unix()
      let fetch = (curr, date) => this.axios.get(`https://min-api.cryptocompare.com/data/pricehistorical?fsym=${curr}&tsyms=USD&ts=${date}`)

      this.axios
          .all([fetch('BTC', date), fetch('ETH', date), fetch('LTC', date)])
          .then(this.axios.spread((BTC, ETH, LTC) => {
              this.previousCurrency[key] = {
                  BTC: BTC.data.BTC.USD,
                  LTC: LTC.data.LTC.USD,
                  ETH: ETH.data.ETH.USD,
                  DATE: this.$moment.unix(date).format("MMMM Do YYYY"),
              }

              localStorage.setItem(`${key}Prices`, JSON.stringify(this.previousCurrency[key]));
          }))
    }
  },
  created() {
    if ( ! navigator.onLine) {
      this.currentCurrency = {
        BTC: localStorage.getItem('BTC'),
        ETH: localStorage.getItem('ETH'),
        LTC: localStorage.getItem('LTC'),
      }

      this.previousCurrency = {
        yesterday: JSON.parse(localStorage.getItem('yesterdayPrices')),
        twoDays:   JSON.parse(localStorage.getItem('twoDaysPrices')),
        threeDays: JSON.parse(localStorage.getItem('threeDaysPrices')),
        fourDays:  JSON.parse(localStorage.getItem('fourDaysPrices')),
        fiveDays:  JSON.parse(localStorage.getItem('fiveDaysPrices'))
      }
    } else {
      this._fetchDataFor('yesterday', 1)
      this._fetchDataFor('twoDays', 2)
      this._fetchDataFor('threeDays', 3)
      this._fetchDataFor('fourDays', 4)
      this._fetchDataFor('fiveDays', 5)
      this._fetchDataForToday()

      let pusher = new Pusher('92328e0f11b44cfe765f', {
        cluster: 'eu',
        encrypted: true
      });

      let channel = pusher.subscribe('price-updates');
      channel.bind('coin-updates', data => {
        this.currentCurrency = {BTC: data.coin.BTC.USD, ETH: data.coin.ETH.USD, LTC: data.coin.LTC.USD}
      });
    }
  }
}
</script>

<style>
@import url('https://fonts.googleapis.com/css?family=Lato');
* {
  margin : 0px;
  padding : 0px;
  font-family: 'Lato', sans-serif;
}
body { height: 100vh; width: 100%; }
.row { display: flex; flex-wrap: wrap; }
h1 { font-size: 48px; }
a { color: #FFFFFF; text-decoration: none; }
a:hover { color: #FFFFFF; }
a:visited { color: #000000; }
.button {
  margin: auto;
  width: 200px;
  height: 60px;
  border: 2px solid #E36F55;
  box-sizing: border-box;
  border-radius: 30px;
}
#body {
  max-width: 90%;
  margin: 0 auto;
  padding: 1.5em;
  text-align: center;
  color:rgb(0, 193, 131);
}
#current { padding: 2em 0em; }
#previous { padding: 2em 0em; }
</style>
