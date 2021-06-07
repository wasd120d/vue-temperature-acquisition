<template>
  <div class="echarts">
    <el-row :gutter="20">
      <el-col :span="12">
        <div id="chart1" :style="{width: '100%', height: '200px'}"></div>
      </el-col>
      <el-col :span="12">
        <div id="chart2" :style="{width: '100%', height: '300px'}"></div>
      </el-col>
    </el-row>
    
    <div>MQTT测试</div>
    <div>{{mqtt_msg}}</div>
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
      mqtt_msg:null,
      //实时数据组
      tempPerSecArray:[],
      xArray:[],
      yArray:[],

      myChart2: null,
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

      client.on('connect', function (connack) {
          if(connack){
            console.log('成功连接服务端')
          }
          client.subscribe(['TEMP/SEN000001', 'TEMP/SEN000001/AVG10S','TEMP/SEN000001/COUNT10S'], function (err) {
            if (!err) {
              console.log("订阅")
              //client.publish('TEMP/SEN000001', 'Hello--来自网页客户端的消息')
            }
          })
      })

      client.on('reconnect', (error) => {
          console.log('正在重连:', error)
      })

      client.on('error', (error) => {
          console.log('连接失败:', error)
      })
      //mqtt数据获取
      client.on('message', (topic, message) => {
        console.log(message.toString());
        this.mqtt_msg = message.toString()
        var json_msg = JSON.parse(this.mqtt_msg)
        console.log(json_msg.temperature);
        // console.log('收到消息：', topic, message.toString())

        var now = new Date()
        if(topic == 'TEMP/SEN000001'){
            this.tempPerSecArray.push(this.storeTempPSec(now, json_msg.temperature))
          if(this.tempPerSecArray.length > 10){
            this.tempPerSecArray.shift()
          }
          this.eChartOpinion.xAxis.data = this.tempPerSecArray.map((item)=>{return item.name});
          this.eChartOpinion.series[0].data = this.tempPerSecArray.map((item)=>{return item.value});
          console.log(this.tempPerSecArray)
          // console.log(this.yArray)
        }
      })
  },
  mounted () {
    this.draw1()
    this.myChart2 = this.$echarts.init(document.getElementById('chart2'))
    setInterval(this.draw2,1000)
  },
  methods: {
    addData(){
    },
    storeTempPSec(now, value){
      var valueName =
        (now.getHours() >= 10 ? now.getHours() : '0' + now.getHours()) +
        ':' +
        (now.getMinutes() >= 10 ? now.getMinutes() : '0' + now.getMinutes()) +
        ':' +
        (now.getSeconds() >= 10 ? now.getSeconds() : '0' + now.getSeconds())
      return {
        name:valueName, value:value,
        }
    },
    draw1 () {
      // 基于准备好的dom，初始化echarts实例，注意id不要写错
      let myChart = this.$echarts.init(document.getElementById('chart1'))
      // 绘制图表
      myChart.setOption({
        title: { text: '实时温度' },
        tooltip: {
          trigger: 'axis',
          formatter: function(params) {
            params = params[0]
            var date = new Date(params.name)
            return (
              date.getDate() +
              '/' +
              (date.getMonth() + 1) +
              '/' +
              date.getFullYear() +
              ' : ' +
              params.value[1]
            )},
            axisPointer: {
            animation: false,
          },
        },
        xAxis: {
          data: ['衬衫', '羊毛衫', '雪纺衫', '裤子', '高跟鞋', '袜子']
        },
        yAxis: {},
        series: [{
          name: '销量',
          type: 'bar',
          data: [5, 20, 36, 10, 10, 20]
        }]
      })
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