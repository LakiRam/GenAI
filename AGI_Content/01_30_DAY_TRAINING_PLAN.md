# 30-Day AI/ML Mastery Plan — Hands-On Engineering Manager Track

> **Goal:** Go from foundational understanding to production-ready AI engineering skills in 30 days.
> **Time commitment:** 3-4 hours/day (weekdays), 5-6 hours/day (weekends)
> **Prerequisites:** Python proficiency, basic ML concepts, software engineering experience
> **Setup needed:** Python 3.11+, VS Code, Azure/OpenAI API keys, GitHub account, Docker

---

## WEEK 1: FOUNDATIONS — LLMs, Prompting & RAG (Days 1-7)

### Day 1: LLM Fundamentals — How They Actually Work
**Theory (1.5 hrs):**
- Transformer architecture: attention mechanism, positional encoding, tokenization
- Pre-training vs. fine-tuning vs. RLHF vs. DPO
- Decoder-only (GPT) vs. encoder-decoder (T5) vs. encoder-only (BERT)
- Token economics: context windows, token counting, cost per million tokens
- Temperature, top-p, top-k — what they do and when to change them

**Hands-on (2 hrs):**
```
Lab 1.1: Set up your AI development environment
- Install Python 3.11+, create virtual environment
- Install: openai, anthropic, langchain, chromadb, tiktoken
- Get API keys: OpenAI, Azure OpenAI (or use local Ollama with Llama 3.1)
- Write a script that calls OpenAI API, counts tokens, measures latency
- Experiment: Same prompt with temperature 0 vs 0.7 vs 1.5

Lab 1.2: Tokenization deep-dive
- Use tiktoken to tokenize text in different encodings (cl100k, o200k)
- Compare token counts across models
- Understand why "tokenization matters" for cost and context window management
```

**Deliverable:** A Python notebook comparing responses across 3 temperature settings with token count analysis.

---

### Day 2: Prompt Engineering Mastery
**Theory (1 hr):**
- Zero-shot, few-shot, chain-of-thought (CoT), tree-of-thought
- System prompts, role definition, output formatting
- Structured output (JSON mode, function calling, tool use)
- Prompt injection attacks and defenses
- Meta-prompting: prompts that generate prompts

**Hands-on (2.5 hrs):**
```
Lab 2.1: Build a prompt engineering test harness
- Create a prompt testing framework that:
  - Takes a prompt template + variables
  - Runs against multiple models (GPT-4o, Claude, Llama)
  - Compares outputs side-by-side
  - Logs cost and latency
  
Lab 2.2: Solve real problems with prompt engineering
- Task 1: Extract structured data from messy text (receipts, emails)
- Task 2: Multi-step reasoning (math word problems with CoT)
- Task 3: Code generation with test-case validation
- Task 4: Build a prompt that resists injection attacks

Lab 2.3: Structured output
- Use OpenAI's JSON mode and function calling
- Parse complex documents into typed Pydantic models
- Handle edge cases: partial data, ambiguous fields
```

**Deliverable:** A prompt testing framework + 4 optimized prompt templates with before/after quality comparisons.

---

### Day 3: Embeddings & Vector Databases
**Theory (1 hr):**
- What are embeddings? Geometric intuition of semantic similarity
- Embedding models: OpenAI text-embedding-3-small/large, BGE, E5, Cohere
- Vector similarity: cosine, dot product, euclidean
- Vector database architectures: HNSW, IVF, PQ
- Choosing an embedding model: dimensions, performance, cost

**Hands-on (2.5 hrs):**
```
Lab 3.1: Embedding exploration
- Embed 100 sentences, visualize with t-SNE/UMAP
- Measure similarity between related and unrelated pairs
- Compare embedding models: OpenAI vs. open-source (BGE-large)

Lab 3.2: Build a semantic search engine
- Load 500+ documents (use Wikipedia articles or your own docs)
- Chunk documents (experiment: 256, 512, 1024 token chunks)
- Embed and store in ChromaDB
- Build a search API that returns top-k similar documents
- Add metadata filtering (date, category, source)

Lab 3.3: Vector DB comparison
- Implement the same search in ChromaDB, Qdrant (local), and pgvector
- Benchmark: insertion speed, query speed, relevance
```

