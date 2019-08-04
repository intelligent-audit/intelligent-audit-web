<template>
  <el-dialog
    :title="!dataForm.id ? '新增' : '基本信息'"
    :close-on-click-modal="false"
    :visible.sync="visible">
    <el-form :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()" label-width="80px">
      <el-form-item label="姓名" prop="userName">
        <el-input v-model="dataForm.userName" placeholder="请输入登录平台的账号，推荐使用英文" :disabled="isDisabled"></el-input>
      </el-form-item>
      <el-form-item label="密码" prop="password" :class="{ 'is-required': !dataForm.id }" v-if="!dataForm.id">
        <el-input v-model="dataForm.password" type="password" placeholder="请设置登录密码"></el-input>
      </el-form-item>
      <el-form-item label="确认密码" prop="comfirmPassword" :class="{ 'is-required': !dataForm.id }" v-if="!dataForm.id">
        <el-input v-model="dataForm.comfirmPassword" type="password" placeholder="请再次输入你的密码"></el-input>
      </el-form-item>
      <el-form-item label="角色" prop="roleId">
        <el-select v-model="dataForm.roleId" style="width:100%;" :disabled="isDisabled">
          <el-option v-for="role in roleList" :key="role.roleId" :label="role.roleName" :value="role.roleId"></el-option>
        </el-select>
      </el-form-item>
      <el-form-item label="联系方式" prop="mobile">
        <el-input v-model="dataForm.mobile" placeholder="请输入联系方式" :disabled="isDisabled"></el-input>
      </el-form-item>
      <el-form-item label="EMAIL" prop="email">
        <el-input v-model="dataForm.email" placeholder="请输入email" :disabled="isDisabled"></el-input>
      </el-form-item>
      <el-form-item label="创建时间" prop="" v-if="dataForm.id">
        <el-input v-model="dataForm.createTime" placeholder="" :disabled="isDisabled"></el-input>
      </el-form-item>
      <el-form-item label="录入者" prop="" v-if="dataForm.id">
        <el-input v-model="dataForm.createUser" placeholder="" :disabled="isDisabled"></el-input>
      </el-form-item>
      <!-- 
      <el-form-item label="状态" size="mini" prop="status">
        <el-radio-group v-model="dataForm.status">
          <el-radio :label="0">禁用</el-radio>
          <el-radio :label="1">正常</el-radio>
        </el-radio-group>
      </el-form-item> -->
    </el-form>
    <span slot="footer" class="dialog-footer">
      <div v-if="!dataForm.id">
        <el-button @click="visible = false">取消</el-button>
        <el-button type="primary" @click="dataFormSubmit()">确定</el-button>
      </div>
      <div v-else>
        <el-button type="primary" @click="dataFormSubmit()">编辑</el-button>
      </div>
    </span>
  </el-dialog>
</template>

<script>
  import { isEmail, isMobile } from '@/utils/validate'
  export default {
    data () {
      var validatePassword = (rule, value, callback) => {
        if (!this.dataForm.id && !/\S/.test(value)) {
          callback(new Error('密码不能为空'))
        } else {
          callback()
        }
      }
      var validateComfirmPassword = (rule, value, callback) => {
        if (!this.dataForm.id && !/\S/.test(value)) {
          callback(new Error('确认密码不能为空'))
        } else if (this.dataForm.password !== value) {
          callback(new Error('确认密码与密码输入不一致'))
        } else {
          callback()
        }
      }
      var validateEmail = (rule, value, callback) => {
        if (!isEmail(value)) {
          callback(new Error('邮箱格式错误'))
        } else {
          callback()
        }
      }
      var validateMobile = (rule, value, callback) => {
        if (!isMobile(value)) {
          callback(new Error('手机号格式错误'))
        } else {
          callback()
        }
      }
      return {
        visible: false,
        isDisabled: false,
        roleList: [
          {
            roleId: 1,
            roleName: "审计人员"
          },{
            roleId: 2,
            roleName: "系统管理员"
          },{
            roleId: 3,
            roleName: "第三方审计人员"
          },{
            roleId: 4,
            roleName: "EIM业务人员"
          },{
            roleId: 5,
            roleName: "领导"
          }
        ],
        dataForm: {
          id: 0,
          userName: '',
          password: '',
          comfirmPassword: '',
          email: '',
          mobile: '',
          roleId: 1,
          createTime: '',
          createUser: '',
          status: 1
        },
        dataRule: {
          userName: [
            { required: true, message: '用户名不能为空', trigger: 'blur' }
          ],
          password: [
            { validator: validatePassword, trigger: 'blur' }
          ],
          comfirmPassword: [
            { validator: validateComfirmPassword, trigger: 'blur' }
          ],
          email: [
            { required: true, message: '邮箱不能为空', trigger: 'blur' },
            { validator: validateEmail, trigger: 'blur' }
          ],
          mobile: [
            { required: true, message: '手机号不能为空', trigger: 'blur' },
            { validator: validateMobile, trigger: 'blur' }
          ]
        }
      }
    },
    methods: {
      init (id) {
        this.dataForm.id = id
        this.visible = true
        if (this.dataForm.id) {
            this.isDisabled = true
            this.$http({
              url: this.$http.adornUrl(`/sys/user/info/${this.dataForm.id}`),
              method: 'get',
              params: this.$http.adornParams()
            }).then(({data}) => {
              if (data && data.code === 0) {
                this.dataForm.userName = data.user.username
                this.dataForm.email = data.user.email
                this.dataForm.mobile = data.user.mobile
                this.dataForm.roleId = parseInt(data.user.roleId)
                this.dataForm.status = data.user.status
                this.dataForm.createTime = data.user.createTime
                this.dataForm.createUser = data.user.createUser
              }
            })
          }else{
            this.isDisabled = false
            this.dataForm.userName = ''
            this.dataForm.email = ''
            this.dataForm.mobile = ''
            this.dataForm.roleId = 1
            this.dataForm.status = 1
            this.dataForm.createTime = ''
            this.dataForm.createUser = ''
          }
      },
      // 表单提交
      dataFormSubmit () {
        this.$refs['dataForm'].validate((valid) => {
          if (valid) {
            this.$http({
              url: this.$http.adornUrl(`/sys/user/${!this.dataForm.id ? 'save' : 'update'}`),
              method: 'post',
              data: this.$http.adornData({
                'userId': this.dataForm.id || undefined,
                'username': this.dataForm.userName,
                'password': this.dataForm.password,
                'email': this.dataForm.email,
                'mobile': this.dataForm.mobile,
                'status': this.dataForm.status,
                'roleIdList': this.dataForm.roleIdList
              })
            }).then(({data}) => {
              if (data && data.code === 0) {
                this.$message({
                  message: '操作成功',
                  type: 'success',
                  duration: 1500,
                  onClose: () => {
                    this.visible = false
                    this.$emit('refreshDataList')
                  }
                })
              } else {
                this.$message.error(data.msg)
              }
            })
          }
        })
      }
    }
  }
</script>
