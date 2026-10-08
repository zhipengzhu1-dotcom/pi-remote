# pi-remote

Remote control channel for the desktop pi agent. This repo is the **only** thing
in it: a prompt protocol. No project code, no secrets — never commit a token or
key here.

## How to send a prompt (phone or desktop)

1. Add a new file to `inbox/` — any name, e.g. `inbox/2026-02-05-what-time.md`.
2. Put the prompt in the file body, plain markdown:

   ```md
   What is the date on the desktop, and list the files in the workspace root?
   ```

3. Commit and push.

That's it. The desktop daemon polls every 15–30 seconds, picks up new
`inbox/` files in FIFO order, and runs one fresh, isolated pi session on each.

## Where to find the answer

Each processed prompt produces a directory in `outbox/`:

```
outbox/<prompt-filename>/
  output.md        # just the final answer — read this first
  prompt.md        # echo of the original prompt (verbatim)
  transcript.md    # full transcript of the agent run
  status.md        # ok | failed, exit status, and error detail on failure
```

The directory name **matches the prompt's `inbox/` filename** (`inbox/hello.md`
→ `outbox/hello.md/`), so a prompt and its result are trivially matched. A
re-run of an edited prompt (same name, new content) takes a `-2`, `-3`, …
suffix — every run keeps its own directory, and the full history stays in
this repo forever. Chronological order is carried by `git log` and the
`started`/`duration` lines in each `status.md`, not by directory names.

## Semantics

- **Multiple prompts:** processed one at a time, in the order they were pushed.
- **PC off:** the desktop is assumed always-on (it never sleeps). If the PC
  shuts down (power loss or manual shutdown) the daemon simply halts; prompts
  queue here and are processed FIFO on the next start. Sending a prompt means
  "runs when the PC is on", not "runs now".
- **Processed prompts:** the daemon marks a prompt handled by creating its
  `outbox/<prompt-filename>/` directory. A prompt is considered in-flight from the
  moment a run starts, so a daemon restart never double-runs it.

## Rules

- One prompt per `inbox/` file.
- The daemon treats every `inbox/*.md` file as a prompt — no test files, no
  stray markdown, no notes. If you need to park something here, it's a prompt.
- Credentials (the daemon's PAT) live in the desktop environment, never in
  this repo.

## License

Released under the MIT License. See [LICENSE](LICENSE).
