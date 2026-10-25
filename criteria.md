# Writing Criteria

Every guide in this repo is written to the same
standard. If you're adding or editing a guide —
human or AI — hold it to these.

1. **Plain words, real ideas.** Simple language,
   never dumbed-down concepts. Explain jargon on
   first use; collect terms in the glossary.
2. **No hype.** No marketing adjectives, no
   "revolutionary," no promises about what
   something will do for the reader.
3. **Honest about limits.** Say what the thing
   *can't* do. State uncertainty instead of
   smoothing it over. Validation notes include
   what remains unverified.
4. **No invented numbers.** Benchmarks, defaults,
   and specs are either measured or qualified with
   "depends on your setup." If it wasn't tested,
   say so.
5. **Checkable.** Commands are meant to be run,
   with "Check:" blocks showing what success looks
   like. Sources are linked, not asserted.
6. **Respect the reader's setup.** Call out
   Windows/Mac/Linux differences. Don't assume a
   GPU, a budget, or prior experience.
7. **Safety isn't skipped.** Real risks get
   stated plainly, not buried.
8. **Scoped.** Each guide says what it covers and
   points elsewhere for the rest. No guide
   pretends to be the whole subject.
9. **Structure over prose.** Tables, before/after
   examples, symptom → fix. Scannable beats essay.

## Verification notes for this guide

Built and verified 2026-10-24/25 against
**llama.cpp at commit
d327bf1f0780f87b8c01b367b61853b0c291d7c4**
(shallow clone of ggml-org/llama.cpp;
tools report "version: 0.7.0-dev
(build 1, commit d327bf1)", built with
GNU 13.3.0), on a 2-CPU, 7.7 GB sandbox
machine running Ubuntu 24.04, **CPU
backend only — the machine has no
GPU.**

**Primary sources read, not remembered:**
the project's own README and the server
tool's README in the cloned tree; the
`--help` output of the built tools
themselves (transcript t00) — which
overruled this guide's first draft
twice: this build has no
`-cnv`/`--conversation` flag (the chat
template is applied by default, and
`-st`/`--single-turn` is the scripting
shape), and its default server slot
count and context come from the server
log, not from older documentation.

**The model.** SmolLM2-360M-Instruct,
Q8_0 GGUF (386,404,992 bytes, sha256
starts 48ab3034d0dd401f), downloaded
with curl from the makers' own repo
(HuggingFaceTB/SmolLM2-360M-Instruct-GGUF,
Apache-2.0). That repo ships **only**
the Q8_0 file, which shaped the quant
work below.

**Ran for real** (transcripts in
`llama-lab/transcripts/`):

- t00 — the cloned commit, and
  `--version` for llama-cli, llama-server
  and llama-bench: all
  0.7.0-dev / d327bf1.
- t01 — a first completion with
  `llama-cli -st`: a coherent
  lighthouse paragraph; timing line
  "Prompt: 85.2 t/s | Generation:
  23.4 t/s".
- t02 — `llama-quantize
  --allow-requantize` from the Q8_0
  file to Q5_K_M (289,944,192 bytes,
  6.37 BPW), Q4_K_M (270,590,592,
  5.94) and Q3_K_M (234,686,592,
  5.15); every run warned that **192
  of 290 tensors required fallback
  quantization** (the model's 960-wide
  layers are not divisible by the
  K-quant block size 256) — the reason
  the files cluster instead of
  shrinking to their headline sizes,
  and chapter 6 is written around it.
- t03 — the same fixed prompt at
  temperature 0 on Q8_0, Q4_K_M and
  Q3_K_M: three coherent paragraphs;
  the Q3 one repeats a phrase three
  times.
- t03b — `llama-perplexity` on a
  1,000+ token sample written for this
  guide (kept at `llama-lab/ppl-sample.txt`):
  Q8_0 **25.85**, Q4_K_M **26.42**,
  Q3_K_M **28.78**.
- t04 — `llama-server` (Q4_K_M,
  127.0.0.1:8080): `/health` →
  `{"status":"ok"}` (after answering
  503 "Loading model" during load,
  kept in the chapter as the honest
  detail); `/completion` answered
  "The capital of France is" →
  " Paris..."; `/v1/chat/completions`
  returned a haiku with a usage
  block; the built-in web page at `/`
  was fetched (it is served
  compressed — plain curl gets a
  gzip error message, also kept).
  Server log: n_slots = 4,
  n_ctx_slot = 8192.
- t05/t05b — `llama-bench`, two
  rounds: prompt processing rises
  with threads (54.9 → 67.0 → 68.5
  t/s at 1/2/4 threads) while
  generation falls (23.7 → 19.8 →
  13.0); and at one thread Q4_K_M
  generation (11.0 t/s, confirmed on
  a repeat) is about half Q8_0's —
  the smaller file was slower on
  this CPU. Error bars are large;
  the machine is a shared sandbox
  and the chapters say so wherever
  the numbers appear.
- t06 — the systemd unit in chapter
  10 (kept verbatim at
  `llama-lab/systemd/llama-server.service`)
  passed `systemd-analyze verify`
  (exit 0; the only warnings concerned
  unrelated host units), with the
  `%h` paths made real via symlinks.

**Errors hit and diagnosed for real:**
the first build died when the
compiler's temporary files filled the
sandbox's small (512 MB) `/tmp` tmpfs
— rerun with TMPDIR inside the
workspace, it completed (t90-build);
`llama-quantize` refusing to
requantize the Q8_0 file without
`--allow-requantize` (t90-requantize);
`llama-perplexity` refusing samples
under 1024 tokens — the sample was
extended in stages, 362 → 852 → 1017
→ enough (t90-perplexity). One
quantize run was also interrupted
mid-write by the lab's own tooling
and left a truncated Q3_K_M file;
it was re-run and the size verified
before use, which is why t02 was
regenerated clean.

**The sandbox, declared.** Speeds
here are a shared 2-vCPU sandbox's:
usable for shapes and comparisons
run back-to-back, not as a benchmark
of llama.cpp, of SmolLM2, or of any
machine a reader owns. Every chapter
that quotes a speed carries that
caveat or points at it.

**Documented only, and labelled as
such in the chapters:** every GPU
backend (CUDA, Metal, Vulkan, HIP,
SYCL) and `-ngl` layer offload — no
GPU exists here; converting a model
from Hugging Face format (chapter 9
is the project's recipe, untested —
its PyTorch install was not made);
actually *starting* the systemd unit
(the sandbox runs no user session
manager; the unit is verified as
text, and the server it wraps is
lab-tested in chapter 7); I-quants
and imatrix generation (described,
not run — the lab's quant story was
already full); and the tools'
built-in `-hf` model downloading
(the lab fetched its file with curl
deliberately, chapter 5).

**Fixtures:** everything the lab
made up is small and generic — a
lighthouse story prompt, a haiku
request, a capital-of-France
completion, and thirteen short
paragraphs about harbours, soup,
bicycles, trains, bread, weather,
gardens, libraries and ferries,
written for the perplexity sample.
Nothing in the lab is modelled on
any real person, organisation or
event.
