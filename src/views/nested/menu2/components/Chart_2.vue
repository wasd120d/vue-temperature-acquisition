<template>
  <div class="echarts">
    <el-row :gutter="20">
      <el-select v-model="select_value" placeholder="请选择" class="selection" @change="changeChart()">
        <el-option
          v-for="item in select_options"
          :key="item.value"
          :label="item.label"
          :value="item.value">
        </el-option>
      </el-select>
    </el-row>
    <el-row :gutter="20">
      <el-col>
        <div :id="chartID" :style="{width: '100%', height: '300px'}"></div>
      </el-col>
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
  data () {
    return {
      mqtt_msg: null,
      //图表数据
      tempPerSecArray: [],
      avgArray:[],
      avgArray:[],
      avgArray:[],
      xArray:[],
      yArray:[],

      client: {
        connected: false,
      },
      select_options: [
        {
          value: 1,
          label: 'BOX000001/UNT000001/BAT000001/TEMP'
        }, 
        {
          value: 2,
          label: 'BOX000001/UNT000001/BAT000001/TEMP'
        }, 
      ],
      select_value:'',
      myChart3: null,
      eChartOpinion:{
        title: { text: 'chart_bat' },
        tooltip: {},
        xAxis: {
          type: 'category',
          data: this.xArray,
        },
        yAxis: {
          type: 'value'
        },
        series: [
          {
            name: 'avg',
            type: 'line',
            data: [],
          },
          {
            name: 'max',
            type: 'line',
            data: this.maxArray,
          },
          {
            name: 'min',
            type: 'line',
            data: this.minArray,
          },
        ]
      },

    }
  },
  created(){
  },
  mounted () {
    this.myChart3 = this.$echarts.init(document.getElementById(this.chartID))
    setInterval(this.drawEchart,1000)
  },
  methods: {
    changeChart() {
      // console.log("组内相对号",this.select_value)
      // console.log("tessssssssssssss:",this.select_options[this.select_value-1])
      this.avgArray = []
      this.maxArray = []
      this.minArray = []
      this.subTopic(this.select_options[this.select_value-1].label)
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
        // json_msg格式{"avg":,"max":,"min":,"ts":}
        // 时间戳
        console.log(Date(json_msg.ts))
        // console.log('收到消息：', topic, message.toString())

        if (topic_r === topic) {
          this.tempPerSecArray.push(this.storeTempP(json_msg.ts, json_msg.avg, json_msg.max, json_msg.min))
          if (this.tempPerSecArray.length > 50) {
            this.tempPerSecArray.shift()
          }
          this.eChartOpinion.xAxis.data = this.tempPerSecArray.map((item) => { return item.ts })
          this.eChartOpinion.series[0].data = this.tempPerSecArray.map((item) => { return item.avg })
          this.eChartOpinion.series[1].data = this.tempPerSecArray.map((item) => { return item.max })
          this.eChartOpinion.series[2].data = this.tempPerSecArray.map((item) => { return item.min })
          //console.log(this.tempPerSecArray)
        }
      })
    },

    storeTempP(ts, avg, max, min){
      const now = new Date(ts)
      var valueName =
        (now.getHours() >= 10 ? now.getHours() : '0' + now.getHours()) +
        ':' +
        (now.getMinutes() >= 10 ? now.getMinutes() : '0' + now.getMinutes()) +
        ':' +
        (now.getSeconds() >= 10 ? now.getSeconds() : '0' + now.getSeconds())
      return {
        ts: valueName, avg: avg, max: max, min: min
        }
    },
    draw1 () {
      // 基于准备好的dom，初始化echarts实例，注意id不要写错
      let myChart = this.$echarts.init(document.getElementById('chart3'))
      var option = {
        title: {
          text: '折线图堆叠'
        },
        tooltip: {
          trigger: 'axis'
        },
        legend: {
          data: ['邮件营销', '联盟广告']
        },
        grid: {
          left: '3%',
          right: '4%',
          bottom: '3%',
          containLabel: true
        },
        toolbox: {
          feature: {
            saveAsImage: {}
          }
        },
        xAxis: {
          type: 'category',
          boundaryGap: false,
          data: ['周一', '周二', '周三', '周四', '周五', '周六', '周日']
        },
        yAxis: {
          type: 'value'
        },
        series: [
          {
            name: '邮件营销',
            type: 'line',
            stack: '总量',
            data: [120, 132, 101, 134, 90, 230, 210]
          },
          {
            name: '联盟广告',
            type: 'line',
            stack: '总量',
            data: [220, 182, 191, 234, 290, 330, 310]
          },  
        ]
      };
      // 绘制图表
      myChart.setOption(option)
    },
    drawEchart () {
      // 基于准备好的dom，初始化echarts实例，注意id不要写错
      // console.log(this.eChartOpinion)
      // 绘制图表
      this.myChart3.setOption(this.eChartOpinion)
    }
  }
}
</script>