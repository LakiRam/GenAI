# Day 1: LLM Fundamentals — How They Actually Work

> **Time:** 3.5 hours (1.5 hrs theory + 2 hrs hands-on)
> **Goal:** Understand how LLMs work under the hood and set up your full AI development environment.
> **Deliverable:** A Python notebook comparing responses across 3 temperature settings with token count analysis.

---

## Part 1: Theory — The Transformer Revolution (1.5 hrs)

### 1.1 Why Transformers Changed Everything

Before 2017, sequence models (RNNs, LSTMs) processed text one word at a time — slow and forgetful over long sequences. The **Transformer** (Vaswani et al., "Attention Is All You Need", 2017) changed this by processing all words simultaneously using **self-attention**.

**Key insight:** Instead of reading left-to-right, the transformer looks at ALL words at once and learns which words are important to each other.

```
Sentence: "The cat sat on the mat because it was tired"

Q: What does "it" refer to?

Self-attention computes: "it" attends strongly to "cat" (not "mat")
→ The model learns coreference, context, and meaning through attention weights.
```

### 1.2 Transformer Architecture — The Building Blocks

```
┌─────────────────────────────────────────────────────────┐
│                    TRANSFORMER                          │
│                                                         │
│  Input: "The cat sat"                                   │
│     │                                                   │
│     ▼                                                   │
│  ┌──────────────────┐                                   │
│  │  Tokenization    │  "The" → 464, "cat" → 9246, ...  │
│  └────────┬─────────┘                                   │
│           ▼                                             │
│  ┌──────────────────┐                                   │
│  │  Token Embedding │  464 → [0.12, -0.34, 0.56, ...]  │
│  │  (dense vector)  │  Each token → vector of d dims    │
│  └────────┬─────────┘                                   │
│           ▼                                             │
│  ┌──────────────────┐                                   │
│  │  Positional      │  Add position info: word 1, 2, 3  │
│  │  Encoding        │  (transformers have no inherent    │
│  │                  │   notion of order otherwise)       │
│  └────────┬─────────┘                                   │
│           ▼                                             │
│  ┌──────────────────┐                                   │
│  │  Self-Attention  │  Each token attends to all others │
│  │  (Multi-Head)    │  Q, K, V matrices compute         │
│  │                  │  attention weights                 │
│  └────────┬─────────┘                                   │
│           ▼                                             │
│  ┌──────────────────┐                                   │
│  │  Feed-Forward    │  Two linear layers + activation   │
│  │  Network (FFN)   │  (this is where "knowledge" is    │
│  │                  │   stored in the parameters)        │
│  └────────┬─────────┘                                   │
│           ▼                                             │
│  ┌──────────────────┐                                   │
│  │  Layer Norm +    │  Repeat attention+FFN N times     │
│  │  Residual Conn.  │  (GPT-4: ~120 layers)            │
│  └────────┬─────────┘                                   │
│           ▼                                             │
│  ┌──────────────────┐                                   │
│  │  Output Layer    │  Predict next token probability   │
│  │  (Softmax)       │  "sat" → P("on")=0.32,           │
│  │                  │         P("down")=0.18, ...       │
│  └──────────────────┘                                   │
└─────────────────────────────────────────────────────────┘
```

### 1.3 Self-Attention — The Core Mechanism

Self-attention answers: **"For each word, how much should I pay attention to every other word?"**

```
Three matrices per attention head:
- Q (Query):  "What am I looking for?"
- K (Key):    "What do I contain?"
- V (Value):  "What information do I provide?"

Attention(Q, K, V) = softmax(Q × Kᵀ / √d_k) × V

Step-by-step:
1. Each token generates Q, K, V vectors
2. Compute attention scores: Q × Kᵀ (how well does each query match each key?)
3. Scale by √d_k (prevents scores from growing too large)
4. Softmax: convert scores to probabilities (sum to 1)
5. Multiply by V: weighted combination of values

Multi-Head: Run this N times in parallel with different Q, K, V projections
→ Each "head" can learn a different type of relationship
   (one head learns syntax, another learns coreference, etc.)
```

**Why this matters for you:** Understanding attention helps you:
- Debug why a model focuses on wrong context (retrieval problems in RAG)
- Understand context window limits (attention is O(n²) in memory)
- Know why long documents are hard (attention dilution)

### 1.4 Three Architectures You Must Know

