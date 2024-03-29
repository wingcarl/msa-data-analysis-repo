<template>
  <el-form
    ref="formRef"
    v-loading="formLoading"
    :model="formData"
    :rules="formRules"
    label-width="80px"
  >
    <el-form-item label="请假类型" prop="type">
      <el-select v-model="formData.type" clearable placeholder="请选择请假类型">
        <el-option
          v-for="dict in getIntDictOptions(DICT_TYPE.BPM_OA_LEAVE_TYPE)"
          :key="dict.value"
          :label="dict.label"
          :value="dict.value"
        />
      </el-select>
    </el-form-item>
    <el-form-item label="开始时间" prop="startTime">
      <el-date-picker
        v-model="formData.startTime"
        clearable
        placeholder="请选择开始时间"
        type="datetime"
        value-format="x"
      />
    </el-form-item>
    <el-form-item label="结束时间" prop="endTime">
      <el-date-picker
        v-model="formData.endTime"
        clearable
        placeholder="请选择结束时间"
        type="datetime"
        value-format="x"
      />
    </el-form-item>
    <el-form-item label="原因" prop="reason">
      <el-input v-model="formData.reason" placeholder="请输请假原因" type="textarea" />
    </el-form-item>
    <el-form-item>
      <el-button :disabled="formLoading" type="primary" @click="submitForm">确 定</el-button>
    </el-form-item>
  </el-form>
</template>
<script lang="ts" setup>
import { DICT_TYPE, getIntDictOptions } from '@/utils/dict'
import * as LeaveApi from '@/api/bpm/leave'
import { useTagsViewStore } from '@/store/modules/tagsView'

defineOptions({ name: 'BpmOALeaveCreate' })

const message = useMessage() // 消息弹窗
const formLoading = ref(false) // 表单的加载中：1）修改时的数据加载；2）提交的按钮禁用
const formData = ref({
  type: undefined,
  reason: undefined,
  startTime: undefined,
  endTime: undefined
})
const formRules = reactive({
  type: [{ required: true, message: '请假类型不能为空', trigger: 'blur' }],
  reason: [{ required: true, message: '请假原因不能为空', trigger: 'change' }],
  startTime: [{ required: true, message: '请假开始时间不能为空', trigger: 'change' }],
  endTime: [{ required: true, message: '请假结束时间不能为空', trigger: 'change' }]
})
const formRef = ref() // 表单 Ref
const { delView } = useTagsViewStore() // 视图操作
const { push, currentRoute } = useRouter() // 路由
/** 提交表单 */
const submitForm = async () => {
  // 校验表单
  if (!formRef) return
  const valid = await formRef.value.validate()
  if (!valid) return
  // 提交请求
  formLoading.value = true
  try {
    const data = formData.value as unknown as LeaveApi.LeaveVO
    await LeaveApi.createLeave(data)
    message.success('发起成功')
    // 关闭当前 Tab
    delView(unref(currentRoute))
    await push({ name: 'BpmOALeave' })
  } finally {
    formLoading.value = false
  }
}
</script>
