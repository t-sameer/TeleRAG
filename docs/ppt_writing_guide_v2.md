# 📊 Blueprint PPT Writing Guide — AX Hackathon Phase 1

## General PPT Principles

- **Judges scan, they don't read.** Each slide should have a clear headline that states the takeaway, not just a topic.
  - ❌ "Architecture Overview"
  - ✅ "4-Layer Retrieval Stack With Self-Healing Corrective RAG"
- **Visual > Text.** Use diagrams, tables, and flow charts. Minimize paragraphs.
- **Numbers > Adjectives.** "12.5 GB / 16 GB VRAM budget" beats "efficient resource usage."
- **Every slide answers: "Why should we qualify?"**

---

## Core Blueprint Slides

### Slide 1: Title + Team
**What to include:**
- Project name: **TeleRAG-4** (or your chosen name)
- Tagline: One sentence (e.g., "A Multimodal RAG System for Telecom RAN with Chain-of-Thought Root Cause Analysis")
- Team members + roles (e.g., Person 1: RAG Pipeline + Evaluation, Person 2: Training + Data Engineering)
- Problem statement number: #10

### Slide 2: Problem Understanding
**What judges want:** Proof you actually understand the domain, not just parroting the problem statement.
- Paraphrase the problem in YOUR words
- The 3 core challenges: root cause analysis, anomaly detection, network optimization
- Why current tools fail (manual SME intervention, scalability issues)
- Your 5 target KPIs in a clean table

### Slide 3: Solution Architecture
**The most important slide.** This is your architecture diagram.
- Use the Mermaid flowchart from our plan, converted to a polished visual
- Label every component: Query Router → 4-Layer Retrieval → CRAG Loop → Gemma 4 Generator → Self-RAG → Output
- Call out the 3 modes: 3GPP Q&A, RCA, Anomaly Detection
- **Tip:** Color-code by function (blue = retrieval, green = generation, red = safety/validation)

### Slide 4: Technical Innovation
**What differentiates TeleRAG-4 from a basic RAG chatbot:**
- Contextual Retrieval chunking for 3GPP specs (+35-49% retrieval precision)
- Semantic Query Router (zero VRAM — reuses generator)
- CRAG self-healing loop with max 2 retries
- Query decomposition for multi-hop telecom questions
- Multimodal table processing via Gemma 4 vision
- 5-step RCA Chain-of-Thought (NOT just retrieval — structured reasoning)
- Citation enforcement + Self-RAG for faithfulness

**Format:** 3-4 bullet callout boxes with icons, not a wall of text.

### Slide 5: Data Strategy (Updated — v2)
- **6 data sources** in a visual pipeline diagram:
  - 3GPP specs (10 must-have) → contextual chunking + table image crops
  - ORAN-Bench-13K (13,952 MCQs from 116 O-RAN specs) → O-RAN text knowledge
  - Colosseum COMMAG (real O-RAN KPIs) → anomaly injection → prose summaries
  - TeleQnA (10K MCQs) → 80/20 stratified split (8K train, 2K eval)
  - Tele-Data → CPT corpus
  - Hand-crafted Incident KB (20-30 RAN incidents) → RCA historical matching
- Index architecture: **4 ChromaDB collections (~47K total chunks)**
- **Key talking point:** "O-RAN domain coverage via ORAN-Bench-13K — 13,952 curated questions from 116 O-RAN specifications — combined with Colosseum real-world RAN telemetry. Dual-source approach: text knowledge + operational data."

### Slide 6: Training Strategy
- 2-stage LoRA pipeline: CPT (Tele-Data) → SFT (TeleQnA + Tele-Eval)
- Visual: Base model → CPT adapter → merged → SFT adapter → final
- Hyperparameters: r=16, alpha=32, 4-bit QLoRA, Unsloth
- VRAM budget table (14.5 GB / 16 GB)

### Slide 7: KPI Attack Plan
- Table with 5 KPIs, targets, projected values, and which techniques drive each
- Show cumulative technique stacking (like our MRR buildup table)
- **This slide proves feasibility** — it shows you've thought about HOW you'll hit each target

### Slide 8: Technology Stack + Resources
- Clean table: LLM (Gemma 4 E4B), Embeddings (BGE-base), Reranker (BGE-reranker), VectorDB (ChromaDB), UI (Streamlit)
- VRAM budget chart (12.5 GB / 16 GB with component breakdown)
- Compute plan: 2 people × Kaggle + Colab free tier = 420+ GPU hrs

### Slide 9: Security & Privacy (EXPANDED — Scored Criterion)

