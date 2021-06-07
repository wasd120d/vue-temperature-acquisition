<template>
  <div class="app-container">
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
  data() {
    return {
      mqtt_msg:null,
    }
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

      client.on('message', (topic, message) => {
        this.mqtt_msg = message.toString()
        console.log('收到消息：', topic, message.toString())
      })
    },
  methods: {
  }
}
</script>
