# The Neighbours

llama.cpp is the engine room. Around
it sits a small town of tools that
use it, compete with it, or wrap it
in friendlier clothes. Knowing which
is which saves real confusion,
because they all run "the same
models".

## Built on llama.cpp

- **Ollama** — the most popular
  front end. It wraps llama.cpp in a
  daemon with a model library of its
  own (`ollama run <name>`), its own
  download registry and its own
  model-file format (built from
  GGUFs). People choose it to skip
  chapter 3 and chapter 5 entirely:
  no build, no file hunting. The
  trade is control — quant choice,
  versions and knobs are Ollama's to
  expose or not — and a model
  library separate from your GGUF
  files.
- **LM Studio** — a desktop
  application (closed source, free
  to use) with llama.cpp inside: a
  model browser, a chat window, and
  a local server. The gentlest on
  ramp of the set, and the least
  scriptable.
- **GPT4All, Jan, KoboldCpp, text
  generation web UI** — desktop and
  community front ends, several with
  llama.cpp at their core, each with
  its own focus (chat, documents,
  role-play, tinkering). All are
  easier than the terminal; none
  teaches you what chapter 2 does.

## Different engines

- **vLLM and SGLang** — serving
  engines for data centres. They
  batch many users' requests through
  big GPUs at high throughput, and
  are the right tool for "a model as
  a company service". They do not
  target your laptop, and they do
  not read GGUF.
- **PyTorch + Transformers** — the
  research framework where models are
  born. It runs models in their
  original format, on GPU-first
  assumptions, with Python as the
  price of admission. If you train,
  fine-tune, or convert (chapter
  10), you will meet it; for
  inference on your own machine it
  is the heavy way to do llama.cpp's
  light job.
- **MLX** — Apple's own framework
  for Apple silicon, with its own
  model format. On a Mac it is a
  genuine alternative engine, and
  some makers publish MLX builds
  beside the GGUFs.
- **ONNX Runtime, TensorRT-LLM** —
  vendor-leaning inference stacks,
  aimed at deployment pipelines
  rather than a person at a terminal.

## Which one, when

| Want… | Reach for |
|---|---|
| To understand and control the run | llama.cpp (this guide) |
| The shortest path to a chatting model | Ollama or LM Studio |
| Many users on a GPU server | vLLM or SGLang |
| To train or fine-tune | PyTorch + Transformers |
| A Mac-native engine | MLX, or llama.cpp's Metal backend |

The GGUF files you collect for this
guide are not wasted if you wander:
most front ends above can import
them, because most of them *are*
llama.cpp underneath.
