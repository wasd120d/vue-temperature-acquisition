<template>
  <div>

    <el-button type="primary" @click="on_click">第一次渲染曲线</el-button>
    <el-button type="primary" @click="on_click_2">更新一次数据</el-button>
    <div>{{mqtt_msg}}</div>
    <!-- echarts -->
    <div id="main" class="main_container">



    </div>
  </div>
</template>
<script>
import mqtt from 'mqtt'

import * as echarts from 'echarts';


export default {
  data() {
    return {

       mqtt_msg:null,
       chartDom:null,
       myChart :null,

       option : {
           xAxis: {
               type: 'category',
               data: ['1', '1', '1', '1', '1', '1', '1']
           },
           yAxis: {
               type: 'value'
           },
           series: [{
               data: [0, 0 ,0, 0, 0, 0, 0],
               type: 'line'
           }]
       },
    }
  },
  computed:{

  //option && myChart.setOption(option),
  },
  mounted() {

  },

  created() {
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
          console.log('成功连接服务端')

          client.subscribe('TEMP/SEN000001', function (err) {
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

      client.on('message', (topic, message) => {
        this.mqtt_msg = JSON.parse(message.toString())['temperature']
        //console.log('收到消息：', topic, message)
       //console.log('收到消息：', topic, parseFloat(message.toString().substr(15,4)))
        //JSON.parse(s
         console.log('收到消息：', topic, JSON.parse(message.toString())['temperature'])
         //console.log("这里应该做一次位运算")
         this.option.series[0].data.push(this.mqtt_msg)
         this.option.series[0].data.shift()
         console.log('this.option.series[0].data',this.option.series[0].data)

         //this.option.series[0].data[6]=this.mqtt_msg
         //console.log('推过之后的数据' ,this.option.series[0].data)
        // console.log('重新配置数据')

         //this.myChart.setOption(this.option);

         //console.log('收到消息：', topic, message.toString())
        //this.initCharts()
      })
    },


  methods: {



    on_click() {
      console.log('看看option',this.option.series[0].data)
      //this.option.series[0].data.push([150, 230, 224, 218, 135, 147, 130])

      // 初始化echarts实例
     this.chartDom = document.getElementById('main');
     this.myChart = echarts.init(this.chartDom);
     //var option;


     this.option && this.myChart.setOption(this.option);
    },

    on_click_2() {
      //this.option.series[0].data.push(200)
      /*
     this.option = {
          xAxis: {
              type: 'category',
              data: ['Mon', 'Tue', 'Wed', 'Thu', 'Fri', 'Sat', 'Sun']
          },
          yAxis: {
              type: 'value'
          },
          series: [{
              data: [0, 0, 0, 0, 0, 0, this.mqtt_msg],
              type: 'line'
          }]
      },*/
       //this.option.series[0].data = [0, 0, 0, 0, 0, 0, this.mqtt_msg]
      //this.option.series[0].data[6]=100
      //console.log('推过之后的数据' ,this.option.series[0].data)
      console.log('重新配置数据')

      this.myChart.setOption(this.option);



    },
  },
}
</script>
<style lang="scss" scoped>
.main_container {
  width: 100%;
  height: 300px;
  overflow: hidden;
}
</style>
