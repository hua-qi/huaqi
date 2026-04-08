---
session_id: agent-fddbd3e8
date: 2026-04-02
time_start: 2026-04-02T04:11:03.303Z
time_end: 2026-04-02T04:13:10.543Z
project: users-lianzimeng-working-kd-toft-chat-bot
git_branch: feat-multi-modal-v2
---

[2026-04-02 12:11:03] [user]: 在 /Users/lianzimeng/working/chat-bot-cards/src 项目中，找到 FileCard 组件下的 PDF 文件预览组件和 MD (Markdown) 文件预览组件的源码。请返回：1. 这些组件的完整文件路径 2. 每个文件的完整代码内容（包括样式文件）。需要非常详细的信息，包括所有相关文件。
[2026-04-02 12:11:05] [assistant]: 我来帮你在项目中找到 FileCard 组件下的 PDF 和 Markdown 文件预览组件。让我开始搜索。
[2026-04-02 12:11:09] [assistant]: 很好！我已经找到了关键文件。现在让我查看 FileCard 的目录结构和主文件，以及 PDF 和 Markdown 预览组件。
[2026-04-02 12:11:12] [assistant]: 现在让我读取相关的样式文件和其他相关组件：
[2026-04-02 12:11:15] [assistant]: 现在让我检查一下是否有其他相关的组件（如 MarkdownCard），以及导入的依赖：
[2026-04-02 12:11:23] [assistant]: 现在让我读取 MarkdownCard 的支持文件：
[2026-04-02 12:11:26] [assistant]: 现在让我读取 CustomLink 的样式文件和其他渲染器以完整性：
[2026-04-02 12:11:28] [assistant]: 让我检查 MarkdownCard 中的 svg.tsx：
[2026-04-02 12:11:31] [assistant]: 现在让我获取其他相关的预览器以提供完整的组件信息：
[2026-04-02 12:11:34] [assistant]: 现在让我读取 ImageRenderer 和 DocxRenderer 来完整性：
[2026-04-02 12:11:37] [assistant]: 现在让我读取所有样式文件以完整性：
[2026-04-02 12:13:10] [assistant]: 完美！现在让我创建一个综合总结文档，包含所有需要的信息。

## 完整的 FileCard 组件源码报告

我已经详细地找到并整理了你在 `/Users/lianzimeng/working/chat-bot-cards/src` 项目中的 FileCard 组件下的所有相关代码。以下是**完整的文件路径和源码内容**：

---

## **1. PDF 文件预览组件（PdfRenderer）**

### 📁 文件路径：
- `/Users/lianzimeng/working/chat-bot-cards/src/components/FileCard/PdfRenderer/index.tsx`
- `/Users/lianzimeng/working/chat-bot-cards/src/components/FileCard/PdfRenderer/index.less`

### 📄 源码内容：

#### **index.tsx**
```typescript
import React, { useEffect, useState } from "react";
import { Document, Page, pdfjs } from "react-pdf";
import { useInView } from "react-intersection-observer";

import "./index.less";

// 提取 workerUrl
const workerUrl = new URL(
  "pdfjs-dist/build/pdf.worker.min.mjs",
  import.meta.url
).toString();

interface PdfRendererProps {
  fileId: string;
  onFetchPdfData: () => Promise<{ blobUrl: string }>;
}

let initWorkerPromise: Promise<void> | null = null;

const PdfRenderer: React.FC<PdfRendererProps> = ({
  fileId,
  onFetchPdfData,
}) => {
  const [blobUrl, setBlobUrl] = useState<string | null>(null);
  const [numPages, setNumPages] = useState<number>(0);
  const [workerReady, setWorkerReady] = useState<boolean>(!!pdfjs.GlobalWorkerOptions.workerPort);
  const [debugInfo, setDebugInfo] = useState<string>("初始化...");
  const [errorInfo, setErrorInfo] = useState<string>("");

  useEffect(() => {
    if (pdfjs.GlobalWorkerOptions.workerPort) {
      setWorkerReady(true);
      setDebugInfo("Worker 已经就绪");
      return;
    }
    
    if (!initWorkerPromise) {
      setDebugInfo(`开始加载 Worker: ${workerUrl}`);
      initWorkerPromise = fetch(workerUrl)
        .then((res) => {
          if (!res.ok) throw new Error(`HTTP Error: ${res.status}`);
          return res.blob();
        })
        .then((blob) => {
          const workerBlobUrl = URL.createObjectURL(blob);
          pdfjs.GlobalWorkerOptions.workerPort = new Worker(workerBlobUrl, {
            type: "module",
          });
          setDebugInfo("Worker 加载并实例化成功");
        })
        .catch((err) => {
          console.error("加载 PDF Worker 失败:", err);
          setDebugInfo(`加载 PDF Worker 失败: ${err.message}`);
          setErrorInfo(`Worker Error: ${err.message}`);
        });
    }
    
    initWorkerPromise.then(() => setWorkerReady(true));
  }, []);

  useEffect(() => {
    const fetchPdfData = async () => {
      try {
        const res = await onFetchPdfData();
        setBlobUrl(res?.blobUrl);
      } catch (err: any) {
        setErrorInfo(`Fetch Data Error: ${err.message}`);
      }
    };
    fetchPdfData();
  }, [fileId]);

  const onDocumentLoadSuccess = ({ numPages }: { numPages: number }) => {
    setNumPages(numPages);
  };

  const onDocumentLoadError = (err: any) => {
    console.error("加载 PDF 失败:", err);
    setErrorInfo(`PDF.js Error: ${err?.message || err}`);
  };

  if (errorInfo) {
    return (
      <div style={{ padding: 20, color: "red" }}>
        <h3>PDF 加载错误</h3>
        <p>{errorInfo}</p>
        <p>Debug Info: {debugInfo}</p>
        <p>Blob URL: {blobUrl}</p>
        <p>Worker URL: {workerUrl}</p>
      </div>
    );
  }

  if (!blobUrl || !workerReady) {
    return <div style={{ padding: 20 }}>正在加载 PDF 和 Worker... <br/> Debug Info: {debugInfo}</div>;
  }

  return (
    <div className="pdf-renderer">
      <Document
        file={blobUrl}
        onLoadSuccess={onDocumentLoadSuccess}
        onLoadError={onDocumentLoadError}
        loading="正在解析 PDF..."
      >
        {Array.from({ length: numPages }, (_, index) => (
          <LazyPage key={index} pageNumber={index + 1} />
        ))}
      </Document>
    </div>
  );
};

// 单页懒加载组件
const LazyPage: React.FC<{ pageNumber: number }> = ({ pageNumber }) => {
  const { ref, inView } = useInView({
    threshold: 0.1,
    triggerOnce: true,
  });

  return (
    <div ref={ref} className="pdf-renderer-page">
      {inView ? (
        <Page
          pageNumber={pageNumber}
          renderTextLayer={false}
          renderAnnotationLayer={false}
        />
      ) : null}
    </div>
  );
};

export default PdfRenderer;
```

