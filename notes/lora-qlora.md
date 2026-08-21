# LoRA and QLoRA

Full fine-tuning a 7B model in bf16 is already ~14 GB of weights. Adam keeps extra optimizer state (typically 8 bytes/param). That does not fit a Colab L4/A100 comfortably once you add activations. **LoRA** trains a few million extra parameters. **QLoRA** also stores the frozen base in 4-bit. That is why this path never full-FT a 7B.

## LoRA: low-rank update

For a frozen weight \(W \in \mathbb{R}^{d \times k}\), LoRA learns

\[
\Delta W = BA, \quad B \in \mathbb{R}^{d \times r},\; A \in \mathbb{R}^{r \times k},\; r \ll \min(d,k)
\]

Forward pass (PEFT-style scaling):

\[
h = Wx + \frac{\alpha}{r}\, BAx
\]

Standard init: **A** Gaussian, **B** zeros, so \(\Delta W = 0\) at step 0 and you do not destroy the base model on the first batch.

| Symbol | Typical values on this path | Role |
|--------|-----------------------------|------|
| \(r\) (rank) | 8, 16, 32 | Capacity of the adapter. Phase 2 default **r=16**; ablation r=8 vs r=16. |
| \(\alpha\) (`lora_alpha`) | often 16 or 32 (same order as \(r\)) | Scale of \(\Delta W\). People quote \(\alpha/r\). |
| Target modules | attention `q,k,v,o`; often MLP `gate,up,down` | Where adapters attach. Attention-only is smaller; adding MLP helps many instruction tasks. |
| Dropout | 0–0.05 | Regularizer on the adapter. Tiny datasets: keep it light; data quality matters more. |

Trainable parameter count is roughly \(2\,r\,(d+k)\) per adapted matrix — usually **< 1%** of the base.

**Merge for serving:** \(W' = W + \frac{\alpha}{r}BA\). After merge you have a single dense model (or you keep adapter + base separate). Later: merge → GGUF/Ollama if you want a local demo.

### What rank actually does

- **Too small (\(r=4\) on a hard skill):** underfit. Loss stalls, eval metric barely moves.
- **Too large on 50–200 examples:** extra capacity memorizes wording. Train loss → 0, held-out paraphrases fail.
- **More rank ≠ more intelligence.** Rank is bandwidth for *how much* you can change \(W\). The data decides *what* change is.

If an interviewer asks “why r=16?”, say: “enough to move instruction/schema behavior on 7B-class models in the Unsloth/PEFT defaults; we ablate r=8 vs r=16 rather than guessing.”

## QLoRA: 4-bit frozen base + 16-bit LoRA

**QLoRA** (Dettmers et al., 2023) is the recipe this repo uses on Colab:

1. Store the base weights in **4-bit NormalFloat (NF4)** — a quantization grid matched to the roughly Gaussian weight distribution.
2. **Dequantize to 16-bit** (fp16/bf16) for the matmul; LoRA math stays in 16-bit.
3. **Double quantization:** quantize the quantization constants themselves to save a bit more VRAM.
4. **Paged optimizers:** spill Adam state to CPU RAM via unified memory if VRAM spikes, instead of hard OOM.

You train **only** \(A,B\) (plus maybe a bit of layernorm / embed if you unfreeze them — default is don’t). The 4-bit \(W\) does not get gradient updates.

**Unsloth** is a faster QLoRA implementation (custom kernels, less VRAM, same scientific idea). The mental model does not change: frozen quantized base, trainable low-rank adapters.

### VRAM sketch (order of magnitude, not a quote)

| Setup | 7B-class, train | Why |
|-------|-----------------|-----|
| Full FT bf16 + Adam | often 60 GB+ | 2 bytes/param weights + ~8 bytes/param optimizer + activations |
| LoRA bf16 base | maybe ~20 GB | Full-precision base + tiny adapters |
| QLoRA 4-bit + LoRA | often ~8–16 GB | Fits L4/A100 with sequence length and batch as the remaining knobs |

Sequence length and batch size dominate remaining VRAM. Packing (Phase 3 M2) is about using that budget well, not about LoRA math.

## Catastrophic forgetting and tiny data

LoRA is *less* destructive than full FT because \(\Delta W\) is low-rank and starts at zero, but it is **not** a forgetting vaccine.

- **Narrow SFT** (one JSON schema, 200 clones of the same template) can make the model worse at open chat, math, or following a different schema. Measure a **general** probe (a few MMLU-style or plain instruct prompts) plus your **task** metric.
- **Tiny datasets** (Phase 2: 50–200 rows): the adapter can memorize exact strings. That looks like “the model learned the FAQ” and fails the moment the user paraphrases. Held-out prompts must not be near-duplicates of train.
- **More epochs** on the same 80 rows is how you overfit. If eval is flat after epoch 1, **rewrite data**, do not go to epoch 8.

Identity LoRA (mini-project M1) exists to *see* this: 10 fictional facts will stick, and the model will also start sounding like your train template. That is the point.

## Practical knobs you will actually touch

| Knob | Default instinct on this path |
|------|-------------------------------|
| Base | Instruct checkpoint (`Qwen2.5-*-Instruct`), not base-pretrained, unless you have a reason |
| Quant | 4-bit QLoRA via Unsloth / bitsandbytes |
| \(r\) | 16, then ablate 8 |
| LR | ~1e-4 to 2e-4 for LoRA is a common band; full-FT LRs are lower. Unsloth recipes are a fair start. |
| Epochs | 1–3 on small custom data; stop on **eval metric**, not train loss |
| Max seq len | Fit the longest *real* example; padding waste is why packing appears in M2 |

## Interview answers (short)

**“What is LoRA?”**  
Freeze \(W\), train \(BA\) with \(r \ll d\). Same architecture at inference if you merge. You change a low-dimensional subspace of each adapted matrix.

**“What is QLoRA?”**  
LoRA on a **4-bit frozen** backbone, adapters and compute in 16-bit. Makes 7B–14B SFT realistic on one Colab GPU.

**“Does QLoRA match 16-bit LoRA quality?”**  
Close enough on instruction tasks that the industry defaulted to it. Your job is to prove it on **your metric**, not to re-derive NF4.

**“Full fine-tune vs LoRA?”**  
Full FT can move more of the network and forget more. LoRA is cheaper, easier to ship as an adapter, and enough for schema/style/tool skills. You would consider full FT only with more GPUs and a forgetting budget — not on this compute envelope.
