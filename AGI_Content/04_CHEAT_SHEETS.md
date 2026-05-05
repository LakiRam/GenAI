# AI Engineering Quick Reference — Cheat Sheets

---

## 1. LLM API Patterns Cheat Sheet

### OpenAI API — Core Pattern
```python
from openai import OpenAI

client = OpenAI()  # uses OPENAI_API_KEY env var

# Basic completion
response = client.chat.completions.create(
    model="gpt-4o",
    messages=[
        {"role": "system", "content": "You are a helpful assistant."},
        {"role": "user", "content": "Explain transformers in 3 sentences."}
    ],
    temperature=0.7,
    max_tokens=500
)
print(response.choices[0].message.content)

# Streaming
stream = client.chat.completions.create(
    model="gpt-4o",
    messages=[{"role": "user", "content": "Write a poem"}],
    stream=True
)
for chunk in stream:
    if chunk.choices[0].delta.content:
        print(chunk.choices[0].delta.content, end="")

# Structured output (JSON mode)
response = client.chat.completions.create(
    model="gpt-4o",
    messages=[{"role": "user", "content": "Extract name and age from: 'John is 30'"}],
    response_format={"type": "json_object"}
)

# Function calling / Tool use
tools = [{
    "type": "function",
    "function": {
        "name": "get_weather",
        "description": "Get weather for a city",
        "parameters": {
            "type": "object",
            "properties": {
                "city": {"type": "string", "description": "City name"}
            },
            "required": ["city"]
        }
    }
}]
response = client.chat.completions.create(
    model="gpt-4o",
    messages=[{"role": "user", "content": "Weather in Seattle?"}],
    tools=tools,
    tool_choice="auto"
)

# Embeddings
embedding = client.embeddings.create(
    model="text-embedding-3-small",
    input="Hello world"
)
vector = embedding.data[0].embedding  # list of 1536 floats
```

### Token Counting
```python
import tiktoken

enc = tiktoken.encoding_for_model("gpt-4o")
tokens = enc.encode("Hello, how are you?")
print(f"Token count: {len(tokens)}")
print(f"Estimated cost: ${len(tokens) * 2.50 / 1_000_000:.6f}")  # input cost
```

---

## 2. RAG Architecture Cheat Sheet

### Minimal RAG in 50 Lines
```python
from langchain_openai import OpenAIEmbeddings, ChatOpenAI
from langchain_community.vectorstores import Chroma
from langchain.text_splitter import RecursiveCharacterTextSplitter
from langchain_community.document_loaders import PyPDFLoader

# 1. Load → 2. Chunk → 3. Embed → 4. Store
loader = PyPDFLoader("document.pdf")
docs = loader.load()

splitter = RecursiveCharacterTextSplitter(chunk_size=512, chunk_overlap=50)
chunks = splitter.split_documents(docs)

vectorstore = Chroma.from_documents(chunks, OpenAIEmbeddings())

# 5. Retrieve → 6. Generate
retriever = vectorstore.as_retriever(search_kwargs={"k": 5})
llm = ChatOpenAI(model="gpt-4o")

query = "What is the main finding?"
relevant_docs = retriever.invoke(query)
context = "\n\n".join([doc.page_content for doc in relevant_docs])

response = llm.invoke(
    f"Based on this context:\n{context}\n\nAnswer: {query}"
)
print(response.content)
```

### Chunking Strategy Decision Tree
```
Is the document structured (HTML, Markdown, code)?
├── YES → Use document-structure-aware chunking
│         (split at headers, functions, sections)
└── NO
    ├── Is semantic coherence critical?
    │   ├── YES → Semantic chunking (group by topic similarity)
    │   └── NO → Recursive text splitting
    │           chunk_size: 512 (detailed retrieval) or 1024 (broader context)
    │           overlap: 10-15% of chunk_size
    └── Are there tables/code blocks?
        └── Keep tables intact, don't split mid-table
```

### Retrieval Strategy Selection
| Scenario | Strategy |
|----------|----------|
| Keyword-heavy queries (error codes, names) | BM25 (sparse) |
| Semantic/conceptual queries | Dense (vector) |
| Mixed queries | Hybrid (dense + BM25 + RRF) |
| High precision needed | Hybrid + reranker |
| Multi-hop questions | Multi-query + fusion |

---

## 3. Agent Patterns Cheat Sheet

