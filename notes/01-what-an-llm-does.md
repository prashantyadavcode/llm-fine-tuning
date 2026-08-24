# 1. What an LLM actually does

**LLM** means *large language model* — the family ChatGPT belongs to. You can think of it as **autocomplete that can write whole answers**.

## It only guesses the next bit

Phone autocomplete guesses the next word. An LLM does the same job, but with more practice and a longer memory of “the text so far.”

It works in tiny pieces called **tokens**. A token is usually a word or *part* of a word. The name “ChatGPT” might be two or three tokens, not one.

The loop is always:

1. Look at everything already written (your question, plus what it has said so far).
2. Guess the **next token**.
3. Glue that token on.
4. Repeat until it decides to stop.

A paragraph is not one smart thought. It is **hundreds of tiny guesses** in a row.

That is why the model is excellent at *sounding* like real writing. Sounding right and *being* right are different skills.

## It is not a filing cabinet

People say “the model knows X” as if it stored a spreadsheet.

It did not. During its original training, it saw a huge pile of internet text and got good at continuing that kind of text. Facts that showed up often (Paris is the capital of France) tend to come out correctly. Facts that showed up rarely, or that conflict, get mixed, dropped, or invented with a confident tone.

**Inventing a fluent wrong answer** is normal for this kind of machine. It is not a bug in the same way a crashed app is a bug. The machine’s job is “continue the text,” not “look up the truth.”

## Why this matters for *your* fine-tuning

Suppose you take your company PDF and “train the model on it.”

You did **not** install a search engine inside the model. You only made its next-token guesses a bit more *PDF-like*: similar tone, similar headings, similar phrasing.

It can still:

- mix two policies together
- drop a number
- answer a question that is not in the PDF, with total confidence

If you need answers that match **today’s** wiki, with a quote you can check, looking the wiki up at question time is the honest tool. Teaching habits (always reply in this JSON shape, always this brand voice) is what fine-tuning is for.

**Wrong story:** “We fine-tuned, so now it contains our documents.”  
**Right story:** “We fine-tuned, so it now tends to finish answers the way our examples finished them.”

## Words from this page

| Word | Plain meaning |
|------|----------------|
| **LLM** | A next-token guesser trained on a huge amount of text |
| **Token** | A small chunk of text the model reads and writes |
| **Hallucination** | A fluent guess that is not true |

Next: [using a model vs teaching a model](02-using-vs-teaching.md).