| Architecture | Examples | How It Works | Best For |
|-------------|----------|-------------|----------|
| **Encoder-only** | BERT, RoBERTa | Reads entire input bidirectionally | Classification, NER, embeddings |
| **Decoder-only** | GPT-4, Claude, Llama | Reads left-to-right, generates next token | Text generation, chatbots, reasoning |
| **Encoder-Decoder** | T5, BART | Encoder reads input, decoder generates output | Translation, summarization |

**2024-2026 reality:** Decoder-only models dominate. Nearly all modern LLMs (GPT-4, Claude, Llama, Gemini) are decoder-only. Encoder models still matter for embeddings and classification.

```
Decoder-only (autoregressive):
  Input:  "The cat sat on the"
  Output: "mat" (one token at a time)
  
  Generation loop:
  "The cat sat on the" → "mat"
  "The cat sat on the mat" → "because"
  "The cat sat on the mat because" → "it"
  ... (repeat until <EOS> or max_tokens)
```

### 1.5 The Training Pipeline — How LLMs Are Built

```
Stage 1: PRE-TRAINING (months, millions of $)
┌─────────────────────────────────────────────────────────┐
│ Data: Trillions of tokens from internet, books, code    │
│ Objective: Predict the next token                       │
│ Result: Base model (knows language, facts, but no       │
│         manners — will not follow instructions well)    │
│ Example: Llama 3.1 base model                          │
│ Cost: $10M-$100M+ in compute                           │
└─────────────────────────────────────────────────────────┘
                        │
                        ▼
Stage 2: SUPERVISED FINE-TUNING / SFT (days, thousands of $)
┌─────────────────────────────────────────────────────────┐
│ Data: 10K-100K+ (instruction, response) pairs          │
│ Objective: Learn to follow instructions                 │
│ Result: Instruction-tuned model (follows prompts,       │
│         but may still be harmful or unhelpful)          │
│ Example: Llama 3.1 Instruct (before alignment)         │
│ Cost: $1K-$50K in compute                              │
└─────────────────────────────────────────────────────────┘
                        │
                        ▼
Stage 3: ALIGNMENT (days-weeks)
┌─────────────────────────────────────────────────────────┐
│ RLHF (Reinforcement Learning from Human Feedback):     │
│   1. Generate multiple responses                        │
│   2. Humans rank them (best to worst)                   │
│   3. Train a "reward model" on these rankings           │
│   4. Use PPO to optimize LLM to maximize reward         │
│                                                         │
│ DPO (Direct Preference Optimization) — simpler:        │
│   1. Collect (chosen, rejected) response pairs          │
│   2. Directly optimize the LLM to prefer "chosen"      │
│   3. No separate reward model needed                    │
│   (This is why DPO is replacing RLHF — simpler, works) │
│                                                         │
│ Result: Aligned model (helpful, harmless, honest)       │
│ Example: GPT-4, Claude 3.5, Llama 3.1 Instruct         │
└─────────────────────────────────────────────────────────┘
```

**Key takeaway for managers:** You will NOT do Stage 1 (pre-training). You may sometimes do Stage 2 (fine-tuning) for specialized tasks. You will focus on using Stage 3 models via APIs.

### 1.6 Tokenization — Why It Matters More Than You Think

Tokens are NOT words. They are sub-word units learned by algorithms like BPE (Byte-Pair Encoding).

```
"Hello world"           → ["Hello", " world"]           = 2 tokens
"Tokenization"          → ["Token", "ization"]           = 2 tokens
"supercalifragilistic"  → ["super", "cal", "ifrag", "ilistic"] = 4 tokens
"こんにちは"              → ["こん", "にち", "は"]          = 3 tokens (Japanese is expensive!)
"123456789"             → ["123", "456", "789"]           = 3 tokens

Why this matters:
- Cost is per token (not per word or character)
- Context window is in tokens
- Non-English languages use MORE tokens = cost more
- Code often tokenizes efficiently (common keywords are single tokens)
- Numbers tokenize poorly (arithmetic is hard because "1234" ≠ "1" + "2" + "3" + "4")
```

### 1.7 Decoding Parameters — Temperature, Top-p, Top-k

When the model predicts the next token, it outputs a probability distribution over ALL tokens in its vocabulary (~100K-200K tokens). The decoding parameters control how we select from this distribution.

```
Model predicts next token after "The cat sat on the":
  "mat"   → 32%
  "floor" → 18%
  "bed"   → 12%
  "roof"  → 8%
  "table" → 6%
  ... (thousands more with tiny probabilities)
```

