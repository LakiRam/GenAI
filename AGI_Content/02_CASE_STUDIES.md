# Case Studies — Hands-On Implementation Projects

> Each case study is designed to be completed in 4-6 hours.
> Choose at least 2 from Week 4 (Days 25-26) and attempt others as stretch goals.

---

## Case Study 1: Enterprise Knowledge Base with RAG
**Difficulty:** ★★★☆☆ | **Time:** 5 hrs | **Skills:** RAG, Vector DB, API Design

### Business Context
A company with 500 employees has 10,000+ internal documents (policies, SOPs, product docs, meeting notes) spread across SharePoint, Confluence, and Google Drive. Employees waste 2+ hours/day searching for information. Build an internal knowledge assistant.

### Requirements
1. Ingest documents from multiple formats (PDF, DOCX, HTML, Markdown)
2. Support conversational Q&A with follow-up questions
3. Always cite sources (document name, page/section)
4. Handle "I don't know" when context is insufficient
5. Role-based access: HR docs only visible to HR team
6. Admin panel: view usage stats, most-asked questions, unanswered queries

### Architecture
```
┌─────────────┐     ┌──────────────┐     ┌─────────────┐
│  Document    │────▶│  Ingestion   │────▶│  Vector DB  │
│  Sources     │     │  Pipeline    │     │  (ChromaDB) │
│  PDF/DOCX/MD │     │  chunk+embed │     │             │
└─────────────┘     └──────────────┘     └──────┬──────┘
                                                │
┌─────────────┐     ┌──────────────┐           │
│  User       │────▶│  FastAPI     │◀──────────┘
│  (Chat UI)  │     │  + RAG Chain │
│             │◀────│  + Auth      │────▶ LLM (GPT-4o)
└─────────────┘     └──────────────┘
```

### Implementation Steps

```python
# Step 1: Document Ingestion Pipeline
"""
Build a multi-format document ingester:
- PDFs: Use PyPDF2 or pdfplumber (handle tables, images)
- DOCX: Use python-docx
- Markdown/HTML: Use markdown-it or beautifulsoup
- Apply recursive text splitting (chunk_size=512, overlap=50)
- Add metadata: source, title, section, date, access_level
- Generate embeddings (OpenAI text-embedding-3-small)
- Store in ChromaDB with metadata filters
"""

# Step 2: Advanced Retrieval
"""
Implement hybrid search:
- Dense: vector similarity search (top-20)
- Sparse: BM25 keyword search (top-20)
- Combine with Reciprocal Rank Fusion (RRF)
- Rerank top-10 with a cross-encoder or Cohere Rerank
- Return top-5 chunks with scores

Add query transformation:
- If query is vague, generate 3 sub-queries
- If query references previous conversation, resolve co-references
"""

# Step 3: Generation with Citations
"""
Build the generation component:
- System prompt that enforces citation format
- Each answer must reference [Source: document_name, section]
- If confidence < threshold, respond with "I don't have enough information"
- Streaming responses via SSE

Template:
'You are a helpful assistant. Answer ONLY based on the provided context.
For every claim, cite the source as [Source: filename, section].
If the context doesn't contain the answer, say "I don't have enough
information to answer this question."'
"""

# Step 4: Conversation Memory
"""
Implement multi-turn conversation:
- Store conversation history per session
- Use history to resolve references ("What about the other policy?")
- Limit context window: summarize old turns if conversation grows long
"""

# Step 5: Access Control
"""
Role-based document access:
- Each document has access_level metadata (public, hr, engineering, finance)
- User authentication via API key with role claims
- Filter vector search results by user's role
- Log access for audit trail
"""

# Step 6: Admin Dashboard
"""
Build analytics endpoints:
- GET /admin/stats: total queries, avg response time, cost
- GET /admin/top-questions: most frequently asked topics
- GET /admin/unanswered: queries where system said "I don't know"
- GET /admin/feedback: user satisfaction ratings
"""
```

