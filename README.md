# ccx — see where your Claude Code tokens actually went

A single HTML file. Drop your Claude Code transcripts in, get a breakdown of your
token spend — including **the prompt-cache misses that quietly cost you real money**.

No install, no account, no backend. Open the page, drag in files, done.
Everything is parsed in your browser; nothing is uploaded. There is no server to
upload to — the whole app is one static file you can read top to bottom.

**[→ Try it](https://lukimcfly.github.io/ccx/)**

---

## Why this exists

Claude Code writes a JSONL transcript for every session under `~/.claude/projects/`.
Those files record exactly what each request cost in tokens — including a field most
tools ignore entirely:

```json
"diagnostics": { "cache_miss_reason": { "type": "messages_changed",
                                        "cache_missed_input_tokens": 150000 } }
```

That is the API telling you **why your prompt cache missed and how many tokens it
cost you**. A cache read is billed at 10% of the input rate; a miss re-reads the whole
prefix at 100%. On a long session that difference is not a rounding error.

`ccx` surfaces those misses, groups them by cause, and prices each one.

## Two things it gets right that most token counters don't

**1. Cache tiers are priced separately.** Input tokens are not one number. A cache
*read* costs 0.1× the input rate; a 5-minute cache *write* costs 1.25×; a 1-hour write
costs 2×. Tools that sum all input tokens and multiply by the input rate can overstate
your spend by a large multiple, because in a healthy session the overwhelming majority
of input tokens are cheap cache reads. `ccx` shows you both numbers so you can see the gap.

**2. It tells you *why* the cache missed.** Not just "your hit rate is 94%", but:

| Reason | What it actually means |
|---|---|
| `messages_changed` | Earlier messages were edited or removed — the prefix stopped matching |
| `previous_message_not_found` | Prefix wasn't cached: resumed after the TTL expired, or a fork |
| `model_changed` | You switched models mid-conversation; caches are per-model |
| `tools_changed` | The tool set changed — tools sit at the very front of the prefix |
| `system_changed` | The system prompt changed — it sits ahead of the whole conversation |
| `unavailable` | Cache was temporarily unavailable server-side |

The first four are things you can act on. That's the point.

## What you get

- **Overview** — list-price value, cache hit rate, output tokens, session count
- **Cache diagnosis** — every miss, grouped by cause, with the cost of each
- **Cost by model** — per-model turns, input/cache/output tokens, and spend
- **Tool usage** — which tools got called and how often
- **Sessions** — sorted by cost, searchable, click through to the message timeline

## Usage

Open <https://lukimcfly.github.io/ccx/> and drag in files from:

| OS | Location |
|---|---|
| macOS / Linux | `~/.claude/projects/` |
| Windows | `%USERPROFILE%\.claude\projects\` |

Drop individual `.jsonl` files or the whole `projects` folder. Or clone and open
`index.html` directly — it works from `file://` too.

## About the numbers

Costs are **list-price equivalents** computed from the token counts in your transcripts,
using public per-model API pricing. If you use Claude Code on a subscription plan, this
is the API value of what you consumed — not a bill you paid. On API billing it
approximates the real charge.

Unknown or future model IDs fall back to Opus-tier pricing rather than being dropped, so
totals stay complete; they're shown under their real name in the model table so you can
spot them.

## Privacy

Your transcripts contain your source code, your prompts, and whatever else you pasted
into a session. So:

- No network requests. No analytics, no telemetry, no fonts, no CDN. Open devtools,
  watch the Network tab, drop a file in — it stays empty.
- No dependencies. One file, no build step, nothing from npm at runtime.
- The GitHub Pages copy is the same file as in this repo, served statically.

If you'd still rather not use a hosted page, download `index.html` and open it locally.
It behaves identically offline.

## License

MIT — free forever, no strings.

If it showed you something useful about your own spend, you can
[buy me a coffee](https://ko-fi.com/lukimcfly). Entirely optional; the tool is
identical either way.
