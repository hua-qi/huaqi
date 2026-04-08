---
session_id: agent-86d32e4a
date: 2026-03-30
time_start: 2026-03-30T13:22:49.601Z
time_end: 2026-03-30T13:23:28.706Z
project: users-lianzimeng-working-kd-toft-chat-bot
git_branch: feat-multi-modal-v2
---

[2026-03-30 21:22:49] [user]: Show me the implementation of `FileCardImagePreviewerWrapper` in `src/modules/StargateChatBot/components/CardsList/index.tsx`. I want to replace the `onDownload` logic to use the blob URL directly.
[2026-03-30 21:23:28] [assistant]: Here's the `FileCardImagePreviewerWrapper` implementation (lines 19-60):

```tsx
const FileCardImagePreviewerWrapper: React.FC<{
  fileId: string;
  fileName?: string;
  isEncrypted: boolean;
  downloadUrl?: string;
  preview: boolean;
  onClose: () => void;
  children: React.ReactNode;
}> = ({ fileId, fileName, isEncrypted, downloadUrl, preview, onClose, children }) => {
  const [imageBlobUrl, setImageBlobUrl] = useState<string | null>(null);

  const { previewFile, downloadFile } = useFile({
    fileId,
    fileName,
    isEncrypted
  });

  useEffect(() => {
    if (!fileId || !preview) return; // Only fetch when preview is opened

    let revoked = false;

    previewFile({}, FileCardTypeList.IMAGE).then((blob) => {
      if (revoked) return;
      if (blob instanceof Blob) {
        const url = URL.createObjectURL(blob);
        setImageBlobUrl(url);
      }
    });

    return () => {
      revoked = true;
    };
  }, [fileId, preview, previewFile]);

  // Cleanup object URL when component unmounts or URL changes
  useEffect(() => {
    return () => {
      if (imageBlobUrl) {
        URL.revokeObjectURL(imageBlobUrl);
      }
    };
  }, [imageBlobUrl]);

  return (
    <FileCardImagePreviewer
      preview={preview}
      onDownload={() => downloadFile(downloadUrl)}  // <-- Current: uses downloadUrl
      onClose={onClose}
      previewSrc={imageBlobUrl || ''}
    >
      {children}
    </FileCardImagePreviewer>
  );
};
```

## To Replace `onDownload` to Use Blob URL Directly

You need to modify the `onDownload` handler. There's already a `handleDownloadImage` function defined in the `CardsList` component (line 178-184) that you can use as a pattern:

```tsx
const handleDownloadImage = (url: string) => {
  const link = document.createElement('a');
  link.href = url;
  link.download = 'image.png';
  document.body.appendChild(link);
  link.click();
  document.body.removeChild(link);
};
```

### Proposed Change:

Modify the `FileCardImagePreviewerWrapper` component to:

```tsx
return (
  <FileCardImagePreviewer
    preview={preview}
    onDownload={() => {
      if (imageBlobUrl) {
        const link = document.createElement('a');
        link.href = imageBlobUrl;
        link.download = fileName || 'image.png';
        document.body.appendChild(link);
        link.click();
        document.body.removeChild(link);
      }
    }}
    onClose={onClose}
    previewSrc={imageBlobUrl || ''}
  >
    {children}
  </FileCardImagePreviewer>
);
```

This change:
1. Uses the already-fetched `imageBlobUrl` instead of calling `downloadFile(downloadUrl)`
2. Preserves the original filename via `fileName` prop
3. Avoids making a separate download request since the blob is already in memory

Would you like me to help you make this change, or do you need to see the `FileCardImagePreviewer` component implementation as well?
