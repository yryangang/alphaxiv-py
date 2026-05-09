# Authenticated Assistant Usage and Context Backlog

## Status

Status: Backlog

Owner issue: PET-19

This spec records confirmed authenticated assistant usage and context-window
surfaces as deferred backlog work. PET-19 does not accept or implement any new
SDK or CLI behavior.

## Endpoint Evidence

- Evidence sources: `docs/api-inventory.md`; current alphaXiv web bundle fetched
  from `https://www.alphaxiv.org/assets/main-qrK10zwy.js` on May 10, 2026;
  unauthenticated live probes against authenticated routes to confirm auth
  boundaries.
- Existing implemented assistant endpoints remain documented in
  `specs/features/authenticated-assistant.md`: assistant session list, message
  history, chat streaming, URL metadata, current user model preference, and
  model preference updates.
- `GET /assistant/v2/{llmChat}/context-window-usage` appears in the web bundle
  with query key `["assistant", "chat", llmChat, "context-window-usage"]`; an
  unauthenticated probe returned `401` with `Missing Authorization`.
- `GET /assistant/v2/usage` appears in the web bundle with query key
  `["assistant", "usage"]`; an unauthenticated probe returned `401` with
  `Missing Authorization`.
- `GET /assistant/v2/usage/history` appears in the web bundle with query key
  `["assistant", "usage", "history"]`; an unauthenticated probe returned `401`
  with `Missing Authorization`.
- `GET /assistant/v2/usage/activity` appears in the web bundle with query key
  `["assistant", "usage", "activity"]`; an unauthenticated probe returned `401`
  with `Missing Authorization`.

## Deferral Rationale

- Usage and context-window payloads are account-specific and likely tied to plan,
  provider, model, or billing state; implementation must not infer those
  semantics from route names alone.
- Context-window usage is chat-scoped, but the accepted contract still needs live
  authenticated examples for valid chat ids, missing chat ids, deleted chats, and
  chats owned by another account.
- The usage endpoints likely have time ranges or aggregation semantics. Those
  must be documented from authenticated payloads before the SDK normalizes them.
- CLI display would need careful wording so users do not mistake usage metadata
  for a complete billing ledger or model catalog.
- These routes do not block the already implemented assistant chat, history,
  model preference, or URL metadata surface.

## Acceptance Criteria

- Future implementation must require authentication and preserve the existing
  browser-backed auth fallback where API-key access is insufficient.
- The accepted spec must include authenticated example payloads for usage,
  usage history, usage activity, and context-window usage.
- The accepted spec must define normalized Python types, raw-payload retention,
  timestamp and unit handling, and missing-data behavior.
- Context-window helpers must document ownership errors, not-found errors, and
  whether the method accepts only remote assistant chat ids.
- Usage helpers must document whether values are per-model, per-provider,
  per-account, per-plan, per-chat, or aggregated across those dimensions.
- CLI commands, if accepted, must be clearly separate from chat commands and must
  not claim to expose complete billing or quota semantics unless alphaXiv
  payload evidence supports that wording.
- No SDK or CLI behavior may ship until a future issue moves this backlog spec
  to Accepted.

## Validation Commands

```bash
uv run python scripts/check_specs.py
uv run pytest
```