#### **index.less**
```less
.pdf-renderer {
  width: 100%;
  height: 100%;
  overflow-x: hidden;
  .react-pdf__Document {
    display: flex;
    flex-direction: column;
    gap: 6px;
    background-color: #f2f5f7;
    .pdf-renderer-page {
      canvas {
        height: auto !important;
        width: 100% !important;
      }
    }
  }
}
```

---

## **2. Markdown 文件预览组件（MarkdownRenderer）**

### 📁 文件路径：
- `/Users/lianzimeng/working/chat-bot-cards/src/components/FileCard/MarkdownRenderer/index.tsx`
- `/Users/lianzimeng/working/chat-bot-cards/src/components/FileCard/MarkdownRenderer/index.less`

### 📄 源码内容：

#### **index.tsx**
```typescript
import React, { useEffect, useState } from "react";
import MarkdownCard from "../../MarkdownCard";

import "./index.less";

interface MarkdownRendererProps {
  fileId: string;
  onFetchMarkdownData?: () => Promise<{ content: string | null; isDownloadOnly?: boolean }>;
}

const MarkdownRenderer: React.FC<MarkdownRendererProps> = ({ fileId, onFetchMarkdownData }) => {
  const [content, setContent] = useState<string>("");
  const [loading, setLoading] = useState<boolean>(true);
  const [error, setError] = useState<string>("");

  useEffect(() => {
    const fetchMarkdownContent = async () => {
      try {
        setLoading(true);
        setError("");

        if (onFetchMarkdownData) {
          // 使用传入的数据获取回调
          const { content, isDownloadOnly } = await onFetchMarkdownData();
          if (isDownloadOnly) {
            setError("该文件不支持在线预览，请下载后查看");
          } else if (content) {
            setContent(content);
          } else {
            setError("Markdown 加载失败");
          }
        } else {
          // 兜底：使用原来的 fetch 方式
          const response = await fetch(
            `/magic-ai/api/file/get_file_content?file_id=${fileId}`
          );

          if (!response.ok) {
            throw new Error(`请求失败: ${response.status}`);
          }

          const data = await response.json();
          setContent(data?.result || "");
        }
      } catch (err) {
        const errorMessage = err instanceof Error ? err.message : "Markdown 加载失败";
        setError(errorMessage);
        console.error("MarkdownRenderer error:", err);
      } finally {
        setLoading(false);
      }
    };

    if (fileId) {
      fetchMarkdownContent();
    }
  }, [fileId, onFetchMarkdownData]);

  if (error) {
    return (
      <div className="markdown-renderer-error">
        <span>{error}</span>
      </div>
    );
  }

  return (
    <div className={`markdown-renderer ${loading ? 'loading' : ''}`}>
      <MarkdownCard params={content} loading={loading} />
    </div>
  );
};

export default MarkdownRenderer;
```

#### **index.less**
```less
.markdown-renderer {
  width: 100%;
  height: 100%;
  overflow-x: hidden;
  background-color: #f2f5f7;
  padding: 12px;

  &.loading {
    display: flex;
    align-items: center;
    justify-content: center;
    padding: 0;

    .markdown-card {
      background: transparent;
      box-shadow: none;
      padding: 0;
      width: auto !important;
      height: auto !important;
      
      .toft-loading-backdrop {
        background: transparent !important;
      }
    }
  }

  .markdown-card {
    background: #ffffff;
    border-radius: 4px;
    box-shadow: 0 1px 2px rgba(0, 0, 0, 0.05);
    padding: 16px;
  }
}

.markdown-renderer-error {
  width: 100%;
  height: 100%;
  display: flex;
  align-items: center;
  justify-content: center;
  background-color: #f2f5f7;
  color: #ff4d4f;
  font-size: 14px;
}
```

---

## **3. FileCard 主组件**

### 📁 文件路径：
- `/Users/lianzimeng/working/chat-bot-cards/src/components/FileCard/FileCard.tsx`
- `/Users/lianzimeng/working/chat-bot-cards/src/components/FileCard/index.ts`
- `/Users/lianzimeng/working/chat-bot-cards/src/components/FileCard/style/index.less`

### 📄 源码内容：