### Evaluation Criteria
| Metric | Target |
|--------|--------|
| Answer accuracy (vs golden answers) | > 85% |
| Citation accuracy | > 90% |
| "I don't know" precision (correctly abstains) | > 80% |
| Response latency (p95) | < 3 seconds |
| Cost per query | < $0.02 |

### Extension Ideas
- Add Slack integration (users query via Slack bot)
- Implement auto-indexing (new docs detected and ingested automatically)
- Add multilingual support

---

## Case Study 2: Customer Support Agent
**Difficulty:** ★★★★☆ | **Time:** 6 hrs | **Skills:** Agents, Tool Use, Guardrails, Evaluation

### Business Context
An e-commerce company handles 5,000 support tickets/day. 60% are repetitive (order status, returns, FAQs). Build an AI agent that handles L1 tickets autonomously and escalates L2+ to humans.

### Requirements
1. Handle: order status, return requests, FAQ answers, product info
2. Access tools: order DB, product catalog, return policy, CRM
3. Know when to escalate to human (complex issues, angry customers)
4. Never hallucinate order details — always query the database
5. Maintain professional, empathetic tone
6. Support conversation handoff to human agents with context

### Tools the Agent Has Access To
```python
tools = [
    {
        "name": "lookup_order",
        "description": "Look up order details by order ID or customer email",
        "parameters": {"order_id": "str", "customer_email": "str"}
    },
    {
        "name": "get_return_policy",
        "description": "Get return policy for a product category",
        "parameters": {"product_category": "str"}
    },
    {
        "name": "initiate_return",
        "description": "Start a return process for an order",
        "parameters": {"order_id": "str", "reason": "str", "items": "list"}
    },
    {
        "name": "search_faq",
        "description": "Search FAQ knowledge base",
        "parameters": {"query": "str"}
    },
    {
        "name": "search_product_catalog",
        "description": "Search for product information",
        "parameters": {"query": "str", "category": "str"}
    },
    {
        "name": "escalate_to_human",
        "description": "Escalate ticket to human agent with context summary",
        "parameters": {"reason": "str", "priority": "str", "context_summary": "str"}
    },
    {
        "name": "send_email",
        "description": "Send follow-up email to customer",
        "parameters": {"to": "str", "subject": "str", "body": "str"}
    }
]
```

### Implementation Steps

```python
# Step 1: Mock the backend systems
"""
Create mock services with realistic data:
- Orders DB: 1000 orders with various statuses
- Product catalog: 200 products with details
- Return policy: rules by category, time limits, conditions
- FAQ: 100 Q/A pairs
- CRM: customer history (past tickets, sentiment score)
"""

# Step 2: Build the agent with guardrails
"""
Agent design:
- System prompt: professional support agent persona
- Guardrails:
  - NEVER disclose internal pricing/margins
  - NEVER promise anything outside policy
  - ALWAYS verify customer identity before sharing order details
  - Detect frustration → acknowledge + offer escalation
  - PII: never echo full credit card numbers
- Escalation rules:
  - Customer asks for supervisor → escalate
  - Refund > $500 → escalate
  - Legal threat → escalate immediately
  - 3+ failed resolution attempts → escalate
"""

# Step 3: Implement conversation flows
"""
Build and test these flows:
1. "Where is my order?" → lookup_order → format status → respond
2. "I want to return this" → verify order → check policy → initiate_return
3. "How do I change my password?" → search_faq → respond
4. "This is ridiculous, I want a manager" → detect frustration → escalate
5. "Can you recommend something like X?" → search_product_catalog → respond
"""

# Step 4: Evaluation framework
"""
Build a comprehensive test suite:
- 50 test conversations covering all intents
- Automated evaluation:
  - Did the agent use the right tools? (tool accuracy)
  - Did it verify identity before sharing info? (security compliance)
  - Did it follow escalation rules? (policy compliance)
  - Was the tone professional and empathetic? (LLM-as-judge)
  - Did it hallucinate any order details? (factuality check)
"""

# Step 5: Analytics dashboard
"""
Track and display:
- Resolution rate (% handled without escalation)
- Average conversation turns to resolution
- Customer satisfaction (simulated feedback)
- Cost per resolution
- Escalation reasons breakdown
"""
```

