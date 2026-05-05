# Day 3: Embeddings & Vector Databases

> **Time:** 3.5 hours (1 hr theory + 2.5 hrs hands-on)
> **Goal:** Understand how text becomes vectors and build real semantic search using vector databases.
> **Deliverable:** Semantic search engine with ChromaDB + embedding visualization + vector DB benchmark report.

---

## Part 1: Theory — Turning Text Into Math (1 hr)

### 3.1 What Are Embeddings?

An embedding is a dense vector (list of numbers) that captures the *meaning* of text. Similar meanings → similar vectors. This is the foundation of semantic search, RAG, recommendations, and clustering.

```
TRADITIONAL SEARCH (keyword):
  Query: "How do I fix a slow computer?"
  Match: Documents containing "fix", "slow", "computer"
  Miss:  "Tips to speed up your PC performance"  ← No keyword overlap!

SEMANTIC SEARCH (embeddings):
  Query: "How do I fix a slow computer?"  →  [0.23, -0.45, 0.87, ...]
  Match: "Tips to speed up your PC performance" → [0.21, -0.43, 0.85, ...]
  Score: cosine_similarity = 0.97  ← Recognized as SAME meaning!
```

### 3.2 Geometric Intuition

Think of embeddings as coordinates in a high-dimensional space.

```
3D ANALOGY (real embeddings have 768-3072 dimensions):

              meaning_technology
              ↑
              |    "GPU"
              |   •
              |  "CPU" •
              |       • "RAM"
              |
              |                    • "pizza"
              |                   • "burger"
              +────────────────────────→ meaning_food
             /
            /  • "Shakespeare"
           /  • "poetry"
          ↓
     meaning_arts

KEY INSIGHT: Similar concepts cluster together in embedding space.
            Distance between vectors = semantic dissimilarity.
```

### 3.3 How Embeddings Are Created

Models like `text-embedding-3-small` are transformer neural networks trained with **contrastive learning**:

```
TRAINING PROCESS:
                                                  
  "The cat sat on the mat"  ──→ Encoder ──→ [0.1, -0.3, 0.8, ...]  ← Anchor
  "A feline rested on a rug" ──→ Encoder ──→ [0.1, -0.3, 0.7, ...]  ← Positive (push close)
  "Stock prices rose today"  ──→ Encoder ──→ [0.9, 0.5, -0.2, ...]  ← Negative (push apart)

  Loss = max(0, dist(anchor,positive) - dist(anchor,negative) + margin)

After billions of examples, the encoder learns to map meaning → geometry.
```

### 3.4 Similarity Metrics

How do we measure "closeness" between two vectors?

| Metric | Formula | Range | Best For |
|--------|---------|-------|----------|
| **Cosine Similarity** | cos(θ) = A·B / (‖A‖‖B‖) | [-1, 1] | Text similarity (most common) |
| **Euclidean Distance** | ‖A - B‖₂ | [0, ∞) | When magnitude matters |
| **Dot Product** | A·B = Σ(aᵢ × bᵢ) | (-∞, ∞) | Normalized vectors, speed |

```
WHY COSINE?

  "cat" = [3, 4]     (magnitude = 5)
  "cat" × 2 = [6, 8] (magnitude = 10)  ← Same direction, different magnitude

  Cosine similarity = 1.0  (same direction = same meaning)
  Euclidean distance = 5.0 (different! penalizes magnitude)

  For text, we care about DIRECTION (meaning), not MAGNITUDE (length).
  → Cosine similarity is the standard choice.
```

### 3.5 Embedding Models Comparison

| Model | Dimensions | Max Tokens | Cost / 1M tokens | Quality |
|-------|-----------|-----------|-------------------|---------|
| text-embedding-3-small | 1536 | 8191 | $0.02 | Good |
| text-embedding-3-large | 3072 | 8191 | $0.13 | Great |
| text-embedding-ada-002 | 1536 | 8191 | $0.10 | Good (legacy) |
| Cohere embed-v3 | 1024 | 512 | $0.10 | Great |
| Voyage AI voyage-3 | 1024 | 32000 | $0.06 | Excellent |
| BGE-large-en (open) | 1024 | 512 | Free | Good |
| Nomic embed (open) | 768 | 8192 | Free | Good |

### 3.6 Vector Databases — Storing and Searching Billions of Vectors

You can't compute cosine similarity against every vector when you have millions. Vector databases use **Approximate Nearest Neighbor (ANN)** algorithms for sub-millisecond search.