**Deliverable:** A working semantic search engine with benchmarks across chunk sizes and vector DBs.

---

### Day 4-5: Building Production RAG Systems (Weekend Block)
**Theory (2 hrs):**
- RAG architecture: naive RAG → advanced RAG → modular RAG
- Chunking strategies: fixed-size, recursive, semantic, document-aware
- Retrieval strategies: dense, sparse (BM25), hybrid, multi-query
- Reranking: cross-encoders, Cohere Rerank, ColBERT
- RAG failure modes: wrong retrieval, insufficient context, hallucination despite retrieval
- Context window management: stuffing, map-reduce, refine

**Hands-on (8 hrs across 2 days):**
```
Lab 4.1: Naive RAG (Day 4 - 3 hrs)
- Build end-to-end RAG over a PDF corpus (use 10+ technical docs)
- Components: PDF parser → chunker → embedder → vector store → retriever → LLM
- Use LangChain for orchestration
- Test with 20 questions, manually evaluate answers

Lab 4.2: Advanced RAG (Day 4 - 2 hrs)
- Add hybrid search (dense + BM25 via rank fusion)
- Implement a reranker (Cohere or cross-encoder)
- Add query transformation (HyDE, multi-query, step-back)
- Compare: naive vs. advanced RAG on the same 20 questions

Lab 5.1: Production RAG hardening (Day 5 - 3 hrs)
- Add citation/source tracking (which chunk answered the question)
- Implement conversation memory (multi-turn RAG)
- Add guardrails: relevance filtering, confidence thresholds
- Handle "I don't know" — when retrieved context isn't sufficient
- Add streaming responses

Lab 5.2: RAG evaluation (Day 5 - 3 hrs)
- Build an evaluation dataset (50 Q/A pairs with golden answers)
- Implement RAGAS metrics: faithfulness, answer relevance, context precision
- Run evaluations, identify failure modes, fix retrieval
- A/B test: different chunk sizes, different embedding models
```

**Deliverable:** A production-grade RAG system with evaluation pipeline showing metrics before/after improvements.

---

### Day 6: Function Calling & Tool Use
**Theory (1 hr):**
- Function calling / tool use across providers (OpenAI, Anthropic, Azure)
- JSON schema for function definitions
- Parallel tool calls, tool choice modes
- MCP (Model Context Protocol) — the emerging standard for tool integration
- Security: sandboxing, rate limiting, input validation for tools

**Hands-on (2.5 hrs):**
```
Lab 6.1: Build a tool-using assistant
- Define 5 tools: web search, calculator, database query, file operations, API calls
- Implement function calling with OpenAI
- Handle multi-step tool use (tool output → more reasoning → another tool)
- Add error handling for tool failures

Lab 6.2: Build an MCP server
- Create a simple MCP server that exposes 3 tools
- Connect it to Claude Desktop or VS Code
- Test tool discovery and invocation
```

**Deliverable:** A tool-using assistant that can chain multiple tool calls to solve complex queries.

---

### Day 7: Week 1 Capstone — Intelligent Document Assistant
**Full day project (5-6 hrs):**
```
Build an end-to-end Intelligent Document Assistant:
- Ingests PDFs, Word docs, and web pages
- Implements advanced RAG with hybrid search and reranking
- Supports tool use (web search for supplementary info, calculator for data)
- Has conversation memory
- Includes evaluation suite
- Provides streaming responses via a FastAPI endpoint
- Has basic monitoring (token usage, latency, retrieval scores logged)

Deploy locally, test with 30 questions, generate evaluation report.
```

