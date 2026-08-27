---
name: fix-spaceId-not-passed
overview: 修复 PictureUpload.vue 中 spaceId 未传入 FormData 导致后端接收为空的 bug
todos:
  - id: fix-spaceId-upload
    content: 修复 PictureUpload.vue 第 42 行，将 body 参数由空对象改为 params，使 spaceId 写入 FormData
    status: completed
---

## 用户需求

后端打印 `spaceId` 为空，确认是前端上传图片时未将 `spaceId` 传入 FormData，导致后端 `@ModelAttribute` 无法绑定该字段。

## 问题根因

`PictureUpload.vue` 调用 `uploadPictureUsingPost(params, {}, file)` 时，`body` 传入空对象 `{}`，`spaceId` 仅通过 `params` 走 Query String，而后端无注解参数默认走 `@ModelAttribute` 绑定 FormData，Query String 中的 `spaceId` 未被绑定，故为 null。

## 修复范围

仅修改 `src/components/PictureUpload.vue` 第 42 行，将 `body` 由空对象改为 `params`，使 `spaceId`、`id` 等字段同时写入 FormData。

## 技术方案

### 问题定位

| 位置 | 现有代码 | 问题 |
| --- | --- | --- |
| `PictureUpload.vue` L42 | `uploadPictureUsingPost(params, {}, file)` | `body` 为空，FormData 中无 `spaceId` |


### 修复策略

`uploadPictureUsingPost` 函数签名：

- 第 1 参数 `params` → 追加到 URL Query String
- 第 2 参数 `body` → 遍历 key 写入 FormData
- 第 3 参数 `file` → 写入 FormData

将第 2 参数由 `{}` 改为 `params`，使 `spaceId`、`id` 等字段同时进入 FormData，后端 `@ModelAttribute` 即可正常绑定。

### 实现细节

- 改动极小，仅一处调用点，不影响其他逻辑
- `params` 对象同时作为 query string 和 body 传入不会产生副作用，后端以 FormData 优先绑定
- `uploadPictureUsingPost` 内部对 `body` 的处理会跳过 `undefined`/`null` 值，类型安全

## 目录结构

```
src/components/
└── PictureUpload.vue  # [MODIFY] 第 42 行：uploadPictureUsingPost(params, {}, file) → uploadPictureUsingPost(params, params, file)
```