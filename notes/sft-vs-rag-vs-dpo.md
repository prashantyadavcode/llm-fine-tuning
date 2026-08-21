# SFT vs RAG vs DPO

This is the decision note. Most “should I fine-tune?” questions collapse into: **what do I need the weights to change?**

## Next-token prediction is not “the model learned my PDF”

A causal LM is trained to model

\[
P(x_t \mid x_{<t})
\]

Every stage below is still this objective (or a close cousin). What changes is **which tokens you score, and against what target**.

Consequences that matter in interviews:

- Putting a document in the training set does **not** create a lookup table. The model may paraphrase, drop numbers, or mix two facts. That is next-token interpolation, not retrieval.
- A fact that appears once in 80 examples is more likely **memorized** than **generalized**. Paraphrase the eval prompts; if accuracy collapses, you overfit the wording.
- SFT on “here is our wiki” is a weak knowledge store. If the wiki changes next week, you retrain. RAG reads the current wiki at inference.

If someone says “we fine-tuned so the model knows our docs,” the precise version is: “we shifted the completion distribution toward the style and patterns in those docs; factual recall is unreliable unless we retrieve or the fact was repeated and tested.”

## The post-pretrain stack

You start from a pretrained (usually **instruct**) checkpoint. You do not pretrain from scratch.

```text
pretrain  →  SFT  →  preference (DPO / ORPO)  →  optional RL (PPO / GRPO)
   ▲           ▲              ▲                         ▲
 cheap to    this path      this path                 skip for now
 consume     does this      does this (M5 + capstone)
```

| Stage | What you show the model | What usually changes |
|-------|-------------------------|----------------------|
| **Pretrain** | Raw text, next token | Language, some world knowledge. You buy this as a checkpoint. |
| **SFT** | Prompt → *one* desired completion | Default behavior: format, tone, tool syntax, a narrow skill. |
| **DPO / ORPO** | Prompt → chosen vs rejected completions | Ranking among outputs the SFT model can already produce. |
| **RL** | Sample, score, update (reward / verifier) | Harder, noisier. Not needed for the resume path. |

SFT teaches *what good looks like*. Preference tuning teaches *this is better than that*. RL is for when you have a reliable scalar reward (unit tests, a judge, a sandbox) and you are willing to pay for sampling.

**ORPO** (odds-ratio preference optimization) folds a preference term into SFT and does not need a frozen reference model. **DPO** is the one this path uses: it needs a reference policy (usually the SFT model) and explicit pairs.

## What each tool is for

### RAG — change the *context*, not the weights

At inference, retrieve chunks, put them in the prompt, generate. Weights stay frozen (or you only SFT the *behavior* of using context).

Use RAG when:

- Facts change (prices, policies, ticket history, API docs).
- You need citations or “I don’t know if it isn’t in the snippet.”
- The knowledge base is large relative to what you can memorize in an adapter.

RAG does **not** reliably teach a new output schema, a house style, or a tool-call dialect. The model can still ignore the retrieved text and hallucinate.

### SFT — change the *default completion*

You maximize likelihood of target assistant tokens given the chat-formatted prompt.

Use SFT when the base instruct model **fails a behavior you can demonstrate with examples**:

- Strict JSON / a fixed schema, every time.
- A tool-call or function-call format.
- A house voice, refusal style, or “cite or abstain” policy.
- A narrow skill (text-to-SQL for *your* schema, ticket routing labels) where prompting is unstable.

SFT is a poor first choice for:

- “Make it know Q3 2026 headcount” → RAG or a tool.
- “Just make it output JSON” if `response_format` / constrained decoding / a good prompt already hits your pass rate. Try that **before** burning GPU hours.

### DPO — change *which* of two completions it prefers

DPO does not invent new facts. It upweights `y_chosen` and downweights `y_rejected` relative to a reference model (typically the SFT checkpoint).

Use DPO when SFT already speaks the language, but:

- It is verbose, hedge-y, or slightly off-schema.
- Typical failures are *recognizable* (you can write a rejected completion).
- You care about style, safety-ish refusals, or “prefer cited over guessed.”

The usual capstone pattern: **SFT until the format exists, then DPO on pairs mined from SFT failures** (rejected = the bad output you actually saw).

Rough DPO idea (enough for an interview):

\[
L_{\text{DPO}} = -\log\sigma\Big(\beta \big[\log\tfrac{\pi_\theta(y_w\mid x)}{\pi_{\text{ref}}(y_w\mid x)} - \log\tfrac{\pi_\theta(y_l\mid x)}{\pi_{\text{ref}}(y_l\mid x)}\big]\Big)
\]

`β` is a KL-style temperature: small `β` stays close to the SFT model; large `β` fits the pairs harder and can collapse or get sycophantic. You will feel this in M5.

## When *not* to fine-tune (say this first)

Fine-tuning is justified when **prompting + tools + RAG still fail a metric you care about**, and you can write or curate demonstrations of the fix.

| Symptom | Try first | Fine-tune if |
|---------|-----------|----------------|
| Wrong or stale facts | RAG, tools, search | You also need a *habit* (always cite, always abstain) |
| JSON sometimes invalid | Prompt, JSON mode, grammar/constrained decode | Pass rate still low at scale, or schema is model-emitted not constrained |
| Tone / brand voice | System prompt, few-shot | Voice must survive long context and paraphrases |
| Tool-calling format | Prompt + parser retries | Parser still fails; you need native `tool_call` reliability |
| Narrow domain skill | Few-shot of 5–20 gold examples | Few-shot is too long, too flaky, or too expensive at serving |

**Rule of thumb:** facts → RAG; format you can already get → prompting/constraints; **style, schema reliability, tool format, or a skill the base model fails** → SFT, then DPO if ranking still matters.

## They compose (this is the adult answer)

A product stack is often:

1. **RAG** supplies current evidence.
2. **SFT** teaches “answer only from context, emit this schema, refuse if empty.”
3. **DPO** prefers grounded schema-valid answers over fluent hallucinations.

Evaluating only train loss cannot tell these layers apart. You need a held-out task metric for each claim (retrieval hit rate vs schema pass vs preference win rate).

## Interview answers (short)

**“SFT vs DPO?”**  
SFT clones a single target completion. DPO only needs *relative* quality: chosen vs rejected. DPO is for steering an SFT model, not for teaching a brand-new format from scratch.

**“Why not just RAG?”**  
RAG puts evidence in the window. It does not force a schema, a tool dialect, or a refusal policy. If the base model ignores context, SFT the *use of context*, still retrieve at inference.

**“Did the model learn our PDF?”**  
No. It learned to continue text that *looks like* the PDF’s distribution. For questions whose answers are in the PDF, retrieve the PDF.

**“ORPO vs DPO?”**  
ORPO: one stage, no reference model. DPO: two stages (SFT then preference), reference KL control via `β`. This repo standardizes on DPO after SFT because it is the common job-interview recipe and pairs well with mining failures.