### ReAct Agent — From Scratch
```python
import json
from openai import OpenAI

client = OpenAI()
MAX_STEPS = 10

def run_agent(user_query: str, tools: dict, max_steps=MAX_STEPS):
    messages = [
        {"role": "system", "content": """You are a helpful agent.
        For each step, respond with JSON:
        {"thought": "...", "action": "tool_name", "action_input": {...}}
        OR when done:
        {"thought": "...", "answer": "final answer"}"""},
        {"role": "user", "content": user_query}
    ]
    
    for step in range(max_steps):
        response = client.chat.completions.create(
            model="gpt-4o", messages=messages,
            response_format={"type": "json_object"}
        )
        result = json.loads(response.choices[0].message.content)
        
        if "answer" in result:
            return result["answer"]
        
        # Execute tool
        tool_name = result["action"]
        tool_input = result["action_input"]
        observation = tools[tool_name](**tool_input)
        
        messages.append({"role": "assistant", "content": json.dumps(result)})
        messages.append({"role": "user", "content": f"Observation: {observation}"})
    
    return "Max steps reached. Could not complete the task."
```

### LangGraph Agent Pattern
```python
from langgraph.graph import StateGraph, END
from typing import TypedDict, Annotated

class AgentState(TypedDict):
    messages: list
    next_action: str

def should_continue(state: AgentState) -> str:
    last_message = state["messages"][-1]
    if last_message.get("tool_calls"):
        return "tools"
    return END

graph = StateGraph(AgentState)
graph.add_node("agent", call_model)
graph.add_node("tools", execute_tools)
graph.add_edge("tools", "agent")
graph.add_conditional_edges("agent", should_continue)
graph.set_entry_point("agent")
app = graph.compile()
```

---

## 4. Fine-Tuning Cheat Sheet

### Decision: Prompt vs. RAG vs. Fine-Tune
```
Need external/updated knowledge? → RAG
Need consistent format/style?   → Fine-tune (SFT)
Need better alignment?          → Fine-tune (DPO/RLHF)
Need specific behavior?         → Few-shot prompting first, then fine-tune
Cost-sensitive at scale?         → Fine-tune small model (distillation)
Need domain adaptation?         → Continued pre-training + SFT
Have < 50 examples?             → Prompting (don't fine-tune)
Have 50-500 examples?           → Consider fine-tuning, may need synthetic data
Have 500+ examples?             → Fine-tuning likely to help
```

### OpenAI Fine-Tuning
```python
# Data format (JSONL)
{"messages": [
    {"role": "system", "content": "You are a support agent."},
    {"role": "user", "content": "How do I reset my password?"},
    {"role": "assistant", "content": "To reset your password..."}
]}

# Upload and train
from openai import OpenAI
client = OpenAI()

file = client.files.create(file=open("training.jsonl", "rb"), purpose="fine-tune")
job = client.fine_tuning.jobs.create(training_file=file.id, model="gpt-4o-mini-2024-07-18")

# Use the fine-tuned model
response = client.chat.completions.create(
    model="ft:gpt-4o-mini-2024-07-18:org:custom:id",
    messages=[{"role": "user", "content": "How do I reset my password?"}]
)
```

### LoRA Fine-Tuning (Open Source)
```python
from unsloth import FastLanguageModel
from trl import SFTTrainer

model, tokenizer = FastLanguageModel.from_pretrained(
    model_name="unsloth/llama-3.1-8b-bnb-4bit",
    max_seq_length=2048, load_in_4bit=True
)

model = FastLanguageModel.get_peft_model(
    model, r=16, target_modules=["q_proj", "k_proj", "v_proj", "o_proj"],
    lora_alpha=16, lora_dropout=0
)

trainer = SFTTrainer(
    model=model, tokenizer=tokenizer,
    train_dataset=dataset,
    max_seq_length=2048,
    args=TrainingArguments(
        per_device_train_batch_size=2,
        gradient_accumulation_steps=4,
        num_train_epochs=3,
        learning_rate=2e-4,
        output_dir="outputs"
    )
)
trainer.train()
```

---

## 5. Evaluation Cheat Sheet

### What to Evaluate
| What | Metric | How |
|------|--------|-----|
| Answer correctness | Accuracy, F1 | Compare to golden answers |
| RAG retrieval quality | Context precision, recall | Check if right chunks retrieved |
| Faithfulness | Groundedness score | Is answer supported by context? |
| Relevance | Answer relevance | Does answer address the question? |
| Harmfulness | Safety score | Red-team testing |
| Format compliance | Schema match rate | Validate output structure |
| Latency | p50, p95, p99 | Time from request to last token |
| Cost | $/query, $/day | Token counting |