#### Temperature
```
Temperature = 0 (deterministic):
  → Always picks "mat" (highest probability)
  → Same input = same output every time
  → USE FOR: Data extraction, classification, factual Q&A

Temperature = 0.7 (balanced):  
  → Redistribution: "mat" 40%, "floor" 25%, "bed" 15%, "roof" 10%
  → Some randomness, usually sensible choices
  → USE FOR: Creative writing, chatbots, general use

Temperature = 1.5 (very creative):
  → Redistribution: "mat" 20%, "floor" 15%, "bed" 12%, "roof" 11%, "trampoline" 8%
  → High randomness, unexpected tokens become likely
  → USE FOR: Brainstorming (with caution), never for production
```

**Mathematical formula:**
```
P_adjusted(token_i) = exp(logit_i / T) / Σ exp(logit_j / T)

Where T = temperature
- T < 1: sharpens distribution (more deterministic)
- T = 1: original distribution
- T > 1: flattens distribution (more random)
```

#### Top-p (Nucleus Sampling)
```
top_p = 0.9:
  → Find smallest set of tokens whose probabilities sum to ≥ 0.9
  → Sample only from that set
  → "mat"(32%) + "floor"(18%) + "bed"(12%) + "roof"(8%) + 
     "table"(6%) + "chair"(5%) + "rug"(4%) + "couch"(3%) + "ground"(3%) = 91%
  → Only these 9 tokens are candidates
  → Prevents sampling extremely unlikely tokens

top_p = 0.1:
  → Only "mat" qualifies (32% > 10%)
  → Very deterministic
```

#### Top-k
```
top_k = 5:
  → Only consider the top 5 tokens by probability
  → "mat", "floor", "bed", "roof", "table"
  → Simpler than top-p but less adaptive
```

**Practical guidance:**
| Use Case | Temperature | Top-p | Recommendation |
|----------|-------------|-------|---------------|
| Data extraction | 0 | 1.0 | Deterministic |
| Factual Q&A | 0-0.3 | 0.95 | Low creativity |
| Chatbot | 0.7 | 0.9 | Balanced |
| Creative writing | 0.9-1.2 | 0.95 | High creativity |
| Code generation | 0-0.2 | 0.95 | Deterministic |

### 1.8 Token Economics — What Things Cost

| Model | Input (per 1M tokens) | Output (per 1M tokens) | Context Window |
|-------|----------------------|------------------------|----------------|
| GPT-4o | $2.50 | $10.00 | 128K |
| GPT-4o-mini | $0.15 | $0.60 | 128K |
| Claude 3.5 Sonnet | $3.00 | $15.00 | 200K |
| Claude 3.5 Haiku | $0.80 | $4.00 | 200K |
| Llama 3.1 70B (self-hosted) | GPU cost only | GPU cost only | 128K |
| Llama 3.1 8B (self-hosted) | GPU cost only | GPU cost only | 128K |

**Quick cost estimation:**
```
1 page of text ≈ 500 tokens
1 typical chatbot turn (input + output) ≈ 800 tokens
1000 chatbot queries/day with GPT-4o:
  Input:  500 × 1000 = 500K tokens = 500K × $2.50/1M = $1.25
  Output: 300 × 1000 = 300K tokens = 300K × $10.00/1M = $3.00
  Daily cost: $4.25 → Monthly: ~$128

Same with GPT-4o-mini:
  Input: $0.075 + Output: $0.18 = $0.26/day → Monthly: ~$8
  
97% cost reduction for tasks where mini is sufficient!
```

---

## Part 2: Hands-On Labs (2 hrs)

### Lab 1.1 — Environment Setup & First API Call

#### Step 1: Create your project structure
```bash
# Create project directory
mkdir ai-engineering-30days
cd ai-engineering-30days

# Create virtual environment
python -m venv .venv

# Activate (Windows)
.venv\Scripts\activate

# Activate (Mac/Linux)
# source .venv/bin/activate
```

#### Step 2: Install dependencies
```bash
pip install openai anthropic tiktoken python-dotenv ipykernel notebook rich
```

#### Step 3: Set up API keys
Create a file named `.env` in your project root:
```env
OPENAI_API_KEY=sk-your-key-here
# Optional: If using Azure OpenAI
AZURE_OPENAI_API_KEY=your-key
AZURE_OPENAI_ENDPOINT=https://your-resource.openai.azure.com/
```

> **Security note:** NEVER commit `.env` to git. Add it to `.gitignore` immediately.

#### Step 4: Your first LLM call
Create `day01_lab1_first_call.py`:

