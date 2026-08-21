# Interview cheatsheet (Phase 1)

If this page is a wall of acronyms, read [00-start-here.md](00-start-here.md) first.

Ninety-second answers. Details live in the topic notes.

**Did fine-tuning teach the model my PDF?**  
No. The LM still predicts the next token. SFT shifts that distribution toward your examples. Facts that must stay current or citable belong in **RAG** (or a tool), not only in weights.

**When do you fine-tune?**  
When prompting, JSON/constrained decoding, and RAG still fail a **task metric**, and you can show demonstrations of the missing behavior: style, schema reliability, tool format, or a narrow skill.

**SFT vs DPO vs RAG?**  
SFT: clone a target assistant completion (format/skill). DPO: prefer chosen over rejected relative to an SFT reference (`β` = how hard you fit the pairs). RAG: swap *context* at inference; weights can stay frozen.

**LoRA?**  
Freeze \(W\), train \(\Delta W = BA\) with rank \(r \ll d\). Init \(B=0\) so the first step does not wreck the base. Merge: \(W' = W + (\alpha/r)BA\).

**QLoRA?**  
4-bit NF4 frozen base, 16-bit LoRA adapters and compute. Why 7B SFT fits Colab. Unsloth = faster kernels, same idea.

**Chat template?**  
The tokenizer’s official role serialization. Train and serve with `apply_chat_template`. Mismatch looks like a “dumb” fine-tune.

**Assistant-only loss?**  
Labels `-100` on user/system tokens. Otherwise you train the model to imitate the user.

**Eval-before-train?**  
Score the **prompted base** on a locked held-out file first. Success is a task metric (schema-valid %, field F1, execution accuracy), not train loss. If the metric does not move, change **data**, not epoch count.

**Forgetting / tiny data?**  
Low-rank still overfits 50–200 near-duplicate strings and can bleed style onto OOD prompts. Paraphrase eval; keep a tiny general probe; stop on the metric.
