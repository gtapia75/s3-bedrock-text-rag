# 📸 Multimodal Image RAG  
**AWS Bedrock (Titan) + ChromaDB + OpenAI Vision**

This repository contains a **from-scratch, production-aware implementation** of a **multimodal image retrieval pipeline** (Image RAG).

It demonstrates how to:

- Download images from **Amazon S3**
- Create **multimodal embeddings** using **Amazon Bedrock Titan**
- Store vectors in **ChromaDB**
- Retrieve images using **natural language queries**
- (Optionally) generate **vision-based summaries** using **OpenAI models**
- Handle failures and corrupted images gracefully

This project is implemented as a **Colab-friendly Python notebook**, but structured so the logic can be reused in production systems.

---

## 🧠 High-Level Architecture

### Ingestion (one-time or periodic)
```
S3 Images
   ↓
Download to local disk
   ↓
Normalize images (JPEG, RGB, resize)
   ↓
Titan Multimodal Image Embeddings
   ↓
ChromaDB (persistent vector store)
```

### Retrieval (per query)
```
User text query
   ↓
Titan Text Embedding
   ↓
Chroma similarity search
   ↓
Top-K images (with distances)
```

### Optional Generation (Vision)
```
Top images
   ↓
OpenAI Vision model
   ↓
Human-readable summary (S1…Sn references)
```

---

## ✨ Key Features

- ✅ True multimodal search (text → image)
- ✅ Shared embedding space (Titan text + image)
- ✅ Failure-tolerant ingestion
- ✅ Cost-aware design (embed once, reuse forever)
- ✅ Colab-ready
- ✅ Production-portable logic

---

## 📁 Project Structure (Recommended)

```
.
├── notebooks/
│   └── image_rag_poc.ipynb
├── data/
│   └── images/              # downloaded S3 images
├── chroma_images_db/        # Chroma persistent storage
├── docs/
│   └── image_rag_docs.html
├── README.md
└── requirements.txt
```

---

## 🔐 Prerequisites

### AWS
- S3 bucket with images (`.jpg`, `.jpeg`, `.png`)
- IAM user with:
  - `s3:GetObject`
  - `s3:ListBucket`
  - `bedrock:InvokeModel`
- Environment variables:
```bash
AWS_ACCESS_KEY_ID
AWS_SECRET_ACCESS_KEY
AWS_DEFAULT_REGION
```

### OpenAI (optional)
```bash
OPENAI_API_KEY
```

---

## 🔁 Typical Usage Flow

```python
# 1. List images in S3
image_keys = list_image_keys(BUCKET, PREFIX, max_keys=200)

# 2. Download locally
local_paths = download_s3_objects(BUCKET, image_keys)

# 3. Build stable records
records = build_image_records(image_keys, local_paths)

# 4. Create / load Chroma collection
collection = get_chroma_collection()

# 5. Embed + upsert
failed = upsert_images_to_chroma(collection, records)

# 6. Query
hits = query_images(collection, "blue t-shirt", k=8)

# 7. Display
show_hit_images(hits, s3_to_local)
```

---

## 🧩 Core Functions (Summary)

### S3 Utilities
- `get_s3_client()` – Create S3 client
- `list_image_keys()` – List image keys from S3
- `download_s3_objects()` – Download images locally

### Image Processing
- `normalize_image_to_jpeg_bytes()` – Normalize images to safe JPEG bytes

### Bedrock Embeddings
- `titan_embed_image_safe()` – Image embeddings
- `titan_embed_text()` – Text embeddings

### Vector Store
- `get_chroma_collection()` – Create/load ChromaDB
- `upsert_images_to_chroma()` – Batch embedding + upsert

### Retrieval & Display
- `query_images()` – Text → image similarity search
- `show_hit_images()` – Display retrieved images

### OpenAI Vision (Optional)
- `image_to_b64()` – Encode image for OpenAI
- `summarize_images_with_openai()` – Vision-based summaries

---

## 💸 Cost Awareness

- **Titan embeddings**: one-time cost per image
- **Text queries**: very low cost
- **ChromaDB**: free (local)
- **OpenAI Vision**: pay per request (optional)

👉 Best practice: embed once, reuse forever.

---

## 🚀 Roadmap / Extensions

- OCR + text indexing
- Metadata filters
- Re-ranking with vision models
- Video → frame → image RAG
- Managed vector store (OpenSearch / pgvector)

---

## 📄 License

MIT (or your preferred license)