```python
"""
Day 1 - Lab 1.1: First LLM API Call
Learn: API basics, response structure, token counting, latency measurement
"""

import time
import os
from openai import OpenAI
from dotenv import load_dotenv

load_dotenv()

client = OpenAI()  # Uses OPENAI_API_KEY from environment


def call_llm(prompt: str, model: str = "gpt-4o-mini", temperature: float = 0.7) -> dict:
    """Make an LLM API call and capture metadata."""
    start_time = time.time()
    
    response = client.chat.completions.create(
        model=model,
        messages=[
            {"role": "system", "content": "You are a helpful AI assistant."},
            {"role": "user", "content": prompt},
        ],
        temperature=temperature,
        max_tokens=500,
    )
    
    elapsed = time.time() - start_time
    
    return {
        "content": response.choices[0].message.content,
        "model": response.model,
        "input_tokens": response.usage.prompt_tokens,
        "output_tokens": response.usage.completion_tokens,
        "total_tokens": response.usage.total_tokens,
        "latency_seconds": round(elapsed, 2),
        "finish_reason": response.choices[0].finish_reason,
    }


# --- Run your first call ---
if __name__ == "__main__":
    result = call_llm("Explain what a transformer is in AI, in 3 sentences.")
    
    print("=" * 60)
    print("RESPONSE:")
    print(result["content"])
    print("=" * 60)
    print(f"Model:         {result['model']}")
    print(f"Input tokens:  {result['input_tokens']}")
    print(f"Output tokens: {result['output_tokens']}")
    print(f"Total tokens:  {result['total_tokens']}")
    print(f"Latency:       {result['latency_seconds']}s")
    print(f"Finish reason: {result['finish_reason']}")
    
    # Estimate cost (GPT-4o-mini pricing)
    input_cost = result["input_tokens"] * 0.15 / 1_000_000
    output_cost = result["output_tokens"] * 0.60 / 1_000_000
    total_cost = input_cost + output_cost
    print(f"Est. cost:     ${total_cost:.6f}")
```

Run it:
```bash
python day01_lab1_first_call.py
```

#### Step 5: Temperature experiment
Create `day01_lab2_temperature.py`:

```python
"""
Day 1 - Lab 1.2: Temperature Experiment
Learn: How temperature affects output diversity, quality, and determinism
"""

import time
from openai import OpenAI
from dotenv import load_dotenv

load_dotenv()

client = OpenAI()

PROMPT = "Write a one-paragraph product description for a pair of wireless headphones."
TEMPERATURES = [0, 0.3, 0.7, 1.0, 1.5]
RUNS_PER_TEMP = 3  # Run multiple times to see variance


def call_with_temperature(prompt: str, temp: float) -> dict:
    start = time.time()
    response = client.chat.completions.create(
        model="gpt-4o-mini",
        messages=[{"role": "user", "content": prompt}],
        temperature=temp,
        max_tokens=200,
    )
    return {
        "content": response.choices[0].message.content,
        "tokens": response.usage.completion_tokens,
        "latency": round(time.time() - start, 2),
    }


def measure_diversity(responses: list[str]) -> float:
    """Simple diversity metric: average pairwise word-set Jaccard distance."""
    if len(responses) < 2:
        return 0.0
    
    word_sets = [set(r.lower().split()) for r in responses]
    distances = []
    for i in range(len(word_sets)):
        for j in range(i + 1, len(word_sets)):
            intersection = len(word_sets[i] & word_sets[j])
            union = len(word_sets[i] | word_sets[j])
            if union > 0:
                distances.append(1 - intersection / union)
    return round(sum(distances) / len(distances), 3) if distances else 0.0


if __name__ == "__main__":
    print("=" * 70)
    print(f"PROMPT: {PROMPT}")
    print(f"Running {RUNS_PER_TEMP} calls per temperature setting...")
    print("=" * 70)
    
    results = {}
    
    for temp in TEMPERATURES:
        print(f"\n{'─' * 70}")
        print(f"TEMPERATURE = {temp}")
        print(f"{'─' * 70}")
        
        responses = []
        total_tokens = 0
        total_latency = 0
        
        for run in range(RUNS_PER_TEMP):
            result = call_with_temperature(PROMPT, temp)
            responses.append(result["content"])
            total_tokens += result["tokens"]
            total_latency += result["latency"]
            
            print(f"\n  Run {run + 1}:")
            print(f"  {result['content'][:150]}...")
        
        diversity = measure_diversity(responses)
        avg_tokens = total_tokens / RUNS_PER_TEMP
        avg_latency = total_latency / RUNS_PER_TEMP
        
        results[temp] = {
            "diversity": diversity,
            "avg_tokens": avg_tokens,
            "avg_latency": avg_latency,
        }
        
        print(f"\n  📊 Diversity score:  {diversity} (0=identical, 1=completely different)")
        print(f"  📊 Avg tokens:      {avg_tokens:.0f}")
        print(f"  📊 Avg latency:     {avg_latency:.2f}s")
    
    # Summary
    print(f"\n{'=' * 70}")
    print("SUMMARY")
    print(f"{'=' * 70}")
    print(f"{'Temp':<8} {'Diversity':<12} {'Avg Tokens':<12} {'Avg Latency':<12}")
    print(f"{'─' * 44}")
    for temp, data in results.items():
        print(f"{temp:<8} {data['diversity']:<12} {data['avg_tokens']:<12.0f} {data['avg_latency']:<12.2f}s")
    
    print(f"\n💡 KEY OBSERVATION:")
    print(f"   temp=0 → diversity={results[0]['diversity']} (deterministic, same output every time)")
    print(f"   temp=1.5 → diversity={results[1.5]['diversity']} (highly varied, sometimes incoherent)")
    print(f"   temp=0.7 → Good balance for most use cases")
```

