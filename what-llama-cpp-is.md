# What llama.cpp Is

**llama.cpp** is a program that runs
**language models** — the kind of AI
model that predicts text, one piece
(called a **token**) at a time — on
ordinary computers. It is written in
C and C++, needs no Python and no
cloud account, and is built to make
do with the machine in front of you:
a laptop with no graphics card, a
small single-board computer, a desktop
with a gaming GPU. If the machine can
hold the model file in memory, it can
probably run it — slowly or quickly
depending on the hardware, but it runs.

A language model on its own is just a
very large set of numbers, called
**weights**, learned from training
text. Running it — feeding in a prompt
and generating a reply — is called
**inference**. Inference is mostly
arithmetic, and the trick of llama.cpp
is doing that arithmetic efficiently
in plain C++ on hardware that was
never built for AI, starting with the
CPU you already have.

## The shape of the thing

Working with llama.cpp has three
pieces, and this guide takes them in
that order:

- **The tools.** A handful of small
  programs you build (or download)
  once: `llama-cli` for running a
  model in the terminal, `llama-server`
  for serving one over HTTP,
  `llama-quantize` for shrinking model
  files, and a few more (chapter 2).
- **The model file.** One file in the
  **GGUF** format holds everything a
  model needs (chapter 5). You
  download it, point a tool at it, and
  it runs. No installation step, no
  conversion, for pre-made files.
- **The settings.** How much of the
  conversation the model can hold in
  mind (the **context**), how many
  processor threads to use, how
  random to be. Small numbers, large
  effects (chapter 8).

## Why people reach for it

- **It runs anywhere.** The same
  project runs on Windows, macOS and
  Linux; on Intel, AMD and ARM; on
  CPUs alone or with help from a GPU
  (NVIDIA, AMD, Apple silicon and
  others). A model file that runs on
  one machine runs on all of them.
- **It is frugal.** Through
  **quantization** — storing the
  weights in fewer bits (chapter 6) —
  models that would need tens of
  gigabytes shrink to files a laptop
  can hold, with a quality cost that
  is real but often small.
- **It is yours.** Nothing phones
  home. A model on your disk answers
  prompts on your machine, offline,
  at nobody's per-token price. Whatever
  you type stays where you typed it.

## What it is not

llama.cpp does not *train* models; it
runs models other people trained and
published. It is not a chat product
with accounts and a polished window —
the terminal tool is plain, and the
server's built-in web page is a
utility, not a product (chapter 12
covers the friendlier front ends
others build on top). And it does not
make a small model smart: a model
that fits on a phone answers like a
model that fits on a phone. Chapter
13 is honest about the rest of the
limits.

Check: by the end of chapter 4 you
will have built the tools, downloaded
a real model file of a few hundred
megabytes, and watched it complete a
prompt at a speed measured on the
lab machine — no GPU involved.

## Who it is for

Anyone who wants to run a language
model on hardware they own and
understand what is happening when
they do: the home-lab keeper curious
what the fuss is about, the developer
who wants a local model to test
against, the privacy-minded who would
rather prompts stayed on their own
disk. The guide assumes you can open
a terminal and follow a recipe; it
assumes nothing about GPUs, budgets,
or prior AI experience.
