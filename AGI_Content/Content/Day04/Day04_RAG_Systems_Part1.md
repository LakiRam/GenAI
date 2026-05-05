# Day 4: Production RAG — Part 1 (Architecture & Retrieval)

> **Time:** 3.5 hours (1 hr theory + 2.5 hrs hands-on)
> **Goal:** Build a production-quality RAG pipeline: document loading, chunking strategies, hybrid retrieval, and reranking.
> **Deliverable:** End-to-end RAG system over a PDF corpus + advanced RAG with hybrid search and reranking.

---

## Part 1: Theory — RAG Architecture Deep Dive (1 hr)

### 4.1 What Is RAG and Why Does It Matter?

**Retrieval-Augmented Generation (RAG)** = Give the LLM relevant external knowledge at query time instead of relying on what it memorized during training.

```
WITHOUT RAG (pure LLM):
  User: "What was our company's Q3 revenue?"
  LLM:  "I don't have access to your company data." ← Useless
        or worse: "Your Q3 revenue was $15M."       ← Hallucination!

WITH RAG:
  User: "What was our company's Q3 revenue?"
  → Retrieve: Search company docs → Find Q3 earnings report
  → Augment:  Inject report into prompt context
  → Generate: "Based on the Q3 earnings report, revenue was $23.4M,
              up 12% from Q2. (Source: Q3_Earnings_2024.pdf, page 3)"
  ← Accurate, grounded, cited!
```

### 4.2 RAG Architecture Progression

```
NAIVE RAG (v1):
  ┌──────────┐     ┌────────────┐     ┌────────────┐     ┌──────┐
  │  Query   │────→│  Embed     │────→│  Retrieve   │────→│ LLM  │
  └──────────┘     │  query     │     │  top-k      │     │      │
                   └────────────┘     │  chunks     │     └──────┘
                                      └────────────┘

Problem: Simple retrieval misses context, returns irrelevant chunks.

ADVANCED RAG (v2):
  ┌──────────┐     ┌────────────┐     ┌────────────┐     ┌──────────┐     ┌──────┐
  │  Query   │────→│  Rewrite   │────→│  Hybrid     │────→│ Rerank   │────→│ LLM  │
  │          │     │  query     │     │  retrieve   │     │ top-k    │     │      │
  └──────────┘     └────────────┘     │  BM25+dense │     └──────────┘     └──────┘
                                      └────────────┘

MODULAR RAG (v3):
  ┌──────────┐     ┌────────┐     ┌────────┐     ┌────────┐     ┌──────┐
  │  Query   │────→│ Route  │────→│ Multi- │────→│ Rerank │────→│ LLM  │
  │ Analysis │     │        │     │ source │     │ +Filter│     │      │
  └──────────┘     └────────┘     │retrieve│     └────────┘     └──────┘
                   ┌────────┐     └────────┘     ┌────────┐
                   │ SQL DB │                     │ Cite + │
                   │ API    │                     │ Ground │
                   │ Web    │                     └────────┘
                   └────────┘
```

### 4.3 The Ingestion Pipeline

Before you can search, you must process your documents:

```
INGESTION PIPELINE:

  ┌───────┐     ┌──────────┐     ┌──────────┐     ┌──────────┐     ┌──────────┐
  │ Load  │────→│ Parse /  │────→│ Chunk    │────→│ Embed    │────→│ Store in │
  │ docs  │     │ clean    │     │          │     │          │     │ vector DB│
  └───────┘     └──────────┘     └──────────┘     └──────────┘     └──────────┘
  PDF, DOCX      Extract text     Split into        Generate         ChromaDB,
  HTML, MD        Remove noise     pieces            vectors          Qdrant, etc.
  CSV, JSON       Normalize        + metadata        per chunk
```

### 4.4 Chunking Strategies — The Most Important Decision

How you split documents into chunks is the **single biggest factor** in RAG quality.

```
STRATEGY 1: Fixed-Size Chunks
  "The quick brown fox jumps | over the lazy dog. The | cat sat on the mat."
  ├── Chunk 1: "The quick brown fox jumps"     (loses context)
  ├── Chunk 2: "over the lazy dog. The"        (breaks mid-sentence!)
  └── Chunk 3: "cat sat on the mat."           (orphaned)

  ✅ Simple, fast
  ❌ Breaks sentences and paragraphs

STRATEGY 2: Recursive Character Splitting
  Split by: ["\n\n", "\n", ". ", " "]  (try each in order)
  
  "Section 1\n\nThe quick brown fox jumps over the lazy dog.\n\nSection 2\n\n..."
  ├── Chunk 1: "Section 1\n\nThe quick brown fox jumps over the lazy dog."
  └── Chunk 2: "Section 2\n\n..."

  ✅ Respects document structure
  ✅ Keeps paragraphs together
  ✅ Most common approach

STRATEGY 3: Semantic Chunking
  Use embeddings to detect topic boundaries:
  
  Sentence embeddings: [0.9, 0.8, 0.85, 0.2, 0.7, 0.75]
  Similarity to next:  [0.95, 0.92, 0.30, 0.88, 0.93]
                                      ↑
                              Topic boundary! Split here.

  ✅ Best semantic coherence
  ❌ Slower, more complex, uses more API calls

STRATEGY 4: Document-Aware (Structural)
  Use document structure: headings, sections, tables
  
  # Chapter 1              → Chunk 1 (with heading as metadata)
  Lorem ipsum dolor...
  
  ## Section 1.1            → Chunk 2 (with heading hierarchy)
  Sit amet consectetur...
  
  | Table Header |          → Chunk 3 (keep tables intact)
  |--------------|
  | Data         |

  ✅ Best for structured documents
  ❌ Requires format-specific parsers
```