**Deliverable:** Complete project on GitHub with README, evaluation report, and architecture diagram.

---

## WEEK 2: FINE-TUNING, AGENTS & MULTI-MODAL (Days 8-14)

### Day 8: Fine-Tuning Fundamentals
**Theory (1.5 hrs):**
- When to fine-tune vs. prompt engineer vs. RAG
- Full fine-tuning vs. LoRA vs. QLoRA — tradeoffs
- Data preparation: instruction format, JSONL, quality over quantity
- Hyperparameters: learning rate, epochs, batch size, LoRA rank
- Catastrophic forgetting and how to avoid it
- DPO (Direct Preference Optimization) vs. RLHF

**Hands-on (2.5 hrs):**
```
Lab 8.1: Fine-tune with OpenAI
- Prepare 100+ training examples for a specific task (e.g., customer support tone)
- Upload and fine-tune GPT-4o-mini via API
- Evaluate: fine-tuned vs. base model on held-out test set
- Calculate cost per training run

Lab 8.2: Fine-tune open-source with LoRA
- Use Unsloth or Hugging Face PEFT to fine-tune Llama 3.1 8B
- Train on a small dataset (200 examples)
- Compare: base model vs. LoRA fine-tuned on task accuracy
- Export and test the adapter
```

**Deliverable:** Two fine-tuned models (OpenAI + open-source) with comparative evaluation.

---

### Day 9: Advanced Fine-Tuning & Data Strategies
**Theory (1 hr):**
- Synthetic data generation for fine-tuning
- Data curation: deduplication, quality filtering, diversity
- Continued pre-training (domain adaptation)
- Merging adapters, multi-task fine-tuning
- Evaluation-driven fine-tuning (iterative improvement)

**Hands-on (2.5 hrs):**
```
Lab 9.1: Synthetic data pipeline
- Use GPT-4o to generate 500 training examples for a domain-specific task
- Implement quality filtering (automated + LLM-as-judge)
- Deduplicate using embeddings
- Fine-tune on synthetic data, evaluate quality

Lab 9.2: DPO training
- Create preference pairs (chosen vs. rejected completions)
- Train a model with DPO using TRL library
- Compare: SFT-only vs. SFT+DPO on alignment benchmarks
```

**Deliverable:** A synthetic data pipeline + DPO-trained model with evaluation metrics.

---

### Day 10: AI Agents — Architecture & Implementation
**Theory (1.5 hrs):**
- Agent architectures: ReAct, Plan-and-Execute, LLM Compiler
- Memory systems: short-term (conversation), long-term (vector store), episodic
- Planning strategies: task decomposition, self-reflection, iterative refinement
- Multi-agent systems: collaboration patterns, delegation, debate
- Production concerns: cost control, timeout handling, infinite loop prevention

**Hands-on (2.5 hrs):**
```
Lab 10.1: Build a ReAct agent from scratch
- Implement the thought → action → observation loop manually (no frameworks)
- Give it 5 tools (search, calculate, code_execute, read_file, ask_user)
- Test on 10 complex multi-step tasks
- Add: step limit, cost budget, error recovery

Lab 10.2: Build with LangGraph
- Implement the same agent using LangGraph's state machine
- Add conditional routing, parallel tool execution
- Implement human-in-the-loop checkpoints
- Visualize the agent's execution graph
```

**Deliverable:** A ReAct agent built from scratch + LangGraph version, with comparison analysis.

---

### Day 11-12: Multi-Agent Systems (Weekend Block)
**Theory (2 hrs):**
- Multi-agent patterns: supervisor, debate, pipeline, swarm
- Microsoft Agent Framework (successor to AutoGen + Semantic Kernel)
- Agent communication protocols
- Shared state and memory across agents
- Testing and debugging multi-agent systems

