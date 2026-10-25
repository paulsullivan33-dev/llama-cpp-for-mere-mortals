# Tuning and Performance

Four numbers decide how a run feels:
how fast the prompt is digested, how
fast tokens then arrive, how much
memory it all takes, and whether the
answers stay good. This chapter is
about the knobs that move them, in
the order to reach for them. The
measurements are the lab's — a
2-CPU, 7 GB machine and a 360M
model — so treat the *shape* as the
lesson and the numbers as one
machine's story.

## Measure first: llama-bench

```bash
llama-bench -m ~/models/smollm2-360m-instruct-q8_0.gguf -t 2
```

`llama-bench` runs the model with no
chat, no template, nothing between
you and the arithmetic, and reports
two speeds:

- **pp** (prompt processing) —
  tokens per second while digesting
  a prompt. This is the parallel
  phase; threads help it a lot.
- **tg** (text generation) — tokens
  per second while generating. This
  is the memory-bandwidth phase;
  model size rules it (chapter 6).

Every other knob below should be
judged against a bench run before
and after. Changing two things at
once teaches you nothing.

## Threads (`-t`)

The default is your physical core
count, which is usually right.
Prompt processing scales with
threads until the cores are busy;
generation gains less, because it
is waiting on memory, not math.

The lab's bench (transcripts t05
and t05b — two rounds, same shape)
on its 2-CPU box, Q8_0 model:

| Threads | pp512 (t/s) | tg128 (t/s) |
|---|---|---|
| 1 | 54.9 ± 5.9 | 23.7 ± 1.1 |
| 2 | 67.0 ± 9.6 | 19.8 ± 6.4 |
| 4 | 68.5 ± 3.6 | 13.0 ± 2.9 |

Prompt processing gained ~20%
going from one thread to two, then
stopped — there are only two cores.
Generation went *backwards* with
more threads, and got steadier at
one thread (note the error bars)
than at two. Read that with the
lab's circumstance attached: its
machine is a shared sandbox, its
CPUs have neighbours, and the noise
in these numbers is part of the
result. On a quiet machine of your
own, generation usually holds or
improves slightly up to the
physical core count — but the
durable lessons survive: extra
threads past the cores buy
contention, not speed, and a
thread count that looks wrong in a
bench run is data, not an insult
to the machine. Bench yours.

## Context (`-c`)

The context is the model's working
memory, measured in tokens, and it
is paid for in RAM (the KV cache)
whether or not the conversation
grows into it. Rules that survive
contact with reality:

- The GGUF carries the length the
  model was *trained* for (the lab
  model: 8192). Asking for more
  than that is possible in llama.cpp
  and degrades answers in ways that
  are hard to notice and easy to
  regret.
- Ask for what the job needs. A
  chat wants 4096; a one-shot
  question wants 1024; a long
  document wants the arithmetic done
  first: document tokens + answer
  tokens + a margin, or the run
  stops mid-answer with the context
  error from chapter 14.
- Doubling `-c` roughly doubles
  the KV cache. On a small machine
  that trade — not the model file —
  is often what pushes a run into
  swapping.

## Batch (`-b`, `-ub`)

Prompt processing moves through the
model in batches (default 2048
tokens, in physical sub-batches of
512, `-ub`). Raising them speeds
long prompts on strong machines and
costs memory and latency on small
ones. On anything resembling the
lab machine, the defaults are
already sane; this knob earns its
keep on servers (chapter 7) chewing
long documents.

## Sampling (temperature and friends)

Sampling decides *which* token is
picked from the model's candidates,
and it is a quality knob, not a
speed one:

- `--temp` — randomness. Near 0:
  the model's first choice every
  time (best for extraction, facts,
  code). 0.6–0.8: natural chat. Past
  1.0: increasingly unhinged,
  measurably so on small models.
- `--top-p`, `--top-k`, `--min-p` —
  fences around the candidate pool.
  Defaults are chosen by people who
  test them; change one at a time,
  with a fixed prompt, and compare
  — chapter 6's quant comparison
  uses exactly that discipline.

## GPU offload (`-ngl`) — documented

With a GPU build (chapter 3),
`-ngl N` places N of the model's
layers on the GPU. All of them
usually beats most of them; the
rest stay on the CPU and every
token crosses between the two.
`-ngl 999` is the common idiom for
"everything". The lab has no GPU,
so this knob is described, not
measured — one more reason the
neighbours in chapter 12 exist.

## The order to tune in

1. Fit first: pick the quant whose
   file + your context sit
   comfortably in memory (chapters
   5–6).
2. Bench it (`llama-bench`), note
   pp and tg.
3. Set `-c` to the job's real need.
4. Confirm `-t` matches physical
   cores; bench again.
5. Only then touch batch and
   sampling — for the job, not for
   the benchmark's sake.
