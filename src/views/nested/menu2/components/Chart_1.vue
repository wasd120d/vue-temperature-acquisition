<template>
  <div class="echarts">
    <el-row :gutter="20">
      <el-select v-model="select_value_4" placeholder="选传感器" class="selection" @change="change_4()">
        <el-option
          v-for="item in select_options_4"
          :key="item.value"
          :label="item.label"
          :value="item.value">
        </el-option>
      </el-select>
    </el-row>
    <el-row :gutter="20">
        <div :id="chartID" :style="{width: '100%', height: '300px'}"></div>
    </el-row>
  </div>
</template>

<script>

import mqtt from 'mqtt'

export default {
  props:{
    chartID:{},
  },
  filters: {
    statusFilter(status) {
      const statusMap = {
        published: 'success',
        draft: 'gray',
        deleted: 'danger'
      }
      return statusMap[status]
    }
  },
  name: 'Echarts',
  data() {
    return {
      mqtt_msg: null,
      // 实时数据组
      tempPerSecArray: [],
      xArray: [],
      yArray: [],
      client: {
        connected: false,
      },
      myChart2: null,
      //选择传感器
      select_options_4: [{
          value: 1,
          label: 'BOX000001/UNT000001/BAT000001/SEN000001/TEMP'
        }, {
          value: 2,
          label: 'BOX000001/UNT000001/BAT000001/SEN000002/TEMP'
        }, ],
      select_value_4:'',

      eChartOpinion: {
        title: { text: 'tempPerSecArray' },
        tooltip: {},
        xAxis: {
          type: 'category',
          data: this.xArray
        },
        yAxis: {
          type: 'value'
        },
        series: [{
          type: 'line',
          data: []
        }]
      }
    }
  },
  created() {
  },
  mounted() {
    this.myChart2 = this.$echarts.init(document.getElementById(this.chartID))
    setInterval(this.draw2, 1000)
  },
  methods: {
    change_4() {
      // console.log("组内相对号",this.select_value_4)
      // console.log("tessssssssssssss:",this.select_options_4[this.select_value_4-1])
      this.tempPerSecArray = []
      this.subTopic(this.select_options_4[this.select_value_4-1].label)
    },
    subTopic(topic) {
      // 连接选项
      const options = {
        clean: true, // true: 清除会话, false: 保留会话
        connectTimeout: 4000, // 超时时间
        // 认证信息
        clientId: 'emqx_test',
        username: 'emqx_test',
        password: 'emqx_test'
      }

      // 连接字符串, 通过协议指定使用的连接方式
      // ws 未加密 WebSocket 连接
      // wss 加密 WebSocket 连接
      // mqtt 未加密 TCP 连接
      // mqtts 加密 TCP 连接
      // wxs 微信小程序连接A
      // alis 支付宝小程序连接
      const connectUrl = 'ws://1.117.42.36:8083/mqtt'
      //创建连接，显示连接失误
      try {
        this.client = mqtt.connect(connectUrl, options)
      } catch(error){
        console.log('mqtt.connect error', error)
      }
      console.log('client数据：',this.client)
      
      this.client.on('connect', () => {
        console.log('成功连接服务端')
      })

      this.client.subscribe([topic], function(err) {
        if (!err) {
          console.log('订阅')
        }
      })

      this.client.on('error', (error) => {
        console.log('连接失败:', error)
      })
      // mqtt数据获取
      this.client.on('message', (topic_r, message) => {
        this.mqtt_msg = message.toString()
        var json_msg = JSON.parse(this.mqtt_msg)
        // json_msg格式{"device_id", "celcius", "ts"}
        // 时间戳
        // console.log(Date(json_msg.ts))
        // console.log('收到消息：', topic, message.toString())

        if (topic_r === topic) {
          this.tempPerSecArray.push(this.storeTempPSec(json_msg.ts, json_msg.celcius))
          if (this.tempPerSecArray.length > 50) {
            this.tempPerSecArray.shift()
          }
          this.eChartOpinion.xAxis.data = this.tempPerSecArray.map((item) => { return item.name })
          this.eChartOpinion.series[0].data = this.tempPerSecArray.map((item) => { return item.value })
          // console.log(this.tempPerSecArray)
        }
      })
    },
    storeTempPSec(ts, value) {
      const now = new Date(ts)
      var valueName =
        (now.getHours() >= 10 ? now.getHours() : '0' + now.getHours()) +
        ':' +
        (now.getMinutes() >= 10 ? now.getMinutes() : '0' + now.getMinutes()) +
        ':' +
        (now.getSeconds() >= 10 ? now.getSeconds() : '0' + now.getSeconds())
      return {
        name: valueName, value: value,
        }
    },
    draw2 () {
      // 基于准备好的dom，初始化echarts实例，注意id不要写错
      // console.log(this.eChartOpinion)
      // 绘制图表
      this.myChart2.setOption(this.eChartOpinion)
    }
  }
}
</script>