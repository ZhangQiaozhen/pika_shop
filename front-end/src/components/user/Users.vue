<template>
  <div>
    <!-- 面包屑 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/welcome' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>用户管理</el-breadcrumb-item>
      <el-breadcrumb-item>用户列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片视图区域 -->
    <el-card>
      <!-- 搜索与添加区域 -->
      <el-row :gutter="20">
        <el-col :span="10">
          <el-input placeholder="请输入内容"
                    v-model="queryInfo.query"
                    clearable
                    @clear="getUserList">
            <el-button slot="append"
                       icon="el-icon-search"
                       @click="getUserList"></el-button>
          </el-input>
        </el-col>
        <el-col :span="1">
          <el-button type="warning"
                     @click="addDialogVisible = true">添加</el-button>
        </el-col>
      </el-row>
      <!-- 用户列表区域 -->
      <el-table :data="userlist"
                border
                stripe>
        <el-table-column type="index"></el-table-column>

        <el-table-column prop="username"
                         label="姓名"></el-table-column>

        <el-table-column prop="email"
                         label="邮箱"></el-table-column>

        <el-table-column prop="mobile"
                         label="电话"></el-table-column>

        <el-table-column prop="role_name"
                         label="角色"></el-table-column>
        <!-- 状态栏里放置了个插槽装按钮 -->
        <el-table-column label="状态">
          <template slot-scope="scope">
            <el-switch active-color="#eec202"
                       inactive-color="#d9d1ad"
                       v-model="scope.row.mg_state"
                       @change="userStateChanged(scope.row)">
            </el-switch>
          </template>
        </el-table-column>
        <!-- 操作栏里放置了个插槽放了三个按钮，最后一个按钮给文字解说包裹住 -->
        <el-table-column label="操作"
                         width="180px">
          <template slot-scope="scope">
            <!-- 修改按钮 -->
            <el-button type="warning"
                       icon="el-icon-edit"
                       size="mini"
                       @click="showEditDialog(scope.row.id)"></el-button>
            <!-- 删除按钮 -->
            <el-button type="success"
                       icon="el-icon-delete"
                       size="mini"
                       @click="removeUserById(scope.row.id)"></el-button>
            <!-- 角色分配按钮 -->
            <el-tooltip effect="dark"
                        content="分配角色"
                        placement="top"
                        :enterable="false">
              <el-button type="danger"
                         icon="el-icon-setting"
                         size="mini"
                         @click="setRole(scope.row)"></el-button>
            </el-tooltip>
          </template>
        </el-table-column>
      </el-table>
      <!-- 分页区域 -->
      <el-pagination @size-change="handleSizeChange"
                     @current-change="handleCurrentChange"
                     :current-page="queryInfo.pagenum"
                     :page-sizes="[1, 2, 3, 10]"
                     :page-size="queryInfo.pagesize"
                     layout="total, sizes, prev, pager, next, jumper"
                     :total="total">
      </el-pagination>

    </el-card>

    <!-- 添加用户-弹出对话框 -->
    <el-dialog title="用户信息"
               :visible.sync="addDialogVisible"
               width="50%"
               @close="addDialogClosed">
      <el-form :model="addForm"
               :rules="addFormRules"
               ref="addFormRef"
               label-width="100px">
        <el-form-item label="用户名"
                      prop="username">
          <el-input v-model="addForm.username"></el-input>
        </el-form-item>

        <el-form-item label="用户密码"
                      prop="password">
          <el-input v-model="addForm.password"></el-input>
        </el-form-item>

        <el-form-item label="邮箱"
                      prop="email">
          <el-input v-model="addForm.email"></el-input>
        </el-form-item>

        <el-form-item label="手机号"
                      prop="mobile">
          <el-input v-model="addForm.mobile"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer"
            class="dialog-footer">
        <el-button @click="addDialogVisible = false">取 消</el-button>
        <el-button type="primary"
                   @click="addUser(scope.row)">确 定</el-button>
      </span>
    </el-dialog>

    <!-- 修改用户-弹出对话框 -->
    <el-dialog title="修改用户信息"
               :visible.sync="editDialogVisible"
               width="50%"
               @close="editDialogClosed">

      <el-form :model="editform"
               :rules="editFormRules"
               ref="editFormRef"
               label-width="100px">
        <!-- 用户名 -->
        <el-form-item label="用户名">
          <el-input v-model="editform.username"
                    disabled></el-input>
        </el-form-item>
        <!-- 邮箱 -->
        <el-form-item label="邮箱"
                      prop="email">
          <el-input v-model="editform.email"></el-input>
        </el-form-item>
        <!-- 手机 -->
        <el-form-item label="手机"
                      prop="mobile">
          <el-input v-model="editform.mobile"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer"
            class="dialog-footer">
        <el-button @click="editDialogVisible = false">取 消</el-button>
        <el-button type="primary"
                   @click="editUserInfo">确 定</el-button>
      </span>
    </el-dialog>

    <!-- 分配角色-弹出对话框 -->
    <el-dialog title="分配角色"
               :visible.sync="setRoleDialogVisible"
               width="50%"
               @close="setRoleDialogClosed">
      <div>
        <p>当前用户：{{userInfo.username}}</p>
        <p>当前角色：{{userInfo.role_name}}</p>
        <p>分配新角色：
          <el-select v-model="selectedRoleId"
                     placeholder="请选择">
            <el-option v-for="item in rolesList"
                       :key="item.id"
                       :label="item.roleName"
                       :value="item.id">
            </el-option>
          </el-select>
        </p>
      </div>
      <span slot="footer"
            class="dialog-footer">
        <el-button @click="setRoleDialogVisible = false">取 消</el-button>
        <el-button type="primary"
                   @click="saveRoleInfo">确 定</el-button>
      </span>
    </el-dialog>

  </div>

