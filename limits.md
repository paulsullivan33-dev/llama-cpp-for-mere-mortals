# Limits

What llama.cpp cannot do, and what
running models locally cannot do,
stated plainly.

## The model is the ceiling

llama.cpp runs the model you give
it, exactly as well as that model
can be run. It cannot make a 360M
model reason like a frontier model,
write like one, or know what it
does not contain. Small models are
genuinely useful — drafting,
summarising, classifying, chatting —
and genuinely limited: they lose
threads, invent facts with a
straight face, and fail at
multi-step reasoning in ways that
are not always obvious. A wrong
answer from a local model costs the
same as a wrong answer from a big
one. Verify anything that matters.

## Memory is the wall

A model runs at the size of its
file, plus its context (chapter 8),
plus overhead. When the total
exceeds memory, the machine swaps
to disk and generation slows from
tokens per second to tokens per
minute. The practical ceiling on a
machine is therefore not its CPU
but its RAM, and for GPU offloading
its VRAM. Quantization (chapter 6)
moves the wall; it does not remove
it.

## CPU generation is bandwidth-bound

On CPU, each generated token means
reading essentially the whole model
from memory once. That is why a
model half the size generates
roughly twice as fast (chapter 6's
measured table shows the shape of
this), why more threads stop helping
past a point, and why a big model on
CPU is a patience exercise: an 8B
model at a large quant generates in
the single digits of tokens per
second on a modest machine. Prompt
*processing* parallelises well;
generation mostly does not.

## One model, so many seats

`llama-server` shares one loaded
model across a small number of
parallel **slots** (chapter 7). It
is a household server, not a fleet
service: a handful of simultaneous
users on hardware that can feed
them. Serving many strangers fast
is the job of the batching engines
in chapter 12, on hardware with
far more memory bandwidth than a
desktop has.

## No safety net is built in

The model will comply with almost
any prompt, and so will the tools:
there is no account, no filter, no
refusal layer you did not bring
yourself. That freedom is the point
of local inference for many users.
It also means the judgement about
what to generate, and what to do
with it, is entirely yours. And
chapter 10's warning stands: the
server has no real authentication;
do not expose it to a network you
do not trust.

## Platform edges

- **New architectures arrive as
  code.** A model family released
  last week may need this week's
  build (chapter 3's update recipe).
- **Exotic hardware is best
  effort.** Unusual CPUs, old GPUs
  and brand-new accelerators are
  where builds fail and backends
  lag; the CPU backend is the
  reference everything falls back
  to.
- **The format is stable, the
  tools are not.** GGUF files from
  years ago still load. Flags,
  defaults and tool names, however,
  have changed before and will
  again — the `--help` of *your*
  build (transcript t00) outranks
  any guide, including this one.
