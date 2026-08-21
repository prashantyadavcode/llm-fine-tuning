# Start here (read this first)

The other notes use words like **SFT**, **DPO**, and **RAG**. Those are just names for three different ways of changing *how a chatbot behaves*. You do not need them yet. This page builds the picture with one running example, then names the tools.

If you already know PyTorch: nothing here contradicts what you know. An LLM is still a neural net, still a loss, still `loss.backward()`. The new part is *what the data looks like* and *which problem you are solving*.

## 1. What the model is actually doing

ChatGPT-style models do **one** job, over and over:

> Given the text so far, guess the **next small piece** (a *token* — a word or part of a word).

Then they glue that piece on, and guess again. “Writing a paragraph” is hundreds of tiny guesses.

That is why:

- The model is great at **sounding like** text it has seen.
- It is **not** a database. It did not store your PDF as rows you can look up.
- If you “train it on a PDF,” you are only making those next-guesses a bit more *PDF-like*. It can still mix facts, drop numbers, or invent confident nonsense.

**You already know the training loop.** For a classifier you show `(image, label)` and push the predicted class toward the label. For an LLM you show `(text so far, next token)` and push the predicted token toward the real next token. Same idea; longer sequences.

## 2. Using the model vs changing the model

Two different activities get mixed up in conversation:

| | Everyday name | What happens to the weights |
|---|----------------|-----------------------------|
| **Inference** | “Just chat with it” | Nothing. You feed text in, tokens come out. |
| **Training / fine-tuning** | “Teach it” | Gradients update (some) weights so future guesses change. |

Almost everything you do in ChatGPT’s box is inference. Fine-tuning is the extra step: run training so the *same prompt* later gets a *different* kind of answer.

You almost never train from zero. Someone already spent millions of dollars on **pretraining** (next-token on a huge internet-scale pile). You download that **checkpoint** (a saved pile of weights) and only nudge it.

Two kinds of checkpoint you will see:

- **Base model** — good at continuing text (“Once upon a time…”). Awkward as a chatbot.
- **Instruct model** — that base, already taught to follow “User: … / Assistant: …” chat. **Start from instruct** unless you have a reason not to.

## 3. The intern analogy (the whole course in one story)

Imagine you hire a smart intern who has read the public internet (**pretrained instruct model**).

**You can just ask better questions.**  
“Always reply as JSON with keys `priority` and `product`.”  
That is **prompting**. Free. Try it first. If it already works well enough, stop.

**You can hand them today’s tickets folder when they answer.**  
“Here are the three relevant policy snippets; now decide.”  
The intern does not memorize the folder. They *read it at question time*.  
That is **RAG** (retrieval-augmented generation): look something up, paste it into the prompt, then generate. Weights unchanged.

**You can sit with them for a week and show 100 examples of perfect replies.**  
Ticket in → one gold JSON out. After enough copies, they start answering in that shape even without you nagging.  
That is **SFT** (supervised fine-tuning): show *(question, the answer you want)* and train next-token on the answer.

**You can then mark pairs: “this reply is better than that one.”**  
Same ticket, two JSONs: one valid and concise, one rambling or wrong key.  
That is **DPO** (direct preference optimization): you do not give a single gold string; you say *A is better than B*. Useful when they already know the format but keep making the *same class of mistake*.

```text
  just ask ──────────────► prompting          (inference only)
  give notes at ask time ► RAG                (inference only)
  show gold answers ─────► SFT                (training)
  show better vs worse ──► DPO                (training, after SFT)
```

**One-liners to memorize**

| Acronym | Stands for | Plain meaning |
|---------|------------|----------------|
| **SFT** | Supervised Fine-Tuning | “Here is the answer I wanted. Copy this habit.” |
| **DPO** | Direct Preference Optimization | “Of these two answers, prefer this one.” |
| **RAG** | Retrieval-Augmented Generation | “Don’t memorize the wiki. Look it up, then speak.” |
| **LoRA** | Low-Rank Adaptation | Don’t rewrite the whole intern’s brain; clip a small sticky-note of extra weights on top. |
| **QLoRA** | Quantized LoRA | Same sticky-note, but the intern’s original brain is stored compressed (4-bit) so it fits on Colab. |

