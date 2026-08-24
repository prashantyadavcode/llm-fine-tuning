# 8. First look: see a change with your own eyes

No long training. Goal: the **same question**, original intern vs intern-plus-someone-else’s sticky notes, so “adapters change behavior” is something you have watched.

Do this after you can load a small compressed Qwen in Colab (Stage 0). If that is not ready yet, treat this page as a checklist and come back.

## Setup

- Load a **small instruct** model you can fit in 4-bit on Colab (for example Qwen 2.5 1.5B Instruct).
- Find a **public adapter** (sticky-note file) trained for **that same** model family. Compatibility matters more than fame. If the adapter was trained on a different intern, nothing sensible will happen.
- Freeze how you generate: no randomness (or temperature 0), same length limit, **same wrapping**.

## Steps

1. Write **5 questions**: 3 that match whatever the adapter claims to be good at (a style, a language, JSON, …) and **2 unrelated** questions.
2. Generate with the **original intern only**. Save the raw replies to a file.
3. Load the **same intern + adapter**. Generate again. Save.
4. Put them side by side. What changed — format? tone? facts? Did the unrelated questions get worse or start sounding like the adapter’s voice?

## What you should notice

- In-domain questions often shift **style or shape** more than “IQ.”
- Unrelated questions may stay the same, get a bit worse, or pick up the adapter’s accent (habit bleed).
- If **nothing** changes, you probably snapped sticky notes onto the wrong intern, forgot to turn the adapter on, or used different wrapping than the adapter was trained with.

## Write three bullets somewhere

- Which intern + which adapter
- One before/after pair where the change is obvious
- One failure or unrelated-question regression

That is the whole lab. Next time **you** write the examples and train the sticky notes.

When you are ready, we start Stage 0: load, chat, save. Then Stage 2: your first SFT.

Back to the [map](README.md).
