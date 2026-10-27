# Support and safety reference

Language: **English** | [Tiếng Việt](./support.vi.md)

This reference documents troubleshooting with `hivemind-doctor`, the vendor-auth-only security boundary, inline permission mediation, turn cancellation, native tool settlement, and recovery guarantees in `hmoc-steward-acp`.

---

## 1. Diagnose and remediate with hivemind-doctor

You can run diagnostics conversationally through `/hivemind-setup` or by asking your OpenCode agent to invoke `hivemind-doctor`. In 0.4.0, the agent reaches `hivemind-doctor` through OpenCode's `execute` (CodeMode) surface, not as a named tool: it searches inside `execute` and calls the exact path returned. A blocked call names the exact missing scope — `execute` itself, or the named tool/action; the plugin never widens permissions on its own.

| Action | Consent Required | What It Does | Readiness & State Effects |
|---|---|---|---|
| `check` (default `check: "initialize"`) | No | Resolves `acpBinary`, checks optional `cliCompanion` (`devin` or `agy` CLI) version, runs a bounded ACP `initialize` handshake, and reads Recorded sign-in state plus `authEpoch`. | Reports `readinessTier: "initialize-checked"` (or `"blocked"` on failure) with zero session creation or prompt spend. |
| `check` with `check: "session"` | Yes (`consent: true` plus an approved native `hivemind-session-probe` ask) | Runs a bounded `<=15s` zero-prompt `initialize` -> `session/new` diagnostic check and immediately tears down the engine. | Reports `readinessTier: "session-checked"` and stamps `CatalogEvidence` (`catalog-evidence/<harnessId>`). Discloses that `session/new` may trigger vendor network auth checks, foreign MCP server startup, and durable vendor session creation. |
| `install` | Yes (approved native `hivemind-install` ask) | Downloads and verifies the managed Antigravity ACP server `1.4.0` for supported macOS/Windows targets, or provides install instructions for the Devin CLI. | Leaves state unchanged if the consent ask is declined or unavailable. |
| `sign-in` | Yes (approved native `hivemind-sign-in` ask) | Uses the ACP server's advertised `authenticate(methodId)` flow. Devin may direct you to `devin auth login`. The separate `agy` CLI login does not prove ACP-server authentication. | Records a confirmed outcome as `signed-in` and increments `authEpoch`. |
| `sign-out` | Yes (approved native `hivemind-sign-out` ask) | Updates the plugin's local sign-in record (`state: "signed-out"`) and increments `authEpoch`. | A declined or unavailable ask aborts before any state change; on approval, recycles idle engines from older generations while preserving in-flight busy turns and pending Permission Requests. |

> [!NOTE]
> **Local `sign-out` boundary:** Running `hivemind-doctor` with `action: "sign-out"` first asks for your approval through a native `hivemind-sign-out` consent prompt, then updates the plugin's local record and recycles idle engines only. It never invokes an unwitnessed ACP `logout` RPC and never deletes or modifies credentials stored by `devin` or `agy` on your machine.

---

## 2. Vendor-auth-only boundary and secret-free configuration

The plugin enforces a strict **vendor-auth-only** boundary (`REQ-HARN-25` and `RC-01`):
- **Zero credential extraction or storage:** The plugin never reads, extracts, imports, or stores macOS Keychain items, SQLite databases, OAuth access/refresh tokens, API keys, or account emails. All authentication state lives exclusively in the vendor's own credential store (`devin auth login`, `agy auth login`, or the ACP `authenticate` handshake).
- **Opaque `authEpoch` generation counter:** Whenever authentication changes (`sign-in`, local `sign-out`, or an observed `auth` Failure class on the wire), the plugin increments an opaque monotonic integer (`authEpoch`). Idle engines and cached `CatalogEvidence` snapshots stamped with an older `authEpoch` are treated as stale and refreshed on the next call.
- **Configuration is not a secret store (`RC-01`):** The parser rejects specific credential-key names. This is not detection of every secret or alternate key name. Never put credentials in `config/delegate`, plugin options, setup answers, or prompts. Follow the [Quickstart privacy notice](./quickstart.md) when using `opencode/big-pickle`.

