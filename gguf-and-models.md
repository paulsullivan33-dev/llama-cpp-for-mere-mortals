# GGUF and Models

Everything llama.cpp runs arrives as
one file with a `.gguf` ending. This
chapter explains what that file is,
where to get one, and how to tell a
trustworthy file from a random one.

## What GGUF is

**GGUF** (the letters come from its
ancestor, GGML) is a file format
designed for exactly one job: hold
everything needed to run one model,
in one file, loadable by mapping it
into memory. Inside:

- **Metadata** — the model's
  architecture (Llama, Qwen, Mistral,
  and dozens more families), its
  size, the context length it was
  trained for, tokenizer settings,
  and — for chat models — the **chat
  template** that formats a
  conversation into the text the
  model expects.
- **The weights** — the model's
  numbers, stored at a chosen
  precision. That choice is
  quantization, and it is most of
  what decides the file's size
  (chapter 6).

Because the template and tokenizer
travel inside the file, a GGUF is
self-describing in a way raw model
checkpoints are not: download one
file, and `llama-cli` knows how to
talk to it. `llama-gguf` (a small
utility in the build) or the load
banner of any tool will show you the
metadata before you run anything.

## Where models come from

Nearly everything is on **Hugging
Face** (huggingface.co), the public
library where model makers publish.
Three kinds of source, in order of
preference:

1. **The maker's own GGUF repo.**
   Many makers publish GGUFs
   themselves — the repo name ends in
   `-GGUF`. These are the safest
   download: the people who trained
   the model also converted it.
2. **Known converters.** A handful of
   community accounts convert popular
   models to GGUF in every quant,
   often within hours of release.
   Convenient, widely used — and one
   step further from the maker.
3. **Anyone else.** A GGUF is data,
   not a program, and llama.cpp loads
   it as tensors only — but the
   *model itself* is a stranger's
   work: its answers can be biased,
   fine-tuned for odd ends, or simply
   bad. Treat unknown uploads the way
   you would an unknown binary,
   minus the execution risk.

Check: a model repo's page lists its
files with sizes. One model usually
appears as *several* files, one per
quant — you download exactly one
(chapter 6 helps you choose).

## Getting the file

Two honest ways:

```bash
# with a browser or curl, from the repo's file list:
curl -L -o model.gguf \
  https://huggingface.co/<owner>/<repo>/resolve/main/<file>.gguf
```

or let the tools fetch, when built
with curl support:

```bash
llama-cli -hf <owner>/<repo>:<QUANT>
```

The lab used `curl`, deliberately:
the file on disk, named plainly,
can be inspected and reused by every
tool. The lab's model was
**SmolLM2-360M-Instruct**, a small
instruction-tuned model published by
Hugging Face's own team; its Q8_0
GGUF (chapter 6 explains the name)
is 386 MB — small enough to fetch
in a minute and fast enough to make
every experiment in this guide quick.

## Licences are part of the file's story

A model's licence travels with its
repo, not its weights. Some are
permissive (Apache-2.0, MIT); some
community licences allow most uses
but fence off a few; some models are
research-only. SmolLM2 is Apache-2.0.
Whatever you plan for a model —
especially anything public or
commercial — read the licence on its
repo page first. The file will never
remind you.

## Sizes, roughly

Model size is quoted in **parameters**
— 360M, 1.7B, 8B — the count of its
weights. File size is parameters ×
bytes per weight, and bytes per
weight is quantization's whole
subject:

| Model | F16 file | Q8_0 file | Q4_K_M file |
|---|---|---|---|
| 360M | ~0.7 GB | 0.39 GB | 0.27 GB |
| 8B | ~16 GB | ~8.5 GB | ~4.9 GB |

Those are rules of thumb (the lab's
measured files are in chapter 6);
they are how you shop: decide the
biggest model your machine holds
comfortably, then the quant that
fits it into your memory with room
left for the context (chapter 8).
