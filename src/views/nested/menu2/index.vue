<template>
  <div style="padding:30px;">
    <el-alert :closable="false" title="menu 2" />
    <HelloWorld msg="Welcome to Your Vue.js App"/>
  </div>
</template>

<script>
import HelloWorld from './components/HelloWorld.vue'
// import Index from './components/index.vue'

export default {
  name: 'App',
  components: {
    HelloWorld,
  }
}
</script>

