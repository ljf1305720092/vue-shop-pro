<template>
  <div>
    <!-- 面包屑区域 -->
    <el-breadcrumb separator="/">
      <el-breadcrumb-item :to="{ path: '/' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>用户管理</el-breadcrumb-item>
      <el-breadcrumb-item>用户列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- Card卡片区 -->
    <el-card class="box-card">
      <!-- 搜索框 -->
      <el-row :gutter="20">
        <el-col :span="8">
          <el-input
            placeholder="请输入搜索内容"
            v-model="querycdt.query"
            :clearable="true"
            @clear="getUserInfos"
            @keyup.enter.native="getUserInfos"
            class="input-with-select"
          >
            <el-button @click="getUserInfos" slot="append" icon="el-icon-search"></el-button>
          </el-input>
        </el-col>
        <el-col :span="6">
          <el-button type="primary" @click="AddDialogVisible=true">添加用户</el-button>
          <el-dialog
            title="添加用户"
            :visible.sync="AddDialogVisible"
            width="50%"
            @close="addDialogclose"
          >
            <!-- 添加用户表单 -->
            <el-form
              :label-position="labelPosition"
              :rules="addRules"
              label-width="80px"
              :model="addFormLabelAlign"
              ref="close"
            >
              <el-form-item label="用户名" prop="username">
                <el-input v-model="addFormLabelAlign.username"></el-input>
              </el-form-item>
              <el-form-item label="密码" prop="password">
                <el-input v-model="addFormLabelAlign.password"></el-input>
              </el-form-item>
              <el-form-item label="邮箱">
                <el-input v-model="addFormLabelAlign.email"></el-input>
              </el-form-item>
              <el-form-item label="手机号码" prop="mobile">
                <el-input v-model="addFormLabelAlign.mobile"></el-input>
              </el-form-item>
            </el-form>

            <span slot="footer" class="dialog-footer">
              <el-button @click="AddDialogVisible = false">取 消</el-button>
              <el-button type="primary" @click="addUser">确 定</el-button>
            </span>
          </el-dialog>
        </el-col>
      </el-row>

      <!-- 表格区 -->
      <el-table :data="userInfos" border style="width: 100%">
        <el-table-column type="index" label="序号" width="60"></el-table-column>
        <el-table-column prop="username" label="用户名"></el-table-column>
        <el-table-column prop="mobile" label="手机号码" width="110"></el-table-column>
        <el-table-column prop="role_name" label="角色" width="120"></el-table-column>
        <el-table-column prop="email" label="邮箱" width="160"></el-table-column>
        <el-table-column prop="mg_state" label="状态" width="70">
          <!-- 插槽填充 -->
          <el-switch
            v-model="info.row.mg_state"
            slot-scope="info"
            @change="changestate(info.row.id,info.row.mg_state)"
          ></el-switch>
        </el-table-column>
        <el-table-column label="操作" width="230">
          <template slot-scope="info">
            <!-- 修改用户 -->
            <el-button
              type="primary"
              size="mini"
              icon="el-icon-edit"
              @click="showEditDialog(info.row.id)"
            ></el-button>
            <el-dialog
              title="修改用户"
              :visible.sync="xiuDialogVisible"
              width="50%"
              @close="xiuDialogClose"
            >
              <!-- 修改用户表单 -->
              <el-form
                :label-position="labelPosition"
                :rules="xiuFormRules"
                label-width="80px"
                :model="xiuForm"
                ref="xiuFormRef"
              >
                <el-form-item label="用户名" prop="username">
                  <el-input v-model="xiuForm.username" :disabled="true"></el-input>
                </el-form-item>
                <el-form-item label="邮箱" prop="email">
                  <el-input v-model="xiuForm.email"></el-input>
                </el-form-item>
                <el-form-item label="手机号码" prop="mobile">
                  <el-input v-model="xiuForm.mobile"></el-input>
                </el-form-item>
              </el-form>

              <span slot="footer" class="dialog-footer">
                <el-button @click="xiuDialogVisible = false">取 消</el-button>
                <el-button type="primary" @click="xiuUser">确 定</el-button>
              </span>
            </el-dialog>

            <!-- 删除用户表单 -->
            <el-button
              type="danger"
              size="mini"
              icon="el-icon-delete"
              @click="delUser(info.row.id)"
            ></el-button>

            <el-tooltip
              class="item"
              effect="dark"
              content="分配角色"
              :enterable="false"
              placement="top"
            >
              <el-button type="warning" size="mini" icon="el-icon-setting"></el-button>
              <el-button>上边</el-button>
            </el-tooltip>
          </template>
        </el-table-column>
      </el-table>

      <!-- 分页区域 -->
      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="querycdt.pagenum - 0"
        :page-sizes="[2, 4, 8, 10]"
        :page-size="querycdt.pagesize-0"
        layout="total, sizes, prev, pager, next, jumper"
        :total="querycdt.tot"
      ></el-pagination>
    </el-card>
  </div>
