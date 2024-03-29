<!-- 跟进记录的添加表单弹窗 -->
<template>
  <Dialog v-model="dialogVisible" :title="dialogTitle" width="50%">
    <el-form
      ref="formRef"
      v-loading="formLoading"
      :model="formData"
      :rules="formRules"
      label-width="120px"
    >
      <el-row>
        <el-col :span="12">
          <el-form-item label="跟进类型" prop="type">
            <el-select v-model="formData.type" placeholder="请选择跟进类型">
              <el-option
                v-for="dict in getIntDictOptions(DICT_TYPE.CRM_FOLLOW_UP_TYPE)"
                :key="dict.value"
                :label="dict.label"
                :value="dict.value"
              />
            </el-select>
          </el-form-item>
        </el-col>
        <el-col :span="12">
          <el-form-item label="下次联系时间" prop="nextTime">
            <el-date-picker
              v-model="formData.nextTime"
              placeholder="选择下次联系时间"
              type="date"
              value-format="x"
            />
          </el-form-item>
        </el-col>
        <!-- TODO @puhui999：不搞富文本哈；然后加个附件、图片两个 form-item 哈 -->
        <el-col :span="24">
          <el-form-item label="跟进内容" prop="content">
            <Editor v-model="formData.content" height="300px" />
          </el-form-item>
        </el-col>
        <!-- TODO @puhui999：因为不考虑编辑的情况，是不是关联要是个弹窗选择哈？ -->
        <el-col :span="24">
          <el-form-item label="关联联系人" prop="contactIds">
            <el-select v-model="formData.contactIds" multiple placeholder="请选择">
              <el-option
                v-for="item in allContactList"
                :key="item.id"
                :label="item.name"
                :value="item.id"
              />
            </el-select>
            <contact-list v-model:contactIds="formData.contactIds" />
          </el-form-item>
          <!--          <el-form-item label="关联联系人" prop="contactIds">-->
          <!--            <el-button @click="handleAddContact">-->
          <!--              <Icon class="mr-5px" icon="ep:plus" />-->
          <!--              选择添加联系人-->
          <!--            </el-button>-->
          <!--            <contact-list v-model:contactIds="formData.contactIds" />-->
          <!--          </el-form-item>-->
        </el-col>
        <el-col :span="24">
          <el-form-item label="关联商机" prop="businessIds">
            <el-select v-model="formData.businessIds" multiple placeholder="请选择">
              <el-option
                v-for="item in allBusinessList"
                :key="item.id"
                :label="item.name"
                :value="item.id"
              />
            </el-select>
            <business-list v-model:businessIds="formData.businessIds" />
          </el-form-item>
          <!--          <el-form-item label="关联商机" prop="businessIds">-->
          <!--            <el-button @click="handleAddBusiness">-->
          <!--              <Icon class="mr-5px" icon="ep:plus" />-->
          <!--              选择添加商机-->
          <!--            </el-button>-->
          <!--            <business-list v-model:businessIds="formData.businessIds" />-->
          <!--          </el-form-item>-->
        </el-col>
      </el-row>
    </el-form>
    <template #footer>
      <el-button :disabled="formLoading" type="primary" @click="submitForm">确 定</el-button>
      <el-button @click="dialogVisible = false">取 消</el-button>
    </template>
  </Dialog>
</template>
<script lang="ts" setup>
import { DICT_TYPE, getIntDictOptions } from '@/utils/dict'
import { FollowUpRecordApi, FollowUpRecordVO } from '@/api/crm/followup'
import { BusinessList, ContactList } from './components'
import * as ContactApi from '@/api/crm/contact'
import * as BusinessApi from '@/api/crm/business'

defineOptions({ name: 'FollowUpRecordForm' })

const { t } = useI18n() // 国际化
const message = useMessage() // 消息弹窗

const dialogVisible = ref(false) // 弹窗的是否展示
const dialogTitle = ref('') // 弹窗的标题
const formLoading = ref(false) // 表单的加载中：1）修改时的数据加载；2）提交的按钮禁用
const formType = ref('') // 表单的类型：create - 新增；update - 修改
const formData = ref<FollowUpRecordVO>({} as FollowUpRecordVO)
const formRules = reactive({
  type: [{ required: true, message: '跟进类型不能为空', trigger: 'change' }],
  content: [{ required: true, message: '跟进内容不能为空', trigger: 'blur' }],
  nextTime: [{ required: true, message: '下次联系时间不能为空', trigger: 'blur' }]
})

const formRef = ref() // 表单 Ref
const allContactList = ref<ContactApi.ContactVO[]>([]) // 所有联系人列表
const allBusinessList = ref<BusinessApi.BusinessVO[]>([]) // 所有商家列表

/** 打开弹窗 */
const open = async (bizType: number, bizId: number, type: string, id?: number) => {
  dialogVisible.value = true
  dialogTitle.value = t('action.' + type)
  formType.value = type
  resetForm()
  formData.value.bizType = bizType
  formData.value.bizId = bizId
  allContactList.value = await ContactApi.getSimpleContactList()
  allBusinessList.value = await BusinessApi.getSimpleBusinessList()
  // 修改时，设置数据
  if (id) {
    formLoading.value = true
    try {
      formData.value = await FollowUpRecordApi.getFollowUpRecord(id)
    } finally {
      formLoading.value = false
    }
  }
}
defineExpose({ open }) // 提供 open 方法，用于打开弹窗

/** 提交表单 */
const emit = defineEmits(['success']) // 定义 success 事件，用于操作成功后的回调
const submitForm = async () => {
  // 校验表单
  await formRef.value.validate()
  // 提交请求
  formLoading.value = true
  try {
    const data = formData.value as unknown as FollowUpRecordVO
    if (formType.value === 'create') {
      await FollowUpRecordApi.createFollowUpRecord(data)
      message.success(t('common.createSuccess'))
    } else {
      await FollowUpRecordApi.updateFollowUpRecord(data)
      message.success(t('common.updateSuccess'))
    }
    dialogVisible.value = false
    // 发送操作成功的事件
    emit('success')
  } finally {
    formLoading.value = false
  }
}

/** 重置表单 */
const resetForm = () => {
  formRef.value?.resetFields()
  formData.value = {} as FollowUpRecordVO
}
</script>
