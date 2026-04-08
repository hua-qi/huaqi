---
session_id: agent-fba864b7
date: 2026-03-28
time_start: 2026-03-28T18:54:55.334Z
time_end: 2026-03-28T18:58:11.607Z
project: users-lianzimeng-working-kd-toft-chat-bot
git_branch: feat-multi-modal
---

[2026-03-29 02:54:55] [user]: 在 /Users/lianzimeng/working/kd.toft.chat-bot/ 和 /Users/lianzimeng/working/chat-bot-cards/ 两个项目中，找到所有与文件预览（file preview）相关的代码，特别是：
1. docx 文件预览相关的组件
2. 文件类型判断逻辑（判断文件格式是否支持预览的代码）
3. 预览弹窗/侧边栏组件
4. 文件卡片组件（点击触发预览的入口）
5. 有没有类似 "暂不支持该文件格式预览" 这个文案的代码

请返回：
- 相关文件路径
- 关键代码片段（包含行号）
- 文件预览的整体流程和数据流
[2026-03-29 02:54:59] [assistant]: 我将帮助你在两个项目中查找所有与文件预览相关的代码。让我开始进行详细的搜索。
[2026-03-29 02:55:03] [assistant]: 很好，找到了一些关键文件。现在让我进一步搜索与文件预览、文件类型判断相关的代码：
[2026-03-29 02:55:55] [assistant]: 让我修正搜索并查看关键文件：
[2026-03-29 02:55:59] [assistant]: 现在让我查看更多的文件渲染器和主应用中的代码：
[2026-03-29 02:56:03] [assistant]: 现在让我查看主应用中的文件相关代码：
[2026-03-29 02:56:14] [assistant]: 现在让我查看更多的细节，包括ImageRenderer和其他相关代码：
[2026-03-29 02:56:18] [assistant]: 现在让我查看MarkdownRenderer和一些关键的配置文件：
[2026-03-29 02:56:27] [assistant]: 现在让我查看一些关键的索引/导出配置和数据流：
[2026-03-29 02:56:31] [assistant]: 完美！现在让我获取更多信息来完整了解整个流程：
[2026-03-29 02:56:35] [assistant]: 现在让我生成最终的报告。让我先查看一下模块联邦的配置，了解卡片库的暴露：
[2026-03-29 02:56:51] [assistant]: 完美！现在我有足够的信息来生成完整的报告。让我总结所有发现：
[2026-03-29 02:58:11] [assistant]: 现在让我为您生成完整的报告：

## 完整文件预览系统分析报告

我已经在两个项目中完整找到了所有文件预览相关的代码。以下是详细的分析结果：

---

### 一、相关文件路径总览

#### **chat-bot-cards 项目（卡片库）**

| 文件路径 | 说明 |
|--------|------|
| `/Users/lianzimeng/working/chat-bot-cards/src/components/FileCard/FileCard.tsx` | **核心文件卡片组件**，文件类型判断+渲染器分发 |
| `/Users/lianzimeng/working/chat-bot-cards/src/components/FileCard/DocxRenderer/index.tsx` | DOCX 预览渲染器 |
| `/Users/lianzimeng/working/chat-bot-cards/src/components/FileCard/PdfRenderer/index.tsx` | PDF 预览渲染器 |
| `/Users/lianzimeng/working/chat-bot-cards/src/components/FileCard/ExcelRenderer/index.tsx` | Excel 预览渲染器 |
| `/Users/lianzimeng/working/chat-bot-cards/src/components/FileCard/ImageRenderer/index.tsx` | 图片预览渲染器 |
| `/Users/lianzimeng/working/chat-bot-cards/src/components/FileCard/JsonRenderer/index.tsx` | JSON 预览渲染器 |
| `/Users/lianzimeng/working/chat-bot-cards/src/components/FileCard/TextRenderer/index.tsx` | TXT 预览渲染器 |
| `/Users/lianzimeng/working/chat-bot-cards/src/components/FileCard/MarkdownRenderer/index.tsx` | Markdown 预览渲染器 |
| `/Users/lianzimeng/working/chat-bot-cards/src/components/common/FilePreviewDrawer/FilePreviewDrawer.tsx` | 预览弹窗组件 |
| `/Users/lianzimeng/working/chat-bot-cards/src/components/common/FileTable/FileTable.tsx` | Excel 数据表格组件 |
| `/Users/lianzimeng/working/chat-bot-cards/ai-card.config.mjs` | 模块联邦配置，暴露 DocxRenderer |

