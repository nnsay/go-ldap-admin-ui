<template>
  <div>
    <el-card class="container-card" shadow="always">
      <el-form size="mini" :inline="true" :model="params" class="demo-form-inline">
        <el-form-item label="用户名">
          <el-input v-model.trim="params.username" clearable placeholder="用户名" @clear="search" />
        </el-form-item>
        <el-form-item label="昵称">
          <el-input v-model.trim="params.nickname" clearable placeholder="昵称" @clear="search" />
        </el-form-item>
        <el-form-item label="状态">
          <el-select v-model.trim="params.status" clearable placeholder="状态" @change="search" @clear="search">
            <el-option label="正常" value="1" />
            <el-option label="禁用" value="2" />
          </el-select>
        </el-form-item>
        <el-form-item label="手机号">
          <el-input v-model.trim="params.mobile" clearable placeholder="手机号" @clear="search" />
        </el-form-item>
        <el-form-item>
          <el-button :loading="loading" icon="el-icon-search" type="primary" @click="search">查询</el-button>
        </el-form-item>
        <el-form-item>
          <el-button :loading="loading" icon="el-icon-plus" type="warning" @click="create">新增</el-button>
        </el-form-item>
        <el-form-item>
          <el-button :disabled="multipleSelection.length === 0" :loading="loading" icon="el-icon-delete" type="danger" @click="batchDelete">批量删除</el-button>
        </el-form-item>
        <br>
        <el-form-item>
          <el-button :loading="loading" icon="el-icon-share" type="danger" @click="syncOpenLdapUsers">同步原ldap用户信息</el-button>
        </el-form-item>
        <el-form-item>
          <el-button :loading="loading" icon="el-icon-share" type="danger" @click="syncDingTalkUsers">同步钉钉用户信息</el-button>
        </el-form-item>
        <el-form-item>
          <el-button :loading="loading" icon="el-icon-share" type="danger" @click="syncFeiShuUsers">同步飞书用户信息</el-button>
        </el-form-item>
        <el-form-item>
          <el-button :loading="loading" icon="el-icon-share" type="danger" @click="syncWeComUsers">同步企业微信用户信息</el-button>
        </el-form-item>
      </el-form>

      <el-table v-loading="loading" :data="tableData" border stripe style="width: 100%" @selection-change="handleSelectionChange">
        <el-table-column type="selection" width="55" align="center" />
        <el-table-column show-overflow-tooltip sortable prop="username" label="用户名" />
        <el-table-column show-overflow-tooltip sortable prop="nickname" label="中文名" />
        <el-table-column show-overflow-tooltip sortable prop="givenName" label="花名" />
        <!-- 使用按钮方式展示，以后改成布尔参数比较合适 -->
        <el-table-column label="状态" align="center">
          <template slot-scope="scope">
            <el-switch v-model="scope.row.status" :active-value='1' :inactive-value='2' @change="userStateChanged(scope.row)"></el-switch>
          </template>
        </el-table-column>
        <!-- <el-table-column show-overflow-tooltip sortable prop="status" label="状态" align="center">
          <template slot-scope="scope">
            <el-tag size="small" :type="scope.row.status === 1 ? 'success':'danger'" disable-transitions>{{ scope.row.status === 1 ? '正常':'禁用' }}</el-tag>
          </template>
        </el-table-column> -->
        <el-table-column show-overflow-tooltip sortable prop="mail" label="邮箱" />
        <el-table-column show-overflow-tooltip sortable prop="mobile" label="手机号" />
        <el-table-column show-overflow-tooltip sortable prop="jobNumber" label="工号" />
        <el-table-column show-overflow-tooltip sortable prop="departments" label="部门" />
        <el-table-column show-overflow-tooltip sortable prop="position" label="职位" />
        <el-table-column show-overflow-tooltip sortable prop="creator" label="创建人" />
        <el-table-column show-overflow-tooltip sortable prop="introduction" label="说明" />
        <el-table-column show-overflow-tooltip sortable prop="userDn" label="DN" />
        <el-table-column show-overflow-tooltip sortable prop="CreatedAt" label="创建时间" />
        <el-table-column show-overflow-tooltip sortable prop="UpdatedAt" label="更新时间" />
        <el-table-column fixed="right" label="操作" align="center" width="120">
          <template slot-scope="scope">
            <el-tooltip content="编辑" effect="dark" placement="top">
              <el-button size="mini" icon="el-icon-edit" circle type="primary" @click="update(scope.row)" />
            </el-tooltip>
            <el-tooltip class="delete-popover" content="删除" effect="dark" placement="top">
              <el-popconfirm title="确定删除吗？" @onConfirm="singleDelete(scope.row.ID)">
                <el-button slot="reference" size="mini" icon="el-icon-delete" circle type="danger" />
              </el-popconfirm>
            </el-tooltip>
          </template>
        </el-table-column>
      </el-table>

      <el-pagination
        :current-page="params.pageNum"
        :page-size="params.pageSize"
        :total="total"
        :page-sizes="[1, 5, 10, 30]"
        layout="total, prev, pager, next, sizes"
        background
        style="margin-top: 10px;float:right;margin-bottom: 10px;"
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
      />

      <el-dialog :title="dialogFormTitle" :visible.sync="dialogFormVisible" width="50%">
        <el-form ref="dialogForm" size="small" :model="dialogFormData" :rules="dialogFormRules" label-width="80px">
          <el-row>
            <el-col :span="12">
              <el-form-item label="用户名" prop="username">
                <el-input ref="password" v-model.trim="dialogFormData.username" :disabled="disabled" placeholder="用户名（拼音）" />
              </el-form-item>
            </el-col>
            <el-col :span="12">
              <el-form-item label="中文名字" prop="nickname">
                <el-input v-model.trim="dialogFormData.nickname" placeholder="中文名字" />
              </el-form-item>
            </el-col>
            <el-col :span="12">
              <el-form-item label="花名" prop="givenName">
                <el-input v-model.trim="dialogFormData.givenName" placeholder="花名" />
              </el-form-item>
            </el-col>
            <el-col :span="12">
              <el-form-item label="邮箱" prop="mail">
                <el-input v-model.trim="dialogFormData.mail" placeholder="邮箱" />
              </el-form-item>
            </el-col>
            <el-col :span="12">
              <el-form-item :label="dialogType === 'create' ? '新密码':'重置密码'" prop="password">
                <el-input v-model.trim="dialogFormData.password" autocomplete="off" :type="passwordType" :placeholder="dialogType === 'create' ? '新密码':'重置密码'" />
                <span class="show-pwd" @click="showPwd">
                  <svg-icon :icon-class="passwordType === 'password' ? 'eye' : 'eye-open'" />
                </span>
              </el-form-item>
            </el-col>
            <el-col :span="12">
              <el-form-item label="角色" prop="roleIds">
                <el-select v-model.trim="dialogFormData.roleIds" multiple placeholder="请选择角色" style="width:100%">
                  <el-option
                    v-for="item in roles"
                    :key="item.ID"
                    :label="item.name"
                    :value="item.ID"
                  />
                </el-select>
              </el-form-item>
            </el-col>
            <el-col :span="12">
              <el-form-item label="状态" prop="status">
                <el-select v-model.trim="dialogFormData.status" placeholder="请选择状态" style="width:100%">
                  <el-option label="正常" :value="1" />
                  <el-option label="禁用" :value="2" />
                </el-select>
              </el-form-item>
            </el-col>
            <el-col :span="12">
              <el-form-item label="手机号" prop="mobile">
                <el-input v-model.trim="dialogFormData.mobile" placeholder="手机号" />
              </el-form-item>
            </el-col>
            <el-col :span="12">
              <el-form-item label="工号" prop="jobNumber">
                <el-input v-model.trim="dialogFormData.jobNumber" placeholder="工号" />
              </el-form-item>
            </el-col>
            <el-col :span="12">
              <el-form-item label="职位" prop="position">
                <el-input v-model.trim="dialogFormData.position" placeholder="职业" />
              </el-form-item>
            </el-col>
            <el-col :span="24">
              <el-form-item label="所属部门" prop="departmentId">
                <treeselect
                  v-model="dialogFormData.departmentId"
                  :options="departmentsOptions"
                  placeholder="请选择部门"
                  :normalizer="normalizer"
                  value-consists-of="ALL"
                  :multiple="true"
                  :flat="true"
                  no-children-text="没有更多选项"
                  no-results-text="没有匹配的选项"
                  @input="treeselectInput"
                />
              </el-form-item>
            </el-col>
            <el-col :span="24">
              <el-form-item label="地址" prop="postalAddress">
                <el-input v-model.trim="dialogFormData.postalAddress" type="textarea" placeholder="地址" :autosize="{minRows: 3, maxRows: 6}" show-word-limit maxlength="100" />
              </el-form-item>
            </el-col>
            <el-col :span="24">
              <el-form-item label="说明" prop="introduction">
                <el-input v-model.trim="dialogFormData.introduction" type="textarea" placeholder="说明" :autosize="{minRows: 3, maxRows: 6}" show-word-limit maxlength="100" />
              </el-form-item>
            </el-col>
          </el-row>
        </el-form>
        <div slot="footer" class="dialog-footer">
          <el-button size="mini" @click="cancelForm()">取 消</el-button>
          <el-button size="mini" :loading="submitLoading" type="primary" @click="submitForm()">确 定</el-button>
        </div>
      </el-dialog>

    </el-card>
  </div>