### RAGAS Evaluation
```python
from ragas import evaluate
from ragas.metrics import faithfulness, answer_relevancy, context_precision

result = evaluate(
    dataset=eval_dataset,  # HF Dataset with question, answer, contexts, ground_truth
    metrics=[faithfulness, answer_relevancy, context_precision]
)
print(result)  # {'faithfulness': 0.85, 'answer_relevancy': 0.92, ...}
```

### LLM-as-Judge Pattern
```python
JUDGE_PROMPT = """Rate the following answer on a scale of 1-5:
Question: {question}
Answer: {answer}
Reference: {reference}

Criteria:
- Accuracy: Does the answer match the reference?
- Completeness: Does it cover all key points?
- Clarity: Is it easy to understand?

Output JSON: {"accuracy": int, "completeness": int, "clarity": int, "reasoning": str}"""
```

---

## 6. Production Checklist

### Pre-Launch Checklist
```
[ ] Evaluation suite passing (>= baseline metrics)
[ ] Load tested (target QPS × 2)
[ ] Prompt injection tested (50+ attack vectors)
[ ] PII detection/redaction in place
[ ] Rate limiting configured
[ ] Cost alerts set (daily + monthly budget)
[ ] Error handling (API failures, timeouts, malformed responses)
[ ] Monitoring dashboards live (latency, cost, errors, quality)
[ ] Logging: every request/response logged (redacted) for debugging
[ ] Fallback: secondary model configured if primary fails
[ ] Rollback plan documented and tested
[ ] On-call runbook written
[ ] Data retention policy defined
[ ] User feedback mechanism in place
```

### Cost Estimation Formula
```
Monthly cost = (queries/day × 30) × (avg_input_tokens × input_price + avg_output_tokens × output_price)

Example (GPT-4o):
100K queries/month × (500 input tokens × $2.50/1M + 300 output tokens × $10.00/1M)
= 100K × ($0.00125 + $0.003)
= 100K × $0.00425
= $425/month
```

---

## 7. Key Terminology Glossary

| Term | Definition |
|------|-----------|
| **Transformer** | Neural network architecture using self-attention; basis for all modern LLMs |
| **Token** | Unit of text (roughly ¾ of a word in English) |
| **Context window** | Maximum tokens a model can process in one call (input + output) |
| **Temperature** | Controls randomness; 0 = deterministic, >1 = creative/random |
| **Top-p (nucleus sampling)** | Only sample from tokens whose cumulative probability adds to p |
| **Embedding** | Dense vector representation of text; captures semantic meaning |
| **RAG** | Retrieval-Augmented Generation; retrieve relevant docs, then generate answer |
| **Fine-tuning** | Further training a pre-trained model on task-specific data |
| **LoRA** | Low-Rank Adaptation; efficient fine-tuning by training small matrices |
| **QLoRA** | Quantized LoRA; LoRA on a 4-bit quantized model (less GPU memory) |
| **RLHF** | Reinforcement Learning from Human Feedback; aligns model to human preferences |
| **DPO** | Direct Preference Optimization; simpler alternative to RLHF |
| **SFT** | Supervised Fine-Tuning; standard fine-tuning on instruction-response pairs |
| **Agent** | LLM + tools + reasoning loop; can take actions, not just generate text |
| **ReAct** | Reasoning + Acting pattern; think → act → observe → repeat |
| **MCP** | Model Context Protocol; standard for connecting LLMs to tools/data |
| **Hallucination** | Model generates plausible-sounding but factually incorrect information |
| **Prompt injection** | Attack where user input overrides the system prompt |
| **Guardrails** | Input/output validation to ensure AI behavior stays within bounds |
| **Vector database** | Database optimized for storing/searching embeddings |
| **Reranker** | Model that re-scores retrieved documents for better relevance ordering |
| **HNSW** | Hierarchical Navigable Small World; algorithm for approximate nearest neighbor search |
| **Quantization** | Reducing model precision (32-bit → 4-bit) to reduce memory and increase speed |
| **vLLM** | High-throughput LLM serving engine with paged attention |
| **KV-cache** | Cache of key-value pairs in attention; speeds up autoregressive generation |
| **Speculative decoding** | Use small model to draft, large model to verify; faster inference |
| **MoE** | Mixture of Experts; architecture where only some parameters activate per token |
| **GraphRAG** | RAG enhanced with knowledge graphs for multi-hop reasoning |
| **Data flywheel** | Production data → model improvement → better product → more data |
| **LLMOps** | Operational practices for LLM applications (monitoring, eval, deployment) |

---

*Keep this file handy as a quick reference during your 30-day journey.*
