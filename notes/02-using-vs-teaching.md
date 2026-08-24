# 2. Using a model vs teaching a model

Two activities get mixed up in conversation. They feel similar (“I typed something, the model answered”) but they are not the same.

## Using = chatting (nothing inside the model changes)

When you talk to ChatGPT, or you load a model in a notebook and generate a reply, you are doing **inference**:

- Text goes in.
- Tokens come out.
- The model’s internal numbers stay the same.

It is like talking to a skilled intern. One conversation does not rewire their brain. Tomorrow they still have the same habits.

Almost everything you do in a chat box is this.

## Teaching = training / fine-tuning (something inside *does* change)

**Training** means: show many examples, measure how wrong the next-token guesses are, and slightly change the model’s internal numbers so tomorrow’s guesses move toward your examples.

**Fine-tuning** is training that starts from an *already trained* model. You are not teaching English from zero. You are giving extra lessons.

After fine-tuning, the **same question** should get a **different kind of answer** — more like your examples.

## You never start from a blank brain

The first, huge training run is called **pretraining**. A lab feeds the model a mountain of public text (“guess the next token” for months on many GPUs). That costs millions of dollars.

You download the result: a **checkpoint** (a saved pile of numbers, also called **weights**). That is your starting intern.

Two flavors of checkpoint you will see:

| Kind | What they are good at | What you should use |
|------|------------------------|---------------------|
| **Base** | Continuing stories and articles | Rarely. Awkward as a chatbot. |
| **Instruct** (or “chat”) | Following “the user said this, now reply as the assistant” | **Start here.** Always, unless you have a strong reason not to. |

People name models like `Qwen 2.5 1.5B Instruct`. In plain language:

- **Qwen 2.5** — the family / recipe
- **1.5B** — about 1.5 billion internal numbers. Bigger is not automatically better at *your* small job. Bigger is slower and hungrier for GPU memory.
- **Instruct** — already taught to chat, not just continue novels

Your path starts with a **small instruct model** so experiments are cheap. Later you can use a 7B model for a stronger final project.

## A picture of the stages (you only do the later ones)

```text
pretraining          someone else already did this (you download it)
    ↓
extra chat lessons   already done if you pick an "Instruct" model
    ↓
YOUR fine-tuning     this is the part you will do
    ↓
optional "this reply
is better than that" a later extra polish, not day one
```

You will not pretrain. You will not build a model from random numbers. You download instruct, then fine-tune.

## Words from this page

| Word | Plain meaning |
|------|----------------|
| **Inference** | Use the model. Weights do not change. |
| **Training / fine-tuning** | Teach the model. Some weights change. |
| **Pretraining** | The original expensive “learn language from the internet” run |
| **Checkpoint / weights** | The saved numbers you download |
| **Instruct model** | A checkpoint already taught to follow chat instructions |
| **1.5B / 7B** | Rough size. More numbers ≈ more GPU memory needed |

Next: [four ways to change how the intern behaves](03-four-ways-to-change-behavior.md).