```
ANN INDEX TYPES:

1. HNSW (Hierarchical Navigable Small World)
   ┌─────────────────────────────────┐
   │ Layer 3:  A ─── B              │ ← Coarse (few nodes, long links)
   │ Layer 2:  A ─ C ─ B ─ D        │
   │ Layer 1:  A─C─E─B─D─F─G       │
   │ Layer 0:  A C E H B D F G I J  │ ← Fine (all nodes, short links)
   └─────────────────────────────────┘
   Search: Start at top layer, greedily descend → O(log N)
   Trade-off: High memory, fast search, no training needed

2. IVF (Inverted File Index)
   ┌───────────────────────────────────┐
   │ Cluster centroids:  C1  C2  C3   │
   │                    / \  |  / \   │
   │ Vectors:       v1 v2 v3 v4 v5 v6│
   └───────────────────────────────────┘
   Search: Find nearest centroid → search only that cluster
   Trade-off: Lower memory, requires training, tunable nprobe

3. PQ (Product Quantization)
   Original: [0.1, 0.3, 0.7, 0.2, 0.5, 0.8, 0.4, 0.9]
   Split:    [0.1, 0.3] [0.7, 0.2] [0.5, 0.8] [0.4, 0.9]
   Quantize: [Code 3]   [Code 7]   [Code 1]   [Code 12]
   → Compress 32 bytes → 4 bytes (8x compression!)
   Trade-off: Very compact, some accuracy loss
```

### 3.7 Vector DB Landscape

| Database | Type | Best For | ANN Index |
|----------|------|----------|-----------|
| **ChromaDB** | Embedded/client | Prototyping, small datasets | HNSW |
| **Qdrant** | Server | Production, filtering | HNSW |
| **Pinecone** | Managed cloud | Zero-ops, enterprise | Proprietary |
| **Weaviate** | Server | Multi-modal, GraphQL | HNSW |
| **pgvector** | PostgreSQL ext | Already using Postgres | IVF/HNSW |
| **FAISS** | Library | Pure speed, research | IVF+PQ, HNSW |
| **Milvus** | Distributed | Billion-scale | IVF+PQ, HNSW |

### 3.8 Metadata Filtering — The Killer Feature

Vector search alone isn't enough. You need to **combine** semantic similarity with structured filters.

```
EXAMPLE: Find similar documents, but only from 2024, written by "Engineering"

  Query embedding: [0.2, -0.4, 0.8, ...]
  Filter: {"year": 2024, "department": "Engineering"}

  Vector DB does:
  1. Apply metadata filter → reduces candidate set
  2. Run ANN search on filtered candidates
  3. Return top-k results

  This is MUCH harder than it sounds — most DBs implement it differently.
```

---

## Part 2: Hands-On Labs (2.5 hrs)

### Lab 3.1 — Embedding Visualization (45 min)

Create `day03_embedding_viz.py`:

