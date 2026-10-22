# Operations guide

Language: **English** | [Tiếng Việt](./operations.vi.md)

This guide covers daily operations across Hosted Views (`/devin`, `/antigravity`), Native delegation (`hivemind-delegate`, `delegate-devin`), creation policies, targeted session option mediation, and a single persisted health check (ranked preflight stays internal to `hivemind-delegate` routing — explicit engine checks run only through `hivemind-doctor`).

---

## 1. Work in Hosted Views (/devin and /antigravity)

Each Hosted View route (`/devin` or `/antigravity`) connects to a TUI-owned **Lane** keyed by `(Harness, cwd)`. A lane outlives route navigation: pressing **esc** returns to OpenCode home while your bound session keeps running in the background.

### Workspace-first Session picker
When you open an unbound route or press **ctrl+o** on an idle bound lane, the **Session picker** displays sessions reported by `session/list`:
- **`+ New session in <cwd>`:** Always pinned at the top so you can start a fresh session in the current workspace directory with one keystroke.
- **Current workspace:** Existing sessions whose recorded directory matches your current workspace `cwd`, grouped by recency.
- **Other workspaces:** Sessions created in other directories on your machine or account, allowing cross-directory **Continuation** (`session/load`). Each row displays its workspace path, last-active time, `●` bound marker, and `⚿` lock badge (`isLocked` / `lockHolderPid`) when held by another live client.
- **Bound `cwd` header visibility:** Once a session binds via `session/new` or `session/load`, the Hosted View header displays the bound session's workspace `cwd` alongside the model, mode, and thinking badges so cross-directory continuations are always unambiguous.

---

## 2. Use origin-labelled / and @ completion in the Composer

In the Hosted View **Composer**, typing a leading `/` (or pressing **ctrl+g**) and typing `@` open **Origin-labelled completion** where every candidate carries an explicit source badge:

| Origin Badge | Source | Behavior on Selection & Submit |
|---|---|---|
| `[foreign command]` | Advertised by the bound foreign ACP session via `available_commands_update`. | Inserts `/<command> ` into the Composer without auto-sending; submitting sends it as an ordinary `session/prompt` text block. If the session advertises zero commands, the popover shows `[foreign command] no commands — this session has not reported any` without fabricating commands or trapping Enter. |
| `[client control]` | Built-in Hosted View controls (`sessions`, `model`, `mode`, `thinking`, `filter`, `help`). | Opens the local picker, tag filter, or shortcut overlay inside the OpenCode TUI with zero `session/prompt` wire traffic. |
| `[host skill]` | Workspace `SKILL.md` files discovered under `.agents/skills`, `.devin/skills`, `.gemini/skills`, and `.opencode/skills`. | Stages an `@<skill-path>` token in the Composer that resolves on submit to a canonical `resource_link` (`file://` URI). Never executes `!` shell lines or claims OpenCode `skill` tool runtime portability. |
| `[host reference]` | Declared local `path` entries under `references` in `<workspace>/opencode.json`. | Stages an `@<reference-path>` token that resolves on submit to a canonical `resource_link` (`file://` URI). |

For `@` file mentions:
- Typing `@<relative-path>` autocompletes files inside the workspace root (symlinks are realpath-checked to prevent escaping traversal).
- Typing an explicit absolute `@/<path>` to an existing local file or directory emits a `resource_link` (`file://` URI) content block while preserving the foreign agent's own permission boundary.

---

## 3. Delegate tasks with hivemind-delegate and delegate-devin

Your primary OpenCode agent can delegate work through two native delegation tools while leaving the builtin `subagent` tool unchanged:

| Tool | Routing Behavior | Typical Use |
|---|---|---|
| `hivemind-delegate` | Uses **Preflight** over your enabled **Harness rank** when `model` is omitted, or routes directly to the provider named in `model` (`devin-native/*` or `antigravity-native/*`). An optional `harness` field (such as `harness: "antigravity"`) pins the call to that registered Harness. | General delegation following your configured Harness preference order, or an explicit Harness pin. |
| `delegate-devin` | **Pinned alias** fixed to `devin-native`. | Tasks explicitly targeting Devin. |