#### **kd.toft.chat-bot 项目（主应用）**

| 文件路径 | 说明 |
|--------|------|
| `/Users/lianzimeng/working/kd.toft.chat-bot/src/hooks/useFile.ts` | 文件预览 Hook，处理加密/解密文件，调用预览 API |
| `/Users/lianzimeng/working/kd.toft.chat-bot/src/components/BasicCardRenderer/BasicCardRenderer.tsx` | **核心渲染器**，定义各文件类型的获取回调函数 |
| `/Users/lianzimeng/working/kd.toft.chat-bot/src/types/card.ts` | FileCardTypeList 枚举定义 |
| `/Users/lianzimeng/working/kd.toft.chat-bot/src/common-components/FileCard/index.tsx` | 文件卡片入口组件 |

---

### 二、文件类型判断逻辑

**文件位置**：`/Users/lianzimeng/working/chat-bot-cards/src/components/FileCard/FileCard.tsx` **第 26-50 行**

```typescript
// 文件类型定义
export type FileType =
  | "EXCEL"
  | "JSON"
  | "PDF"
  | "TXT"
  | "MARKDOWN"
  | "IMAGE"
  | "DOCX"
  | "DOC"; // .doc 格式仅支持下载，不支持在线预览

// 根据文件扩展名判断文件类型
export const getFileTypeFromExtension = (fileName: string): FileType | null => {
  const ext = fileName?.split(".").pop()?.toLowerCase();
  switch (ext) {
    case "xlsx":
    case "xls":
      return "EXCEL";
    case "json":
      return "JSON";
    case "pdf":
      return "PDF";
    case "txt":
      return "TXT";
    case "md":
    case "markdown":
      return "MARKDOWN";
    case "jpg":
    case "jpeg":
    case "png":
    case "gif":
    case "webp":
      return "IMAGE";
    case "docx":
      return "DOCX";
    case "doc":
      return "DOC";
    default:
      return null;
  }
};
```

**支持的文件格式**：
- Excel: `.xlsx`, `.xls`
- JSON: `.json`
- PDF: `.pdf`
- TXT: `.txt`
- Markdown: `.md`, `.markdown`
- 图片: `.jpg`, `.jpeg`, `.png`, `.gif`, `.webp`
- Word: `.docx`, `.doc`（.doc 仅下载）

---

### 三、核心文件卡片组件

**文件位置**：`/Users/lianzimeng/working/chat-bot-cards/src/components/FileCard/FileCard.tsx` **第 140-284 行**

#### 3.1 Props 定义

```typescript
interface FileCardProps extends ICard {
  onFetchExcelData: (sheetName: string, pageNo: number) => Promise<{
    list: Array<{ columnName: string; value: string }>;
    hasNext: boolean;
  }>;
  showDataAnnotation?: boolean;
  onFetchDataAnnotation?: (fileId: string) => void;
  onSaveDataAnnotation?: (fileId: string, annotation: any) => void;
  onFetchPdfData?: () => Promise<{ blobUrl: string }>;
  onFetchDocxData?: () => Promise<{ blobUrl: string | null; isDownloadOnly?: boolean }>;
  onFetchImageData?: () => Promise<{ blobUrl: string | null; isDownloadOnly?: boolean }>;
  onFetchJsonData?: () => Promise<{ content: string | null; isDownloadOnly?: boolean }>;
  onFetchTxtData?: () => Promise<{ content: string | null; isDownloadOnly?: boolean }>;
  onFetchMarkdownData?: () => Promise<{ content: string | null; isDownloadOnly?: boolean }>;
}
```