### Evaluation Criteria
| Metric | Target |
|--------|--------|
| L1 resolution rate | > 70% |
| Tool selection accuracy | > 95% |
| Policy compliance | 100% |
| Factuality (no hallucinated data) | 100% |
| Escalation precision (should escalate → does) | > 90% |
| Avg turns to resolution | < 5 |
| Customer satisfaction (LLM-judged) | > 4.0/5.0 |

---

## Case Study 3: Automated Code Review System
**Difficulty:** ★★★★☆ | **Time:** 5 hrs | **Skills:** Multi-Agent, Code Analysis, Evaluation

### Business Context
An engineering team of 50 developers generates 30+ PRs/day. Code reviews are a bottleneck — senior developers spend 2+ hours daily reviewing. Build an AI code review system that provides first-pass reviews, catching common issues before human review.

### Architecture: Multi-Agent Code Review
```
               ┌─────────────────────┐
               │   Supervisor Agent   │
               │   (orchestrator)     │
               └──────────┬──────────┘
                          │
          ┌───────────────┼───────────────┐
          │               │               │
   ┌──────▼──────┐ ┌──────▼──────┐ ┌──────▼──────┐
   │  Bug Finder  │ │  Security   │ │  Style &    │
   │  Agent       │ │  Reviewer   │ │  Quality    │
   │              │ │  Agent      │ │  Agent      │
   └──────────────┘ └─────────────┘ └─────────────┘
```

### Implementation Steps

```python
# Step 1: Code parsing and context extraction
"""
Build a code analysis frontend:
- Parse Git diffs (new lines, deleted lines, context)
- Extract: function signatures, class hierarchies, imports
- Identify: what changed, what functions are affected
- Get file-level context (surrounding code, not just the diff)
"""

# Step 2: Specialist agents
"""
Bug Finder Agent:
- Analyzes code for logical errors, null references, off-by-one, race conditions
- Checks error handling patterns
- Identifies missing edge cases
- Severity rating: critical, warning, info

Security Reviewer Agent:
- OWASP Top 10 checklist for each change
- SQL injection, XSS, SSRF, path traversal
- Secret detection (API keys, passwords in code)
- Dependency vulnerability awareness
- Authentication/authorization issues

Style & Quality Agent:
- Code complexity analysis (cyclomatic complexity)
- Naming convention adherence
- DRY violations (duplicated code)
- Code documentation completeness
- Test coverage considerations
"""

# Step 3: Supervisor agent
"""
Supervisor orchestration:
- Receives PR diff
- Dispatches to all 3 specialist agents in parallel
- Collects findings
- Deduplicates similar findings
- Prioritizes: critical security > bugs > style
- Generates unified review report with:
  - Summary (1-2 sentences)
  - Critical findings (must fix before merge)
  - Warnings (should address)
  - Suggestions (nice to have)
  - Praise (what's done well — important for morale!)
"""

# Step 4: Evaluation
"""
Test on 20 real PRs (open-source projects):
- Compare AI review vs. actual human review comments
- Metrics:
  - True positive rate (found real issues)
  - False positive rate (flagged non-issues)
  - Coverage (% of human-found issues also found by AI)
  - Unique finds (issues AI found that humans missed)
"""
```