Run it:
```bash
python day01_lab2_temperature.py
```

### Lab 1.2 — Tokenization Deep-Dive

Create `day01_lab3_tokenization.py`:

```python
"""
Day 1 - Lab 1.3: Tokenization Deep-Dive
Learn: How text becomes tokens, why it matters for cost and context windows
"""

import tiktoken


def analyze_tokenization(text: str, encoding_name: str = "o200k_base") -> dict:
    """Tokenize text and return detailed analysis."""
    enc = tiktoken.get_encoding(encoding_name)
    tokens = enc.encode(text)
    
    # Decode each token individually to see the sub-words
    token_strings = [enc.decode([t]) for t in tokens]
    
    return {
        "text": text,
        "encoding": encoding_name,
        "token_count": len(tokens),
        "token_ids": tokens,
        "token_strings": token_strings,
        "chars_per_token": round(len(text) / len(tokens), 2) if tokens else 0,
    }


def compare_encodings(text: str):
    """Compare token counts across different encodings."""
    encodings = {
        "cl100k_base": "GPT-4, GPT-3.5",
        "o200k_base": "GPT-4o, GPT-4o-mini",
    }
    
    print(f"\nText: \"{text}\"")
    print(f"Characters: {len(text)}")
    print(f"{'Encoding':<16} {'Model':<22} {'Tokens':<8} {'Chars/Token':<12}")
    print("─" * 58)
    
    for enc_name, models in encodings.items():
        enc = tiktoken.get_encoding(enc_name)
        tokens = enc.encode(text)
        ratio = round(len(text) / len(tokens), 2) if tokens else 0
        print(f"{enc_name:<16} {models:<22} {len(tokens):<8} {ratio:<12}")


def estimate_cost(text: str, model: str = "gpt-4o") -> dict:
    """Estimate the cost of processing this text as input."""
    pricing = {
        "gpt-4o": {"input": 2.50, "output": 10.00},
        "gpt-4o-mini": {"input": 0.15, "output": 0.60},
    }
    
    enc = tiktoken.encoding_for_model(model)
    token_count = len(enc.encode(text))
    
    input_cost = token_count * pricing[model]["input"] / 1_000_000
    
    return {
        "model": model,
        "tokens": token_count,
        "input_cost": f"${input_cost:.6f}",
        "per_1000_calls": f"${input_cost * 1000:.4f}",
        "per_1M_calls": f"${input_cost * 1_000_000:.2f}",
    }


if __name__ == "__main__":
    # --- Experiment 1: See how text gets tokenized ---
    print("=" * 60)
    print("EXPERIMENT 1: How text becomes tokens")
    print("=" * 60)
    
    test_texts = [
        "Hello, world!",
        "Tokenization is fundamental to understanding LLMs.",
        "The price is $42.99 for 3 items.",
        "def fibonacci(n): return n if n < 2 else fibonacci(n-1) + fibonacci(n-2)",
        "こんにちは世界",  # Japanese: "Hello world"
        "El gato se sentó en la alfombra.",  # Spanish
        "supercalifragilisticexpialidocious",
        "HTTP 404 Not Found: https://api.example.com/v2/users/12345",
        "    if (x > 0) {\n        return true;\n    }",
    ]
    
    for text in test_texts:
        result = analyze_tokenization(text)
        print(f"\nText: \"{text}\"")
        print(f"  Tokens ({result['token_count']}): {result['token_strings']}")
        print(f"  Chars/token: {result['chars_per_token']}")
    
    # --- Experiment 2: Compare encodings ---
    print(f"\n{'=' * 60}")
    print("EXPERIMENT 2: Token counts across model encodings")
    print("=" * 60)
    
    compare_texts = [
        "Explain the transformer architecture in machine learning.",
        "SELECT u.name, o.total FROM users u JOIN orders o ON u.id = o.user_id WHERE o.total > 100",
        "The quick brown fox jumps over the lazy dog.",
        "東京は日本の首都です。人口は約1400万人です。",  # Japanese about Tokyo
    ]
    
    for text in compare_texts:
        compare_encodings(text)
    
    # --- Experiment 3: Cost estimation ---
    print(f"\n{'=' * 60}")
    print("EXPERIMENT 3: Cost estimation")
    print("=" * 60)
    
    # Simulate a typical RAG prompt
    rag_prompt = """Based on the following context, answer the user's question.

Context:
The transformer architecture was introduced in 2017 by Vaswani et al. in the paper 
"Attention Is All You Need." It revolutionized natural language processing by replacing 
recurrent neural networks with a self-attention mechanism that processes all tokens in 
parallel. The key innovation is the multi-head attention mechanism, which allows the model 
to attend to different representation subspaces at different positions simultaneously. 
Transformers consist of an encoder and decoder, each made up of layers containing 
multi-head attention and feed-forward neural networks.

The transformer has become the foundation of nearly all modern language models, including 
GPT (Generative Pre-trained Transformer), BERT (Bidirectional Encoder Representations 
from Transformers), and T5 (Text-to-Text Transfer Transformer). These models are 
pre-trained on large corpora of text data and can be fine-tuned for specific downstream 
tasks such as text classification, question answering, and text generation.

Question: What is the key innovation in the transformer architecture?
Answer:"""
    
    print(f"\nRAG prompt length: {len(rag_prompt)} characters")
    
    for model in ["gpt-4o", "gpt-4o-mini"]:
        cost = estimate_cost(rag_prompt, model)
        print(f"\n  {cost['model']}:")
        print(f"    Tokens: {cost['tokens']}")
        print(f"    Cost per call (input only): {cost['input_cost']}")
        print(f"    Cost for 1,000 calls: {cost['per_1000_calls']}")
        print(f"    Cost for 1,000,000 calls: {cost['per_1M_calls']}")
    
    # --- Experiment 4: Context window math ---
    print(f"\n{'=' * 60}")
    print("EXPERIMENT 4: Context window planning")
    print("=" * 60)
    
    enc = tiktoken.encoding_for_model("gpt-4o")
    
    scenarios = [
        ("Short email", "Please review the Q3 report and let me know your thoughts."),
        ("1-page document", "A " * 500),  # ~500 tokens
        ("10-page document", "A " * 5000),  # ~5000 tokens
        ("100-page document", "A " * 50000),  # ~50000 tokens
    ]
    
    context_window = 128_000
    
    print(f"\nGPT-4o context window: {context_window:,} tokens")
    print(f"\n{'Scenario':<22} {'Tokens':<10} {'% of Context':<15} {'Remaining for Output':<20}")
    print("─" * 67)
    
    for name, text in scenarios:
        tokens = len(enc.encode(text))
        pct = round(tokens / context_window * 100, 1)
        remaining = context_window - tokens
        print(f"{name:<22} {tokens:<10,} {pct:<15}% {remaining:<20,}")
    
    print(f"\n💡 KEY INSIGHTS:")
    print(f"   1. Non-English text costs MORE tokens (Japanese ≈ 2-3x English)")
    print(f"   2. Code tokenizes efficiently (common keywords = single tokens)")
    print(f"   3. GPT-4o-mini is 16x cheaper than GPT-4o for input")
    print(f"   4. A 100-page doc uses ~39% of GPT-4o's context window")
    print(f"   5. Always estimate tokens BEFORE sending to avoid context overflow")
```

