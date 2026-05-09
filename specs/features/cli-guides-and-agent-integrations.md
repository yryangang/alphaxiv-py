# CLI Guides And Agent Integrations

## Status

Status: Implemented

Owner issue: PET-12 records the existing implementation.

This spec records the current implemented guide, skill, and agent integration
surface. PET-12 is documentation-only and does not change SDK or CLI behavior.

## CLI Surface

- `alphaxiv guide` lists task-level workflow guides.
- `alphaxiv guide research` explains discovery, shortlist inspection, and when
  to use the authenticated assistant.
- `alphaxiv guide paper` explains how to choose between paper metadata,
  abstract, summary, overview, full text, PDF, resources, similar papers, and
  saved context commands.
- `alphaxiv guide assistant` explains assistant start/reply/history/model usage
  after deterministic retrieval.
- `alphaxiv guide feed` explains feed filters, topic and organization discovery,
  and ranking modes.
- `alphaxiv skill install [--target all|codex|claude-code|opencode]
  [--scope user|project] [--force]` installs packaged integration assets.
- `alphaxiv skill status [--scope user|project|all] [--json]` reports install
  path, managed/source-copy status, installed version, expected CLI version, and
  whether a managed install needs an update.
- `alphaxiv skill show [--target source|codex|claude-code|opencode]` prints the
  canonical Codex skill source or a target-specific bundle.
- `alphaxiv skill uninstall [--target all|codex|claude-code|opencode]
  [--scope user|project] [--yes]` removes only managed integration installs.
- `alphaxiv agent show <codex|claude-code|opencode>` prints target-specific
  guidance, user/project install paths, and included source files.

## Integration Surface

- Codex integration installs the repository `skills/alphaxiv/` tree, including
  `SKILL.md`, `agents/openai.yaml`, and reference workflow files.
- Claude Code integration installs
  `integrations/claude-code/alphaxiv-research.md` as a Markdown subagent with
  YAML frontmatter.
- OpenCode integration installs the
  `integrations/opencode/commands/alphaxiv/` command pack.
- User-scope Codex installs go to `$CODEX_HOME/skills/alphaxiv` when
  `CODEX_HOME` is set, otherwise `~/.codex/skills/alphaxiv`.
- Project-scope Codex installs go to `./skills/alphaxiv`.
- User-scope Claude Code installs go to
  `~/.claude/agents/alphaxiv-research.md`; project-scope installs go to
  `./.claude/agents/alphaxiv-research.md`.
- User-scope OpenCode installs go to
  `~/.config/opencode/commands/alphaxiv/`; project-scope installs go to
  `./.opencode/commands/alphaxiv/`.
- Installed assets receive an `alphaxiv-py v<version>` marker so `skill status`
  and `skill uninstall` can distinguish managed installs from unmanaged files.

## Endpoint Evidence

- Endpoint evidence source: this surface is local CLI and file integration
  behavior; it does not add alphaXiv HTTP endpoints.
- Implementation source: `src/alphaxiv/cli/guide.py`,
  `src/alphaxiv/cli/skill.py`, `src/alphaxiv/cli/agent.py`,
  `src/alphaxiv/agent_assets.py`, and `src/alphaxiv/catalog.py`.
- Packaged asset sources: `skills/alphaxiv/`,
  `integrations/claude-code/alphaxiv-research.md`, and
  `integrations/opencode/commands/alphaxiv/`.
- Test evidence: `tests/unit/test_agent_ux.py::test_guide_root_and_research_output`,
  `tests/unit/test_agent_ux.py::test_skill_install_status_show_and_uninstall`,
  and `tests/unit/test_agent_ux.py::test_agent_show_and_codex_skill_source_match`.

## Inputs And Outputs

- `guide` commands render human-readable workflow prose from
  `src/alphaxiv/catalog.py`.
- `skill install` defaults to all supported targets and user scope.
- `skill install --force` may overwrite unmanaged existing targets; without
  `--force`, unmanaged targets are skipped or reported as failures.
- `skill status --json` emits normalized records with `target`, `label`,
  `scope`, `path`, `installed`, `managed`, `source_copy`, `version`,
  `expected_version`, and `needs_update`.
- `skill show --target source` prints the Codex skill source; target-specific
  values render the complete installable bundle with file headings when the
  bundle has multiple files.
- `skill uninstall` prompts unless `--yes` is provided and refuses to delete
  unmanaged installs.
- `agent show` reads integration metadata and source bundle contents without
  writing files.

## Acceptance Criteria

- CLI users can discover workflow guides without consulting README prose.
- CLI users can install, inspect, show, and uninstall supported Codex,
  Claude Code, and OpenCode integration assets.
- Managed install detection, version reporting, and source-copy detection remain
  available for automation.
- Agent guidance documents deterministic retrieval first and assistant usage
  after search, feed, or paper inspection.
- This surface remains local-file and CLI-only; PET-12 does not add or change
  network endpoint behavior.
- PET-12 makes no SDK or CLI behavior changes.

## Validation Commands

```bash
uv run python scripts/check_specs.py
uv run pytest tests/unit/test_agent_ux.py -q
uv run pytest
```