---

## 3. Permission mediation and persistence scopes

Foreign agents request permission for sensitive actions over ACP via `session/request_permission`:

### Hosted View inline Permission Request cards
- **Human-only decisions:** Hosted lanes never auto-approve permission requests. Every request renders as an inline **Permission Request** card in the Entry Log with numbered options (`1..9` to select, **esc** to cancel).
- **Verbatim foreign options and warnings:** Cards display the exact options advertised by the foreign agent (`allow_once`, `reject_once`, `allow_always`) along with any vendor security warning (such as Antigravity's `agy.security.warning`) without fabricating unadvertised permanent choices.
- **Foreign scope notice:** Each card reminds you that **Persistence scope** (turn-scoped, session-scoped, or persistent across restarts) is controlled solely by the foreign agent process, not stored or enforced by this plugin.

### Native delegation and tool-lane permissions
- **Native delegation (`hivemind-delegate`, `delegate-devin`):** Child sessions run in the Harness's **Delegated mode** (`bypass` on Devin, `yolo` on Antigravity) and bridge any surfaced `session/request_permission` asks through OpenCode's native `Permission.Service` (`permission.asked` event) where `reject` denies and `once` / `always` answer under session policy.
- **`devin_session` tool lane:** Defaults to `autoApprove: false` (`auto_approve: false`), answering `cancelled` unless you explicitly pass `auto_approve: true` for a trusted workload. When `auto_approve: true` is set, deny-shaped labels are excluded first and ambiguous option sets resolve to `cancelled`.

### CodeMode permission boundary (0.4.0)
- **Two gates:** a CodeMode call needs both the outer `execute` permission and the named leaf tool/action permission. Allowing a native permission card never grants either gate.
- **Explicit deny wins:** a saved allow card cannot override an explicit `deny` in `opencode.json`. The plugin never broadens permissions automatically and never promises to repair its own permissions.
- **Config repair is manual-by-consent:** for a user-requested permission repair, the agent asks for the exact project or global edit scope and uses the host's own tools. Manual human steps are offered only when those tools are denied.
- **Exact path:** the agent searches inside `execute` synchronously and calls the exact returned path. There is no hardcoded tool namespace, guessed name, or eval path.

---

## 4. Turn cancellation, native tool settlement, and recovery

### Turn cancellation (**ctrl+x** and abort signals)
- **Hosted View (`ctrl+x`):** Pressing **ctrl+x** sends `session/cancel` for the active turn, waits for the foreign session to settle, and keeps the lane bound so you can send your next prompt or open the Session picker (**ctrl+o**). While a `session/load` (**Pending binding**) is in flight, new prompts and `session/set_config_option` writes are fenced so a configuration change never lands on the wrong session.
- **Native delegation cancellation:** Cancelling a native child job or aborting the caller's `AbortSignal` sends `session/cancel` to the bound foreign trajectory and waits for turn-correlated settlement before **Ownership release**.

### Native tool and thought truthfulness (`flushUnsettled`)
- **Explicit unsettled tool settlement (`flushUnsettled`):** Foreign tool calls (`tool_call` and `tool_call_update`) are recorded as provider-executed observations. If a delegated turn finishes while a foreign tool call is still non-terminal, `flushUnsettled` settles that tool call at turn end with an explicit missing-terminal error rather than leaving an infinite spinner or inventing a fake success result.
- **Zero synthetic thoughts:** Only genuine foreign `agent_thought_chunk` updates enter reasoning output. If a Harness emits zero thought chunks for a turn, the driver emits zero synthetic thought placeholders.

### Crash and restart recovery
- Each Native child records its binding at `bindings/<childSessionID>` with `sentText` and `foreignAck` markers.
- If a connection drops after prompt submission, recovery loads the bound foreign session (`session/load`) and inspects the replayed history: a matching user prompt echo proves an **Accepted turn** without fabricating `foreignAck`, reconnecting on the same foreign trajectory rather than resending a duplicate prompt.

---

## 5. Error taxonomy and common troubleshooting table

| Symptom or Error Code | Cause | Remedy |
|---|---|---|
| `ENOENT` / `blocked` on `acpBinary` | Required `acpBinary` (`devin` or `agy_acp_server.par`) is missing or non-executable. | Run `/hivemind-setup` or `hivemind-doctor` (`action: "check"`), then consent to `action: "install"` or set a valid executable path in `harnesses.<id>.bin`. Remember that `agy` (`cliCompanion`) cannot substitute for `agy_acp_server.par` (`acpBinary`). |
| `Failure class: "auth"` or `signed-out` | Vendor authentication failed or local `sign-out` marked the Harness `signed-out`. | Request `hivemind-doctor` with `action: "sign-in"` and approve the native ask. Follow the selected ACP server's flow; Devin may require `devin auth login`. |
| `Failure class: "quota"` | Vendor account reached its billing or usage limit. | Retries stop immediately without looping. Check your vendor account quota or adjust your **Harness rank** in `/hivemind-setup`. |
| `Failure class: "invalid"` on creation | The requested selection is absent from the fresh catalog. | Choose an advertised model or `model#variant` through `/hivemind-setup change my model policy`. An optional consented session check can refresh `CatalogEvidence`. Hosted defaults are not automatically applied. |
| `tui-owner-absent` | No matching Hosted TUI session is bound. | Open `/devin` or `/antigravity` and bind a session, then use its pickers or request a targeted change. There is no automatic `hosted.<harness>` default policy. |
| `ambiguous-owner-conflict` | Multiple active TUI instances are bound to the same `(Harness, cwd)` slot. | Close the duplicate TUI route or use **ctrl+m**, **ctrl+e**, or **ctrl+t** directly inside the intended Hosted View. |
| `expired` / `owner-gone` / `unconfirmed` | Targeted hosted option request timed out (`15s` TTL) or TUI reloaded during mediation. | Re-check the current session badge in the Hosted View header (`expired`/`owner-gone` refused before write; `unconfirmed` indicates the write was already dispatched before disconnect). |

### Missing `/devin` or `/antigravity` palette commands

If neither the palette nor the `/` completion list shows `/devin` or `/antigravity`, and `/plugins` reports no error, the TUI may not have discovered this plugin. The usual cause is starting the TUI outside the project that holds the plugin entry. In OpenCode `2.1.2`, the TUI resolves plugin lookup from the directory you launch it in, and resolves it once at launch.

- A **project-scoped** install is found only when you start the TUI from that project's directory. Starting elsewhere leaves the palette empty even though the plugin is installed correctly.
- A **global** install (`~/.config/opencode/plugins/`) is found from any launch directory, so this symptom points at a project-scoped entry.

See the [Quickstart](./quickstart.md) for the project-scoped install entry.

Start the TUI from the project that holds the plugin entry, then attach the session you already have:

```sh
cd /path/to/project && opencode -s <session-id>
```

`-s` selects the session to continue on the server; it does not add the launch project's plugin context to a TUI that started elsewhere. Changing the session working directory or picking another project inside the TUI does not change plugin lookup either, because the TUI's plugin lookup location was fixed at launch. Restarting the background service or swapping the plugin build does not help for the same reason.

Resuming with `-s` keeps your chat history, so do not reinstall the plugin and do not recreate the session.

`/freebuff` is expected to be absent: Freebuff is registered for delegation only and declares no Hosted View, so it has no palette command or hosted route.

---

## 6. Platform and legacy tool limits

- **Supported Harnesses:** Devin (`devin-native`) and Antigravity (`antigravity-native`) are the two hosted Harnesses in `0.3.0` and `0.4.0`. Freebuff (`freebuff-native`) is a separate delegation-only, deny-mode lane with no Hosted View. Claude Code and Cursor are future milestones documented in [ROADMAP.md](../ROADMAP.md) and cannot be enabled or routed.
- **Managed Antigravity installer platforms:** Server `1.4.0` has measured bundles for macOS arm64/x64 and Windows arm64/x64 (`agy_acp_server.par` or `agy_acp_server.exe`). Managed installation on Linux is not supported by this release. An explicit path does not by itself establish platform compatibility.
- **`devin_session` tool lane:** Persistent Devin sessions over ACP remain, including the `cloud` and `agent_type` inputs, but they do not create OpenCode Native children (`ses_*`) and do not participate in multi-Harness `rankOrder` routing. In `0.4.0`, the one-shot `devin_run` tool and the duplicate `devin_doctor` tool are removed with no aliases; `hivemind-doctor` is the sole doctor surface. Prefer `hivemind-delegate` and `delegate-devin` for native child workflows.
- **Freebuff admission (0.4.0, refined in 0.4.1):** Freebuff starts only after an explicit owner opt-in: enabling it in the `config/delegate` document, or selecting `harness: "freebuff"`. The first prompt on a new owned conversation reads the server's candidate facts, shows the consent card and sends the admission request only after you approve it. That admission opens a server-issued window with a session-scoped wallet spend limit. Later turns on the same child reuse that conversation and its instance id, so a turn neither admits again nor reopens the consent card. Cancelling a turn stops the turn but does not revoke that upstream window. The consent card shows the live facts the server supplied for the selected model: the declared session price, the available balance and the daily reset instant. An omitted or null fact renders `unknown`, never zero and never free, and a prospective new-window duration is not claimed because those facts do not provide one. After admission, the expiry comes from the server. 0.4.1 status: the upstream service refuses free-mode completions with `403 free_mode_cli_required` on both free and paid catalog rows, and the published Freebuff terms restrict free-tier inference to the official interactive clients. The Freebuff harness therefore ships unconfigured/experimental; supported headless alternatives are the paid Codebuff SDK or BYOK inside the official client. `0.3.0` resolved the credential from `CODEBUFF_API_KEY` alone. `0.4.0` also reads the native Freebuff CLI/Desktop sign-in: the token from `credentials.json` or the `freebuff-cli` keychain entry, bound to the registration in `device-key.json`. An explicit `CODEBUFF_API_KEY` remains valid and takes precedence. Process limits are stricter than turn cancellation: cancelling a turn preserves the running worker's in-process conversation state, while terminating the worker or the host drops the lease and the SDK continuation state without closing the server-issued window. Continuing an old child afterwards fails closed. No window repurchases automatically; opening a new one always requires consent. Cross-process resume is unproven.
- **Freebuff observations (0.4.1):** Read-only child inspection reports lease and conversation facts as timestamped observations from the last attachment. They are not a claim of current entitlement, and they do not prove restart resumption. After a restart the recorded observation reports `liveAttached: false` instead of presenting a stale lease as live.
- **Enablement and install boundaries:** OpenCode's native `/plugins` toggle mounts or unmounts the TUI view only, and it does not stop server-side delegation tools. The persistent per-Harness `enabled` setting lives in `config/delegate` and is set through `/hivemind-setup`; `enabled: false` gates new admissions and continuation turns while already-admitted turns settle. There is no separate custom enable panel. The package installs from npm only, and the public GitHub repository is documentation rather than an installation channel. A paid live Freebuff journey remains unverified.

---

## 7. Related guides and security reporting

- First-time setup and free `opencode/big-pickle` privacy guidance: [Quickstart tutorial](./quickstart.md) ([Tiếng Việt](./quickstart.vi.md))
- Daily Hosted View and Native delegation workflows: [Operations guide](./operations.md) ([Tiếng Việt](./operations.vi.md))
- Environment comparison and prompt prefixes (`/`, `@`, `!`, `#`): [Hosted CLI capability reference](./hosted-cli.md)
- Vulnerability reporting and threat model: [SECURITY.md](../SECURITY.md)
- Project overview, roadmap, glossary, and release notes: [README.md](../README.md), [PROJECT.md](../PROJECT.md), [ROADMAP.md](../ROADMAP.md), [GLOSSARY.md](../GLOSSARY.md), and [CHANGELOG.md](../CHANGELOG.md)