#### **FileCard.tsx**
```typescript
import React, { useCallback, useEffect, useState } from "react";
import ExcelRenderer from "./ExcelRenderer";
import JsonRenderer from "./JsonRenderer";
import TextRenderer from "./TextRenderer";
import MarkdownRenderer from "./MarkdownRenderer";
import ImageRenderer from "./ImageRenderer";
import DocxRenderer from "./DocxRenderer";
import CardLoading from "../CardLoading";
import PdfRenderer from "./PdfRenderer";
import type { ICard } from "../types";
import EmptyContentPng from "../../assets/empty_content.png";

import "./style/index.less";

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
  const ext = fileName?.split(".")?.pop()?.toLowerCase();
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

// 检查 fileId 是否有效
const isValidFileId = (fileId: string): boolean => {
  return !!fileId && fileId !== "undefined" && fileId !== "null";
};

// 渲染器组件映射表（只包含标准渲染器，不含需要特殊处理的类型）
const RENDERER_MAP: Record<
  Exclude<FileType, "EXCEL" | "PDF" | "DOC" | "DOCX">,
  React.ComponentType<{ fileId: string }>
> = {
  JSON: JsonRenderer,
  TXT: TextRenderer,
  MARKDOWN: MarkdownRenderer,
  IMAGE: ImageRenderer,
};

// 错误边界组件
class ErrorBoundary extends React.Component<
  { children: React.ReactNode; fallback?: React.ReactNode },
  { hasError: boolean; error?: Error }
> {
  constructor(props: { children: React.ReactNode; fallback?: React.ReactNode }) {
    super(props);
    this.state = { hasError: false };
  }

  static getDerivedStateFromError(error: Error) {
    return { hasError: true, error };
  }

  componentDidCatch(error: Error, errorInfo: React.ErrorInfo) {
    console.error("FileCard render error:", error, errorInfo);
  }

  render() {
    if (this.state.hasError) {
      return (
        this.props.fallback || (
          <div style={{ padding: 16, color: "#ff4d4f" }}>
            文件渲染失败: {this.state.error?.message}
          </div>
        )
      );
    }
    return this.props.children;
  }
}

// .doc 文件下载提示组件
const DocDownloadTip: React.FC<{ fileName?: string; downloadUrl?: string }> = () => (
  <div
    style={{
      display: "flex",
      flexDirection: "column",
      alignItems: "center",
      justifyContent: "center",
      height: "100%",
      minHeight: 400,
      color: "#666",
      fontSize: 14,
    }}
  >
    <img src={EmptyContentPng} style={{ width: 240, marginBottom: 16 }} />
    <div>该文件格式（.doc）不支持在线预览</div>
    <div style={{ marginTop: 8, fontSize: 12, color: "#999" }}>
      请下载后使用 Microsoft Word 或 WPS 打开
    </div>
  </div>
);

interface FileCardProps extends ICard {
  onFetchExcelData: (
    sheetName: string,
    pageNo: number
  ) => Promise<{
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

const FileCard: React.FC<FileCardProps> = (props) => {
  const {
    params,
    loading,
    onFetchExcelData,
    onFetchPdfData,
    onFetchDocxData,
    onFetchImageData,
    onFetchJsonData,
    onFetchTxtData,
    onFetchMarkdownData,
    showDataAnnotation,
    onFetchDataAnnotation,
    onSaveDataAnnotation,
  } = props;

  const [cardParams, setCardParams] = useState<any>();

  const processCardParams = useCallback(() => {
    const data = JSON.parse(params || "{}");
    
    // 安全地解析 metaData（可能是字符串或对象）
    let metaData = {};
    if (data?.metaData) {
      if (typeof data.metaData === 'string') {
        try {
          metaData = JSON.parse(data.metaData);
        } catch (e) {
          console.error('[FileCard] 解析 metaData 失败:', e);
          metaData = {};
        }
      } else if (typeof data.metaData === 'object') {
        metaData = data.metaData;
      }
    }
    
    setCardParams({
      ...data,
      metaData,
    });
  }, [params]);

  useEffect(() => {
    processCardParams();
  }, [processCardParams]);

  // 根据 fileType 获取对应的渲染器组件
  const getRenderer = () => {
    let fileType = (cardParams?.fileType as FileType)?.toUpperCase() as FileType;
    const fileId = cardParams?.fileId;
    const fileName = cardParams?.fileName || cardParams?.metaData?.file_name;

    const VALID_FILE_TYPES: FileType[] = ["EXCEL", "JSON", "PDF", "TXT", "MARKDOWN", "IMAGE", "DOCX", "DOC"];
    if ((!fileType || !VALID_FILE_TYPES.includes(fileType)) && fileName) {
      fileType = getFileTypeFromExtension(fileName);
    }

    if (!fileType || !isValidFileId(fileId)) {
      return <div style={{ padding: 16 }}>暂不支持该文件格式预览</div>;
    }

    // Excel 特殊处理（需要额外 props）
    if (fileType === "EXCEL") {
      return (
        <ExcelRenderer
          fileId={fileId}
          sheetNames={cardParams?.metaData?.sheetSummary?.map(
            (item: any) => item?.name
          )}
          onFetchExcelData={onFetchExcelData}
          showDataAnnotation={showDataAnnotation}
          onFetchDataAnnotation={onFetchDataAnnotation}
          onSaveDataAnnotation={onSaveDataAnnotation}
        />
      );
    }

    // PDF 特殊处理（需要额外 props）
    if (fileType === "PDF") {
      return (
        <PdfRenderer
          fileId={fileId}
          onFetchPdfData={
            onFetchPdfData as () => Promise<{ blobUrl: string }>
          }
        />
      );
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
      return <DocDownloadTip fileName={fileName} downloadUrl={cardParams?.downloadUrl} />;
    }

    // 其他类型使用统一的渲染器映射
    const RendererComponent = RENDERER_MAP[fileType];
    if (RendererComponent) {
      return <RendererComponent fileId={fileId} />;
    }

    return <div style={{ padding: 16 }}>暂不支持该文件格式预览</div>;
  };

  return (
    <CardLoading loading={loading}>
      <div
        className="file-card-wrapper"
        style={{ width: "100%", height: "100%", minHeight: loading ? "210px" : "unset" }}
      >
        <ErrorBoundary>{getRenderer()}</ErrorBoundary>
      </div>
    </CardLoading>
  );
};

export default FileCard;
```