You can ignore **RL / PPO / GRPO / RLHF** for this path. They are “let the intern try, score the attempt, update.” Later, if ever.

## 4. Running example: a support-ticket JSON bot

Suppose the product goal is:

> User pastes a messy ticket. Model must return **only** JSON:  
> `{"priority": "high"|"medium"|"low", "product": "...", "next_action": "..."}`

Walk the options in the order a sane engineer would:

1. **Prompt it.** System message + one example in the prompt. Measure: what % of replies `json.loads` and have the right keys?  
   If that is already 95%, you may **not** need training.

2. **If the failure is “wrong product name / stale policy”** — the intern cannot know last week’s SKU list. Put the SKU list in a file and **RAG** (or a tool). Training will not stay up to date.

3. **If the failure is “it keeps writing markdown, extra commentary, missing keys”** — the intern does not have the *habit*. Collect 50–200 hand-written `(ticket, gold JSON)` pairs and **SFT**. You are teaching a format/skill, not the entire company wiki.

4. **If after SFT it is valid JSON but wordy, hedges, or picks `medium` when it should `high`** — collect pairs of (good JSON, typical bad JSON) and **DPO**. You are ranking, not teaching JSON from scratch.

That is the whole “SFT vs RAG vs DPO” debate, with no extra theory.

## 5. What “fine-tuning” is allowed to mean

People say “we fine-tuned Llama on our data” to mean five different things. In this repo it means:

- Start from a downloaded instruct model.
- Change **some** weights (almost always a small **LoRA adapter**, not every parameter).
- Using **your** examples (SFT) and maybe **your** better/worse pairs (DPO).
- Then check a **held-out** set of tickets the trainer never saw: did JSON validity actually go up?

It does **not** mean:

- The model now “contains” your Confluence.
- Train loss went down, so the project succeeded.
- You trained 70 billion parameters from scratch.

## 6. A few words you will trip on (glossary)

| Word | Meaning |
|------|---------|
| **Token** | Chunk of text the model sees (often a subword). “ChatGPT” might be 2–3 tokens, not 1 word. |
| **Context / prompt** | Everything already in the window before it guesses the next token: system rules, history, retrieved notes. |
| **Completion / generation** | The new tokens it appends (the “assistant” part). |
| **Checkpoint / weights** | The saved numbers. “Qwen 2.5 1.5B Instruct” is a named checkpoint. |
| **Chat template** | The exact wrapping (`<|im_start|>user` …) that this checkpoint was taught. Wrong wrapping = the intern thinks you are speaking gibberish. |
| **Adapter / LoRA** | A small extra pile of weights you train and can save separately. The big model stays mostly frozen. |
| **Held-out eval** | Tickets you **do not** train on, used as the exam. Always exist before you train, even if only 20 items. |
| **Base vs SFT model** | Base (here: the instruct checkpoint before *your* training) vs the same model after your SFT. Compare them on the same exam. |

## 7. How this maps to the rest of Phase 1

Read in this order. Each later note is the same story with more precision:

1. **This file** — picture and names.
2. [sft-vs-rag-vs-dpo.md](sft-vs-rag-vs-dpo.md) — when to use which; why a PDF is not a database.
3. [lora-qlora.md](lora-qlora.md) — why we train a sticky-note instead of the whole brain, and how it fits on Colab.
4. [chat-templates.md](chat-templates.md) — the wrapping paper around User/Assistant so training matches chat.
5. [eval-before-train.md](eval-before-train.md) — give the intern the exam *before* the week of drilling, or you cannot tell if the week helped.

If a later page feels dense, come back here and re-read section 3–4. The math did not replace the intern story; it only names the knobs.
