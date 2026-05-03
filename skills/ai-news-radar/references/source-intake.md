# Source Intake Reference

Use this when evaluating a new information source for AI News Radar.

## Decision Order

1. Prefer official RSS/Atom/JSON feeds.
2. Prefer public GitHub-generated feeds over rebuilding another project's crawler.
3. Use OPML for private or user-specific feeds.
4. Use a built-in fetcher only when the source improves the public default.
5. Avoid login, cookies, browser automation, private inboxes, and committed secrets.

## Source Classes

| Source class | Preferred integration | Default suitability |
| --- | --- | --- |
| Official RSS/Atom | OPML or built-in official source | High |
| GitHub generated JSON/RSS | Built-in fetcher reading raw GitHub URLs | High if curated and timestamped |
| GitHub releases/commits | Atom feed | Medium to high |
| Public changelog page | Focused `requests` + BeautifulSoup fetcher | Medium |
| Aggregator site | Existing custom fetcher pattern | Medium |
| Newsletter archive | RSS if available; stable archive page otherwise | Medium |
| X/Twitter public timeline | Curated central feed or optional X API adapter | Low as default |
| Email inbox | Do not default; require explicit user-owned bridge/API | Low |
| WeChat/private social | Optional only, bridge-dependent | Low |

## GitHub Project Intake

When a user gives a GitHub repo:

1. Read `README*`, `SKILL.md`, `config/*`, and `.github/workflows/*`.
2. Look for generated outputs: `feed*.json`, `latest*.json`, `rss.xml`,
   `atom.xml`, `state*.json`, or a `gh-pages` branch.
3. Inspect output schema:
   - title/text field
   - canonical URL
   - timestamp (`createdAt`, `publishedAt`, `generatedAt`, `pubDate`)
   - source/person/feed name
   - stable IDs for dedupe
4. Inspect workflow secrets and dependencies:
   - If the repo uses API keys centrally but publishes public feed files, consume
     the feed files.
   - If every user must provide keys, treat it as an optional advanced source.
5. Test raw URLs from GitHub Actions-friendly locations:
   - `https://raw.githubusercontent.com/<owner>/<repo>/<branch>/<file>`
   - public GitHub Pages URL if provided
6. Add a fetcher only after a source-only probe succeeds locally.

## X/Twitter Rules

Do not depend on public RSSHub/XCancel/Nitter routes as a default source unless
the user explicitly accepts instability. They can work briefly and then time out.

Stable patterns:

- Read a curated central feed that already uses official X API, like
  Follow Builders.
- Add a self-hosted optional X adapter using `X_BEARER_TOKEN`; skip cleanly when
  the token is missing.

For optional X API adapters:

- Store tokens only in GitHub Secrets or environment variables.
- Never print or commit tokens.
- Exclude retweets/replies by default.
- Cap per-account items.
- Record rate-limit or permission failures in `source-status.json`.

## Newsletter Rules

Prefer public archive feeds:

- Substack RSS when available.
- Beehiiv/public archive pages when RSS is blocked.
- Jina Reader only when direct fetch is blocked and the archive is public.

Avoid private inbox ingestion as a default feature. Email requires OAuth, IMAP,
App Passwords, forwarding, or third-party bridges and raises privacy concerns.

## Built-In Fetcher Pattern

Add focused code to `scripts/update_news.py`:

```python
def fetch_example(session: requests.Session, now: datetime) -> list[RawItem]:
    resp = session.get("https://example.com/feed.json", timeout=20)
    resp.raise_for_status()
    data = resp.json()
    out: list[RawItem] = []
    for item in data.get("items", []):
        published = parse_date_any(item.get("publishedAt"), now)
        if not published:
            continue
        out.append(
            RawItem(
                site_id="example",
                site_name="Example",
                source=item.get("source") or "Example",
                title=maybe_fix_mojibake(item["title"]),
                url=item["url"],
                published_at=published,
                meta={},
            )
        )
    if not out:
        raise ValueError("No Example items parsed")
    return out
```

Then register it in `collect_all`, update docs, and add tests for parser behavior.

## Validation Checklist

Run:

```bash
python -m py_compile scripts/update_news.py
pytest -q
python scripts/update_news.py --output-dir /tmp/ai-news-radar-data --window-hours 24 --archive-days 21
```

Inspect `/tmp/ai-news-radar-data/source-status.json`:

- source appears with `ok: true`
- `failed_sites` is empty or the failure is expected and documented
- item count is plausible
- AI-focused view is not flooded with off-topic items
