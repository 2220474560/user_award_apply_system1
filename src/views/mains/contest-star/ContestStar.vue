<!--
    * @FileDescription: 竞赛之星报名页面。
    * @Author: 钟胡琴
    * @Date: 2024年1月22日
    * @LastEditors: 李思佳
    * @LastEditTime: 2024年1月22日
-->
<template>
  <div class="contest_main">
    <div class="describe">
      <span class="describe_content">评选说明:</span><br />
      <span class="describe_content">报名参加竞赛达3次以上</span>
    </div>
    <a-form
      ref="formRef"
      :model="formState"
      :rules="rules"
      :label-col="labelCol"
      :wrapper-col="wrapperCol"
      hideRequiredMark
      class="form"
    >
      <a-form-item ref="name" label="参与竞赛名称" class="custom-label" name="name">
        <a-input v-model:value="formState.name" placeholder="请输入内容" />
      </a-form-item>
      <a-form-item label="竞赛报名时间" name="date1">
        <a-config-provider :locale="locale">
          <a-date-picker
            v-model:value="formState.date1"
            type="date"
            placeholder="请选择日期"
            style="width: 100%"
            :locale="locale"
          />
        </a-config-provider>
      </a-form-item>
      <a-form-item label="佐证材料">
        <a-form-item name="dragger" no-style>
          <a-upload-dragger
            v-model="formState.dragger"
            name="file"
            :max-count="1"
            :file-list="fileList"
            :action="ossUploadUrl"
            :headers="headers"
            :beforeUpload="beforeUpload"
            @change="handleChange"
            @remove="handleRemove"
          >
            <p class="ant-upload-drag-icon">
              <InboxOutlined />
            </p>
            <p class="ant-upload-text">将文件拖到此处或点击上传</p>
          </a-upload-dragger>
        </a-form-item>
      </a-form-item>
      <a-form-item :wrapper-col="{ span: 15, offset: 12 }">
        <a-button type="primary" @click="onSubmit">提交</a-button>
      </a-form-item>
    </a-form>
  </div>
</template>

<script setup lang="ts">
import { reactive, ref } from 'vue'
import type { UnwrapRef } from 'vue'
import type { Rule } from 'ant-design-vue/es/form'
import { InboxOutlined } from '@ant-design/icons-vue'
import { message } from 'ant-design-vue'
import cssAnimation from 'ant-design-vue/es/_util/css-animation'
import style = cssAnimation.style
import { ContestRequest } from '@/service/mains/contest-star/contest-star'
import { BASE_URL } from '@/service/config'
import { format } from 'date-fns'
import dayjs from 'dayjs'
import 'dayjs/locale/zh-cn'
import locale from 'ant-design-vue/es/date-picker/locale/zh_CN'
import type { UploadChangeParam, UploadFile, UploadProps } from 'ant-design-vue/es/upload'

dayjs.locale('zh-cn')
interface FormState {
  name: string
  date1: string
  url: string
  dragger: string
}
const formRef = ref()
const labelCol = { span: 9 }
const wrapperCol = { span: 8 }
const formState: UnwrapRef<FormState> = reactive({
  name: '',
  date1: '',
  url: '',
  dragger: ''
})
const token = localStorage.getItem('access_Token')
const headers = {
  Authorization: 'Bearer ' + token
}
const rules: Record<string, Rule[]> = {
  name: [{ required: true, message: '请填写竞赛名称', trigger: 'change' }],
  date1: [{ required: true, message: '请填写竞赛报名时间', trigger: 'change', type: 'object' }],
  dragger: [{ required: true, message: '请上传佐证材料', trigger: 'change' }]
}
// 时间戳转换
const formatDate = (timestamp, formatStr = 'yyyy-MM-dd') => {
  return format(new Date(timestamp), formatStr)
}

const fileList = ref<UploadProps['fileList'] | null>(null)!
// 移除已上传的文件
const handleRemove = (file: UploadChangeParam) => {
  if (!fileList.value) {
    fileList.value = []
  }
  const index = fileList.value.findIndex((item: UploadFile | any) => item.url === file)
  if (index !== -1) {
    fileList.value.splice(index, 1)
    formState.url = ''
    fileList.value = []
  }
}

/**
 * @description 提交表单
 */
async function onSubmit() {
  // 检查表单是否填写完整
  if (!formState.name || !formState.date1 || !formState.url) {
    message.error('请填写完整表单')
    return
  }
  // 将字符串日期转换为日期对象
  const dateObject = new Date(formState.date1)
  // 将日期对象转换为格式化后的日期字符串
  const signuptime = formatDate(dateObject.getTime(), 'yyyy-MM-dd')
  // 创建符合期望类型的对象
  const requestData = {
    entryname: formState.name,
    url: formState.url,
    signuptime: signuptime
  }
  try {
    // 调用 ContestRequest 函数
    const response = await ContestRequest(requestData)
    console.log(response)

    // 在接口请求成功后进行提示
    message.success('提交成功')
    formState.name = ''
    // 清空佐证材料
    fileList.value = []
    formState.dragger = '' // 清空文件路径
    formState.date1 = ''
  } catch (error) {
    // 在接口请求失败时进行提示
    message.error('提交失败')
  }
}

// 上传PDF地址
const ossUploadUrl = BASE_URL + 'api/stu/OssUpdate'
// 判断只能上传PDF文件
const beforeUpload = (file: any) => {
  const isPDF = file.type === 'application/pdf'
  const maxFileSize = 10 * 1024 * 1024

  if (!isPDF) {
    message.error('只能上传 PDF 文件！')
  } else if (file.size > maxFileSize) {
    message.error('文件大小超过限制10MB！')
  } else {
    // message.success('PDF 文件上传成功！');
  }

  return isPDF && file.size <= maxFileSize
}

/**
 * @description 上传PDF状态
 */
const handleChange = (info: UploadChangeParam) => {
  // const status = info.file.status
  // if (status !== 'uploading') {
  //   // console.log(info.file, info.fileList)
  //   message.success(`${info.file.name} 文件上传中，请稍候.`)
  // }
  // if (status === 'done') {
  //   const fileurl = info.file.response.data
  //   formState.dragger = fileurl
  //   message.success(`${info.file.name} 文件上传成功.`)
  // } else if (status === 'error') {
  //   message.error(`${info.file.name} 文件上传失败.`)
  // }
  if (!fileList.value) {
    fileList.value = []
  }
  let resFileList = [...info.fileList]
  resFileList = resFileList.slice(-1)
  resFileList = resFileList.map((file) => {
    const newFile = {
      uid: file.uid,
      name: file.name || 'Untitled',
      url: file.response ? file.response.data : file.url
    }
    return newFile
  })
  fileList.value = resFileList

  // 打印上传文件的响应信息
  if (info.file.status === 'done' && info.file.response) {
    message.success(`${info.file.name} 文件上传成功！.`)
    // console.log(info.file.response.data)
    formState.url = info.file.response.data
  }
}
</script>

<style scoped>
.describe {
  width: 100%;
  background-color: rgb(255, 255, 255);
  border-radius: 16px;
  padding: 10px;
  overflow: hidden;
}
.describe_content {
  line-height: normal;
  color: #02509b;
}
.form {
  width: 90%;
  margin-top: 40px;
}
:deep(:where(.css-dev-only-do-not-override-19yxfbp).ant-form-item .ant-form-item-label > label) {
  color: #02509b;
}
</style>
