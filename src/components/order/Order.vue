<template>
    <div>
              <!-- 面包屑导航 -->
      <el-breadcrumb separator-class="el-icon-arrow-right">
          <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
            <el-breadcrumb-item>订单管理</el-breadcrumb-item>
            <el-breadcrumb-item>订单列表</el-breadcrumb-item>
       </el-breadcrumb> 

          <!-- 卡片视图区 -->
        <el-card class="box-card">
            <!-- 1.搜索框 -->
            <el-row>
                <el-col :span="8">
                         <el-input placeholder="请输入内容">
                            <el-button slot="append" icon="el-icon-search"></el-button>
                        </el-input>
                </el-col>
            </el-row>
            <!-- 2.表格 -->
             <el-table :data="orderlist" border stripe  style="width: 100%">
                  <el-table-column type="index" label="#"> </el-table-column>
                  <el-table-column  prop="order_number" label="订单编号" width="280"> </el-table-column>
                  <el-table-column prop="order_price"  label="订单价格"></el-table-column>
                  <el-table-column prop="pay_status"  label="是否付款">
                        <template slot-scope="scope">
                            <el-tag type="success" v-if="scope.row.pay_status=== '1'">已付款</el-tag>
                            <el-tag type="danger" v-else>未付款</el-tag>
                        </template>
                  </el-table-column>
                  <el-table-column prop="is_send"  label="是否发货"></el-table-column>
                  <el-table-column prop="create_time"  label="下单时间">
                      <template slot-scope="scope">
                          {{scope.row.create_time | dateFormat}}
                      </template>
                  </el-table-column>
                  <el-table-column label="操作">
                      <template slot-scope="scope">
                          <el-button type="primary" icon="el-icon-edit" size="mini" @click="showBox"></el-button>
                          <el-button type="success" icon="el-icon-location" size="mini" @click="showProgress"></el-button>
                      </template>
                  </el-table-column>
             </el-table>

            <!--3. 分页 -->
         <el-pagination 
             @size-change="handleSizeChange"
             @current-change="handleCurrentChange"
             :current-page="queryInfo.pagenum"
             :page-sizes="[10, 20, 30]"
             :page-size="queryInfo.pagesize"
             layout="total, sizes, prev, pager, next, jumper"
             :total="total">
         </el-pagination>
       
        </el-card>
        <!-- 修改地址对话框 -->
            <el-dialog title=修改地址 :visible.sync="addressVisible"
             width="50%" @close="addressDialogClosed">
            <el-form :model="addressForm" :rules="addressFormRules" ref="addressFormRef" label-width="100px">
                <el-form-item label="省市区/县" prop="address1">
                        <el-cascader :options="cityData" v-model="addressForm.address1"></el-cascader>
                 </el-form-item>
                  <el-form-item label="详细地址" prop="address2">
                    <el-input v-model="addressForm.address2"></el-input>
                 </el-form-item>
            </el-form>
           <span slot="footer" class="dialog-footer">
    <el-button @click="addressVisible = false">取 消</el-button>
    <el-button type="primary" @click="addressVisible = false">确 定</el-button>
  </span>
            </el-dialog>

            <!--展示物流信息对话框 -->
            <el-dialog title="物流信息" :visible.sync="progressVisible" width="50%">
                <!-- 物流时间线 -->
                  <el-timeline>
                      <el-timeline-item
                     v-for="(activity, index) in progressInfo"
                      :key="index" :timestamp="activity.time">
                    {{activity.context}}
                    </el-timeline-item>
                 </el-timeline>
            </el-dialog>


    </div>
</template>

<script>
import LoginVue from '../Login.vue'
import cityData from './city_data2017_element.js'

export default {
  // 生命周期函数
    created(){
        this.getOrderList()
    },
    // 数据
    data(){
        return{
            // 请求数据参数
                queryInfo:{
                    query:'',
                    pagenum:1,
                    pagesize:10
                },
                // 数据条数
                total:1,
                // 订单列表
                orderlist:[],
                // 修改地址对话框
                addressVisible:false,
                addressForm:{
                    address1:[],
                    address2:''
                },
                addressFormRules:{
                    address1:[{required:true, message: '请选择省市区/县'}],
                     address2:[{required:true, message: '请填写详细地址'}]
                },
                //导入的地址
                cityData ,
                // 物流对话框
                progressVisible:false,
                progressInfo:[],
        }
    },
    // 方法/函数
    methods:{
        // 获取订单列表
       async getOrderList(){
        const {data:res}= await this.$http.get('orders',{params: this.queryInfo})
            if(res.meta.status !==200){
                return this.$message.error("获取订单列表失败！")
            }
            this.total = res.data.total;
            this.orderlist = res.data.goods; 
            console.log(res.data);
        },
        
     // 监听pagesize改变的事件
          handleSizeChange(newSize){
        // console.log(newSize);
        this.queryInfo.pagesize=newSize;
        this.getOrderList();
      },

         // 监听页面值改变的事件  
        handleCurrentChange(newPage){
        //  console.log(newPage);
        this.queryInfo.pagenum=newPage;
         this.getOrderList();
    },
    // 展示修改订单地址
    showBox(){
        this.addressVisible = true;
    },

    // 关闭对话框
    addressDialogClosed(){
        // 清空输入框
        this.$refs.addressFormRef.resetFields()
    },

    // 查看物流信息
   async showProgress(){
    // const { data:res } = await this.$http.get(`/kuaidi/`)
    //  if(res.meta.status !== 200){
    //      return this.$message.error("获取网络信息失败!")
    //  }
    //     this.progressInfo =res.data;
    //     console.log(res.data);
        this.progressVisible=true
},


   },

}
</script>

<style lang="less" scoped>

</style>