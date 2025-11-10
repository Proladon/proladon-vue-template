# displayData Pattern

## 概述

使用 `computed` 將後端原始數據轉換為統一的 `{ label, value }[]` 結構，用於 UI 層的資訊展示。

## 核心概念

- 統一管理 UI 層需要顯示的資料結構
- 將後端數據格式化為使用者友好的顯示格式
- 集中處理空值、類型轉換、映射等邏輯

## 基本用法

```typescript
const configData = ref<ConfigType>({})

const displayData = computed(() => [
  {
    label: '欄位名稱',
    value: configData.value.field || '-',
  },
  // 更多欄位...
])
```

## 常見轉換場景

### 配置映射

```typescript
import { statusConfig } from '@/config/status'

const displayData = computed(() => [
  {
    label: '狀態',
    value: configData.value.status ? statusConfig[configData.value.status]?.label || '-' : '-',
  },
])
```

### 布林值轉換

```typescript
const displayData = computed(() => [
  {
    label: '是否啟用',
    value: configData.value.enabled ? '是' : '否',
  },
])
```

### 日期格式化

```typescript
import dayjs from 'dayjs'

const displayData = computed(() => [
  {
    label: '建立時間',
    value: configData.value.createdAt
      ? dayjs(configData.value.createdAt).format('YYYY-MM-DD HH:mm:ss')
      : '-',
  },
])
```

### 數值格式化

```typescript
const displayData = computed(() => [
  {
    label: '金額',
    value: configData.value.amount ? `NT$ ${configData.value.amount.toLocaleString()}` : '-',
  },
])
```

### 條件式欄位

```typescript
const displayData = computed(() => {
  const fields = [{ label: '基本欄位', value: configData.value.basic || '-' }]

  if (configData.value.enableAdvanced) {
    fields.push({
      label: '進階欄位',
      value: configData.value.advanced || '-',
    })
  }

  return fields
})
```

## 最佳實踐

1. **統一空值處理**：使用 `'-'` 作為預設值
2. **類型安全**：定義 `DisplayField` 介面
3. **提取複雜邏輯**：將轉換邏輯封裝為獨立函數
4. **保持簡潔**：避免在 computed 中處理過於複雜的業務邏輯

## 使用範例

```vue
<template>
  <BaseElForm label-position="left" label-width="200px">
    <BaseElFormItem v-for="field in displayData" :key="field.label" :label="field.label">
      {{ field.value }}
    </BaseElFormItem>
  </BaseElForm>
</template>
```
