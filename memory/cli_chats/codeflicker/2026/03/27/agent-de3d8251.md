---
session_id: agent-de3d8251
date: 2026-03-27
time_start: 2026-03-27T08:21:19.627Z
time_end: 2026-03-27T08:23:14.696Z
project: users-lianzimeng-working-kd-toft-chat-bot
git_branch: feat-multi-modal
---

[2026-03-27 16:21:19] [user]: 请探索当前代码库中文件上传相关的实现。我需要了解：
1. 对话框中的文件上传功能在哪里实现（看截图中的回形针图标按钮）
2. 当前支持的文件类型和文件数量限制
3. 业务域（如finmi）是如何区分和识别的
4. 文件选择器的实现方式

请搜索相关组件、hooks、工具函数等，提供文件路径和关键代码。
[2026-03-27 16:21:26] [assistant]: 我将帮你探索文件上传相关的实现。让我从多个角度搜索相关代码。
[2026-03-27 16:23:14] [assistant]: 现在我已经收集了足够的信息。让我为你总结文件上传相关的实现：

---

## 文件上传功能实现总结

### 1. 对话框中的文件上传功能位置

**核心组件路径：**

| 组件 | 路径 | 作用 |
|------|------|------|
| **UploaderTool** | `/Users/lianzimeng/working/kd.toft.chat-bot/src/components/InputTools/UploaderTool/index.tsx` | 文件上传按钮组件，渲染回形针图标按钮 |
| **Uploader** | `/Users/lianzimeng/working/kd.toft.chat-bot/src/common-components/Uploader/index.tsx` | 底层上传组件，封装文件选择和上传逻辑 |
| **InputFeatureButton** | `/Users/lianzimeng/working/kd.toft.chat-bot/src/common-components/InputFeatureButton/index.tsx` | 功能按钮样式组件，包含 `attachment` 图标 |
| **useFile** | `/Users/lianzimeng/working/kd.toft.chat-bot/src/hooks/useFile.ts` | 文件上传核心 hook，处理加密上传逻辑 |

**组件层级关系：**
```
StargateInput / ChatBotUserInput
  └── UploaderTool  (回形针按钮)
        └── Uploader  (文件选择器)
              └── Upload (@toft/react)
              └── InputFeatureButton (icon="attachment")
```

**关键代码 - 回形针按钮渲染** (`UploaderTool/index.tsx:182-193`):
```tsx
<Uploader
  ref={ref}
  accept={getAcceptFileTypesList(appType!)}
  multiple={true}
  onBeforeUpload={handleBeforeUpload}
  onUploading={handleUploadingAttachment}
  // ...
>
  <InputFeatureButton
    className="attachment-btn"
    size={buttonSize}
    icon="attachment"  // 回形针图标
    onClick={() => {}}
    id={id}
  />
</Uploader>
```

---

### 2. 支持的文件类型和限制

**文件类型映射** (`src/utils/getAcceptFileTypesList.ts`):
```typescript
export const getAcceptFileTypesList = (appType: AppTypeEnum) => {
  switch (appType) {
    case AppTypeEnum.HR:
    case AppTypeEnum.FINANCE:
    case AppTypeEnum.ZONGHE_PROCUREMENT:
    case AppTypeEnum.PROCUREMENT_FIN:
      return AcceptFileTypesList + ',.pdf';  // .xlsx,.xls,.pdf
    case AppTypeEnum.FINANCE_IR:
      return 'pdf';  // 仅 PDF
    default:
      return AcceptFileTypesList;  // .xlsx,.xls
  }
};
```

**默认文件类型** (`src/const.ts:203`):
```typescript
export const AcceptFileTypesList = '.xlsx,.xls';
```

**文件数量和大小限制** (`src/common-components/Uploader/index.tsx:11-12`):
```typescript
const MAX_FILES_NUM = 6;      // 最多 6 个文件
const MAX_FILES_SIZE = 400;   // 总大小不超过 400MB
```

**同类型限制** (`UploaderTool/index.tsx:107-125`):
```typescript
const handleBeforeUpload = (beforeUploadFiles: Array<any>, currentFiles: Array<any>) => {
  // ...
  // 只允许上传同一类型的文件
  if (hasMixedType) {
    message.warning({ message: '当前仅支持上传同一类型的文件' });
    return false;
  }
  return true;
};
```

---

### 3. 业务域（如 finmi）的区分和识别

