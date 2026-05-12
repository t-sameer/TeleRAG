# ⚡ Compute & Model Feasibility Analysis
**Target:** Gemma 4 E4B, BGE-Base, BGE-Reranker vs. Kaggle/Colab Free Tiers
**Date:** May 12, 2026

## 1. Model VRAM Feasibility (Target: Nvidia T4 16GB)

The choice of **Gemma 4 E4B-it** (Edge 4 Billion parameters) is highly optimized for free-tier constraints. 

### 1.1 Inference Budget (Notebook 3)
| Component | Precision | VRAM Cost | Notes |
|---|---|---|---|
| Gemma 4 E4B | 4-bit (bitsandbytes) | ~2.8 GB | Base weights |
| KV Cache (Gemma) | FP16 | ~1.5 - 2.5 GB | Depends on context length (up to 8K used) |
| BGE-base-en-v1.5 | FP16 | ~0.3 GB | Very lightweight embedding |
| BGE-reranker-base | FP16 | ~0.7 GB | Cross-encoder |
| CUDA/PyTorch Overhead | — | ~1.5 GB | Standard framework overhead |
| **Total Peak VRAM** | | **~7.8 GB** | **Result: Extremely safe (16GB T4)** |

*Verdict:* We have over 8GB of VRAM headroom during inference. This guarantees no Out-Of-Memory (OOM) errors even when processing long 3GPP document contexts and generating CoT reasoning.

### 1.2 Training Budget (Notebook 2 - QLoRA via Unsloth)
| Component | VRAM Cost | Notes |
|---|---|---|
| Gemma 4 E4B Base | ~2.8 GB | 4-bit loaded via Unsloth |
| LoRA Adapters (r=16, α=32) | ~0.4 GB | Targeting `q_proj`, `k_proj`, `v_proj`, `o_proj`, `gate_proj`, `up_proj`, `down_proj` |
| Paged AdamW 8-bit Optimizer | ~1.2 GB | Paged optimizer prevents VRAM spikes |
| Activations / Gradients | ~2.5 - 4.0 GB | With Gradient Checkpointing enabled (BS=1, GradAccum=4) |
| **Total Peak VRAM** | **~6.9 - 8.4 GB** | **Result: Extremely safe (16GB T4)** |

*Verdict:* Unsloth makes training a 4B model trivial on a T4. We don't even need to fallback to `r=8` or reduce the target modules. 

---

## 2. Training Time Feasibility (Kaggle Quota limits)

Kaggle provides **30 hours per week** of T4x2 GPU time per account. We have 2 team members, meaning **60 hours/week total**.

### Phase 1: CPT (Continued Pre-Training)
- **Dataset:** ~500MB of Tele-Data (3GPP + arXiv).
- **Process:** Causal Language Modeling next-token prediction.
- **Estimated Time:** Unsloth trains 4B models at ~1.5x-2x speed. 1 epoch over 500MB of text will take approximately **4.5 to 6.5 hours** on a single T4.
- **Quota Impact:** ~20% of one person's weekly quota. Highly doable.

### Phase 2: SFT (Supervised Fine-Tuning)
- **Dataset:** 8K TeleQnA training examples + handful of synthetic anomalies.
- **Process:** Instruction tuning formatting.
- **Estimated Time:** 8K examples × 2 epochs ≈ 16K steps (at BS=1). With gradient accumulation, this takes **~2 to 3 hours**.
- **Quota Impact:** ~10% of one person's weekly quota.

*Verdict:* A full training loop (CPT + SFT) costs ~9 hours. You can run 3 full experiment iterations per week per person. **Compute time is NOT the bottleneck.**

---

## 3. The Hidden Killers: Disk Space & CPU RAM

While GPU limits are fine, Kaggle and Colab free tiers have strict non-GPU limits that frequently fail hackathon projects.

### 3.1 The Disk Space Trap (Kaggle 20GB Limit)
- **The Threat:** Kaggle's `/kaggle/working/` directory has a hard 20GB limit. During CPT, HuggingFace `Trainer` saves checkpoints. A single adapter checkpoint is small (~100MB), but if `save_steps` is frequent and optimizer states are saved, it can rapidly balloon to 15GB+.
- **The Fix:** We MUST set `save_total_limit=2` in `TrainingArguments` to ensure old checkpoints are deleted. At the end of training, the final model MUST be pushed to HuggingFace Hub immediately (`model.push_to_hub()`), rather than relying on Kaggle's local disk persistence.

### 3.2 The CPU RAM Trap (Colab 12GB / Kaggle 30GB Limit)
- **The Threat:** Loading 500MB of raw text into a Python string and tokenizing it simultaneously will spike CPU RAM to 4x-5x the dataset size (~2.5GB). If we use Pandas or naive lists, Kaggle might handle it, but Colab's 12GB RAM will crash.
- **The Fix:** Use HuggingFace `datasets` library. We must use `.map()` with `batched=True` and `remove_columns` to stream the tokenization to disk rather than keeping it all in RAM.

### 3.3 Colab Disconnects
- **The Threat:** Colab Free tier terminates idle sessions fast and aggressively targets long-running background tasks. A 6-hour CPT run on Colab Free has a >70% chance of being interrupted.
- **The Fix:** Do NOT use Colab for training. 

---

## 4. Final Platform Allocation Strategy

To survive the hackathon without paying for compute:

| Task | Platform | Why? |
|---|---|---|
| **Data Parsing & Chunking** | Local Machine / CPU Colab | PyMuPDF parsing and contextual chunking is CPU-heavy. Save GPU quota. |
| **LoRA Training (CPT + SFT)** | **Kaggle** (Primary) | Kaggle guarantees 12-hour session persistence. The 6-hour CPT run will not be interrupted. |
| **UI Dev & Inference Testing** | **Colab T4** (Secondary) | Streamlit runs well on Colab via `localtunnel`. Colab starts faster for rapid code testing. |
| **10K Final Evaluation** | **Kaggle** | The full 10K RAG evaluation will take 3-4 hours of inference time. Kaggle won't disconnect. |

## 5. Conclusion & Confidence
**Feasibility Score: 9.5 / 10**

The architecture is perfectly sized for free tiers. By selecting the Gemma 4 E4B (4B params) instead of a heavier 7B/8B model (like Llama 3 8B), we completely eliminated VRAM pressure. Unsloth is the MVP here, allowing us to do aggressive CPT and CoT optimization without hitting quota ceilings. As long as we strictly manage Kaggle's 20GB disk limit during training, compute will not block this project.
