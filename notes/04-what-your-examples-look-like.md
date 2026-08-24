# 4. What your examples look like

Fine-tuning is not “upload a folder of PDFs.” It is “show conversations where the assistant already did the job perfectly.”

If the examples are sloppy, the intern copies sloppy. This page is about the homework packet.

## One example is a short chat, not a mystery string

Each training row looks like a tiny conversation with roles:

- **system** — standing orders (“Reply with one JSON object. No markdown.”)
- **user** — the ticket / question
- **assistant** — the gold reply you wish the model had given

In a file (one JSON object per line), that looks like:

```json
{
  "messages": [
    {"role": "system", "content": "Reply with a single JSON object. No markdown."},
    {"role": "user", "content": "Ticket: login fails after the SSO deploy. Whole team blocked."},
    {"role": "assistant", "content": "{\"priority\":\"high\",\"product\":\"sso\",\"next_action\":\"page_oncall\"}"}
  ]
}
```

You will write dozens to hundreds of these. Quality beats quantity at the start. Fifty *careful* rows beat five thousand noisy ones.

Keep the system message **the same** across almost every example, unless you are deliberately teaching several modes. If you train with a system message and later test without it, you measured a prompt change, not a smarter model.

## Wrapping paper: the chat template

Every instruct model was taught a **specific wrapping** around those roles. Special marker tokens mean “the user is speaking now” vs “the assistant is speaking now.”

For one popular family (Qwen), it looks conceptually like:

```text
<|im_start|>system
Reply with a single JSON object.<|im_end|>
<|im_start|>user
Ticket: login fails...<|im_end|>
<|im_start|>assistant
{"priority":"high",...}<|im_end|>
```

That wrapping is the **chat template**. It lives with the model’s tokenizer (the tool that turns text into tokens). You should not invent `"User:"` / `"### Instruction"` by hand for a model that never saw that format.

Wrong wrapping is like using the wrong secret handshake. The intern hears gibberish and continues like a raw autocomplete. Training loss can still “look healthy” while the skill you care about barely moves.

**Rule:** train and later *use* the model with the **same** official wrapping. Libraries have a helper (`apply_chat_template`) so you do not format by eye.

Spaces and end-markers matter. One missing end marker is enough to make instruction-following worse. Do not mix Llama wrapping onto a Qwen model.

## Grade the intern’s answers, not the customer’s email

During training the computer scores “how wrong was the next-token guess?”

You only want to score the **assistant** part. If you also score the user’s ticket, the model spends effort learning to *imitate the customer* (typos, rambling, always asking a follow-up). Useless when you deploy.

The standard trick: tell the loss function to **ignore** user and system tokens, and only learn from assistant tokens.

Good training libraries do this if you pass data as `messages` the way they expect. Once, print one training example and confirm: user text is ignored, assistant text is what gets learned. That single check saves a week of “why is my fine-tune dumb?”

## Train and serve must match

| While teaching | While using later |
|----------------|-------------------|
| Official wrapping for that model | Same wrapping |
| Same system message | Same system message |
| Gold assistant reply is in the example | You start generation at “now the assistant speaks” and stop at the end marker |

If you train in a notebook with the official template, then paste text into a random chat UI that wraps differently, you did not test your fine-tune. You tested a different handshake.

## Common ways people silently mess this up

1. Using a **base** (story-completer) model with a chat wrapping, or an instruct model with raw “just complete this” text.
2. Training with wrapping A, evaluating in an app that applies wrapping B.
3. Scoring the whole conversation, including the user’s words.
4. Cutting examples off in the middle because they are too long — the “assistant is about to speak” marker falls off the end.
5. Training on JSON inside markdown fences (```json ...```), then grading *bare* JSON. You trained a different habit than you test.

## Words from this page

| Word | Plain meaning |
|------|----------------|
| **Chat template** | The official wrapping around system / user / assistant |
| **Tokenizer** | Turns text into token ids and back; owns the wrapping recipe |
| **Assistant-only loss** | Only grade the intern’s reply, not the user’s question |
| **JSONL** | A file with one JSON example per line — how you will store data |

Next: [how this training fits on a normal GPU](05-how-it-fits-on-a-normal-gpu.md).
