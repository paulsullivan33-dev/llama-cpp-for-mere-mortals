# Quantization

A model's weights start life as
16-bit floats: two bytes for every
number, so an 8B model is a ~16 GB
file. **Quantization** stores those
numbers at lower precision — fewer
bits per weight — trading a little
accuracy for a much smaller, faster
file. It is the single most important
idea in this guide: it is why models
fit on ordinary machines at all.

## Reading a quant name

Take `Q4_K_M` apart:

- **Q4** — roughly 4 bits per weight
  (the average; the fine print is
  below).
- **K** — a *k-quant*: weights are
  stored in blocks, each block with
  its own scale, and different kinds
  of tensor inside the model get
  different precision — the important
  ones kept finer. The older plain
  quants (`Q8_0`, `Q4_0`) use one
  simple scheme throughout.
- **M** — medium. Most K-quants come
  as `_S`, `_M`, `_L` (small, medium,
  large): the same bit budget spent
  slightly differently, M the usual
  default.

The family, by appetite:

| Quant | Bits/weight (headline) | Character |
|---|---|---|
| F16 | 16 | The unquantized original |
| Q8_0 | ~8.5 | Near-lossless; the safe ceiling |
| Q6_K | ~6.6 | Very close to Q8 |
| Q5_K_M | ~5.7 | Small loss, rarely visible |
| Q4_K_M | ~4.9 | The common default: big shrink, modest cost |
| Q3_K_M | ~3.9 | Noticeable cost; for tight fits |
| Q2_K / IQ | ~2–3 | Last resorts; expect strangeness |

And **I-quants** (`IQ3_M` and kin)
go further using an **importance
matrix** — statistics from sample
text that tell the quantizer which
weights to protect (chapter 2). They
are the smallest usable files and
the easiest to get wrong.

## What the lab measured

Headline numbers are averages over
well-shaped models. The lab's model
taught a sharper lesson. SmolLM2-360M
is *narrow* — its layers are 960
numbers wide — and k-quant blocks
want widths divisible by 256.
960 is not. So when
`llama-quantize` rewrote the file,
**192 of its 290 tensors fell back**
to simpler, coarser types, and the
sizes came out like this
(transcript t02):

| File | Bytes | Bits/weight (reported) |
|---|---|---|
| Q8_0 (downloaded) | 386,404,992 | 8.50 |
| Q5_K_M | 289,944,192 | 6.37 |
| Q4_K_M | 270,590,592 | 5.94 |
| Q3_K_M | 234,686,592 | 5.15 |

Read that table twice. Q3_K_M's
headline is ~3.9 bits per weight;
this model got 5.15, because most of
it is not stored as Q3 at all. The
four files huddle within 40% of
each other instead of spanning the
3-to-1 range the names suggest.
**The lesson generalises: a quant's
name is a recipe, not a result.**
On wide models (an 8B's layers are
typically 4096 across, divisible by
everything) the files land near
their headlines — chapter 5's
rule-of-thumb table assumes those.
On small or oddly-shaped models,
measure the file you actually got.

## What the smaller files cost

Two prices, both measured on the
same fixed prompt and the same
sample text (transcripts t03):

**Perplexity** (chapter 2: how
surprised the model is by ordinary
prose; lower is better) on a
1,000-token sample written for this
guide:

| Quant | Perplexity |
|---|---|
| Q8_0 | 25.85 |
| Q4_K_M | 26.42 (+2%) |
| Q3_K_M | 28.78 (+11%) |

**The outputs themselves.** All
three quants wrote a coherent
lighthouse paragraph at temperature
0. The Q3_K_M version wobbled —
one sentence repeated its key
phrase three times — the kind of
small loop you'd forgive in a draft
and notice in a tool. Q4_K_M's
paragraph was, to this reader,
indistinguishable in quality from
Q8_0's. Different text, same job:
the quant cost shows up as texture
before it shows up as failure.

And **speed** — where the lab got
its second surprise. Generation
reads the whole model per token, so
smaller *usually* means faster. On
this machine it did not
(transcripts t05b, one thread,
`llama-bench` tg128): Q8_0 generated
at **23.7 t/s**, Q3_K_M at 20.0, and
Q4_K_M at **11.0** — the middle file
was the slowest, confirmed on a
repeat run. K-quants cost arithmetic
to unpack (dequantizing blocks back
to numbers the CPU can multiply),
and on this model and this CPU that
cost ate the bandwidth saving whole.
The general rule bends both ways:
on wide models with fast K kernels
the smaller file wins; here it
lost. Measure on your machine,
with your model — chapter 8 shows
the bench — before assuming a
smaller quant is a faster one.

## Making a quant: the honest path

```bash
llama-quantize model-f16.gguf model-q4_k_m.gguf Q4_K_M
```

Quants are made **from the
full-precision file**. The lab's
download, though, was Q8_0 — the
maker's repo ships nothing else —
and the tool refuses to requantize
an already-quantized file without
being told twice:

```text
failed to quantize: requantizing from type q8_0 is disabled
```

(transcript t90). The lab added
`--allow-requantize` and says so
plainly: this stacks one quant's
loss on another's. From Q8_0 —
itself near-lossless — the stack is
shallow, and the perplexity table
above is the result *including*
that handicap. If you are publishing
quants for others, start from F16;
if you are shrinking a file for
yourself, Q8_0 is a decent place
to start and Q4_K_M a decent place
to land.

## Choosing one

- **Memory plentiful:** Q8_0 (or
  Q6_K). Buy the quality; it is
  cheap at small sizes.
- **The default:** Q4_K_M — the
  balance the community settled on:
  most of the shrink, little of the
  cost.
- **A tight fit:** Q3_K_M or an
  I-quant, with your eyes open —
  and measure your actual file,
  because as the lab's table shows,
  the saving over Q4 may be smaller
  than the name promises, while the
  perplexity cost (+11% here) is
  real.
- **Below that:** you are no longer
  shrinking a model, you are
  auditioning a smaller model's
  impersonation of it. Sometimes
  that is the right trade. Know
  that it is the trade you're
  making.
