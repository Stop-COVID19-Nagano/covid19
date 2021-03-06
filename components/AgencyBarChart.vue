<template>
  <data-view :title="title" :title-id="titleId" :date="date" :url="url">
    <template v-slot:button>
      <small :class="$style.DataViewDesc">
        ※土・日・祝日を除く庁舎開庁日の1週間累計数
      </small>
    </template>
    <bar
      :chart-id="chartId"
      :chart-data="displayData"
      :options="displayOption"
      :height="240"
    />
  </data-view>
</template>

<style module lang="scss">
.DataView {
  &Desc {
    margin-top: 10px;
    margin-bottom: 0 !important;
    font-size: 12px;
    color: $gray-3;
  }
}
</style>

<script>
import agencyData from '@/data/agency.json'
import DataView from '@/components/DataView.vue'
import { getGraphSeriesStyle } from '@/utils/colors'

export default {
  components: { DataView },
  props: {
    title: {
      type: String,
      required: false,
      default: ''
    },
    titleId: {
      type: String,
      required: false,
      default: ''
    },
    chartId: {
      type: String,
      required: false,
      default: 'agency-bar-chart'
    },
    unit: {
      type: String,
      required: false,
      default: ''
    },
    url: {
      type: String,
      required: false,
      default: ''
    }
  },
  data() {
    const agencies = [
      this.$t('第一庁舎計'),
      this.$t('第二庁舎計'),
      this.$t('議事堂計')
    ]
    agencyData.datasets.map(dataset => {
      dataset.label = this.$t(dataset.label)
    })
    return {
      chartData: agencyData,
      date: agencyData.date,
      agencies
    }
  },
  computed: {
    displayData() {
      const graphSeries = getGraphSeriesStyle(this.chartData.datasets.length)
      return {
        labels: this.chartData.labels,
        datasets: this.chartData.datasets.map((item, index) => {
          return {
            label: this.agencies[index],
            data: item.data,
            backgroundColor: graphSeries[index].fillColor,
            borderColor: graphSeries[index].strokeColor,
            borderWidth: 1
          }
        })
      }
    },
    displayOption() {
      const self = this
      return {
        tooltips: {
          displayColors: false,
          callbacks: {
            title(tooltipItem) {
              const dateString = tooltipItem[0].label
              return self.$t('期間: {duration}', {
                duration: dateString
              })
            },
            label(tooltipItem, data) {
              const index = tooltipItem.datasetIndex
              const title = self.$t(data.datasets[index].label)
              const num = tooltipItem.value
              const unit = self.$t(self.unit)
              return `${title}: ${num}${unit}`
            }
          }
        },
        legend: {
          display: true,
          onHover: e => {
            e.currentTarget.style.cursor = 'pointer'
          },
          onLeave: e => {
            e.currentTarget.style.cursor = 'default'
          }
        },
        scales: {
          xAxes: [
            {
              stacked: true,
              gridLines: {
                display: false
              },
              ticks: {
                fontSize: 9,
                fontColor: '#808080'
              }
            }
          ],
          yAxes: [
            {
              stacked: true,
              gridLines: {
                display: true
              },
              ticks: {
                fontSize: 9,
                fontColor: '#808080',
                maxTicksLimit: 10,
                stepSize: 1,
                callback(label) {
                  return `${label}${self.unit}`
                }
              }
            }
          ]
        }
      }
    }
  }
}
</script>
