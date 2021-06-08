<template>
  <div style="padding:30px;">
    <el-alert :closable="false" title="menu 2" />
    <HelloWorld msg="Welcome to Your Vue.js App"/>
    <Chart2 msg="Welcomet"/>
  </div>
</template>

<script>
import HelloWorld from './components/HelloWorld.vue'
import Chart2 from './components/Chart_2.vue'

export default {
  name: 'App',
  components: {
    HelloWorld,
    Chart2
  }
}
</script>

