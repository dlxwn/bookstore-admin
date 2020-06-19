<template>
  <div>
    <!-- 面包屑导航区 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>权限管理</el-breadcrumb-item>
      <el-breadcrumb-item>权限列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片视图 -->
    <el-card>
      <el-table :data="rightsList" border stripe>
        <el-table-column type="index" label="#" align="center"></el-table-column>
        <el-table-column label="角色名称" prop="name" align="center"></el-table-column>
        <el-table-column label="职 务" prop="position" align="center"></el-table-column>
        <el-table-column label="权 限" prop="level" align="center">
          <template>
            <el-switch
            v-model="value1"
            @click="change">
            </el-switch>
          </template>
        </el-table-column>
      </el-table>
    </el-card>
  </div>
</template>

<script>
export default {
  data () {
    return {
      queryInfo: {
        query: '',
        current: 1,
        size: 10
      },
      // 权限列表
      rightsList: [],
      userInfo: [],
      value1: true
    }
  },
  created () {
    this.getRightsList()
    this.getUserInfo()
  },
  methods: {
    async getRightsList () {
      const { data: res } = await this.$http.get('/employee/findAll/' + this.queryInfo.current + '/' + this.queryInfo.size)
      console.log(res)
      if (res.pages === 0) {
        return this.$message.error('获取权限列表失败！')
      }
      this.$message.success('获取权限列表成功')
      this.rightsList = res.records
    },
    getUserInfo () {
    },
    changeSwitch (data) {
      console.log(data)
    }
  }
}
</script>

<style lang='less' scoped>
</style>
