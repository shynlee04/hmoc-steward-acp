# Changelog

## 0.4.0 — 2026-10-22

Breaking pre-1.0 cleanup.

### Removed / unsupported

- `devin_run` and its one-shot `devin` CLI interface are removed: `-p`,
  `--cloud`, `--agent-type`, `--permission-mode` and
  `--respect-workspace-trust`, plus the one-shot kill/timeout path, are
  unsupported **as removed one-shot interface only**. The retained
  `devin_session` tool still takes its own `cloud` and `agent_type` inputs
  through the shared `devinAcpArgv` builder — those flags are gone with the
  one-shot CLI, not with the ACP session surface. The one-shot
  `--respect-workspace-trust` behavior and the exact one-shot kill/timeout
  semantics have **no replacement** on the ACP surface.
- The duplicate `devin_doctor` tool is removed with no aliases or
  compatibility shims; `hivemind-doctor` is the sole setup-diagnostic
  doctor surface — engine checks are doctor verbs, not setup verbs.
  `devin_session` (persistent ACP sessions), `delegate-devin`,
  `hivemind-delegate`, `devin-children` and `hivemind-setup` remain the
  public agent surface.

### Changed

- `hivemind-setup` exposes exactly three actions — `health`, `next`,
  `choose`. `health` and `next` are static reads with zero writes and zero
  engine spawns; `choose` remains the only mutation and persists settings
  only when a setting actually changes. `status`, `verify`, `probe` and
  `preflight` leave the public surface; engine checks run through
  `hivemind-doctor` under its existing consent guard.
- The three situational tools — `hivemind-setup`, `hivemind-doctor` and
  `hivemind-hosted` — are reached through OpenCode's `execute` (CodeMode)
  surface instead of as named tools: the agent searches inside `execute`
  synchronously and calls the exact path returned. A call needs both the
  outer `execute` permission and the named leaf tool/action permission; an
  explicit `deny` wins over any saved allow card.
- The opt-in Freebuff delegation lane opens a server-issued session lease
  with a session-scoped wallet spend limit per run; cancelling a turn stops
  the turn but does not revoke that upstream window. Its credential may
  egress only to a first-party HTTPS endpoint or loopback. `0.3.0` resolved
  the credential from `CODEBUFF_API_KEY` alone; `0.4.0` intends native
  Freebuff CLI login with an existing registered device-key fallback,
  pending safety review; a live entitled turn remains unverified.

## 0.3.0

- Repair OpenCode 2.1.2 support: the plugin loads and binds correctly on
  the v2.1.2 host.
- Fix the first-run bind path so a fresh install reaches a hosted session.
- Pin the Freebuff delegate bundle shipped in the package.

## 0.2.1

- Rename the published CLI to `bin/hmoc-steward-acp.mjs`. The npm file list no
  longer shows the old `hivemind-steward-oc-devin` name.
- Rewrite the README as the page agents actually read: install, one-time setup,
  daily use, health, requirements, failure recovery, and safety. English and
  Vietnamese stay side by side.
- Add a support section pointing at the project portal, Ko-fi and Buy Me a
  Coffee. Donations are optional and unrelated to installation.
- Keep the plugin id and its `config/delegate` namespace unchanged so saved
  setup survives the package rename.

## 0.2.0

- Compile the npm TUI with the Solid transform before packing. The exported
  `dist/tui.js` loads from `node_modules` without React or repository JSX config.
- Check the extracted npm package by opening a hosted view and rendering a
  streamed reply from the deterministic ACP test agent.
- Keep the saved setup when unchanged, allow explicit setting changes, and
  finish setup when foreign model defaults are selected.
- Install from npm through OpenCode. GitHub is the front page and user guides,
  not an alternative installation source. Correct both language guides for
  model variants, ACP authentication and supported managed server bundles.
- Advanced tools still have direct schemas. Code Mode thinning and a dedicated
  TUI setup wizard are deferred, not claimed as shipped.

## 0.1.0

Initial npm release. Registry readback now confirms `0.1.0` is published.
Its raw TSX package entry can fail with a missing React import; use `0.2.1`.

Onboarding is now persistent and one-step:

- `hivemind-setup {action:"health"}` is the only start-of-use check: one
  static read (filesystem stat + KV read + option parse; zero spawns, zero
  writes) answering `ready`, `action-required` or `blocked` with the one
  remedy. The stored `config/delegate` document is the receipt, so a ready
  setup is never re-walked at session start.
- `/hivemind-setup` asks one question per setting with the host `question`
  tool, in English or Vietnamese, and only runs the doctor, install/sign-in
  or an opt-in probe when health names them.
- The installer accepts `--scope global|project`; entries append after
  existing plugins so a later plugin can override an earlier one, and
  conflicting scopes refuse before any write.
- README and quickstart lead with a plain-language English/Vietnamese
  walkthrough of what you type, what you need and what persists.

Initial public drop of the OpenCode V2 plugin: `/devin` and `/antigravity`
hosted routes with live ACP streaming, session picker and session swap,
permission cards, `@file` mentions, dock overlays, palette commands, and the
`hivemind-delegate`, `delegate-devin`, `devin-children`, and `hivemind-doctor`
agent tools (legacy `devin_run` / `devin_session` retained).

Notable hardening in this drop:

- `auto_approve` on the delegation tools now defaults to **false** and picks
  permission options by label semantics — deny-shaped labels are never
  auto-selected.
- `@` mention containment resolves symlinks before the cwd check.
