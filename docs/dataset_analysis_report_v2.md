# 📊 TeleRAG-4 — Dataset Analysis Report v2

> **Date:** May 12, 2026 | **Phase 1 Deadline:** May 13 (TOMORROW) | **Phase 2 Build Window:** May 13 → Jun 22 (40 days)

---

## 1. Timeline Alignment Check

> [!IMPORTANT]
> The critical analysis from the previous conversation was written on **April 20-21, 2026** — 22 days ago. Let's check what's still accurate and what's changed.

### What the Critical Analysis Got Right ✅

| Item | Assessment Then | Status Now |
|---|---|---|
| Compute budget (420+ GPU hrs) | 🟢 Comfortable | ✅ Still valid — 6 weeks of Kaggle GPU ahead |
| TeleQnA availability | ✅ Clear path | ✅ Confirmed on HuggingFace |
| Simu5G infeasibility | ❌ Not feasible | ✅ Correct — still a 12-20 hr rabbit hole |
| 5G3E unreliability | ❌ Don't put on critical path | ✅ Correct — no response from authors likely |
| CRAG + Self-RAG impact projections | 🟢 Reasonable | ✅ Architecturally sound |
| Security gap identification | 🔴 Critical | ✅ Must be addressed in PPT tomorrow |

### What Needs Updating ⚠️

| Item | Old Assessment | Reality Check |
|---|---|---|
| O-RAN SC Nexus datasets | "Scattered, gated, 2-4 hrs" | **Worse: O-RAN Alliance specs are PDFs behind a portal, NOT structured datasets. No CSV/JSON telemetry data exists on o-ran.org** |
| Colosseum COMMAG as O-RAN data | "✅ DO THIS (10 min git clone)" | ✅ Still valid — but this is **network KPI data**, not O-RAN specification text |
| 3GPP download path | "Go to 3gpp.org, click..." | ⚠️ Specs are `.docx` in `.zip`. Need conversion pipeline. More effort than stated |
| ORAN-Bench-13K | Not mentioned in critical analysis | **NEW: 13,952 MCQs from 116 O-RAN specs — freely available on GitHub. This is your O-RAN text proxy** |

---

## 2. Complete Dataset Inventory

### 🟢 Tier 1: MUST HAVE (Core deliverables depend on these)

#### Dataset A: TeleQnA