</template>

<script>
export default {
  data () {
    // 验证邮箱的规则
    var checkEmail = (rule, value, cb) => {
      // 验证邮箱的正则表达式
      const regEmail = /^([a-zA-Z0-9_-])+@([a-zA-Z0-9_-])+(\.[a-zA-Z0-9_-])+/
      if (regEmail.test(value)) {
        // 合法的邮箱
        return cb()
      }
      cb(new Error('您的邮箱格式有误！'))
    }
    // 验证手机号的规则
    var checkMobile = (rule, value, cb) => {
      // 验证手机号的正则表达式
      const regMobile = /^(0|86|17951)?(13[0-9]|15[012356789]|17[678]|18[0-9]|14[57])[0-9]{8}$/
      if (regMobile.test(value)) {
        // 合法的邮箱
        return cb()
      }
      cb(new Error('您手机号码有误！'))
    }
    return {
      // 获取用户列表的参数对象
      queryInfo: {
        query: '',
        pagenum: 1,
        pagesize: 3
      },
      userlist: [],
      total: 0,
      // 控制弹出框的显示与隐藏
      addDialogVisible: false,
      // 添加用户的表单数据
      addForm: {
        username: '',
        password: '',
        email: '',
        mobile: ''
      },
      // 添加用户的表单规则
      addFormRules: {
        username: [
          { required: true, message: '请输入用户名', trigger: 'blur' },
          { min: 3, max: 10, message: '长度在 3 到 10 个字符', trigger: 'blur' }
        ],
        password: [
          { required: true, message: '请输入密码', trigger: 'blur' },
          { min: 6, max: 12, message: '长度在 6 到 12个字符', trigger: 'blur' }
        ],
        email: [
          { required: true, message: '请输入邮箱', trigger: 'blur' },
          { validator: checkEmail, trigger: 'blur' }
        ],
        mobile: [
          { required: true, message: '请输入手机号码', trigger: 'blur' },
          { validator: checkMobile, trigger: 'blur' }
        ]
      },
      editDialogVisible: false,
      // 用于暂存修改表单的数据
      editform: {},
      editFormRules: {
        email: [
          { required: true, message: '请输入邮箱', trigger: 'blur' },
          { validator: checkEmail, trigger: 'blur' }
        ],
        mobile: [
          { required: true, message: '请输入手机号码', trigger: 'blur' },
          { validator: checkMobile, trigger: 'blur' }
        ]
      },
      // 分配角色对话框是否显示
      setRoleDialogVisible: false,
      // 需要被分配权限的用户信息
      userInfo: {},
      // 所有角色的数据列表
      rolesList: [],
      // 已经选中的角色id
      selectedRoleId: ''

    }
  },
  created () {
    this.getUserList()
  },
  methods: {
    // 获取用户列表信息
    async getUserList () {
      const { data: res } = await this.$http.get('users', { params: this.queryInfo })
      if (res.meta.status !== 200) return this.$message.error(res.meta.msg);
      this.userlist = res.data.users
      this.total = res.data.total
      console.log(res);
    },
    // 监听 pagesize 改变的事件
    handleSizeChange (newPagezize) {
      // console.log(this.queryInfo.pagesize);
      this.queryInfo.pagesize = newPagezize
      // 重新获取列表
      this.getUserList()
    },
    // 监听 页码值 改变的事件
    handleCurrentChange (newPagenum) {
      // console.log(this.queryInfo.pagenum);
      this.queryInfo.pagenum = newPagenum
      this.getUserList()
    },
    // 监听 switch 开关状态的改变
    async userStateChanged (userinfo) {
      // console.log(userinfo);
      const { data: res } = await this.$http.put(`users/${userinfo.id}/state/${userinfo.mg_state}`)
      if (res.meta.status !== 200) {
        userinfo.mg_state = !userinfo.mg_state
        return this.$message.error('设置状态失败！')
      }
      this.$message.success(res.meta.msg)
    },
    // 监听添加用户对话框的关闭事件
    addDialogClosed () {
      this.$refs.addFormRef.resetFields()
    },
    // 点击按钮，添加新用户
    addUser () {
      this.$refs.addFormRef.validate(async valid => {
        if (!valid) return
        //验证成功则可以发起添加用户的网络请求
        const { data: res } = await this.$http.post('users', this.addForm)
        if (res.meta.status !== 201) {
          this.$message.error('用户创建失败')
        }
        this.$message.success(res.meta.msg)
        // 隐藏弹出框
        this.addDialogVisible = false
        // 重新获取用户列表数据
        this.getUserList()
      })
    },
    // 展示编辑用户的对话框
    async showEditDialog (id) {
      // console.log(id);
      const { data: res } = await this.$http.get('users/' + id)
      if (res.meta.status != 200) {
        return this.$message.error('查询失败！')
      }
      this.editform = res.data
      this.editDialogVisible = true
    },
    // 监听修改用户对话款的关闭事件
    editDialogClosed () {
      this.$refs.editFormRef.resetFields()
    },
    // 修改用户信息并提交
    editUserInfo () {
      this.$refs.editFormRef.validate(async valid => {
        if (!valid) return
        // 成功则发起修改用户信息请求
        const { data: res } = await this.$http.put('users/' + this.editform.id, {
          email: this.editform.email,
          mobile: this.editform.mobile
        })
        if (res.meta.status !== 200) {
          return this.$message.error('更新失败！')
        }
        this.$message.success(res.meta.msg)
        this.editDialogVisible = false
        this.getUserList()
      })
    },
    // 弹窗询问是否删除用户
    async removeUserById (id) {
      const confirmResult = await this.$confirm('此操作将永久删除该用户, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).catch((err) => {
        return err
      });
      //上面可简写成 catch(err => err)
      //  如果用户确认删除，则返回字符串 confirm
      // 如果用户取消了删除，则返回字符串 cancel
      // console.log(confirmResult);
      if (confirmResult !== 'confirm') {
        return this.$message.warning('已取消删除')
      }
      const { data: res } = await this.$http.delete('users/' + id)
      if (res.meta.status != 200) {
        return this.$message.error('删除失败')
      }
      this.$message.success(res.meta.msg)
      this.getUserList()
    },
    // 展示分配权限的对话框 
    async setRole (userInfo) {
      this.userInfo = userInfo
      // 展示对话框的下拉列表之前获取所有的角色列表
      const { data: res } = await this.$http.get('roles')
      if (res.meta.status !== 200) {
        return this.$message.error('获取列表失败')
      }
      // 获取列表成功的话，把列表保存在data中
      this.rolesList = res.data
      this.setRoleDialogVisible = true
    },
    // 点击确认按钮，分配角色
    async saveRoleInfo () {
      if (!this.selectedRoleId) {
        return this.$message.error('更新角色失败')
      }
      const { data: res } = await this.$http.put(`users/${this.userInfo.id}/role`, { rid: this.selectedRoleId })
      if (res.meta.status !== 200) {
        return this.$message.error('更新角色失败！')
      }
      this.$message.success('更新角色成功！')
      this.getUserList()
      this.setRoleDialogVisible = false
    },
    setRoleDialogClosed () {
      this.selectedRoleId = ''
      this.userInfo = {}
    }
  }
}
</script>

<style lang="less" scoped>
</style>