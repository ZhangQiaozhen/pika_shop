<template>
  <div>
    <!-- 面包屑导航 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>权限管理</el-breadcrumb-item>
      <el-breadcrumb-item>角色列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片视图区域 -->
    <el-card>
      <!-- 添加角色按钮区域 -->
      <el-row>
        <el-col>
          <el-button type="warning"
                     @click="addDialogVisible='true'">添加角色</el-button>
        </el-col>
      </el-row>
      <!-- 角色列表区域 -->
      <el-table :data="rolelist"
                border
                stripe>
        <!-- 展开列 -->
        <el-table-column type="expand">
          <template slot-scope="scope">
            <el-row v-for="(item1,i1) in scope.row.children"
                    :key="item1.id"
                    :class="['bdbottom',i1===0?'bdtop':'','vcenter']">
              <!-- 渲染一级权限 -->
              <el-col :span="5">
                <el-tag type="warning"
                        closable
                        @close="removeRightById(scope.row,item1.id)">{{item1.authName}}</el-tag>
                <i class="el-icon-caret-right"></i>
              </el-col>
              <!-- 渲染二级、三级权限 -->
              <el-col :span="19">
                <el-row v-for="(item2,i2) in item1.children"
                        :key="item2.id"
                        :class="[i2 === 0 ? '':'bdtop','vcenter']">
                  <el-col :span="6">
                    <el-tag type="danger"
                            closable
                            @close="removeRightById(scope.row,item2.id)">{{item2.authName}}</el-tag>
                    <i class="el-icon-caret-right"></i>
                  </el-col>
                  <el-col :span="18">
                    <el-tag v-for="(item3) in item2.children"
                            :key="item3.id"
                            :class="[i2 === 0 ? '':'bdtop']"
                            closable
                            @close="removeRightById(scope.row,item3.id)">{{item3.authName}}</el-tag>
                  </el-col>
                </el-row>
              </el-col>
            </el-row>
          </template>
        </el-table-column>
        <!-- 索引列 -->
        <el-table-column type="index"></el-table-column>
        <el-table-column label="角色名称"
                         prop="roleName"></el-table-column>
        <el-table-column label="角色描述"
                         prop="roleDesc"></el-table-column>
        <el-table-column label="操作">
          <template slot-scope="scope">
            <el-button size="mini"
                       type="warning"
                       icon="el-icon-edit"
                       @click="showEditDialog(scope.row.id)">编辑</el-button>
            <el-button size="mini"
                       type="primary"
                       icon="el-icon-delete"
                       @click="removeRoleById(scope.row.id)">删除</el-button>
            <el-button size="mini"
                       type="danger"
                       icon="el-icon-setting"
                       @click="showSetRightDialog(scope.row)">分配权限</el-button>
          </template>
        </el-table-column>

      </el-table>
    </el-card>

    <!-- 添加新角色-弹出对话框 -->
    <el-dialog title="添加角色"
               :visible.sync="addDialogVisible"
               width="50%"
               @close="addDialogClosed">
      <el-form :model="addForm"
               :rules="addFormRules"
               ref="addFormRef"
               label-width="100px">
        <!-- 角色名称 -->
        <el-form-item label="角色名称"
                      prop="roleName">
          <el-input v-model="addForm.roleName"></el-input>
        </el-form-item>
        <!-- 角色描述 -->
        <el-form-item label="角色描述"
                      prop="roleDesc">
          <el-input v-model="addForm.roleDesc"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer">
        <el-button @click="addDialogVisible = false">取 消</el-button>
        <el-button type="primary"
                   @click="addUser">确 定</el-button>
      </span>
    </el-dialog>

    <!-- 编辑提交角色 -->
    <el-dialog title="修改角色"
               :visible.sync="editDialogVisible"
               width="50%">
      <el-form :model="editForm"
               :rules="editFormRules"
               ref="editFormRef"
               label-width="100px">
        <!-- 角色名称 -->
        <el-form-item label="角色名称"
                      prop="roleName">
          <el-input v-model="editForm.roleName"></el-input>
        </el-form-item>
        <!-- 角色描述 -->
        <el-form-item label="角色描述">
          <el-input v-model="editForm.roleDesc"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer">
        <el-button @click="editDialogVisible = false">取 消</el-button>
        <el-button type="primary"
                   @click="editRole">确 定</el-button>
      </span>
    </el-dialog>

    <!-- 分配权限-弹出框 -->
    <el-dialog title="提示"
               :visible.sync="setRightDialogVisible"
               width="50%"
               @close="setRightDialogClosed">
      <!-- 给这棵树定义一个默认勾选框的数组 default-checked-keys -->
      <el-tree :data="rightslist"
               :props="treeProps"
               show-checkbox
               node-key="id"
               default-expand-all
               :default-checked-keys="defKeys"
               ref="treeRef"></el-tree>
      <span slot="footer">
        <el-button @click="setRightDialogVisible = false">取 消</el-button>
        <el-button type="primary"
                   @click="allotRights">确 定</el-button>
      </span>
    </el-dialog>

  </div>
</template>

