# AI Engineering Manager — Self-Assessment & Knowledge Check

> Take this assessment on Day 30 to identify your strengths and remaining gaps.
> Score yourself honestly. Use gaps to plan your next 90 days of learning.

---

## Section 1: LLM Fundamentals (20 questions)

### Conceptual Understanding

**Q1.** Explain the difference between a decoder-only model (GPT) and an encoder-decoder model (T5). When would you choose each?

**Q2.** What is the attention mechanism? Why is "self-attention" the breakthrough that made transformers work? Explain in terms a smart junior engineer would understand.

**Q3.** What happens when you increase the temperature parameter from 0 to 1.5? When would you use temperature 0 vs. 0.7 vs. 1.5?

**Q4.** A context window is 128K tokens. You have a 200-page document. What are your options? Compare the tradeoffs of: truncation, RAG, summarize-then-query, and map-reduce.

**Q5.** What is RLHF and why did it matter for making ChatGPT useful? What is DPO and why is it replacing RLHF?

### Practical Skills

**Q6.** You're getting inconsistent JSON output from GPT-4o. What are 3 techniques to ensure reliable structured output?

**Q7.** Your prompt works great on GPT-4o but poorly on Claude. What's the likely cause and how do you debug it?

**Q8.** A user reports that the LLM is "making stuff up." Walk through your debugging process to identify whether it's a hallucination, a retrieval problem, or a prompt issue.

**Q9.** You need to process 10,000 documents through an LLM API. Design the pipeline considering: rate limits, cost, error handling, and idempotency.

**Q10.** Write a system prompt for a financial advisor chatbot that: stays on topic, never gives specific investment advice, always adds disclaimers, and handles prompt injection attempts.

---

## Section 2: RAG Architecture (15 questions)

**Q11.** Draw the architecture of a production RAG system. Label every component and explain why each exists.

**Q12.** You built a RAG system and users say "it doesn't find the right information." Walk through your debugging process (at least 5 things to check).

**Q13.** What is hybrid search? Why does combining dense + sparse retrieval outperform either alone? How do you implement Reciprocal Rank Fusion?

**Q14.** Your RAG system has 100K documents. Users ask questions that require synthesizing information across 5+ documents. How do you handle this? (Current chunk-based retrieval misses cross-document connections.)

**Q15.** Explain the difference between naive RAG, advanced RAG, and modular RAG. Give a concrete example of when you'd need each.

**Q16.** Your embedding model was trained on English text. You now need to support Japanese documents. What are your options?

**Q17.** A user asks: "What changed in our pricing policy between January and March?" — this requires temporal reasoning across document versions. How do you architect this?

**Q18.** Compare these chunking strategies and when to use each: fixed-size, recursive, semantic, document-structure-aware.

**Q19.** What is a reranker? When does reranking improve results and when is it unnecessary overhead?

**Q20.** Your RAG system costs $0.15 per query. The business wants it under $0.03. What are 5 cost reduction strategies that preserve quality?

---

## Section 3: Agents & Multi-Agent Systems (15 questions)

**Q21.** What is the ReAct pattern? Write pseudocode for a ReAct agent loop. What stops it from running forever?

**Q22.** An agent has access to 20 tools. It consistently picks the wrong tool for database queries. How do you fix this? (At least 3 approaches.)

**Q23.** You're building a multi-agent system for financial analysis. Agent A does data retrieval, Agent B does analysis, Agent C writes reports. How do they share state? What coordination pattern do you use?

**Q24.** Your agent system costs $2.50 per complex task (20+ LLM calls). The budget is $0.50. Propose an architecture that reduces cost without sacrificing quality.

**Q25.** An agent is stuck in a loop — it keeps trying the same failed action. What safeguards should be built in?

**Q26.** When should you use a single powerful agent vs. multiple specialized agents? Give criteria for the decision.

**Q27.** What is MCP (Model Context Protocol)? How does it standardize tool integration? What are its limitations?

**Q28.** Your multi-agent system needs to handle partial failures (Agent B fails mid-workflow). Design a recovery strategy.

**Q29.** How do you test an agent system? Unit testing is insufficient because behavior is non-deterministic. Propose a testing strategy.

**Q30.** What is agentic RAG? How does it differ from traditional RAG? When is the extra complexity worth it?

---

## Section 4: Fine-Tuning & Training (10 questions)

**Q31.** You have a task where GPT-4o gets 85% accuracy with a good prompt. Should you fine-tune? What factors drive this decision?

**Q32.** Explain LoRA at a conceptual level. Why does it use 1000x fewer parameters than full fine-tuning yet achieve comparable results?

**Q33.** You have 50 training examples. Is that enough to fine-tune? What's the minimum viable dataset for different types of tasks?

**Q34.** You fine-tuned a model and it's great at the new task but worse at general tasks. What happened? How do you prevent this?

**Q35.** Design a synthetic data generation pipeline for fine-tuning a customer support model. How do you ensure quality and diversity?

**Q36.** What is DPO and how does it differ from SFT? When would you use DPO on top of SFT?