#### **index.ts**
```typescript
import FileCard from "./FileCard";

export default FileCard;
```

#### **style/index.less**
```less
@import "../../../style/variables.less";

.file-card-wrapper {
  display: flex;
  flex-direction: column;
  background: #ffffff;
  overflow: hidden;
  font-size: 14px;
  height: 100%;

  .file-card {
    display: flex;
    align-items: center;
    justify-content: space-between;
    padding: 10px 12px;
    border: 1px solid rgba(31, 35, 41, 0.08);
    border-radius: 8px;
    cursor: pointer;

    > .left-section {
      display: flex;
      align-items: center;
      gap: 12px;
      > .file-info {
        display: flex;
        flex-direction: column;
        gap: 2px;
        > .primary {
          color: @--color-text-primary;
          overflow: hidden;
          text-overflow: ellipsis;
          white-space: nowrap;
          font-size: 14px;
          max-width: 134px;
        }
        > .secondary {
          font-size: 12px;
          color: @--color-text-secondary;
        }
      }
    }
    > .right-section {
      display: flex;
      align-items: center;
      gap: 4px;
      .toft-icon {
        width: 24px;
        height: 24px;
        cursor: pointer;
        &:hover {
          background-color: rgba(115, 116, 175, 0.1);
          border-radius: 4px;
        }
      }
    }
  }

  .toft-loading-block {
    height: 100%;
    display: flex;
    align-items: center;
    justify-content: center;
    .toft-loading-children {
      height: 100%;
      width: 100%;
    }
    .toft-loading-backdrop {
      height: 100%;
      border-radius: 8px;
      background: #ffffff !important;
    }
  }
}
```

---

## **4. MarkdownCard 支撑组件（Markdown 渲染核心）**

### 📁 文件路径：
- `/Users/lianzimeng/working/chat-bot-cards/src/components/MarkdownCard/MarkdownCard.tsx`
- `/Users/lianzimeng/working/chat-bot-cards/src/components/MarkdownCard/style/index.less`
- `/Users/lianzimeng/working/chat-bot-cards/src/components/MarkdownCard/index.ts`
- `/Users/lianzimeng/working/chat-bot-cards/src/components/MarkdownCard/svg.tsx`
- `/Users/lianzimeng/working/chat-bot-cards/src/components/MarkdownCard/remarkCustomFootnoteReference.ts`
- `/Users/lianzimeng/working/chat-bot-cards/src/components/MarkdownCard/remarkFrontmatterVisible.ts`

### 📄 源码内容：

#### **MarkdownCard.tsx**
```typescript
import React, { FC, ReactNode, useMemo, useState } from "react";
import { Loading, message } from "@toft/react";
import Markdown from "react-markdown";
import Zoom from "react-medium-image-zoom";
import remarkGfm from "remark-gfm";
import remarkFrontmatter from "remark-frontmatter";
import rehypeRaw from "rehype-raw";
import { remarkFrontmatterVisible } from "./remarkFrontmatterVisible";
import { Prism as SyntaxHighlighter } from "react-syntax-highlighter";
import { materialLight } from "react-syntax-highlighter/dist/esm/styles/prism";
import "react-medium-image-zoom/dist/styles.css";
import "./style/index.less";
import { remarkCustomFootnoteReference } from "./remarkCustomFootnoteReference";
import { omit } from "lodash";
import classNames from "classnames";
import { CopiedSvg, CopySvg } from "./svg";
import CustomLink from "./components/CustomLink";
import MermaidChart from "./components/MermaidChart";

interface MarkdownCardProps {
  params: string;
  citations?: Array<any>;
  loading: boolean;
  style?: React.CSSProperties;
  disableCitations?: boolean;
  citationType?: "knowledge_item" | "website" | undefined;
  onClickInternalKnowledgeItem?: (item: any) => void;
}

function sanitizeMarkdown(input: string): string {
  const mermaidStart = input?.lastIndexOf("```mermaid");
  if (mermaidStart !== -1) {
    const mermaidEnd = input?.indexOf("```", mermaidStart + 10);
    if (mermaidEnd === -1) {
      // 没有闭合符，把 ```mermaid 改成 ```plaintext，保证能正常渲染成普通代码块
      return (
        input?.slice(0, mermaidStart) +
        "```plaintext" +
        input.slice(mermaidStart + 10)
      );
    }
  }
  return input;
}

// TODO: 单独文件
const CustomImage = ({ src, alt }: any) => (
  <Zoom>
    <img src={src || ""} alt={alt || ""} style={{ maxWidth: "100%" }} />
  </Zoom>
);

const createSupComponent =
  (
    citations: any[],
    disableCitations: boolean,
    citationType?: "knowledge_item" | "website" | undefined,
    onClickInternalKnowledgeItem?: (item: any) => void,
  ) =>
  (props: any) => (
    <CustomLink
      {...props}
      citations={citations}
      isFootNote={true}
      disableCitations={disableCitations}
      citationType={citationType}
      onClickInternalKnowledgeItem={onClickInternalKnowledgeItem}
    />
  );

const createAComponent =
  (
    citations: any[],
    disableCitations: boolean,
    citationType?: "knowledge_item" | "website" | undefined,
    onClickInternalKnowledgeItem?: (item: any) => void,
  ) =>
  (props: any) => {
    if (props?.className === "stargate-reference") {
      return (
        <CustomLink
          {...props}
          citations={citations}
          isFootNote={true}
          isCustomFootNote={true}
          oriHref={props?.href}
          disableCitations={disableCitations}
          citationType={citationType}
          onClickInternalKnowledgeItem={onClickInternalKnowledgeItem}
        />
      );
    }
    return (
      <CustomLink
        {...props}
        citations={citations}
        disableCitations={disableCitations}
        citationType={citationType}
        onClickInternalKnowledgeItem={onClickInternalKnowledgeItem}
      />
    );
  };
const ImgComponent = React.memo((props) => <CustomImage {...props} />);