**Why this slide matters:** Security & Privacy is an **explicitly scored evaluation criterion**. Most teams will forget it. This is free differentiation.

**Slide structure — 4 quadrants:**

| Quadrant | Feature | What to Show |
|---|---|---|
| **Input Layer** | PII Sanitization | Regex filters for IMSI (15-digit), IMEI, MSISDN, IPv4/IPv6, Subscriber IDs. Show code snippet |
| **Output Layer** | Guardrails | Post-generation redaction of any leaked sensitive patterns. Show before/after example |
| **Prompt Layer** | Injection Protection | System prompt hardening + input validation. Show rejected malicious query example |
| **Architecture** | Data Isolation | Diagram showing: all models run locally, ChromaDB local-only, no data leaves notebook |

**Demo script for this slide:**
```
Judge sees: User types "My subscriber SUB_123456 at IP 192.168.1.1 has issues"
System shows: Query sanitized → "My subscriber [REDACTED_Subscriber_ID] at [REDACTED_IPv4] has issues"
System proceeds: Processes sanitized query through RAG pipeline
Output shows: Clean answer with no PII leakage + sanitization notice
```

**Key talking points:**
- All data stays local — ChromaDB persistent storage, Gemma 4 runs on-device
- All training/inference datasets are public/open-source — no proprietary operator data
- System designed for air-gapped deployment capability
- Compliance with telecom data privacy principles (data minimization, purpose limitation)

### Slide 10: Timeline & Milestones
- Gantt chart or timeline visual: Notebook 1 (Weeks 1-2) → Notebook 2 (Weeks 3-4) → Notebook 3 (Weeks 4-6) → Eval (Weeks 7-8)
- Key milestones: Data indexed, Training complete, Pipeline working, KPIs measured

---

## Three Additional Required Slides

### Slide 11: Open Models Used (Trained or Fine-Tuned)

**What they're asking:** List every open-source model in your pipeline and what you're doing with it.

| Model | Role | What We Do With It | Why This Model |
|---|---|---|---|
| **Gemma 4 E4B-it** (4.5B params) | Generator + Vision | 2-stage LoRA fine-tuning (CPT + SFT). Multimodal: processes table image crops via native vision encoder. | Native vision, 128K context, fits T4 at 4-bit. Most capable multimodal model in the <5B class |
| **BAAI/bge-base-en-v1.5** (110M) | Dense retriever | Used as-is (no fine-tuning). Embeds 32K chunks into 768-dim vectors | Best quality/size ratio. Proven on MTEB benchmark |
| **BAAI/bge-reranker-base** (278M) | Cross-encoder reranker | Used as-is. Scores query-document pairs for reranking | Token-level relevance scoring at 1.5 GB VRAM |
| **Gemma 4 27B** (via AI Studio) | Evaluation judge | Used via API for faithfulness assessment. Not deployed locally | Independent judge needed — can't self-evaluate |

**Key points to emphasize:**
- Gemma 4 E4B is **trained** (not just prompted) — this shows deeper ML work
- Two-stage training pipeline is a recognized SOTA approach (Tele-LLMs methodology)
- All models are truly open (Apache 2.0 / open weights)
- Mention Unsloth for training efficiency (2x speedup, 60% less VRAM)

### Slide 12: AI/Agentic AI Tools & Best Practices

**What they're asking:** How did your team USE AI tools during development, and what agentic patterns exist in your solution?

**Two sections on this slide:**

#### Section A: Agentic AI in the Solution
- **Semantic Query Router** — autonomous classification of query intent
- **CRAG Self-Healing Loop** — agent detects low-relevance retrieval and autonomously re-retrieves with query expansion
- **Query Decomposer** — agent autonomously breaks complex questions into sub-queries
- **Self-RAG Reflection** — agent self-checks faithfulness before returning answer
- **RCA Reasoning Agent** — 5-step autonomous chain: detect → extract → hypothesize → retrieve evidence → conclude

**Frame it as:** "Our system uses 5 agentic patterns where the AI autonomously makes decisions without human intervention."

#### Section B: AI Tools Used by the Team
Be honest about how you used AI during development:
- LLM-assisted code generation (highlight which tools: Copilot, Claude, Gemini, etc.)
- AI-assisted research and literature review
- AI-generated architecture diagrams (if applicable)
- Prompt engineering iteration workflow

**Best practices discovered:**
- Contextual chunking headers dramatically improve telecom retrieval (empirical finding during development)
- BM25 is essential alongside dense search for acronym-heavy domains
- CRAG loop catches ~20% of queries that would otherwise fail
- 4-bit quantization on T4 requires strict FP16 (never BF16)