```python
"""
Day 3 - Lab 3.1: Embedding Visualization
Embed 100 sentences, reduce to 2D with t-SNE, and visualize clusters.
"""

import json
import numpy as np
from openai import OpenAI
from dotenv import load_dotenv

load_dotenv()

client = OpenAI()


# ─────────────────────────────────────────────
# Step 1: Create diverse text samples
# ─────────────────────────────────────────────

CATEGORIES = {
    "Technology": [
        "Python is a popular programming language for data science",
        "The new GPU accelerates machine learning training by 10x",
        "Docker containers simplify application deployment",
        "Kubernetes orchestrates containerized applications at scale",
        "React and Vue are popular frontend JavaScript frameworks",
        "API rate limiting prevents abuse of web services",
        "SQL databases use tables with rows and columns",
        "Git enables version control and collaborative coding",
        "Cloud computing provides on-demand computing resources",
        "Microservices architecture breaks apps into small services",
        "The CPU processes instructions from computer programs",
        "Encryption protects data by converting it to unreadable text",
        "HTTP is the protocol used for web communication",
        "Machine learning models learn patterns from training data",
        "DevOps combines software development and IT operations",
        "The Linux kernel is the core of many operating systems",
        "SSL certificates ensure secure data transmission",
        "Continuous integration automates build and test processes",
        "NoSQL databases handle unstructured data efficiently",
        "WebSockets enable real-time bidirectional communication",
    ],
    "Cooking": [
        "Preheat the oven to 375 degrees Fahrenheit",
        "Sauté the onions in olive oil until translucent",
        "Fresh basil adds amazing flavor to tomato sauce",
        "Knead the bread dough for about 10 minutes",
        "Season the steak with salt and pepper before grilling",
        "A sharp knife is the most important kitchen tool",
        "Simmer the soup on low heat for two hours",
        "Whisk the eggs until they form stiff peaks",
        "Caramelize the sugar slowly to make a rich sauce",
        "Marinate the chicken overnight for maximum flavor",
        "Use a meat thermometer to ensure proper doneness",
        "Fresh herbs are always better than dried ones",
        "Blanch vegetables in boiling water then ice bath",
        "The Maillard reaction creates the browned crust on steak",
        "Deglaze the pan with wine to make a sauce",
        "Rest the roast for 15 minutes before carving",
        "Fold the flour gently to keep the batter light",
        "Toast spices in a dry pan to bring out their aroma",
        "A roux thickens sauces: equal parts butter and flour",
        "Brine the turkey for a moist and flavorful result",
    ],
    "Sports": [
        "The quarterback threw a perfect spiral for the touchdown",
        "She broke the world record in the 100-meter dash",
        "The soccer team won the championship with a penalty shootout",
        "Tennis players train for hours to perfect their serve",
        "The basketball game went into triple overtime",
        "Marathon runners carb-load before a big race",
        "The pitcher threw a 95 mph fastball for a strikeout",
        "Swimmers use flip turns to maintain speed in the pool",
        "The golf tournament was decided on the 18th hole",
        "Hockey players practice skating drills for agility",
        "The gymnast performed a flawless balance beam routine",
        "Rugby requires both physical strength and strategic thinking",
        "Track and field athletes peak in their late twenties",
        "The boxing match went the full twelve rounds",
        "Volleyball teams rotate positions after each serve",
        "Cross country skiing is one of the toughest endurance sports",
        "The cycling race covered 200 kilometers through mountains",
        "Professional surfers travel the world chasing perfect waves",
        "Cricket matches can last up to five days in test format",
        "The Formula One car reached speeds over 200 mph",
    ],
    "Finance": [
        "The stock market rose 2% after the Fed announcement",
        "Compound interest is the most powerful force in investing",
        "Diversification reduces portfolio risk across asset classes",
        "The company's quarterly earnings exceeded analyst expectations",
        "Bond yields move inversely to bond prices",
        "A bear market is defined as a 20% decline from recent highs",
        "Dollar cost averaging removes emotion from investing",
        "The PE ratio measures a stock's price relative to earnings",
        "Inflation erodes the purchasing power of money over time",
        "Mutual funds pool money from multiple investors",
        "Cryptocurrency markets operate 24 hours a day",
        "The Federal Reserve sets interest rates for the US economy",
        "REITs allow investors to own real estate without buying property",
        "Options contracts give the right but not obligation to buy",
        "A 401k provides tax-advantaged retirement savings",
        "Market capitalization is share price times shares outstanding",
        "Hedge funds use leverage to amplify investment returns",
        "The yield curve shows interest rates across bond maturities",
        "ESG investing considers environmental and social factors",
        "Technical analysis uses charts to predict price movements",
    ],
    "Medicine": [
        "The patient's blood pressure was elevated at 150/95",
        "Antibiotics should only be prescribed for bacterial infections",
        "MRI scans provide detailed images of soft tissues",
        "The vaccine stimulates the immune system to produce antibodies",
        "Physical therapy helps patients recover after surgery",
        "Type 2 diabetes is managed through diet and medication",
        "The surgeon performed a minimally invasive laparoscopic procedure",
        "Cholesterol levels should be monitored regularly after age 40",
        "Anesthesia ensures patients feel no pain during surgery",
        "Clinical trials test new drugs for safety and efficacy",
        "The stethoscope remains an essential diagnostic tool",
        "CRISPR gene editing shows promise for genetic diseases",
        "Telemedicine expanded rapidly during the pandemic",
        "CPR can save lives when performed within minutes",
        "Chemotherapy targets rapidly dividing cancer cells",
        "Sleep deprivation impairs immune function and cognitive ability",
        "The hippocampus plays a crucial role in memory formation",
        "Vitamins and minerals are essential micronutrients",
        "Radiology uses imaging to diagnose and treat diseases",
        "Stem cell therapy holds potential for regenerative medicine",
    ],
}


def get_embeddings(texts: list[str], model: str = "text-embedding-3-small") -> list[list[float]]:
    """Get embeddings from OpenAI API."""
    response = client.embeddings.create(input=texts, model=model)
    return [item.embedding for item in response.data]


def cosine_similarity(a: list[float], b: list[float]) -> float:
    """Calculate cosine similarity between two vectors."""
    a_arr, b_arr = np.array(a), np.array(b)
    return float(np.dot(a_arr, b_arr) / (np.linalg.norm(a_arr) * np.linalg.norm(b_arr)))


if __name__ == "__main__":
    # ─── Collect all texts with labels ───
    texts = []
    labels = []
    for category, sentences in CATEGORIES.items():
        texts.extend(sentences)
        labels.extend([category] * len(sentences))
    
    print(f"Total texts: {len(texts)} across {len(CATEGORIES)} categories")
    
    # ─── Generate embeddings ───
    print("Generating embeddings...")
    embeddings = get_embeddings(texts)
    print(f"Embedding dimensions: {len(embeddings[0])}")
    
    # ─── Similarity analysis ───
    print(f"\n{'=' * 70}")
    print("SIMILARITY ANALYSIS")
    print(f"{'=' * 70}")
    
    # Intra-category vs inter-category similarity
    from itertools import combinations
    
    for cat in CATEGORIES:
        cat_indices = [i for i, l in enumerate(labels) if l == cat]
        cat_pairs = list(combinations(cat_indices, 2))
        intra_sims = [cosine_similarity(embeddings[i], embeddings[j]) for i, j in cat_pairs[:20]]
        avg_intra = np.mean(intra_sims)
        
        other_indices = [i for i, l in enumerate(labels) if l != cat][:20]
        inter_sims = [cosine_similarity(embeddings[cat_indices[0]], embeddings[j]) for j in other_indices]
        avg_inter = np.mean(inter_sims)
        
        print(f"\n{cat}:")
        print(f"  Avg intra-category similarity: {avg_intra:.4f}")
        print(f"  Avg inter-category similarity: {avg_inter:.4f}")
        print(f"  Separation ratio:              {avg_intra / avg_inter:.2f}x")
    
    # ─── Find most/least similar pairs ───
    print(f"\n{'=' * 70}")
    print("SEMANTIC SIMILARITY EXPLORER")
    print(f"{'=' * 70}")
    
    test_queries = [
        "How do I make my computer faster?",
        "What should I eat for dinner?",
        "How do I invest my savings?",
    ]
    
    query_embeddings = get_embeddings(test_queries)
    
    for query, q_emb in zip(test_queries, query_embeddings):
        sims = [(texts[i], labels[i], cosine_similarity(q_emb, emb)) for i, emb in enumerate(embeddings)]
        sims.sort(key=lambda x: x[2], reverse=True)
        
        print(f"\nQuery: \"{query}\"")
        print("  Top 5 most similar:")
        for text, cat, sim in sims[:5]:
            print(f"    [{cat:12}] {sim:.4f} — {text[:60]}")
        print("  Top 3 LEAST similar:")
        for text, cat, sim in sims[-3:]:
            print(f"    [{cat:12}] {sim:.4f} — {text[:60]}")
    
    # ─── Dimensionality reduction for visualization ───
    print(f"\n{'=' * 70}")
    print("DIMENSIONALITY REDUCTION (t-SNE → 2D)")
    print(f"{'=' * 70}")
    
    try:
        from sklearn.manifold import TSNE
        import matplotlib
        matplotlib.use("Agg")
        import matplotlib.pyplot as plt
        
        emb_array = np.array(embeddings)
        tsne = TSNE(n_components=2, random_state=42, perplexity=15)
        coords = tsne.fit_transform(emb_array)
        
        colors = {"Technology": "blue", "Cooking": "red", "Sports": "green", "Finance": "purple", "Medicine": "orange"}
        
        plt.figure(figsize=(14, 10))
        for cat, color in colors.items():
            mask = [l == cat for l in labels]
            cat_coords = coords[mask]
            plt.scatter(cat_coords[:, 0], cat_coords[:, 1], c=color, label=cat, alpha=0.7, s=60)
        
        plt.title("Embedding Space Visualization (t-SNE)", fontsize=16)
        plt.legend(fontsize=12)
        plt.xlabel("t-SNE Dimension 1")
        plt.ylabel("t-SNE Dimension 2")
        plt.tight_layout()
        plt.savefig("embedding_visualization.png", dpi=150)
        print("✅ Plot saved to embedding_visualization.png")
    except ImportError:
        print("⚠️  Install sklearn and matplotlib for visualization:")
        print("   pip install scikit-learn matplotlib")
    
    # ─── Embedding model comparison ───
    print(f"\n{'=' * 70}")
    print("EMBEDDING MODEL COMPARISON")
    print(f"{'=' * 70}")
    
    test_pairs = [
        ("Python is great for data science", "Data analysis works well with Python"),  # high sim expected
        ("The stock market crashed today", "She baked a chocolate cake"),              # low sim expected
        ("Machine learning predicts outcomes", "AI algorithms forecast results"),      # high sim expected
    ]
    
    models = ["text-embedding-3-small", "text-embedding-3-large"]
    
    for model in models:
        print(f"\nModel: {model}")
        for a, b in test_pairs:
            embs = get_embeddings([a, b], model=model)
            sim = cosine_similarity(embs[0], embs[1])
            print(f"  {sim:.4f}  |  \"{a[:40]}\" vs \"{b[:40]}\"")
```