| Attribute | Details |
|---|---|
| **Source** | [HuggingFace: netop/TeleQnA](https://huggingface.co/datasets/netop/TeleQnA) |
| **Format** | 10,000 MCQs in JSON. 5 categories: Lexicon, Research Overview, Research Publications, Standards Overview, Standards Specifications |
| **Size** | ~15 MB |
| **Access** | ✅ **Immediate** — Public on HuggingFace. Also available as password-protected ZIP (`teleqnadataset`) |
| **Effort to Use** | 🟢 **2-3 hours** — Parse JSON, stratified 80/20 split, format for SFT + eval |
| **Impact on Quality** | 🔴 **CRITICAL** — Primary evaluation benchmark AND SFT training data. Without this, you can't measure any KPI |
| **Download Command** | `from datasets import load_dataset; ds = load_dataset("netop/TeleQnA")` |
| **Usage Strategy** | **Split:** 8K train (SFT) / 2K eval (held-out). **Retrieval:** Embed question+explanation pairs into `telecom_knowledge` collection. **Eval:** Run all 5 KPIs against held-out 2K |

#### Dataset B: 3GPP Release 16/18 Specifications

| Attribute | Details |
|---|---|
| **Source** | [3GPP Spec Search](https://www.3gpp.org/specifications) or [ETSI Portal](https://www.etsi.org/standards) |
| **Format** | `.docx` inside `.zip` archives. **NOT PDFs** — requires conversion |
| **Size** | ~50-80 MB total for the 10 must-have specs |
| **Access** | ✅ **Free** — Public download, no account needed |
| **Effort to Use** | 🟡 **6-8 hours** — Download 10 specs + `.docx→.pdf` conversion + PyMuPDF parsing + contextual chunking + table image extraction |
| **Impact on Quality** | 🔴 **CRITICAL** — Primary knowledge base for 3GPP Spec Q&A (UC1). Directly drives MRR, Top-k, Recall |
| **Download Strategy** | Manual download from ETSI portal OR script via 3GPP FTP. Priority specs: TS 38.300, 38.331, 38.211-214, 38.401, 38.321-323 |
| **Conversion** | `soffice --headless --convert-to pdf *.docx` (LibreOffice CLI) — add to Notebook 1 |
| **Usage Strategy** | Parse → contextual chunk (prepend `[TS 38.331 | §5.3.3 | RRC Reconfiguration]`) → embed into `3gpp_specs` ChromaDB collection + BM25 index |

#### Dataset C: Colosseum O-RAN COMMAG

| Attribute | Details |
|---|---|
| **Source** | [GitHub: wineslab/colosseum-oran-commag-dataset](https://github.com/wineslab/colosseum-oran-commag-dataset) |
| **Format** | CSVs with per-UE, per-slice KPIs: `tx_brate downlink [Mbps]`, `ratio_granted_req`, buffer sizes, scheduling labels |
| **Size** | ~500 MB |
| **Access** | ✅ **Immediate** — `git clone` from GitHub |
| **Effort to Use** | 🟢 **5-6 hours** — Clone + anomaly injection script + prose template conversion + embed |
| **Impact on Quality** | 🔴 **HIGH** — Enables RCA (UC2) and Anomaly Detection (UC3) demos. Real O-RAN data satisfies hackathon requirement R3 |
| **Download Command** | `git clone https://github.com/wineslab/colosseum-oran-commag-dataset.git` |
| **Dataset Contents** | 4 BSs × 40 UEs × 3 slices (eMBB/MTC/URLLC). 3 scheduling policies (Round-robin, Waterfilling, Proportionally fair). 3 RF scenarios (Close/Medium/Far). Static + mobile UEs |
| **Usage Strategy** | (1) Clone raw CSVs → (2) Python script: inject synthetic anomalies (latency spikes, throughput drops, slice starvation) → (3) Convert anomalies to prose via `ANOMALY_TEMPLATE` → (4) Embed into `oran_anomalies` ChromaDB collection. Also create 20-30 hand-crafted incident reports for RCA historical matching |

---

### 🟡 Tier 2: SHOULD HAVE (Significant quality boost, moderate effort)

#### Dataset D: Tele-Data (Continued Pre-Training Corpus)

| Attribute | Details |
|---|---|
| **Source** | [HuggingFace: AliMaatouk/Tele-Data](https://huggingface.co/datasets/AliMaatouk/Tele-Data) |
| **Format** | Text corpus: 3GPP specs text + arXiv telecom papers + Wikipedia telecom articles + Common Crawl telecom subset |
| **Size** | ~2-3 GB (subset needed, full is larger) |
| **Access** | ✅ **Immediate** — Public on HuggingFace |
| **Effort to Use** | 🟢 **2-3 hours** — Download subset, format for causal LM, feed to CPT pipeline |
| **Impact on Quality** | 🟡 **MEDIUM-HIGH** — Teaches Gemma the "language of telecom." Expected +3-5% accuracy on domain terminology. CPT is Phase 1 of the 2-stage LoRA |
| **Download Command** | `from datasets import load_dataset; ds = load_dataset("AliMaatouk/Tele-Data")` |
| **Usage Strategy** | Filter to 3GPP + arXiv subsets (~500MB). Format as raw text for causal LM continued pre-training. Run 1-2 epochs on Kaggle T4 (~4-6 GPU hours) |

#### Dataset E: ORAN-Bench-13K ⭐ NEW — YOUR O-RAN TEXT PROXY

| Attribute | Details |
|---|---|
| **Source** | [GitHub: prnshv/ORAN-Bench-13K](https://github.com/prnshv/ORAN-Bench-13K) |
| **Format** | 13,952 MCQs in JSON. 3 difficulty levels: Easy, Medium, Difficult. Derived from 116 O-RAN spec documents |
| **Size** | ~10-15 MB |
| **Access** | ✅ **Immediate** — Public GitHub repo |
| **Effort to Use** | 🟢 **3-4 hours** — Parse JSONs, extract Q&A pairs + explanations, embed for retrieval |
| **Impact on Quality** | 🟡 **HIGH** — Provides O-RAN domain knowledge as text (questions + explanations covering O-RAN architecture, interfaces, protocols). **This replaces the need to manually process O-RAN Alliance PDFs** |
| **Download Command** | `git clone https://github.com/prnshv/ORAN-Bench-13K.git` |
| **Usage Strategy** | (1) Parse `MCQA/Fin_E.json`, `Fin_M.json`, `Fin_D.json` → (2) Extract question+explanation pairs as knowledge chunks → (3) Embed into `telecom_knowledge` or a new `oran_knowledge` collection → (4) Optionally use as supplementary SFT data for O-RAN queries → (5) Use as a second evaluation benchmark alongside TeleQnA |

> [!TIP]
> **ORAN-Bench-13K is the missing puzzle piece.** It gives you O-RAN domain text without needing to manually download and parse 116 O-RAN Alliance PDFs. Combined with Colosseum COMMAG (telemetry data), you have both the **text knowledge** and **operational data** sides of O-RAN covered.

---

### 🟢 Tier 3: NICE TO HAVE (Small boost, low effort)

#### Dataset F: Hand-Crafted Incident KB

| Attribute | Details |
|---|---|
| **Source** | Self-authored (you write these) |
| **Format** | 20-30 structured incident reports in markdown/JSON |
| **Size** | ~50 KB |
| **Access** | ✅ You create it |
| **Effort to Use** | 🟡 **2-3 hours** — Write realistic RAN incident reports based on real patterns |
| **Impact on Quality** | 🟡 **MEDIUM** — Enables RCA engine to do historical pattern matching. Makes demos compelling |
| **Usage Strategy** | Write incidents covering: handover storms, PRACH congestion, S1 failures, interference events, slice resource starvation. Embed into `oran_anomalies` collection |

---

### ❌ Tier 4: SKIP (High effort, low/uncertain return)

| Dataset | Why Skip | Effort | Risk |
|---|---|---|---|
| **O-RAN Alliance Spec PDFs** (direct) | PDFs from o-ran.org. 116+ documents. Requires manual download, PDF parsing, heavy curation. ORAN-Bench-13K already distills this | 15-25 hrs | 🔴 High — scattered quality, parsing failures |
| **5G3E Dataset** | 1.1 TB, author contact needed, unreliable timeline | 3-7 days wait + 3-4 hrs processing | 🔴 High — may never arrive |
| **Simu5G Generated Data** | OMNeT++ setup, Linux only, 12-20 hrs | 12-20 hrs | 🔴 Very High — setup failures likely |
| **O-RAN SC Nexus** | Docker container images, not structured datasets. Scattered across projects | 4-8 hrs | 🔴 High — wrong type of data |
| **5GAD** | Attack traffic PCAPs. Niche, not directly useful for RAN RAG | 3-4 hrs | 🟡 Medium — tangential |

---

## 3. The O-RAN Decision 🔴

> [!CAUTION]
> You said "ORAN is undoable" — here's why you're right, and what to do instead.

### The Problem with Direct O-RAN Spec Ingestion

| Issue | Reality |
|---|---|
| **Access** | O-RAN Alliance specs are PDFs on o-ran.org. They're downloadable but there are 116+ documents across 10+ working groups |
| **Parsing** | Each PDF has different formatting. Tables, figures, YANG models mixed with prose. PyMuPDF parsing quality will be inconsistent |
| **Volume** | Processing 116 PDFs through contextual chunking + embedding = 15-25 hours of development time |
| **Testing** | You'd need to validate chunk quality across all docs — no ground truth exists for retrieval from O-RAN specs |
| **Timeline** | You have 40 build days. Spending 3-5 days on O-RAN PDF parsing alone is ~10% of your total build time for uncertain quality |

### My Recommendation: ❌ DROP Direct O-RAN Spec Parsing

**Instead, use the 2-pronged O-RAN proxy strategy:**

```
O-RAN Requirement Coverage:
├── TEXT KNOWLEDGE: ORAN-Bench-13K (13,952 MCQs from 116 O-RAN specs)
│   └── Already distilled, structured, ready to embed
│   └── Effort: 3-4 hours
│   └── Coverage: O-RAN architecture, interfaces, protocols, security
│
└── OPERATIONAL DATA: Colosseum COMMAG (real O-RAN RAN KPIs)
    └── Real base station + UE + slice metrics
    └── Effort: 5-6 hours (with anomaly injection)
    └── Coverage: O-RAN deployment scenarios, scheduling, anomalies
```

### Why This Works for the Hackathon

1. **Satisfies R3** ("Use of public datasets like TeleQnA and O-RAN") — Colosseum IS an O-RAN dataset. ORAN-Bench IS derived from O-RAN specs
2. **Saves 15-20 hours** — which you redirect to pipeline quality, security features, and UI polish
3. **Better chunk quality** — ORAN-Bench questions + explanations are already coherent text vs. raw PDF parsing artifacts
4. **Dual evaluation** — You can evaluate on both TeleQnA (10K) AND ORAN-Bench (13K) for comprehensive coverage
5. **Defensible in presentation** — "We leveraged ORAN-Bench-13K, a curated benchmark derived from 116 O-RAN specification documents, providing comprehensive O-RAN domain coverage"

### What You Lose (And Why It's Acceptable)

| Lost Capability | Impact | Mitigation |
|---|---|---|
| Deep O-RAN spec retrieval (e.g., "What does O-RAN WG2 A1 interface spec say about...") | 🟡 Medium | ORAN-Bench explanations cover the key concepts. For deep spec questions, the model can cite the source spec number |
| Full YANG model parsing | 🟢 Low | Not relevant for the 3 use cases (Q&A, RCA, Anomaly Detection) |
| Complete O-RAN architecture diagrams | 🟢 Low | Can be supplemented with 5-10 hand-crafted architecture summaries |

> [!IMPORTANT]
> **Final Verdict: DROP O-RAN spec PDF ingestion. Use ORAN-Bench-13K + Colosseum COMMAG instead. Net savings: 15-20 hours. Net quality loss: minimal.**

---

## 4. Revised Data Pipeline Plan

### Priority Order (Execute in This Sequence)

| # | Dataset | Action | Effort | Week | Owner |
|---|---|---|---|---|---|
| 1 | **TeleQnA** | Download from HuggingFace. Parse. Stratified 80/20 split. Format for SFT + eval + retrieval | 2-3 hrs | W1 | Person 1 |
| 2 | **3GPP Specs** (10 must-have) | Download from ETSI. Convert `.docx→.pdf`. PyMuPDF parse. Contextual chunk. Table image extraction. Embed | 6-8 hrs | W1-W2 | Person 1 |
| 3 | **Colosseum COMMAG** | `git clone`. Write anomaly injection script. Prose template conversion. Build incident KB. Embed | 5-6 hrs | W1 | Person 2 |
| 4 | **ORAN-Bench-13K** | `git clone`. Parse 3 JSON files. Extract Q+explanation pairs. Embed into knowledge collection | 3-4 hrs | W1 | Person 2 |
| 5 | **Tele-Data** | Download from HuggingFace. Filter 3GPP+arXiv subsets. Format for CPT | 2-3 hrs | W2 | Person 2 |
| 6 | **Incident KB** | Write 20-30 RAN incident reports. Embed | 2-3 hrs | W2 | Either |
| | **TOTAL** | | **21-27 hrs** | | |

### ChromaDB Collection Plan (Revised)

| Collection | Sources | Est. Chunks | Notes |
|---|---|---|---|
| `3gpp_specs` | 3GPP Rel-16/18 PDFs (10 specs) | ~15,000 | Contextual chunking + table image crops |
| `oran_knowledge` | ORAN-Bench-13K Q+explanations + Colosseum prose summaries + Incident KB | ~20,000 | NEW: combines O-RAN text knowledge with operational data |
| `telecom_knowledge` | TeleQnA explanations + Tele-Data subsets | ~12,000 | General telecom domain knowledge |
| **TOTAL** | | **~47,000** | Up from 32K estimate — more comprehensive |

---

## 5. Download Commands Cheat Sheet

```bash
# === TIER 1: MUST HAVE ===

# TeleQnA (Python)
pip install datasets
python -c "from datasets import load_dataset; ds = load_dataset('netop/TeleQnA'); ds.save_to_disk('./data/teleqna')"

# Colosseum O-RAN COMMAG
git clone https://github.com/wineslab/colosseum-oran-commag-dataset.git ./data/colosseum

# 3GPP Specs — Manual download from:
# https://www.3gpp.org/DynaReport/38-series.htm
# OR https://www.etsi.org/standards (search by spec number)
# Priority: TS 38.300, 38.331, 38.211, 38.212, 38.213, 38.214, 38.401, 38.321, 38.322, 38.323
# After download:
# soffice --headless --convert-to pdf *.docx  (LibreOffice)

# === TIER 2: SHOULD HAVE ===

# ORAN-Bench-13K
git clone https://github.com/prnshv/ORAN-Bench-13K.git ./data/oran-bench

# Tele-Data (Python)
python -c "from datasets import load_dataset; ds = load_dataset('AliMaatouk/Tele-Data'); ds.save_to_disk('./data/tele-data')"
```

---

## 6. Impact Matrix: Effort vs. Quality vs. Risk

```
                    HIGH IMPACT
                        │
    ┌───────────────────┼───────────────────┐
    │                   │                   │
    │  3GPP Specs       │  TeleQnA          │
    │  (6-8 hrs)        │  (2-3 hrs)        │
    │                   │                   │
    │  Colosseum COMMAG │                   │
    │  (5-6 hrs)        │                   │
HIGH├───────────────────┼───────────────────┤LOW
EFFORT                  │                   EFFORT
    │                   │  ORAN-Bench-13K   │
    │  O-RAN Specs ❌   │  (3-4 hrs)        │
    │  (15-25 hrs)      │                   │
    │                   │  Tele-Data        │
    │  5G3E ❌          │  (2-3 hrs)        │
    │  (3-7 days wait)  │                   │
    │                   │  Incident KB      │
    │                   │  (2-3 hrs)        │
    └───────────────────┼───────────────────┘
                        │
                    LOW IMPACT
```

> [!NOTE]
> The sweet spot is the **upper-right quadrant**: high impact, low effort. TeleQnA, ORAN-Bench-13K, and Tele-Data all live there. Colosseum and 3GPP specs are upper-left (high impact, moderate effort) — still worth doing. O-RAN Alliance spec PDFs are lower-left (high effort, uncertain impact) — SKIP.

---

## 7. Summary & Decisions Needed

### ✅ Confirmed Decisions

| Decision | Status |
|---|---|
| TeleQnA is primary eval + SFT source | ✅ Locked |
| 3GPP specs (10 must-have) are primary knowledge base | ✅ Locked |
| Colosseum COMMAG is primary O-RAN operational data | ✅ Locked |
| Tele-Data is CPT corpus | ✅ Locked |
| 5G3E, Simu5G, O-RAN SC Nexus are SKIPPED | ✅ Locked |

### 🔴 Decision Requested: O-RAN Spec PDFs

| Option | Effort | Benefit | Risk | Recommendation |
|---|---|---|---|---|
| **Option A: Drop O-RAN specs entirely** → Use ORAN-Bench-13K as text proxy | 3-4 hrs | Saves 15-20 hrs. Good coverage via 13K MCQs | Lose deep spec retrieval | **✅ RECOMMENDED** |
| **Option B: Process 10-15 key O-RAN specs** (partial) | 8-12 hrs | Better spec retrieval depth | Significant time sink, uncertain parsing quality | ⚠️ Only if you have spare time in Week 5+ |
| **Option C: Process all 116 O-RAN specs** | 15-25 hrs | Complete coverage | Unsustainable for 2-person team in 40 days | ❌ NOT RECOMMENDED |

> [!WARNING]
> **Your Phase 1 deadline is TOMORROW (May 13).** The blueprint PPT should reflect whichever decision you make here. If you go with Option A (recommended), the Data Strategy slide should say:
> 
> *"O-RAN domain knowledge sourced from ORAN-Bench-13K (13,952 curated questions from 116 O-RAN specifications) + Colosseum COMMAG real-world RAN telemetry. This dual-source approach provides both textual O-RAN knowledge and operational data coverage."*
