# Tiny experiment (~2 hours): see an adapter change behavior

No long training. Goal: **the same prompt, base vs a public LoRA**, so “adapters change next-token behavior” is something you have watched, not only derived.

Do this after Phase 0 can load a 4-bit Qwen. If Phase 0 is not ready, keep this as a checklist.

## Setup

- Pick a **small instruct** model you can load in 4-bit on Colab (e.g. Qwen 2.5 1.5B Instruct).
- Pick a **public PEFT adapter trained for that architecture** (Hugging Face `adapter_config.json` must name a compatible base). If you cannot find a matching Qwen adapter quickly, use any tiny official example adapter from Unsloth/HF docs for the model you loaded — compatibility matters more than the adapter’s fame.
- Freeze decoding: `temperature=0` (or `do_sample=False`), same `max_new_tokens`, same chat template.

## Procedure

1. Write **5 prompts**: 3 in-distribution for whatever the adapter claims (style, language, JSON, etc.) and **2 out-of-distribution** (unrelated questions).
2. Generate with **base only**. Save raw strings to a file (Drive is fine).
3. Load the **same base + adapter** (`PeftModel.from_pretrained` / Unsloth equivalent). Generate again. Save.
4. Side-by-side: what changed? Format? Tone? Factual content? Did OOD prompts get worse?

## What you should notice

- In-domain prompts often shift **style or schema** more than “IQ.”
- OOD prompts may be unchanged, slightly worse, or oddly in the adapter’s voice (forgetting / style bleed).
- If **nothing** changes, you likely loaded the adapter on the wrong base, skipped `merge`/`enable` adapters, or used a different template at inference.

## Writeup (a few bullets in this folder later is enough)

- Adapter id + base id.
- One before/after pair that makes the change obvious.
- One failure or OOD regression.

That is the whole lab. Phase 2 is when *you* train the adapter.