### Lab 3.2 — Semantic Search Engine with ChromaDB (60 min)

Create `day03_semantic_search.py`:

```python
"""
Day 3 - Lab 3.2: Build a Semantic Search Engine with ChromaDB
Full-featured search with metadata filtering, hybrid search, and ranked results.
"""

import json
import hashlib
import chromadb
from chromadb.utils.embedding_functions import OpenAIEmbeddingFunction
from dotenv import load_dotenv
import os

load_dotenv()

# ─────────────────────────────────────────────
# Setup ChromaDB with OpenAI embeddings
# ─────────────────────────────────────────────

embedding_fn = OpenAIEmbeddingFunction(
    api_key=os.getenv("OPENAI_API_KEY"),
    model_name="text-embedding-3-small",
)

chroma_client = chromadb.PersistentClient(path="./chroma_db")


def get_or_create_collection(name: str):
    return chroma_client.get_or_create_collection(
        name=name,
        embedding_function=embedding_fn,
        metadata={"hnsw:space": "cosine"},
    )


# ─────────────────────────────────────────────
# Knowledge Base: Tech company documentation
# ─────────────────────────────────────────────

DOCUMENTS = [
    {
        "text": "To reset your password, go to Settings > Security > Change Password. You'll need your current password and a new one that's at least 12 characters with a mix of letters, numbers, and symbols.",
        "metadata": {"category": "account", "product": "all", "audience": "user", "difficulty": "easy"},
    },
    {
        "text": "Two-factor authentication (2FA) adds an extra layer of security. Enable it in Settings > Security > Two-Factor Auth. You can use an authenticator app or SMS verification.",
        "metadata": {"category": "security", "product": "all", "audience": "user", "difficulty": "easy"},
    },
    {
        "text": "The API rate limit is 1000 requests per minute for the Pro plan and 100 for the Free plan. If you exceed the limit, you'll receive a 429 status code. Implement exponential backoff.",
        "metadata": {"category": "api", "product": "developer-tools", "audience": "developer", "difficulty": "medium"},
    },
    {
        "text": "To deploy your application, push to the main branch. Our CI/CD pipeline will automatically build, test, and deploy to staging. Promote to production via the dashboard.",
        "metadata": {"category": "deployment", "product": "developer-tools", "audience": "developer", "difficulty": "medium"},
    },
    {
        "text": "Billing is monthly. Upgrade or downgrade your plan anytime in Settings > Billing. Changes take effect at the next billing cycle. Refunds are available within 14 days.",
        "metadata": {"category": "billing", "product": "all", "audience": "user", "difficulty": "easy"},
    },
    {
        "text": "Webhooks allow your application to receive real-time notifications. Configure them in Developer Settings > Webhooks. Each webhook must have an HTTPS endpoint with a valid SSL certificate.",
        "metadata": {"category": "api", "product": "developer-tools", "audience": "developer", "difficulty": "hard"},
    },
    {
        "text": "The analytics dashboard shows real-time user engagement metrics including DAU, session length, retention rates, and conversion funnels. Export data as CSV or via the Analytics API.",
        "metadata": {"category": "analytics", "product": "business-suite", "audience": "admin", "difficulty": "medium"},
    },
    {
        "text": "For enterprise SSO setup, we support SAML 2.0 and OpenID Connect. Contact your account manager for configuration. You'll need your IdP metadata URL and entity ID.",
        "metadata": {"category": "security", "product": "enterprise", "audience": "admin", "difficulty": "hard"},
    },
    {
        "text": "Data export is available for all plans. Go to Settings > Data > Export. You can export your data in JSON, CSV, or Parquet format. Large exports are processed asynchronously.",
        "metadata": {"category": "data", "product": "all", "audience": "user", "difficulty": "easy"},
    },
    {
        "text": "The GraphQL API provides flexible querying. Use introspection to explore the schema. Authentication requires a Bearer token in the Authorization header.",
        "metadata": {"category": "api", "product": "developer-tools", "audience": "developer", "difficulty": "hard"},
    },
    {
        "text": "Team management: Invite members via Settings > Team. Assign roles: Viewer (read-only), Editor (modify), Admin (full access). Owners can transfer ownership.",
        "metadata": {"category": "account", "product": "business-suite", "audience": "admin", "difficulty": "easy"},
    },
    {
        "text": "Our SLA guarantees 99.9% uptime for Business plans and 99.99% for Enterprise. Check real-time status at status.example.com. Incident reports are published within 24 hours.",
        "metadata": {"category": "reliability", "product": "enterprise", "audience": "admin", "difficulty": "medium"},
    },
    {
        "text": "Database backups run automatically every 6 hours. Point-in-time recovery is available for the last 30 days. To restore, go to Settings > Backups > Restore.",
        "metadata": {"category": "data", "product": "business-suite", "audience": "admin", "difficulty": "medium"},
    },
    {
        "text": "Custom domains require DNS configuration. Add a CNAME record pointing to custom.example.com. SSL certificates are provisioned automatically via Let's Encrypt within 15 minutes.",
        "metadata": {"category": "deployment", "product": "business-suite", "audience": "admin", "difficulty": "medium"},
    },
    {
        "text": "Mobile push notifications can be configured via the SDK. Initialize with your project key, register for push tokens, and handle incoming notifications in your app delegate or activity.",
        "metadata": {"category": "mobile", "product": "developer-tools", "audience": "developer", "difficulty": "hard"},
    },
]


def index_documents(collection_name: str = "knowledge_base"):
    """Index all documents into ChromaDB."""
    collection = get_or_create_collection(collection_name)
    
    # Generate stable IDs from content hash
    ids = [hashlib.md5(doc["text"].encode()).hexdigest()[:16] for doc in DOCUMENTS]
    texts = [doc["text"] for doc in DOCUMENTS]
    metadatas = [doc["metadata"] for doc in DOCUMENTS]
    
    collection.upsert(ids=ids, documents=texts, metadatas=metadatas)
    print(f"✅ Indexed {len(DOCUMENTS)} documents into '{collection_name}'")
    print(f"   Collection count: {collection.count()}")
    return collection


def semantic_search(query: str, collection, n_results: int = 5, where: dict = None, where_document: dict = None):
    """Search with optional metadata and full-text filters."""
    kwargs = {"query_texts": [query], "n_results": n_results}
    if where:
        kwargs["where"] = where
    if where_document:
        kwargs["where_document"] = where_document
    
    results = collection.query(**kwargs)
    return results


def print_results(query: str, results: dict, label: str = ""):
    """Pretty-print search results."""
    print(f"\n{'─' * 70}")
    print(f"🔍 Query: \"{query}\"" + (f"  [{label}]" if label else ""))
    print(f"{'─' * 70}")
    
    docs = results["documents"][0]
    distances = results["distances"][0]
    metas = results["metadatas"][0]
    
    for i, (doc, dist, meta) in enumerate(zip(docs, distances, metas)):
        sim = 1 - dist  # ChromaDB returns distance, we want similarity
        bar = "█" * int(sim * 20)
        print(f"\n  #{i+1} [sim={sim:.3f}] {bar}")
        print(f"      {doc[:100]}...")
        print(f"      Tags: {meta}")


# ─────────────────────────────────────────────
# Demo: Various search scenarios
# ─────────────────────────────────────────────

if __name__ == "__main__":
    collection = index_documents()
    
    # ─── Basic semantic search ───
    print(f"\n{'=' * 70}")
    print("BASIC SEMANTIC SEARCH")
    print(f"{'=' * 70}")
    
    queries = [
        "How do I change my login credentials?",
        "What happens if I make too many API calls?",
        "How reliable is the service?",
        "Can I get my data out of the system?",
    ]
    
    for q in queries:
        results = semantic_search(q, collection, n_results=3)
        print_results(q, results)
    
    # ─── Filtered search ───
    print(f"\n{'=' * 70}")
    print("FILTERED SEARCH (metadata)")
    print(f"{'=' * 70}")
    
    # Only developer-facing docs
    results = semantic_search(
        "How do I authenticate?",
        collection,
        n_results=3,
        where={"audience": "developer"},
    )
    print_results("How do I authenticate?", results, label="developer only")
    
    # Only easy difficulty
    results = semantic_search(
        "security features",
        collection,
        n_results=3,
        where={"difficulty": "easy"},
    )
    print_results("security features", results, label="easy difficulty only")
    
    # Compound filter
    results = semantic_search(
        "deployment and hosting",
        collection,
        n_results=3,
        where={"$and": [{"audience": "admin"}, {"product": "business-suite"}]},
    )
    print_results("deployment and hosting", results, label="admin + business-suite")
    
    # ─── Full-text + semantic ───
    print(f"\n{'=' * 70}")
    print("HYBRID SEARCH (semantic + keyword)")
    print(f"{'=' * 70}")
    
    # Must contain "API" in the text
    results = semantic_search(
        "rate limiting and throttling",
        collection,
        n_results=3,
        where_document={"$contains": "API"},
    )
    print_results("rate limiting", results, label="must contain 'API'")
    
    # ─── Similarity threshold ───
    print(f"\n{'=' * 70}")
    print("SIMILARITY THRESHOLD FILTERING")
    print(f"{'=' * 70}")
    
    results = semantic_search("quantum physics experiments", collection, n_results=5)
    
    print(f"\nQuery: \"quantum physics experiments\" (off-topic)")
    docs = results["documents"][0]
    distances = results["distances"][0]
    
    threshold = 0.5
    for i, (doc, dist) in enumerate(zip(docs, distances)):
        sim = 1 - dist
        status = "✅ RELEVANT" if sim > threshold else "❌ BELOW THRESHOLD"
        print(f"  #{i+1} [sim={sim:.3f}] {status} — {doc[:60]}...")
    
    print(f"\n💡 For off-topic queries, all results have low similarity.")
    print(f"   Set a threshold (e.g., {threshold}) to return 'No relevant results found.'")
```

