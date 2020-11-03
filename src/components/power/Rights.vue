<template>
  <div>
    <!-- 面包屑导航区域 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>权限管理</el-breadcrumb-item>
      <el-breadcrumb-item>权限列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片视图区域 -->
    <el-card>
      <!-- 权限列表区域-->
      <el-table :data="rightslist" border stripe>
        <el-table-column label="索引" type="index"></el-table-column>
        <el-table-column label="权限名称" prop="authName"></el-table-column>
        <el-table-column label="路径" prop="path"></el-table-column>
        <el-table-column label="权限等级" prop="level">
          <template slot-scope="scope">
            <el-tag v-if="scope.row.level === '0'">一级 </el-tag>
            <el-tag v-else-if="scope.row.level === '1'" type="success"
              >二级
            </el-tag>
            <el-tag v-else-if="scope.row.level === '2'" type="warning"
              >三级
            </el-tag>
          </template>
        </el-table-column>
      </el-table>
    </el-card>
  </div>
</template>
<script>
export default {
  data() {
    return {
      rightslist: []
    }
  },
  methods: {
    async getRightsList(type) {
      const { data: res } = await this.$http.get(`rights/${type}`)
      if (res.meta.status !== 200) {
        return this.$message.error(res.meta.msg)
      }
      this.rightslist = res.data
    }
  },
  created() {
    this.getRightsList('list')
  }
}
</script>
<style lang="less" scoped></style>
