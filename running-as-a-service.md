# Running as a Service

A model you use every day should not
need a terminal left open.
`llama-server` is a normal long-running
program, so the normal tools apply:
this chapter runs it under **systemd**
on Linux. (On macOS the equivalent is
a `launchd` plist; on Windows, a
scheduled task or a service wrapper —
the server flags below are the same
everywhere.)

**What the lab did and did not do:**
the unit file below was written for
this guide and checked with
`systemd-analyze verify`; actually
starting it under systemd is
documented only, because the lab
machine runs no user session manager
to start it under. The server itself
is lab-tested in chapter 7 — only the
wrapping is documented.

## The unit

A user service needs no root. Save
as `~/.config/systemd/user/llama-server.service`:

```ini
[Unit]
Description=llama.cpp server (small local model)
After=network.target

[Service]
ExecStart=%h/llama.cpp/build/bin/llama-server \
  -m %h/models/smollm2-360m-instruct-q4_k_m.gguf \
  --host 127.0.0.1 --port 8080 \
  -c 4096 -t 4
Restart=on-failure

[Install]
WantedBy=default.target
```

Then:

```bash
systemctl --user daemon-reload
systemctl --user enable --now llama-server
systemctl --user status llama-server
curl http://127.0.0.1:8080/health
```

Check: `/health` answers
`{"status":"ok"}` once the model has
loaded, and the service survives
logout if you enable lingering
(`sudo loginctl enable-linger $USER`)
— that part is a real decision, not a
default: a lingering user session
runs from boot, which is what you
want for a home server and not
necessarily for a laptop.

## The two settings that matter

- **`--host 127.0.0.1`.** The server
  has **no authentication of its
  own**. Bound to localhost, only
  your machine can ask it questions.
  Binding `0.0.0.0` publishes an
  open, unauthenticated model to
  your whole network — and if the
  port is ever forwarded, to the
  internet. There is an `--api-key`
  flag for a shared-secret key; treat
  localhost binding as the real
  boundary and the key as a
  seatbelt, not a wall (chapter 13
  returns to this).
- **The model path is absolute in
  effect.** The unit's working
  directory is your home; the `%h`
  paths above keep that honest. When
  a service "cannot find the model",
  it is almost always a relative
  path meeting a different working
  directory (chapter 14).

## Logs and care

The journal takes the server's
output: `journalctl --user -u
llama-server -f` follows it, the same
place chapter 7's access log lines
go when the terminal isn't there.
Restarting after a model swap is
`systemctl --user restart
llama-server` — expect clients to see
a brief refusal while the new model
loads; the load, not the restart, is
the slow part on a big model.