**Creative uses:**
- Using Gemma 4's vision encoder to read 3GPP tables (traditionally lost in PDF parsing)
- Anomaly-to-prose conversion: bridging time-series data and text-based RAG
- Using a larger model (Gemma 4 27B via API) to evaluate a smaller model (Gemma 4 E4B local)

### Slide 13: Supporting Materials & Artifacts

**What they're asking:** Evidence that you've thought deeply and have the artifacts to back it up.

**What to include:**

| Artifact | Description | Status |
|---|---|---|
| Architecture deep-dive | Full TeleRAG-4 roadmap v3 (condensed) | ✅ Complete |
| VRAM budget spreadsheet | Component-by-component: 12.5 GB / 16 GB | ✅ Complete |
| Dataset inventory | 6 sources, verified availability, download commands | ✅ Complete (v2) |
| Security architecture | Input/output sanitization + data isolation diagram | ✅ Complete |
| Evaluation methodology | 5 KPIs, ground truth approach, AI Studio judge setup | ✅ Complete |
| Risk register | 10 risks with mitigations (2 resolved) | ✅ Complete |
| Code repository | Planned 3-notebook structure | 📋 Planned |

**References:** TeleQnA, Tele-LLMs, ORAN-Bench-13K (arXiv:2407.06245), Contextual Retrieval (Anthropic), CRAG, Colosseum COMMAG

---

## Multi-Day Evaluation Planning

You asked: *"How do we plan this for multiple days? I'm assuming we hit rate limits or GPU runs out."*

### The Key Insight: Decouple Generation from Judging

Evaluation has two independent steps that use **different resources:**

| Step | Resource Needed | Can Run Offline? |
|---|---|---|
| **Step 1: Generate answers** (run RAG pipeline on 10K questions) | T4 GPU (Kaggle) | No — needs GPU loaded |
| **Step 2: Judge faithfulness** (score each answer against context) | AI Studio API (cloud) | Yes — just HTTP calls |

**These run on DIFFERENT infrastructure.** You're never blocked on both at once.

### Day-by-Day Plan

```
Day 1 (GPU): Generate answers for questions 1-5,000
  - Load model on Kaggle T4
  - Run full RAG pipeline: retrieve + rerank + generate
  - Save: {question_id, query, retrieved_chunks, generated_answer}
  - Output: answers_batch_1.jsonl
  - GPU time: ~5-6 hours

Day 2 (GPU): Generate answers for questions 5,001-10,000
  - Same process
  - Output: answers_batch_2.jsonl  
  - GPU time: ~5-6 hours

Day 2 (API, parallel): Start judging batch 1 faithfulness
  - Upload answers_batch_1.jsonl to a scoring script
  - Script calls Gemma 4 27B via AI Studio for each answer
  - Rate: ~15 RPM × 2 accounts = 30 requests/min
  - Process: ~1,000-1,500 scores per day
  - No GPU needed — runs on CPU/laptop

Day 3 (API): Continue judging batch 1 + start batch 2
  - Faithfulness scoring continues
  
Day 4 (API): Finish all faithfulness scoring

Day 5 (CPU): Compute final metrics
  - MRR, Top-k, Recall: computed from retrieved_chunks (instant, no LLM)
  - Accuracy: compare generated answer to correct MCQ option (instant)
  - Faithfulness: aggregate scores from AI Studio judging
```

### Handling GPU Quota Exhaustion

```
If Kaggle GPU runs out mid-day:
  1. Save all generated answers so far to a .jsonl file
  2. Download the file to Google Drive
  3. Switch to Person 2's Kaggle account
  4. Upload the .jsonl, resume from where you left off
  5. Append new results to the same file

If AI Studio rate limit (RPD) hits:
  1. Switch to Person 2's AI Studio account
  2. Continue scoring with their API key
  3. Both accounts combined = ~2,000 RPD
  4. If still not enough, pause and resume next day
```

### Checkpointing Strategy

```python
# Save after every 100 questions (not 10K at once)
for i, question in enumerate(questions):
    result = run_rag_pipeline(question)
    results.append(result)
    
    if (i + 1) % 100 == 0:
        save_checkpoint(results, f"eval_checkpoint_{i+1}.jsonl")
        print(f"Saved checkpoint at {i+1} questions")
```

> [!TIP]
> **Golden rule:** Never run a long eval job without checkpointing. Kaggle WILL disconnect. Colab WILL timeout. Save every 100 questions, and your worst case is losing 100 questions of work (~10 minutes), not 5,000.