**URL 参数提取** (`src/hooks/usePageParams.ts`):
```typescript
export default function usePageParams() {
  const { domain } = useParams<Partial<ChatBotPageParams>>();
  // domain 从 URL 中提取，如 'fin', 'hr' 等
  
  return {
    appType: (Domain2AppTypeMap[domain!] || null) as AppTypeEnum | null,
    // ...
  };
}
```

**Domain 到 AppType 的映射** (`src/const.ts:121-136`):
```typescript
// AppType 和 URL domain 的映射
export const AppType2DomainMap = {
  FINANCE: 'fin',      // Finmi 对应 'fin' 域名
  HR: 'hr',
  FINANCE_IR: 'finance_ir',
  // ...
};

// 反向映射
export const Domain2AppTypeMap = {
  'fin': 'FINANCE',    // URL 中的 'fin' 映射到 FINANCE 类型
  'hr': 'HR',
  'finance_ir': 'FINANCE_IR',
  // ...
};
```

**AppType 枚举定义** (`src/types/app.ts:8-32`):
```typescript
export enum AppTypeEnum {
  HR = 'HR',
  FINANCE = 'FINANCE',           // Finmi
  ZONGHE_PROCUREMENT = 'ZONGHE_PROCUREMENT',
  PROCUREMENT_FIN = 'PROCUREMENT_FIN',
  FINANCE_IR = 'FINANCE_IR',
  // ...
}
```

---

### 4. 文件选择器的实现方式

**文件选择器核心实现** (`src/common-components/Uploader/index.tsx`):

```tsx
// 使用 @toft/react 的 Upload 组件
import { Upload } from '@toft/react';

const Uploader = forwardRef<UploaderRef, UploaderProps>((props, ref) => {
  const {
    accept,
    multiple = false,
    onUploading,
    onBeforeUpload,
    // ...
  } = props;

  // 文件选择前校验
  const handleBeforeUpload = (beforeUploadFiles: Array<any>) => {
    // 1. 检查文件总大小
    if (uploadFilesSizeRef.current > MAX_FILES_SIZE) {
      message.warning({ message: '上传成功的文件大小总和不允许超过400MB' });
      return false;
    }
    // 2. 检查文件数量
    if (currentFileCount > MAX_FILES_NUM) {
      message.warning({ message: '上传成功的文件数量不允许大于6' });
      return false;
    }
    // ...
  };

  return (
    <div className="attachment-uploader">
      <Upload
        multiple={!!multiple}
        onUpload={handleUploadFile}
        onBeforeUpload={handleBeforeUpload}
        accept={accept}  // 动态文件类型
      >
        {children}  {/* 回形针按钮 */}
      </Upload>
    </div>
  );
});
```

**文件加密上传流程** (`src/hooks/useFile.ts:109-145`):

```typescript
const uploadFile = async (files: Array<File>) => {
  const formData = new FormData();

  // 根据业务域判断是否需要加密
  if (UnEncFileUploadBotList?.includes(appType!)) {
    // 不加密上传
    files?.forEach((file) => formData.append('files', file as any));
  } else {
    // AES-GCM 加密后上传
    for (const file of files) {
      const { encryptedFile, encryptedKey } = await encryptFile(file);
      formData.append('encryptedFiles', encryptedFile, file.name);
      formData.append('encryptedKeys', encryptedKeyBase64);
    }
  }

  // SSE 流式上传
  const result = await uploadEncryptedFileSSE(formData);
};
```

---

### 关键文件路径汇总

| 功能 | 文件路径 |
|------|----------|
| 上传按钮组件 | `/Users/lianzimeng/working/kd.toft.chat-bot/src/components/InputTools/UploaderTool/index.tsx` |
| 底层上传器 | `/Users/lianzimeng/working/kd.toft.chat-bot/src/common-components/Uploader/index.tsx` |
| 文件上传 hook | `/Users/lianzimeng/working/kd.toft.chat-bot/src/hooks/useFile.ts` |
| 文件类型判断 | `/Users/lianzimeng/working/kd.toft.chat-bot/src/utils/getAcceptFileTypesList.ts` |
| 加密工具函数 | `/Users/lianzimeng/working/kd.toft.chat-bot/src/utils/fileUtils.ts` |
| 常量定义 | `/Users/lianzimeng/working/kd.toft.chat-bot/src/const.ts` |
| AppType 枚举 | `/Users/lianzimeng/working/kd.toft.chat-bot/src/types/app.ts` |
| URL 参数提取 | `/Users/lianzimeng/working/kd.toft.chat-bot/src/hooks/usePageParams.ts` |
| 输入框组件 | `/Users/lianzimeng/working/kd.toft.chat-bot/src/modules/StargateChatBot/components/StargateInput/index.tsx` |
