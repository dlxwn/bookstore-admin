<template>
  <div>
    <!-- 面包屑导航区 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>图书管理</el-breadcrumb-item>
      <el-breadcrumb-item>图书列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片视图 -->
    <el-card>
      <el-row :gutter="20">
        <el-col :span="6">
          <el-input placeholder="请输入内容" @keyup.enter="adddbbssc" v-model="input" clearable @clear="getBooksList" >
            <el-button slot="append" icon="el-icon-search" @click="getBooksList" ></el-button>
          </el-input>
        </el-col>
        <el-col :span="0.5">
          <el-button type="primary" @click="addBookInfo" class="el-icon-plus el-icon-right" > 添加书籍</el-button>
        </el-col>
        <el-col :span="0.5">
          <el-button type="primary" @click="exportBook">导出图书 <i class="el-icon-download el-icon-right"></i></el-button>
        </el-col>
        <!-- <el-col :span="0.5">
          <el-upload
            action=""
            :on-change="handleChange"
            :on-exceed="handleExceed"
            :on-remove="handleRemove"
            :file-list="fileListUpload"
            :limit="limitUpload"
            accept="application/vnd.openxmlformats-officedocument.spreadsheetml.sheet,application/vnd.ms-excel"
            :auto-upload="false">
            <el-button  type="primary" >点击上传 <i class="el-icon-download el-icon-right"></i></el-button>
          </el-upload>
        </el-col> -->
      </el-row>
      <!-- 表格数据 -->
      <el-table :data="booksList" border stripe>
        <el-table-column label="图书编号" prop="isbn" align="center"></el-table-column>
        <el-table-column label="书 名" prop="bookName" align="center"></el-table-column>
        <el-table-column label="作 者" prop="author" align="center"></el-table-column>
        <el-table-column label="价 格" prop="price" align="center" width="50px"></el-table-column>
        <el-table-column label="类 型" prop="bookType" align="center"  width="50px"></el-table-column>
        <el-table-column label="库 存" prop="repertory" align="center" width="50px"></el-table-column>
        <el-table-column label="出版社" prop="press" align="center" ></el-table-column>
        <el-table-column label="收藏量" prop="cllectNum" align="center" width="70px"></el-table-column>
        <el-table-column label="月销量" prop="saleNum" align="center" width="70px"></el-table-column>
        <el-table-column label="出版日期" prop="publicDate" align="center" width="100px"></el-table-column>
        <el-table-column label="操作" width="160px" align="center">
          <template slot-scope="scope">
            <el-button type="primary" icon="el-icon-edit" size="mini" @click="handleUpdate(scope.row.isbn)"  circle></el-button>
            <el-button type="success" icon="el-icon-search" size="mini"  circle @click="details(scope.row.isbn)"></el-button>
            <el-button
              type="danger"
              icon="el-icon-delete"
              size="mini"
              @click="removeById(scope.row.isbn)"
              circle
            ></el-button>
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
        background
      ></el-pagination>
    </el-card>
    <el-dialog
      title="添加页面"
      :visible.sync="addDialogVisible"
      width="30%"
      :before-close="handleClose">
      <el-form ref="addFormRef" :model="addForm" :rules="addFormRules" label-width="80px">
        <el-form-item label="书籍编号" prop="isbn" style="margin-top: 20px;">
          <el-input v-model="addForm.isbn"></el-input>
        </el-form-item>
        <el-form-item label="书 名" prop="bookName">
          <el-input v-model="addForm.bookName"></el-input>
        </el-form-item>
       <el-form-item label="图片地址" prop="bookPicture">
          <el-input v-model="addForm.bookPicture"></el-input>
        </el-form-item>
        <el-form-item label="作 者" prop="author">
          <el-input v-model="addForm.author"></el-input>
        </el-form-item>
        <el-form-item label="价 格" prop="price">
          <el-input v-model="addForm.price"></el-input>
        </el-form-item>
        <el-form-item label="类 型" prop="bookType">
          <el-input v-model="addForm.bookType"></el-input>
        </el-form-item>
         <el-form-item label="库 存" prop="repertory">
          <el-input-number v-model="addForm.repertory" :min="0" label="库 存"></el-input-number>
        </el-form-item>
        <el-form-item label="出版社" prop="press">
          <el-input v-model="addForm.press"></el-input>
        </el-form-item>
        <el-form-item label="收藏量" prop="cllectNum">
          <el-input-number v-model="addForm.cllectNum"  :min="0" label="收藏量"></el-input-number>
        </el-form-item>
        <el-form-item label="月销量" prop="saleNum">
          <el-input-number v-model="addForm.saleNum"  :min="0" label="月销量"></el-input-number>
        </el-form-item>
        <el-form-item label="简 介">
          <el-input type="textarea" v-model="addForm.description"></el-input>
        </el-form-item>
        <el-form-item>
          <el-button type="primary" @click="addBook">添 加</el-button>
          <el-button @click="closeAdd">取 消</el-button>
        </el-form-item>
      </el-form>
    </el-dialog>
    <!-- 修改 -->
    <el-dialog
      title="修改页面"
      :visible.sync="updateDialogVisible"
      width="30%"
      :before-close="handleClose">
      <el-form ref="updateFormRef" :model="updateForm" :rules="updateFormRules" label-width="80px">
        <el-form-item label="书籍编号" prop="isbn" style="margin-top: 20px;">
          <el-input v-model="updateForm.isbn"></el-input>
        </el-form-item>
        <el-form-item label="书 名" prop="bookName">
          <el-input v-model="updateForm.bookName"></el-input>
        </el-form-item>
       <el-form-item label="图片地址" prop="bookPicture">
          <el-input v-model="updateForm.bookPicture"></el-input>
        </el-form-item>
        <el-form-item label="作 者" prop="author">
          <el-input v-model="updateForm.author"></el-input>
        </el-form-item>
        <el-form-item label="价 格" prop="price">
          <el-input v-model="updateForm.price"></el-input>
        </el-form-item>
        <el-form-item label="类 型" prop="bookType">
          <el-input v-model="updateForm.bookType"></el-input>
        </el-form-item>
         <el-form-item label="库 存" prop="repertory">
          <el-input-number v-model="updateForm.repertory" :min="0" label="库 存"></el-input-number>
        </el-form-item>
        <el-form-item label="出版社" prop="press">
          <el-input v-model="updateForm.press"></el-input>
        </el-form-item>
        <el-form-item label="收藏量" prop="cllectNum">
          <el-input-number v-model="updateForm.cllectNum"  :min="0" label="收藏量"></el-input-number>
        </el-form-item>
        <el-form-item label="月销量" prop="saleNum">
          <el-input-number v-model="updateForm.saleNum"  :min="0" label="月销量"></el-input-number>
        </el-form-item>
        <el-form-item label="简 介">
          <el-input type="textarea" v-model="updateForm.description"></el-input>
        </el-form-item>
        <el-form-item>
          <el-button type="primary" @click="update">修 改</el-button>
          <el-button @click="closeUpdate" >取消</el-button>
        </el-form-item>
      </el-form>
  </el-dialog>
  <!-- 详情 -->
  <el-dialog
      title="详情页面"
      :visible.sync="detailDialogVisible"
      width="30%"
      :before-close="handleClose">
      <el-form ref="detailFormRef" :model="detailForm" label-width="80px">
        <el-form-item label="书籍编号" prop="isbn" style="margin-top: 20px;">
          <el-input v-model="detailForm.isbn"></el-input>
        </el-form-item>
        <el-form-item label="书 名" prop="bookName">
          <el-input v-model="detailForm.bookName"></el-input>
        </el-form-item>
       <el-form-item  prop="bookPicture">
          <img :src="detailForm.bookPicture" style="width:150px;height:200px;padding-left:25%;">
        </el-form-item>
        <el-form-item label="简 介">
          <el-input type="textarea" v-model="detailForm.description"></el-input>
        </el-form-item>
        <el-form-item>
          <el-button type="primary" @click="back">返 回</el-button>
        </el-form-item>
      </el-form>
  </el-dialog>
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
      addForm: {
        isbn: '',
        bookName: '',
        bookPicture: '',
        author: '',
        price: 0,
        description: '',
        bookType: 1,
        repertory: 0,
        press: '',
        cllectNum: 0,
        saleNum: 0,
        publicDate: ''
      },
      updateForm: {},
      detailForm: {},
      updateDialogVisible: false,
      detailDialogVisible: false,
      addDialogVisible: false,
      // 商品列表
      booksList: [],
      // 商品总数
      total: 0,
      input: '',
      exportBookUrl: 'http://localhost:8088/bookstore/book/exportBooks',
      fileTemp: '',
      updateFormRules: {
        isbn: [
          { required: true, message: '请输入书籍编号', trigger: 'blur' }
        ],
        bookName: [
          { required: true, message: '请输入书籍名称', trigger: 'blur' }
        ],
        author: [
          { required: true, message: '请输入书籍作者', trigger: 'blur' }
        ],
        price: [
          { required: true, message: '请选择书籍价格', trigger: 'blur' }
        ],
        description: [
          { required: true, message: '请选择书籍简介', trigger: 'blur' }
        ],
        typeId: [
          { required: true, message: '请选择书籍类型', trigger: 'blur' }
        ],
        repertory: [
          { required: true, message: '请选择书籍库存', trigger: 'blur' }
        ],
        press: [
          { required: true, message: '请选择书籍出版社', trigger: 'blur' }
        ],
        cllectNum: [
          { required: true, message: '请选择书籍收藏量', trigger: 'blur' }
        ],
        saleNum: [
          { required: true, message: '请选择书籍月销量', trigger: 'blur' }
        ],
        publicDate: [
          { required: true, message: '请选择书籍出版日期', trigger: 'blur' }
        ]
      },
      addFormRules: {
        isbn: [
          { required: true, message: '请输入书籍编号', trigger: 'blur' }
        ],
        bookName: [
          { required: true, message: '请输入书籍名称', trigger: 'blur' }
        ],
        author: [
          { required: true, message: '请输入书籍作者', trigger: 'blur' }
        ],
        price: [
          { required: true, message: '请选择书籍价格', trigger: 'blur' }
        ],
        description: [
          { required: true, message: '请选择书籍简介', trigger: 'blur' }
        ],
        typeId: [
          { required: true, message: '请选择书籍类型', trigger: 'blur' }
        ],
        repertory: [
          { required: true, message: '请选择书籍库存', trigger: 'blur' }
        ],
        press: [
          { required: true, message: '请选择书籍出版社', trigger: 'blur' }
        ],
        cllectNum: [
          { required: true, message: '请选择书籍收藏量', trigger: 'blur' }
        ],
        saleNum: [
          { required: true, message: '请选择书籍月销量', trigger: 'blur' }
        ],
        publicDate: [
          { required: true, message: '请选择书籍出版日期', trigger: 'blur' }
        ]
      }
    }
  },
  created () {
    this.getBooksList()
  },
  methods: {
    // 根据分页获取对应的商品列表
    async getBooksList () {
      if (this.input !== '') {
        this.getFuzzyPages()
        return
      }
      const { data: res } = await this.$http.get('/book/listPages/' + this.queryInfo.current + '/' + this.queryInfo.size, {
        params: this.queryInfo
      })
      if (res.pages === 0) {
        return this.$message.error('获取商品列表失败！')
      }
      this.booksList = res.records
      this.total = res.total
      console.log(res)
    },
    handleSizeChange (newSize) {
      this.queryInfo.size = newSize
      console.log(this.queryInfo.size)
      this.getBooksList()
    },
    handleCurrentChange (newSize) {
      this.queryInfo.current = newSize
      console.log(this.queryInfo.current)
      this.getBooksList()
    },
    // 通过Id删除商品
    async removeById (isbn) {
      const confirmResult = await this.$confirm(
        '此操作将永久删除该商品, 是否继续?',
        '提示',
        {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }
      ).catch(err => err)
      if (confirmResult !== 'confirm') {
        return this.$message.info('已取消删除！')
      }
      const { data: res } = await this.$http.delete('/book/remove/' + isbn)
      console.log(res)
      if (res.code !== 200) {
        return this.$message.error('删除商品失败！')
      }
      this.$message.success('删除商品成功！')
      this.getBooksList()
    },
    // 实现模糊搜索分页功能
    async getFuzzyPages () {
      if (this.input === '') {
        this.getBooksList()
        return
      }
      const { data: res } = await this.$http.get('/book/listFuzzy/' + this.input + '/' + this.queryInfo.current + '/' + this.queryInfo.size)
      this.total = res.total
      this.booksList = res.records
      console.log('哈哈哈哈哈')
      console.log(res)
      console.log(this.booksList)
    },
    async handleUpdate (isbn) {
      this.updateDialogVisible = true
      const { data: res } = await this.$http.get('/book/get/' + isbn)
      this.updateForm = res.data
    },
    async details (isbn) {
      this.detailDialogVisible = true
      const { data: res } = await this.$http.get('/book/get/' + isbn)
      this.detailForm = res.data
    },
    addBookInfo () {
      this.addDialogVisible = true
    },
    addBook () {
      // 添加书籍
      this.$refs.addFormRef.validate(async valid => {
        if (!valid) return
        const { data: res } = await this.$http.post('/book/save/', this.addForm)
        console.log('添加的操作：', res)
        if (res.code !== 200) {
          return this.$message.error('添加商品失败！')
        }
        this.$message.success('添加商品成功！')
        this.closeAdd()
        this.getBooksList()
      })
    },
    closeAdd () {
      this.$refs.addFormRef.resetFields()
      this.addDialogVisible = false
    },
    back () {
      this.$refs.detailFormRef.resetFields()
      this.detailDialogVisible = false
    },
    handleClose (done) {
      this.$confirm('确认关闭？')
        .then(_ => {
          done()
        })
        .catch(_ => {
        })
    },
    async exportBook () {
      window.location.href = this.exportBookUrl
    },
    update () {
      this.$refs.updateFormRef.validate(async valid => {
        if (!valid) return
        const { data: res } = await this.$http.post('/book/update/', this.updateForm)
        console.log('更新的操作：', res)
        if (res.code !== 200) {
          return this.$message.error('修改失败')
        }
        this.$message.success('更新成功')
        this.closeUpdate()
        this.getBooksList()
      })
    },
    closeUpdate () {
      this.$refs.updateFormRef.resetFields()
      this.updateDialogVisible = false
    }
  }
}
</script>

<style lang="less" scoped>
</style>
