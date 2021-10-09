<template lang="pug">
main(
  style="min-height: 100vh; display: flex; align-items: center"
)
  v-container
    v-row
      v-col.text-center
        h1.mb-10(
          style="font-weight: 300; text-align: center"
        ) Uncle Eric Loves You, Jae Moon
    v-row
      v-col(
        cols="12"
        md="4"
      )
        v-card(
          elevation="10"
        )
          v-card-subtitle Original Value
          v-card-text
            h1.text-h1.text-right $500
      v-col(
        cols="12"
        md="4"
      )
        v-card(
          elevation="10"
        )
          v-card-subtitle Current Value
          v-card-text
            h1.text-h1.text-right ${{ currentValue }}
      v-col(
        cols="12"
        md="4"
      )
        v-card(
          :elevation="10"
        )
          v-card-subtitle Profit/Loss
          v-card-text
            h1.text-h1.text-right ${{ profitLossDollars }}
              | &nbsp;
              v-icon {{ profitLossPercentage > 0 ? 'mdi-arrow-up' : 'mdi-arrow-down' }}
              | {{ Math.abs(profitLossPercentage) }}%
    v-row
      v-col
        apexchart(
          type="area" height="350" :options="chartOptions" :series="series"
        )
    v-row
      v-col
        h2(
          style="text-align: center; font-weight: 300"
        )
          strong {{ Math.abs($dayjs.duration($dayjs().diff($dayjs('10/09/2039'))).asDays()).toFixed() }}
          | &nbsp;days until maturity
</template>

<script>
export default {
  data () {
    return {
      startingValue: 500,
      currentPrice: null,
      numberOfCoins: 200000000,
      series: [
        {
          name: 'Safemoon',
          data: [
            {
              x: '10-11-2021',
              y: 100
            },
            {
              x: '10-12-2021',
              y: 110
            },
            {
              x: '10-13-2021',
              y: 90
            }
          ]
        }
      ],
      chartOptions: {
        chart: {
          type: 'area',
          stacked: false,
          height: 350,
          zoom: {
            type: 'x',
            enabled: true,
            autoScaleYaxis: true
          },
          toolbar: {
            autoSelected: 'zoom'
          }
        },
        dataLabels: {
          enabled: false
        },
        markers: {
          size: 0
        },
        fill: {
          type: 'gradient',
          gradient: {
            shadeIntensity: 1,
            inverseColors: false,
            opacityFrom: 0.5,
            opacityTo: 0,
            stops: [0, 90, 100]
          }
        },
        yaxis: {
          // labels: {
          //   formatter (val) {
          //     return (val / 1000000).toFixed(0)
          //   }
          // },
          title: {
            text: 'Price'
          }
        },
        xaxis: {
          type: 'datetime',
          min: new Date('10-09-2021').getTime(),
          max: new Date('10-09-2039').getTime(),
          labels: {
            format: 'yyyy'
          }
        },
        tooltip: {
          shared: false
        }
      }

    }
  },
  computed: {
    currentValue () {
      return this.currentPrice * this.numberOfCoins
    },
    profitLossDollars () {
      return this.currentValue - this.startingValue
    },
    profitLossPercentage () {
      return (this.currentValue - this.startingValue) / this.startingValue * 100
    }
  },
  async mounted () {
    try {
      const response = await this.$axios.$get('https://api.coingecko.com/api/v3/coins/safemoon?localization=false&community_data=false&developer_data=false&sparkline=false')
      this.currentPrice = response.market_data.current_price.usd
    } catch (error) {
      console.error(error)
    }
  }
}

</script>