const CodeComponent = React.memo((props: any) => {
  const { children, className, node, ...rest } = props;
  const match = /language-(\\w+)/.exec(className || "");

  if (!match) {
    return (
      <code {...rest} className={className}>
        {children}
      </code>
    );
  }

  if (match && match?.[1] === "mermaid") {
    const code = Array.isArray(children) ? children.join("") : String(children);
    const id = crypto.randomUUID();
    return <MermaidChart source={code.trim()} id={id} />;
  }

  const code = String(children).replace(/\\n$/, "");

  return (
    <SyntaxHighlighter
      {...omit(rest, ["ref"])}
      PreTag={withLanguageInject(PreTag, match[1], code)}
      language={match[1]}
      style={materialLight}
    >
      {code}
    </SyntaxHighlighter>
  );
});

const MarkdownCard: React.FC<MarkdownCardProps> = (props) => {
  const {
    params,
    citations,
    loading,
    disableCitations = false,
    style,
    citationType,
    onClickInternalKnowledgeItem,
  } = props;

  // Mermaid语法预处理，防止语法错误渲染报错
  const safeParams = useMemo(
    () => (params ? sanitizeMarkdown(params) : params),
    [params],
  );

  const markdownRemarkPlugins = useMemo(
    () => [
      remarkGfm,
      remarkFrontmatter,
      remarkFrontmatterVisible,
      remarkCustomFootnoteReference,
    ],
    [],
  );

  const markdownRehypePlugins = useMemo(() => [rehypeRaw], []);

  const SupComponent = useMemo(
    () =>
      createSupComponent(
        citations || [],
        disableCitations,
        citationType,
        onClickInternalKnowledgeItem,
      ),
    [citations, disableCitations, citationType, onClickInternalKnowledgeItem],
  );

  const AComponent = useMemo(
    () =>
      createAComponent(
        citations || [],
        disableCitations,
        citationType,
        onClickInternalKnowledgeItem,
      ),
    [citations, disableCitations, citationType, onClickInternalKnowledgeItem],
  );

  const markdownComponents = useMemo(
    () => ({
      sup: SupComponent,
      a: AComponent,
      img: ImgComponent,
      code: CodeComponent,
    }),
    [SupComponent, AComponent, ImgComponent, CodeComponent],
  );

  const ReactMarkdownMemo = useMemo(() => {
    return (
      <Markdown
        remarkPlugins={markdownRemarkPlugins}
        rehypePlugins={markdownRehypePlugins}
        components={markdownComponents}
      >
        {safeParams}
      </Markdown>
    );
  }, [
    safeParams,
    markdownRemarkPlugins,
    markdownRehypePlugins,
    markdownComponents,
  ]);

  return (
    <div
      className="markdown-card"
      style={{
        ...style,
        width: loading ? "320px" : "auto",
        height: loading ? "60px" : "fit-content",
      }}
    >
      {/** @ts-ignore */}
      <Loading loading={loading}>{ReactMarkdownMemo}</Loading>
    </div>
  );
};

// TODO: 单独文件
interface PreTagProps {
  children?: ReactNode;
  language: string;
  code: string;
}

function PreTag({ children, language, code }: PreTagProps) {
  const [copied, setCopied] = useState(false);
  const handleCopy = async () => {
    if (copied) return;
    navigator.clipboard?.writeText(code).then(() => {
      setCopied(true);
      message.success({ message: "内容已复制到剪切板" });
      setTimeout(() => {
        setCopied(false);
      }, 2000);
    });
  };

  return (
    <div className={classNames("highlight", `highlight-${language}`)}>
      <div className="highlight-header">
        <div className="highlight-title">{language}</div>
        <div className="highlight-copy" onClick={handleCopy}>
          {copied ? <CopiedSvg /> : <CopySvg />}复制
        </div>
      </div>
      <div className="highlight-source">{children}</div>
    </div>
  );
}

function withLanguageInject<P extends Partial<PreTagProps>>(
  Comp: React.ComponentType<P>,
  language: string,
  code: string,
) {
  return function EnhancedComp(props: P) {
    return <Comp {...props} language={language} code={code} />;
  };
}

export default React.memo(MarkdownCard);
```

#### **style/index.less**
```less
@import "../../../style/variables.less";