Run it:
```bash
python day01_lab3_tokenization.py
```

### Lab 1.3 — Streaming Responses

Create `day01_lab4_streaming.py`:

```python
"""
Day 1 - Lab 1.4: Streaming Responses
Learn: How streaming works, why it matters for UX, measuring time-to-first-token
"""

import time
from openai import OpenAI
from dotenv import load_dotenv

load_dotenv()

client = OpenAI()

PROMPT = "Explain the attention mechanism in transformers. Include a simple analogy."


def non_streaming_call(prompt: str) -> dict:
    """Standard non-streaming call — user waits for entire response."""
    start = time.time()
    response = client.chat.completions.create(
        model="gpt-4o-mini",
        messages=[{"role": "user", "content": prompt}],
        max_tokens=300,
    )
    total_time = time.time() - start
    content = response.choices[0].message.content
    
    return {
        "content": content,
        "total_time": round(total_time, 2),
        "time_to_first_char": round(total_time, 2),  # Same as total for non-streaming
        "tokens": response.usage.completion_tokens,
    }


def streaming_call(prompt: str) -> dict:
    """Streaming call — user sees tokens as they arrive."""
    start = time.time()
    first_token_time = None
    content = ""
    token_count = 0
    
    stream = client.chat.completions.create(
        model="gpt-4o-mini",
        messages=[{"role": "user", "content": prompt}],
        max_tokens=300,
        stream=True,
    )
    
    print("\n  Streaming output: ", end="", flush=True)
    
    for chunk in stream:
        if chunk.choices[0].delta.content:
            text = chunk.choices[0].delta.content
            
            if first_token_time is None:
                first_token_time = time.time() - start
            
            print(text, end="", flush=True)
            content += text
            token_count += 1
    
    total_time = time.time() - start
    print()  # newline after streaming
    
    return {
        "content": content,
        "total_time": round(total_time, 2),
        "time_to_first_token": round(first_token_time, 2) if first_token_time else 0,
        "tokens": token_count,
    }


if __name__ == "__main__":
    print("=" * 60)
    print("STREAMING vs NON-STREAMING COMPARISON")
    print("=" * 60)
    
    # Non-streaming
    print("\n1. NON-STREAMING (user sees nothing until complete):")
    ns_result = non_streaming_call(PROMPT)
    print(f"  Response: {ns_result['content'][:100]}...")
    print(f"  Total time:           {ns_result['total_time']}s")
    print(f"  Time to first char:   {ns_result['time_to_first_char']}s (same as total)")
    
    # Streaming
    print(f"\n2. STREAMING (user sees tokens as they arrive):")
    s_result = streaming_call(PROMPT)
    print(f"  Total time:           {s_result['total_time']}s")
    print(f"  Time to first token:  {s_result['time_to_first_token']}s")
    
    print(f"\n{'=' * 60}")
    print(f"💡 KEY INSIGHT:")
    print(f"   Non-streaming: User waits {ns_result['total_time']}s seeing NOTHING")
    print(f"   Streaming: User sees first token in {s_result['time_to_first_token']}s")
    print(f"   → Streaming feels {ns_result['total_time'] / max(s_result['time_to_first_token'], 0.01):.0f}x faster even though total time is similar")
    print(f"   → ALWAYS use streaming in user-facing applications")
```