**Hands-on (8 hrs across 2 days):**
```
Lab 11.1: Supervisor pattern (Day 11)
- Build a supervisor agent that delegates to specialist agents:
  - Research Agent (web search + summarization)
  - Code Agent (code generation + execution)
  - Analysis Agent (data analysis + visualization)
- Supervisor decides routing based on user intent
- Implement shared memory between agents

Lab 11.2: Debate pattern (Day 11)
- Two agents debate a topic, third agent judges
- Implement turn-taking, argument tracking, consensus detection

Lab 12.1: Production multi-agent system (Day 12)
- Build an automated code review system:
  - Agent 1: Analyzes code for bugs
  - Agent 2: Checks security vulnerabilities
  - Agent 3: Reviews code style and best practices
  - Supervisor: Aggregates findings into a unified report
- Test on real GitHub PRs
- Add cost tracking per agent

Lab 12.2: Agent evaluation (Day 12)
- Build an evaluation framework for agents:
  - Task completion rate
  - Step efficiency (did it take unnecessary steps?)
  - Cost per task
  - Error recovery rate
- Run eval on 20 tasks, analyze results
```

**Deliverable:** Multi-agent code review system with evaluation framework and cost analysis.

---

### Day 13: Multi-Modal AI
**Theory (1 hr):**
- Vision-Language Models (VLMs): GPT-4V, Claude Vision, Gemini
- Audio: Whisper (transcription), text-to-speech (ElevenLabs, Azure Speech)
- Image generation: DALL-E 3, Stable Diffusion
- Multi-modal RAG: images + text retrieval
- Document AI: OCR, layout analysis, form extraction

**Hands-on (2.5 hrs):**
```
Lab 13.1: Vision analysis pipeline
- Build an image analysis system:
  - Upload images → GPT-4V/Claude describes and extracts info
  - Process receipts, diagrams, screenshots
  - Structured output: extract tables from images into JSON

Lab 13.2: Multi-modal RAG
- Build RAG over a corpus that mixes text + images
- Embed images using CLIP or multi-modal embedding models
- Retrieve and display both text chunks and relevant images
- Test: "Show me the architecture diagram for X"
```

**Deliverable:** A multi-modal document processing pipeline handling text + images.

---

### Day 14: Week 2 Capstone — AI-Powered Research Assistant
**Full day project (5-6 hrs):**
```
Build a multi-agent research assistant:
- Agent 1 (Researcher): Searches the web, reads papers, extracts key findings
- Agent 2 (Analyst): Analyzes data, creates charts, identifies trends
- Agent 3 (Writer): Synthesizes findings into a structured report
- Supervisor: Orchestrates the workflow, manages quality

Features:
- Multi-modal: handles PDFs with images, tables, charts
- RAG over a knowledge base of previous research
- Exports formatted Markdown reports
- Cost tracking dashboard
- Human-in-the-loop for key decisions

Test cases: Generate research reports on 3 different industry topics.
```

**Deliverable:** Complete multi-agent research assistant with 3 sample research reports.

---

## WEEK 3: PRODUCTION SYSTEMS — MLOps, Serving & Scale (Days 15-21)

### Day 15: Model Serving & Optimization
**Theory (1.5 hrs):**
- Serving frameworks: vLLM, TGI, Triton, Ollama, NVIDIA NIM
- Optimization techniques: quantization (GPTQ, AWQ, GGUF), KV-cache, continuous batching
- Latency optimization: speculative decoding, prompt caching, streaming
- GPU memory management: model parallelism, tensor parallelism
- Cost optimization: model routing, caching, request batching

**Hands-on (2.5 hrs):**
```
Lab 15.1: Self-host an open model
- Deploy Llama 3.1 8B using vLLM on local GPU (or CPU with GGUF via Ollama)
- Benchmark: tokens/sec, latency p50/p95/p99, throughput
- Apply quantization (4-bit, 8-bit), measure quality impact
- Compare: vLLM vs. Ollama serving

Lab 15.2: Smart model routing
- Build a router that sends requests to different models based on:
  - Complexity (simple → small model, complex → large model)
  - Cost budget (stay under $X/day)
  - Latency requirements
- Implement caching for repeated queries (semantic cache)
```

