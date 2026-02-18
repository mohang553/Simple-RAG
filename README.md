# 🤖 RAG API with Gemini & Pinecone

A **Retrieval-Augmented Generation (RAG)** system that lets you upload private documents and query them using natural language — powered by **Google Gemini**, **Pinecone**, and **Sentence Transformers**.

---

## 📌 Problem Statement

### Background

Organizations and individuals frequently work with large volumes of unstructured documents — PDFs, Word files, text reports — where finding specific information requires manual reading. Large Language Models (LLMs) like Gemini are powerful but have two major limitations:

- They have **no knowledge of your private documents**
- They can **hallucinate** answers when asked about content they haven't seen

### Problem

There is no efficient way for users to **query their own private documents using natural language** and receive accurate, grounded answers without manually searching through files.

Specifically:
- LLMs have no awareness of user-uploaded documents
- Keyword search lacks semantic understanding (synonyms, paraphrasing, context)
- Storing entire documents in an LLM prompt is costly and exceeds context limits
- Responses without source grounding lead to fabricated or inaccurate answers

### Solution

This API solves the problem using a **3-step RAG pipeline**:

1. **Ingest** — Upload documents (.txt, .pdf, .docx), which are chunked and embedded using `all-MiniLM-L6-v2`
2. **Retrieve** — On query, perform semantic similarity search in **Pinecone** to find the most relevant chunks
3. **Generate** — Pass retrieved chunks as context to **Gemini**, which generates a grounded, accurate answer with source attribution

---

## 🏗️ Architecture

```
User Upload
    │
    ▼
Text Extraction (PyPDF2 / python-docx / plain text)
    │
    ▼
Chunking (1000 chars, 200 overlap)
    │
    ▼
Embedding (all-MiniLM-L6-v2 → 384-dim vectors)
    │
    ▼
Pinecone Vector Store (cosine similarity)
    
User Query
    │
    ▼
Query Embedding
    │
    ▼
Pinecone Similarity Search (top-k chunks)
    │
    ▼
Gemini LLM (context + question → answer)
    │
    ▼
Response with Sources
```

---

## 🚀 Getting Started

### Prerequisites

- Python 3.8+
- A [Pinecone](https://www.pinecone.io/) account and API key
- A [Google Gemini](https://ai.google.dev/) API key

### Installation

```bash
# Clone the repository
git clone <your-repo-url>
cd Simple_RAG

# Create and activate virtual environment
python -m venv venv
venv\Scripts\activate      # Windows
source venv/bin/activate   # Mac/Linux

# Install dependencies
pip install fastapi uvicorn pinecone-client google-genai PyPDF2 python-docx python-dotenv sentence-transformers
```

### Environment Setup

Create a `.env` file in the project root:

```env
PINECONE_API_KEY=your_pinecone_api_key_here
GEMINI_API_KEY=your_gemini_api_key_here
```

### Run the API

```bash
python rag.py
```

The server starts at `http://localhost:8010`

---

## 📡 API Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/` | Root — lists all endpoints |
| GET | `/health` | Health check + Pinecone connection status |
| GET | `/stats` | Total vectors stored in the database |
| POST | `/upload` | Upload and index a document |
| POST | `/query` | Query documents with a natural language question |
| DELETE | `/clear` | Clear all vectors from the database |

### Upload a Document

```bash
curl -X POST "http://localhost:8010/upload" \
  -F "file=@leave_policy.pdf"
```

**Response:**
```json
{
  "message": "File uploaded and processed successfully",
  "filename": "leave_policy.pdf",
  "chunks_added": 3
}
```

### Query the Documents

```bash
curl -X POST "http://localhost:8010/query" \
  -H "Content-Type: application/json" \
  -d '{"question": "How many sick leave days are allowed?", "top_k": 3}'
```

**Response:**
```json
{
  "answer": "Employees are entitled to 10 sick leave days per year...",
  "sources": [
    {
      "chunk_id": "leave_policy.pdf_0",
      "filename": "leave_policy.pdf",
      "score": 0.91,
      "text": "Sick leave is capped at 10 days per year..."
    }
  ]
}
```

---

## 🧪 Testing with Swagger UI

Visit `http://localhost:8010/docs` for an interactive API interface where you can upload files and run queries directly in the browser.

---

## ⚙️ Configuration

| Parameter | Default | Description |
|-----------|---------|-------------|
| `CHUNK_SIZE` | 1000 | Number of characters per chunk |
| `CHUNK_OVERLAP` | 200 | Overlap between consecutive chunks |
| `INDEX_NAME` | `gemini-rag` | Pinecone index name |
| `top_k` | 3 | Number of chunks retrieved per query |
| Embedding Model | `all-MiniLM-L6-v2` | 384-dimensional sentence embeddings |
| LLM | `gemini-2.5-flash` | Google Gemini model for generation |

---

## 📁 Supported File Types

| Extension | Library Used |
|-----------|-------------|
| `.txt` | Built-in Python |
| `.pdf` | PyPDF2 |
| `.docx` | python-docx |

---

## 🔒 Limitations & Scope

- File types limited to `.txt`, `.pdf`, and `.docx`
- Embeddings are fixed at 384 dimensions
- Retrieval uses approximate cosine similarity — no re-ranking applied
- No user authentication or multi-tenancy
- No persistent conversation memory between queries

---

## 🛠️ Tech Stack

| Component | Technology |
|-----------|-----------|
| API Framework | FastAPI |
| Vector Database | Pinecone (Serverless) |
| LLM | Google Gemini 2.5 Flash |
| Embeddings | Sentence Transformers (`all-MiniLM-L6-v2`) |
| PDF Parsing | PyPDF2 |
| DOCX Parsing | python-docx |
| Environment | python-dotenv |

---