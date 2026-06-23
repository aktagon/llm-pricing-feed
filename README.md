# llm-pricing-feed

Generated daily feed of correctly-normalized, fully-traceable LLM API pricing. Produced
by the `publish-feed` workflow in the private `aktagon/llm-pricing` repo. Do not edit by
hand — every commit is a scheduled publish (06:17 UTC daily).

- `dist/latest.json` — current normalized feed (the file you usually want)
- `dist/v1/snapshots/<date>.json` — immutable dated snapshots (append-only history)
- `dist/ATTRIBUTION.md` — source attribution (models.dev, MIT) — ship this if you redistribute
- `dist/rejected.json` — what was excluded and why (transparency)
- `data/store/<provider>/<model>.json` — append-only per-model history (the audit trail)

## Consuming the feed

It is a public git repo — the "API" is `git clone` / `git pull`, no auth or keys.

```bash
git clone https://github.com/aktagon/llm-pricing-feed.git
git -C llm-pricing-feed pull   # refresh later
```

Or fetch a single file over HTTPS without cloning:

```bash
curl -sL https://raw.githubusercontent.com/aktagon/llm-pricing-feed/master/dist/latest.json
```

### `latest.json` shape

```json
{
  "schema_version": "1.2.0",
  "currency": "USD",
  "unit": "per_million_tokens",
  "models": [
    {
      "id": "anthropic/claude-opus-4-8",
      "provider": "anthropic",
      "model": "claude-opus-4-8",
      "kind": "chat",
      "input": 5,
      "output": 25,
      "cache_read": 0.5,
      "cache_write_5m": 6.25,
      "context_window": 1000000,
      "effective_date": "2026-05-28",
      "effective_date_kind": "published",
      "source_url": "https://models.dev/api.json",
      "observed_at": "2026-06-23T13:31:24.070Z"
    }
  ]
}
```

- Prices are **USD per million tokens** (read `unit`/`currency`; don't hardcode). `input: 5` = $5 / 1M input tokens.
- `id` (`provider/model`) is the stable join key.
- `kind` is `chat` or `embedding`; embeddings carry `input` only (no `output`). Optional fields (`cache_read`, `cache_write_5m`) appear only when the source provides them.

### Example: cost of a call (Python)

```python
import json, urllib.request

feed = json.load(urllib.request.urlopen(
    "https://raw.githubusercontent.com/aktagon/llm-pricing-feed/master/dist/latest.json"))
price = {m["id"]: m for m in feed["models"]}

m = price["anthropic/claude-opus-4-8"]
cost = (in_tokens * m["input"] + out_tokens * m["output"]) / 1_000_000  # unit = per 1M
```

### Look up one model (jq)

```bash
curl -sL https://raw.githubusercontent.com/aktagon/llm-pricing-feed/master/dist/latest.json \
  | jq '.models[] | select(.id=="anthropic/claude-opus-4-8")'
```

### Reproducible builds

Pin to a dated snapshot instead of `latest.json` — it never changes:

```
dist/v1/snapshots/2026-06-23.json
```

Notes: the feed refreshes once a day (not intra-day live); v1 covers token-priced models —
image/video/speech and per-request pricing are out of scope (see `dist/rejected.json`).