**Deliverable:** A self-hosted model with benchmarks + model routing system.

---

### Day 16: LLMOps — Versioning, Testing & CI/CD
**Theory (1 hr):**
- Prompt versioning strategies (git-based, registry-based)
- Testing LLM applications: deterministic tests, fuzzy matching, LLM-as-judge
- CI/CD for AI: prompt regression testing, eval-gated deployments
- A/B testing and canary deployments for LLM features
- Configuration management: model versions, prompts, parameters

**Hands-on (2.5 hrs):**
```
Lab 16.1: Build an LLM testing pipeline
- Create a test suite with 3 layers:
  - Unit tests: prompt template rendering, tool schema validation
  - Integration tests: API calls return valid responses
  - Eval tests: quality metrics meet thresholds
- Use pytest + custom fixtures for LLM testing
- Implement LLM-as-judge for subjective quality

Lab 16.2: CI/CD pipeline
- Set up GitHub Actions workflow:
  - On PR: run eval tests against changed prompts
  - Compare metrics: new prompts vs. baseline
  - Auto-comment PR with eval results
  - Block merge if quality regresses
```

**Deliverable:** A complete LLMOps pipeline with automated testing and PR evaluation.

---

### Day 17: Monitoring, Observability & Debugging
**Theory (1 hr):**
- What to monitor: latency, cost, token usage, error rates, quality scores
- Traces: capturing the full chain (retrieval → augmentation → generation)
- Detecting hallucinations in production
- Alert design for LLM systems
- Debugging tools: LangSmith, Phoenix (Arize), Azure Monitor

**Hands-on (2.5 hrs):**
```
Lab 17.1: Instrument an LLM application
- Add OpenTelemetry tracing to your RAG system from Week 1
- Capture: retrieval latency, chunk relevance scores, generation time, token counts
- Create dashboards: cost/day, latency distribution, error rate
- Implement alerting: latency > threshold, cost spike, error burst

Lab 17.2: Hallucination detection
- Build a production hallucination detector:
  - Cross-reference generated claims against retrieved sources
  - Flag unsupported statements
  - Log hallucination rate over time
  - Alert when rate exceeds baseline
```

**Deliverable:** A fully instrumented LLM application with monitoring dashboard and hallucination detection.

---

### Day 18-19: Guardrails, Safety & Security (Weekend Block)
**Theory (2 hrs):**
- Prompt injection: direct, indirect, jailbreaks
- Data leakage: PII in prompts, training data extraction
- Content safety: toxicity detection, content filtering
- Access control: who can use which models/tools
- Guardrails frameworks: Guardrails AI, NeMo Guardrails, Azure Content Safety
- Compliance: EU AI Act, GDPR, model cards, bias auditing

**Hands-on (8 hrs across 2 days):**
```
Lab 18.1: Prompt injection defense (Day 18)
- Build a comprehensive prompt injection test suite:
  - 50+ injection attempts (direct, indirect, encoded)
  - Test your RAG system against each
  - Implement defenses: input validation, output filtering, sandwich defense
  - Measure: attack success rate before vs. after defenses

Lab 18.2: PII protection (Day 18)
- Build a PII detection and redaction pipeline:
  - Detect PII in user inputs (before sending to LLM)
  - Redact PII in LLM outputs
  - Use: regex + NER model + LLM-based detection
  - Test with 100 inputs containing various PII types

Lab 19.1: Guardrails system (Day 19)
- Implement a comprehensive guardrails layer:
  - Input guardrails: topic restriction, language detection, toxicity filter
  - Output guardrails: factuality check, format validation, brand safety
  - Use NeMo Guardrails for conversational guardrails
  - Build a guardrails config that handles 10 different safety scenarios

Lab 19.2: Red-teaming your system (Day 19)
- Conduct a red-team exercise on your Week 1 RAG system:
  - Generate adversarial inputs using an LLM
  - Test: prompt injection, jailbreaks, data extraction, bias
  - Document findings in a security report
  - Implement fixes and re-test
```