### 4.5 Chunk Size and Overlap

```
CHUNK SIZE TRADE-OFFS:

  Small chunks (100-200 tokens):
  ✅ Precise retrieval — each chunk has one idea
  ❌ Missing context — "it" refers to what?
  ❌ More chunks to search, more API calls

  Large chunks (1000-2000 tokens):
  ✅ More context per chunk
  ❌ Diluted relevance — key info buried in noise
  ❌ Wastes context window space

  Sweet spot: 300-800 tokens with 50-100 token overlap

OVERLAP IS CRITICAL:

  Without overlap:
  Chunk 1: "...the API key is IMPORTANT_KEY_1234."
  Chunk 2: "Using the key, authenticate to the..."
  → Searching for "API key authentication" might miss the connection!

  With overlap (50 tokens):
  Chunk 1: "...the API key is IMPORTANT_KEY_1234. Using the key, authenticate"
  Chunk 2: "IMPORTANT_KEY_1234. Using the key, authenticate to the..."
  → Both chunks contain the full context!
```

### 4.6 Retrieval Strategies

```
1. DENSE RETRIEVAL (embedding similarity)
   Query → embed → find similar vectors → return top-k
   ✅ Captures semantic meaning
   ❌ Misses exact keyword matches

2. SPARSE RETRIEVAL (BM25 / keyword)
   Query → tokenize → TF-IDF scoring → return top-k
   ✅ Great for exact terms, names, codes
   ❌ No semantic understanding

3. HYBRID RETRIEVAL (best of both) ← RECOMMENDED
   Dense results + BM25 results → merge with Reciprocal Rank Fusion (RRF)
   
   RRF Score = Σ 1/(k + rank_i)  for each retrieval system
   
   ✅ Best overall quality
   ✅ Handles both semantic AND keyword queries
```

### 4.7 Reranking — The Quality Multiplier

After initial retrieval, use a cross-encoder model to re-score results.

```
WITHOUT RERANKING:
  1. Retrieve top-20 chunks (fast, approximate)
  2. Send top-5 to LLM
  → Problem: best chunk might be #8, not in top-5!

WITH RERANKING:
  1. Retrieve top-20 chunks (fast, approximate)
  2. Rerank all 20 with cross-encoder (slower, accurate)
  3. Send reranked top-5 to LLM
  → The most relevant chunks rise to the top!

CROSS-ENCODER vs BI-ENCODER:
  Bi-encoder:    embed(query) × embed(doc) → fast but approximate
  Cross-encoder: model(query + doc) → slow but precise
  
  Use bi-encoder for retrieval (speed), cross-encoder for reranking (quality).
```

---

## Part 2: Hands-On Labs (2.5 hrs)

### Lab 4.1 — End-to-End RAG Over PDFs (75 min)

Create `day04_rag_pipeline.py`:

```python
"""
Day 4 - Lab 4.1: End-to-End RAG Pipeline
Build a complete RAG system: load PDFs, chunk, embed, store, retrieve, generate.
"""

import os
import json
import hashlib
from pathlib import Path
from dataclasses import dataclass
from openai import OpenAI
from dotenv import load_dotenv
import chromadb
from chromadb.utils.embedding_functions import OpenAIEmbeddingFunction

load_dotenv()

client = OpenAI()


# ─────────────────────────────────────────────
# Data Models
# ─────────────────────────────────────────────

@dataclass
class Document:
    text: str
    metadata: dict


@dataclass
class Chunk:
    text: str
    metadata: dict
    chunk_id: str


@dataclass
class RetrievedContext:
    chunks: list[Chunk]
    scores: list[float]


@dataclass
class RAGResponse:
    answer: str
    sources: list[dict]
    context_used: int  # number of chunks used


# ─────────────────────────────────────────────
# Step 1: Document Loading
# ─────────────────────────────────────────────

def load_text_documents(directory: str) -> list[Document]:
    """Load text/markdown files from a directory."""
    docs = []
    dir_path = Path(directory)
    
    if not dir_path.exists():
        print(f"Directory {directory} not found. Using sample documents.")
        return load_sample_documents()
    
    for file_path in dir_path.glob("**/*"):
        if file_path.suffix in (".txt", ".md", ".csv"):
            text = file_path.read_text(encoding="utf-8", errors="ignore")
            docs.append(Document(
                text=text,
                metadata={"source": str(file_path.name), "path": str(file_path), "type": file_path.suffix},
            ))
    
    return docs


def load_sample_documents() -> list[Document]:
    """Create sample documents for testing (simulating real PDFs)."""
    documents = [
        Document(
            text="""# Machine Learning Operations (MLOps) Guide

## Chapter 1: Introduction to MLOps

MLOps is a set of practices that combines Machine Learning, DevOps, and Data Engineering. 
The goal is to deploy and maintain ML systems in production reliably and efficiently.

### 1.1 Why MLOps Matters

Traditional software development follows well-established practices: version control, CI/CD, 
testing, and monitoring. ML systems introduce additional complexity:

- **Data dependencies**: Models depend on training data that changes over time
- **Model versioning**: Unlike code, models are binary artifacts that need tracking
- **Feature stores**: Features need consistent computation in training and serving
- **Model monitoring**: Models degrade over time due to data drift and concept drift
- **Reproducibility**: Training runs must be reproducible for debugging and auditing

### 1.2 MLOps Maturity Levels

**Level 0 - Manual Process:**
- Jupyter notebooks for experimentation
- Manual model deployment
- No CI/CD pipeline
- No monitoring

**Level 1 - ML Pipeline Automation:**
- Automated training pipeline
- Experiment tracking (MLflow, W&B)
- Model registry
- Basic monitoring

**Level 2 - CI/CD for ML:**
- Automated testing (data validation, model validation)
- Continuous training with new data
- A/B testing and canary deployments
- Comprehensive monitoring and alerting

## Chapter 2: Experiment Tracking

### 2.1 MLflow Basics

MLflow is an open-source platform for managing the ML lifecycle. Key components:
- **Tracking**: Log parameters, metrics, and artifacts
- **Models**: Package models in a standard format
- **Registry**: Central model store with versioning
- **Projects**: Reproducible ML code packaging

### 2.2 Setting Up Experiment Tracking

To track experiments effectively:
1. Log all hyperparameters (learning rate, batch size, etc.)
2. Log metrics at each epoch (loss, accuracy, F1)
3. Save model artifacts (checkpoints, final model)
4. Tag experiments with meaningful names
5. Compare runs side-by-side in the UI

### 2.3 Best Practices for Experiment Tracking

- Use meaningful experiment names: `project_model_date`
- Always log the git commit hash for reproducibility
- Save sample predictions alongside metrics
- Track computational resources (GPU hours, cost)
- Document failed experiments — they're valuable too
""",
            metadata={"source": "mlops_guide.pdf", "type": "guide", "pages": 45},
        ),
        Document(
            text="""# API Design Best Practices

## RESTful API Design

### Resource Naming
- Use nouns, not verbs: `/users` not `/getUsers`
- Use plural nouns: `/users` not `/user`
- Use kebab-case for multi-word: `/user-profiles`
- Nest resources logically: `/users/123/orders`

### HTTP Methods
- GET: Retrieve resources (safe, idempotent)
- POST: Create new resources
- PUT: Replace entire resource (idempotent)
- PATCH: Partial update
- DELETE: Remove resource (idempotent)

### Status Codes
- 200 OK: Successful GET, PUT, PATCH
- 201 Created: Successful POST
- 204 No Content: Successful DELETE
- 400 Bad Request: Invalid input
- 401 Unauthorized: Authentication required
- 403 Forbidden: Insufficient permissions
- 404 Not Found: Resource doesn't exist
- 429 Too Many Requests: Rate limited
- 500 Internal Server Error: Server-side error

### Pagination
For large collections, use cursor-based pagination:
```json
{
  "data": [...],
  "pagination": {
    "next_cursor": "abc123",
    "has_more": true,
    "total": 1500
  }
}
```

### Rate Limiting
Implement rate limiting to protect your API:
- Use token bucket algorithm
- Return `429 Too Many Requests` with `Retry-After` header
- Include rate limit headers: `X-RateLimit-Limit`, `X-RateLimit-Remaining`

### Authentication
JWT-based authentication flow:
1. Client sends credentials to `/auth/login`
2. Server validates and returns JWT access + refresh tokens
3. Client includes JWT in `Authorization: Bearer <token>` header
4. Server validates JWT on each request
5. Client uses refresh token when access token expires

### Versioning
Use URL-based versioning: `/api/v1/users`
- Major version in URL for breaking changes
- Minor/patch versions handled in implementation
- Support at least 2 major versions simultaneously
- Deprecate old versions with 6-month notice

### Error Responses
Standardize error format:
```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Email format is invalid",
    "details": [
      {"field": "email", "issue": "Must be a valid email address"}
    ],
    "request_id": "req_abc123"
  }
}
```
""",
            metadata={"source": "api_design.pdf", "type": "reference", "pages": 28},
        ),
        Document(
            text="""# Cloud Architecture Patterns

## 1. Microservices Architecture

### When to Use Microservices
- Team size > 20 developers
- Different components need different scaling
- Components have different release cycles
- Technology diversity is needed (Python ML + Go API + React UI)

### Service Communication
**Synchronous (REST/gRPC):**
- Use for request-response patterns
- gRPC for internal service-to-service (binary, fast, typed)
- REST for external APIs (human-readable, standard)

**Asynchronous (Message Queues):**
- Use for event-driven patterns
- RabbitMQ for task queues
- Kafka for event streaming
- SQS for simple cloud queues

### Database per Service
Each microservice owns its data:
- User Service → Users DB (PostgreSQL)
- Order Service → Orders DB (MongoDB)  
- Analytics Service → Analytics DB (ClickHouse)
- Cache → Redis

Never let services directly access another service's database!

## 2. Event-Driven Architecture

### Event Sourcing
Instead of storing current state, store all events:
- OrderCreated → OrderItemAdded → PaymentProcessed → OrderShipped
- Can replay events to reconstruct state at any point in time
- Perfect for audit trails and debugging

### CQRS (Command Query Responsibility Segregation)
- Separate read and write models
- Write model: optimized for consistency (normalized)
- Read model: optimized for queries (denormalized)
- Sync via events

## 3. Serverless Patterns

### Function-as-a-Service (FaaS)
- AWS Lambda, Azure Functions, Google Cloud Functions
- Pay only for execution time
- Auto-scales to zero
- Max execution time limits (15 min for Lambda)

### Best Use Cases
- API handlers (small, stateless requests)
- Event processing (S3 uploads, queue messages)
- Scheduled tasks (cron jobs)
- Webhooks

### Anti-Patterns
- Long-running processes → Use containers (ECS/EKS)
- Stateful workloads → Use VMs or containers
- Low-latency requirements → Cold starts add 100ms-2s
- Heavy computation → Use dedicated compute (GPU instances)

## 4. Caching Strategies

### Cache-Aside (Lazy Loading)
1. Check cache → if HIT, return cached data
2. If MISS → query database → store in cache → return

### Write-Through
1. Write to cache AND database simultaneously
2. Ensures cache is always up-to-date
3. Higher write latency

### Cache Invalidation
The two hardest things in computer science:
1. Cache invalidation
2. Naming things

**TTL-based:** Set expiration time (simple but stale data possible)
**Event-based:** Invalidate on write events (consistent but complex)
**Versioned:** Cache key includes version number (clean but requires coordination)
""",
            metadata={"source": "cloud_architecture.pdf", "type": "reference", "pages": 62},
        ),
    ]
    return documents


# ─────────────────────────────────────────────
# Step 2: Chunking
# ─────────────────────────────────────────────

def recursive_text_splitter(
    text: str, chunk_size: int = 500, chunk_overlap: int = 50,
    separators: list[str] = None,
) -> list[str]:
    """Split text recursively by trying different separators."""
    if separators is None:
        separators = ["\n\n", "\n", ". ", " "]
    
    if len(text) <= chunk_size:
        return [text.strip()] if text.strip() else []
    
    # Find the best separator
    chosen_sep = separators[-1]
    for sep in separators:
        if sep in text:
            chosen_sep = sep
            break
    
    parts = text.split(chosen_sep)
    
    chunks = []
    current_chunk = ""
    
    for part in parts:
        candidate = (current_chunk + chosen_sep + part).strip() if current_chunk else part.strip()
        
        if len(candidate) <= chunk_size:
            current_chunk = candidate
        else:
            if current_chunk:
                chunks.append(current_chunk)
            
            # If single part exceeds chunk_size, split further
            if len(part) > chunk_size:
                remaining_seps = separators[separators.index(chosen_sep) + 1:] if chosen_sep in separators else separators[-1:]
                if remaining_seps:
                    sub_chunks = recursive_text_splitter(part, chunk_size, chunk_overlap, remaining_seps)
                    chunks.extend(sub_chunks)
                    current_chunk = ""
                else:
                    # Last resort: hard split
                    for i in range(0, len(part), chunk_size - chunk_overlap):
                        chunks.append(part[i:i + chunk_size].strip())
                    current_chunk = ""
            else:
                current_chunk = part.strip()
    
    if current_chunk:
        chunks.append(current_chunk)
    
    # Add overlap
    if chunk_overlap > 0 and len(chunks) > 1:
        overlapped_chunks = [chunks[0]]
        for i in range(1, len(chunks)):
            prev_tail = chunks[i - 1][-chunk_overlap:]
            overlapped_chunks.append(prev_tail + " " + chunks[i])
        return overlapped_chunks
    
    return chunks


def chunk_documents(documents: list[Document], chunk_size: int = 500, chunk_overlap: int = 50) -> list[Chunk]:
    """Chunk all documents and attach metadata."""
    all_chunks = []
    
    for doc in documents:
        text_chunks = recursive_text_splitter(doc.text, chunk_size, chunk_overlap)
        
        for i, chunk_text in enumerate(text_chunks):
            chunk_id = hashlib.md5(f"{doc.metadata.get('source', '')}_{i}_{chunk_text[:50]}".encode()).hexdigest()[:16]
            
            chunk = Chunk(
                text=chunk_text,
                metadata={
                    **doc.metadata,
                    "chunk_index": i,
                    "total_chunks": len(text_chunks),
                    "char_count": len(chunk_text),
                },
                chunk_id=chunk_id,
            )
            all_chunks.append(chunk)
    
    return all_chunks


# ─────────────────────────────────────────────
# Step 3: Indexing
# ─────────────────────────────────────────────

def index_chunks(chunks: list[Chunk], collection_name: str = "rag_demo") -> chromadb.Collection:
    """Index chunks into ChromaDB."""
    embedding_fn = OpenAIEmbeddingFunction(
        api_key=os.getenv("OPENAI_API_KEY"),
        model_name="text-embedding-3-small",
    )
    
    chroma = chromadb.PersistentClient(path="./rag_chroma_db")
    
    # Clean start
    try:
        chroma.delete_collection(collection_name)
    except Exception:
        pass
    
    collection = chroma.create_collection(
        name=collection_name,
        embedding_function=embedding_fn,
        metadata={"hnsw:space": "cosine"},
    )
    
    # Batch insert
    batch_size = 100
    for i in range(0, len(chunks), batch_size):
        batch = chunks[i:i + batch_size]
        collection.add(
            ids=[c.chunk_id for c in batch],
            documents=[c.text for c in batch],
            metadatas=[c.metadata for c in batch],
        )
    
    return collection


# ─────────────────────────────────────────────
# Step 4: Retrieval
# ─────────────────────────────────────────────

def retrieve(query: str, collection, n_results: int = 5, where: dict = None) -> RetrievedContext:
    """Retrieve relevant chunks from vector DB."""
    kwargs = {"query_texts": [query], "n_results": n_results}
    if where:
        kwargs["where"] = where
    
    results = collection.query(**kwargs)
    
    chunks = []
    scores = []
    for i in range(len(results["documents"][0])):
        chunks.append(Chunk(
            text=results["documents"][0][i],
            metadata=results["metadatas"][0][i],
            chunk_id=results["ids"][0][i],
        ))
        scores.append(1 - results["distances"][0][i])  # Convert distance to similarity
    
    return RetrievedContext(chunks=chunks, scores=scores)


# ─────────────────────────────────────────────
# Step 5: Generation
# ─────────────────────────────────────────────

def generate_answer(query: str, context: RetrievedContext, model: str = "gpt-4o-mini") -> RAGResponse:
    """Generate an answer using retrieved context."""
    # Build context string with source attribution
    context_parts = []
    for i, (chunk, score) in enumerate(zip(context.chunks, context.scores)):
        source = chunk.metadata.get("source", "unknown")
        context_parts.append(f"[Source {i+1}: {source} (relevance: {score:.2f})]\n{chunk.text}")
    
    context_str = "\n\n---\n\n".join(context_parts)
    
    system_prompt = """You are a knowledgeable assistant that answers questions using ONLY the provided context.

RULES:
1. ONLY use information from the provided context to answer
2. If the context doesn't contain enough information, say "I don't have enough information to answer this fully"
3. ALWAYS cite your sources using [Source N] format
4. Be concise and direct
5. If multiple sources cover the same topic, synthesize the information"""

    user_prompt = f"""Context:
{context_str}

Question: {query}

Answer the question based ONLY on the context above. Cite sources."""

    response = client.chat.completions.create(
        model=model,
        messages=[
            {"role": "system", "content": system_prompt},
            {"role": "user", "content": user_prompt},
        ],
        temperature=0,
        max_tokens=800,
    )
    
    sources = [{"source": c.metadata.get("source", "unknown"), "score": round(s, 3), "chunk_index": c.metadata.get("chunk_index", -1)} for c, s in zip(context.chunks, context.scores)]
    
    return RAGResponse(
        answer=response.choices[0].message.content,
        sources=sources,
        context_used=len(context.chunks),
    )


# ─────────────────────────────────────────────
# Step 6: Full RAG Pipeline
# ─────────────────────────────────────────────

class RAGPipeline:
    """Complete RAG pipeline: load → chunk → index → retrieve → generate."""
    
    def __init__(self, collection_name: str = "rag_demo"):
        self.collection_name = collection_name
        self.collection = None
        self.chunks = []
    
    def ingest(self, documents: list[Document], chunk_size: int = 500, chunk_overlap: int = 50):
        """Ingest documents into the pipeline."""
        print(f"📄 Loading {len(documents)} documents...")
        
        self.chunks = chunk_documents(documents, chunk_size, chunk_overlap)
        print(f"✂️  Created {len(self.chunks)} chunks (size={chunk_size}, overlap={chunk_overlap})")
        
        # Show chunk stats
        chunk_lens = [len(c.text) for c in self.chunks]
        print(f"   Chunk lengths: min={min(chunk_lens)}, max={max(chunk_lens)}, avg={sum(chunk_lens)//len(chunk_lens)}")
        
        self.collection = index_chunks(self.chunks, self.collection_name)
        print(f"✅ Indexed {self.collection.count()} chunks into ChromaDB")
    
    def query(self, question: str, n_results: int = 5, where: dict = None) -> RAGResponse:
        """Ask a question and get an answer with sources."""
        if not self.collection:
            raise RuntimeError("Call ingest() first")
        
        context = retrieve(question, self.collection, n_results, where)
        return generate_answer(question, context)
    
    def query_with_debug(self, question: str, n_results: int = 5):
        """Ask a question with full debug output."""
        print(f"\n{'=' * 70}")
        print(f"🔍 QUESTION: {question}")
        print(f"{'=' * 70}")
        
        # Retrieve
        context = retrieve(question, self.collection, n_results)
        
        print(f"\n📚 Retrieved {len(context.chunks)} chunks:")
        for i, (chunk, score) in enumerate(zip(context.chunks, context.scores)):
            print(f"\n  [{i+1}] Score: {score:.3f} | Source: {chunk.metadata.get('source', '?')}")
            print(f"      {chunk.text[:120]}...")
        
        # Generate
        response = generate_answer(question, context)
        
        print(f"\n💡 ANSWER:")
        print(f"   {response.answer}")
        print(f"\n📖 SOURCES: {json.dumps(response.sources, indent=2)}")
        
        return response


# ─────────────────────────────────────────────
# Run the demo
# ─────────────────────────────────────────────

if __name__ == "__main__":
    rag = RAGPipeline()
    
    # Load sample documents (replace with real PDFs later)
    docs = load_sample_documents()
    rag.ingest(docs, chunk_size=500, chunk_overlap=50)
    
    # Test queries
    questions = [
        "What are the MLOps maturity levels?",
        "How should I handle API authentication?",
        "When should I use microservices vs monolith?",
        "What are the best caching strategies?",
        "How do I track ML experiments?",
        "What is CQRS and when should I use it?",
    ]
    
    for q in questions:
        rag.query_with_debug(q, n_results=3)
    
    # Test edge case: question NOT covered by documents
    print(f"\n{'═' * 70}")
    print("EDGE CASE: Out-of-scope question")
    print(f"{'═' * 70}")
    rag.query_with_debug("What is the recipe for chocolate cake?", n_results=3)
```