### Lab 3.3 — Vector DB Benchmark (45 min)

Create `day03_vector_benchmark.py`:

```python
"""
Day 3 - Lab 3.3: Vector Database Benchmark
Compare ChromaDB performance: indexing speed, query speed, recall at different scales.
"""

import time
import random
import numpy as np
import chromadb
from chromadb.utils.embedding_functions import OpenAIEmbeddingFunction
from dotenv import load_dotenv
import os

load_dotenv()


def generate_synthetic_data(n: int, dim: int = 1536):
    """Generate synthetic documents and embeddings for benchmarking."""
    topics = ["tech", "science", "health", "finance", "sports", "food", "travel", "education"]
    sentiments = ["positive", "negative", "neutral"]
    
    documents = [f"Document {i}: This is a synthetic document about {random.choice(topics)} topic #{i}" for i in range(n)]
    metadatas = [{"topic": random.choice(topics), "sentiment": random.choice(sentiments), "score": round(random.uniform(0, 1), 2)} for _ in range(n)]
    embeddings = [np.random.randn(dim).tolist() for _ in range(n)]
    ids = [f"doc_{i}" for i in range(n)]
    
    return documents, metadatas, embeddings, ids


def benchmark_chromadb(sizes: list[int] = None):
    """Benchmark ChromaDB at various scales."""
    if sizes is None:
        sizes = [100, 500, 1000, 5000]
    
    results = []
    
    for n in sizes:
        print(f"\n{'=' * 60}")
        print(f"  BENCHMARK: {n:,} documents")
        print(f"{'=' * 60}")
        
        # Fresh client for each benchmark
        chroma = chromadb.Client()
        
        # Delete if exists (clean benchmark)
        try:
            chroma.delete_collection("benchmark")
        except Exception:
            pass
        
        collection = chroma.create_collection(
            name="benchmark",
            metadata={"hnsw:space": "cosine"},
        )
        
        # Generate data
        print(f"  Generating {n} synthetic documents...")
        docs, metas, embs, ids = generate_synthetic_data(n)
        
        # ─── Benchmark: Indexing ───
        batch_size = 500
        start = time.time()
        for i in range(0, n, batch_size):
            end_idx = min(i + batch_size, n)
            collection.add(
                ids=ids[i:end_idx],
                documents=docs[i:end_idx],
                metadatas=metas[i:end_idx],
                embeddings=embs[i:end_idx],
            )
        index_time = time.time() - start
        throughput = n / index_time
        
        print(f"  📥 Indexing:  {index_time:.2f}s ({throughput:.0f} docs/sec)")
        
        # ─── Benchmark: Basic query ───
        query_emb = np.random.randn(1536).tolist()
        
        times = []
        for _ in range(20):
            start = time.time()
            collection.query(query_embeddings=[query_emb], n_results=10)
            times.append(time.time() - start)
        
        avg_query = np.mean(times) * 1000
        p95_query = np.percentile(times, 95) * 1000
        
        print(f"  🔍 Query (top-10):  avg={avg_query:.1f}ms  p95={p95_query:.1f}ms")
        
        # ─── Benchmark: Filtered query ───
        times_filtered = []
        for _ in range(20):
            start = time.time()
            collection.query(
                query_embeddings=[query_emb],
                n_results=10,
                where={"topic": "tech"},
            )
            times_filtered.append(time.time() - start)
        
        avg_filtered = np.mean(times_filtered) * 1000
        p95_filtered = np.percentile(times_filtered, 95) * 1000
        
        print(f"  🔍 Filtered query:  avg={avg_filtered:.1f}ms  p95={p95_filtered:.1f}ms")
        
        # ─── Benchmark: Different result sizes ───
        for k in [1, 10, 50, 100]:
            if k > n:
                break
            times_k = []
            for _ in range(10):
                start = time.time()
                collection.query(query_embeddings=[query_emb], n_results=k)
                times_k.append(time.time() - start)
            avg_k = np.mean(times_k) * 1000
            print(f"  🔍 Top-{k:<4}:        avg={avg_k:.1f}ms")
        
        # ─── Benchmark: Recall verification ───
        # Insert a known vector, then search for it
        known_emb = np.random.randn(1536).tolist()
        collection.add(
            ids=["recall_test"],
            documents=["Known recall test document"],
            embeddings=[known_emb],
        )
        
        result = collection.query(query_embeddings=[known_emb], n_results=1)
        recall_passed = result["ids"][0][0] == "recall_test"
        print(f"  🎯 Recall check:   {'✅ PASS' if recall_passed else '❌ FAIL'}")
        
        results.append({
            "n": n,
            "index_time_s": round(index_time, 2),
            "throughput_docs_s": round(throughput, 0),
            "query_avg_ms": round(avg_query, 1),
            "query_p95_ms": round(p95_query, 1),
            "filtered_avg_ms": round(avg_filtered, 1),
            "recall_pass": recall_passed,
        })
    
    # ─── Summary table ───
    print(f"\n{'=' * 80}")
    print("BENCHMARK SUMMARY")
    print(f"{'=' * 80}")
    print(f"{'Docs':>8} | {'Index (s)':>10} | {'Throughput':>12} | {'Query avg':>10} | {'Query p95':>10} | {'Filtered':>10} | {'Recall':>8}")
    print(f"{'-'*8}-+-{'-'*10}-+-{'-'*12}-+-{'-'*10}-+-{'-'*10}-+-{'-'*10}-+-{'-'*8}")
    
    for r in results:
        print(f"{r['n']:>8,} | {r['index_time_s']:>10.2f} | {r['throughput_docs_s']:>10,.0f}/s | {r['query_avg_ms']:>8.1f}ms | {r['query_p95_ms']:>8.1f}ms | {r['filtered_avg_ms']:>8.1f}ms | {'✅' if r['recall_pass'] else '❌':>8}")
    
    print(f"\n💡 KEY OBSERVATIONS:")
    print(f"   - ChromaDB uses HNSW index: query time grows ~O(log N)")
    print(f"   - Filtered queries may be slower due to post-filtering")
    print(f"   - For >100K docs, consider Qdrant or pgvector for better performance")
    print(f"   - Batch size affects indexing throughput significantly")


if __name__ == "__main__":
    benchmark_chromadb()
```

