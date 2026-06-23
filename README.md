# llm-pricing-feed

Generated daily LLM API pricing feed. Produced by the `publish-feed` workflow in the
private `aktagon/llm-pricing` repo (ADR-003). Do not edit by hand — every commit is a
scheduled publish.

- `dist/latest.json` — current normalized feed
- `dist/v1/snapshots/<date>.json` — immutable dated snapshots (append-only history)
- `dist/ATTRIBUTION.md` — source attribution (models.dev, MIT)
- `data/store/` — append-only store (the audit trail)

Clone: `git clone https://github.com/aktagon/llm-pricing-feed.git`