**Deliverable:** A security-hardened AI system with red-team report and guardrails configuration.

---

### Day 20: Scalable Architecture & Cost Engineering
**Theory (1 hr):**
- Architecture patterns: sync API, async queue, streaming, batch
- Scaling: horizontal (more instances), vertical (better hardware)
- Caching strategies: exact match, semantic cache, prompt cache
- Cost engineering: token optimization, model selection, batching
- Multi-region deployment, failover, and latency optimization

**Hands-on (2.5 hrs):**
```
Lab 20.1: Build a production API gateway for LLMs
- FastAPI service with:
  - Rate limiting per user/API key
  - Request queuing for high-load scenarios
  - Semantic caching (cache similar queries)
  - Token budget enforcement per user
  - Fallback: if primary model fails, route to secondary
  - Request/response logging for audit

Lab 20.2: Cost analysis dashboard
- Build a cost tracking system:
  - Log every API call with token counts and costs
  - Dashboard: daily cost, cost per endpoint, cost per user
  - Forecast: projected monthly cost at current rate
  - Alerts: daily budget exceeded, cost anomaly detected
```

**Deliverable:** A production LLM API gateway with caching, rate limiting, and cost dashboard.

---

### Day 21: Week 3 Capstone — Production RAG Service
**Full day project (5-6 hrs):**
```
Productionize your Week 1 RAG system:
- FastAPI service with streaming responses
- Authentication and rate limiting
- Semantic caching layer
- Full observability (traces, metrics, logs)
- Guardrails (input/output validation, PII protection)
- Evaluation pipeline that runs daily
- Cost tracking and alerting
- Docker containerized, docker-compose for local deployment
- Load test with 100 concurrent users (use locust)
- Document: architecture diagram, API docs, runbook

This is what a production AI system looks like.
```

**Deliverable:** Dockerized production RAG service with full documentation and load test results.

---

## WEEK 4: ADVANCED TOPICS & STRATEGIC MASTERY (Days 22-30)

### Day 22: Continuous Learning & Retraining
**Theory (1.5 hrs):**
- When and why to retrain/fine-tune
- Data flywheel: production data → curation → retraining → better model
- Active learning: identifying high-value examples from production
- Online learning vs. batch retraining
- A/B testing model versions in production
- Model regression detection

**Hands-on (2.5 hrs):**
```
Lab 22.1: Build a data flywheel
- Collect production logs from your RAG system
- Implement feedback collection (user thumbs up/down)
- Build a data curation pipeline:
  - Filter low-quality interactions
  - Identify patterns in negative feedback
  - Generate fine-tuning data from high-quality interactions
- Fine-tune on curated production data
- A/B test: original vs. retrained model

Lab 22.2: Automated retraining pipeline
- Build a pipeline that:
  - Monitors model quality metrics
  - When quality drops below threshold, triggers:
    - Data curation from recent production logs
    - Fine-tuning run
    - Evaluation on held-out test set
    - If eval passes, deploy new model
    - If eval fails, alert and keep old model
```

**Deliverable:** An automated retraining pipeline with quality gates.

---

### Day 23: Advanced RAG Patterns
**Theory (1 hr):**
- GraphRAG: knowledge graphs + RAG
- Self-RAG: model decides when to retrieve
- CRAG (Corrective RAG): verify and correct retrieval
- Agentic RAG: agent decides retrieval strategy dynamically
- Long-context vs. RAG: when 100K+ context windows eliminate RAG

