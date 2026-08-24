# 6. How you know it worked

Training software will print a number called **loss**. When loss goes down, it means: “the sticky notes are getting better at reproducing the *training* answers.”

That is not the same as: “the product got better.”

The intern can memorize the study packet and still fail new tickets. You need an **exam the intern did not study**.

## The loop (do this in this order, every time)

```text
1. Write the exam and the scoring rule     ← before any GPU
2. Score the untouched instruct model      ← the baseline
3. Train
4. Score the SAME exam
5. If the score did not move → change the study packet,
   not “train longer”
```

Step 2 is the one people skip. Without it you cannot say fine-tuning helped. A strict prompt on the original intern might already have been good enough.

## Score the job, not “it sounded nice”

Pick a score a skeptic cannot game with fluent nonsense.

For the ticket bot, a simple score is:

> Percent of replies that parse as JSON **and** have the keys `priority`, `product`, `next_action`.

“Sounds like a reasonable priority” is not a score. `json.loads` either works or it does not.

Other jobs have their own honest scores:

| Job | Honest score |
|-----|----------------|
| Strict JSON | Percent valid + required keys present |
| Copy fields out of text | Did this field match the gold value? |
| Text → SQL | Run the SQL on a frozen database; did the result match? |
| “Cite or say you don’t know” | Citation when evidence exists; refusal when it does not |

At the beginning, **about 20 exam tickets** is enough to learn from, as long as you do not cheat. It is a signal, not a scientific paper. You still need the number so you cannot lie to yourself.

Also write **10 failures by hand** after you look at outputs. That paragraph will teach you more than a pretty training graph.

## The exam must be a real exam

- **Paraphrase.** Same intent, different wording than training.
- **Same job.** Do not train “JSON tickets” and test “write a poem.”
- **No leakage.** If a test ticket is almost a training ticket, you measured memory, not skill.
- **Include “should refuse” cases.** Tickets that should say unknown, or omit a field. A score that only counts happy-path JSON will hide collapse.

Lock the exam file **before** you start tweaking training. If you keep editing the exam to make your new run look good, the exam is no longer an exam.

## Signs you memorized the study packet

You are using 50–200 hand-written examples. That is enough to teach a format. It is also enough to memorize.

Watch for:

- Training loss near zero, exam score flat or worse
- The intern recites a training answer for a similar but not identical ticket
- You change one product name in the ticket and still get the old product name out

Fixes that are not “train longer”:

- More variety in wording and in the entities (products, names)
- Fewer passes (1–2), stop when the **exam** stops improving
- A smaller sticky note if you are clearly memorizing
- Add paraphrases to **training** only after the exam file is locked

**If the exam did not move, iterate on data, not epochs.** That sentence is the whole beginner superpower.

## Fair fights only

When you compare “before” and “after”:

- Same exam file
- Same wrapping
- Same generation settings (how random, how long, when to stop)

If the original intern answers at “creative and random” and your fine-tune answers at “always pick the most likely token,” you did not compare training. You compared two different ways of talking. For JSON jobs, prefer boring, low-randomness generation on both sides.

## What you are allowed to claim

You may say: “Loss went down, **and** valid-JSON on 20 held-out tickets went from 40% to 90%.”

You may not say: “Loss went down, therefore the model learned the task.”

If the exam did not improve, the honest writeup says so, then you show the data change you try next. That honesty is part of learning this for real.

## Words from this page

| Word | Plain meaning |
|------|----------------|
| **Loss** | “How well am I copying the training answers?” — debugging only |
| **Held-out eval** | An exam the trainer never saw |
| **Baseline** | The original instruct model, scored on that exam *before* you train |
| **Task metric** | A score tied to the job (valid JSON %), not a vibe |
| **Overfitting** | Memorized the study packet; fails new wording |

Next: [the path you will actually walk with your own hands](07-your-path-to-doing-this-yourself.md).