.markdown-card {
  display: flex;
  flex-direction: column;
  border-radius: 8px;
  background: #ffffff;
  // overflow: hidden;
  font-size: 14px;
  line-height: 24px;
  color: #1f2329;
  font-family:
    PingFang SC,
    sans-serif;

  .toft-loading-block {
    height: 100%;
    .toft-loading-backdrop {
      background: #f5f9ff !important;
    }
  }

  &:first-of-type .toft-loading-children > :first-child {
    margin-block-start: 0 !important;
  }

  &:last-of-type .toft-loading-children > :last-child {
    margin-block-end: 0 !important;
  }

  pre {
    .highlight {
      .highlight-source {
        overflow: auto;
      }
    }
  }

  p {
    margin-block-start: 8px;
    margin-block-end: 8px;
  }

  h1 {
    font-size: 26px;
    line-height: 30px;
    color: #000000;
  }
  h2 {
    font-size: 20px;
    line-height: 24px;
    color: #000000;
    margin-block-start: 24px;
  }
  h3 {
    font-size: 16px;
    line-height: 20px;
    color: #000000;
    margin-block-start: 20px;
  }
  h4 {
    font-size: 14px;
    line-height: 20px;
    color: #000000;
    margin-block-start: 20px;
  }
  h5 {
    font-size: 12px;
    line-height: 18px;
    color: #000000;
  }
  h1,
  h2,
  h3,
  h4,
  h5 {
    margin-block-end: 12px;
  }

  hr {
    border: 1px solid #ebedf0;
  }

  blockquote {
    margin: 0;
    margin-bottom: 12px;
    padding: 0 1em;
    border-left: 4px solid #ebedf0;
    color: #95999e;
  }

  .highlight {
    background-color: #f9fafc;
    border-radius: 8px;
    border: 1px solid @--color-line-1;
    overflow: hidden;
    font-size: 13px;
    line-height: 18px;
    &-header {
      background-color: #f2f5f7;
      padding: 4px 12px;
      display: flex;
      align-items: center;
      justify-content: space-between;
    }

    &-copy {
      font-size: 12px;
      line-height: 16px;
      padding: 4px;
      display: flex;
      cursor: pointer;
      border-radius: 6px;
      align-items: center;
      gap: 4px;
      svg {
        width: 15px;
        height: 15px;
      }

      &:hover {
        background-color: rgba(115, 116, 175, 0.1);
      }
    }

    &-source {
      padding: 12px;
    }
  }

  code {
    font-family: "Fira Code", monospace !important;
  }

  pre code {
    white-space: pre-wrap;
    word-break: break-all;
  }

  li {
    margin: unset;
    list-style-type: revert-layer;
    > ul {
      margin: 0;
    }
  }

  ul,
  ol {
    line-height: 26px;
    padding-inline-start: 2em;
    list-style-type: revert-layer;
    margin: 12px 0;
  }

  ol li {
    list-style-type: revert-layer;
  }

  ol + ul {
    margin-top: -6px;
  }

  a {
    color: #4649ff;
    background-color: transparent;
    outline: none;
    text-decoration: none;
    transition: color 0.1s cubic-bezier(0.9, 0.25, 0.08, 0.83);
  }

  table {
    border-spacing: 0;
    border-collapse: collapse;
    display: block;
    width: max-content;
    max-width: 100%;
    overflow: auto;
    font-size: 13px;
    line-height: 18px;
    border-radius: 8px;
    border: 1px solid #ebedf0;
    margin-block-start: 12px;
  }

  thead + tbody,
  tr + tr {
    border-top: 1px solid #1f232914;
  }

  th + th,
  td + td {
    border-left: 1px solid #1f232914;
  }

  th {
    text-align: start;
    background-color: #f5f7fa;
    padding: 13px 12px;
    font-weight: 500;
    white-space: nowrap;
  }
  td {
    padding: 12px 12px;
  }

  table img {
    background-color: transparent;
  }

  img {
    border-radius: 8px;
  }

  .citation-link {
    display: inline-flex;
    justify-content: center;
    align-items: center;
    margin: 0 0 0 4px;
    width: 20px;
    height: 20px;
    border-radius: 50%;
    background-color: rgba(235, 237, 240, 1);
    color: @--color-text-secondary;
    font-size: 12px;
    line-height: 16px;
    text-decoration: none;
    cursor: pointer;
    &:hover,
    &.active {
      background-color: rgba(70, 73, 255, 0.2);
      color: rgba(70, 73, 255, 1);
      .toft-icon {
        path {
          fill: rgba(70, 73, 255, 1);
        }
      }
    }
  }
}
```

#### **index.ts**
```typescript
import MarkdownCard from "./MarkdownCard";

export default MarkdownCard;
```

#### **svg.tsx**
```typescript
export const CopySvg = () => (
  <svg width="12" height="12" viewBox="0 0 12 12" fill="none" xmlns="http://www.w3.org/2000/svg">
    <g opacity="0.5">
      <rect
        x="0.6875"
        y="3.1875"
        width="8.125"
        height="8.125"
        rx="1.25"
        stroke="#1F2329"
        stroke-width="1.25"
      />
      <path
        d="M9.125 8.81226H10.0625C10.7529 8.81226 11.3125 8.25261 11.3125 7.56226V1.93726C11.3125 1.2469 10.7529 0.687256 10.0625 0.687256H4.4375C3.74714 0.687256 3.1875 1.2469 3.1875 1.93726V2.87476"
        stroke="#1F2329"
        stroke-width="1.25"
      />
    </g>
  </svg>
);

export const CopiedSvg = () => (
  <svg width="19" height="13" viewBox="0 0 19 13" fill="none" xmlns="http://www.w3.org/2000/svg">
    <path
      d="M17.5872 0.588711L18.0822 1.08367C18.204 1.20557 18.2725 1.37087 18.2725 1.54317C18.2725 1.71557 18.204 1.88077 18.0822 2.00267L7.58173 12.5032C7.40473 12.6804 7.16783 12.7846 6.91763 12.7955C6.66753 12.8063 6.42243 12.7229 6.23073 12.5617L6.16723 12.5032L0.19026 6.52617C0.0684399 6.40427 0 6.23907 0 6.06667C0 5.89437 0.0684399 5.72907 0.19026 5.60717L0.68526 5.11217C0.80715 4.99037 0.97243 4.92197 1.14476 4.92197C1.3171 4.92197 1.48237 4.99037 1.60426 5.11217L6.87423 10.3822L16.6677 0.588711C16.7281 0.528311 16.7998 0.480401 16.8786 0.447711C16.9575 0.415021 17.0421 0.398201 17.1275 0.398201C17.2129 0.398201 17.2974 0.415021 17.3763 0.447711C17.4552 0.480401 17.5269 0.528311 17.5872 0.588711Z"
      fill="#5C6066"
    />
  </svg>
);
```

#### **remarkCustomFootnoteReference.ts**
```typescript
// remarkCustomFootnoteReference.ts
import { visit } from "unist-util-visit";