### Sample Output Format
```markdown
## AI Code Review Summary

**PR:** #1234 - Add user authentication middleware
**Risk Level:** 🟡 Medium

### 🔴 Critical (1)
1. **SQL Injection in user lookup** (security_reviewer)
   - File: `auth/middleware.py:45`
   - `query = f"SELECT * FROM users WHERE email = '{email}'"` — use parameterized queries
   - Fix: `cursor.execute("SELECT * FROM users WHERE email = %s", (email,))`

### 🟡 Warnings (2)  
1. **Missing rate limiting** (security_reviewer)
   - Login endpoint has no rate limiting — vulnerable to brute force
2. **Unhandled exception** (bug_finder)
   - `auth/middleware.py:62` — database connection failure not caught

### 💡 Suggestions (1)
1. **Consider extracting auth logic** (quality_agent)
   - Authentication logic in middleware could be a separate service for reuse

### 👍 What's Good
- Clean separation of concerns between auth and business logic
- Good use of type hints throughout
```

---

## Case Study 4: Real-Time Data Analysis Copilot
**Difficulty:** ★★★★★ | **Time:** 6 hrs | **Skills:** Text-to-SQL, Agents, Visualization, Streaming

### Business Context
A data team gets 50+ ad-hoc data requests daily from business stakeholders. "What were sales last quarter by region?" "Show me customer churn trends." Build a natural language data analysis copilot.

### Requirements
1. Natural language to SQL query generation
2. Automatic visualization (charts/graphs) based on data shape
3. Multi-turn analysis ("Now break that down by product category")
4. Safeguards: read-only queries, sensitive column masking, query cost estimation
5. Explain results in plain English

### Implementation Steps

```python
# Step 1: Database setup with sample data
"""
Create a realistic e-commerce database (SQLite or PostgreSQL):
- Tables: orders, customers, products, order_items, regions, campaigns
- 100K+ rows for realistic query performance
- Include: dates, amounts, categories, regions, customer segments
"""

# Step 2: Schema-aware text-to-SQL
"""
Build a text-to-SQL engine:
- Extract schema: table names, columns, types, relationships, sample values
- Include schema in system prompt (compact format)
- Support: aggregations, joins, window functions, CTEs
- Query validation: parse SQL, check for writes (block INSERT/UPDATE/DELETE)
- Query explanation: generate natural language explanation of what the SQL does
"""

# Step 3: Intelligent visualization
"""
Auto-visualization pipeline:
- Analyze query results: number of rows, column types, cardinality
- Decision logic:
  - Time series → line chart
  - Categorical comparison → bar chart
  - Distribution → histogram
  - Two numeric variables → scatter plot
  - Single value → big number display
  - Geographic → map
- Generate charts using matplotlib/plotly
- Add: titles, labels, formatting based on data
"""

# Step 4: Multi-turn analysis
"""
Conversation flow:
User: "What were total sales in Q4 2024?"
→ Generate SQL, execute, show result + bar chart

User: "Break that down by region"
→ Understand context, modify query to add GROUP BY region

User: "Which region had the highest growth vs Q3?"
→ Generate comparative query (Q4 vs Q3), calculate growth %

User: "Why did the West region decline?"
→ Drill down: by product, by customer segment, by channel
→ Identify potential root causes
"""

# Step 5: Safety and guardrails
"""
- Read-only: wrap all queries in read-only transactions
- Column masking: mask SSN, credit card, salary columns  
- Query cost estimation: EXPLAIN query before executing, warn if >10s
- Row limit: cap results at 10,000 rows
- Audit log: track all queries executed, by whom, when
"""
```

### Evaluation Criteria
| Metric | Target |
|--------|--------|
| Query correctness (valid SQL that answers the question) | > 85% |
| Visualization appropriateness | > 80% |
| Multi-turn context retention | > 90% |
| Security compliance (no write queries executed) | 100% |
| Response time (query + visualization) | < 5 seconds |

---

## Case Study 5: Document Intelligence Pipeline
**Difficulty:** ★★★☆☆ | **Time:** 4 hrs | **Skills:** Multi-modal, Extraction, Classification

