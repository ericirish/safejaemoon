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

    v-row.mb-10(align-content="stretch")
      v-col(
        cols="12"
        md="4"
      )
        v-card(
          height="100%"
          :elevation="10"
        )
          v-sheet(
            dense
            color="primary lighten-4"
          )
            v-card-subtitle Original Value
          v-card-text.text-right
            .text-h5.primary--text ${{ startingValue }}
      v-col(
        cols="12"
        md="4"
      )
        v-card(
          height="100%"
          elevation="10"
        )
          v-sheet(
            dense
            color="primary lighten-4"
          )
            v-card-subtitle Current Value
          v-card-text.primary--text
            .text-h5.text-right ${{ currentValue }}
      v-col(
        cols="12"
        md="4"
      )
        v-card(
          height="100%"
          :elevation="10"
        )
          v-sheet(
            dense
            color="primary lighten-4"
          )
            v-card-subtitle Profit/Loss
          v-card-text(
            :class="{ 'success--text' : isProfitable, 'error--text': !isProfitable }"
          )
            .text-h5.text-right ${{ profitLossDollars }}
              | &nbsp;
              v-icon {{ profitLossPercentage > 0 ? 'mdi-arrow-up' : 'mdi-arrow-down' }}
              | {{ Math.abs(profitLossPercentage) }}%
    .text-center
      v-icon mdi-ethereum
      small {{ numberOfCoins }} ETH

    v-row
      v-col
        apexchart(
          type="area" height="350" :options="chartOptions" :series="series"
        )
        h2(
          style="text-align: center; font-weight: 300"
        )
          strong {{ Math.abs($dayjs.duration($dayjs().diff($dayjs(startDate).add(18, 'years'))).asDays()).toFixed() }}
          | &nbsp;days until maturity
</template>

<script>
export default {
  data () {
    return {
      startingValue: 541.93,
      currentPrice: null,
      numberOfCoins: 0.14988643,
      historicalPrices: [],
      startDate: null,
      // series: [
      //   {
      //     name: 'Ethereum',
      //     data: [
      //       {
      //         x: '10-11-2021',
      //         y: 100
      //       },
      //       {
      //         x: '10-12-2021',
      //         y: 110
      //       },
      //       {
      //         x: '10-13-2021',
      //         y: 90
      //       }
      //     ]
      //   }
      // ],
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
          forceNiceScale: true,
          title: {
            text: 'Percentage Profit/Loss'
          },
          decimalsInFloat: 0
        },
        xaxis: {
          type: 'datetime',
          min: new Date('8-01-2021').getTime(),
          max: new Date('10-09-2039').getTime(),
          labels: {
            format: 'yyyy'
          }
        },
        tooltip: {
          shared: false,
          x: {
            format: 'M/d/yy'
          },
          y: {
            formatter (value, { series, seriesIndex, dataPointIndex, w }) {
              return value + '%'
            },
            title: ''
          }
        }
      }

    }
  },
  computed: {
    currentValue () {
      return (this.currentPrice * this.numberOfCoins).toFixed(2)
    },
    profitLossDollars () {
      return (this.currentValue - this.startingValue).toFixed(2)
    },
    profitLossPercentage () {
      return ((this.currentValue - this.startingValue) / this.startingValue * 100).toFixed(2)
    },
    isProfitable () {
      return this.profitLossDollars > 0
    },
    series () {
      const data = this.historicalPrices

      data.forEach((point) => {
        const value = point[1] * this.numberOfCoins

        point[1] = ((value - this.startingValue) / this.startingValue * 100).toFixed(2)
      })

      return [
        {
          name: 'Profit/Loss',
          data
        }
      ]
    }
  },
  created () {
    this.startDate = this.$dayjs('10/09/2021')
  },
  async mounted () {
    try {
      const response = await this.$axios.$get('https://api.coingecko.com/api/v3/coins/ethereum?localization=false&community_data=false&developer_data=false&sparkline=false')
      this.currentPrice = response.market_data.current_price.usd
    } catch (error) {
      console.error(error)
    }

    let daysSince = this.$dayjs.duration(this.$dayjs().diff(this.startDate)).asDays()
    daysSince = Math.floor(daysSince)

    try {
      const response = await this.$axios.$get(`https://api.coingecko.com/api/v3/coins/ethereum/market_chart?vs_currency=usd&days=${daysSince}&interval=daily`)
      console.log(response)
      this.historicalPrices = response.prices
    } catch (error) {
      console.error(error)
    }
  }
}

</script>
