# 7. Your path to doing this yourself

You now have the picture. This page is the **order of work** — what you will actually do, on Colab, with a small model first.

You do not need a cluster. You need a Hugging Face account, Colab Pro (or similar GPU), patience, and examples you are willing to edit by hand.

## Tools you will use (boring on purpose)

| Piece | What it is in plain language |
|-------|------------------------------|
| **Google Colab** | A notebook in the browser with a rented GPU |
| **Hugging Face** | Where you download models and, later, share your sticky notes |
| **Unsloth + TRL + PEFT** | The usual training stack: load a compressed instruct model, attach sticky notes, run SFT (and later DPO) |
| **Your JSONL file** | The homework packet you wrote |
| **A small scoring script** | Counts valid JSON (or whatever your job is) on the exam file |

You will not write CUDA kernels. You will not train 70-billion-number models. Interviewers and your future self care more about **data, exam, and judgment** than about bragging-size.

**Default family:** Qwen instruct models (good quality, easy license for this kind of work). Llama is a fine backup.

## Stage 0 — Prove you can load a model and chat

Before any teaching:

1. Create a Hugging Face account and a access token.
2. Open one Colab notebook.
3. Load a **small** compressed instruct model (about 1.5B numbers).
4. Send three prompts, get three replies, save them to a file.

When that works, every later idea has a place to run.

## Stage 1 — These notes (you are here)

You should now be able to explain, in your own words:

- next-token guessing vs “it contains my PDF”
- prompting vs lookup vs SFT vs DPO
- sticky notes (LoRA / QLoRA) instead of rewriting the whole intern
- why the exam comes *before* the GPU

Then do the short [before/after look](08-first-look-see-a-change.md) so “an adapter changes behavior” is something you have *seen*.

## Stage 2 — Your first real fine-tune (about a week)

This is the “I actually trained something” checkpoint.

- Start from a small Qwen **instruct** model (1.5B or 3B), QLoRA, mid-size sticky notes.
- Write **50–200 examples yourself** (or heavily edit them) that teach **one** habit the original intern fails — strict JSON for tickets is a perfect first target.
- Write ~20 exam tickets **first**. Score the original intern.
- Train 1–3 passes.
- Score the same exam.
- Try one small comparison: smaller sticky note vs default, or 1 pass vs 3 — **one table**, not twenty.

**Done when:** you can write “the exam went from X to Y.” If it did not move, you change examples, not magic settings.

## Stage 3 — Five short projects (a few days each)

Each folder: problem, data, train, exam table, what failed.

1. **Identity sticky notes** — teach 10 made-up facts or a house voice. You will *see* memorization. That is the point.
2. **Public instruction set** — train a 7B on a *cleaned* slice of a public dataset (thousands of rows). Learn that more data is not automatically better.
3. **Strict JSON / tool format** — the exam is “did it parse?” This is the muscle you want for a portfolio.
4. **One niche with a data-cleaning pipeline** — tickets, SQL, or docs Q&A. Filtering and splitting is most of the job.
5. **DPO on top** — pairs of better vs worse replies, mined from real failures of your SFT model.

Always compare original intern vs your SFT (and SFT vs DPO in the last one). Always keep one automatic score. Always write ten failure cases by hand.

## Stage 4 — One serious project (the resume piece)

Not “I ran a famous tutorial on Llama.” Something with:

- a prompted **baseline** on the same test set
- 7B QLoRA SFT
- DPO on typical failures
- scores that cannot be gamed by sounding nice
- an honest writeup of what got worse
- a small demo (notebook or simple web UI)

Pick **one** domain and stick to it: ticket → JSON, text → SQL, or “cite or abstain.”

## Stage 5 — Package it

A public repo with: problem, method, **results table**, how to reproduce on Colab. A short model card (what data, what it is for, what it is not for). Resume lines that name **scores**, not only tool names.

## Weekly picture (if you have ~8–10 hours/week)

- Weeks 1–2: load a model, finish these notes, first SFT
- Weeks 3–8: the five short projects
- Weeks 9–12: serious project + packaging

Do not skip the tiny “see an adapter change” exercise. It stops you from treating Unsloth as magic.

## What you skip on purpose (for now)

Full reinforcement-learning stacks, multi-GPU server farms, writing GPU kernels, 70B models “for the resume.” None of that teaches the judgment you need first.

## What to do the moment you finish page 8

Come back and say you are ready for **Stage 0**: one Colab notebook that loads a small Qwen instruct model, chats, and saves replies to a file. We build that together. We will not dump every notebook at once.

Previous: [how you know it worked](06-how-you-know-it-worked.md) · Next: [see a change with your own eyes](08-first-look-see-a-change.md)
