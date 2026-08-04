# CLI

## Commands

| Command | Description |
| --- | --- |
| `gnosis` | Open interactive chat mode. |
| `gnosis chat` | Open interactive chat mode explicitly. |
| `gnosis run [options] <prompt>` | Run one headless prompt and exit without session persistence. |
| `gnosis sessions` | List saved chat sessions. |
| `gnosis doctor` | Check local config, credentials, sessions, git, and workspace readiness. |
| `gnosis sessions search <query>` | Search saved session transcripts locally. |
| `gnosis resume <id>` | Resume a saved chat session. |
| `gnosis login` | Log in to an OpenAI ChatGPT/Codex subscription with device-code auth. |
| `gnosis logout` | Remove stored OpenAI subscription credentials. |
| `gnosis version` | Print the build-time Gnosis version. Also available as `gnosis -v` and `gnosis --version`. |
| `gnosis help` | Print usage. |

## Environment

- `ANTHROPIC_API_KEY` is required when `provider: anthropic`.
- `OPENAI_API_KEY` is required when `provider: openai` uses `openai_auth: api_key`.
- `OPENROUTER_API_KEY` is required when `provider: openrouter`.
- `GOOGLE_API_KEY` is required when `provider: google`.
- `openai_auth: subscription` uses stored ChatGPT/Codex device-code credentials created by `gnosis login` instead of an API key.

## Runtime Notes

- `gnosis` with no subcommand defaults to chat.
- `gnosis run` executes one prompt without opening the TUI, prints the final answer, and exits. It is intended for scripts and eval harnesses.
- `gnosis run` applies a `10m` timeout, does not create or update sessions, and supports `--json` for a machine-readable summary containing elapsed time and tool counts.
- Headless runs receive the standard tool registry and do not use interactive `tool_approvals`. Run Gnosis inside a VM or sandbox that provides the required filesystem, process, network, and credential boundaries.
- The removed `--permission` option returns migration guidance instead of being silently accepted.
- `gnosis run` accepts prompt text as arguments and prepends piped stdin when present, e.g. `cat prompt.md | gnosis run --json`.
- `gnosis doctor` is local-first: it checks config, required credential presence, session store access, git availability, and whether the current directory is a git workspace without calling providers or printing secrets.
- The interactive `@` file picker indexes files under Gnosis's effective startup working directory and inserts paths relative to that directory.
- `gnosis login` prints the OpenAI Codex device-code URL and one-time code, then stores refreshable subscription credentials in `~/.gnosis/auth.json` with file permissions intended to protect secrets.
- `gnosis logout` deletes the stored OpenAI subscription credential entry.
- Resuming a session attempts to change into the saved session cwd. If unavailable, Gnosis warns and stays in the current directory.
- Session saves happen after each user turn through the TUI `WithAfterSend` callback.

## Process Boundary

`cmd/gnosis.run(args, streams)` is the command dispatcher. It installs signal
handling, writes through the supplied stdin, stdout, and stderr streams, and
returns the command's exit code. Command helpers return status codes rather
than terminating the process.

The top-level `main` function contains the only `os.Exit` call. Before reaching
that boundary, `execute` initializes logging, defers logging cleanup, and calls
`run`. This guarantees cleanup completes before both successful and failed
commands terminate.
