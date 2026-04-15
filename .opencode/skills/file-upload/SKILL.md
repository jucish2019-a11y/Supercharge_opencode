---
name: file-upload
description: Implement file uploads — drag-and-drop, progress tracking, image optimization, resumable uploads, storage integration (S3, GCS, Cloudinary), and security
---

## What I do

I implement file upload systems that are secure, performant, and user-friendly:

- **Upload patterns** — Direct upload, presigned URL, chunked upload, resumable upload
- **UI/UX** — Drag-and-drop, progress bars, preview thumbnails, multi-file, paste
- **Image optimization** — Resize, compress, format conversion, responsive variants
- **Storage** — S3, GCS, Cloudflare R2, Cloudinary integration
- **Security** — File type validation, size limits, virus scanning, signed URLs
- **Performance** — Chunked uploads, parallel upload, resume on failure

## When to use me

Use this skill when:
- Building file upload functionality (images, documents, videos)
- Adding drag-and-drop upload areas
- Optimizing image uploads (resize, compress, responsive)
- Implementing progress bars and upload status
- Choosing between direct upload and presigned URL patterns
- Setting up file storage (S3, GCS, Cloudinary)
- Handling large file uploads (chunked, resumable)

## Upload architecture patterns

### Pattern 1: Direct upload to server (simple, small files)

```
Client → Server API → S3/GCS

Best for: Files < 5MB, simple apps, prototype
Cons: Server bandwidth, server as bottleneck
```

### Pattern 2: Presigned URL upload (recommended)

```
Client → Server API (get presigned URL) → Client uploads directly to S3 → Client notifies server

Best for: Most production apps, files 5MB-100MB
Pros: No server bandwidth, direct to storage, parallel uploads
```

### Pattern 3: Chunked/Resumable upload (large files)

```
Client → Split into chunks → Upload chunks in parallel to S3 → Server assembles

Best for: Files > 100MB, video uploads, unreliable connections
Pros: Resumes on failure, parallel upload, progress tracking per chunk
```

### Pattern 4: Upload service (Cloudinary, Uploadthing)

```
Client → Upload service SDK → CDN

Best for: Quick implementation, image-heavy apps, don't want to manage storage
Pros: No backend code, built-in transforms, CDN delivery
Cons: Cost, vendor lock-in, less control
```

## Presigned URL upload (recommended)

### Server: Generate presigned URL

