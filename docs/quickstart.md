# Quickstart tutorial

Language: **English** | [Tiếng Việt](./quickstart.vi.md)

This tutorial walks you from a clean [OpenCode V2](https://opencode.ai/v2/docs) installation to your first hosted session (`/devin`, `/antigravity`) and native delegation (`hivemind-delegate`, `delegate-devin`) using the `hmoc-steward-acp` plugin.

---

## 1. Check prerequisites per Harness

You need an OpenCode V2 host in the supported range: `opencode --version` reports v2.1.4 or later within v2 (below v3) on your `PATH`. A V1 host cannot load the plugin's TUI entry point, and v2.0.x is below the supported floor.

The installer and setup workflow verify each **Harness** independently. You can configure Devin alone, Antigravity alone, or both together:

| Harness | Provider ID | Required `acpBinary` (ACP stdio server) | Optional `cliCompanion` (terminal CLI) | Authentication |
|---|---|---|---|---|
| **Devin** | `devin-native` | `devin` on `PATH` (or `DEVIN_BIN` / `harnesses.devin.bin`) | Same CLI; development baseline `3000.11.92` | `devin auth login` |
| **Antigravity** | `antigravity-native` | Official ACP server `1.4.0`, `agy_acp_server.par` on macOS or `agy_acp_server.exe` on Windows; managed install or `harnesses.antigravity.bin` | `agy` CLI `1.4.2` is optional and separate | ACP server's advertised `authenticate` flow |

Key rules for executable resolution:
- An Antigravity-only setup succeeds without requiring the `devin` CLI on your machine.
- Antigravity tracks `acpBinary` (`agy_acp_server.par`) and `cliCompanion` (`agy`) separately. The standalone `agy` CLI is never used as a fallback for `acpBinary`.
- Explicit executable overrides (`harnesses.devin.bin`, `harnesses.antigravity.bin`) must point to an existing executable file or valid `PATH` binary; directory paths and non-executable files fail closed.

---

## 2. Install hmoc-steward-acp

Documentation for `0.4.1`. Install it through OpenCode. This command adds it globally:

```bash
opencode plugin add hmoc-steward-acp@0.4.1
```

`0.4.1` is a patch on `0.4.0`, which builds on `0.3.0`. `0.3.0` carries the OpenCode 2.1.2 support repair, the first-run bind fix, and the pinned Freebuff delegate. Earlier releases (`0.2.2`, and `0.2.2-test.5` on the `testing` tag) predate the 2.1.2 repair — do not use them on an OpenCode 2.1.2 host.

For project-only installation, add `"hmoc-steward-acp@0.4.1"` to `plugins` in `<repo>/opencode.json` instead. Global configuration lives at `~/.config/opencode/opencode.json`. Preserve other plugins and settings. Replace the old entry when upgrading; do not add a duplicate. The npm install needs no GitHub checkout or separate package installer.

Restart OpenCode or open a new session after installation so the host loads both the server plugin and the TUI routes.

---

## 3. Onboard with the free opencode/big-pickle model

If you are setting up OpenCode V2 for the first time and have not connected a paid model provider yet, you can run the guided setup and coordinate delegated work using OpenCode's default free no-login model (`opencode/big-pickle`) alongside the bundled `hivemind-setup` skill.

> [!IMPORTANT]
> **Data privacy notice for free `opencode/big-pickle` endpoints:** Prompts, file context, and completions sent through the free no-login `opencode/big-pickle` model may be logged and retained by the endpoint provider to evaluate and train models. Do **not** submit confidential, proprietary, regulated, or secret source code, customer data, or credentials while using `opencode/big-pickle`. Switch your OpenCode host model to a private or enterprise-governed provider before working with sensitive repositories.

---

## 4. Run guided setup once — it persists

In the OpenCode TUI, run `/hivemind-setup` (or just ask in plain English or Vietnamese):

- **English:** `"Run /hivemind-setup and help me choose my defaults."`
- **Vietnamese:** `"Chạy /hivemind-setup và giúp tôi chọn mặc định."`

Your agent uses the native `question` tool; you choose. The agent runs the operations and explains each option in your language. There is no separate TUI wizard.

**0.4.0 routing:** the agent reaches `hivemind-setup` through OpenCode's `execute` surface, not as a named tool. It searches inside `execute` and calls the exact path returned. If `execute` or a leaf action is denied, the call names the exact missing scope; the plugin never widens permissions on its own. Permission edits happen only after you choose project or global scope.

1. **Health (`action: "health"`)** — one static read. `ready` means setup is already persisted: the agent says so and stops. `action-required` or `blocked` names the single remedy.
2. **Choices (`action: "next"` then `action: "choose"`)** — enable Harnesses, set the rank, and optionally pick model slots or limits. Each write is guarded by the issued step fingerprint and the document revision (`expectedVersion`); a stale answer refuses visibly and re-issues the step.
3. **Completion (`action: "health"` again)** — `ready` is the static setup finish line, not a live login or quota check. The stored `config/delegate` document is the receipt. To change it later, state the change, for example `/hivemind-setup change my harness order`.
4. **Remedies, only when named** — `hivemind-doctor` check/install/sign-in under your explicit consent. Engine checks are doctor verbs, not setup actions: 0.4.0 exposes only `health`, `next` and `choose` on setup, and an opt-in doctor check caches scope-stamped `catalog-evidence/<harnessId>`.

Scope: delegation policy is global per plugin (`config/delegate`). Host permissions and the plugin entry are per scope — global (`~/.config/opencode/opencode.json`) or project (`<repo>/opencode.json` or `.opencode/opencode.json`); ask the agent for the scope you want. Permission rules are ordered and the last match wins; installed plugins load in array order, so a later entry can override an earlier one.

---

## 5. Launch your first Hosted session

Hosted sessions let you converse directly with Devin or Antigravity inside a full-screen OpenCode TUI route:

1. Type `/devin` or `/antigravity` in the OpenCode TUI (or choose **Devin: open session** / **Antigravity: open session** from the command palette).
2. When the **Session picker** opens, select **`+ New session in <cwd>`** at the top to create a fresh session in your current workspace, or pick an existing session from **Current workspace** or **Other workspaces** to continue it via `session/load`.
3. Type your prompt in the **Composer**. Use `@` to attach workspace file mentions as `resource_link` blocks, or type `/` to browse origin-labelled completions (`[foreign command]`, `[client control]`, `[host skill]`, `[host reference]`).
4. Use **ctrl+m** (model), **ctrl+e** (mode), or **ctrl+t** (thinking effort on Devin) to adjust live **Config Options**, **ctrl+x** to cancel an in-flight turn, and **esc** to return to OpenCode home while keeping the lane alive in the background.

---

## 6. Run your first Native delegation

Native delegation lets your primary OpenCode agent hand a task to a genuine OpenCode **Native child** (`ses_*`) backed by Devin or Antigravity:

- **Rank-routed delegation (`hivemind-delegate`):** Ask your OpenCode agent:
  `"Use hivemind-delegate to inspect src/ and summarize the module boundaries."`
  When no `model` is specified, **Preflight** evaluates your enabled Harnesses in configured `rankOrder` and picks the highest-ranked available Harness.
- **Harness-pinned delegation (`delegate-devin`, or `hivemind-delegate` with `harness: "antigravity"`):** Ask your OpenCode agent:
  `"Call delegate-devin with description 'Audit retry logic', prompt 'Check src/acp.ts for error handling edge cases', and background: false."`
- Pass `background: true` to run independent children concurrently while you keep working in the parent session, or pass an existing child's `sessionID` (`ses_*`) to run a **Child continuation** on the same bound foreign trajectory.
- **Standard ACP `devin_session`** remains for persistent Devin conversations, including its `cloud` and `agent_type` inputs. It does not create an OpenCode Native child and does not join `rankOrder` routing.

A third delegate Harness, **Freebuff** (`freebuff-native`), ships as an opt-in deny-only lane: a task delegated with `harness: "freebuff"` runs in a genuine native child session on the compiled pinned-source Freebuff SDK bundled in the package, with every tool denied — zero tool effects. It never auto-enables and never takes the rank fallback; explicit opt-in is the only admission. In 0.4.0, each Freebuff run starts by opening a server-issued session lease with a session-scoped wallet spend limit; cancelling a turn stops the turn but does not revoke that upstream window. Its credential may egress only to a first-party HTTPS endpoint or loopback — the fence refuses before any Authorization header otherwise. `0.3.0` resolved the credential from `CODEBUFF_API_KEY` alone. `0.4.0` intends native Freebuff CLI login with an existing registered device-key fallback; an explicit environment key remains valid, and that path is safety-review pending. 0.4.1 status: the upstream service refuses free-mode completions with `403 free_mode_cli_required` on both free and paid catalog rows, and the published Freebuff terms restrict free-tier inference to the official interactive clients. The Freebuff harness therefore ships unconfigured/experimental; supported headless alternatives are the paid Codebuff SDK or BYOK inside the official client.

---

## 7. Next steps and related guides

- Daily workflows, session pickers, creation policies, targeted option mediation, and 5-tier readiness: [Operations guide](./operations.md) ([Tiếng Việt](./operations.vi.md))
- Doctor diagnostics, vendor-auth-only boundaries, permission mediation, cancellation, and recovery: [Support and safety reference](./support.md) ([Tiếng Việt](./support.vi.md))
- Environment comparison table and prompt prefix reference (`/`, `@`, `!`, `#`): [Hosted CLI capability reference](./hosted-cli.md)
- Repository overview and architecture: [README.md](../README.md), [PROJECT.md](../PROJECT.md), [ROADMAP.md](../ROADMAP.md), [GLOSSARY.md](../GLOSSARY.md), [SECURITY.md](../SECURITY.md), and [CHANGELOG.md](../CHANGELOG.md)
