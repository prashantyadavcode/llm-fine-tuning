# Phase 1 — Mental models

Goal: answer “why fine-tune, how, and how do you know it worked?” without waving hands. You already know loss and backprop; this phase is the LLM-specific layer on top.

**Compute reminder:** Colab Pro (L4/A100). QLoRA on 7B–14B is the ceiling. Interviewers care about data, eval, and judgment more than parameter count.

## What you should be able to say out loud

1. Fine-tuning does not “upload a PDF.” It shifts next-token probabilities.
2. The usual post-pretrain stack is **SFT → preference (DPO/ORPO) → optional RL**. This path does the first two.
3. **Do not fine-tune** when RAG, prompting, or constrained decoding already solves the problem.
4. Instruct models expect a **chat template**. Train and serve with the same one. Mask user tokens so loss is assistant-only.
5. **LoRA** is a low-rank update `ΔW = BA`. **QLoRA** keeps a 4-bit frozen base and trains those adapters in 16-bit.
6. Measure the **base model on a held-out task metric before any training**. If the metric does not move, iterate on data, not epochs.

## Reading order

| Order | Note | Why it exists |
|-------|------|----------------|
| 1 | [sft-vs-rag-vs-dpo.md](sft-vs-rag-vs-dpo.md) | When to fine-tune, what SFT vs DPO vs RAG actually change |
| 2 | [lora-qlora.md](lora-qlora.md) | Why full FT does not fit Colab; rank, α, merge |
| 3 | [chat-templates.md](chat-templates.md) | Tokenizers, `apply_chat_template`, assistant-only loss |
| 4 | [eval-before-train.md](eval-before-train.md) | Held-out metrics, overfitting, catastrophic forgetting |
| 5 | [tiny-experiment.md](tiny-experiment.md) | 2-hour lab: same prompt, base vs a public LoRA |
| — | [interview-cheatsheet.md](interview-cheatsheet.md) | 90-second answers for the six points above |

Skip for now (on purpose): full RLHF/PPO, GRPO at scale, multi-GPU Axolotl, writing CUDA kernels.

## Exit

You can explain each row in the table above in ~90 seconds, and you have run (or scheduled) the tiny adapter experiment. Phase 2 is the first real SFT.
