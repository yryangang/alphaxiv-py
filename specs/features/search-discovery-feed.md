# Search, Discovery, And Feed

## Status

Status: Implemented

Owner issue: PET-9 records the existing implementation.

This spec records the current implemented search, discovery, and homepage feed
surface. PET-9 is documentation-only and does not change SDK or CLI behavior.

## SDK Surface

- `AlphaXivClient.search.papers(query, include_private=False)` calls paper
  search and returns `list[SearchResult]`.
- `AlphaXivClient.search.organizations(query)` calls organization search and
  returns `list[OrganizationResult]`.
- `AlphaXivClient.search.closest_topics(query)` calls closest-topic search and
  returns `list[str]`.
- `AlphaXivClient.search.homepage(query, include_private=False)` gathers paper,
  organization, and topic search concurrently and returns
  `HomepageSearchResults`.
- `AlphaXivClient.explore.top_organizations()` returns
  `list[OrganizationResult]` from the top organizations endpoint.
- `AlphaXivClient.explore.filter_options()` returns static feed sorts, static
  menu categories, static intervals, static sources, and live top
  organizations as `ExploreFilterOptions`.
- `AlphaXivClient.explore.search_filters(query)` gathers closest topics and
  organization search concurrently and returns `FeedFilterSearchResults`.
- `AlphaXivClient.explore.feed(...)` calls the homepage feed endpoint and
  returns `list[FeedCard]`.

## CLI Surface

- `alphaxiv search all <query> [--json]` prints homepage-style paper, topic, and
  organization search results.
- `alphaxiv search papers <query> [--json]` prints paper search results.
- `alphaxiv search organizations <query> [--json]` prints organization search
  results.
- `alphaxiv search topics <query> [--json]` prints topic suggestions.
- `alphaxiv feed filters [--json]` prints supported feed sorts, categories,
  intervals, sources, and live top organizations.
- `alphaxiv feed filters search <query> [--json]` prints live topic and
  organization filter matches.
- `alphaxiv feed list [options] [--json]` prints homepage feed cards.

All listed CLI commands use the same unauthenticated client construction as the
corresponding SDK calls. `--json` emits normalized JSON through
`src/alphaxiv/cli/serialize.py`; otherwise commands render rich tables.

## Endpoint Evidence

- Evidence source: `docs/api-inventory.md`, "Search and Discovery" and "Routes
  Currently Used By This Repository".
- Implementation source: `src/alphaxiv/_search.py`,
  `src/alphaxiv/_explore.py`, and `src/alphaxiv/cli/explore.py`.
- Test evidence: `tests/integration/test_client.py::test_search_papers`,
  `tests/integration/test_client.py::test_homepage_search_and_feed`,
  `tests/integration/test_client.py::test_feed_filter_search`,
  `tests/integration/test_client.py::test_feed_with_topic_and_github_sort`, and
  search/feed cases in `tests/unit/test_cli_endpoints.py`.

| Method | Path | SDK/CLI use | Parameters |
| --- | --- | --- | --- |
| `GET` | `/search/v2/paper/fast` | `search.papers`, `search.homepage`, `alphaxiv search papers`, `alphaxiv search all` | `q=<query>`, `includePrivate=<true|false>` |
| `GET` | `/v1/search/closest-topic` | `search.closest_topics`, `search.homepage`, `explore.search_filters`, `alphaxiv search topics`, `alphaxiv search all`, `alphaxiv feed filters search` | `input=<query>` |
| `GET` | `/organizations/v2/search` | `search.organizations`, `search.homepage`, `explore.search_filters`, `alphaxiv search organizations`, `alphaxiv search all`, `alphaxiv feed filters search` | `q=<query>` |
| `GET` | `/organizations/v2/top` | `explore.top_organizations`, `explore.filter_options`, `alphaxiv feed filters` | none |
| `GET` | `/papers/v3/feed` | `explore.feed`, `alphaxiv feed list` | `pageNum`, `pageSize`, `sort`, `interval`, optional JSON-encoded filters |

## Inputs

### Search

- `query` is passed through to the relevant endpoint as `q` or `input`.
- `include_private` defaults to `False`; paper search serializes it as
  lowercase `includePrivate=false` or `includePrivate=true`.

### Feed Filters

