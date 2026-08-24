# 3. Four ways to change how the intern behaves

Imagine you hired a smart intern who has read a lot of the public internet. That intern is your downloaded **instruct model**.

You want them to turn messy support tickets into one strict JSON object. There are four tools. Only two of them are “fine-tuning.” Most people jump to training too early.

## Tool 1 — Just ask better (prompting)

You write clearer instructions:

> Always reply with JSON keys `priority`, `product`, and `next_action`. No markdown. No extra sentences.

That is **prompting**. It costs nothing extra. The intern’s brain does not change. You are just being a better manager.

**Try this first.** If 19 out of 20 replies already parse as valid JSON, you may not need training at all.

## Tool 2 — Hand them today’s folder when they answer (RAG)

The intern cannot know last week’s product names or a policy that changed this morning. Those are **facts that move**.

So at question time you:

1. Search your files for the relevant snippets.
2. Paste those snippets into the prompt.
3. Ask the intern to answer using that text.

The intern does not memorize the folder. They **read it while answering**.

This is called **RAG** (*retrieval-augmented generation*). Ugly name. Plain meaning: **look it up, then speak**.

Weights stay frozen. When the wiki changes, you update the files, not the model.

Use this when:

- facts go stale (prices, policies, ticket history, API docs)
- you need a citation (“this came from page 4”)
- the knowledge pile is huge and not worth stuffing into training examples

RAG does **not** reliably teach a new output shape or a house voice. The intern can still ignore the notes and ramble.

## Tool 3 — Sit with them and show perfect replies (SFT)

You collect 50–200 examples:

> Here is a ticket. Here is the **exact** JSON I wanted.

You train so the intern’s next-token guesses copy those gold answers. After enough copies, they start producing that shape even when you nag less.

This is **SFT** (*supervised fine-tuning*). Ugly name. Plain meaning: **here is the answer I wanted — copy this habit.**

This is what most people mean by “fine-tune.”

Use this when the instruct model **fails a behavior you can demonstrate**:

- always this JSON shape, no chatter
- a specific “call this tool like this” format
- a house voice or a “say you don’t know” policy
- a narrow skill (labels for *your* ticket types) where prompting is flaky

Do **not** use this as your first idea for:

- “Make it know Q3 headcount” → look it up (RAG) or call a tool
- “Just make it output JSON” if a strict prompt already hits your success rate

## Tool 4 — Mark “this reply is better than that one” (DPO)

After SFT, the intern can speak JSON, but they still:

- ramble
- pick `medium` when it should be `high`
- hedge instead of deciding

Now you show **pairs**: same ticket, one good JSON, one typical bad JSON. You train “prefer the good one.”

This is **DPO** (*direct preference optimization*). Plain meaning: **of these two answers, like this one more.**

Do this **after** SFT, not instead of it. DPO is polish. It does not teach a brand-new format from scratch.

You can ignore heavier cousins (RL, PPO, RLHF) for a long time. Those are “let the intern try, score the attempt, update.” Later, if ever.

## The whole debate on one diagram

```text
just ask better              prompting     (no training)
give notes at ask time       RAG           (no training)
show gold answers            SFT           (training)
show better vs worse         DPO           (training, after SFT)
```

**Facts → look up. Habits → train.**  
Mixing those two jobs is why “should I fine-tune?” feels confusing.

## Walk the ticket bot in a sane order

1. **Prompt it.** Measure: what percent of replies are valid JSON with the right keys? If that is already high, stop.
2. **If the mistake is wrong product name or stale policy** — the intern cannot know last week’s catalog. Put the catalog in a file and look it up (RAG), or call a tool. Training will go stale.
3. **If the mistake is markdown, extra commentary, missing keys** — they do not have the *habit*. Write 50–200 `(ticket, gold JSON)` pairs and do SFT.
4. **If after SFT the JSON is valid but wordy or weakly labeled** — collect good-vs-bad pairs and do DPO.

## They can stack

A real product often uses more than one:

1. RAG puts today’s evidence in the prompt.
2. SFT teaches “answer only from that evidence, emit this JSON, refuse if empty.”
3. DPO prefers grounded, valid JSON over fluent guessing.

You still measure each claim separately: did lookup find the right snippet? did the JSON parse? did people prefer the new replies?

## Words from this page

| Short name | Stands for | Plain meaning |
|------------|------------|----------------|
| **Prompting** | — | Better instructions. No training. |
| **RAG** | Retrieval-augmented generation | Look something up, paste it in, then generate |
| **SFT** | Supervised fine-tuning | Show the answer you wanted. Copy that habit. |
| **DPO** | Direct preference optimization | Prefer reply A over reply B |

Next: [what your training examples actually look like](04-what-your-examples-look-like.md).
