# Tokenizers, chat templates, assistant-only loss

Instruct models were SFT’d on a **specific string format** with special tokens. If you train or serve with a different wrapper, you are off-distribution: the model does not “see a user message,” it sees weird text and continues like a raw LM.

## Tokenizer vs chat template

- **Tokenizer:** maps text ↔ token ids. Includes a vocab, BPE/unigram merges, special-token ids, and often a **chat template** (a Jinja string on the tokenizer).
- **Chat template:** the official way to turn `[{role, content}, ...]` into the exact token sequence the model expects.

Always build prompts with:

```python
tokenizer.apply_chat_template(
    messages,
    tokenize=True,
    add_generation_prompt=True,  # inference: end at the assistant prefix
    return_tensors="pt",
)
```

Do **not** hand-concatenate `"User:"` / `"### Instruction"` for a ChatML model. Do **not** copy Llama-3 headers onto Qwen.

`add_generation_prompt=True` at **inference** appends the assistant-side header with no completion yet, so generation starts in the right role. At **training**, you usually pass the full conversation including the gold assistant reply and `add_generation_prompt=False`.

## Qwen 2.5 (ChatML-style)

Qwen instruct checkpoints use an IM-style template. Conceptually:

```text
<|im_start|>system
You are a helpful assistant.<|im_end|>
<|im_start|>user
{user text}<|im_end|>
<|im_start|>assistant
{assistant text}<|im_end|>
```

The exact spaces and newlines are part of the template. One missing `<|im_end|>` is enough to degrade instruction following. That is why you call `apply_chat_template` instead of formatting by eye.

Other families (Llama 3, Gemma, Mistral) have **different** special tokens. Mixing templates is a silent bug: loss still falls, eval looks like “the model is dumb.”

## Multi-turn

History is just more tokens in the same template: system, then user, assistant, user, assistant, … The model does not have a separate “memory API.” If you drop turns, the model never saw them.

For SFT data, prefer **complete** conversations in the native roles. If you only train single-turn `user → assistant`, do not be surprised when multi-turn eval gets worse (the model never practiced closing prior turns).

## Assistant-only loss (mask user tokens)

SFT still uses next-token cross-entropy, but you **do not** want the model to learn to predict the user’s tokens.

Standard trick: set labels to `-100` (ignored by `CrossEntropyLoss`) on every token that is **not** the assistant completion.

```text
tokens:  [sys...] [user question........] [assistant answer........]
labels:  [-100  ] [-100 ................] [answer token ids ......]
```

If you train on the full sequence without masking:

- The model spends capacity modeling **user** text (useless at deploy).
- It can pick up artifacts (always ask a follow-up, mimic user typos).
- Loss numbers look “healthy” while the skill you care about barely moves.

TRL `SFTTrainer` + a proper chat-template dataset typically does this for you **if** you pass conversational data the way the trainer expects (e.g. a `messages` column), not a raw concatenated string you built wrong.

**Check once:** print one tokenized training example and confirm user spans are `-100` and assistant spans are real ids. This is the Phase 2 “I am not cargo-culting Unsloth” check.

## Data you actually write (JSONL mental model)

A single SFT row is a conversation, not a `"prompt"` / `"completion"` pair with a mystery separator:

```json
{
  "messages": [
    {"role": "system", "content": "Reply with a single JSON object. No markdown."},
    {"role": "user", "content": "Ticket: login fails after SSO deploy..."},
    {"role": "assistant", "content": "{\"priority\":\"high\",\"product\":\"sso\",\"next_action\":\"page_oncall\"}"}
  ]
}
```

Keep system prompts **stable** across the dataset unless you are deliberately teaching several modes. If train uses a system prompt and eval does not, you measured a prompt change, not an adapter.

## Inference must match train

| Train | Serve |
|-------|--------|
| Qwen template via `apply_chat_template` | Same tokenizer, same template |
| System prompt in the examples | Same system prompt (or you accept a domain shift) |
| Generation starts after assistant header | `add_generation_prompt=True` |
| Stop at `<|im_end|>` / EOS | Configure `eos_token_id` / stop strings or you get run-on roles |

Merging LoRA does not change the template. Shipping GGUF/Ollama still requires the **same** chat template in the runner config.

## Common failure modes

1. **Base model + instruct template** or **instruct model + raw completion format.**
2. **Training with template A, evaluating by pasting text into a ChatGPT-like UI** that applies template B.
3. **Loss on full sequence** (user tokens not masked).
4. **Special tokens truncated** by `max_seq_length` — the assistant header falls off, labels become nonsense. Print length histograms before training.
5. **JSON in markdown fences in train, then scoring raw JSON at eval** — you trained a different format than you grade.

## Interview answers (short)

**“What is a chat template?”**  
The canonical serialization of roles into special tokens. It is part of the **tokenizer card**, not a prompt-engineering preference.

**“Why mask user tokens?”**  
Causal LM loss on the whole string would teach the model to imitate the user. We only want to shift **assistant** next-token distributions.

**“Why does my fine-tune ignore instructions?”**  
First suspect is template mismatch or truncation, not rank. Verify one decoded training string against the model card.