### Business Context
A legal firm processes 200 contracts/day. They need to extract key clauses, classify risk level, compare against templates, and flag anomalies. Currently takes 30 min per contract manually.

### Requirements
1. Ingest contracts (PDF, scanned images, Word docs)
2. Extract: parties, dates, amounts, key clauses, obligations, termination conditions
3. Classify: contract type, risk level (low/medium/high/critical)
4. Compare against standard templates, flag deviations
5. Generate a one-page summary for each contract

### Implementation Steps

```python
# Step 1: Document ingestion
"""
Multi-format processing:
- PDF with text: pdfplumber for text extraction
- Scanned PDF/images: Azure Document Intelligence or Tesseract OCR
- DOCX: python-docx for structured extraction
- Maintain document structure: sections, paragraphs, tables
"""

# Step 2: Information extraction
"""
Use LLM for structured extraction:
- Design extraction schema (Pydantic models):
  - ContractMetadata: parties, date, type, jurisdiction
  - FinancialTerms: amounts, payment schedule, penalties
  - KeyClauses: termination, liability, indemnification, confidentiality
  - Obligations: deliverables, deadlines, conditions

- Extraction strategy:
  - For short contracts (<4K tokens): process entire document
  - For long contracts: extract section by section, merge results
  - Use structured output (JSON mode) for reliable parsing
"""

# Step 3: Risk classification
"""
Multi-factor risk scoring:
- Unusual clauses (deviation from templates)
- Missing standard clauses (no liability cap = high risk)
- Unfavorable terms (auto-renewal, exclusive jurisdiction)
- Financial exposure assessment
- Combine into overall risk score with explanation
"""

# Step 4: Template comparison
"""
- Maintain a library of "gold standard" contract templates
- For each new contract, find most similar template
- Diff: identify clauses that deviate from template
- Flag: missing clauses, modified clauses, added clauses
- Risk-weight each deviation
"""

# Step 5: Summary generation
"""
Generate a one-page executive summary:
- Contract type and parties
- Key dates and financial terms
- Top 3 risks with severity
- Notable deviations from standard
- Recommended actions (review clause X, negotiate term Y)
"""
```

### Sample Output
```json
{
  "contract_id": "C-2024-0892",
  "type": "Software License Agreement",
  "risk_level": "HIGH",
  "parties": {
    "licensor": "Acme Software Inc.",
    "licensee": "Customer Corp."
  },
  "key_dates": {
    "effective": "2024-01-15",
    "expiration": "2026-01-14",
    "auto_renews": true
  },
  "financial_summary": {
    "annual_fee": "$450,000",
    "payment_terms": "Net 30",
    "late_penalty": "1.5% per month"
  },
  "risk_factors": [
    {
      "severity": "HIGH",
      "clause": "Unlimited liability for licensor IP claims",
      "recommendation": "Negotiate liability cap"
    },
    {
      "severity": "MEDIUM",
      "clause": "Auto-renewal with 90-day notice requirement",
      "recommendation": "Add calendar reminder 120 days before renewal"
    }
  ],
  "template_deviations": 3,
  "summary": "High-risk SLA with unlimited liability exposure and auto-renewal lock-in..."
}
```

---

## Case Study 6: LLM-Powered CI/CD Quality Gate
**Difficulty:** ★★★★★ | **Time:** 6 hrs | **Skills:** LLMOps, CI/CD, Evaluation, Automation

### Business Context
An AI team deploys prompt changes weekly. They need automated quality assurance that catches regressions before production. Build an LLM-powered CI/CD pipeline that evaluates prompt quality and blocks bad deployments.

### Architecture
```
┌──────────┐    ┌───────────┐    ┌──────────────┐    ┌──────────────┐
│  Git     │───▶│  CI/CD    │───▶│  Evaluation  │───▶│  Deploy /    │
│  Push    │    │  Pipeline │    │  Engine      │    │  Block       │
│  (prompt │    │  (GitHub  │    │  (run evals) │    │  (pass/fail) │
│  changes)│    │  Actions) │    │              │    │              │
└──────────┘    └───────────┘    └──────────────┘    └──────────────┘
```

