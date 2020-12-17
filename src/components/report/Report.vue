<template>
    <div>
              <!-- 面包屑导航 -->
      <el-breadcrumb separator-class="el-icon-arrow-right">
          <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
            <el-breadcrumb-item>数据统计</el-breadcrumb-item>
            <el-breadcrumb-item>数据报表</el-breadcrumb-item>
       </el-breadcrumb> 
          <!-- 卡片视图区 -->
          
           <el-card class="box-card">
                 <div id="main" style="width:800px;height:450px;"></div>
           </el-card>

    </div>
</template>

<script>
let echarts = require('echarts')
import _ from 'lodash'

export default {
  // 生命周期函数
    created(){},
    // 页面元素渲染完毕
   async mounted(){
    // 基于准备好的dom，初始化echarts实例
     var myChart = echarts.init(document.getElementById('main'));
     const {data :res}= await this.$http.get('reports/type/1')
     if(res.meta.status !==200){
         return this.$message.error("获取折线图失败！·")
     }
        // 指定图表的配置项和数据
      const result =  _.merge( res.data , this.options)
 
        // 使用刚指定的配置项和数据显示图表。
     myChart.setOption(result);
    },

    // 数据
    // 需要合并的数据
    data(){
        return{
        options: {
            title: {
                text: 'ECharts 入门示例'
            },
            tooltip: {},
            legend: {
                data:['销量']
            },
            xAxis: {
                data: ["衬衫","羊毛衫","雪纺衫","裤子","高跟鞋","袜子"]
            },
            yAxis: {},
            series: [{
                name: '销量',
                type: 'bar',
                data: [5, 20, 36, 10, 10, 20]
            }]
        },
        }
    },

    // 方法/函数
    methods:{},
    
}
</script>

<style lang="less" scoped>
*{
    margin: 0px;
    padding: 0px;
}

</style>