### Lab 4.2 — Advanced RAG with Hybrid Search + Reranking (75 min)

Create `day04_advanced_rag.py`:

```python
"""
Day 4 - Lab 4.2: Advanced RAG with Hybrid Search and Reranking
Combines dense + sparse retrieval and cross-encoder reranking.
"""

import os
import json
import re
import math
from collections import Counter
from dataclasses import dataclass
from openai import OpenAI
from dotenv import load_dotenv
import chromadb
from chromadb.utils.embedding_functions import OpenAIEmbeddingFunction

load_dotenv()

client = OpenAI()


# ─────────────────────────────────────────────
# BM25 Sparse Retrieval
# ─────────────────────────────────────────────

class BM25:
    """Simple BM25 implementation for sparse retrieval."""
    
    def __init__(self, k1: float = 1.5, b: float = 0.75):
        self.k1 = k1
        self.b = b
        self.corpus_size = 0
        self.avg_dl = 0
        self.doc_freqs: dict[str, int] = {}
        self.doc_lengths: list[int] = []
        self.doc_tokens: list[list[str]] = []
    
    @staticmethod
    def tokenize(text: str) -> list[str]:
        """Simple whitespace + lowercase tokenization."""
        text = text.lower()
        tokens = re.findall(r'\b[a-z0-9]+\b', text)
        # Remove common stop words
        stop_words = {"the", "a", "an", "is", "are", "was", "were", "in", "on", "at", "to", "for", "of", "and", "or", "but", "not", "with", "this", "that", "it", "be", "as", "by", "from"}
        return [t for t in tokens if t not in stop_words]
    
    def index(self, documents: list[str]):
        """Index a corpus of documents."""
        self.corpus_size = len(documents)
        self.doc_tokens = [self.tokenize(doc) for doc in documents]
        self.doc_lengths = [len(tokens) for tokens in self.doc_tokens]
        self.avg_dl = sum(self.doc_lengths) / self.corpus_size if self.corpus_size else 0
        
        # Calculate document frequencies
        self.doc_freqs = {}
        for tokens in self.doc_tokens:
            unique_tokens = set(tokens)
            for token in unique_tokens:
                self.doc_freqs[token] = self.doc_freqs.get(token, 0) + 1
    
    def score(self, query: str, doc_idx: int) -> float:
        """Calculate BM25 score for a single document."""
        query_tokens = self.tokenize(query)
        doc_tokens = self.doc_tokens[doc_idx]
        doc_len = self.doc_lengths[doc_idx]
        tf = Counter(doc_tokens)
        
        score = 0.0
        for qt in query_tokens:
            if qt not in tf:
                continue
            
            term_freq = tf[qt]
            doc_freq = self.doc_freqs.get(qt, 0)
            
            # IDF
            idf = math.log((self.corpus_size - doc_freq + 0.5) / (doc_freq + 0.5) + 1)
            
            # TF normalization
            tf_norm = (term_freq * (self.k1 + 1)) / (term_freq + self.k1 * (1 - self.b + self.b * doc_len / self.avg_dl))
            
            score += idf * tf_norm
        
        return score
    
    def search(self, query: str, top_k: int = 10) -> list[tuple[int, float]]:
        """Search and return top-k (doc_index, score) pairs."""
        scores = [(i, self.score(query, i)) for i in range(self.corpus_size)]
        scores.sort(key=lambda x: x[1], reverse=True)
        return scores[:top_k]


# ─────────────────────────────────────────────
# LLM-based Reranker (using GPT as cross-encoder)
# ─────────────────────────────────────────────

def llm_rerank(query: str, documents: list[str], top_k: int = 5) -> list[tuple[int, float]]:
    """Rerank documents using LLM scoring."""
    if not documents:
        return []
    
    # Score each document
    scores = []
    for i, doc in enumerate(documents):
        prompt = f"""Rate the relevance of the following document to the query on a scale of 0-10.

Query: {query}

Document: {doc[:500]}

Respond with ONLY a JSON object: {{"score": <number 0-10>, "reason": "<brief reason>"}}"""
        
        response = client.chat.completions.create(
            model="gpt-4o-mini",
            messages=[{"role": "user", "content": prompt}],
            temperature=0,
            max_tokens=100,
            response_format={"type": "json_object"},
        )
        
        result = json.loads(response.choices[0].message.content)
        scores.append((i, result.get("score", 0) / 10.0))
    
    scores.sort(key=lambda x: x[1], reverse=True)
    return scores[:top_k]


# ─────────────────────────────────────────────
# Reciprocal Rank Fusion
# ─────────────────────────────────────────────

def reciprocal_rank_fusion(
    rankings: list[list[tuple[int, float]]],
    k: int = 60,
) -> list[tuple[int, float]]:
    """Merge multiple rankings using RRF.
    
    Args:
        rankings: List of ranked lists, each containing (doc_index, score) tuples
        k: RRF parameter (default 60, standard value)
    
    Returns:
        Fused ranking as (doc_index, rrf_score) tuples
    """
    rrf_scores: dict[int, float] = {}
    
    for ranking in rankings:
        for rank, (doc_idx, _) in enumerate(ranking):
            rrf_scores[doc_idx] = rrf_scores.get(doc_idx, 0) + 1.0 / (k + rank + 1)
    
    fused = sorted(rrf_scores.items(), key=lambda x: x[1], reverse=True)
    return fused


# ─────────────────────────────────────────────
# Advanced RAG Pipeline
# ─────────────────────────────────────────────

class AdvancedRAGPipeline:
    """RAG with hybrid search (dense + BM25) and reranking."""
    
    def __init__(self):
        self.documents: list[str] = []
        self.metadatas: list[dict] = []
        self.bm25 = BM25()
        self.collection = None
    
    def ingest(self, texts: list[str], metadatas: list[dict] = None):
        """Index documents for both dense and sparse retrieval."""
        self.documents = texts
        self.metadatas = metadatas or [{}] * len(texts)
        
        # Sparse index (BM25)
        self.bm25.index(texts)
        print(f"✅ BM25 indexed {len(texts)} documents")
        
        # Dense index (ChromaDB)
        embedding_fn = OpenAIEmbeddingFunction(
            api_key=os.getenv("OPENAI_API_KEY"),
            model_name="text-embedding-3-small",
        )
        
        chroma = chromadb.Client()
        try:
            chroma.delete_collection("advanced_rag")
        except Exception:
            pass
        
        self.collection = chroma.create_collection(
            name="advanced_rag",
            embedding_function=embedding_fn,
            metadata={"hnsw:space": "cosine"},
        )
        
        ids = [f"doc_{i}" for i in range(len(texts))]
        self.collection.add(ids=ids, documents=texts, metadatas=self.metadatas)
        print(f"✅ Dense indexed {len(texts)} documents")
    
    def hybrid_retrieve(self, query: str, n_results: int = 10) -> list[tuple[int, float]]:
        """Hybrid retrieval: combine dense and sparse results with RRF."""
        # Dense retrieval
        dense_results = self.collection.query(query_texts=[query], n_results=n_results)
        dense_ranking = []
        for i in range(len(dense_results["ids"][0])):
            doc_idx = int(dense_results["ids"][0][i].replace("doc_", ""))
            score = 1 - dense_results["distances"][0][i]
            dense_ranking.append((doc_idx, score))
        
        # Sparse retrieval (BM25)
        sparse_ranking = self.bm25.search(query, top_k=n_results)
        
        # Fuse with RRF
        fused = reciprocal_rank_fusion([dense_ranking, sparse_ranking])
        
        return fused[:n_results]
    
    def query(
        self, question: str, n_retrieve: int = 10, n_rerank: int = 5,
        use_reranking: bool = True, debug: bool = False,
    ) -> dict:
        """Full advanced RAG query with optional reranking."""
        # Step 1: Hybrid retrieval
        hybrid_results = self.hybrid_retrieve(question, n_retrieve)
        
        if debug:
            print(f"\n📚 Hybrid retrieval ({len(hybrid_results)} docs):")
            for doc_idx, score in hybrid_results[:5]:
                print(f"  [{doc_idx}] RRF={score:.4f} — {self.documents[doc_idx][:80]}...")
        
        # Step 2: Reranking (optional)
        if use_reranking:
            candidate_docs = [self.documents[idx] for idx, _ in hybrid_results]
            reranked = llm_rerank(question, candidate_docs, top_k=n_rerank)
            
            # Map back to original indices
            final_indices = [(hybrid_results[i][0], score) for i, score in reranked]
            
            if debug:
                print(f"\n🔄 After reranking ({len(final_indices)} docs):")
                for doc_idx, score in final_indices:
                    print(f"  [{doc_idx}] Relevance={score:.2f} — {self.documents[doc_idx][:80]}...")
        else:
            final_indices = hybrid_results[:n_rerank]
        
        # Step 3: Generate answer
        context_parts = []
        for i, (doc_idx, score) in enumerate(final_indices):
            source = self.metadatas[doc_idx].get("source", f"doc_{doc_idx}")
            context_parts.append(f"[Source {i+1}: {source}]\n{self.documents[doc_idx]}")
        
        context_str = "\n\n---\n\n".join(context_parts)
        
        response = client.chat.completions.create(
            model="gpt-4o-mini",
            messages=[
                {"role": "system", "content": "Answer questions using ONLY the provided context. Cite sources as [Source N]. If the context doesn't contain the answer, say so."},
                {"role": "user", "content": f"Context:\n{context_str}\n\nQuestion: {question}"},
            ],
            temperature=0,
            max_tokens=500,
        )
        
        return {
            "answer": response.choices[0].message.content,
            "sources": [{"doc_idx": idx, "score": round(score, 3)} for idx, score in final_indices],
            "method": "hybrid+rerank" if use_reranking else "hybrid",
        }


# ─────────────────────────────────────────────
# Demo: Compare naive vs advanced retrieval
# ─────────────────────────────────────────────

if __name__ == "__main__":
    # Sample knowledge base
    corpus = [
        "Python is a high-level programming language known for its readability and versatility. It supports multiple paradigms including OOP and functional programming.",
        "The Python GIL (Global Interpreter Lock) prevents multiple threads from executing Python bytecode simultaneously. Use multiprocessing for CPU-bound tasks instead.",
        "REST APIs use HTTP methods: GET for reading, POST for creating, PUT for updating, DELETE for removing resources. Always use proper status codes.",
        "Rate limiting protects APIs from abuse. Common algorithms include token bucket and sliding window. Return 429 status with Retry-After header.",
        "JWT (JSON Web Tokens) provide stateless authentication. They contain header, payload, and signature. Use short expiration times and refresh tokens.",
        "Docker containers package applications with their dependencies. Use multi-stage builds to reduce image size. Never run containers as root.",
        "Kubernetes orchestrates containers at scale. Key concepts: Pods, Deployments, Services, Ingress. Use Helm charts for package management.",
        "PostgreSQL supports JSONB columns for semi-structured data. Use GIN indexes for JSONB queries. Consider partitioning for tables over 100M rows.",
        "Redis is an in-memory data store used for caching, sessions, and pub/sub. Set appropriate TTLs and use Redis Cluster for high availability.",
        "Machine learning pipelines include data collection, feature engineering, model training, evaluation, and deployment. Track experiments with MLflow.",
        "CQRS separates read and write operations. Write model uses normalized tables for consistency. Read model uses denormalized views for query performance.",
        "Event sourcing stores all state changes as events. Benefits: full audit trail, temporal queries, event replay. Drawback: eventual consistency.",
        "Microservices communicate via REST or gRPC synchronously, or message queues asynchronously. Use circuit breakers to handle service failures.",
        "BM25 is a classical information retrieval algorithm. It ranks documents by term frequency, inverse document frequency, and document length normalization.",
        "Vector databases like ChromaDB and Qdrant store embeddings for similarity search. They use ANN indexes (HNSW, IVF) for fast approximate search.",
    ]
    
    metadatas = [
        {"source": "python_guide.md", "topic": "python"},
        {"source": "python_guide.md", "topic": "python"},
        {"source": "api_design.md", "topic": "api"},
        {"source": "api_design.md", "topic": "api"},
        {"source": "auth_guide.md", "topic": "security"},
        {"source": "docker_guide.md", "topic": "devops"},
        {"source": "k8s_guide.md", "topic": "devops"},
        {"source": "database_guide.md", "topic": "database"},
        {"source": "database_guide.md", "topic": "database"},
        {"source": "ml_guide.md", "topic": "ml"},
        {"source": "architecture.md", "topic": "architecture"},
        {"source": "architecture.md", "topic": "architecture"},
        {"source": "architecture.md", "topic": "architecture"},
        {"source": "search_guide.md", "topic": "search"},
        {"source": "search_guide.md", "topic": "search"},
    ]
    
    pipeline = AdvancedRAGPipeline()
    pipeline.ingest(corpus, metadatas)
    
    # ─── Test queries ───
    test_queries = [
        # Semantic query — should find relevant docs by meaning
        "How do I make my API secure from too many requests?",
        # Keyword-heavy query — BM25 should excel
        "GIL multiprocessing Python",
        # Complex query — needs multiple sources
        "What database and caching strategies should I use for a high-traffic application?",
        # Out-of-scope query
        "How do I bake chocolate chip cookies?",
    ]
    
    for query in test_queries:
        print(f"\n{'═' * 70}")
        print(f"❓ {query}")
        print(f"{'═' * 70}")
        
        # Compare with and without reranking
        result_no_rerank = pipeline.query(query, use_reranking=False, debug=True)
        print(f"\n📝 Answer (hybrid, no rerank):")
        print(f"   {result_no_rerank['answer'][:200]}...")
        
        result_reranked = pipeline.query(query, use_reranking=True, debug=True)
        print(f"\n📝 Answer (hybrid + reranked):")
        print(f"   {result_reranked['answer'][:200]}...")
```

