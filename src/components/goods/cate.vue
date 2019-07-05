<template>
  <div>
    <!-- 面包屑导航 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>商品列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片视图 -->
    <el-card>
      <el-row>
        <el-col>
          <el-button type="primary" @click="showAddCatDialog">添加分类</el-button>
        </el-col>
      </el-row>
      <!-- 树形表格 -->
      <tree-table
        :data="cateList"
        :columns="columns"
        :expand-type="false"
        :selectable="false"
        show-index
        index-text="#"
        border
      >
        <!-- 是否有效 -->
        <template slot="isok" slot-scope="scope">
          <i v-if="scope.row.cat_deleted==0" style="color:#67C23A" class="el-icon-success"></i>
          <i v-if="scope.row.cat_deleted==1" style="color:#F56C6C" class="el-icon-error"></i>
        </template>
        <!-- 排序 -->
        <template slot="order" slot-scope="scope">
          <el-tag v-if="scope.row.cat_level==0">一级</el-tag>
          <el-tag v-if="scope.row.cat_level==1" type="success">二级</el-tag>
          <el-tag v-if="scope.row.cat_level==2" type="warning">三级</el-tag>
        </template>
        <!-- 操作 -->
        <template slot="opt" slot-scope="scope">
          <el-button type="primary" icon="el-icon-edit" size="mini">编辑</el-button>
          <el-button type="danger" icon="el-icon-delete" size="mini">删除</el-button>
        </template>
      </tree-table>
      <!-- 分页 -->
      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="queryInfo.pagenum"
        :page-sizes="[1,2,5,10]"
        :page-size="queryInfo.pagesize"
        layout="total, sizes, prev, pager, next, jumper"
        :total="total"
      ></el-pagination>
    </el-card>

    <!-- 添加分类的对话框 -->
    <el-dialog
      title="添加分类"
      :visible.sync="addCateDialogVisible"
      width="30%"
      @close="resetForm('addCateFormRef')"
    >
      <!-- 表单 -->
      <el-form :model="addCateForm" :rules="formRules" ref="addCateFormRef" label-width="80px">
        <el-form-item label="分类名称" prop="cat_name">
          <el-input v-model.trim="addCateForm.cat_name"></el-input>
        </el-form-item>
        <el-form-item label="分类层级">
          <!-- 层级选择器 -->
          <el-cascader
            v-model="selectedParentCateId"
            :options="parentCateList"
            :props="cascaderConfig"
            clearable
          ></el-cascader>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="addCateDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addCate">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  created() {
    this.getCateList()
  },
  data() {
    return {
      // 分类树查询参数
      queryInfo: {
        type: 3,
        pagenum: 1,
        pagesize: 5
      },
      // 分类数据
      cateList: [],
      // 数据总条数
      total: null,
      // 树形表格每列配置
      columns: [
        {
          title: '分类名称',
          key: 'cat_name'
        },
        {
          title: '是否有效',
          // 表示，将当前列定义为模板列
          type: 'template',
          // 表示当前这一列使用模板名称
          template: 'isok'
        },
        {
          title: '排序',
          // 表示，将当前列定义为模板列
          type: 'template',
          // 表示当前这一列使用模板名称
          template: 'order'
        },
        {
          title: '操作',
          // 表示，将当前列定义为模板列
          type: 'template',
          // 表示当前这一列使用模板名称
          template: 'opt',
          width: '200px'
        }
      ],
      // 添加分类的对话框是否可见的标识
      addCateDialogVisible: false,
      // 添加分类的参数
      addCateForm: {
        cat_pid: '',
        cat_name: '',
        cat_level: ''
      },
      // 表单验证规则
      formRules: {
        cat_name: [
          { required: true, message: '请输入分类名称', trigger: 'blur' }
        ]
      },
      // 层级选择器配置
      cascaderConfig: {
        expandTrigger: 'hover',
        value: 'cat_id',
        label: 'cat_name',
        checkStrictly: true
      },
      // 父级分类数据
      parentCateList: [],
      // 已选择的父级分类 id
      selectedParentCateId: []
    }
  },
  methods: {
    // 获取分类信息
    async getCateList() {
      const { data: result } = await this.$http.get('categories', {
        params: this.queryInfo
      })
      if (result.meta.status !== 200) {
        return this.$message.error(result.meta.msg)
      }
      this.cateList = result.data.result
      this.total = result.data.total
    },
    // 分页大小
    handleSizeChange(pagesize) {
      this.queryInfo.pagesize = pagesize
      this.getCateList()
    },
    // 页码
    handleCurrentChange(pagenum) {
      this.queryInfo.pagenum = pagenum
      this.getCateList()
    },
    // 展示添加分类的对话框
    showAddCatDialog() {
      this.getParentCateList()
      this.addCateDialogVisible = true
    },
    // 重置表单
    resetForm(ref) {
      this.$refs[ref].resetFields()
      if (ref === 'addCateFormRef') {
        this.selectedParentCateId = []
        this.addCateForm = {
          cat_pid: '',
          cat_name: '',
          cat_level: ''
        }
      }
    },
    // 获取父级分类信息
    async getParentCateList() {
      const { data: result } = await this.$http.get('categories', {
        params: { type: 2 }
      })
      if (result.meta.status !== 200) {
        return this.$message.error(result.meta.msg)
      }
      this.parentCateList = result.data
    },
    // 添加分类
    async addCate() {
      // 获取层级选择器选择结果数组的长度
      const length = this.selectedParentCateId.length
      // 长度大于0，说明选择了，父级 id 为数组最后一项
      if (length > 0) {
        this.addCateForm.cat_pid = this.selectedParentCateId[length - 1]
      } else {
        // 没选择，则作为一级分类，父级 id 为0
        this.addCateForm.cat_pid = 0
      }
      // 分类层级刚好等于长度
      this.addCateForm.cat_level = length
      const { data: result } = await this.$http.post(
        'categories',
        this.addCateForm
      )
      if (result.meta.status !== 201) {
        return this.$message.error(result.meta.msg)
      }
      this.getCateList()
      this.$message.success(result.meta.msg)
      this.addCateDialogVisible = false
    }
  }
}
</script>

<style lang="less" scoped>
.zk-table {
  margin-top: 15px;
  margin-bottom: 15px;
}
</style>