```ts
import { S3Client, PutObjectCommand } from '@aws-sdk/client-s3';
import { getSignedUrl } from '@aws-sdk/s3-request-presigner';

const s3 = new S3Client({
  region: process.env.AWS_REGION,
  credentials: {
    accessKeyId: process.env.AWS_ACCESS_KEY_ID!,
    secretAccessKey: process.env.AWS_SECRET_ACCESS_KEY!,
  },
});

const ALLOWED_TYPES = [
  'image/jpeg', 'image/png', 'image/webp', 'image/gif',
  'application/pdf',
  'video/mp4',
];

const MAX_FILE_SIZE = 10 * 1024 * 1024; // 10MB

app.post('/api/upload/presign', async (req, res) => {
  const { filename, contentType } = req.body;

  // Validate
  if (!ALLOWED_TYPES.includes(contentType)) {
    return res.status(400).json({ error: 'File type not allowed' });
  }

  const key = `uploads/${req.user.id}/${Date.now()}-${filename}`;

  const command = new PutObjectCommand({
    Bucket: process.env.S3_BUCKET!,
    Key: key,
    ContentType: contentType,
  });

  const uploadUrl = await getSignedUrl(s3, command, { expiresIn: 300 }); // 5 min

  res.json({
    uploadUrl,
    key,
    publicUrl: `https://${process.env.S3_BUCKET}.s3.${process.env.AWS_REGION}.amazonaws.com/${key}`,
  });
});
```

### Client: Upload with progress

```tsx
function useFileUpload() {
  const [uploads, setUploads] = useState<Map<string, UploadState>>(new Map());

  const upload = async (file: File) => {
    const id = crypto.randomUUID();

    // Initialize upload state
    setUploads(prev => new Map(prev).set(id, {
      file, progress: 0, status: 'pending' as const,
    }));

    try {
      // Step 1: Get presigned URL
      setUploads(prev => new Map(prev).set(id, { ...prev.get(id)!, status: 'presigning' }));

      const { uploadUrl, key, publicUrl } = await fetch('/api/upload/presign', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          filename: file.name,
          contentType: file.type,
        }),
      }).then(r => r.json());

      // Step 2: Upload directly to S3
      setUploads(prev => new Map(prev).set(id, { ...prev.get(id)!, status: 'uploading' }));

      await new Promise<void>((resolve, reject) => {
        const xhr = new XMLHttpRequest();
        xhr.open('PUT', uploadUrl);
        xhr.setRequestHeader('Content-Type', file.type);

        xhr.upload.onprogress = (e) => {
          if (e.lengthComputable) {
            const progress = Math.round((e.loaded / e.total) * 100);
            setUploads(prev => new Map(prev).set(id, { ...prev.get(id)!, progress }));
          }
        };

        xhr.onload = () => {
          if (xhr.status === 200) resolve();
          else reject(new Error(`Upload failed: ${xhr.status}`));
        };

        xhr.onerror = () => reject(new Error('Network error'));
        xhr.send(file);
      });

      // Step 3: Mark as complete
      setUploads(prev => new Map(prev).set(id, {
        ...prev.get(id)!, status: 'complete', progress: 100, url: publicUrl,
      }));

      return { id, url: publicUrl, key };
    } catch (error) {
      setUploads(prev => new Map(prev).set(id, {
        ...prev.get(id)!, status: 'error', error: error.message,
      }));
      throw error;
    }
  };

  return { upload, uploads };
}
```

## Drag and drop upload component

```tsx
function UploadZone({ onUpload, accept, maxSize = 10 * 1024 * 1024 }) {
  const [isDragging, setIsDragging] = useState(false);
  const inputRef = useRef<HTMLInputElement>(null);

  const handleFiles = (files: FileList) => {
    const validFiles = Array.from(files).filter(file => {
      if (file.size > maxSize) {
        alert(`${file.name} exceeds ${maxSize / 1024 / 1024}MB limit`);
        return false;
      }
      return true;
    });
    validFiles.forEach(onUpload);
  };

  return (
    <div
      onDragOver={(e) => { e.preventDefault(); setIsDragging(true); }}
      onDragLeave={() => setIsDragging(false)}
      onDrop={(e) => { e.preventDefault(); setIsDragging(false); handleFiles(e.dataTransfer.files); }}
      onClick={() => inputRef.current?.click()}
      className={cn(
        'border-2 border-dashed rounded-lg p-8 text-center cursor-pointer transition-colors',
        isDragging ? 'border-primary bg-primary/5' : 'border-border hover:border-primary/50'
      )}
    >
      <input
        ref={inputRef}
        type="file"
        multiple
        accept={accept}
        className="hidden"
        onChange={(e) => e.target.files && handleFiles(e.target.files)}
      />
      <UploadIcon className="mx-auto h-8 w-8 text-muted-foreground" />
      <p className="mt-2 text-sm text-muted-foreground">
        Drag files here or <span className="text-primary">browse</span>
      </p>
      <p className="mt-1 text-xs text-muted-foreground">
        Max {maxSize / 1024 / 1024}MB per file
      </p>
    </div>
  );
}
```

## Image optimization

### Server-side processing (sharp)

```ts
import sharp from 'sharp';
import { S3Client, PutObjectCommand } from '@aws-sdk/client-s3';

const variants = [
  { name: 'thumbnail', width: 200, height: 200, fit: 'cover' as const },
  { name: 'medium', width: 600, height: 600, fit: 'inside' as const },
  { name: 'large', width: 1200, height: 1200, fit: 'inside' as const },
];

async function processImageUpload(buffer: Buffer, key: string, contentType: string) {
  const results: Record<string, string> = {};

  for (const variant of variants) {
    const transformed = await sharp(buffer)
      .resize(variant.width, variant.height, { fit: variant.fit, withoutEnlargement: true })
      .webp({ quality: 80 })
      .toBuffer();

    const variantKey = key.replace(/(\.\w+)$/, `-${variant.name}.webp`);

    await s3.send(new PutObjectCommand({
      Bucket: process.env.S3_BUCKET!,
      Key: variantKey,
      Body: transformed,
      ContentType: 'image/webp',
    }));

    results[variant.name] = getPublicUrl(variantKey);
  }

  return results;
}
```

### Client-side image preview and crop

```tsx
function ImagePreview({ file, onRemove }: { file: File; onRemove: () => void }) {
  const [preview, setPreview] = useState<string>();

  useEffect(() => {
    const url = URL.createObjectURL(file);
    setPreview(url);
    return () => URL.revokeObjectURL(url);
  }, [file]);

  if (!preview) return null;

  return (
    <div className="relative group">
      <img src={preview} alt={file.name} className="w-24 h-24 object-cover rounded-lg" />
      <button
        onClick={onRemove}
        className="absolute -top-2 -right-2 w-5 h-5 bg-red-500 text-white rounded-full
                   opacity-0 group-hover:opacity-100 transition-opacity"
      >
        ×
      </button>
      <p className="text-xs text-muted-foreground mt-1 truncate w-24">{file.name}</p>
    </div>
  );
}
```

## Chunked/resumable upload (large files)

### Server: Create and assemble chunks

```ts
import { S3Client, CreateMultipartUploadCommand, UploadPartCommand, CompleteMultipartUploadCommand } from '@aws-sdk/client-s3';

