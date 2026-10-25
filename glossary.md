# Glossary

- **Backend** — the part of ggml that
  runs the arithmetic on a kind of
  hardware: CPU, CUDA, Metal, Vulkan.
  Chosen when llama.cpp is built.
- **Batch** — a group of tokens
  processed together. Bigger batches
  make prompt processing faster, up
  to a point.
- **Chat template** — the exact text
  format a chat model was trained
  with: where the system prompt
  goes, how turns are marked. Stored
  inside the GGUF.
- **Completion** — text a model
  generates continuing a prompt.
  Also the server's raw endpoint
  (`/completion`) as opposed to the
  chat endpoint.
- **Context (context length, `-c`)**
  — how many tokens the model can
  hold in view at once: prompt plus
  reply, past and present.
- **Eval / generation (`tg`)** — the
  one-token-at-a-time phase after
  the prompt is processed.
- **GGUF** — the single-file model
  format llama.cpp loads: metadata,
  tokenizer, template and weights
  in one package.
- **imatrix (importance matrix)** —
  statistics gathered from sample
  text that steer the finest quants
  toward protecting important
  weights.
- **Inference** — running a trained
  model to produce output, as
  opposed to training it.
- **KV cache** — the model's working
  memory of the tokens so far; grows
  with the context, and is why
  context costs memory.
- **Layer offload (`-ngl`)** — how
  many of the model's layers are
  placed on the GPU instead of the
  CPU.
- **Perplexity (PPL)** — a score of
  how surprised a model is by a
  text it should predict well. Lower
  is better; used to price what a
  quant lost.
- **Prompt processing (`pp`)** — the
  phase that digests the whole
  prompt at once, before generation
  begins.
- **Quant / quantization** — storing
  weights at lower precision (fewer
  bits per weight) to shrink the
  file and speed generation, at
  some quality cost. *Q4_K_M* and
  friends are quant types.
- **Sampling** — how the next token
  is chosen from the model's scored
  candidates: temperature, top-k,
  top-p and friends shape it.
- **Slot** — one of the server's
  parallel conversation seats, each
  with its own KV cache.
- **Tensor** — an array of numbers;
  a model's weights are a pile of
  named tensors.
- **Token** — a piece of text (part
  of a word, usually) — the unit
  models read and write. Speeds are
  quoted in tokens per second
  (tok/s).
- **Weights** — the model's learned
  numbers; the bulk of every GGUF
  file, and the thing quantization
  shrinks.
