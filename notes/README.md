# Fine-tuning from scratch

This folder is your starting point. Read it like a short book, **in order**. You do not need to already know the acronyms. Each page introduces words as they appear.

## What you are learning

You want to take an existing chatbot (a smaller cousin of ChatGPT) and **teach it a new habit** using examples you write — on a rented GPU (Google Colab), without a data center.

That extra teaching is called **fine-tuning**.

You are **not** building a model from nothing. Companies already spent millions teaching models English and general chat. You download that finished model, then give it extra lessons for *your* job.

## How to read

Sit with one page at a time. If a later page feels fuzzy, go back one page — do not skip ahead and hope it clicks.

| Order | File | What you will understand |
|-------|------|--------------------------|
| 1 | [01-what-an-llm-does.md](01-what-an-llm-does.md) | A chatbot is a next-word guesser, not a filing cabinet |
| 2 | [02-using-vs-teaching.md](02-using-vs-teaching.md) | Chatting with a model is different from teaching it |
| 3 | [03-four-ways-to-change-behavior.md](03-four-ways-to-change-behavior.md) | When to just ask better, look things up, or actually train |
| 4 | [04-what-your-examples-look-like.md](04-what-your-examples-look-like.md) | The homework you show the model, and why wrapping matters |
| 5 | [05-how-it-fits-on-a-normal-gpu.md](05-how-it-fits-on-a-normal-gpu.md) | Why we add a small “sticky note” of extra weights instead of rewriting the whole model |
| 6 | [06-how-you-know-it-worked.md](06-how-you-know-it-worked.md) | Give it a test *before* training, or you cannot tell if training helped |
| 7 | [07-your-path-to-doing-this-yourself.md](07-your-path-to-doing-this-yourself.md) | The actual steps you will take with your own hands |
| 8 | [08-first-look-see-a-change.md](08-first-look-see-a-change.md) | A 2-hour exercise: same question, before vs after an add-on |

## One running example

Every page uses the same imaginary product, so the ideas stay concrete:

> A user pastes a messy support ticket.  
> The bot must reply with **only** JSON, like:  
> `{"priority": "high", "product": "login", "next_action": "reset_password"}`

That is a *habit* (always this shape, no chatter). Fine-tuning is good at habits. It is a bad way to store a company wiki that changes every week.

## If you remember only this

1. The model guesses the **next small piece of text**, over and over.
2. Fine-tuning makes those guesses more like **your examples**. It does not upload a PDF into a database.
3. Try **better instructions** first. If the facts change, **look them up at question time**. Train only when you need a habit the model still fails.
4. You train a small add-on (not the whole brain) so it fits on Colab.
5. Success is a **test score that went up**, not a training number that went down.

When you finish page 7, you will know *what* to do. Then we start doing it: load a small model, write examples, train, measure.
