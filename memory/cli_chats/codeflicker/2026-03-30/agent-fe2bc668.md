---
session_id: agent-fe2bc668
date: 2026-03-30
time_start: 2026-03-30T09:50:49.206Z
time_end: 2026-03-30T09:51:07.755Z
project: users-lianzimeng-working-kd-toft-chat-bot
git_branch: feat-multi-modal-v2
---

[2026-03-30 17:50:49] [user]: Look at how download is handled in `FileCardImagePreviewerWrapper` inside `src/modules/StargateChatBot/components/CardsList/index.tsx`.
Currently it uses `onDownload={downloadFile}`. `downloadFile` comes from `useFile` hook.
Check `src/hooks/useFile.ts` to see what `downloadFile` expects. Does it expect an argument? Does passing a React synthetic event (from button click) to it cause issues?
Also check how `FileCard/index.tsx` implements the download functionality. How does it call `downloadFile`?
We likely need to wrap it: `onDownload={() => downloadFile()}` or similar, to avoid passing the event object as an argument.
[2026-03-30 17:50:53] [assistant]: I'll investigate the download handling across these files. Let me read them to understand the issue.
[2026-03-30 17:51:07] [assistant]: ## Summary

You're absolutely correct about the issue. Here's the analysis:

### The Problem

**In `CardsList/index.tsx` (line 54):**
```tsx
<FileCardImagePreviewer
  preview={preview}
  onDownload={downloadFile}  // <-- PROBLEM: passing function directly
  onClose={onClose}
  previewSrc={imageBlobUrl || ''}
>
```

**The `downloadFile` function from `useFile.ts` (lines 318-330) expects:**
```ts
const downloadFile = async (downloadUrl?: string) => {
  // ...
  if (!downloadUrl) return;
  normalDownloadFile(downloadUrl, fileName || '');
};
```

When `onDownload={downloadFile}` is passed directly and the download button is clicked, **React's synthetic event object will be passed as the first argument** instead of a URL string. This is problematic because:
- The event object is truthy, so `if (!downloadUrl) return;` won't guard against it
- `normalDownloadFile(event, fileName)` will fail trying to use an event as a URL

### The Correct Pattern (from `FileCard/index.tsx`, lines 72-74)

```tsx
const handleDownloadImage = useCallback(() => {
  downloadFile(item?.content?.downloadUrl);
}, [item?.content?.downloadUrl]);

// Then used as:
<FileCardImagePreviewer
  onDownload={handleDownloadImage}  // <-- Wrapped in a callback
  // ...
/>
```

`FileCard/index.tsx` correctly wraps `downloadFile` in a callback function that calls it with no arguments (for encrypted files) or with the download URL.

### Recommended Fix

In `CardsList/index.tsx`, change line 54 from:
```tsx
onDownload={downloadFile}
```

To:
```tsx
onDownload={() => downloadFile()}
```

Or if there's a download URL available for non-encrypted files:
```tsx
onDownload={() => downloadFile(downloadUrl)}
```
