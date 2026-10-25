# The Server

`llama-cli` answers one person at a
terminal. `llama-server` loads a model
once and answers HTTP requests — from
scripts, from other programs, from a
web page it serves itself. This is
the shape most local models end up
in: running quietly in the background
of a household or a project, asked
questions by software.

## Starting it

```bash
./build/bin/llama-server \
  -m ~/models/smollm2-360m-instruct-q4_k_m.gguf \
  --host 127.0.0.1 --port 8080
```

The lab's run (transcript t04) took
about six seconds to load its model,
and its log said what it had made:

```text
load_model: initializing, n_slots = 4, n_ctx_slot = 8192
listening on http://127.0.0.1:8080
```

Two ideas in that line:

- **Slots** are parallel seats. Each
  slot holds one conversation's KV
  cache, so four slots can generate
  four answers at once — sharing the
  one loaded model, and sharing the
  machine's memory bandwidth between
  them. Four is this build's default
  (`-np` changes it).
- **`n_ctx_slot`** is the context
  per slot — here 8192, the model's
  own trained length, taken as the
  default. Four slots × 8192 tokens
  of KV cache is the memory bill;
  chapter 8's arithmetic applies
  per slot.

While the model loads, the server
already answers — honestly:

```bash
curl http://127.0.0.1:8080/health
# {"error":{"message":"Loading model", ... "code":503}}
```

and once loaded:

```text
{"status":"ok"}
```

Check: scripts that start the server
should poll `/health` for that `ok`,
not just for any answer — the lab's
first script got the 503 and marched
on, which is how the 503 ended up in
its notes. `/health` is also the
endpoint chapter 10's service check
uses.

## Asking it questions

Two doors, one model.

**`/completion`** — the raw door.
You send a prompt as plain text; the
model continues it:

```bash
curl http://127.0.0.1:8080/completion \
  -H 'Content-Type: application/json' \
  -d '{"prompt": "The capital of France is",
       "n_predict": 16, "temperature": 0}'
```

The lab's answer came back
`" Paris.\n\nParis is the capital of
France..."` — a 360M model
reassuring itself — inside a JSON
envelope carrying the stop reason,
the token counts, and a `timings`
block with the prompt and generation
speeds of *that request*. Nothing is
formatted for you: no chat template
is applied, which makes this the
right door for completion-style work
(filling, continuing, classifying
with a fixed format) and the wrong
door for conversation.

**`/v1/chat/completions`** — the
chat door, shaped like OpenAI's API,
which is the point: tools written
for that API mostly work against
this server by changing the address.
You send messages; the template from
the GGUF is applied for you:

```bash
curl http://127.0.0.1:8080/v1/chat/completions \
  -H 'Content-Type: application/json' \
  -d '{"messages": [{"role": "user",
        "content": "Give me a haiku about autumn rain."}],
       "temperature": 0.7, "max_tokens": 64}'
```

The lab's model replied with a
serviceable haiku ("Autumn rain, soft
as mist, / Conveys the change of
seasons, / Nature's gentle touch."),
plus a `usage` block counting prompt
and completion tokens. Two honest
notes: the reply's `model` field
echoes your file path, not a product
name; and "OpenAI-compatible" means
the *shape* — a client that leans on
a specific provider's extras will
still trip.

**The web page.** Visiting
`http://127.0.0.1:8080/` in a browser
gets a built-in chat interface —
handy for trying a model, and the
quickest way to show someone what
the machine is doing. (It is served
compressed; `curl` needs
`--compressed` to see it, which the
lab learned from the error message.)

## The warning that belongs here

The server has **no accounts and no
login**. Its optional `--api-key` is
one shared secret checked at the
door — a seatbelt, not a wall. Bound
to `127.0.0.1`, as every example in
this guide binds it, only programs
on the same machine can reach it,
and that binding *is* the security
model. The day you bind `0.0.0.0`
you have published a model that
anyone on the network can use, at
your electricity's expense, with
your machine's name on its answers.
Chapter 13 counts this among the
limits; chapter 10 keeps the service
on localhost. Keep it there unless
you can name the network and trust
every machine on it.

## Stopping it

Ctrl-C in its terminal, or the
service controls of chapter 10.
In-flight requests die with it;
nothing is written anywhere that
needs cleaning up — a model file,
a process, and a port are the whole
estate.
