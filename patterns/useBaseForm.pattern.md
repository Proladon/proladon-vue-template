# useBaseForm Pattern

This pattern demonstrates how to utilize the `useBaseForm` composable to manage form state and validation in a Vue.js component. The `useBaseForm` composable provides a structured way to handle form data, validation rules, and submission logic.

## Basic Usage

```vue
<template>
  <BaseForm :form="form" :rules="rules" @submit="handleSubmit">
    <BaseFormItem label="Username" prop="username">
      <el-input v-model="form.username" />
    </BaseFormItem>
    <BaseFormItem label="Email" prop="email">
      <el-input v-model="form.email" />
    </BaseFormItem>
    <BaseButton type="primary" native-type="submit">Submit</BaseButton>
  </BaseForm>
</template>

<script setup lang="ts">
const { formData, formRef, loading, initFormData, checkForm } = useBaseForm<{
  username: string
  email: string
}>()

// 定義表單初始值
initFormData({
  username: '',
  email: '',
})

// 定義驗證規則
const formRules = computed(() => ({
  username: [
    { required: true, message: 'Username is required', trigger: 'blur' },
    { min: 3, max: 15, message: 'Username must be between 3 and 15 characters', trigger: 'blur' },
  ],
  email: [
    { required: true, message: 'Email is required', trigger: 'blur' },
    { type: 'email', message: 'Please enter a valid email address', trigger: 'blur' },
  ],
}))

const handleSubmit = async () => {
  if (loading.value) return
  loading.value = true
  const formPass = await checkForm()
  if (!formPass) {
    loading.value = false
    return
  }
}
</script>
```