- Supported feed sort labels are `Hot`, `Likes`, `GitHub`, and `Twitter (X)`.
- CLI and SDK sort aliases normalize as follows:
  - `hot` -> `Hot`
  - `likes` -> `Likes`
  - `github`, `most-stars`, `stars` -> `GitHub` and imply source `GitHub`
  - `twitter`, `twitter-x`, `most-twitter-likes`,
    `most-twitter-x-likes` -> `Twitter (X)` and imply source `Twitter (X)`
- Supported feed sources are `GitHub` and `Twitter (X)`. CLI values are
  `github` and `twitter`.
- Supported intervals are `3 Days`, `7 Days`, `30 Days`, `90 Days`, and
  `All time`. CLI values use hyphens, for example `30-days` and `all-time`.
- `organizations` are passed as trimmed strings in a JSON-encoded
  `organizations` query parameter.
- `menu_categories` and `categories` are normalized to lowercase slug tokens,
  deduped, and sent as JSON-encoded `categories`.
- `subcategories` are normalized, deduped, and sent as JSON-encoded
  `subcategories`.
- `custom_categories` are normalized, deduped, and sent as JSON-encoded
  `customCategories`.
- `topics` preserve raw alphaXiv topic codes such as `cs.AI`; other values are
  normalized to lowercase slug tokens, deduped, and sent as JSON-encoded
  `topics`.
- `limit` controls `pageSize` when provided. If omitted in SDK calls, the
  endpoint receives `pageSize=20`. The CLI default is `--limit 10`.
- Feed requests always include `pageNum=0`, `sort`, and `interval`.

## Outputs

- `SearchResult` exposes `link`, `paper_id`, `title`, `snippet`, and `raw`.
- `OrganizationResult` exposes `id`, cleaned `name`, `image`, `slug`, and
  `raw`.
- `HomepageSearchResults` exposes the original `query`, `papers`,
  `organizations`, `topics`, and a `raw` dictionary containing the normalized
  raw child payloads.
- `ExploreFilterOptions` exposes `sorts`, `menu_categories`, `intervals`,
  `sources`, `organizations`, and `raw`.
- `FeedFilterSearchResults` exposes the original `query`, `topics`,
  `organizations`, and `raw` endpoint payloads.
- `FeedCard` exposes paper identifiers, title, abstract, summary, result
  highlights, publication/update dates, topics, organizations, authors, image
  URL, vote/visit/source metrics, GitHub URL, and `raw`.
- `FeedCard.link` returns `/abs/{paper_id}`.
- Feed parsing accepts either a response object with a `papers` list or a raw
  response list. Non-dictionary list items are ignored.
- If `limit` is provided, the SDK returns at most that many parsed feed cards,
  even if the endpoint returns more.

## Error Behavior

- HTTP status codes `429`, `500`, `502`, `503`, and `504` are retried by
  `ClientCore` before surfacing errors.
- Any final HTTP status code `>=400` raises `APIError`. If the JSON response
  contains `error.message` or `message`, that text becomes the error message.
- Network request failures raise `APIError` after retries are exhausted.
- Invalid JSON responses raise `APIError("Response was not valid JSON")`.
- Calling SDK methods outside `async with AlphaXivClient()` raises
  `RuntimeError("Client not initialized. Use 'async with AlphaXivClient()'.")`.
- Paper search, organization search, topic search, top organizations, and feed
  parsing return empty lists when successful endpoint payloads have an
  unexpected top-level shape.
- `closest_topics` returns an empty list when the top-level payload is not an
  object or when `data` is absent.
- `explore.search_filters` returns empty topic and/or organization lists when
  either successful endpoint payload has an unexpected shape.
- Unsupported SDK feed sort, source, or interval values raise `ValueError`
  before any HTTP request is made.
- A source-specific sort with a conflicting explicit source raises
  `ValueError`, for example `sort="most-stars"` with `source="Twitter (X)"`.
- CLI `feed list` constrains sort, source, and interval choices through Click
  before calling the SDK.

## Acceptance Criteria

- Python users can search papers, organizations, and topics without
  authentication.
- Python users can retrieve homepage-style search suggestions and feed cards.
- CLI users can run grouped `search` and `feed` commands with JSON output where
  documented.
- Feed filtering preserves existing sort, source, topic, organization, and
  limit behavior.
- The spec documents endpoints, inputs, outputs, and error behavior for the
  existing search and feed surfaces.
- PET-9 makes no SDK or CLI behavior changes.

## Validation Commands

```bash
uv run python scripts/check_specs.py
uv run pytest tests/integration/test_client.py::test_search_papers tests/unit/test_cli_endpoints.py::test_search_papers_command -q
uv run pytest
```