### Lab 1.4 — Bonus: Compare Multiple Models

Create `day01_lab5_model_comparison.py`:

```python
"""
Day 1 - Lab 1.5: Model Comparison
Learn: How different models respond to the same prompt — quality, speed, cost tradeoffs
"""

import time
from openai import OpenAI
from dotenv import load_dotenv

load_dotenv()

client = OpenAI()

PROMPT = """Analyze the following customer review and extract:
1. Overall sentiment (positive/negative/mixed)
2. Key topics mentioned
3. Specific complaints or praise
4. Suggested action for the business

Review: "I've been using this project management tool for 6 months now. The interface 
is clean and intuitive, which I love. However, the mobile app crashes constantly — 
I've lost data twice this month. Customer support was responsive but couldn't fix the 
issue. The pricing is fair for what you get, but I'm considering switching if the 
mobile issues aren't resolved soon."
"""

MODELS = [
    {"name": "gpt-4o-mini", "input_price": 0.15, "output_price": 0.60},
    {"name": "gpt-4o", "input_price": 2.50, "output_price": 10.00},
]


def call_model(prompt: str, model: str) -> dict:
    start = time.time()
    response = client.chat.completions.create(
        model=model,
        messages=[
            {"role": "system", "content": "You are a data analyst. Respond in structured format."},
            {"role": "user", "content": prompt},
        ],
        temperature=0,
        max_tokens=500,
    )
    elapsed = time.time() - start
    
    return {
        "content": response.choices[0].message.content,
        "input_tokens": response.usage.prompt_tokens,
        "output_tokens": response.usage.completion_tokens,
        "latency": round(elapsed, 2),
    }


if __name__ == "__main__":
    print("=" * 70)
    print("MODEL COMPARISON — Same prompt, different models")
    print("=" * 70)
    
    results = {}
    
    for model_info in MODELS:
        model_name = model_info["name"]
        print(f"\n{'─' * 70}")
        print(f"MODEL: {model_name}")
        print(f"{'─' * 70}")
        
        result = call_model(PROMPT, model_name)
        
        # Calculate cost
        input_cost = result["input_tokens"] * model_info["input_price"] / 1_000_000
        output_cost = result["output_tokens"] * model_info["output_price"] / 1_000_000
        total_cost = input_cost + output_cost
        
        print(f"\nResponse:\n{result['content']}")
        print(f"\nMetrics:")
        print(f"  Latency:       {result['latency']}s")
        print(f"  Input tokens:  {result['input_tokens']}")
        print(f"  Output tokens: {result['output_tokens']}")
        print(f"  Cost:          ${total_cost:.6f}")
        print(f"  Cost at 10K calls/day: ${total_cost * 10_000:.2f}/day = ${total_cost * 10_000 * 30:.2f}/month")
        
        results[model_name] = {
            "latency": result["latency"],
            "cost": total_cost,
            "monthly_10k": total_cost * 10_000 * 30,
        }
    
    # Summary
    print(f"\n{'=' * 70}")
    print("DECISION FRAMEWORK")
    print(f"{'=' * 70}")
    print(f"\n{'Model':<18} {'Latency':<12} {'Cost/call':<14} {'Monthly (10K/day)':<18}")
    print(f"{'─' * 60}")
    for model, data in results.items():
        print(f"{model:<18} {data['latency']:<12}s ${data['cost']:<13.6f} ${data['monthly_10k']:<17.2f}")
    
    if len(results) >= 2:
        models_list = list(results.keys())
        cost_ratio = results[models_list[1]]["cost"] / max(results[models_list[0]]["cost"], 0.000001)
        print(f"\n💡 {models_list[1]} is {cost_ratio:.0f}x more expensive than {models_list[0]}")
        print(f"   → Use {models_list[0]} for simple tasks (extraction, classification)")
        print(f"   → Use {models_list[1]} for complex reasoning, nuanced analysis")
        print(f"   → Many production systems use BOTH: route by task complexity")
```