---

## Part 3: Review & Reflection (30 min)

### What You Learned Today

1. **An embedding captures:** ________________________________
2. **Cosine similarity is preferred because:** ________________________________
3. **HNSW enables fast search by:** ________________________________
4. **Metadata filtering is important because:** ________________________________
5. **ChromaDB is best for:** ________________________________

### Embedding Decision Tree
```
What do you need?
├── Simple similarity search → Cosine similarity + brute force
├── <10K documents → ChromaDB (embedded, zero setup)
├── <1M documents → Qdrant or pgvector (server mode)
├── >1M documents → Milvus or Pinecone (distributed)
├── Already have PostgreSQL → pgvector extension
├── Need multi-modal (images + text) → Weaviate
└── Pure research/speed → FAISS library
```

### Homework: Day 3 Deliverable

1. **Push all three lab scripts** to GitHub (`day03/`)
2. **Embedding visualization** — save the t-SNE plot as `embedding_visualization.png`
3. **Semantic search engine** — demonstrate 5 different query types (basic, filtered, hybrid, threshold, off-topic)
4. **Benchmark report** — table comparing ChromaDB performance at 100, 500, 1K, and 5K documents
5. **Write a 1-paragraph analysis:** When would you use ChromaDB vs. a managed vector database?

---

*Tomorrow: Day 4 — Production RAG Part 1 (chunking strategies, retrieval pipelines, hybrid search)*
