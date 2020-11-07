<template>
  <div>
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>商品列表</el-breadcrumb-item>
    </el-breadcrumb>
    <el-card>
      <el-row>
        <el-col :span="4">
          <el-button type="primary" @click="showAddCateDialog"
            >添加分类</el-button
          >
        </el-col>
      </el-row>
      <tree-table
        :data="cate_list"
        :columns="columnsConfig"
        :show-row-hover="false"
        :selection-type="false"
        :expand-type="false"
        border
        show-index
        index-text="索引"
      >
        <template slot="isok" slot-scope="scope">
          <i
            class="el-icon-check"
            v-if="scope.cat_deleted"
            style="color:green"
          ></i>
          <i class="el-icon-close" v-else style="color:red"></i>
        </template>
        <template slot="order" slot-scope="scope">
          <el-tag type="" v-if="scope.row.cat_level === 0">一级</el-tag>
          <el-tag type="success" v-else-if="scope.row.cat_level === 1"
            >二级</el-tag
          >
          <el-tag type="warning" v-else>三级</el-tag>
        </template>
        <template slot="opt" slot-scope="">
          <el-button type="primary" size="mini" icon="el-icon-edit"
            >编辑</el-button
          >
          <el-button type="danger" size="mini" icon="el-icon-delete"
            >删除</el-button
          >
        </template>
      </tree-table>
      <el-pagination
        :total="total"
        :current-page="cateParams.pagenum"
        :page-sizes="[3, 5, 10, 15]"
        :page-size="cateParams.pagesize"
        @current-change="handleCurrentChange"
        @size-change="handleSizeChange"
        layout="total,sizes,prev,pager,next,jumper"
      ></el-pagination>
    </el-card>
    <!-- 添加用户的对话框 -->
    <el-dialog
      title="添加分类"
      :visible.sync="addCateDialogVisible"
      width="50%"
      @close="addCateDialogClosed"
    >
      <!-- 内容主题区域 -->
      <el-form
        :model="addCateForm"
        :rules="addCateFormRules"
        ref="addCateFormRef"
        label-width="100px"
      >
        <el-form-item label="分类名称:" prop="cat_name">
          <el-input v-model="addCateForm.cat_name"></el-input>
        </el-form-item>
        <el-form-item label="父级分类:" prop="catePid">
          <el-cascader
            expand-trigger="hover"
            :options="parentCateList"
            :props="cateCascaderProps"
            v-model="cateSelectedKeys"
            @change="parentCateChanged"
            clearable
            change-on-select
          ></el-cascader>
        </el-form-item>
      </el-form>
      <!-- 底部区域 -->
      <span slot="footer" class="dialog-footer">
        <el-button type="primary" @click="addCate">确定</el-button>
        <el-button @click="addCateDialogVisible = false">取消</el-button>
      </span>
    </el-dialog>
  </div>
</template>
<script>
export default {
  data() {
    return {
      cateSelectedKeys: [],
      // 级联选择器配置
      cateCascaderProps: {
        // 级联选择器选中的值
        value: 'cat_id',
        // 选择器显示的值
        label: 'cat_name',
        // 父子嵌套级联关系
        children: 'children'
      },
      // 父级分类的列表
      parentCateList: [],
      addCateDialogVisible: false,
      addCateFormRules: {
        cat_name: [
          { required: true, message: '请输入分类名称', trigger: 'blur' }
        ]
      },
      addCateForm: {
        cat_pid: 0,
        cat_name: '',
        cat_level: 0
      },
      total: 0,
      cate_list: [],
      cateParams: {
        type: 3,
        pagenum: 1,
        pagesize: 5
      },
      columnsConfig: [
        {
          label: '分类名称',
          prop: 'cat_name'
        },
        {
          label: '是否有效',
          type: 'template',
          template: 'isok'
        },
        {
          label: '排序',
          type: 'template',
          template: 'order'
        },
        {
          label: '操作',
          type: 'template',
          template: 'opt'
        }
      ]
    }
  },
  methods: {
    showAddCateDialog() {
      this.addCateDialogVisible = true
      this.getParentCateList()
    },
    async getParentCateList() {
      const { data: res } = await this.$http.get('categories', {
        params: { type: 2 }
      })
      if (res.meta.status !== 200) return this.$message.error(res.meta.msg)
      this.parentCateList = res.data
    },
    parentCateChanged() {
      console.log(this.cateSelectedKeys)
      if (this.cateSelectedKeys.length > 0) {
        this.addCateForm.cat_pid = this.cateSelectedKeys[
          this.cateSelectedKeys.length - 1
        ]
        this.addCateForm.cat_level = this.cateSelectedKeys.length
      } else {
        this.addCateForm.cat_pid = 0
        this.addCateForm.cat_level = 0
      }
    },
    addCateDialogClosed() {
      this.$refs.addCateFormRef.resetFields()
      this.cateSelectedKeys = []
      this.addCateForm.cat_pid = 0
      this.addCateForm.cat_level = 0
    },
    async addCate() {
      this.$refs.addCateFormRef.validate(async valid => {
        if (!valid) return
        const { data: res } = await this.$http.post(
          'categories',
          this.addCateForm
        )
        if (res.meta.status !== 201) return this.$message.error(res.meta.msg)
        this.$message.success(res.meta.msg)
        this.getCateList()
        this.addCateDialogVisible = false
      })
    },
    handleCurrentChange(newPage) {
      this.cateParams.pagenum = newPage
      this.getCateList()
    },
    handleSizeChange(newSize) {
      console.log(newSize)
      this.cateParams.pagesize = newSize
      this.getCateList()
    },
    async getCateList() {
      const { data: res } = await this.$http.get('categories', {
        params: this.cateParams
      })
      if (res.meta.status !== 200) return this.$message.error(res.meta.msg)
      this.cate_list = res.data.result
      this.total = res.data.total
    }
  },
  created() {
    this.getCateList()
  }
}
</script>
<style lang="less" scoped></style>