Execution and continuation rules:
- **Foreground vs. background:** `background: false` (default) waits for the child turn to complete and returns the result; `background: true` launches a background **Native child** (`ses_*`) and delivers its completion to the parent session.
- **Concurrency and opt-in limits:** Independent children overlap freely in foreground or background with no default plugin concurrency cap. If you configure opt-in per-mode limits (`limits.foreground`, `limits.background` in `config/delegate`), calls exceeding the limit fail fast before child creation.
- **Child continuation and inventory:** Pass a child's OpenCode `sessionID` (`ses_*`) to continue that exact Native child on its bound foreign trajectory. Use `devin-children` with `action: "list"` and `action: "inspect"` to inspect direct children and their `foreign.sessionID` binding provenance.

---

## 4. Configure future-child delegation policy

The plugin keeps future-child delegation defaults in `config/delegate` (manageable via `/hivemind-setup`):

### Future-child delegation policy (`delegate.<harness>`)
- Controls optional `models` (a **Model slot chain** with 1 primary slot and up to 2 ordered fallback slots) and per-mode admission `limits`. An advertised `model#variant` selects effort for that model. Standalone `thinking` policy is retired; keeping models unset preserves foreign defaults.
- Applies **exclusively** to newly created Native children during the **Bound-but-unprompted child** window (after `session/new` and binding persistence, before the first `session/prompt`).
- If the primary model slot is unadvertised in the fresh `session/new` catalog, admission advances to the next configured slot in the chain; if all configured slots fail validation, the call fails closed before sending any prompt.
- Existing, running, or continued (`session/load`) Native children are never mutated, re-slotted, or rerouted.

### Hosted sessions perform zero automatic config writes
- The retired `HostedCreationPolicy` / `hosted.<harness>` document block no longer applies: `session/new` binds the session's advertised catalog as-is and emits **zero** `session/set_config_option` or `session/set_mode` calls. A legacy `hosted.<harness>` block in `config/delegate` is preserved as untouched user data and never drives a write.
- Configure a bound session's `model`, `mode`, or `thinking` through the interactive pickers or the agent-mediated targeted request in §5.
- Reopening an existing session via `session/load` (**Continuation**) likewise never writes and never overwrites the session's existing settings.

---

## 5. Mediate current Hosted session options safely

To inspect or change `model`, `mode`, or `thinking` on an **already-bound** Hosted session, you have two supported paths:

1. **Direct TUI keyboard shortcuts:** Inside `/devin` or `/antigravity`, press **ctrl+m** (model picker), **ctrl+e** (mode picker), or **ctrl+t** (thinking effort picker on Devin) to select from the session's live wire-advertised `configOptions`.
2. **Agent-mediated targeted request (`/hivemind-setup`):** Ask your OpenCode agent in English or Vietnamese, for example:
   - `"Change the model in my current Antigravity session"`
   - `"Đổi model trong phiên Antigravity hiện tại"`

How targeted mediation protects your active sessions:
- The TUI Lane registers an authenticated capability handle (`registerOwnerHandle`) at the plugin's RPC service registration location (`pluginLocation`) while reporting its workspace `cwd` in the payload, keeping the public `handleId` separate from its unguessable `handleSecret`.
- When you ask the agent to inspect or change current session options, the server broker addresses the request to the matching `targetHandleId`. Non-target TUI instances ignore the event.
- If no matching TUI Lane is bound, the tool refuses immediately with `tui-owner-absent`. If multiple TUI instances hold the same `(Harness, cwd)` slot, it refuses with `ambiguous-owner-conflict`.
- Before applying any mutation to the live wire session, the target TUI requires interactive human confirmation (`ui.dialog.confirm`) and verifies a final guard over the session binding, `catalogFingerprint`, and uncancelled status.

