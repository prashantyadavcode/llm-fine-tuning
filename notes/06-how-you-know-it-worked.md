# 6. How you know it worked

Fine-tuning prints numbers while it runs. Those numbers are not enough. These are the terms you use to measure whether the **job** got better.

Order of work: lock an eval set → score the **baseline** → train → score the **same** eval set with the same **task metric**. If the metric did not move, change data, not extra **epochs**.

---

## Loss

**Definition:** A number that says how wrong the model’s next-token guesses are on the **training** examples. Lower = closer to copying those examples.

**Why it is used:** To check that training is actually running (the adapter is fitting the homework). It is a debug signal, not proof the product improved.

**Parameters:**
- Computed only on tokens you train (usually assistant tokens)
- Goes down if the model memorizes the train file, even when new prompts fail
- Do not pick the best run by lowest loss alone

**Example:** Loss 1.8 → 0.3 on 80 train rows, while valid-JSON on new tickets stays 40%. Training copied homework; the job did not improve.

---

## Held-out eval set

**Definition:** A fixed file of prompts (and gold answers) the trainer **never** sees. Also called test set or held-out evaluation.

**Why it is used:** So you measure skill on **new** wording, not memory of train rows.

**Parameters:**
- **Size:** ~20 items is enough to start; larger later
- **Split:** write this file before training; do not edit it to make a run look good
- **Overlap:** no copy-paste of train prompts (paraphrase; different entities)
- **Coverage:** same task as train; include hard / refuse / unknown cases

**Example:** Train on 100 tickets. Eval is 20 **other** tickets with the same JSON schema, different wording.

---

## Baseline

**Definition:** The task-metric score of the **original instruct model** on the eval set, **before** your fine-tune, using the same prompt and generation settings.

**Why it is used:** Without it you cannot say fine-tuning helped. A strong prompt on the base model might already be good enough.

**Parameters:**
- Same eval file as after training
- Same chat template
- Same decoding (see below)
- Record this number first; then train

**Example:** Untouched Qwen Instruct: 8/20 eval tickets produce valid JSON. After SFT: 16/20. The baseline is 8/20.

---

## Task metric

**Definition:** A score tied to the **job**, that a fluent wrong answer cannot fake. Not “it sounded nice,” not loss.

**Why it is used:** Language models can be confident and wrong. The metric must fail when the output is unusable.

**Parameters (pick one that matches the job):**
- Structured JSON → % that parse + required keys present
- Field extraction → exact match or F1 vs gold fields
- Text-to-SQL → execution accuracy on a frozen database
- Cite-or-abstain → citation when evidence exists; refusal when it does not

**Example:** Ticket bot metric = `json.loads` succeeds **and** keys `priority`, `product`, `next_action` exist. “Sounds like high priority” is not a metric.

---

## Epoch

**Definition:** One full pass through the training file.

**Why it is used:** You need a stop rule. More epochs on a tiny file is the usual way to memorize.

**Parameters:**
- Small custom data (50–200 rows): **1–3** epochs
- Stop when the **task metric** stops improving (or gets worse)
- If eval is flat after epoch 1, do not go to epoch 8 on the same rows — rewrite data

**Example:** Epoch 1: eval 12/20. Epoch 2: 13/20. Epoch 5: train loss ~0, eval 11/20. Stop at 2; the extra epochs overfit.

---

## Overfitting

**Definition:** The model memorizes training strings (or near-copies) and fails when the user paraphrases or changes an entity.

**Why it matters:** 50–200 examples can teach a format **and** be memorized. Low loss with a flat eval is the usual signature.

**Parameters that make it worse:**
- Many epochs on few rows
- High LoRA rank on tiny data (too much capacity)
- Train and eval that are near-duplicates
- Little variety in templates and entity values

**Example:** Train ticket mentions product `sso`. Eval ticket says `VPN`. Model still outputs `"product": "sso"`. That is overfitting, not a learned skill.

---

## Decoding (generation settings)

**Definition:** How the model picks the next token at **eval time** (not training): randomness, max length, stop tokens.

**Why it is used:** Baseline vs fine-tune must be a fair comparison. Changing decoding between the two runs mixes two different experiments.

**Parameters:**
- **Temperature** / sampling: for schema jobs use greedy or temperature `0` on **both** sides
- **Max new tokens:** same cap
- **Stop / EOS:** same stop strings so replies do not run on
- Chat template: same as training ([page 4](04-what-your-examples-look-like.md))

**Example:** Base at temperature 0.8, SFT at temperature 0.0 is not a valid “fine-tuning helped” claim.

---

## How to read a result

Valid claim: task metric on the locked eval set went from **baseline X** to **SFT Y**.  
Invalid claim: loss went down, therefore the model learned the task.

If Y ≤ X: change the **training data**, not epoch count.

Next: [the path you will actually walk with your own hands](07-your-path-to-doing-this-yourself.md).
