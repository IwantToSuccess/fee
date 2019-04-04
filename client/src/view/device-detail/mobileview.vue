<template>
  <div style="overflow:hidden">
    <Row :gutter="20">
        <Card shadow >
          <div
            slot="title"
            style="display:flex;align-items: center;justify-content: space-between;"
          >
            <b>系统分布</b>
          </div>
          <div style='height:400'>
            <multiPie
              :height="height"
              :multiData="systemRows"
              :typeColor="sysTypeColor"
              :tooltip="sysTooltip"
              :multiOutData="systemOutRows"
              :tooltipOut="sysOutTooltip1"
              :isSpinShow="isSysShow"
            ></multiPie>
          </div>
        </Card>
      </Row>
      <Row>
        <Card shadow>
          <div
            slot="title"
            style="display:flex;align-items: center;justify-content: space-between;"
          >
            <b>浏览器分布</b>
          </div>
          <div style='height:400'>
            <multiPie
              :height="height"
              :multiData="browserData"
              :typeColor="browserColor"
              :tooltip="browserTooltip"
              :multiOutData="browserOutData"
              :tooltipOut="browserOutTooltip"
              :isSpinShow="isBrowserShow"
            ></multiPie>
          </div>
        </Card>
    </Row>
  </div>
</template>

<script>
import moment from 'moment'
import InforCard from '_c/info-card'
import CountTo from '_c/count-to'
import {
  getSysCount,
  getChromeCount
} from '@/api/info'
import MenuCount from '@/view/menu-count/menu-count.vue'
import OnlineTime from '@/view/online-time/online-time.vue'
import 'v-charts/lib/style.css'
import DataSet from '@antv/data-set'
import multiPie from '@/view/components/multi-pie/multi-pie.vue'
import Loading from '@/view/components/loading/loading.vue'

export default {
  name: 'home',
  components: {
    InforCard,
    CountTo,
    MenuCount,
    OnlineTime,
    multiPie,
    Loading
  },
  data () {
    return {
      height: 400,
      dataEmpty: true,
      sysMonth: moment().format('YYYY-MM'),
      chromeMonth: moment().format('YYYY-MM'),
      systemRows: [],
      browserData: [],
      browserOutData: [],
      browserTooltip: [],
      browserOutTooltip: [],
      browser: [],
      sysHeight: 400,
      type: [],
      sysOutTooltip1: [],
      sysTooltip: [],
      systemOutRows: [],
      options3: {
        disabledDate (date) {
          return date && date.valueOf() > Date.now()
        }
      },
      sysTypeColor: ['type', ['#BAE7FF', '#71E3E3', '#7FC9FE', '#ABF5F5', '#8EE0A1', '#BAF5C4']],
      browserColor: ['type', ['#ABF5F5', '#8EE0A1', '#BAF5C4', '#7FC9FE', '#BAE7FF', '#71E3E3']],
      isSysShow: true,
      isBrowserShow: true
    }
  },

  async mounted () {
    this.getSysCount()
    this.getChromeCount()
  },
  methods: {
    sysDateChange (val) {
      this.sysMonth = val
      this.getSysCount()
    },
    chromeDateChange (val) {
      this.chromeMonth = val
      this.getChromeCount()
    },

    getViewData (skey, svalue, data = []) {
      var ret = {
        columns: [skey, svalue],
        rows: []
      }
      data.forEach(({
                        key,
                        value
                      }) => {
        ret.rows.push({
          [skey]: key,
          [svalue]: value
        })
      })
      return ret
    },
    async getSysCount () {
      const { data } = await getSysCount({
        month: this.sysMonth
      })

      let res = data.map(item => {
        return {
          ...item,
          key: item['type'] + '_' + item['key']
        }
      })
      const sysDv = new DataSet.View().source(res)
      sysDv.transform({
        type: 'percent',
        field: 'value',
        dimension: 'type',
        as: 'percent'
      })
      this.systemRows = sysDv.rows
      const sysOutDv = new DataSet.View().source(res)
      sysOutDv.transform({
        type: 'percent',
        field: 'value',
        dimension: 'key',
        as: 'percent'
      })
      this.systemOutRows = sysOutDv.rows
      this.sysTooltip = [
        'type*value*percent', (item, value, percent) => {
          percent = (percent * 100).toFixed(2) + '%'
          return {
            name: item,
            value: percent + '&nbsp;&nbsp;&nbsp;&nbsp;数量：' + value
          }
        }
      ]
      this.sysOutTooltip1 = [
        'key*value*percent', (item, value, percent) => {
          percent = (percent * 100).toFixed(2) + '%'
          return {
            name: item,
            value: percent + '&nbsp;&nbsp;&nbsp;&nbsp;数量：' + value
          }
        }
      ]
      if (this.systemRows && this.systemOutRows) {
        this.isSysShow = false
      }
      this.resize()
    },
    async getChromeCount () {
      const res2 = await getChromeCount({
        q: 'chrome',
        month: this.chromeMonth
      })
      const chromeData = []
      res2.data.forEach(item => {
        chromeData.push({
          type: item['browser'],
          key: item['version'],
          total_count: item['total_count']
        })
      })
      const dv = new DataSet.View().source(chromeData)
      dv.transform({
        type: 'percent',
        field: 'total_count',
        dimension: 'type',
        as: 'percent'
      })
      this.browserData = dv.rows
      const viewDv = new DataSet.View().source(chromeData)
      viewDv.transform({
        type: 'percent',
        field: 'total_count',
        dimension: 'key',
        as: 'percent'
      })
      this.browserOutData = viewDv.rows
      this.browserTooltip = [
        'type*total_count*percent', (item, count, percent) => {
          percent = (percent * 100).toFixed(2) + '%'
          return {
            name: item,
            value: percent + '&nbsp;&nbsp;&nbsp;&nbsp;数量：' + count
          }
        }
      ]
      this.browserOutTooltip = [
        'key*total_count*percent', (item, count, percent) => {
          percent = (percent * 100).toFixed(2) + '%'
          return {
            name: item,
            value: percent + '&nbsp;&nbsp;&nbsp;&nbsp;数量：' + count
          }
        }
      ]
      if (this.browserData && this.browserOutData) {
        this.isBrowserShow = false
      }
    }
  }
}
</script>

<style lang="less" scoped>
.count-style {
  font-size: 50px;
}
</style>
