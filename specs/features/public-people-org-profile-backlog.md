# Public People, Organization, and Profile Backlog

## Status

Status: Backlog

Owner issue: PET-19

This spec records confirmed public people, user profile, and organization
surfaces as deferred backlog work. PET-19 does not accept or implement any new
SDK or CLI behavior.

## Endpoint Evidence

- Evidence sources: `docs/api-inventory.md`; current alphaXiv web bundle fetched
  from `https://www.alphaxiv.org/assets/main-qrK10zwy.js` on May 10, 2026;
  unauthenticated live probes against representative public routes.
- Existing implemented organization endpoints remain documented in
  `specs/features/search-discovery-feed.md`: `GET /organizations/v2/search` and
  `GET /organizations/v2/top`.
- `GET /users/v3/leaderboard` is used by the web app people route with query
  key `users-leaderboard`; an unauthenticated probe returned leaderboard buckets
  containing compact user records.
- `GET /users/v3/search?q={query}` is used by the web app people search UI; an
  unauthenticated probe returned `{"users": [...]}` with compact user records.
- `GET /users/v3/{id}/followers` is used by profile connection views; an
  unauthenticated probe returned `{"followers": [...]}` for a public user id.
- `GET /users/v3/{id}/following` is used by profile connection views; an
  unauthenticated probe returned `{"users": [...]}` for a public user id.
- `GET /users/v3/by-username/{username}` appears in the web bundle as a profile
  lookup route for user handles.
- `GET /users/v3/by-username/{username}/profile-page` is used by public profile
  pages and returns a user profile plus claimed papers, featured items, and
  activity.
- `GET /users/v3/{id}/citations/summary`,
  `GET /users/v3/{id}/citations/graph`, `GET /users/v3/{id}/claimed-papers`,
  `GET /users/v3/{id}/activity`, and `GET /users/v3/{id}/featured` appear in
  profile-related web bundle query functions.
- `GET /v1/users/{uid}/notes` appears in profile-related web bundle query
  functions for user notes.
- `GET /organizations/v2/by-name/{name}` is used by web bundle organization
  query code; an unauthenticated probe for `MIT` returned a compact organization
  object.
- `GET /organizations/v2/{id}` is used by web bundle organization query code; an
  unauthenticated probe for a known organization id returned a compact
  organization object.

## Deferral Rationale

- These routes expose account and identity data, so the public SDK contract must
  first define privacy boundaries, stable field names, and raw-payload retention
  rules before client helpers ship.
- Profile-page payloads aggregate several resources. Follow-up work should
  decide whether the SDK exposes one high-level profile object, lower-level
  route methods, or both.
- Organization detail support overlaps with existing search/top organization
  helpers. Follow-up work should avoid duplicate method names and define clear
  resolution behavior for slug, name, and id inputs.
- User activity, notes, featured items, citation graphs, and claimed papers are
  each separately paginated or shape-sensitive enough to warrant accepted
  per-route examples before implementation.
- CLI commands are deferred until the Python API names and output shapes are
  accepted.

## Acceptance Criteria

- Future implementation must keep these routes read-only unless a separate
  accepted spec covers authenticated profile mutation.
- The accepted spec must document method names, CLI command names if any,
  identifier inputs, pagination or offset parameters, normalized output types,
  empty-result behavior, auth behavior, and error handling.
- User/profile output must not expose or normalize private account fields unless
  live public unauthenticated evidence proves that alphaXiv intentionally exposes
  them.
- Organization helpers must distinguish search/top lists from exact organization
  detail lookup by id, name, or slug.
- Profile aggregation must document whether claimed papers, featured items,
  activity, notes, followers, following, and citation graph data are fetched
  together or independently.
- No SDK or CLI behavior may ship until a future issue moves this backlog spec
  to Accepted.

## Validation Commands

```bash
uv run python scripts/check_specs.py
uv run pytest
```
