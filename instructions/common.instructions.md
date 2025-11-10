---
applyTo: 'src/**/*.{ts,vue}'
description: 'Coding standards for TypeScript and Vue files'
---

- 如果取用物件中的路徑超過2層，應使用lodash-es 的 get 函數。例如：

```typescript
import { get } from 'lodash-es'
const value = get(object, 'a.b.c', defaultValue)
```
