<template>
  <div>
    <!-- 面包屑导航区 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>订单管理</el-breadcrumb-item>
      <el-breadcrumb-item>订单列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片视图 -->
    <el-card>
      <el-row>
        <el-col :span="6">
          <el-input v-model="input" placeholder="请输入内容" clearable @clear="getOrderList">
            <el-button slot="append" icon="el-icon-search" @click="searchOrderByFuzzy"></el-button>
          </el-input>
        </el-col>
      </el-row>

      <!-- 订单列表 -->
      <el-table :data="orderList" border stripe style="width: 100%;">
        <el-table-column type="index" label="#"></el-table-column>
        <el-table-column label="订单编号" prop="orderId"></el-table-column>
        <el-table-column label="消费总额 (单位:元)" prop="amount"></el-table-column>
        <el-table-column label="是否审核">
          <template slot-scope="scope">
            <el-tag type="danger" size="medium" v-if="scope.row.status === 0">未审核</el-tag>
            <el-tag type="success" size="medium" v-else>已审核</el-tag>
          </template>
        </el-table-column>
        <el-table-column label="收货人姓名" prop="orderName"></el-table-column>
        <el-table-column label="收货人电话" prop="orderTel"></el-table-column>
        <el-table-column label="收货人地址" prop="orderAddress"></el-table-column>
        <el-table-column label="下单时间" prop="orderDate"></el-table-column>
        <el-table-column label="操作">
          <template slot-scope="scope">
            <el-tooltip class="item" effect="light" open-delay="500" content="审核订单" placement="bottom" :enterable="false">
            <el-button type="primary" size="mini" icon="el-icon-edit"
            circle
            @click="checkOrderById(scope.row.orderId)"></el-button>
            </el-tooltip>
            <el-tooltip class="item" effect="light" open-delay="500" content="查看明细" placement="bottom" :enterable="false">
            <el-button
              type="success"
              size="mini"
              icon="el-icon-location"
              circle
              @click="showOrderDetailById(scope.row.orderId)"
            ></el-button>
            </el-tooltip>
            <el-tooltip class="item" effect="light" open-delay="500" content="删除订单" placement="bottom" :enterable="false">
            <el-button
              type="danger"
              icon="el-icon-delete"
              size="mini"
              circle
              @click="removeOrderById(scope.row.orderId)"
            ></el-button>
            </el-tooltip>
          </template>
        </el-table-column>
      </el-table>
      <!-- 分页区域 -->
      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="queryInfo.pagenum"
        :page-sizes="[5, 10, 15, 20]"
        :page-size="queryInfo.pagesize"
        layout="total, sizes, prev, pager, next, jumper"
        :total="total"
      ></el-pagination>
    </el-card>

    <!-- 展示物订单详情对话框 -->
    <el-dialog title="查看订单详情" :visible.sync="progressDialogVisible" width="50%">
    <!-- 订单详情 -->
    <template>
    <el-table
        :data="tableData"
        height="250"
        border
        style="width: 100%">
        <el-table-column
          prop="bookName"
          label="书名"
          width="180">
        </el-table-column>
        <el-table-column
          prop="price"
          label="单价 (单位:元)"
          width="180">
        </el-table-column>
        <el-table-column
          prop="num"
          label="购买数量 (单位:本)">
        </el-table-column>
      </el-table>
      </template>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data () {
    return {
      // 搜索框的内容
      input: '',
      // 订单列表查询参数
      queryInfo: {
        query: '',
        pagenum: 1,
        pagesize: 5
      },
      total: 0,
      tableData: [],
      // 订单列表
      orderList: [],
      // 订单详情对话框
      progressDialogVisible: false
    }
  },
  created () {
    this.getOrderList()
  },
  methods: {
    // 获取订单列表
    async getOrderList () {
      if (this.input !== '') {
        this.searchOrderByFuzzy()
        return
      }
      const { data: res } = await this.$http.get('/orderlist/selectAll/' + this.queryInfo.pagenum + '/' + this.queryInfo.pagesize)
      console.log(res)
      if (res.pages === 0) {
        return this.$message.error('获取订单列表失败！')
      }
      this.total = res.total
      // console.log(this.total)
      this.orderList = res.records
    },
    // 实现删除订单
    async removeOrderById (id) {
      const confirmResult = await this.$confirm(
        '此操作将永久删除该订单, 是否继续?',
        '提示',
        {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }
      ).catch(err => err)
      // 点击确定 返回值为：confirm
      // 点击取消 返回值为： cancel
      if (confirmResult !== 'confirm') {
        return this.$message.info('已取消删除')
      }
      const { data: res } = await this.$http.delete('orderlist/delete/' + id)
      if (res.code !== 1) return this.$message.error('删除订单失败！')
      this.$message.success('删除订单成功！')
      this.getOrderList()
      // console.log(res)
    },
    // 实现订单审核功能
    async checkOrderById (id) {
      const confirmResult = await this.$confirm(
        '是否完成审核？',
        '提示',
        {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }
      ).catch(err => err)
      // 点击确定 返回值为：confirm
      // 点击取消 返回值为： cancel
      if (confirmResult !== 'confirm') {
        return this.$message.info('已取消审核')
      }
      const { data: res } = await this.$http.put('orderlist/update/' + id)
      if (res.code !== 1) return this.$message.error('审核订单失败！')
      this.$message.success('审核订单成功！')
      this.getOrderList()
      // console.log(res)
    },
    // 实现搜索功能
    async searchOrderByFuzzy () {
      if (this.input === '') {
        this.getOrderList()
        return
      }
      const { data: res } = await this.$http.get('/orderlist/selectList/' + this.input + '/' + this.queryInfo.pagenum + '/' + this.queryInfo.pagesize)
      console.log(res)
      this.total = res.total
      this.orderList = res.records
    },
    // 分页
    handleSizeChange (newSize) {
      this.queryInfo.pagesize = newSize
      this.getOrderList()
    },
    handleCurrentChange (newSize) {
      this.queryInfo.pagenum = newSize
      this.getOrderList()
    },
    showEditDialog () {
      this.addressDialogVisible = true
    },
    addressDialogClosed () {
      this.$refs.addressFormRef.resetFields()
    },
    // 实现查看订单明细功能
    async showOrderDetailById (id) {
      const { data: res } = await this.$http.get('/orderdetail/findByOrderId/' + id)
      console.log(res)
      this.tableData = res
      this.progressDialogVisible = true
    }
  }
}
</script>

<style lang="less" scoped>
.el-cascader {
  width: 100%;
}
</style>