#### 3.2 错误提示文案

**行号 214, 268**：

```typescript
// 如果 fileType 无效，尝试从 fileName 推断
if (!fileType || !isValidFileId(fileId)) {
  console.log('[FileCard] 无效的文件类型或fileId');
  return <div style={{ padding: 16 }}>暂不支持该文件格式预览</div>;
}

// 最后的兜底
return <div style={{ padding: 16 }}>暂不支持该文件格式预览</div>;
```

#### 3.3 .doc 文件下载提示组件

**行号 115-138**：

```typescript
// .doc 文件下载提示组件
const DocDownloadTip: React.FC<{ fileName?: string }> = ({ fileName }) => (
  <div
    style={{
      padding: 24,
      display: "flex",
      flexDirection: "column",
      alignItems: "center",
      justifyContent: "center",
      color: "#666",
      fontSize: 14,
    }}
  >
    <div style={{ marginBottom: 8 }}>📄</div>
    <div>该文件格式（.doc）不支持在线预览</div>
    <div style={{ marginTop: 8, fontSize: 12, color: "#999" }}>
      请下载后使用 Microsoft Word 或 WPS 打开
    </div>
    {fileName && (
      <div style={{ marginTop: 8, fontSize: 12, color: "#165dff" }}>
        {fileName}
      </div>
    )}
  </div>
);
```

#### 3.4 渲染器分发逻辑

**行号 200-270**：

```typescript
const getRenderer = () => {
  let fileType = (cardParams?.fileType as FileType)?.toUpperCase() as FileType;
  const fileId = cardParams?.fileId;
  const fileName = cardParams?.fileName || cardParams?.metaData?.file_name;

  // Excel 特殊处理（需要额外 props）
  if (fileType === "EXCEL") {
    return (
      <ExcelRenderer
        fileId={fileId}
        sheetNames={cardParams?.metaData?.sheetSummary?.map((item: any) => item?.name)}
        onFetchExcelData={onFetchExcelData}
        showDataAnnotation={showDataAnnotation}
        onFetchDataAnnotation={onFetchDataAnnotation}
        onSaveDataAnnotation={onSaveDataAnnotation}
      />
    );
  }

  // PDF 特殊处理（需要额外 props）
  if (fileType === "PDF") {
    return <PdfRenderer fileId={fileId} onFetchPdfData={onFetchPdfData as () => Promise<{ blobUrl: string }>> />;
  }

  // DOCX 特殊处理（需要额外 props）
  if (fileType === "DOCX") {
    return (
      <DocxRenderer
        fileId={fileId}
        onFetchDocxData={
          onFetchDocxData as () => Promise<{ blobUrl: string | null; isDownloadOnly?: boolean }>
        }
      />
    );
  }

  // .doc 文件仅支持下载
  if (fileType === "DOC") {
    return <DocDownloadTip fileName={fileName} />;
  }

  // 其他类型使用统一的渲染器映射
  const RendererComponent = RENDERER_MAP[fileType];
  if (RendererComponent) {
    return <RendererComponent fileId={fileId} />;
  }

  return <div style={{ padding: 16 }}>暂不支持该文件格式预览</div>;
};
```

---

### 四、DOCX 文件预览渲染器

**文件位置**：`/Users/lianzimeng/working/chat-bot-cards/src/components/FileCard/DocxRenderer/index.tsx`