</template>

<script>
import JSEncrypt from 'jsencrypt'
import Treeselect from '@riophae/vue-treeselect'
import '@riophae/vue-treeselect/dist/vue-treeselect.css'
import { getUsers, createUser, updateUserById, batchDeleteUserByIds, changeUserStatus, syncDingTalkUsersApi, syncWeComUsersApi, syncFeiShuUsersApi, syncOpenLdapUsersApi } from '@/api/personnel/user'
import { getRoles } from '@/api/system/role'
import { getGroupTree } from '@/api/personnel/group'

export default {
  name: 'User',
  components: {
    Treeselect
  },
  props: {
    disabled: { // username 默认不可编辑，若需要至为可编辑，请（在新增和编辑处）去掉这个值的控制，且配合后端的ldap-user-name-modify配置使用
      type: Boolean,
      default: false
    }
  },
  data() {
    var checkPhone = (rule, value, callback) => {
      if (!value) {
        return callback(new Error('手机号不能为空'))
      } else {
        const reg = /^1[3|4|5|6|7|8|9][0-9]\d{8}$/
        if (reg.test(value)) {
          callback()
        } else {
          return callback(new Error('请输入正确的手机号'))
        }
      }
    }
    return {
      // 查询参数
      params: {
        username: '',
        nickname: '',
        status: '',
        mobile: '',
        pageNum: 1,
        pageSize: 10
      },
      // 表格数据
      tableData: [],
      total: 0,
      loading: false,
      isUpdate: false,
      // 部门信息数据
      treeselectValue: 0,
      // 角色
      roles: [],
      // 部门信息
      departmentsOptions: [],

      passwordType: 'password',

      publicKey: `-----BEGIN PUBLIC KEY-----
MIGfMA0GCSqGSIb3DQEBAQUAA4GNADCBiQKBgQDbOYcY8HbDaNM9ooYXoc9s+R5o
R05ZL1BsVKadQBgOVH/kj7PQuD+ABEFVgB6rJNi287fRuZeZR+MCoG72H+AYsAhR
sEaB5SuI7gDEstXuTyjhx5bz0wUujbDK4VMgRfPO6MQo+A0c95OadDEvEQDG3KBQ
wLXapv+ZfsjG7NgdawIDAQAB
-----END PUBLIC KEY-----`,

      // dialog对话框
      submitLoading: false,
      dialogFormTitle: '',
      dialogType: '',
      dialogFormVisible: false,
      dialogFormData: {
        username: '',
        password: '',
        nickname: '',
        status: 1,
        mobile: '',
        avatar: '',
        introduction: '',
        roleIds: '',
        ID: '',
        mail: '',
        givenName: '',
        jobNumber: '',
        postalAddress: '',
        departments: '',
        position: '',
        departmentId: undefined
      },
      dialogFormRules: {
        username: [
          { required: true, message: '请输入用户名', trigger: 'blur' },
          { min: 2, max: 20, message: '长度在 2 到 20 个字符', trigger: 'blur' }
        ],
        password: [
          { required: false, message: '请输入密码', trigger: 'blur' },
          { min: 6, max: 30, message: '长度在 6 到 30 个字符', trigger: 'blur' }
        ],
        mail: [
          { required: true, message: '请输入邮箱', trigger: 'blur' }
        ],
        jobNumber: [
          { required: true, message: '请输入工号', trigger: 'blur' },
          { min: 0, max: 20, message: '长度在 0 到 20 个字符', trigger: 'blur' }
        ],
        nickname: [
          { required: true, message: '请输入昵称', trigger: 'blur' },
          { min: 2, max: 20, message: '长度在 2 到 20 个字符', trigger: 'blur' }
        ],
        mobile: [
          { required: true, validator: checkPhone, trigger: 'blur' }
        ],
        status: [
          { required: true, message: '请选择状态', trigger: 'change' }
        ],
        departmentId: [
          { required: true, message: '请选择部门', trigger: 'change' },
          { validator: (rule, value, callBack) => {
            if (value < 1) {
              callBack('请选择有效的部门')
            } else {
              callBack()
            }
          }
          }
        ],
        introduction: [
          { required: false, message: '说明', trigger: 'blur' },
          { min: 0, max: 100, message: '长度在 0 到 100 个字符', trigger: 'blur' }
        ]
      },

      // 删除按钮弹出框
      popoverVisible: false,
      // 表格多选
      multipleSelection: [],
      changeUserStatusFormData: {
        id: '',
        status: '',
      },
    }
  },
  created() {
    this.getTableData()
    this.getRoles()
  },
  methods: {
    // 查询
    search() {
      this.params.pageNum = 1
      this.getTableData()
    },

    // 获取表格数据
    async getTableData() {
      this.loading = true
      try {
        const { data } = await getUsers(this.params)
        data.users.forEach(item => {
          const dataStrArr = item.departmentId.split(',')
          const dataIntArr = []
          dataStrArr.forEach(item => {
            dataIntArr.push(+item)
          })
          item.departmentId = dataIntArr
        })
        this.tableData = data.users
        this.total = data.total
      } finally {
        this.loading = false
      }
    },
    // 获取所有的分组信息，用于弹框选取上级分组
    async getAllGroups() {
      this.loading = true
      try {
        const checkParams = {
          pageNum: 1,
          pageSize: 1000 // 平常百姓人家应该不会有这么多数据吧
        }
        const { data } = await getGroupTree(checkParams)
        this.departmentsOptions = [{ ID: 0, groupName: '请选择部门信息', groupType: 'T', children: data }]
      } finally {
        this.loading = false
      }
    },
    // 获取角色数据
    async getRoles() {
      const res = await getRoles(null)

      this.roles = res.data.roles
    },

    // 新增
    create() {
      this.dialogFormTitle = '新增用户'
      this.dialogType = 'create'
      this.disabled = false
      this.getAllGroups()
      this.dialogFormVisible = true
    },

    // 修改
    update(row) {
      this.disabled = true
      this.getAllGroups()
      this.dialogFormData.ID = row.ID
      this.dialogFormData.username = row.username
      this.dialogFormData.password = ''
      this.dialogFormData.nickname = row.nickname
      this.dialogFormData.status = row.status
      this.dialogFormData.mobile = row.mobile
      this.dialogFormData.introduction = row.introduction
      // 遍历角色数组，获取角色ID
      this.dialogFormData.roleIds = row.roles.map(item => item.ID)

      this.dialogFormTitle = '修改用户'
      this.dialogType = 'update'
      this.passwordType = 'password'
      this.dialogFormVisible = true

      this.dialogFormData.mail = row.mail
      this.dialogFormData.givenName = row.givenName
      this.dialogFormData.jobNumber = row.jobNumber
      this.dialogFormData.postalAddress = row.postalAddress
      this.dialogFormData.departments = row.departments
      this.dialogFormData.departmentId = row.departmentId
      this.dialogFormData.position = row.position
    },

    // 将 部门id 转换为 部门name
    setDepartmentNameByDepartmentId() {
      const ids = this.dialogFormData.departmentId
      if (!ids || !ids.length) return
      const departments = []
      // 深度优先遍函数
      const dfs = (node, cb) => {
        if (!node) return
        cb(node)
        if (node.children && node.children.length) {
          node.children.forEach(item => {
            dfs(item, cb)
          })
        }
      }
      dfs(this.departmentsOptions[0], node => {
        if (ids.includes(node.ID)) {
          departments.push(node.groupName)
        }
      })
      this.dialogFormData.departments = departments.join(',')
    },

    // 提交表单
    submitForm() {
      if (this.dialogFormData.nickname === '') {
        this.$message({
          showClose: true,
          message: '请填写昵称',
          type: 'error'
        })
        return false
      }
      if (this.dialogFormData.username === '') {
        this.$message({
          showClose: true,
          message: '请填写用户名',
          type: 'error'
        })
        return false
      }
      if (this.dialogFormData.mail === '') {
        this.$message({
          showClose: true,
          message: '请填写邮箱',
          type: 'error'
        })
        return false
      }
      if (this.dialogFormData.jobNumber === '') {
        this.$message({
          showClose: true,
          message: '请填写工号',
          type: 'error'
        })
        return false
      }
      if (this.dialogFormData.mobile === '') {
        this.$message({
          showClose: true,
          message: '请填写手机号',
          type: 'error'
        })
        return false
      }
      if (this.dialogFormData.status === '') {
        this.$message({
          showClose: true,
          message: '请填写状态',
          type: 'error'
        })
        return false
      }
      if (this.dialogFormData.roleIds === '') {
        this.$message({
          showClose: true,
          message: '请选择角色列表',
          type: 'error'
        })
        return false
      }
      this.$refs['dialogForm'].validate(async valid => {
        if (valid) {
          this.submitLoading = true
          // 在这里自动填充下部门字段
          this.setDepartmentNameByDepartmentId()
          this.dialogFormDataCopy = { ...this.dialogFormData }
          if (this.dialogFormData.password !== '') {
          // 密码RSA加密处理
            const encryptor = new JSEncrypt()
            // 设置公钥
            encryptor.setPublicKey(this.publicKey)
            // 加密密码
            const encPassword = encryptor.encrypt(this.dialogFormData.password)
            this.dialogFormDataCopy.password = encPassword
          }
          let message = ''
          try {
            if (this.dialogType === 'create') {
              const { msg } = await createUser(this.dialogFormDataCopy)

              message = msg
            } else {
              const { msg } = await updateUserById(this.dialogFormDataCopy)
              message = msg
            }
          } finally {
            this.submitLoading = false
          }

          this.resetForm()
          this.getTableData()
          this.$message({
            showClose: true,
            message: message,
            type: 'success'
          })
        } else {
          this.$message({
            showClose: true,
            message: '表单校验失败',
            type: 'error'
          })
          return false
        }
      })
    },

    // 提交表单
    cancelForm() {
      this.resetForm()
    },

    resetForm() {
      this.dialogFormVisible = false
      this.$refs['dialogForm'].resetFields()
      this.dialogFormData = {
        username: '',
        password: '',
        nickname: '',
        status: 1,
        mobile: '',
        avatar: '',
        introduction: '',
        roleIds: '',
        departments: '',
        position: '',
        departmentId: undefined
      }
    },

    // 批量删除
    batchDelete() {
      this.$confirm('此操作将永久删除, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(async res => {
        this.loading = true
        const userIds = []
        this.multipleSelection.forEach(x => {
          userIds.push(x.ID)
        })
        let msg = ''
        try {
          const { message } = await batchDeleteUserByIds({ userIds: userIds })
          msg = message
        } finally {
          this.loading = false
        }

        this.getTableData()
        this.$message({
          showClose: true,
          message: msg,
          type: 'success'
        })
      }).catch(() => {
        this.$message({
          showClose: true,
          type: 'info',
          message: '已取消删除'
        })
      })
    },

    // 监听 switch 开关 状态改变
   async userStateChanged(userInfo) {
      this.changeUserStatusFormData.id = userInfo.ID
      this.changeUserStatusFormData.status = userInfo.status

      const { code } = await changeUserStatus(this.changeUserStatusFormData)

      if (code !== 0) {
        userInfo.status = !userInfo.status
        return this.$message.error('更新用户状态失败')
      }
      this.$message.success('更新用户状态成功')
    },

    // 表格多选
    handleSelectionChange(val) {
      this.multipleSelection = val
    },

    // 单个删除
    async singleDelete(Id) {
      this.loading = true
      let msg = ''
      try {
        const { message } = await batchDeleteUserByIds({ userIds: [Id] })
        msg = message
      } finally {
        this.loading = false
      }

      this.getTableData()
      this.$message({
        showClose: true,
        message: msg,
        type: 'success'
      })
    },

    showPwd() {
      if (this.passwordType === 'password') {
        this.passwordType = ''
      } else {
        this.passwordType = 'password'
      }
    },

    // 分页
    handleSizeChange(val) {
      this.params.pageSize = val
      this.getTableData()
    },
    handleCurrentChange(val) {
      this.params.pageNum = val
      this.getTableData()
    },
    // treeselect
    normalizer(node) {
      return {
        id: node.ID,
        label: node.groupType + '=' + node.groupName,
        isDisabled: node.groupType === 'ou',
        children: node.children
      }
    },
    treeselectInput(value) {
      this.treeselectValue = value
    },
    syncDingTalkUsers(obj) {
      this.loading = true
      syncDingTalkUsersApi().then(res => {
        this.loading = false
        this.$message({
          showClose: true,
          message: res.message,
          type: 'success'
        })
      })
      this.getTableData()
      this.loading = false
    },
    syncWeComUsers(obj) {
      this.loading = true
      syncWeComUsersApi().then(res => {
        this.loading = false
        this.$message({
          showClose: true,
          message: res.message,
          type: 'success'
        })
      })
      this.getTableData()
      this.loading = false
    },
    syncFeiShuUsers(obj) {
      this.loading = true
      syncFeiShuUsersApi().then(res => {
        this.loading = false
        this.$message({
          showClose: true,
          message: res.message,
          type: 'success'
        })
      })
      this.getTableData()
      this.loading = false
    },
    syncOpenLdapUsers(obj) {
      this.loading = true
      syncOpenLdapUsersApi().then(res => {
        this.loading = false
        this.$message({
          showClose: true,
          message: res.message,
          type: 'success'
        })
      })
      this.getTableData()
      this.loading = false
    }
  }
}
</script>

<style scoped>
  .container-card{
    margin: 10px;
  }

  .delete-popover{
    margin-left: 10px;
  }

  .show-pwd {
    position: absolute;
    right: 10px;
    top: 3px;
    font-size: 16px;
    color: #889aa4;
    cursor: pointer;
    user-select: none;
  }
</style>
