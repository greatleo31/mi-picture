---
name: fix-picture-upload-binding
overview: 修复上传接口multipart绑定，确保PictureUploadRequest字段随文件一起传到后端
todos:
  - id: review-backend-upload
    content: 定位 uploadPicture 控制器与 PictureUploadRequest 定义
    status: completed
  - id: fix-binding
    content: 在 uploadPicture 方法为 DTO 添加 @ModelAttribute 并声明 consumes multipart/form-data
    status: completed
    dependencies:
      - review-backend-upload
  - id: verify-upload
    content: 本地构造 multipart 请求验证 spaceId/id/picName/fileUrl 正确绑定
    status: completed
    dependencies:
      - fix-binding
---

## 产品概述

修复图片上传接口后端无法绑定 PictureUploadRequest，导致 spaceId 等字段为 null。

## 核心功能

- 后端正确接收 multipart/form-data 中的 PictureUploadRequest 字段
- 确保 file 与 DTO 同时绑定，无需额外前端改动

## 技术方案

- 后端：Spring MVC Multipart 绑定修正
- 控制器：`@PostMapping` 声明 `consumes = multipart/form-data`；在 `PictureUploadRequest` 参数上添加 `@ModelAttribute`（或 `@RequestPart` 指定名称）确保从 FormData 绑定；保留 `@RequestPart("file") MultipartFile` 接收文件。
- 校验：保持现有异常抛出，验证绑定后字段非空。