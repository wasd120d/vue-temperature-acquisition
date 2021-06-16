<template>
  <div class="index" style="padding:30px;">
    <el-row :gutter="20">
      <el-col :span="12">
        <div> MQTT测试 </div>
      </el-col>
      <el-col :span="12">
        <div> MQTT测试 </div>
      </el-col>
    </el-row>
    <el-row :gutter="20">
      <el-col :span="12">
        <Chart2 id='chart_bat' chartID="ID_chart_Bat"/>
      </el-col>
      <el-col :span="12">
        <Chart1 id='chart_sen' chartID="ID_chart_sen"/>
      </el-col>
    </el-row>
  </div>
</template>

<script>
import Chart1 from './components/Chart_1.vue'
import Chart2 from './components/Chart_2.vue'

export default {
  name: 'App',
  data() {
    return {
      chartSenID:'ID_chart_sen',
    }
  },
  components: {
    Chart1,
    Chart2,

  }
}
</script>