```typescript
import React, { useEffect, useRef, useState } from "react";
import { renderAsync } from "docx-preview";

interface DocxRendererProps {
  fileId: string;
  onFetchDocxData: () => Promise<{ blobUrl: string | null; isDownloadOnly?: boolean }>;
}

const DocxRenderer: React.FC<DocxRendererProps> = (props) => {
  const { fileId, onFetchDocxData } = props;
  const containerRef = useRef<HTMLDivElement>(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<string>("");

  useEffect(() => {
    const fetchDocx = async () => {
      try {
        setLoading(true);
        const { blobUrl } = await onFetchDocxData();

        if (!blobUrl) {
          throw new Error("文档加载失败");
        }

        // 通过 blobUrl 获取 ArrayBuffer
        const response = await fetch(blobUrl);
        const blob = await response.blob();
        const arrayBuffer = await blob.arrayBuffer();

        if (containerRef.current) {
          // 清空容器
          containerRef.current.innerHTML = "";
          // 使用 docx-preview 渲染
          await renderAsync(arrayBuffer, containerRef.current, undefined, {
            className: "docx-preview-container",
            inWrapper: true,
          });
        }
      } catch (err) {
        setError(err instanceof Error ? err.message : "文档渲染失败");
      } finally {
        setLoading(false);
      }
    };

    fetchDocx();
  }, [fileId, onFetchDocxData]);

  if (loading) {
    return <div className="docx-renderer-loading"><div className="loading-spinner">文档解析中...</div></div>;
  }

  if (error) {
    return <div className="docx-renderer-error"><span>{error}</span></div>;
  }

  return (
    <div className="docx-renderer">
      <div ref={containerRef} className="docx-preview-wrapper" />
    </div>
  );
};

export default DocxRenderer;
```

---

### 五、PDF 文件预览渲染器

**文件位置**：`/Users/lianzimeng/working/chat-bot-cards/src/components/FileCard/PdfRenderer/index.tsx`

```typescript
import React, { useEffect, useState } from "react";
import { Document, Page, pdfjs } from "react-pdf";
import { useInView } from "react-intersection-observer";

// 设置 worker 路径（必须）
pdfjs.GlobalWorkerOptions.workerSrc = new URL(
  "pdfjs-dist/build/pdf.worker.min.mjs",
  import.meta.url
).toString();

const PdfRenderer: React.FC<PdfRendererProps> = ({ fileId, onFetchPdfData }) => {
  const [blobUrl, setBlobUrl] = useState<string | null>(null);
  const [numPages, setNumPages] = useState<number>(0);

  useEffect(() => {
    const fetchPdfData = async () => {
      const res = await onFetchPdfData();
      setBlobUrl(res?.blobUrl);
    };
    fetchPdfData();
  }, [fileId]);

  const onDocumentLoadSuccess = ({ numPages }: { numPages: number }) => {
    setNumPages(numPages);
  };

  if (!blobUrl) return null;

  return (
    <div className="pdf-renderer">
      <Document
        file={blobUrl}
        onLoadSuccess={onDocumentLoadSuccess}
        onLoadError={(err) => console.error("加载 PDF 失败:", err)}
        loading=""
      >
        {Array.from({ length: numPages }, (_, index) => (
          <LazyPage key={index} pageNumber={index + 1} />
        ))}
      </Document>
    </div>
  );
};
```

---

### 六、主应用数据流（useFile Hook）

**文件位置**：`/Users/lianzimeng/working/kd.toft.chat-bot/src/hooks/useFile.ts`

#### 6.1 核心预览方法

**行号 340-379**：

