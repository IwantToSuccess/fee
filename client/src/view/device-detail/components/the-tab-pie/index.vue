<template>
  <Tabs>
    <TabPane
      label="分布图"
      icon="md-contrast"
    >
      <div style='height:400'>
        <ve-pie  v-if="pieData!== undefined"  :data="pieData" :settings="chartSettings" ></ve-pie>
        <multiPie v-else
          :height="height"
          :multiData="innerData"
          :typeColor="sysTypeColor"
          :tooltip="innerToolTip"
          :multiOutData="outerData"
          :tooltipOut="outerToolTip"
          :isSpinShow="isSpinShow"
        ></multiPie>
      </div>
    </TabPane>
    <TabPane
      label="明细"
      icon="md-menu"
    >
      <Table
        :columns="tableColumns"
        :data="tableData"
        height="425"
      ></Table>
    </TabPane>
  </Tabs>
  </Card>
  </Row>
</template>
<script>
import multiPie from '@/view/components/multi-pie/multi-pie.vue'

export default {
  components: {
    multiPie
  },
  props: {
    innerData: {
      type: Array,
    },
    outerData: {
      type: Array,
    },
    tableData: {
      type: Array,
    },
    innerToolTip: {
      type: Array,
    },
    outerToolTip: {
      type: Array,
    },
    tableColumns: {
      type: Array,
    },
    isSpinShow: {
      type: Boolean,
    },
    pieData:{
      type: Object
    }
  },
  data () {
    return {
      height: 600,
      sysTypeColor: ['type', ['#BAE7FF', '#71E3E3', '#7FC9FE', '#ABF5F5', '#8EE0A1', '#BAF5C4']],
      browserColor: ['type', ['#ABF5F5', '#8EE0A1', '#BAF5C4', '#7FC9FE', '#BAE7FF', '#71E3E3']],
      chartSettings:{
        radius: 150,
        offsetY: 250
      },
      isSysShow: false,
      isBrowserShow: true
    }
  },
  mounted () {},
  methods: {}
}
</script>

<style lang="less" scoped>
</style>