// Initialize multipart upload
app.post('/api/upload/multipart/init', async (req, res) => {
  const { filename, contentType } = req.body;
  const key = `uploads/${req.user.id}/${Date.now()}-${filename}`;

  const command = new CreateMultipartUploadCommand({
    Bucket: process.env.S3_BUCKET!,
    Key: key,
    ContentType: contentType,
  });

  const { UploadId } = await s3.send(command);

  res.json({ uploadId: UploadId, key });
});

// Get presigned URL for each part
app.post('/api/upload/multipart/presign', async (req, res) => {
  const { key, uploadId, partNumber } = req.body;

  const command = new UploadPartCommand({
    Bucket: process.env.S3_BUCKET!,
    Key: key,
    UploadId: uploadId,
    PartNumber: partNumber,
  });

  const presignedUrl = await getSignedUrl(s3, command, { expiresIn: 3600 });

  res.json({ url: presignedUrl });
});

// Complete multipart upload
app.post('/api/upload/multipart/complete', async (req, res) => {
  const { key, uploadId, parts } = req.body;

  const command = new CompleteMultipartUploadCommand({
    Bucket: process.env.S3_BUCKET!,
    Key: key,
    UploadId: uploadId,
    MultipartUpload: { Parts: parts },
  });

  await s3.send(command);

  res.json({ url: getPublicUrl(key) });
});
```

### Client: Chunked uploader with tus protocol

```ts
const CHUNK_SIZE = 5 * 1024 * 1024; // 5MB chunks (S3 minimum)

async function uploadLargeFile(file: File) {
  const totalChunks = Math.ceil(file.size / CHUNK_SIZE);

  // Step 1: Initialize
  const { uploadId, key } = await fetch('/api/upload/multipart/init', {
    method: 'POST',
    body: JSON.stringify({ filename: file.name, contentType: file.type }),
  }).then(r => r.json());

  const parts: Array<{ PartNumber: number; ETag: string }> = [];

  // Step 2: Upload chunks in parallel (max 3 concurrent)
  for (let i = 0; i < totalChunks; i += 3) {
    const chunkPromises = [];
    for (let j = i; j < Math.min(i + 3, totalChunks); j++) {
      chunkPromises.push(uploadChunk(file, key, uploadId, j + 1));
    }
    const results = await Promise.all(chunkPromises);
    parts.push(...results);
  }

  // Step 3: Complete
  const { url } = await fetch('/api/upload/multipart/complete', {
    method: 'POST',
    body: JSON.stringify({ key, uploadId, parts }),
  }).then(r => r.json());

  return url;
}
```

## Security

### Server-side validation

```ts
import { createReadStream } from 'fs';
import { fileTypeFromBuffer } from 'file-type';

const ALLOWED_MIME_TYPES = new Set([
  'image/jpeg', 'image/png', 'image/webp', 'image/gif',
  'application/pdf',
  'video/mp4',
]);

const MAX_FILE_SIZE = 10 * 1024 * 1024; // 10MB