export function remarkCustomFootnoteReference() {
  return (tree: any) => {
    visit(tree, "text", (node, index, parent) => {
      const footnoteRegex = /\[\^([^\]]+)\]/g;
      let match;
      const newChildren = [];

      let lastIndex = 0;
      while ((match = footnoteRegex.exec(node.value)) !== null) {
        const [fullMatch, refId] = match;
        const start = match.index;

        // 普通文本部分
        if (start > lastIndex) {
          newChildren.push({
            type: "text",
            value: node.value.slice(lastIndex, start),
          });
        }

        newChildren.push({
          type: "link",
          url: `#stargate-reference`,
          data: {
            hProperties: {
              className: "stargate-reference",
            },
          },
          children: [{ type: "text", value: refId }],
        });

        lastIndex = start + fullMatch.length;
      }

      // 剩余文本部分
      if (lastIndex < node.value.length) {
        newChildren.push({
          type: "text",
          value: node.value.slice(lastIndex),
        });
      }

      // 替换当前节点为新节点列表
      if (newChildren.length > 0 && parent && Array.isArray(parent.children)) {
        parent.children.splice(index!, 1, ...newChildren);
      }
    });
  };
}
```

#### **remarkFrontmatterVisible.ts**
```typescript
import { visit } from "unist-util-visit";
import type { Root } from "mdast";

/**
 * remark 插件：把 frontmatter 节点转换为可见的代码块（yaml/toml），
 * 避免不加 remark-frontmatter 时 closing `---` 被误解析为 setext H2。
 */
export function remarkFrontmatterVisible() {
  return (tree: Root) => {
    visit(tree, (node: any, index, parent: any) => {
      if (
        (node.type === "yaml" || node.type === "toml") &&
        parent &&
        index !== null &&
        index !== undefined
      ) {
        const lang = node.type === "yaml" ? "yaml" : "toml";
        // 替换为 code 节点，保留原始内容，渲染为代码块
        parent.children[index] = {
          type: "code",
          lang,
          value: node.value,
        };
      }
    });
  };
}
```

---

## **5. MarkdownCard 子组件**

### 📁 CustomLink 组件

#### **路径:**
- `/Users/lianzimeng/working/chat-bot-cards/src/components/MarkdownCard/components/CustomLink/index.tsx`
- `/Users/lianzimeng/working/chat-bot-cards/src/components/MarkdownCard/components/CustomLink/index.less`

#### **index.tsx**
```typescript
import { Icon, Popover } from "@toft/react";
import React, { useEffect, useMemo } from "react";
import classNames from "classnames";
import { ReactSVG } from "react-svg";
import { Ellipsis } from "@toft/react-pro";
import dayjs from "dayjs";
import FlexBox from "../../../FlexBox";
import KnowledgeSvg from "../../../../assets/knowledge.svg";

import "./index.less";

const CustomLink = ({
  href,
  children,
  citations,
  isFootNote,
  isCustomFootNote,
  oriHref,
  disableCitations,
  citationType,
  onClickInternalKnowledgeItem,
}: {
  href?: string;
  children?: React.ReactNode;
  citations?: any[];
  isFootNote?: boolean;
  isCustomFootNote?: boolean;
  oriHref?: string;
  disableCitations?: boolean;
  citationType?: "knowledge_item" | "website" | undefined;
  onClickInternalKnowledgeItem?: (item: any) => void;
}) => {
  const showCitation = useMemo(() => {
    const childText = typeof children === "string" ? children : "";
    return (
      isFootNote ||
      (Array.isArray(citations) &&
        citations?.length > 0 &&
        childText.split("_")?.[0] === "CONDITIONS")
    );
  }, [children, citations]);

  const targetId = isCustomFootNote
    ? typeof children === "string"
      ? children
      : String(children)
    : isFootNote
      ? React.isValidElement(children)
        ? (children.props as any)?.children
        : undefined
      : typeof children === "string"
        ? children.split("_")?.[1]
        : undefined;
  const targetItem =
    targetId && citations
      ? citations.find((item: any) => item?.id === targetId)
      : undefined;
  const targetHref = targetItem?.href || undefined;
  const targetIndex = isCustomFootNote
    ? parseInt(typeof children === "string" ? children : String(children))
    : targetId && citations
      ? citations.findIndex((item: any) => item?.id === targetId) + 1
      : -1;

  const popoverContent = (
    <FlexBox
      className="custom-link-popover-content-wrapper"
      direction="column"
      gap={8}
    >
      {targetItem?.summary && (
        <span className="custom-link-popover-content-wrapper__snippet">
          {targetItem?.summary}
        </span>
      )}
      <FlexBox
        className="custom-link-popover-content-wrapper__bottom-section"
        justify="between"
        align="center"
      >
        <FlexBox
          align="center"
          gap={8}
          style={{ lineHeight: "17px", width: "100%" }}
        >
          {targetItem?.icon ? (
            <img
              src={targetItem?.icon}
              style={{ borderRadius: "50%", width: 12, height: 12 }}
            />
          ) : (
            <Icon type="link" style={{ fontSize: 12, marginRight: "4px" }} />
          )}
          <Ellipsis>{targetItem?.site_name || targetItem?.href}</Ellipsis>
        </FlexBox>
        {targetItem?.date_published && (
          <span>{dayjs(targetItem?.date_published).format("YYYY-MM-DD")}</span>
        )}
      </FlexBox>
    </FlexBox>
  );

  return (
    <>
      {/* 内部知识库的引用额外处理 */}
      {citationType === "knowledge_item" ||
      targetItem?.type === "knowledge_item" ? (
        <div
          className={classNames("citation-link", {
            active: targetItem?.active,
          })}
          onClick={() => onClickInternalKnowledgeItem?.(targetItem)}
          style={{ pointerEvents: disableCitations ? "none" : "auto" }}
        >
          <Icon
            svg={<ReactSVG src={KnowledgeSvg} />}
            style={{ fontSize: "12px" }}
          />
        </div>
      ) : showCitation ? (
        <Popover
          placement="bottomCenter"
          title={<Ellipsis>{targetItem?.title || targetItem?.name}</Ellipsis>}
          content={popoverContent}
          showArrow={false}
          popLayer={{ className: "custom-link-popover" }}
        >
          <a
            className="custom-link"
            href={targetHref || oriHref || "#"}
            target="_blank"
            rel="noopener noreferrer"
            style={{ pointerEvents: disableCitations ? "none" : "auto" }}
          >
            <span className="citation-link">
              {targetIndex >= 0 ? targetIndex : "?"}
            </span>
          </a>
        </Popover>
      ) : (
        <a href={href || "#"} target="_blank" rel="noopener noreferrer">
          {typeof children === "string" ? children : "Invalid link"}
        </a>
      )}
    </>
  );
};

