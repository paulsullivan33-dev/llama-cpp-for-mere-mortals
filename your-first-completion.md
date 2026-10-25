# Your First Completion

Time to run a model. This chapter uses
the lab's model — SmolLM2-360M-Instruct,
Q8_0, downloaded in chapter 5's manner
(the chapters are ordered for reading;
the lab downloaded first and read
later, and so can you). Swap in your
own GGUF path and everything here
stands.

## One prompt, one answer

```bash
./build/bin/llama-cli \
  -m ~/models/smollm2-360m-instruct-q8_0.gguf \
  -p "Write one short paragraph about a lighthouse keeper who finds a message in a bottle." \
  -st -n 96 --temp 0.7
```

What happens, in order:

1. A short banner: the build number,
   the model path, and its file type
   (`ftype: Q8_0` — chapter 6's subject).
   When anything looks wrong later,
   this banner is the first evidence.
2. Your prompt, echoed after a `>`.
3. The model's reply, streamed token
   by token as it is generated.
4. A timing line:

```text
[ Prompt: 85.2 t/s | Generation: 23.4 t/s ]
```

Those two speeds are the shape of
every run in this guide (chapter 2):
digesting the prompt is fast and
parallel — 85 tokens per second on
the lab's modest 2-CPU machine — and
generation is the steady drumbeat,
23 tokens per second here, roughly
three times reading speed. A 360M
model on a small machine is a kind
first subject: the whole run above
took seconds.

The flags that mattered:

- `-m` — the model file. The only
  required flag.
- `-p` — the prompt. In this build of
  `llama-cli`, a prompt to a chat model
  is treated as one conversation turn:
  the chat template stored in the
  GGUF (chapter 5) is applied for
  you. There is no separate
  conversation flag to remember.
- `-st` (`--single-turn`) — answer
  once, then exit. Without it, the
  tool stays open for a conversation
  (next section).
- `-n 96` — generate at most 96
  tokens. Left alone, a model may
  ramble to the end of its context;
  a budget keeps experiments quick
  and comparable.
- `--temp 0.7` — sampling randomness
  (chapter 8). 0.7 is a reasonable
  middle for prose.

## Talking to it instead

Leave off `-p` and `-st`:

```bash
./build/bin/llama-cli \
  -m ~/models/smollm2-360m-instruct-q8_0.gguf
```

You get a prompt of your own: type a
message, read the reply, type the
next. The conversation's history is
kept in the context (chapter 8's
working memory), so follow-ups can
refer back — until the context fills
or the small model loses the thread,
whichever comes first. In-session
commands: `/exit` leaves, `/clear`
forgets the conversation, `/regen`
asks again. Ctrl-C stops a generation
mid-stream.

Check: a healthy session answers in
character and remembers your previous
message. If it answers with raw,
unformatted oddity — repeating your
words, ignoring the turn structure —
the model file likely lacks a chat
template, or is a *base* model rather
than an instruct one. Base models
complete text; instruct models hold
conversations. The file name usually
says which you have (`-Instruct`,
`-Chat`), and chapter 14 has the
rest of the symptoms.

## What just happened, honestly

The reply in the lab's transcript
(t01) is fluent, on-topic — and pure
invention, delivered without a flicker
of doubt, by a model with 360 million
parameters that has never seen a
lighthouse. That pairing — real
fluency, no grounding — is the
permanent condition of these tools,
at every size, and chapter 13 returns
to it. Enjoy the trick; verify the
facts.
