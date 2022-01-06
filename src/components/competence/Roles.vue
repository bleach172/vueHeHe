<template>
  <div>
    <!-- 面包屑导航 -->
    <el-breadcrumb separator="/">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>权限管理</el-breadcrumb-item>
      <el-breadcrumb-item>角色列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片视图 -->
    <el-card>
      <el-table :data="rolesList" border stripe>
        <el-table-column type="expand">
          <template slot-scope="scope">
            <el-row :class="['bottom-line', 'roles-center' , i1 === 0 ? 'top-line' :'']" v-for="(item1, i1) in scope.row.children" :key="item1.id">
              <!-- 渲染一级权限 -->
              <el-col :span="5">
                <el-tag>{{item1.authName}}</el-tag>
                <i class="el-icon-caret-right"></i>
              </el-col>
              <!-- 渲染二级权限 -->
              <el-col :span="19">
                <el-row :class="[ 'roles-center' , i2 === 0? '' : 'top-line']" v-for="(item2,i2) in item1.children" :key="item2.id">
                  <el-col :span="4">
                    <el-tag type="success">{{item2.authName}}</el-tag>
                    <i class="el-icon-caret-right"></i>
                  </el-col>
                  <!-- 渲染三级权限 -->
                  <el-col :span="20">
                    <el-tag @close="removetag3(scope.row,item3.id)" v-for="(item3) in item2.children" :key="item3.id" type="warning" closable>{{item3.authName}}</el-tag>
                  </el-col>
                </el-row>
              </el-col>
            </el-row>
          </template>
        </el-table-column>
        <el-table-column type="index"></el-table-column>
        <el-table-column label="角色名称" prop="roleName"></el-table-column>
        <el-table-column label="角色描述" prop="roleDesc"></el-table-column>
        <el-table-column label="操作" width="300px">
          <template slot-scope="scope">
            <el-button type="primary" icon="el-icon-edit" size="mini">编辑</el-button>
            <el-button type="danger" icon="el-icon-delete" size="mini">删除</el-button>
            <el-button @click="showSetRightsDialog(scope.row)" type="warning" icon="el-icon-setting" size="mini">分配权限</el-button>
          </template>
        </el-table-column>
      </el-table>
    </el-card>
    <!-- 分配权限的对话框 -->
    <el-dialog title="分配权限" @close="setRightsdialogClose"  :visible.sync="showSetRightsVisible" width="50%"  class="roll-dialog">
      <el-tree :data="rightsList" :props="rightsProps" ref="treeRef" :default-checked-keys="defKeys" node-key="id" show-checkbox default-expand-all></el-tree>
      <span slot="footer" class="dialog-footer">
        <el-button @click="showSetRightsVisible = false">取 消</el-button>
        <el-button type="primary" @click="allotRights">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data () {
    return {
      rolesList: [],
      showSetRightsVisible: false,
      // 所有权限的数据
      rightsList: [],
      // 树形控件的属性绑定对象
      rightsProps: {
        label: 'authName',
        children: 'children'
      },
      defKeys:[],
      // 需要分配权限的ID
      roleId:''
    }
  },
  created () {
    this.getRolesList()
  },
  methods: {
    async getRolesList () {
      const { data: res } = await this.$http.get('roles')
      if (res.meta.status !== 200) return this.$message.error('获取角色列表失败')
      this.rolesList = res.data
    },
    async removetag3 (role, rightId) {
      const confirmResult = await this.$confirm('此操作将永久删除该文件, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).catch(err => err)
      if (confirmResult !== 'confirm') {
        return this.$message.info('用户取消了删除')
      }
      const { data: res } = await this.$http.delete(`roles/${role.id}/rights/${rightId}`)
      if (res.meta.status !== 200) {
        return this.$message.error('删除权限失败')
      }
      role.children = res.data
    },
    async showSetRightsDialog (role) {
      this.roleId = role.id
      const { data: res } = await this.$http.get('rights/tree')
      if (res.meta.status !== 200) {
        return this.$message.error('分配权限失败')
      }
      this.rightsList = res.data
      this.getDefKeys(role,this.defKeys)
      this.showSetRightsVisible = true
    },
    // 递归找到最底层id
    getDefKeys (node ,arr) {
      if(!node.children){
        return arr.push(node.id)
      }
      node.children.forEach(item => {
        this.getDefKeys (item,arr)
      })
    },
    setRightsdialogClose () {
      this.defKeys = []
    },
    async allotRights () {
      const rightKeys =[
        ...this.$refs.treeRef.getCheckedKeys(),
        ...this.$refs.treeRef.getHalfCheckedKeys()
      ]
      const idStr = rightKeys.join(',')
      const { data: res } =await this.$http.post(`roles/${this.roleId}/rights`,{rids: idStr})
      if (res.meta.status !== 200) {return this.$message.error('分配权限失败')}
      this.$message.success('分配权限成功')
      this.getRolesList()
      showSetRightsVisible = false
    }
  }
}
</script>
<!-- 样式区域 -->
<style lang='less' scoped>
/deep/ .el-tree-node__children{
  line-height: initial;
}
.el-table .el-row .el-col:first-child {
  width: 120px;
}
// .el-col {
//   min-width: 200px;
// }
.el-table .el-row .el-col:nth-child(2) .el-row .el-col:first-child {
  width: 160px;
}
.el-row {
  padding: 0 40px;
  min-width: 1100px;
}
.el-tag {
  margin: 10px;
}
.top-line {
  border-top: 1px solid #eeeeee;
}
.bottom-line {
  border-bottom: 1px solid #eeeeee;
}
/deep/ .roll-dialog .el-dialog__body {
  padding: 3px 30px;
  overflow-y: auto;
    margin-top: 40px;
  height: calc(100vh - 400px);
}
/deep/ .el-dialog {
  margin-top: 40px!important;
}
</style>
