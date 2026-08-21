# Eval-before-train

Train loss going down means “the adapter is fitting the training strings.” It does not mean the product got better. The rule for this repo: **never train without a held-out eval file**, even if it has ~30 examples.

## The loop (do this in order)

```text
1. Write the task metric and the eval file   ← before any GPU
2. Score the BASE instruct model (prompted)  ← baseline
3. Train (QLoRA SFT)
4. Score the SAME eval file
5. If metric did not move → change DATA, not epoch count
```

Step 2 is the one people skip. Without it you cannot say “fine-tuning helped”; you might have had a strong prompt already.

## What to measure (task success, not perplexity)

Perplexity / train loss are debugging signals. **Ship and write up a metric that a skeptic cannot game with fluent nonsense.**

| Task type | Automatic metric | Why it is hard to fake |
|-----------|------------------|-------------------------|
| Structured JSON | Schema-valid % (JSON parse + required keys/types) | “Sounds right” still fails `json.loads` |
| Extraction | Field-level exact match / F1 vs gold | Partial credit without vibes |
| Text-to-SQL | **Execution accuracy** on a frozen DB | Equivalent SQL can still pass; string match alone is brittle |
| Tool calling | Valid `tool_call` block + argument schema | Format + types |
| Cite-or-abstain | Citation present when evidence exists; refusal when not; hallucination rate | Rewards silence and grounding, not verbosity |
| Identity / FAQ | Exact or rubric match on held-out *paraphrases* | Catches memorization of train wording |

Phase 2 bar: **~20 held-out prompts**, scored by a script (valid JSON? required fact present?). Mini-projects and the capstone add a real table: **base vs SFT** (and **SFT vs DPO** later).

Human notes still matter: **write 10 failure cases by hand**. That paragraph is more convincing than a W&B screenshot.

## Held-out data that actually tests generalization

Eval must not be a shuffled clone of train.

- **Paraphrase** user wording. Same intent, different surface form.
- **Keep the same schema / label space.** Do not change the task between train and test.
- **No leakage:** if you generated synthetically, split **before** heavy filtering, or hash prompts and drop near-duplicates (M3).
- **Tiny eval (30 rows) is allowed** at Phase 2. It is a **signal**, not a publication. Confidence intervals are huge; you still need the number so you cannot lie to yourself.
- Include **negatives**: inputs that should refuse, omit a field, or return `unknown`. A metric that only scores happy-path JSON will hide collapse.

## Overfitting tiny datasets

Phase 2 uses **50–200 hand-edited examples**. That is enough to teach a format and also enough to memorize.

Signs you memorized:

- Train loss near zero, eval metric flat or down.
- Model recites a training answer for a *similar* but not identical ticket.
- Changing one entity in the prompt still yields the old entity.

Mitigations that are not “train longer”:

- Diversify templates and entity values.
- Fewer epochs (1–2), early stop on **eval metric**.
- Lower rank if you are clearly memorizing (see LoRA note).
- Add paraphrases to **train** only after eval is locked.

**If the metric did not move, iterate on data, not epochs.** That sentence is an exit criterion for Phase 2.

## Catastrophic forgetting

Narrow SFT can hurt capabilities you did not score.

Cheap insurance:

- A **small general probe** (10–20 generic instruct prompts: math one-liner, a refusal, a rewrite). Track pass/fail by eye or a simple rubric.
- Do not declare victory if schema-pass went 40% → 95% but the model now ignores “answer in one sentence” on unrelated prompts — unless that trade is explicit.

LoRA reduces how much of \(W\) you touch; it does not make forgetting impossible.

## Baseline discipline (this is what “outshines”)

For every later project, the writeup should contain:

1. **Prompted base** on the eval file (same decoding: temperature, max tokens, stop strings).
2. **SFT adapter** on the same file.
3. Optional: **SFT + DPO**.
4. One ablation: rank, data size, or epochs — **one table**, not twenty.

Decoding must be fixed. Sampling at T=0.8 for base and T=0.0 for SFT is not a fair fight. Prefer greedy or low-T for schema tasks.

## What “loss dropped” is allowed to mean

You may write: “Loss dropped, **and** schema-pass moved from X to Y on 20 held-out prompts.”

You may not write: “Loss dropped, therefore the model learned the task.”

If Y ≤ X: the honest README says the metric did not move, then you show the data change that you try next. That honesty is part of the portfolio.

## Interview answers (short)

**“How do you know fine-tuning worked?”**  
Held-out task metric vs the **same** prompted base. Loss is not the metric.

**“How big does eval need to be?”**  
As big as you can label. Thirty well-chosen, non-leaking items beat 5,000 noisy ones. Capstone: large enough to report schema-pass and field F1 without a straight face.

**“Won’t a small eval overfit your design choices?”**  
Yes. Lock the eval file before you iterate. Touch it only to fix labeling errors, and disclose that.

**“Eval-before-train — why?”**  
Because you might not need to train. A 7B with a strict system prompt and JSON mode might already hit 90% schema-pass. Then fine-tuning is optional, not a hero narrative.
