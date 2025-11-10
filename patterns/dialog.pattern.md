# Dialog usage pattern

## Open and close dialog with useDialogState (Recommended)

使用 `useDialogState` composable 來管理多個對話框的開啟/關閉狀態，提供更好的狀態管理和渲染控制機制。

### 基本用法

```vue
<!--  Child Component: EditAdminModal.vue -->
<template>
  <!-- 不需要 v-model 或是 modelValue，會直接由父組件傳入-->
  <BaseDialog :title="'EditAdminModal'">
    <p>This is an edit admin modal dialog.</p>
  </BaseDialog>
</template>

<script setup lang="ts">
const emit = defineEmits<{
  close: []
  closed: []
}>()
</script>
```

```vue
<template>
  <!-- 使用 v-if 綁定 dialogRender 控制組件的掛載/卸載 -->
  <!-- 使用 v-model 綁定 dialogs 控制對話框的顯示/隱藏 -->
  <!-- 監聽 @close 事件來關閉對話框 -->
  <!-- 監聽 @closed 事件來卸載組件，避免內容殘留 -->
  <EditAdminModal
    v-if="dialogRender.edit"
    v-model="dialogs.edit"
    :action="editAction"
    :select-row="selectedRow"
    @close="closeDialog('edit')"
    @closed="closeDialogRender('edit')"
    @refresh="handleRefresh"
  />
</template>

<script setup lang="ts">
import { useDialogState } from '@/composables/useDialogState'

// 初始化對話框狀態，定義所有需要管理的對話框
const { dialogs, dialogRender, openDialog, closeDialog, closeDialogRender } = useDialogState({
  edit: false,
  create: false,
  delete: false,
})

// 開啟對話框
const handleEdit = () => {
  openDialog('edit')
}

// 處理確認事件
const handleConfirm = () => {
  // 處理邏輯
  closeDialog('edit')
}
</script>
```

### useDialogState 提供的功能

- **dialogs**: 響應式對話框狀態物件，控制對話框的顯示/隱藏（v-model 綁定）
- **dialogRender**: 響應式渲染控制物件，控制組件的掛載/卸載（v-if 綁定）
- **openDialog(name)**: 開啟指定的對話框（同時設置 dialogRender 和 dialogs 為 true）
- **closeDialog(name)**: 關閉指定的對話框（設置 dialogs 為 false，觸發關閉動畫）
- **closeDialogRender(name)**: 卸載對話框組件（設置 dialogRender 為 false，在 @closed 事件中調用）

### 標準事件處理流程

1. **開啟對話框**: 使用 `openDialog('dialogName')`
   - 同時設置 `dialogRender.dialogName = true`（掛載組件）
   - 設置 `dialogs.dialogName = true`（顯示對話框）

2. **關閉對話框**: 在對話框的 `@close` 事件中調用 `closeDialog('dialogName')`
   - 設置 `dialogs.dialogName = false`（觸發關閉動畫）

3. **卸載組件**: 在對話框的 `@closed` 事件中調用 `closeDialogRender('dialogName')`
   - 設置 `dialogRender.dialogName = false`（卸載組件）
   - 確保下次開啟時是全新的組件實例

### 為什麼需要 dialogRender？

- **兩階段關閉機制**:
  - 第一階段：`closeDialog` 觸發關閉動畫
  - 第二階段：`closeDialogRender` 在動畫結束後卸載組件
- **避免狀態殘留**:
  - 通過卸載組件確保每次開啟都是全新實例
  - 避免表單數據、驗證狀態、computed 值等殘留問題
- **性能優化**:
  - 關閉的對話框完全從 DOM 中移除
  - 減少不必要的響應式監聽和計算

### 使用注意事項

1. **必須同時使用 v-if 和 v-model**:

   ```vue
   <!-- ✅ 正確 -->
   <SomeDialog
     v-if="dialogRender.edit"
     v-model="dialogs.edit"
     @close="closeDialog('edit')"
     @closed="closeDialogRender('edit')"
   />

   <!-- ❌ 錯誤：缺少 v-if -->
   <SomeDialog
     v-model="dialogs.edit"
     @close="closeDialog('edit')"
     @closed="closeDialogRender('edit')"
   />
   ```

2. **必須同時監聽 @close 和 @closed**:
   - `@close`: 用戶點擊關閉按鈕時立即調用 `closeDialog`
   - `@closed`: 關閉動畫結束後調用 `closeDialogRender`

3. **簡單對話框可以省略 v-if**:
   - 如果對話框沒有複雜狀態或表單，可以只使用 v-model
   - 例如：簡單的確認對話框

---

## 完整範例

### 帶表單的編輯對話框（推薦使用 v-if）

```vue
<template>
  <EditAdminModal
    v-if="dialogRender.edit"
    v-model="dialogs.edit"
    :action="editAction"
    :select-row="selectedRow"
    @close="closeDialog('edit')"
    @closed="closeDialogRender('edit')"
    @refresh="handleRefresh"
  />
</template>

<script setup lang="ts">
const { dialogs, dialogRender, openDialog, closeDialog, closeDialogRender } = useDialogState({
  edit: false,
})

const editAction = ref<'create' | 'update'>('create')
const selectedRow = ref(null)

const handleCreate = () => {
  editAction.value = 'create'
  selectedRow.value = null
  openDialog('edit')
}

const handleEdit = (row) => {
  editAction.value = 'update'
  selectedRow.value = row
  openDialog('edit')
}

const handleRefresh = () => {
  // 刷新列表邏輯
}
</script>
```

---

## 舊版用法（不推薦）

如果只需要管理單一對話框，可以使用簡單的 ref：

```vue
<template>
  <BaseDialog v-if="showDialog" :model-value="showDialog" @closed="showDialog = false" />
</template>

<script setup lang="ts">
const showDialog = ref<boolean>(false)
</script>
```

**注意**: 此方法不提供 key 更新機制，可能導致對話框內容殘留問題。
