<template>
  <div>
    <el-button type="primary">添加角色</el-button>
    <el-table :data="roleslist" border stripe>
      <el-table-column type="expand">
        <template slot-scope="scope">
          <el-row
            :class="['bdbottom', k1 === 0 ? 'bdtop' : '', 'vcenter']"
            v-for="(v1, k1) in scope.row.children"
            :key="v1.id"
          >
            <el-col :span="5">
              <el-tag closable @close="deleteRight(scope.row, v1.id)">
                {{ v1.authName }}</el-tag
              >
              <i class="el-icon-caret-right"></i>
            </el-col>
            <el-col :span="19">
              <el-row
                :class="[k2 === 0 ? '' : 'bdtop', 'vcenter']"
                v-for="(v2, k2) in v1.children"
                :key="v2.id"
              >
                <el-col :span="6">
                  <el-tag
                    type="success"
                    closable
                    @close="deleteRight(scope.row, v2.id)"
                    >{{ v2.authName }}</el-tag
                  >
                  <i class="el-icon-caret-right"></i>
                </el-col>
                <el-col :span="18">
                  <el-row v-for="v3 in v2.children" :key="v3.id">
                    <el-tag
                      type="warning"
                      closable
                      @close="deleteRight(scope.row, v3.id)"
                      >{{ v3.authName }}</el-tag
                    >
                  </el-row>
                </el-col>
              </el-row>
            </el-col>
          </el-row>
        </template>
      </el-table-column>
      <el-table-column label="索引" type="index"> </el-table-column>
      <el-table-column label="角色名称" prop="roleName"></el-table-column>
      <el-table-column label="角色描述" prop="roleDesc"></el-table-column>
      <el-table-column label="操作" width="300px">
        <template slot-scope="scope">
          <el-button type="primary" icon="el-icon-edit" size="mini"
            >编辑</el-button
          >
          <el-button type="danger" icon="el-icon-delete" size="mini"
            >删除</el-button
          >
          <el-button
            type="warning"
            icon="el-icon-setting"
            size="mini"
            @click="showSetRightsDialog(scope.row)"
            >分配权限</el-button
          >
        </template>
      </el-table-column>
    </el-table>
    <el-dialog
      title="分配权限"
      :visible.sync="setRightsDialogVisible"
      width="50%"
      @close="clearDefaultCheckedRights"
    >
      <el-tree
        :data="rightstree"
        :props="defaultProps"
        show-checkbox
        node-key="id"
        default-expand-all
        :default-checked-keys="defaultCheckedRights"
        ref="treeRef"
      ></el-tree>
      <span slot="footer" class="dialog-footer">
        <el-button @click="setRightsDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="allocRights()">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>
<script>
export default {
  data() {
    return {
      roleslist: [],
      rightstree: [],
      setRightsDialogVisible: false,
      defaultProps: {
        children: 'children',
        label: 'authName'
      },
      defaultCheckedRights: [],
      roleId: ''
    }
  },
  methods: {
    async allocRights() {
      let rids = [
        ...this.$refs.treeRef.getCheckedKeys(),
        ...this.$refs.treeRef.getHalfCheckedKeys()
      ]
      rids = rids.join(',')
      const { data: res } = await this.$http.post(
        `roles/${this.roleId}/rights`,
        { rids }
      )
      if (res.meta.status !== 200) {
        this.$message.error(res.meta.msg)
      }
      this.$message.success(res.meta.msg)
      this.getRolesList()
      this.setRightsDialogVisible = false
    },
    showSetRightsDialog(role) {
      this.roleId = role.id
      this.setRightsDialogVisible = true
      this.getRightsTree()
      this.getLeafRightsIdList(role, this.defaultCheckedRights)
    },
    getLeafRightsIdList(node, arr) {
      if (!node.children) return arr.push(node.id)
      node.children.forEach(v => {
        this.getLeafRightsIdList(v, arr)
      })
    },
    clearDefaultCheckedRights() {
      this.defaultCheckedRights = []
    },
    async getRightsTree() {
      const { data: res } = await this.$http.get('rights/tree')
      if (res.meta.status !== 200) {
        return this.$message.error(res.meta.msg)
      }
      this.rightstree = res.data
    },
    async getRolesList() {
      const { data: res } = await this.$http.get('roles')
      if (res.meta.status !== 200) {
        return this.$message.error(res.meta.msg)
      }
      this.roleslist = res.data
    },
    async deleteRight(role, rightId) {
      const confirmResult = await this.$confirm('删除么？', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).catch(err => err)
      if (confirmResult === 'confirm') {
        const { data: res } = await this.$http.delete(
          `roles/${role.id}/rights/${rightId}`
        )
        if (res.meta.status !== 200) {
          return this.$message.error(res.meta.msg)
        }
        // 不建议重新渲染RolesList，会收起expand
        // this.getRolesList()
        // 给role重新赋值，不会收起expand
        // console.log(role, role.children, res.data)
        role.children = res.data
      }
    }
  },
  created() {
    this.getRolesList()
  }
}
</script>
<style lang="less" scoped>
.el-tag {
  margin: 7px;
}
.bdtop {
  border-top: 1px solid #eee;
}
.bdbottom {
  border-bottom: 1px solid #eee;
}
.vcenter {
  display: flex;
  align-items: center;
}
</style>