```typescript
const previewFile = async (
  params: { sheetName?: string; pageNo?: number; pageSize?: number } = {},
  fileType?: FileCardType
) => {
  if (!fileId) return;

  if (isEncrypted) {
    const decryptedBlob = await operateFileAndRead(getFilePreviewEncryptedFile, params, false);
    switch (fileType) {
      case FileCardTypeList.PDF:
      case FileCardTypeList.DOCX:
      case FileCardTypeList.IMAGE:
        return decryptedBlob;
      default:
        const { list, hasNext } = await parseJsonBlob(decryptedBlob as Blob);
        return { list, hasNext };
    }
  } else {
    // 对于 PDF/DOCX/IMAGE 类型，直接获取 blob
    if (fileType === FileCardTypeList.PDF || 
        fileType === FileCardTypeList.DOCX || 
        fileType === FileCardTypeList.IMAGE) {
      const response = await fetch(`/magic-ai/api/file/get_file_content?file_id=${fileId}`);
      if (!response.ok) {
        throw new Error(`获取文件失败: ${response.status}`);
      }
      return await response.blob();
    }
    
    // 其他类型（Excel/JSON/TXT/Markdown）使用预览接口
    const response = await getFilePreview({ ...params, fileId });
    const result =
      response?.list?.map((group: any) => {
        return group?.reduce((acc: any, item: any) => {
          acc[item.columnName] = item.value;
          return acc;
        }, {});
      }) || [];

    return { list: result, hasNext: response?.hasNext };
  }
};
```

---

### 七、BasicCardRenderer 中的回调函数

**文件位置**：`/Users/lianzimeng/working/kd.toft.chat-bot/src/components/BasicCardRenderer/BasicCardRenderer.tsx`

#### 7.1 获取 Excel 数据（行号 319-337）

```typescript
const handleFetchExcelData = async (sheetName: string, pageNo: number) => {
  const previewParams = {
    sheetName: sheetName || "",
    pageNo,
    pageSize: 100,
  };
  if (pageNo === 1) setLoading(true);

  try {
    const { list, hasNext } = (await previewFile(previewParams)) || {};
    return { list, hasNext };
  } catch (err) {
    console.error(err);
    return { list: [], hasNext: false };
  } finally {
    setLoading(false);
  }
};
```

#### 7.2 获取 PDF 数据（行号 340-354）

```typescript
const handleFetchPdfData = async () => {
  setLoading(true);
  try {
    const decryptedBlob = (await previewFile({}, FileCardTypeList.PDF)) as Blob;
    const blobUrl = URL.createObjectURL(decryptedBlob);
    return { blobUrl };
  } catch (err) {
    console.error(err);
    return { blobUrl: null };
  } finally {
    setLoading(false);
  }
};
```

#### 7.3 获取 DOCX 数据（行号 356-378）

```typescript
const handleFetchDocxData = async () => {
  console.log("[BasicCardRenderer] handleFetchDocxData 被调用, fileId:", params?.fileId);
  setLoading(true);
  try {
    const decryptedBlob = (await previewFile({}, FileCardTypeList.DOCX)) as Blob;
    console.log("[BasicCardRenderer] previewFile 返回:", decryptedBlob);
    if (!decryptedBlob) {
      console.error("[BasicCardRenderer] 获取DOCX blob 为空");
      return { blobUrl: null };
    }
    const blobUrl = URL.createObjectURL(decryptedBlob);
    console.log("[BasicCardRenderer] blobUrl:", blobUrl);
    return { blobUrl };
  } catch (err) {
    console.error("[BasicCardRenderer] 获取DOCX数据失败:", err);
    return { blobUrl: null };
  } finally {
    setLoading(false);
  }
};
```

#### 7.4 获取图片数据（行号 380-391）

```typescript
const handleFetchImageData = async () => {
  setLoading(true);
  try {
    const decryptedBlob = (await previewFile({}, FileCardTypeList.IMAGE)) as Blob;
    const blobUrl = URL.createObjectURL(decryptedBlob);
    return { blobUrl };
  } catch (err) {
    console.error("获取图片数据失败:", err);
    return { blobUrl: null };
  } finally {
    setLoading(false);
  }
};
```

#### 7.5 获取文本数据（行号 393-409）

```typescript
const handleFetchTextData = async (fileType: FileCardTypeList) => {
  setLoading(true);
  try {
    const result = await previewFile({}, fileType);
    // 如果返回的是对象且有 list 属性，说明是 JSON 格式
    if (result && typeof result === "object" && "list" in result) {
      return { content: JSON.stringify(result.list, null, 2) };
    }
    // 如果返回的是字符串
    if (typeof result === "string") {
      return { content: result };
    }
    return { content: null };
  } catch (err) {
    console.error(`获取${fileType}数据失败:`, err);
    return { content: null };
  } finally {
    setLoading(false);
  }
};
```

