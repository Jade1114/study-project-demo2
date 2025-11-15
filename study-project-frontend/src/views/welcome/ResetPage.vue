<script setup>
import {computed, reactive, ref} from "vue";
import {EditPen, Lock, Message} from "@element-plus/icons-vue";
import {get, post} from "@/net/index.js";
import router from "@/router/index.js";
import {ElMessage} from "element-plus";

const active = ref(0)
const formRef = ref()
const coldTime = ref(0)


const form = reactive({
  email: '',
  code: '',
  password: '',
  password_repeat: '',
})

const validatePassword = (rule, value, callback) => {
  if (value === '') {
    callback(new Error('请再次输入密码'))
  } else if (value !== form.password) {
    callback(new Error('两次输入的密码不一致'))
  } else {
    callback()
  }
}

const rules = {
  password: [
    {required: true, message: '请输入密码', trigger: 'blur'},
    {min: 6, max: 16, message: '密码的长度必须在6-16个字符之间', trigger: ['blur', 'change']},
  ],
  password_repeat: [
    {validator: validatePassword, trigger: ['blur', 'change']}
  ],
  email: [
    {required: true, message: '请输入电子邮件', trigger: ['blur']},
    {
      type: 'email',
      message: '请输入合法的电子邮件地址', trigger: ['blur', 'change']
    }
  ],
  code: [
    {required: true, message: '请输入验证码', trigger: ['blur']}
  ]
}

function askCode() {
  if (isEmailValid) {
    coldTime.value = 60
    get(`/api/auth/ask-code?email=${form.email}&type=reset`, () => {
      ElMessage.success(`验证码已发送到邮箱:${form.email},请注意查收`)
      setInterval(() => coldTime.value--, 1000);
    }, (message) => {
      ElMessage.error(message)
      coldTime.value = 0
    })
  } else {
    ElMessage.warning('请输入正确的电子邮件!')
  }
}

const isEmailValid = computed(() => {
  return /^[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Za-z]{2,}$/.test(form.email)
})

function confirmReset() {
  formRef.value.validate((valid) => {
    if (valid) {
      post('/api/auth/reset-confirm', {
        email: form.email,
        code: form.code,
      }, () => active.value++)
    }
  })
}

function doReset() {
  formRef.value.validate((valid) => {
    if (valid) {
      post('/api/auth/reset-password', {
        ...form
      }, () => {
        ElMessage.success('密码重置成功,请重新登录')
        router.push('/')
      })
    }
  })
}


</script>

<template>
  <div>
    <div style="margin: 30px 20px;width: 100%">
      <el-steps :active="active" finish-status="success" align-center>
        <el-step title="验证电子邮件"/>
        <el-step title="重新设定密码"/>
      </el-steps>
    </div>
    <div style="text-align:center;margin: 0 20px;height: 100%" v-if="active === 0">
      <div style="text-align: center;margin-top: 80px">
        <div style="font-size: 25px;font-weight: bold">重置密码</div>
        <div style="font-size: 14px;color: grey">请输入需要重置密码的邮件地址</div>
      </div>
      <div style="margin-top: 50px">
        <el-form :model="form" :rules="rules" ref="formRef">
          <el-form-item prop="email">
            <el-input v-model="form.email" type="text" placeholder="电子邮箱地址">
              <template #prefix>
                <el-icon>
                  <Message/>
                </el-icon>
              </template>
            </el-input>
          </el-form-item>
          <el-form-item prop="code" style="width: 100%">
            <el-row :gutter="10">
              <el-col :span="18">
                <el-input v-model="form.code" :maxlength="6" type="text" placeholder="请输入验证码">
                  <template #prefix>
                    <el-icon>
                      <EditPen/>
                    </el-icon>
                  </template>
                </el-input>
              </el-col>
              <el-col :span="6">
                <el-button type="success" plain :disabled="!isEmailValid || coldTime > 0" @click="askCode">
                  {{ coldTime > 0 ? '请稍后' + coldTime + '秒' : '获取验证码' }}
                </el-button>
              </el-col>
            </el-row>
          </el-form-item>
        </el-form>
      </div>
      <div style="margin-top: 70px">
        <el-button @click="confirmReset()" style="width: 270px;" type="danger" plain>验证</el-button>
        <el-button @click="router.push('/')" style="width: 270px;" type="danger" plain>返回</el-button>
      </div>
    </div>
    <div style="text-align:center;margin: 0 20px;height: 100%" v-if="active === 1">
      <div style="text-align: center;margin-top: 80px">
        <div style="font-size: 25px;font-weight: bold">重置密码</div>
        <div style="font-size: 14px;color: grey">请输入您的新密码，务必牢记，避免丢失</div>
      </div>
      <div style="margin-top: 50px">
        <el-form :model="form" :rules="rules"  ref="formRef">
          <el-form-item prop="password">
            <el-input v-model="form.password" :maxlength="20" type="password" placeholder="新密码">
              <template #prefix>
                <el-icon>
                  <Lock/>
                </el-icon>
              </template>
            </el-input>
          </el-form-item>
          <el-form-item prop="password_repeat">
            <el-input v-model="form.password_repeat" :maxlength="20" type="password" placeholder="重复新密码">
              <template #prefix>
                <el-icon>
                  <Lock/>
                </el-icon>
              </template>
            </el-input>
          </el-form-item>
          <el-form-item prop="code">
            <template #prefix>
              <el-icon>
                <EditPen/>
              </el-icon>
            </template>
          </el-form-item>
        </el-form>
      </div>
      <div style="margin-top: 80px">
        <el-button @click="doReset()" style="width: 270px; " type="danger" plain>重置密码</el-button>

        <el-button @click="router.push('/')" style="width: 270px; margin-top: 20px" type="danger" plain>返回
        </el-button>

      </div>
    </div>
  </div>
</template>
<style scoped>

</style>