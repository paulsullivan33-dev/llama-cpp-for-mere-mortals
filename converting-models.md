# Converting Models

Most models are *published* in the
format of the training frameworks —
a Hugging Face repo full of
safetensors files — and become a
GGUF through conversion. You may
never need to do this: chapter 5's
downloads are other people's
conversions. This chapter explains
the process so those files are not
magic, and gives the recipe for the
day a model you want exists only in
its original form.

**This chapter is documented, not
lab-tested.** Conversion needs a
Python environment with PyTorch and
the model's own libraries — a
multi-gigabyte install the lab did
not make for a step its experiments
did not require. The recipe below
follows the project's own
documentation; the numbers in every
other chapter, by contrast, were
measured.

## The script

The converter ships in the llama.cpp
repo:

```bash
# in the llama.cpp clone, with its
# Python requirements installed:
pip install -r requirements.txt

python3 convert_hf_to_gguf.py \
  --outfile model-f16.gguf \
  --outtype f16 \
  /path/to/hf-model/
```

- The input is a *local copy* of the
  Hugging Face repo (download it with
  the `hf` command-line tool or git).
- `--outtype f16` writes the weights
  at full 16-bit precision. Converters
  publish this F16 (or BF16) file
  first, because every smaller quant
  is made from it with
  `llama-quantize` (chapter 6), and
  quantizing from a full-precision
  source beats quantizing from an
  already-small file.
- The script reads the model's own
  config and tokenizer, and writes
  the chat template into the GGUF
  when the model declares one. A
  conversion that loses the template
  still *runs* — it just chats
  badly, which is a conversion bug,
  not a model bug (chapter 14).

## Fine-tunes and adapters

Two variations you will meet in the
wild:

- **Fine-tunes** are full models
  trained further from a base. They
  convert exactly like the base —
  the whole recipe above, nothing
  special.
- **LoRA adapters** are small add-on
  files that modify a base model's
  behaviour. llama.cpp converts them
  with `convert_lora_to_gguf.py`,
  and the result is loaded
  *alongside* the base GGUF
  (`--lora` in the tools) rather
  than replacing it. Merging an
  adapter into the base happens
  upstream, in the training
  framework, before conversion.

## Checking a conversion

Load the result and look at the
banner (`llama-cli -m model.gguf
-p "Hello" -n 1`): the architecture,
parameter count and context length
printed should match the source
repo's claims, and a chat model
should answer in character, in the
right format. A mismatch in the
parameter count means the converter
read a different model than you
think you downloaded — stop there,
not after an afternoon of odd
answers.

When conversion matters to get
right, prefer the maker's own GGUF
when it exists (chapter 5); the
recipe here is the fallback for
when it does not.
