<!--
 * @author: gaoweixuan
 * @since: 2023-02-25
-->

<!-- 流程分类添加修改弹出框 -->
<script lang="ts" setup>
import { ref } from 'vue'
import { ElMessage } from 'element-plus'
import { addCategory, getCategory, editCategory, checkCategoryCode } from '@/api/bpm/def/category'
import type { CategoryForm } from '@/api/bpm/def/category/type.ts'
import { useI18n } from 'vue-i18n'
import JSONBigInt from 'json-bigint'
import useUserStore from '@/store/modules/user.ts'
import useWidth from '@/hooks/dialogWidth'

defineOptions({
  name: 'CategoryAddOrEdit',
  inheritAttrs: false,
})

const { t } = useI18n()
const $emit = defineEmits(['reloadDataList'])
const visible = ref<boolean>(false)
const loading = ref<boolean>(false)
const categoryDataFormRef = ref()
const categoryDataForm = ref<CategoryForm>({ categoryCode: '', categoryName: '', tenantId: '' })
const rules = ref({
  categoryName: [
    {
      required: true,
      message: t('common.placeholder.enter') + t('category.fields.categoryName'),
      trigger: 'blur',
    },
  ],
  categoryCode: [
    {
      required: true,
      message: t('common.placeholder.enter') + t('category.fields.categoryCode'),
      trigger: 'blur',
    },
    {
      validator: (rule: any, value: any, callback: any) => {
        checkCategoryCode(
          value,
          !categoryDataForm.value.id ? undefined : JSONBigInt.parse(categoryDataForm.value.id),
        ).then((response: any) => {
          if (response.data) {
            callback()
            return
          }
          callback(new Error(t('category.rules.categoryCodeDuplicate')))
        })
      },
      trigger: 'blur',
    },
  ],
})

/**
 * 初始化
 *
 * @param id
 */
const init = async (id: number) => {
  categoryDataForm.value.id = undefined
  visible.value = true
  // 重置表单数据
  if (categoryDataFormRef.value) {
    categoryDataFormRef.value.resetFields()
  }

  if (id) {
    await getInfo(id)
  }
}

/**
 * 获取信息
 *
 * @param id
 */
const getInfo = async (id: number) => {
  const response: any = await getCategory(JSONBigInt.parse(id))
  if (response.code === '0000') {
    Object.assign(categoryDataForm.value, response.data)
  }
}

/**
 * 表单提交
 */
const handleDataFormSubmit = () => {
  categoryDataFormRef.value.validate(async (valid: boolean) => {
    if (!valid) {
      return false
    }
    categoryDataForm.value.tenantId = useUserStore().userInfo.tenantId.toString()
    loading.value = true
    const id = categoryDataForm.value.id
    if (id) {
      await editCategory(id, categoryDataForm.value)
      ElMessage.success({
        message: `${t('common.modify') + t('common.success')}`,
        duration: 1000,
        onClose: () => {
          visible.value = false
          loading.value = false
          $emit('reloadDataList')
        },
      })
    } else {
      await addCategory(categoryDataForm.value)
      ElMessage.success({
        message: `${t('common.save') + t('common.success')}`,
        duration: 1000,
        onClose: () => {
          visible.value = false
          loading.value = false
          $emit('reloadDataList')
        },
      })
    }
  })
}

defineExpose({
  init,
})
</script>

<template>
  <el-dialog
    v-model="visible"
    :width="useWidth()"
    :title="!categoryDataForm.id ? t('common.add') : t('common.edit')"
    :close-on-click-modal="false"
    :close-on-press-escape="false"
  >
    <el-form
      :model="categoryDataForm"
      :rules="rules"
      ref="categoryDataFormRef"
      @keyup.enter="handleDataFormSubmit()"
      label-width="90px"
    >
      <el-form-item :label="t('category.fields.categoryCode')" prop="categoryCode">
        <el-input
          v-model="categoryDataForm.categoryCode"
          autocomplete="off"
          clearable
          :placeholder="t('common.placeholder.enter') + t('category.fields.categoryCode')"
        />
      </el-form-item>
      <el-form-item :label="t('category.fields.categoryName')" prop="categoryName">
        <el-input
          v-model="categoryDataForm.categoryName"
          autocomplete="off"
          clearable
          :placeholder="t('common.placeholder.enter') + t('category.fields.categoryName')"
        />
      </el-form-item>
    </el-form>
    <template #footer>
      <el-button
        @click="
          () => {
            visible = false
            loading = false
          }
        "
      >
        {{ t('common.cancel') }}
      </el-button>
      <el-button type="primary" :loading="loading" @click="handleDataFormSubmit()">{{ t('common.confirm') }}</el-button>
    </template>
  </el-dialog>
</template>