---

## 6. Run one health check, then interpret 5-tier readiness

You do not need a start-of-session check: the stored setup is the state. When something looks broken, `hivemind-setup` with `action: "health"` answers in one static read — `ready`, `action-required` or `blocked` with one remedy — and reuses the persisted setup instead of re-walking it. `hivemind-doctor` (`action: "check"`) stays the diagnostic when you want per-harness readiness detail:
- **Rank-preserving fail-fast routing (internal):** `hivemind-delegate`'s internal preflight evaluates enabled Harnesses in your exact configured `rankOrder` and stops at the highest-ranked available candidate. It never runs speed benchmarks, never reorders Harnesses by latency, and never spawns test Native children or paid prompts. Direct engine contact stays with `hivemind-doctor`: initialize checks are bounded, and `depth: "session"` requires `consent: true`.

**Tool surface (CodeMode):** `hivemind-setup`, `hivemind-doctor` and `hivemind-hosted` are discovered through the host's `execute` tool, not as named provider tools — the agent runs `search({query})` inside `execute` and calls the exact returned path. A blocked call names the exact missing scope (`execute` itself, or the named tool/action inside it); an explicit `deny` wins over any saved allow card. Any `opencode.json` permission edit is applied only with your consent, at the project or global scope you choose, preserving unrelated rules and plugin order — never a wildcard `hivemind-*` allow.

Every health, next-step and doctor report classifies Harness readiness into one of five honest tiers (`ReadinessTier`):

| Readiness Tier | What Was Checked | What Remains Unproven |
|---|---|---|
| `static-checked` | `acpBinary` exists and is executable on disk; local config and Recorded sign-in state read with zero subprocess spawns. | Does not prove the binary launches, completes ACP handshake, is authenticated with the vendor, or has billing quota. |
| `initialize-checked` | Subprocess launched and completed the bounded ACP `initialize` handshake with zero session creation. | Does not prove live account authentication, model catalog availability, or prompt billing quota. |
| `session-checked` | Explicit user-consented (`consent: true`) bounded `<=15s` zero-prompt `initialize` -> `session/new` check completed and tore down the engine, stamping `CatalogEvidence` (`catalog-evidence/<harnessId>`). | Proves live `session/new` catalog admission at `observedAt`, but does not prove live prompt quota or external MCP tool health. |
| `blocked` | `acpBinary` is missing/non-executable, Recorded sign-in state is `signed-out`, or configuration is malformed. | Cannot run until remediated via setup or `hivemind-doctor`. |
| `unknown` | Harness is unprofiled or untracked. | No readiness facts established. |

**Scope-stamped `CatalogEvidence`:** When a consented `session-checked` probe or live session observes a wire catalog, the plugin caches a non-authoritative `CatalogEvidence` hint stamped with `(harness, acpBinary, authEpoch, cwd, observedAt)`. Setup and doctor use this hint only while its stamp matches the current binary, `authEpoch`, and workspace; runtime creation still validates against the live wire catalog.

---

## 7. Related guides

- First-time installation, `/hivemind-setup` onboarding, and `opencode/big-pickle` privacy notice: [Quickstart tutorial](./quickstart.md) ([Tiếng Việt](./quickstart.vi.md))
- Doctor troubleshooting, vendor-auth-only boundaries, permission mediation, cancellation, and recovery: [Support and safety reference](./support.md) ([Tiếng Việt](./support.vi.md))
- Environment comparison table and prompt prefixes (`/`, `@`, `!`, `#`): [Hosted CLI capability reference](./hosted-cli.md)
- Repository overview, glossary, and security policy: [README.md](../README.md), [PROJECT.md](../PROJECT.md), [ROADMAP.md](../ROADMAP.md), [GLOSSARY.md](../GLOSSARY.md), and [SECURITY.md](../SECURITY.md)