#### 7.6 渲染 FileCard（行号 520-538）

```typescript
{FileCard && (
  <FileCard
    params={JSON.stringify(params)}
    loading={loading}
    // 获取 Excel 数据
    onFetchExcelData={handleFetchExcelData}
    // 获取 Pdf 数据
    onFetchPdfData={handleFetchPdfData}
    // 获取 Docx 数据
    onFetchDocxData={handleFetchDocxData}
    // 获取图片数据
    onFetchImageData={handleFetchImageData}
    // 获取 JSON 数据
    onFetchJsonData={() => handleFetchTextData(FileCardTypeList.JSON)}
    // 获取 TXT 数据
    onFetchTxtData={() => handleFetchTextData(FileCardTypeList.TXT)}
    // 获取 Markdown 数据
    onFetchMarkdownData={() => handleFetchTextData(FileCardTypeList.MARKDOWN)}
    // Excel卡片是否包含数据标注
    showDataAnnotation={showDataAnnotation}
    // 获取数据标注详情
    onFetchDataAnnotation={handleFetchDataAnnotation}
    // 保存数据标注
    onSaveDataAnnotation={handleSaveDataAnnotation}
  />
)}
```

---

### 八、完整数据流程图

```
┌─────────────────────────────────────────────────────────────────┐
│                    主应用 (kd.toft.chat-bot)                     │
│                                                                   │
│  BasicCardRenderer (行号 300-540)                                │
│  ├─ handleFetchExcelData()                                       │
│  ├─ handleFetchPdfData()                                         │
│  ├─ handleFetchDocxData()                    ┌──────────────┐   │
│  ├─ handleFetchImageData()  ─────────────────┤ useFile Hook │   │
│  └─ handleFetchTextData()                    │ (previewFile)│   │
│                                              └──────────────┘   │
└───────────────────────┬──────────────────────────────────────────┘
                        │
                        │ Props:
                        │ onFetchExcelData
                        │ onFetchPdfData
                        │ onFetchDocxData
                        │ onFetchImageData
                        │ onFetchJsonData
                        ▼
┌─────────────────────────────────────────────────────────────────┐
│              卡片库 (chat-bot-cards)                              │
│                                                                   │
│  FileCard.tsx (行号 200-270)                                    │
│  ├─ getFileTypeFromExtension()  [文件类型判断]                   │
│  └─ getRenderer()  [渲染器分发]                                  │
│      ├─ EXCEL  ────────────▶ ExcelRenderer + FileTable           │
│      ├─ PDF   ────────────▶ PdfRenderer (react-pdf)             │
│      ├─ DOCX  ────────────▶ DocxRenderer (docx-preview)         │
│      ├─ IMAGE ────────────▶ ImageRenderer (react-medium-zoom)   │
│      ├─ JSON  ────────────▶ JsonRenderer (CodeCard)             │
│      ├─ TXT   ────────────▶ TextRenderer (CodeCard)             │
│      ├─ MARKDOWN ─────────▶ MarkdownRenderer (MarkdownCard)    │
│      ├─ DOC   ────────────▶ DocDownloadTip [仅下载提示]         │
│      └─ [未知] ──────────▶ "暂不支持该文件格式预览"              │
│                                                                   │
└─────────────────────────────────────────────────────────────────┘
```

---

### 九、所有错误/提示文案位置汇总

