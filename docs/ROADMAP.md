# AI News Radar Roadmap

## v0.3.0 — Source Overlap Check

Goal: before adding a new public default source, evaluate whether its recent items are mostly duplicates of the existing source set.

Status: implemented as a maintainer-facing intake tool.

### What ships

- `scripts/evaluate_source_overlap.py`
  - Fetches a candidate RSS/Atom source.
  - Compares recent candidate items against `data/archive.json` or another baseline JSON.
  - Reports hard duplicates, possible duplicates, unique items, top overlapping sources, and a recommendation.
- `tests/test_source_overlap.py`
  - Covers URL exact matches, title similarity, duplicate-rate statistics, threshold recommendations, and the small-sample guard.

### Default decision thresholds

- `< 35%` duplicate rate: `accept_default`
- `35%–65%` duplicate rate: `watchlist`
- `>= 65%` duplicate rate: `skip_duplicate`
- `< 5` recent candidate items: always `watchlist`, because the sample is too small for automatic rejection.

### Example

```bash
python scripts/evaluate_source_overlap.py \
  --source-url https://aihot.virxact.com/feed.xml \
  --source-name "AI HOT" \
  --site-id aihot_candidate \
  --baseline data/archive.json \
  --lookback-days 7 \
  --output /tmp/aihot-overlap.json
```

The tool is advisory only. It does not change `update_news.py`, does not remove any items, and does not publish the report to GitHub Pages by default.

## v0.4.0 — Story Merge / Event Cluster

Goal: move beyond simple filtering and represent the same event as one story with multiple source references.

### Planned direction

- Keep the current filter-first behavior as the safe default.
- Add a story clustering layer after source normalization and before page payload generation.
- Preserve one primary title plus secondary source references, instead of randomly choosing one duplicated item.
- Show repeated coverage as a trust signal: "多个来源报道了这件事".

### Non-goals for v0.3.0

- No LLM semantic clustering.
- No cross-language deep semantic matching.
- No automatic deletion of sources based only on overlap score.
- No change to the public page layout.