</template>

<script>
export default {
  created() {
    this.getUserInfos()
  },
  methods: {
    xiuDialogClose() {
      this.$refs.xiuFormRef.resetFields()
    },
    xiuUser() {
      this.$refs.xiuFormRef.validate(async valid => {
        if (valid) {
          const { data: res } = await this.$http.put(
            `users/${this.xiuForm.id}`,
            this.xiuForm
          )

          if (res.meta.status !== 200) {
            return this.$message.error(res.meta.msg)
          }

          this.xiuDialogVisible = false
          this.$message.success(res.meta.msg)
          this.getUserInfos()
        }
      })
    },
    //   修改数据
    async showEditDialog(id) {
      // 显示对话框
      this.xiuDialogVisible = true
      // 根据id获得被修改的用户数据
      const { data: res } = await this.$http.get(`users/${id}`)

      if (res.meta.status !== 200) {
        return this.$message.error(res.meta.msg)
      }

      // 把获取到的用户信息  赋予给xiuFrom表单数据对象
      this.xiuForm = res.data
    },
    //   删除数据
    delUser(id) {
      this.$confirm('确认删除该用户吗？', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      })
        .then(async () => {
          const { data: res } = await this.$http.delete(`users/${id}`)
          if (res.meta.status !== 200) {
            return this.$message.error(res.meta.msg)
          }
          // 删除成功
          this.$message.success(res.meta.msg)
          this.getUserInfos()
        })
        .catch(() => {})
    },
    addUser() {
      // 先验证form表单 vaildate
      this.$refs.close.validate(async valid => {
        if (valid) {
          const { data: res } = await this.$http.post(
            'users',
            this.addFormLabelAlign
          )
          // 添加失败
          if (res.meta.status !== 201) {
            return this.$message.error(res.meta.msg)
          }
          // 添加成功：关闭对话框
          this.AddDialogVisible = false
          this.$message.success(res.meta.msg)
          this.getUserInfos()
        }
      })
    },
    addDialogclose() {
      this.$refs.close.resetFields()
    },
    // 修改用户状态的方法
    // uid：被修改状态用户的id值
    // state：被修改后状态信息true/false
    async changestate(uid, state) {
      const { data: res } = await this.$http.put(`users/${uid}/state/${state}`)
      if (res.meta.status !== 200) {
        return this.$message.error(res.meta.msg)
      }
      this.$message.success(res.meta.msg)
    },
    // 从数据库获取数据
    handleSizeChange(arg) {
      // arg 变化后的记录条数
      this.querycdt.pagesize = arg
      // 重新获得数据
      this.getUserInfos()
    },
    handleCurrentChange(arg) {
      // arg 变化后的页数
      this.querycdt.pagenum = arg
      // 重新获得数据
      this.getUserInfos()
    },
    async getUserInfos() {
      const { data: res } = await this.$http.get('users', {
        params: this.querycdt
      })

      if (res.meta.status !== 200) {
        return this.$message.error(res.meta.msg)
      }
      //   记录总记录条数
      this.querycdt.tot = res.data.total

      this.userInfos = res.data.users
    }
  },
  data() {
    //   为效验手机号码生成一个函数
    var checkMobile = (rule, value, callback) => {
      if (!/^1[35789]\d{9}$/.test(value)) {
        // 效验失败（给页面提示错误信息）
        return callback(new Error('手机号码格式不正确'))
      }
      // 效验成功,继续执行
      callback()
    }
    return {
      // 修改用户区域
      // 控制修改用户对话框是否显示
      xiuDialogVisible: false,
      // form表单需要的数据
      xiuForm: {
        username: '',
        email: '',
        mobile: ''
      },
      // 制作表单域效验规则
      xiuFormRules: {
        mobile: [
          { required: true, message: '请输入手机号码', trigger: 'blur' },
          { validator: checkMobile, trigger: 'blur' }
        ]
      },
      // 添加用户表单区域
      labelPosition: 'right',
      addFormLabelAlign: {
        username: '',
        password: '',
        email: '',
        mobile: ''
      },
      addRules: {
        username: [
          { required: true, message: '请输入用户名', trigger: 'blur' }
        ],
        password: [{ required: true, message: '请输入密码', trigger: 'blur' }],
        mobile: [
          { required: true, message: '请输入手机号码', trigger: 'blur' },
          { validator: checkMobile, trigger: 'blur' }
        ]
      },
      //   制作表单验证规则
      xiuDialogVisible: false,

      AddDialogVisible: false,
      //   搜索关键词
      keywords: '',
      //   用户数据
      userInfos: [],
      //   查询条件
      querycdt: {
        //    查询关键字
        query: '',
        // 当前页码
        pagenum: '1',
        // 一页显示几个
        pagesize: '2',
        // 总记录条数
        tot: 0
      }
    }
  }
}
</script>

<style lang="less" scoped>
</style>
