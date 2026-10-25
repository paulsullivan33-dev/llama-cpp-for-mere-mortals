# llama.cpp for Mere Mortals

A language model does not have to live
in someone else's data centre. **llama.cpp**
is the open-source program that runs
language models — the text-predicting
kind of AI — on ordinary computers: a
laptop with no graphics card, a small
single-board computer, a desktop with a
gaming GPU. You build it once, download
one model file, and the model answers on
your machine, offline, at nobody's
per-token price. This guide explains how
it works in plain terms: building the
tools from source, running a first
completion, the GGUF model files and
where they come from, quantization — the
trick that shrinks models onto small
machines — the built-in HTTP server, and
the limits, stated honestly.

The lab behind this guide ran the real
thing: **llama.cpp built from source**
(commit `d327bf1`) on a small Linux
machine — 2 CPUs, 7.7 GB of RAM, no GPU —
and a real model, **SmolLM2-360M-Instruct**,
downloaded from its makers' own Hugging
Face repo. The tools were exercised end
to end: completions in the terminal with
their speeds recorded, the model
re-quantized three ways with every file
size measured, perplexity scored per
quant on a sample text written for the
guide, the server answering real HTTP
requests (`/health`, `/completion`, the
OpenAI-shaped chat endpoint, even its
built-in web page), and `llama-bench`
runs across thread counts and quants.
The lab's surprises are in the guide, not
smoothed over: this model's K-quants
barely shrank (most of its tensors fell
back to coarser types), its middle quant
benched *slower* than its biggest file,
and extra threads made generation worse,
not better, on a shared machine.

What the lab could not do, it did not
fake. There was **no GPU**, so every GPU
backend — CUDA, Metal, Vulkan, HIP,
SYCL — and GPU layer offload are taught
from the documentation and labelled as
such wherever they appear. **Converting**
a model from its original Hugging Face
form is likewise a documented chapter:
the recipe is the project's own, the
multi-gigabyte Python install was not
made. A systemd unit for the server was
written and passed `systemd-analyze
verify`, but actually starting it under
systemd is documented only — the lab
machine runs no user session manager.
`criteria.md` keeps the full ledger of
what was run and what was only read.

## What this guide covers

- [**What llama.cpp is**](what-llama-cpp-is.md) —
  the mental model: inference on your own
  hardware, the three pieces (tools, model
  file, settings), and what it is not — a
  trainer, a chat product, a way to make a
  small model smart
- [**The moving parts**](the-moving-parts.md) —
  `llama-cli`, `llama-server`,
  `llama-quantize`, `llama-bench`, the ggml
  engine and its backends, and the five
  steps every run is made of
- [**Building llama.cpp**](building-llama-cpp.md) —
  the clone-and-compile recipe, what you
  need first, the GPU backend flags
  (documented, not tested), and keeping
  the build current
- [**Your first completion**](your-first-completion.md) —
  one prompt, one answer, the timing line
  decoded; then a real conversation, its
  in-session commands, and the honest
  paragraph about fluency without grounding
- [**GGUF and models**](gguf-and-models.md) —
  the one-file format: metadata, weights,
  the chat template inside; where models
  come from, who to trust, licences, and
  shopping by size
- [**Quantization**](quantization.md) —
  fewer bits per weight: reading a name
  like Q4_K_M, the family table, the lab's
  measured sizes, perplexity and outputs
  per quant, the requantize refusal, and
  choosing one
- [**The server**](the-server.md) — one
  model answering HTTP: slots and context,
  `/health` and its honest 503, the raw
  `/completion` door, the OpenAI-shaped
  chat door, the built-in web page, and
  the no-authentication warning
- [**Tuning and performance**](tuning-and-performance.md) —
  measure first with `llama-bench`;
  threads, context, batch and sampling in
  the order to reach for them, with the
  lab's bench table including the parts
  where more was worse
- [**Converting models**](converting-models.md) —
  how a Hugging Face model becomes a GGUF
  (documented, not lab-tested), F16 first,
  fine-tunes and LoRA adapters, and
  checking a conversion
- [**Running as a service**](running-as-a-service.md) —
  the server under systemd: a user unit
  (verified as text, not started), the two
  settings that matter, lingering, logs
  and care
- [**Recipes**](recipes.md) — a private
  writing assistant, summarising from the
  shell, an endpoint for scripts, benching
  five quants, and moving a model between
  machines by copying one file
- [**The neighbours**](the-neighbours.md) —
  Ollama, LM Studio and the other front
  ends built on llama.cpp; vLLM, PyTorch
  and MLX as different engines; which to
  reach for when
- [**Limits**](limits.md) — the model is
  the ceiling, memory is the wall, CPU
  generation is bandwidth-bound, a handful
  of seats is not a service, and no safety
  net is built in
- [**Troubleshooting**](troubleshooting.md) —
  the model that will not load, absurd
  slowness, gibberish and loops, the
  context error, server puzzles, quantize
  refusals and build failures —
  symptom → cause → fix
- [**Glossary**](glossary.md) — the terms,
  collected in one place

## What it doesn't cover

Training or fine-tuning models (llama.cpp
runs them; others make them), the Python
ecosystem around model publishing, and
serving models to many users at company
scale — the batching engines in chapter 12
own that job. Model *quality* rankings are
not attempted: which 7B chats best is a
moving target this guide deliberately
doesn't chase. Exposing the server to the
internet gets a warning, not a tutorial.

## How to use this guide

Read chapter 1 for the model and chapter 2
before building anything — knowing which
piece holds what turns later confusion
into a named problem. Chapter 3 builds the
tools and chapter 4 runs a model in the
same sitting. Chapter 6 is the one not to
skip: quantization decides what fits, what
it costs, and — the lab's bench insists —
not always what runs fastest, so measure
before you commit. Chapter 10 is the other
one to read before you need it: the
localhost binding is the server's whole
security model. The lab's transcripts live
in `llama-lab/transcripts/`; its notes are
`examples/lab-notes.txt`.

---

## Part of the series

This guide is one of the [For Mere Mortals](https://paulsullivan33-dev.github.io/mere-mortals/) guides: plain-language guides to the tools programmers and makers use.

Related guides:

- [Ollama for Mere Mortals](https://paulsullivan33-dev.github.io/ollama-for-mere-mortals/)
- [Local AI at Home for Mere Mortals](https://paulsullivan33-dev.github.io/local-ai-for-mere-mortals/)
- [Open WebUI for Mere Mortals](https://paulsullivan33-dev.github.io/open-webui-for-mere-mortals/)

## License

The writing in this guide is licensed under [CC BY 4.0](https://github.com/paulsullivan33-dev/llama-cpp-for-mere-mortals/blob/main/LICENSE). Code examples and lab scripts are licensed under the [MIT License](https://github.com/paulsullivan33-dev/llama-cpp-for-mere-mortals/blob/main/LICENSE-CODE).