---

## Part 3: Review & Reflection (30 min)

### What You Learned Today

1. **RAG solves these LLM limitations:** ________________________________
2. **The best chunking strategy for structured docs is:** ________________________________
3. **Hybrid retrieval combines:** ________________________________
4. **Reranking improves results by:** ________________________________
5. **The sweet spot for chunk size is:** ________________________________

### RAG Quality Improvement Checklist
```
□ Good chunking (preserve semantic boundaries)
□ Appropriate chunk size (300-800 tokens)
□ Chunk overlap (50-100 tokens)
□ Hybrid retrieval (dense + sparse)
□ Reranking (cross-encoder or LLM judge)
□ Source attribution (cite every claim)
□ Guardrails ("I don't know" for out-of-scope)
□ Context window management (don't overflow)
```

### Homework: Day 4 Deliverable

1. **Push both lab scripts** to GitHub (`day04/`)
2. **RAG pipeline** that ingests 3+ documents and answers 6 questions with source citations
3. **Advanced RAG** showing side-by-side comparison: naive retrieval vs. hybrid+reranking
4. **Write a 1-paragraph analysis:** Which retrieval combination gave the best results and why?

---

*Tomorrow: Day 5 — Production RAG Part 2 (citations, memory, guardrails, evaluation with RAGAS)*