async function validateFile(file: { name: string; type: string; size: number; buffer: Buffer }) {
  // Check file size
  if (file.size > MAX_FILE_SIZE) {
    throw new Error(`File size exceeds ${MAX_FILE_SIZE / 1024 / 1024}MB limit`);
  }

  // Check MIME type (from Content-Type header — can be spoofed)
  if (!ALLOWED_MIME_TYPES.has(file.type)) {
    throw new Error('File type not allowed');
  }

  // Verify actual file type from magic bytes
  const detectedType = await fileTypeFromBuffer(file.buffer);
  if (!detectedType || !ALLOWED_MIME_TYPES.has(detectedType.mime)) {
    throw new Error('File type mismatch: content does not match extension');
  }

  // Check file extension matches MIME type
  const ext = file.name.split('.').pop()?.toLowerCase();
  const mimeToExt: Record<string, string[]> = {
    'image/jpeg': ['jpg', 'jpeg'],
    'image/png': ['png'],
    'image/webp': ['webp'],
    'image/gif': ['gif'],
    'application/pdf': ['pdf'],
    'video/mp4': ['mp4'],
  };
  const allowedExts = mimeToExt[detectedType.mime] || [];
  if (!allowedExts.includes(ext || '')) {
    throw new Error('File extension does not match content type');
  }
}
```

### Signed URLs for download

```ts
import { GetObjectCommand } from '@aws-sdk/client-s3';

async function getDownloadUrl(key: string, expiresIn = 3600): Promise<string> {
  const command = new GetObjectCommand({
    Bucket: process.env.S3_BUCKET!,
    Key: key,
  });

  return getSignedUrl(s3, command, { expiresIn });
}

// Usage: return a temporary URL that expires in 1 hour
app.get('/api/files/:id/download', async (req, res) => {
  const file = await db.file.findUnique({ where: { id: req.params.id } });

  if (!file || file.userId !== req.user.id) {
    return res.status(404).json({ error: 'File not found' });
  }

  const url = await getDownloadUrl(file.storageKey);
  res.json({ url });
});
```

## Uploadthing (simpler alternative)

For faster implementation without managing S3 directly:

```tsx
import { createUploadthing } from 'uploadthing/next';

const f = createUploadthing();

export const ourFileRouter = {
  imageUploader: f({ image: { maxFileSize: '4MB', maxFileCount: 8 } })
    .onUploadComplete(async ({ file }) => {
      return { url: file.url, name: file.name };
    }),

  documentUploader: f({ pdf: { maxFileSize: '16MB' } })
    .onUploadComplete(async ({ file }) => {
      return { url: file.url, name: file.name };
    }),
};

export type OurFileRouter = typeof ourFileRouter;
```

```tsx
// Client component
import { useDropzone } from '@uploadthing/react';
import { generateMimeTypes } from 'uploadthing/client';

function UploadButton() {
  const { startUpload, isUploading } = useUploadThing('imageUploader');

  return (
    <button
      onClick={() => {
        const input = document.createElement('input');
        input.type = 'file';
        input.accept = generateMimeTypes(['image']).join(',');
        input.multiple = true;
        input.onchange = (e) => {
          const files = Array.from((e.target as HTMLInputElement).files || []);
          startUpload(files);
        };
        input.click();
      }}
      disabled={isUploading}
    >
      {isUploading ? 'Uploading...' : 'Upload Images'}
    </button>
  );
}
```

## Quality checklist

- [ ] File type validation on both client and server (MIME + magic bytes)
- [ ] File size limit enforced on client (immediate feedback) and server (security)
- [ ] Presigned URL pattern for direct S3 uploads (no server bottleneck)
- [ ] Upload progress visible to user (percentage + file name)
- [ ] Drag-and-drop upload with visual feedback
- [ ] Image preview before upload (URL.createObjectURL)
- [ ] Image optimization (resize, compress, WebP conversion)
- [ ] Responsive image variants (thumbnail, medium, large)
- [ ] Signed download URLs for private files (expiry: 1 hour)
- [ ] Cleanup of orphaned uploads (files uploaded but not linked to a record)
- [ ] Chunked upload for files > 10MB (resumable, parallel)
- [ ] Error handling for network failures (retry, resume)
- [ ] File name sanitization (remove special characters, prevent path traversal)
- [ ] Rate limiting on upload endpoint (prevent abuse)

## Anti-patterns I avoid

- Accepting file uploads directly through the server (bandwidth bottleneck) — use presigned URLs
- Trusting the client-provided Content-Type (spoofable) — validate with magic bytes
- No file size limit — DoS vulnerability
- Storing files in the database — use object storage (S3)
- No progress indication — users think the upload is broken
- Using PUT with XMLHttpRequest for large files without chunking — fails on slow connections
- Not cleaning up orphaned uploads — storage costs grow unbounded
- Allowing any file type — accept only what's needed (images need image/*, documents need pdf)
- Storing uploaded files with user-provided filenames — sanitize and generate unique names
- Not using CDN for file delivery — S3 direct is slow globally