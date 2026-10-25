# The Moving Parts

A llama.cpp install is a folder of
small programs, each doing one job.
You will use four of them constantly
and meet the rest when you need them.
All of them read the same GGUF model
files, and all of them print a small
banner of settings when they start —
when something behaves oddly, that
banner is the first place to look.

## The four you will live in

| Tool | What it does |
|---|---|
| `llama-cli` | Run a model in the terminal: one prompt in, one completion out, or an interactive chat. |
| `llama-server` | Run a model as an HTTP server, so programs (and a built-in web page) can ask it questions. |
| `llama-quantize` | Rewrite a model file at a lower precision, trading a little quality for a much smaller file. |
| `llama-bench` | Measure a model's speed on this machine, in tokens per second, with no chat in the way. |

## Underneath: ggml and backends

The tools share one engine, the
**ggml** tensor library (ggml is the
project llama.cpp grew out of, and
the GG in GGUF). ggml does the
arithmetic; *how* it does it depends
on the **backend** the build was
compiled with:

- **CPU** — always available, the
  subject of this guide's lab. Modern
  CPUs are surprisingly capable at
  this, especially with small models.
- **Metal** — Apple silicon GPUs;
  the default on macOS builds.
- **CUDA** — NVIDIA GPUs.
- **Vulkan** — GPUs from most makers,
  including Intel and AMD.
- **HIP/ROCm, SYCL** — AMD and Intel
  GPU stacks, respectively.

Backends are chosen at *build* time
(chapter 3). One build can carry
several; at run time llama.cpp loads
the model into whichever backends
were compiled in and it is told to
use. The lab built the CPU backend
only, so everything here is measured
on plain CPU; the GPU backends are
documented, not tested, in this
guide, and chapter 13 says so again.

## The supporting cast

You may meet these as you go further;
none is needed for the core chapters.

- `llama-perplexity` — scores how
  surprised a model is by a sample
  text. Lower is better; it is the
  standard way to put a number on
  what a smaller quant lost
  (chapter 6).
- `llama-gguf-split` — splits a large
  GGUF into pieces (or merges them),
  for moving big models on small
  media.
- `llama-imatrix` — builds an
  *importance matrix* from sample
  text, which the finer quantizers
  use to protect the weights that
  matter most.
- `convert_hf_to_gguf.py` — the
  Python script that turns a model
  published in Hugging Face format
  into a GGUF (chapter 9).

## What a run is made of

Every run, whichever tool drives it,
does the same five things:

1. **Load** the GGUF: read its
   metadata (architecture, context
   length, chat template) and map its
   weights into memory.
2. **Tokenize** the prompt: split the
   text into tokens, the model's
   vocabulary pieces — roughly
   three-quarters of a word each, for
   English.
3. **Process the prompt** (often
   labelled *prompt eval* or *pp* in
   the timings): run the whole prompt
   through the model at once,
   building its **KV cache** — the
   model's working memory of the
   conversation so far.
4. **Generate** (labelled *eval* or
   *tg*): predict one token, append
   it, repeat. This is the slow,
   steady drumbeat you watch in the
   timings; one token needs the whole
   model read from memory, which is
   why generation speed is mostly
   about memory bandwidth and model
   size.
5. **Stop** at the requested token
   count, at an end-of-turn marker,
   or when the context fills.

Hold that shape and the rest of the
guide is commentary: quantization
(chapter 6) shrinks step 1 and speeds
step 4; threads and batch sizes
(chapter 8) tune steps 3 and 4; the
server (chapter 7) keeps steps 1–3
warm so many requests can share
them.
