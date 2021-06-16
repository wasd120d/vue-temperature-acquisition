<template>
  <div class="echarts">
    <el-row :gutter="20">
      <el-col :span="12">
        <div id="chart3" :style="{width: '100%', height: '200px'}"></div>
      </el-col>
      <el-col :span="12">
        <div id="chart4" :style="{width: '100%', height: '300px'}"></div>
      </el-col>
    </el-row>
  </div>

</template>

<script>

import mqtt from 'mqtt'

export default {
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
      //实时数据组
      tempPerSecArray:[],
      xArray:[],
      yArray:[],
      client: {
        connected: false,
      },

      myChart3: null,
      eChartOpinion:{
        title: { text: 'tempPerSecArray' },
        tooltip: {},
        xAxis: {
          type: 'category',
          data: this.xArray,
        },
        yAxis: {
          type: 'value'
        },
        series: [{
          type: 'line',
          data: this.yArray,
        }]
      },

    }
  },
  created(){
    // 连接选项
      const options = {
            clean: true, // true: 清除会话, false: 保留会话
            connectTimeout: 4000, // 超时时间
            // 认证信息
            clientId: 'emqx_test',
            username: 'emqx_test',
            password: 'emqx_test',
      }

      // 连接字符串, 通过协议指定使用的连接方式
      // ws 未加密 WebSocket 连接
      // wss 加密 WebSocket 连接
      // mqtt 未加密 TCP 连接
      // mqtts 加密 TCP 连接
      // wxs 微信小程序连接A
      // alis 支付宝小程序连接
      //const connectUrl = 'wss://broker.emqx.io:8084/mqtt'
      const connectUrl = 'ws://1.117.42.36:8083/mqtt'
      const client = mqtt.connect(connectUrl, options)

      //创建连接，显示连接失误
      try {
        this.client = mqtt.connect(connectUrl, options)
      } catch(error){
        console.log('mqtt.connect error', error)
      }

      this.client.on('connect', () => {
        console.log('成功连接服务端')
      })

      this.client.subscribe(['BOX000001/UNT000001/BAT000001/TEMP/AVG'], function(err) {
        if (!err) {
          console.log('订阅')
        }
      })

      this.client.on('error', (error) => {
       console.log('连接失败:', error)
      })
      //mqtt数据获取
      this.client.on('message', (topic, message) => {
        console.log(message.toString());
        this.mqtt_msg = message.toString()
        var json_msg = JSON.parse(this.mqtt_msg)

        this.tempPerSecArray.push(this.storeTempPSec(json_msg.ts, json_msg.avg))
        if(this.tempPerSecArray.length > 10){
          this.tempPerSecArray.shift()
        }
        this.eChartOpinion.xAxis.data = this.tempPerSecArray.map((item)=>{ return item.name });
        this.eChartOpinion.series[0].data = this.tempPerSecArray.map((item)=>{ return item.value });
        console.log(this.tempPerSecArray)
      })
  },
  mounted () {
    this.draw1()
    this.myChart3 = this.$echarts.init(document.getElementById('chart4'))
    setInterval(this.draw3,1000)
  },
  methods: {
    addData(){
    },
    storeTempPSec(ts, value){
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
    draw3 () {
      // 基于准备好的dom，初始化echarts实例，注意id不要写错
      // console.log(this.eChartOpinion)
      // 绘制图表
      this.myChart3.setOption(this.eChartOpinion)
    }
  }
}
</script>