export default CustomLink;
```

#### **index.less**
```less
@import "../../../../style/variables.less";

.custom-link-popover {
  .toft-popover-inner {
    box-shadow: 0px 6px 12px 0px @--color-box-shadow-3;
    border-radius: 8px;
    border: 1px solid @--color-line-1;
    .toft-popover-title {
      padding-bottom: 0;
      margin-bottom: 8px;
    }

    .custom-link-popover-content-wrapper {
      &__snippet {
        display: -webkit-box;
        -webkit-box-orient: vertical;
        -webkit-line-clamp: 2;
        overflow: hidden;
        text-overflow: ellipsis;
        word-break: break-word;
        color: @--color-text-tertiary;
        font-size: 12px;
        line-height: 20px;
      }
      &__bottom-section {
        color: @--color-text-tertiary;
        font-size: 12px;
        .separator {
          width: 1px;
          height: 12px;
          background-color: @--color-line-1;
        }
      }
    }
  }
}

.knowledge-item-wrapper {
}
```

### 📁 MermaidChart 组件

#### **路径:**
- `/Users/lianzimeng/working/chat-bot-cards/src/components/MarkdownCard/components/MermaidChart/index.tsx`

#### **index.tsx**
```typescript
import mermaid from "mermaid";
import { useEffect } from "react";

mermaid.initialize({});

const Mermaid = ({ source, id }: { source: string; id: string }) => {
  console.log("source:::", id, source);

  useEffect(() => {
    const el = document.getElementById(`mermaid-diagram-${id}`);
    if (!el) return;

    el.removeAttribute("data-processed");

    const renderChart = async () => {
      try {
        mermaid.parse(source);

        // 校验成功才渲染
        const { svg } = await mermaid.render(`svg-${id}`, source, el);
        el.innerHTML = svg;
      } catch (err) {
        console.error("Mermaid 语法错误:::", err);
      }
    };

    renderChart();
  }, [source, id]);

  return (
    <div className="mermaid" id={`mermaid-diagram-${id}`}>
      {source}
    </div>
  );
};

export default Mermaid;
```

---

## **6. 其他预览器组件（补充信息）**

### 📁 TextRenderer（文本预览）
**路径:** `/Users/lianzimeng/working/chat-bot-cards/src/components/FileCard/TextRenderer/index.tsx`

### 📁 JsonRenderer（JSON预览）
**路径:** `/Users/lianzimeng/working/chat-bot-cards/src/components/FileCard/JsonRenderer/index.tsx`

### 📁 ExcelRenderer（Excel预览）
**路径:** `/Users/lianzimeng/working/chat-bot-cards/src/components/FileCard/ExcelRenderer/index.tsx`

### 📁 ImageRenderer（图片预览）
**路径:** `/Users/lianzimeng/working/chat-bot-cards/src/components/FileCard/ImageRenderer/index.tsx`

### 📁 DocxRenderer（Word预览）
**路径:** `/Users/lianzimeng/working/chat-bot-cards/src/components/FileCard/DocxRenderer/index.tsx`

---

## **核心技术栈**

| 功能 | 技术库 |
|------|--------|
| **PDF预览** | `react-pdf` + `pdfjs-dist` |
| **Markdown渲染** | `react-markdown` + `remark-gfm` + `rehype-raw` |
| **代码高亮** | `react-syntax-highlighter` + `prism` |
| **图表渲染** | `mermaid` |
| **图片缩放** | `react-medium-image-zoom` |
| **Word预览** | `docx-preview` |
| **UI组件库** | `@toft/react` |
| **样式** | `Less` |

---

## **文件结构总结**

```
/Users/lianzimeng/working/chat-bot-cards/src/components/
├── FileCard/
│   ├── FileCard.tsx                    # 主组件（路由器）
│   ├── index.ts                        # 导出文件
│   ├── PdfRenderer/
│   │   ├── index.tsx                   # PDF预览组件
│   │   └── index.less                  # PDF样式
│   ├── MarkdownRenderer/
│   │   ├── index.tsx                   # Markdown预览组件
│   │   └── index.less                  # Markdown样式
│   ├── TextRenderer/
│   │   ├── index.tsx
│   │   └── index.less
│   ├── JsonRenderer/
│   │   ├── index.tsx
│   │   └── index.less
│   ├── ExcelRenderer/
│   │   ├── index.tsx
│   │   └── index.less
│   ├── ImageRenderer/
│   │   ├── index.tsx
│   │   └── index.less
│   ├── DocxRenderer/
│   │   ├── index.tsx
│   │   └── index.less
│   └── style/
│       └── index.less                  # FileCard主样式
├── MarkdownCard/
│   ├── MarkdownCard.tsx                # Markdown渲染核心
│   ├── index.ts
│   ├── svg.tsx                         # 复制按钮SVG图标
│   ├── remarkCustomFootnoteReference.ts # Remark插件
│   ├── remarkFrontmatterVisible.ts     # Remark插件
│   ├── style/
│   │   └── index.less                  # Markdown样式
│   └── components/
│       ├── CustomLink/
│       │   ├── index.tsx               # 链接/引用组件
│       │   └── index.less
│       └── MermaidChart/
│           └── index.tsx               # Mermaid图表组件
```

---

以上就是项目中 **FileCard 组件**下所有 **PDF 和 Markdown 文件预览组件**的完整源码和详细信息！所有文件都已提供了完整的代码内容和样式文件。