**Hands-on (2.5 hrs):**
```
Lab 23.1: GraphRAG
- Build a knowledge graph from your document corpus
- Use Neo4j or NetworkX for graph storage
- Implement graph-based retrieval (entity → related entities → context)
- Compare: vector RAG vs. GraphRAG vs. hybrid on 20 complex questions

Lab 23.2: Agentic RAG
- Build an agent that dynamically decides:
  - Whether to retrieve (vs. answer from knowledge)
  - What query to use for retrieval
  - Whether retrieved context is sufficient (if not, re-retrieve)
  - Whether to do web search for supplementary info
- Compare: static RAG vs. agentic RAG on challenging questions
```

**Deliverable:** GraphRAG and Agentic RAG implementations with comparative evaluation.

---

### Day 24: Small Models, Edge AI & Distillation
**Theory (1 hr):**
- Small Language Models: Phi-3, Gemma 2, Llama 3.2 1B/3B
- Knowledge distillation: large model → small model
- On-device inference: ONNX Runtime, Core ML, TensorRT
- Use cases: offline, low-latency, privacy-preserving, cost-sensitive
- When small beats big: narrow tasks, classification, extraction

**Hands-on (2.5 hrs):**
```
Lab 24.1: Distillation pipeline
- Use GPT-4o to generate 1000 high-quality examples for a specific task
- Fine-tune Phi-3-mini or Llama 3.2 3B on this data
- Compare: GPT-4o vs. distilled small model on task accuracy
- Measure: cost per inference (100x cheaper?), latency (10x faster?)

Lab 24.2: Local AI deployment
- Deploy a small model with Ollama
- Build a privacy-preserving application:
  - All inference happens locally
  - No data leaves the device
  - Handles PII-heavy use case (medical notes, legal docs)
- Benchmark: latency on CPU vs. GPU
```

**Deliverable:** A distilled small model that matches GPT-4o quality on a narrow task at 100x lower cost.

---

### Day 25-26: Real-World Case Study Implementation (Weekend Block)
**See `02_CASE_STUDIES.md` for detailed case studies.**

Choose and implement 2 of the 6 case studies fully over this weekend.

---

