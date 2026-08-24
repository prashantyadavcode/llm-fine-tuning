# 5. How this fits on a normal GPU

A 7-billion-number model is huge. If you try to update **every** number the way textbooks describe “train a neural net,” you need far more GPU memory than Colab gives you. The optimizer (the math that applies the updates) also keeps extra copies of numbers for each knob.

So this path never “full fine-tunes” a 7B model. You add a small extra piece and train **that**.

## Sticky notes on a cookbook (LoRA)

Think of the downloaded model as a giant cookbook you bought. You do not rewrite every recipe.

You stick a few **sticky notes** on the pages that matter. When you cook, you read the original page *plus* the sticky note.

That add-on is **LoRA** (*low-rank adaptation*). Plain meaning: **train a small patch, leave the original book mostly frozen.**

Why this works well enough for your goals:

- Style, JSON shape, and tool format are *habits*. A small patch can change habits.
- You can save just the sticky notes (tiny file) and snap them onto the original model later.
- At step zero the sticky notes are blank, so the first batch of training does not instantly wreck the intern.

**Rank** is “how much room is on the sticky note.”

- Too little room: the intern cannot pick up the new habit. The test score barely moves.
- Too much room on only 80 nearly identical examples: the intern memorizes exact wording. A paraphrased ticket fails.
- More room is **not** more intelligence. The examples decide *what* changes. Rank only decides *how much capacity* you have to change.

A practical default on this path: start with a mid-size sticky note (people call this `r=16`), and later try a smaller one (`r=8`) to see if you still get the same test score.

You can **merge** the sticky notes into the cookbook when you want one single model to serve. The wrapping (chat template) does not change when you merge.

## Cheap photocopy plus sticky notes (QLoRA)

Even *storing* the full cookbook in high quality can crowd a Colab GPU.

**QLoRA** means:

1. Store the original cookbook as a **cheap compressed copy** (4-bit — fewer shades of each number).
2. Keep the sticky notes in higher quality.
3. Only the sticky notes get updated.

Plain meaning: **compressed frozen intern + small trainable patch.** That is why 7B–14B extra lessons fit on one Colab Pro GPU.

You will use a library called **Unsloth** (and Hugging Face tools under it). Unsloth is a faster, more memory-friendly way to do this same idea. You do not need to understand the speed tricks. The mental model stays: frozen compressed base, trainable sticky notes.

## What you will actually set (later, in a notebook)

You do not need these numbers memorized today. You need to know they exist so the notebook is not magic.

| Knob | Beginner instinct |
|------|-------------------|
| Which model | An **instruct** checkpoint, small first (1.5B or 3B), 7B later |
| How it is stored | 4-bit compressed base + LoRA (QLoRA) |
| Sticky-note size (`r`) | 16, then try 8 |
| How many times through the data (**epochs**) | 1–3 on small custom data. Stop when the **test score** stops improving |
| How long each example can be | Long enough for your real tickets; cutting them off breaks wrapping |

If the test score is flat after one pass, **rewrite examples**. Do not run eight more passes over the same 80 rows. That is how you memorize.

## Sticky notes are not a forgetting vaccine

Because you only patch a small part of the intern, you damage general chat *less* than rewriting the whole brain. You can still damage it.

A narrow JSON-only week of drilling can make the intern worse at ordinary questions, or make every answer sound like your ticket template. That is **catastrophic forgetting** in plain clothes: they got great at the new job and rusty at the old ones.

Cheap insurance: keep a handful of ordinary chat questions as a second mini-test. If JSON got better but the intern now ignores “answer in one sentence” on unrelated prompts, you should know that — even if you accept the trade.

Tiny datasets (50–200 rows) can also **memorize**. The intern recites a training answer for a *similar* but not identical ticket. Your test questions must not be copy-pastes of training questions with one word changed.

## Words from this page

| Word | Plain meaning |
|------|----------------|
| **LoRA** | Small trainable sticky notes on a frozen model |
| **QLoRA** | Same sticky notes, original model stored compressed so it fits |
| **Rank (`r`)** | How much room is on the sticky note |
| **Adapter** | The sticky-note file you train and can share |
| **Epoch** | One full pass through your example file |
| **Unsloth** | A popular, efficient way to run QLoRA on Colab |

Next: [how you know training actually helped](06-how-you-know-it-worked.md).
