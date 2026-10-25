# Troubleshooting

Symptom → likely cause → fix. Every
entry here is a real failure mode of
this stack; the ones the lab caused
on purpose or by accident are
marked *(lab)*.

## "unknown architecture" / the model will not load

The build is older than the model
family. `git pull` and rebuild
(chapter 3). If it still refuses,
check the file itself: an HTML error
page saved as `.gguf` (a download
that quietly failed) loads as
nothing at all — `file model.gguf`
should say *data*, and the first
four bytes should read `GGUF`.

## It runs, but absurdly slowly

- **The model is swapping.** File
  size + context near or over RAM.
  Drop a quant (chapter 6), shrink
  `-c`, or close something. *(lab:
  the surest way to meet this is a
  big model and a big context at
  once.)*
- **Threads left at default.**
  `-t` set far below your physical
  cores wastes the prompt processing
  phase; far above it adds contention
  (chapter 8).
- **The wrong layer split on GPU.**
  With partial offload (`-ngl` short
  of the layer count), every token
  crosses the bus. All-or-nothing
  usually wins.

## It answers, but the answers are gibberish or loops

- **Wrong chat format.** A chat
  model fed raw completion-style
  text (or the reverse) talks
  nonsense politely. `llama-cli`
  applies the GGUF's chat template
  itself; gibberish usually means
  the file lacks a template (pass
  `--chat-template` if you know the
  right one) or the model is a base
  model wearing an instruct name.
- **Temperature at extremes.**
  `--temp 0` can lock a small model
  into repeating itself; very high
  values dissolve into noise.
  0.6–0.8 is a sane conversational
  band (chapter 4).
- **A quant too far.** At the
  smallest sizes (chapter 6's
  table), a model can go visibly
  strange: repeated words, lost
  instructions. That is the measured
  cost, not a bug — step up one
  quant.

## "the request exceeds the available context size"

The prompt plus the reply you asked
for is bigger than `-c`. Raise `-c`
(it costs memory), shorten the
prompt, or ask for fewer tokens
(`-n`). Server clients hit the same
wall as an HTTP 400 with the same
message.

## The server answers on one machine only

It is bound to `127.0.0.1`, as
chapter 7 advises. That is the safe
default working as intended. To
share it on a trusted network,
bind the machine's LAN address
deliberately — and reread the
authentication warning in chapter
11 before you do.

## Port already in use

Something else owns 8080 (often:
yesterday's server). `ss -ltnp |
grep 8080` names it on Linux. Pick
another port with `--port` rather
than killing strangers.

## Quantize refuses, or the output is the same size

- **Same size:** you asked for the
  type the file already is (an F16
  file "quantized" to F16). Check
  the source type in the load banner
  or the file name.
- **Refuses with a tensor error:**
  a few architectures carry tensors
  that cannot take certain quant
  types; the error names the tensor.
  The fallbacks are one step up
  (Q4_K_M instead of an I-quant) or
  `--allow-requantize` — know that
  requantizing an already-small
  file stacks two losses (chapter
  6).
- **imatrix missing:** IQ quants
  want an importance matrix file
  (`--imatrix`); without one the
  tool warns or falls back, and the
  result is not the quant the tables
  praise.

## Build failures

- **CMake cannot find a compiler:**
  the toolchain is not installed
  (chapter 3's list), or on Windows,
  the build was not run from a
  developer shell.
- **A backend flag fails:**
  `-DGGML_CUDA=ON` without the CUDA
  toolkit installed fails at
  configure time, by design. Install
  the toolkit or drop the flag — the
  CPU build is complete without it.
- **Stale build tree after a pull:**
  when in doubt, delete `build/` and
  configure again. It costs minutes
  and removes a whole class of
  ghosts.
