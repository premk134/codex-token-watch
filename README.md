# Codex Token Watch

Saw a lot of discussion around Codex cache hits and had the same confusion, so
I built a small tool for the info and analysis.

Codex Token Watch is a lightweight local CLI for viewing per-turn token usage,
prompt-cache performance, and API-equivalent cost for Codex Desktop tasks.

It runs only when called, reads Codex files locally in read-only mode, and has
no third-party Python dependencies.

## Requirements

- macOS
- Codex Desktop with local task history
- Python 3

## Install

From the project directory:

```bash
chmod +x codex-token-watch
mkdir -p "$HOME/.local/bin"
ln -s "$(pwd)/codex-token-watch" "$HOME/.local/bin/codex-token-watch"
```

Make sure `~/.local/bin` is on your `PATH`, then check:

```bash
codex-token-watch --help
```

## Usage

For a task report, pass a full task ID, a unique ID prefix, or a Codex task
link. For a cross-task cache-miss report, omit the task.

```bash
# Cache misses across every task active in the last 24 hours
codex-token-watch --zero-cache

# Cache misses across every task active in the last six hours
codex-token-watch --zero-cache --since 6h

# Every recorded turn
codex-token-watch codex://threads/YOUR-TASK-ID

# Latest five turns
codex-token-watch codex://threads/YOUR-TASK-ID --last 5

# Calls with no cached input
codex-token-watch codex://threads/YOUR-TASK-ID --zero-cache

# Cache misses within the latest three turns
codex-token-watch codex://threads/YOUR-TASK-ID --zero-cache --last 3

# Plain output without colors
codex-token-watch codex://threads/YOUR-TASK-ID --no-color
```

The normal report includes input, cache percentage, output, reasoning, estimated
API cost, cumulative task tokens, model-call count, duration, and model/effort
changes. The final `ALL` row always summarizes the complete task.

## Notes

- `INPUT` adds together input from every model call in the turn, so it can be
  larger than the model's context window.
- `OUTPUT` excludes reasoning tokens; `REASON` shows them separately.
- `--zero-cache` automatically excludes cache resets caused by explicit Codex
  compaction.
- Without a task, `--zero-cache` scans a rolling 24-hour window. Use `--since`
  with minutes, hours, or days, such as `30m`, `6h`, or `3d`.
- Cross-task reports omit first calls when the preceding call is unavailable,
  because their cache-miss gap cannot be established.
- `API EST.` is an estimate using Standard API token rates, not an actual
  Codex subscription charge. It excludes tool charges, taxes, subscription
  entitlements, and Fast/Flex/Batch adjustments.
- Unknown models or incomplete historical data display `—` instead of an
  unsafe estimate.

Embedded prices were checked on 2026-09-19 against the official pages for
[GPT-6 Astra](https://developers.openai.com/api/docs/models/gpt-6-astra),
[GPT-5.6 Sol](https://developers.openai.com/api/docs/models/gpt-5.6-sol),
[GPT-5.6 Terra](https://developers.openai.com/api/docs/models/gpt-5.6-terra), and
[GPT-5.6 Luna](https://developers.openai.com/api/docs/models/gpt-5.6-luna).

## Privacy and performance

Codex Token Watch reads local Codex state and rollout files. It does not modify
Codex data, upload task contents, or run in the background.

A 57 MB rollout containing 873 model calls was processed in about 0.2 seconds
on the development Mac.
