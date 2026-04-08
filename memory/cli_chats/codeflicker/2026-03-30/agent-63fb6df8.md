---
session_id: agent-63fb6df8
date: 2026-03-30
time_start: 2026-03-30T08:14:53.057Z
time_end: 2026-03-30T08:15:57.316Z
project: users-lianzimeng-working-kd-toft-chat-bot
git_branch: feat-multi-modal-v2
---

[2026-03-30 16:14:53] [user]: The user provided a screenshot showing that `FileCardImagePreviewer` is displaying the DOM element (the card itself) instead of the actual image. 
Look at `src/common-components/FileCard/FileCardImagePreviewer.tsx` to understand why this happens. Look at how `previewSrc` is used and what happens when it is missing or invalid.
Then look at `src/modules/StargateChatBot/components/CardsList/index.tsx` to see what `previewSrc` we are passing for `FILE_CARD`. For `FILE_CARD`, the URL might not be in `loadUrl` or `url`. It might require fetching via `useFile` using `fileId`, similar to how `FileCard/index.tsx` does it.
Find how `FileCard/index.tsx` gets the image URL and design a fix for `CardsList/index.tsx`.
[2026-03-30 16:15:05] [assistant]: I'll investigate the issue by reading the relevant files to understand the preview logic and how the image URL should be fetched.
[2026-03-30 16:15:17] [assistant]: Now let me look at the `useFile` hook to understand its interface:
[2026-03-30 16:15:57] [assistant]: Now I have a complete understanding of the issue. Let me summarize my findings:

## Root Cause Analysis

### In `FileCardImagePreviewer.tsx` (lines 143-152):
```tsx
{previewSrc ? (
   <img src={previewSrc} ... />
) : (
   children  // <-- This is why the DOM element shows instead of the image!
)}
```
When `previewSrc` is falsy (null/undefined/empty string), it falls back to rendering `{children}` - which is the card thumbnail itself.

### In `CardsList/index.tsx` (lines 160-170):
For `FILE_CARD`, the code uses:
```tsx
previewSrc={item?.card?.content?.loadUrl || item?.card?.content?.url}
```
But for `FILE_CARD`, the image URL is **not** in `loadUrl` or `url`. It must be fetched via API using `fileId`.

### How `FileCard/index.tsx` solves it (lines 50-67):
```tsx
const { previewFile } = useFile({
  fileId: isImageFile ? item?.content?.fileId : undefined,
  fileName: item?.content?.fileName,
  isEncrypted: item?.content?.encryptType === 'client'
});

useEffect(() => {
  if (!isImageFile || !item?.content?.fileId) return;
  previewFile({}, FileCardTypeList.IMAGE).then((blob) => {
    if (blob instanceof Blob) {
      blobUrl = URL.createObjectURL(blob);
      setImageBlobUrl(blobUrl);  // <-- This becomes previewSrc
    }
  });
}, [isImageFile, item?.content?.fileId]);
```

## Proposed Fix for `CardsList/index.tsx`

**Option A (Recommended): Extract to a sub-component**

Create a `FileCardImagePreviewerWrapper` component that handles the blob URL fetching internally, similar to how `FileCard/index.tsx` does it:

```tsx
// New component to handle FILE_CARD image preview
interface FileCardImagePreviewerWrapperProps {
  fileId: string;
  fileName?: string;
  isEncrypted: boolean;
  preview: boolean;
  onClose: () => void;
  children: React.ReactNode;
}

const FileCardImagePreviewerWrapper: React.FC<FileCardImagePreviewerWrapperProps> = ({
  fileId,
  fileName,
  isEncrypted,
  preview,
  onClose,
  children
}) => {
  const [imageBlobUrl, setImageBlobUrl] = useState<string | null>(null);
  
  const { previewFile, downloadFile } = useFile({
    fileId,
    fileName,
    isEncrypted
  });

  useEffect(() => {
    if (!fileId) return;
    let revoked = false;
    let blobUrl: string | null = null;
    
    previewFile({}, FileCardTypeList.IMAGE).then((blob) => {
      if (revoked) return;
      if (blob instanceof Blob) {
        blobUrl = URL.createObjectURL(blob);
        setImageBlobUrl(blobUrl);
      }
    });
    
    return () => {
      revoked = true;
      if (blobUrl) URL.revokeObjectURL(blobUrl);
    };
  }, [fileId]);

  const handleDownload = useCallback(() => {
    downloadFile(); // uses fileId internally
  }, [downloadFile]);

  return (
    <FileCardImagePreviewer
      preview={preview}
      onDownload={handleDownload}
      onClose={onClose}
      previewSrc={imageBlobUrl}
    >
      {children}
    </FileCardImagePreviewer>
  );
};
```

Then use it in `CardsList`:
```tsx
{isImageFile && item?.card?.content?.fileId ? (
  <FileCardImagePreviewerWrapper
    key={item?.card?.index || item?.messageId}
    fileId={item?.card?.content?.fileId}
    fileName={item?.card?.content?.fileName}
    isEncrypted={item?.card?.content?.encryptType === 'client'}
    preview={previewingCardId === currentCardId}
    onClose={() => setPreviewingCardId(null)}
  >
    {thumbnail}
  </FileCardImagePreviewerWrapper>
) : (
  thumbnail
)}
```

**Key differences from current implementation:**
1. Uses `useFile` hook with `fileId` to fetch the image content
2. Calls `previewFile({}, FileCardTypeList.IMAGE)` to get the blob
3. Creates a blob URL via `URL.createObjectURL(blob)` for `previewSrc`
4. Properly handles cleanup with `URL.revokeObjectURL` in the effect cleanup

This mirrors exactly how `FileCard/index.tsx` works and will ensure the actual image is displayed in the previewer instead of the DOM element.