---

## Part 3: Review & Reflection (30 min)

### What You Learned Today
Fill in after completing the labs:

1. **Transformers process text by:** ________________________________
2. **Self-attention lets the model:** ________________________________
3. **Temperature controls:** ________________________________
4. **Temperature 0 is best for:** ________________________________
5. **Tokenization matters because:** ________________________________
6. **GPT-4o-mini vs GPT-4o tradeoff:** ________________________________
7. **Streaming matters because:** ________________________________

### Key Numbers to Remember
| Fact | Number |
|------|--------|
| Tokens per English word (average) | ~1.3 |
| GPT-4o context window | 128K tokens |
| GPT-4o-mini is ___ cheaper than GPT-4o | ~16x |
| 1 page of text ≈ | 500 tokens |
| Typical chatbot turn ≈ | 500-1000 tokens |

### Common Mistakes to Avoid
1. **Using GPT-4o for everything** — start with mini, upgrade only when quality demands it
2. **Ignoring token counts** — always measure; costs add up fast at scale
3. **Temperature > 1** in production — unpredictable outputs, use 0-0.7
4. **Not using streaming** — non-streaming feels broken to users for any response > 2s
5. **Hardcoding model names** — use config; you'll switch models frequently

---

## Homework: Day 1 Deliverable

Create a Jupyter notebook called `Day01_Deliverable.ipynb` that:

1. **Calls GPT-4o-mini** with a complex prompt at 5 different temperatures (0, 0.3, 0.7, 1.0, 1.5)
2. **Runs each temperature 3 times** and captures: response text, token count, latency
3. **Calculates diversity** (how different are responses at the same temperature?)
4. **Visualizes:**
   - A bar chart: token count vs. temperature
   - A chart: diversity score vs. temperature
5. **Includes a written analysis** (3-4 sentences) of when you would use each temperature setting
6. **Cost analysis:** What would 100K daily queries cost at each model tier?

Push to your GitHub repo: `ai-engineering-30days/day01/`

---

*Tomorrow: Day 2 — Prompt Engineering Mastery (zero-shot, few-shot, CoT, structured output, injection defense)*