**Q37.** You need to adapt a model to a specialized domain (medical, legal, financial). Compare: continued pre-training vs. fine-tuning vs. RAG. When to use each?

**Q38.** Your fine-tuned model performs well on eval but poorly in production. What are the likely causes?

**Q39.** How do you evaluate a fine-tuned model? What metrics matter beyond loss?

**Q40.** What is knowledge distillation? Design a pipeline to distill GPT-4o into Phi-3 for a narrow task.

---

## Section 5: Production & Operations (15 questions)

**Q41.** Your LLM application has p99 latency of 8 seconds. Users complain. What are 5 techniques to reduce latency?

**Q42.** Design a monitoring dashboard for a production LLM application. What are the top 10 metrics you'd track?

**Q43.** Your LLM costs jumped 3x this week. How do you investigate? What preventive measures should be in place?

**Q44.** A user found a prompt injection that makes your chatbot ignore its system prompt and reveal internal instructions. How do you:
   a) Fix this specific attack?
   b) Prevent future attacks systematically?

**Q45.** You need to deploy an LLM application that handles PII (medical records). What architecture ensures compliance with HIPAA/GDPR?

**Q46.** Design an A/B testing framework for comparing two different prompts in production. How do you measure "better"?

**Q47.** Your RAG system's quality has been slowly declining over 3 months but no code changed. What's happening? How do you detect and fix this?

**Q48.** A model provider raises prices 50%. Design a vendor migration plan that minimizes risk and downtime.

**Q49.** Build vs. buy: when should you self-host an open model vs. use an API? Create a decision matrix with at least 5 factors.

**Q50.** Explain the data flywheel concept. How do production interactions improve your model over time? Design this pipeline.

---

## Section 6: Strategy & Leadership (10 questions)

**Q51.** Your CEO asks: "Should we build our own LLM?" What is your answer and how do you structure the argument?

**Q52.** You're hiring for your AI team. You have budget for 3 people. What roles do you hire and why? What skills are must-haves vs. nice-to-haves?

**Q53.** A business unit wants an AI chatbot "like ChatGPT but for our data." Estimate the project: timeline, team size, cost, risks. What questions do you ask before committing?

**Q54.** You need to present an AI strategy to the board. What are the 5 key slides? What metrics demonstrate ROI?

**Q55.** An AI project is 3 months in with no clear results. How do you assess whether to continue, pivot, or kill it?

**Q56.** How do you measure the ROI of an AI engineer productivity tool (code assistant, documentation bot)?

**Q57.** Your team is split between "move fast and ship" and "we need more evals before launching." How do you resolve this?

**Q58.** A competitor just launched an AI feature similar to what you're building. What's your response?

**Q59.** How do you keep your AI team's skills current when the field changes every 3 months?

**Q60.** Explain the EU AI Act's risk classification to a non-technical executive. How does it affect your product roadmap?

---

## Scoring Guide

### Per Question
| Score | Meaning |
|-------|---------|
| 0 | Can't answer / never heard of this |
| 1 | Vaguely aware, can't explain clearly |
| 2 | Can explain conceptually but haven't done it |
| 3 | Have done it, can explain with confidence |
| 4 | Expert — can teach others and handle edge cases |

### Section Scoring
| Section | Max Score | Proficient (Manager) | Expert |
|---------|-----------|---------------------|--------|
| 1. LLM Fundamentals | 40 | 30+ | 36+ |
| 2. RAG Architecture | 40 | 30+ | 36+ |
| 3. Agents & Multi-Agent | 40 | 28+ | 36+ |
| 4. Fine-Tuning & Training | 40 | 28+ | 36+ |
| 5. Production & Operations | 40 | 32+ | 36+ |
| 6. Strategy & Leadership | 40 | 32+ | 36+ |
| **TOTAL** | **240** | **180+** | **216+** |

### Gap Analysis
After scoring, identify your bottom 2 sections and create a focused 2-week drill-down plan for those areas.

---

## Next Steps After 30 Days

### Days 31-60: Specialization
Pick 2-3 areas to go deep:
- [ ] Advanced agent architectures (research and implement 3 novel patterns)
- [ ] Production scale (deploy a system handling 1000+ req/sec)
- [ ] Fine-tuning mastery (train 5+ models, build automated training pipeline)
- [ ] Multi-modal deep dive (vision + audio + text end-to-end systems)
- [ ] MLOps/LLMOps (build full CI/CD with evaluation-gated deployments)

### Days 61-90: Leadership & Community
- [ ] Write 3 technical blog posts about your learnings
- [ ] Give a tech talk to your team
- [ ] Contribute to an open-source AI project
- [ ] Build relationships with AI leads at other companies
- [ ] Design your team's AI engineering standards document
- [ ] Create your company's AI architecture decision records (ADRs)

### Ongoing: Stay Current
- Subscribe to the newsletters/blogs listed in the training plan
- Allocate 3 hours/week to experimenting with new models/tools
- Attend 1 AI conference or major online event per quarter
- Maintain your portfolio with new projects

---

*This completes the 30-Day AI/ML Mastery program. Good luck!*
