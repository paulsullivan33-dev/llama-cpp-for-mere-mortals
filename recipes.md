# Recipes

Short, complete jobs. All assume the
build from chapter 3, a model from
chapter 5, and the lab's small model
stands in for yours throughout —
swap the path and every recipe
stands.

## A private writing assistant in the terminal

```bash
llama-cli -m ~/models/smollm2-360m-instruct-q4_k_m.gguf \
  -t 2 --temp 0.7 \
  -sys "You are a concise writing assistant."
```

Run with no `-p`, the tool opens a
conversation: the chat template from
the GGUF is applied to every turn,
and `-sys` sets the job once, at
the top. Nothing leaves the machine.
Small models drift on long chats —
start a fresh session per task
rather than nursing one conversation
all day.

## Summarise a document from the shell

```bash
{ echo "Summarise this in three sentences:"; cat notes.txt; } \
| llama-cli -m ~/models/smollm2-360m-instruct-q4_k_m.gguf \
    -f /dev/stdin -st -n 220 --temp 0.3
```

`-st` (`--single-turn`) is the
scripting shape: take the prompt,
answer once, exit — no waiting at a
prompt nobody is watching. Low
temperature for factual work. The
document must fit the context along
with the answer — chapter 8's
arithmetic — so this suits notes
and articles, not books.

## A household endpoint for scripts

Run the server (chapter 7), then any
script can ask questions:

```bash
curl -s http://127.0.0.1:8080/v1/chat/completions \
  -H 'Content-Type: application/json' \
  -d '{"messages":[{"role":"user",
        "content":"Give me a haiku about autumn rain."}],
       "temperature":0.7}'
```

This is the seam where local models
become plumbing: a cron job that
drafts, a notebook that classifies,
a home-automation rule that phrases
its own notifications. Keep it on
localhost (chapter 10).

## Try five quants before breakfast

```bash
for q in q8_0 q5_k_m q4_k_m q3_k_m; do
  echo "== $q"
  llama-bench -m ~/models/smollm2-360m-instruct-$q.gguf \
    -t 2 -p 128 -n 64 2>/dev/null | tail -3
done
```

(Chapter 6's warning applies: on
the lab's machine the smallest file
was *not* the fastest. Your bench,
your answer.)

(Chapter 6 shows how each file is
made.) Speed is only half the story
— chat with the smallest one for two
minutes before deciding the table's
winner is your daily driver.

## The same model on a second machine

Copy the one file — that is the whole
migration:

```bash
scp ~/models/smollm2-360m-instruct-q4_k_m.gguf \
  otherhost:~/models/
```

Same GGUF, same answers, any OS the
tools are built for. There is no
registry to log into and nothing to
re-download, which is chapter 5's
point in a single command.