<script>
export default {
  data () {
    return {
      // 所有角色列表数据
      rolelist: [],
      // 默认添加用户中的弹框隐藏
      addDialogVisible: false,
      // 添加新角色中的表单数据
      addForm: {
        roleName: '',
        roleDesc: ''
      },
      // 添加新角色的表单规则
      addFormRules: {
        roleName: { required: true, message: '请输入角色名称', trigger: 'blur' }
      },
      // 默认编辑角色中的弹框隐藏
      editDialogVisible: false,
      // 存放编辑角色的表单数据
      editForm: {},
      // 编辑角色中的表单规则
      editFormRules: {
        roleName: { required: true, message: '请输入角色名称', trigger: 'blur' }
      },
      // 默认分配权限中的弹框隐藏
      setRightDialogVisible: false,
      // 保存所有权限列表
      rightslist: [],
      // 树形控件的树形绑定对象
      treeProps: {
        label: 'authName',
        children: 'children'
      },
      // 默认选中的节点Id值数组
      defKeys: [],
      // 当前即将分配角色的id
      roleId: ''
    }
  },
  created () {
    this.getRolesList()
  },
  methods: {
    // 获取用户列表信息
    async getRolesList () {
      const { data: res } = await this.$http.get('roles')
      if (res.meta.status !== 200) {
        return this.$message.error('获取数据失败')
      }
      this.rolelist = res.data
      // console.log(this.rolelist);
    },
    //  监听添加角色对话框的关闭事件
    addDialogClosed () {
      this.$refs.addFormRef.resetFields()
    },
    // 点击按钮添加新角色
    addUser () {
      this.$refs.addFormRef.validate(async valid => {
        if (!valid) return
        // 验证成功则发起添加用户的网络请求
        const { data: res } = await this.$http.post('roles', this.addForm)
        if (res.meta.status !== 201) {
          return this.$message.error('创建失败')
        }
        this.getRolesList()
        this.$message.success(res.meta.msg)
        this.addDialogVisible = false
      })
    },
    // 根据Id删除对应的权限
    async removeRightById (role, rightId) {
      const confirmResult = await this.$confirm('此操作将永久删除该信息, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).catch(err => err)
      if (confirmResult !== 'confirm') {
        return this.$message.warning('已经取消删除！')
      }
      const { data: res } = await this.$http.delete(`roles/${role.id}/rights/${rightId}`)
      if (res.meta.status !== 200) {
        return this.$message.error('删除权限失败')
      }
      role.children = res.data
      this.$message.success(res.meta.msg)
    },
    // 根据展示分配权限的对话框
    async showSetRightDialog (role) {
      this.roleId = role.id
      const { data: res } = await this.$http.get('rights/tree')
      if (res.meta.status !== 200) {
        return this.$message.error('获取权限失败！')
      }
      this.rightslist = res.data
      // console.log(this.rightslist);
      // 递归获取三级节点的id
      this.getLeafKeys(role, this.defKeys)

      this.setRightDialogVisible = true
    },
    // 通过递归的形式，获取角色下所有三级权限的id,并保存到 deKeys 数组中
    getLeafKeys (node, arr) {
      // 如果当前 node 节点不包含 children属性，则是三级节点
      if (!node.children) {
        return arr.push(node.id)
      }
      // 循环node节点，每循环一次，拿到子节点item，再用item作为节点再次调用递归函数
      node.children.forEach(item => {
        this.getLeafKeys(item, arr)
      });
    },
    setRightDialogClosed () {
      this.defKeys = []
    },

    // 点击确认为角色分配权限
    async allotRights () {
      const keys = [
        ...this.$refs.treeRef.getCheckedKeys(),
        ...this.$refs.treeRef.getHalfCheckedKeys()
      ]
      // console.log(keys);
      const idStr = keys.join(',')
      const { data: res } = await this.$http.post(`roles/${this.roleId}/rights`, { rids: idStr })
      if (res.meta.status !== 200) {
        return this.$message.error('更新失败')
      }
      this.$message.success(res.meta.msg)
      this.getRolesList()
      this.setRightDialogVisible = false
    },

    // 编辑角色前提：根据id获取该用户信息并保存
    async showEditDialog (id) {
      const { data: res } = await this.$http.get('roles/' + id)
      if (res.meta.status !== 200) {
        return this.$message.error('查询失败')
      }
      this.editForm = res.data
      console.log(this.editForm);
      this.editDialogVisible = true
    },
    // 编辑角色：关闭弹窗时清空信息
    editDialogClosed () {
      this.$refs.editFormRef.resetFields()
    },
    // 编辑提交角色
    editRole () {
      this.$refs.editFormRef.validate(async valid => {
        if (!valid) return
        // console.log(valid);
        // 验证成功
        const { data: res } = await this.$http.put('roles/' + this.editForm.roleId, {
          roleName: this.editForm.roleName,
          roleDesc: this.editForm.roleDesc
        })
        // console.log(res);
        if (res.meta.status !== 200) {
          return this.$message.error('修改角色失败')
        }
        this.editDialogVisible = false
        this.$message.success('修改角色成功')
        this.getRolesList()
      })
    },
    // 删除角色
    // 弹窗询问是否删除角色
    async removeRoleById (id) {
      const confirmResult = await this.$confirm('此操作将永久删除该角色, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).catch((err) => {
        return err
      });
      if (confirmResult !== 'confirm') {
        return this.$message.warning('已取消删除')
      }
      const { data: res } = await this.$http.delete('roles/' + id)
      if (res.meta.status != 200) {
        return this.$message.error('删除失败')
      }
      this.$message.success(res.meta.msg)
      this.getRolesList()
    },

  }
}
</script>

<style lang="less" scoped>
.el-tag {
  margin: 6px;
}
.bdtop {
  border-top: 1px solid #f3e6ad;
}
.bdbottom {
  border-bottom: 1px solid #f3e6ad;
}
.vcenter {
  display: flex;
  align-items: center;
}
</style>