### Day 27: AI Strategy & Decision Frameworks for Managers
**Theory (3 hrs):**
- Build vs. buy vs. fine-tune decision matrix
- Vendor evaluation framework (model providers, infra, tools)
- Team structure: roles, skills, hiring priorities
- Project estimation for AI projects (why it's different from traditional software)
- ROI calculation for AI projects
- Risk management: model failures, vendor lock-in, regulatory changes
- AI governance: model cards, fairness audits, decision logs

**Hands-on (1 hr):**
```
Lab 27.1: Decision framework
- For 5 real-world scenarios, apply the build/buy/fine-tune framework:
  1. Customer support chatbot for 50-person company
  2. Legal document review for law firm
  3. Code review automation for engineering team
  4. Product recommendation for e-commerce
  5. Medical report summarization for hospital
- Document: decision, reasoning, estimated cost, timeline, risks
```

**Deliverable:** A decision framework document with 5 analyzed scenarios.

---

### Day 28: Emerging Technologies & Future Trends
**Theory (2 hrs):**
- Reasoning models: o1, o3, DeepSeek-R1 — what's different
- Mixture of Experts (MoE) architectures
- State space models (Mamba, S4) — beyond transformers?
- Multimodal native models (Gemini, GPT-4o native audio)
- World models and video generation (Sora, Gen-3)
- AI agents in the wild: Devin, Manus AI, OpenAI Operator
- Open vs. closed source: the ongoing battle
- Regulation landscape: EU AI Act, US executive orders

**Hands-on (2 hrs):**
```
Lab 28.1: Reasoning models
- Compare standard GPT-4o vs. o3-mini on complex reasoning tasks:
  - Math competition problems
  - Code debugging (tricky bugs)
  - Multi-step logical reasoning
- When is the extra cost/latency of reasoning models worth it?

Lab 28.2: Future architecture exploration
- Deploy a Mixture-of-Experts model (Mixtral, DeepSeek-V3) locally
- Benchmark vs. dense model of similar size
- Understand: when MoE wins on efficiency
```

**Deliverable:** Comparative analysis of reasoning models + MoE benchmarks.

---

### Day 29: Final Capstone — End-to-End AI Product
**Full day project (6 hrs):**
```
Build a complete AI product from scratch:

"AI-Powered Technical Documentation Assistant"
- Ingests: GitHub repos, API docs, Confluence pages, Slack threads
- Multi-agent architecture:
  - Ingestion Agent: crawls, chunks, indexes new content
  - Search Agent: finds relevant info using hybrid search + reranking
  - Answer Agent: generates accurate answers with citations
  - Quality Agent: evaluates answer quality, flags hallucinations
- Production features:
  - User authentication (API keys)
  - Streaming responses
  - Conversation memory
  - Feedback collection
  - Cost tracking per user
  - Guardrails: topic restriction, PII redaction
  - Monitoring: latency, quality, cost dashboards
- Deployment: Docker Compose (API + vector DB + monitoring)
- Evaluation: 50-question eval suite with automated scoring
- Documentation: architecture diagram, API docs, operations runbook
```

**Deliverable:** A portfolio-worthy AI product deployed locally with full documentation.

---

### Day 30: Synthesis, Portfolio & Roadmap
**Activities (4 hrs):**
```
1. Portfolio Assembly (1.5 hrs)
   - Organize all 30 days of work into a GitHub portfolio
   - Each project gets: README, architecture diagram, evaluation results
   - Write a blog-style summary of your journey and key learnings

2. Knowledge Assessment (1 hr)
   - Take the self-assessment quiz (see 03_ASSESSMENT.md)
   - Identify remaining gaps
   - Create a personal study plan for the next 90 days

3. Architecture Exercise (1.5 hrs)
   - Design (whiteboard-style) architectures for 3 scenarios:
     - AI-powered customer support platform (10K tickets/day)
     - Internal knowledge base for 5000 employees
     - Real-time fraud detection with LLM-based reasoning
   - Document: components, model choices, scaling strategy, cost estimate, risks
```

**Deliverable:** Complete portfolio + self-assessment + 3 architecture designs.

---

## Daily Routine Template

| Time Block | Activity | Duration |
|------------|----------|----------|
| Block 1 | Theory: Read/watch materials | 1-1.5 hrs |
| Block 2 | Hands-on: Labs and coding | 2-2.5 hrs |
| Block 3 | Review: Document learnings, update portfolio | 30 min |

---

## Required Tools & Accounts

| Tool | Purpose | Cost |
|------|---------|------|
| Python 3.11+ | Primary language | Free |
| VS Code | IDE | Free |
| OpenAI API | GPT-4o, embeddings | ~$30-50 for 30 days |
| Azure OpenAI | Enterprise alternative | Pay-as-you-go |
| Ollama | Local model serving | Free |
| Docker | Containerization | Free |
| ChromaDB | Vector database | Free (open-source) |
| GitHub | Code hosting | Free |
| Hugging Face | Models and datasets | Free tier |

---

## Recommended Learning Resources

### Books
- "Building LLM Apps" by Valentina Alto
- "Designing Machine Learning Systems" by Chip Huyen
- "AI Engineering" by Chip Huyen (2025)

### Courses
- DeepLearning.AI: "LangChain for LLM Application Development"
- DeepLearning.AI: "Building Agentic RAG Applications"
- fast.ai: Practical Deep Learning (foundational)

### Newsletters & Blogs
- The Batch (Andrew Ng)
- Simon Willison's blog (practical LLM engineering)
- Lilian Weng's blog (deep technical)
- Latent Space podcast
- Ahead of AI by Sebastian Raschka

---

*Next: See `02_CASE_STUDIES.md` for detailed implementation case studies.*
*See `03_ASSESSMENT.md` for self-assessment quiz.*