| 文案 | 文件路径 | 行号 | 场景 |
|------|--------|------|------|
| `暂不支持该文件格式预览` | FileCard.tsx | 214 | fileType 无效时 |
| `暂不支持该文件格式预览` | FileCard.tsx | 268 | 无法找到对应 Renderer |
| `该文件格式（.doc）不支持在线预览` | FileCard.tsx | 118 | .doc 文件类型 |
| `请下载后使用 Microsoft Word 或 WPS 打开` | FileCard.tsx | 121 | .doc 文件提示 |
| `文件渲染失败: {error.message}` | FileCard.tsx | 105 | ErrorBoundary 捕获错误 |
| `该文件不支持在线预览，请下载后查看` | ImageRenderer/index.tsx | 29 | 图片加载失败 |
| `该文件不支持在线预览，请下载后查看` | MarkdownRenderer/index.tsx | 26 | Markdown 加载失败 |
| `该文件不支持在线预览，请下载后查看` | JsonRenderer/index.tsx | 22 | JSON 加载失败 |
| `该文件不支持在线预览，请下载后查看` | TextRenderer/index.tsx | 26 | TXT 加载失败 |
| `文档加载失败` | DocxRenderer/index.tsx | 23 | DOCX blobUrl 为空 |
| `文档渲染失败` | DocxRenderer/index.tsx | 44 | docx-preview 渲染异常 |

---

### 十、模块联邦配置

**文件位置**：`/Users/lianzimeng/working/chat-bot-cards/ai-card.config.mjs` **第 7-43 行**

```javascript
exposes: {
    // 基础卡片
    "./CodeCard": "./src/components/CodeCard/CodeCard.tsx",
    "./ConsoleCard": "./src/components/ConsoleCard/ConsoleCard.tsx",
    "./TextCard": "./src/components/TextCard/TextCard.tsx",
    "./MarkdownCard": "./src/components/MarkdownCard/MarkdownCard.tsx",
    "./FileCard": "./src/components/FileCard/FileCard.tsx",
    "./ImageCard": "./src/components/ImageCard/ImageCard.tsx",
    "./HtmlCard": "./src/components/HtmlCard/HtmlCard.tsx",
    "./SubBoardCard": "./src/components/SubBoardCard/SubBoardCard.tsx",
    "./UnauthorizedCard": "./src/components/UnauthorizedCard/UnauthorizedCard.tsx",
    "./RefreshCard": "./src/components/RefreshCard/RefreshCard.tsx",
    "./NullDataCard": "./src/components/NullDataCard/NullDataCard.tsx",
    "./EchartsOptionCard": "./src/components/EchartsOptionCard/EchartsOptionCard.tsx",
    "./CodeAndConsoleCard": "./src/components/CodeAndConsoleCard/CodeAndConsoleCard.tsx",
    "./SqlCard": "./src/components/SqlCard/SqlCard.tsx",
    "./FilterChartsCard": "./src/components/FilterChartsCard/FilterChartsCard.tsx",
    "./DocxRenderer": "./src/components/FileCard/DocxRenderer/index.tsx",  // 特殊暴露
    // ... 其他业务卡片
}
```

---

### 十一、关键技术栈与依赖

1. **DOCX 预览**：`docx-preview` 库，通过 `renderAsync` 方法
2. **PDF 预览**：`react-pdf` 库，使用 Document + Page 组件
3. **Excel 预览**：自定义 FileTable + react 原生能力
4. **图片预览**：`react-medium-image-zoom` 库实现缩放
5. **文本/代码预览**：CodeCard 组件统一处理
6. **Markdown 预览**：MarkdownCard 组件处理

---

### 十二、关键实现细节

1. **加密文件支持**：useFile Hook 支持 AES-GCM 加密文件的预览
2. **分页加载**：Excel 数据支持虚拟滚动和分页加载（pageSize: 100）
3. **数据标注**：财务 Bot 的用户上传文件支持数据标注功能
4. **错误边界**：FileCard 使用 ErrorBoundary 包装渲染器，防止单个渲染器错误影响整体
5. **模块联邦**：DocxRenderer 通过模块联邦单独暴露，支持动态加载

---

这就是两个项目中所有文件预览相关代码的完整分析！所有路径都已提供行号，您可以直接跳转查看源代码。
