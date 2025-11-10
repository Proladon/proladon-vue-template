---
applyTo: 'src/**/*.{ ts,vue }'
description: 'Coding standards for api usage in TypeScript and Vue files'
---

# Project coding standards for API usage in TypeScript and Vue files

## 基本使用方式

```ts
import { apiFunction } from 'path/to/api/hooks'
const [data, errMsg, rawErr] = await apiFunction<TypeOfData>(params)
if (errMsg) {
  ElMessage.error(errMsg)
  return
}
// 使用 data 進行後續操作
```
