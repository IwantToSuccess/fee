<template>
  <Row :gutter="20">
    <Col span="24">
    <Card shadow>
      选择日期：
      <DatePicker :value="selectMonth"
                  type="month"
                  placeholder="选择月份"
                  style="width: 200px"
                  @on-change="dateChange"
                  :options="options3"
                  :clearable="false" />
    </Card>
    <Card shadow>
      <Tabs type="card">
        <TabPane label="系统分布">
          <the-tab-pie :innerData='systemRows'
                       :outerData='systemOutRows'
                       :innerToolTip='sysTooltip'
                       :outerToolTip='sysOutTooltip'
                       :tableData='table.data.OS'
                       :tableColumns='table.config.OSTableColumns'
                       :isSpinShow='isSysShow'></the-tab-pie>
        </TabPane>
        <TabPane label="浏览器分布">
          <the-tab-pie :innerData='browserData'
                       :outerData='browserOutData'
                       :tableData='table.data.browser'
                       :innerToolTip='browserTooltip'
                       :outerToolTip='browserOutTooltip'
                       :tableColumns='table.config.browserTableColumns'
                       :isSpinShow='isBrowserShow'></the-tab-pie>
        </TabPane>
      </Tabs>
    </Card>
    </Col>
  </Row>
</template>
<script>
  import moment from 'moment'
  import {
    getSysCount,
    getChromeCount
  } from '@/api/info'
  import DataSet from '@antv/data-set'
  import TheTabPie from './components/the-tab-pie'

  let OSTableColumns = [
    {
      title: ' ',
      type: 'index'
    },
    {
      title: '系统',
      key: 'type'
    }, {
      title: '版本',
      key: 'key'
    }, {
      title: '个数',
      key: 'value',
      sortable: true
    }
  ]
  let browserTableColumns = [
    {
      title: '浏览器',
      key: 'browser'
    }, {
      title: '版本',
      key: 'version'
    }, {
      title: '个数',
      key: 'total_count',
      sortable: true
    }
  ]
  let sysOutTooltip = [
    'key*value*percent', (item, value, percent) => {
      percent = (percent * 100).toFixed(2) + '%'
      return {
        name: item,
        value: percent + '&nbsp;&nbsp;&nbsp;&nbsp;数量：' + value
      }
    }
  ]
  let sysTooltip = [
    'type*value*percent', (item, value, percent) => {
      percent = (percent * 100).toFixed(2) + '%'
      return {
        name: item,
        value: percent + '&nbsp;&nbsp;&nbsp;&nbsp;数量：' + value
        // value: '数量：' + count
      }
    }
  ]
  let browserTooltip = [
    'type*total_count*percent', (item, count, percent) => {
      percent = (percent * 100).toFixed(2) + '%'
      return {
        name: item,
        value: percent + '&nbsp;&nbsp;&nbsp;&nbsp;数量：' + count
      }
    }
  ]
  let browserOutTooltip = [
    'key*total_count*percent', (item, count, percent) => {
      percent = (percent * 100).toFixed(2) + '%'
      return {
        name: item,
        value: percent + '&nbsp;&nbsp;&nbsp;&nbsp;数量：' + count
      }
    }
  ]

  export default {
    components: {
      TheTabPie
    },
    data () {
      return {
        table: {
          config: {
            OSTableColumns,
            browserTableColumns
          },
          data: {
            OS: [],
            browser: []
          }
        },
        dataEmpty: true,
        // 时间选择器
        selectMonth: moment().format('YYYY-MM'),
        options3: {
          disabledDate (date) {
            return date && date.valueOf() > Date.now()
          }
        },
        // 系统图数据
        sysOutTooltip,
        sysTooltip,
        systemRows: [],
        systemOutRows: [],
        // 浏览器图数据
        browserData: [],
        browserOutData: [],
        browserTooltip,
        browserOutTooltip,
        //应用版本数据
        isSysShow: true,
        isBrowserShow: true,
      }
    },
    mounted () {
      this.getSysCount()
      this.getChromeCount()
    },
    methods: {
      dateChange (val) {
        this.selectMonth = val
        this.getSysCount()
        this.getChromeCount()
        this.getSysRuntimeVersion()
      },
      getViewData (skey, svalue, data = []) {
        var ret = {
          columns: [skey, svalue],
          rows: []
        }
        data.forEach(({ key, value }) => {
          ret.rows.push({
            [skey]: key,
            [svalue]: value
          })
        })
        return ret
      },
      async getSysCount () {
        const { data } = await getSysCount({
          month: this.selectMonth
        })
        this.table.data.OS = data.sort((a, b) => b['value'] - a['value'])
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
        if (this.systemRows && this.systemOutRows) {
          this.isSysShow = false
        }
        this.resize()
      },
      async getChromeCount () {
        const res2 = await getChromeCount({
          q: 'chrome',
          month: this.selectMonth
        })
        this.table.data.browser = res2.data.sort((a, b) => b['total_count'] - a['total_count'])
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
        if (this.browserData && this.browserOutData) {
          this.isBrowserShow = false
        }
      }
    }
  }
</script>

<style lang="less" scoped>
</style>
