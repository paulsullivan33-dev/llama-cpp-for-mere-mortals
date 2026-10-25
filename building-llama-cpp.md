# Building llama.cpp

llama.cpp is source-first software:
the normal way to get it is to clone
the repository and compile it, which
takes a few minutes and leaves you
with the tools in `build/bin/`. There
are other doors — prebuilt archives
on the project's GitHub releases
page, and packages such as Homebrew's
`llama.cpp` on macOS — and they are
fine. This chapter teaches the build,
because the build is where the
backends (chapter 2) are chosen, and
because building is what the lab did.

## What you need

- A C/C++ compiler — `gcc`/`g++` or
  Clang. On Debian or Ubuntu:
  `sudo apt install build-essential`.
  On macOS, the Xcode command line
  tools (`xcode-select --install`).
  On Windows, Visual Studio's C++
  workload, or build inside WSL.
- **CMake**, 3.14 or newer:
  `sudo apt install cmake`.
- **git**, to fetch the source:
  `sudo apt install git`.
- For the tools' built-in model
  downloading (a convenience, not a
  requirement), `curl` and its headers
  — Ubuntu's package is
  `libcurl4-openssl-dev`. Without it
  the build quietly proceeds with
  downloading disabled, and you fetch
  model files yourself (chapter 5).

## The build, CPU only

```bash
git clone https://github.com/ggml-org/llama.cpp
cd llama.cpp
cmake -B build
cmake --build build --config Release -j
```

That is the whole recipe. CMake works
out what your machine offers; on an
ordinary Linux machine it builds the
CPU backend with the best instruction
sets it finds. When it finishes, the
tools are in `build/bin/`:

```bash
ls build/bin | grep llama
```

Check:

```bash
./build/bin/llama-cli --version
```

prints a version line naming the
commit it was built from — the
project versions its builds by
number and commit both. The lab's
build is recorded in transcript t00,
including the exact commit, so its
numbers can be compared against a
later one honestly.

## Choosing a GPU backend

One flag at the `cmake -B build` step
adds a backend; everything else is
the same. These are documented here,
not tested by the lab, which had no
GPU:

```bash
# NVIDIA (needs the CUDA toolkit)
cmake -B build -DGGML_CUDA=ON

# Vulkan (most GPU makers, incl. Intel/AMD)
cmake -B build -DGGML_VULKAN=ON

# AMD via ROCm/HIP (Linux)
cmake -B build -DGGML_HIP=ON

# Intel GPUs via SYCL (needs Intel's oneAPI)
cmake -B build -DGGML_SYCL=ON
```

On macOS, Metal support is built by
default and needs no flag. A build
can carry the CPU backend plus a GPU
one; run time flags (`-ngl`, the
number of layers to offload,
chapter 8) decide how much of the
model the GPU holds.

## Keeping it current

The project moves quickly — new model
architectures land as support in new
commits. Updating is the same two
commands after a `git pull`:

```bash
git pull
cmake --build build --config Release -j
```

If a model file ever refuses to load
with an architecture error, a stale
build is the first suspect
(chapter 14).

## The lab's build

The lab built on a small Linux
machine (2 CPUs, 7 GB RAM) from a
shallow clone, CPU backend only,
Release configuration, building the
four main tools and `llama-perplexity`:

```bash
cmake -B build -DCMAKE_BUILD_TYPE=Release
cmake --build build -j 2 --target llama-cli llama-server llama-quantize llama-bench llama-perplexity
```

The build took minutes, not hours,
even on two cores; a full default
build (every example and test) takes
longer and is not needed to follow
this guide.