### Implementation Steps

```python
# Step 1: Prompt configuration management
"""
Structure prompts as versioned config:
prompts/
  customer_support/
    v1.0.yaml  # system prompt, few-shot examples, parameters
    v1.1.yaml  # updated version
    eval_suite.yaml  # test cases for this prompt
    baseline_metrics.json  # metrics to beat
"""

# Step 2: Evaluation engine
"""
Build a flexible evaluation engine:
- Input: prompt config + eval suite
- Run each test case against the prompt
- Score using multiple criteria:
  - Correctness (exact match, fuzzy match, or LLM-as-judge)
  - Formatting compliance (output format correct?)
  - Safety (no harmful content, no PII leakage)
  - Latency (within budget?)
  - Cost (within token budget?)
- Compare against baseline metrics
- Generate evaluation report
"""

# Step 3: CI/CD integration
"""
GitHub Actions workflow:
1. Detect changed prompt files
2. Load corresponding eval suites
3. Run evaluations (parallel across test cases)
4. Compare vs. baseline:
   - All metrics >= baseline → PASS → auto-deploy
   - Any critical metric < baseline → FAIL → block + alert
   - Marginal regression → WARN → require manual approval
5. Post results as PR comment
6. Store metrics history for trending
"""

# Step 4: Regression detection
"""
Track metrics over time:
- Plot quality metrics across versions
- Detect: gradual degradation (metrics slowly declining)
- Alert: sudden drops (new prompt broke something)
- Monthly reports: quality trends, cost trends
"""

# Step 5: Automated rollback
"""
If production monitoring detects quality drop:
- Automatic rollback to previous prompt version
- Alert team with diagnostics
- Log incident for post-mortem
"""
```

### Evaluation Report Example
```
╔══════════════════════════════════════════════════════╗
║           PROMPT EVALUATION REPORT                  ║
║   Prompt: customer_support/v1.1                     ║
║   Date: 2025-01-15                                  ║
╠══════════════════════════════════════════════════════╣
║                                                      ║
║  OVERALL: ✅ PASS (deploy approved)                  ║
║                                                      ║
║  Metric              v1.0     v1.1    Change         ║
║  ──────              ────     ────    ──────         ║
║  Correctness         82.0%    87.0%   +5.0% ✅       ║
║  Format compliance   95.0%    96.0%   +1.0% ✅       ║
║  Safety compliance   100.0%   100.0%   0.0% ✅       ║
║  Avg latency (ms)    1,200    1,150   -4.2% ✅       ║
║  Avg cost/query      $0.018   $0.016  -11.1% ✅      ║
║                                                      ║
║  Test Cases: 50/50 passed                            ║
║  Regression Tests: 0 failures                        ║
║                                                      ║
╚══════════════════════════════════════════════════════╝
```

---

## Case Study Selection Guide

| Case Study | Best For Learning | Prerequisite Days |
|------------|-------------------|-------------------|
| 1. Knowledge Base RAG | RAG mastery, production patterns | Days 1-7 |
| 2. Customer Support Agent | Agent design, guardrails | Days 1-12 |
| 3. Code Review System | Multi-agent, evaluation | Days 10-12 |
| 4. Data Analysis Copilot | Tool use, text-to-SQL | Days 6, 10 |
| 5. Document Intelligence | Multi-modal, extraction | Days 3-5, 13 |
| 6. CI/CD Quality Gate | LLMOps, production | Days 16-17 |

**Recommended path:** Start with Case Study 1 (foundational), then Case Study 2 or 3 (agents), then Case Study 6 (production).

---

*Next: See `03_ASSESSMENT.md` for the self-assessment quiz.*
