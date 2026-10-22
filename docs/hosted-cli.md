# Hosted CLI capability and interaction reference

This guide is a reference for hosted Agent Client Protocol (ACP) sessions in [OpenCode V2](https://opencode.ai/v2/docs), covering hosted [Devin](https://docs.devin.ai) (`/devin`) and [Antigravity](https://antigravity.google/docs) (`/antigravity`) sessions. It separates the operational boundaries between native command-line interfaces, hosted ACP communication, and the OpenCode host environment.

---

## 1. Environment comparison: native CLI vs. hosted ACP vs. OpenCode host

When interacting with AI agents through this plugin, three distinct execution environments are involved:

| Dimension | Native CLI (`devin` / `agy`) | Hosted ACP (`/devin`, `/antigravity`) | OpenCode host environment |
|---|---|---|---|
| **Execution runtime** | Standalone terminal processes running interactive REPLs and TUIs. | Background ACP server processes (`devin acp`, `agy_acp_server.par`) communicating over stdio via JSON-RPC. | The host OpenCode V2 TUI and plugin runtime. |
| **Slash commands** | Internal REPL commands and interactive configuration dialogs (e.g. `/resume`, `/model`, `/permissions`). | Advertised by the foreign agent over ACP via `session/update` (`available_commands_update`). | Host-level slash commands and command templates defined in OpenCode. |
| **Skills** | Scanned directly from local filesystem trees (`.agents/skills`, `.devin/skills`, `~/.gemini/antigravity-cli/skills`). | Discovered and executed inside the agent. Whether each skill is also advertised as an ACP command is unverified. | Registered OpenCode skills (`.opencode/skills`, `@skill-id`). |
| **File mentions and references** | Terminal-local path suggestions inserting text into the REPL. | Typed `@path` tokens converted to standard ACP `resource_link` (`file://` URI) content blocks sent to the agent. | Host file attachments and directory references. |
| **Attachments** | Terminal clipboard paste (e.g. images). | `image`, `audio` and `embeddedContext` prompt capabilities are advertised by the agent in `promptCapabilities` (for example `agy_acp_server` advertises all three). Client capabilities (`fs`, `terminal`) are unrelated to prompt content. The hosted composer sends `text` and `resource_link` blocks only; there is no media ingestion path. | OpenCode host attachment pipeline. |
| **Workflows** | Legacy workflow files migrate to skills (for example `devin migrate workflows`); Antigravity treats workflows as skills. | No ACP workflow concept. Anything workflow-like runs inside the agent. | OpenCode command templates and skills. |
| **Agents and subagents** | Foreground and background subagents with local profiles (`.devin/agents`, `~/.gemini` agent panels). | Subagents are agent-internal. Their calls surface to the host as ordinary `tool_call` updates; no ACP subagent surface exists. | OpenCode's own subagent and delegation tools. |
| **Tools** | Built-in exec, edit, read, fetch and MCP tools. | Foreign tools execute inside the agent process. Calls stream to the host as `tool_call` and `tool_call_update` render data only. | OpenCode-owned tools under OpenCode permissions. |
| **MCP** | CLI-managed servers (`devin mcp`, `mcp_config.json`, the `/mcp` overlay). | MCP servers are supplied to the agent (for example in `session/new`) and MCP prompts may arrive as advertised commands. No host MCP forwarding bridge exists. | OpenCode MCP configuration and tools. |
| **Rules** | Agent-owned rule discovery (`AGENTS.md`, `.agents/rules`, `~/.config` rule directories). | Rules load inside the agent process; nothing crosses the wire. Whether a hosted session honored a rule is not observable over ACP. | OpenCode loads `AGENTS.md` and its own instruction sources. |
| **Sessions** | REPL session commands (`/resume`, `/clear`, `/fork`). | `session/list` and `session/load` from the agent. A Devin listing covers the account catalog including Desktop and CLI sessions; an Antigravity listing is scoped to the launch `GEMINI_HOME`, and CLI-to-server session interoperability is unverified. | OpenCode's own session store, separate from either agent. |

---

## 2. Prompt prefix semantics

The hosted composer interprets prompt prefixes predictably:

### `/` (slash commands, client controls, and host context)
- **Origin-Labelled Completion**: Typing a leading `/` or pressing **ctrl+g** opens completion where every item carries an explicit origin badge:
  - `[foreign command]`: Advertised by the bound foreign agent over ACP via `session/update` (`available_commands_update`). Selecting a `[foreign command]` row inserts `/<command> ` into the composer with trailing whitespace without auto-submitting, and submitting sends it as an ordinary `session/prompt` text block.
  - `[client control]`: Local Hosted View controls (`sessions`, `model`, `mode`, `thinking`, `filter`, `help`). Selecting a `[client control]` row executes the local picker, filter, or shortcut help inside the OpenCode TUI with zero `session/prompt` wire calls.
  - `[host skill]`: Configured workspace/project `SKILL.md` skills discovered from `.agents/skills`, `.devin/skills`, `.gemini/skills`, and `.opencode/skills` under the workspace root. Selecting a `[host skill]` row stages an explicit `@<skill-path>` token in the composer that resolves on submit to a canonical `resource_link` (`file://` URI). Hosted ACP never claims OpenCode `skill` tool runtime portability and never executes or forwards `!` shell commands, `model`, `agent`, or `subtask` frontmatter fields.
  - `[host reference]`: Declared local `path` entries under `references` in `<workspace>/opencode.json` (skipping `hidden`, missing, home-relative `~`, or escaping paths). Selecting a `[host reference]` row stages an `@<reference-path>` token that resolves on submit to a canonical `resource_link` (`file://` URI).
- **Empty Foreign Catalog Notice**: If the bound foreign session has not reported any `available_commands_update` entries, the popover displays `[foreign command] no commands — this session has not reported any` without fabricating foreign ACP commands and without trapping Enter on a bare `/`, while `[client control]`, `[host skill]`, and `[host reference]` items remain available.
- **Mid-Prompt**: A slash appearing after the first character is treated strictly as literal text (e.g. in paths such as `/src/index.ts`).

### `@` (file mentions, host skills/references, and explicit external references)
- **Workspace Paths and Bounded Host Context**: Typing `@` triggers autocomplete for files within the current workspace root alongside `[foreign command]`, `[host skill]`, and `[host reference]` items. Host skill and reference discovery is strictly bounded to configured project skill directories and declared `opencode.json` `references` — it never crawls arbitrary filesystem roots.
- **Explicit External References**: Typing an explicit absolute path starting with `@/` (such as `@/var/log/system.log`) referencing an existing file or directory produces a canonical `resource_link` content block with its `file://` URI.
- **Authorization Boundary**: Emitting a `resource_link` provides context to the agent but does not bypass agent sandboxing. The foreign agent's internal security policies remain authoritative; the agent may still prompt for permission or refuse an out-of-workspace read.
- **Escaping Traversal**: Relative escaping paths (such as `@../file`) and non-existent absolute paths remain literal text.

### `!` (shell execution)
- **Native CLI Behavior**: In standalone terminal CLIs, entering `!<command>` on an empty input executes a local shell command directly.
- **Hosted ACP Reality**: The standard ACP `session/prompt` protocol does not provide a raw shell passthrough method. A prompt beginning with `!` is treated as ordinary literal text and passed to the agent.

### `#` (hashes and identifiers)
- **Hosted Composer**: `#` is treated as literal text (for Markdown headings, comments, or issue references).
- **OpenCode Host Comparison**: While OpenCode host documentation references `#` for file line ranges (e.g. `@src/file.ts#20-45`), hosted ACP prompt text treats `#` literally.

---

## 3. Permission architecture and persistence scopes

When a foreign agent attempts a sensitive operation (such as file edits, command execution, or out-of-workspace file access), it requests permission over ACP via `session/request_permission`.

### Foreign-advertised options
- The hosted Permission UI presents an inline card displaying the options offered by the foreign agent.
- **Notice of Scope**: Cards include concise notice that options are foreign-advertised and that persistence scope is controlled solely by the agent.
- **No Synthetic Options**: The plugin never fabricates unadvertised permanent choices. If the agent offers only turn-level options (such as `allow_once` and `reject_once`), only those choices are shown.
- **Verbatim Warnings**: When an agent offers `allow_always` (such as Antigravity's `"Allow Always (risky)"` with `agy.security.warning`), the card relays its exact foreign `optionId`, label, and security warning verbatim.

### Grant persistence scopes
1. **Turn-scoped**: Options such as `allow_once` or `reject_once` apply only to the immediate tool call or turn.
2. **Session-scoped**: In-memory grants remembered by the agent for the remainder of the active session.
3. **Persistent / cross-session**: Options such as `allow_always` offer a persistent grant that asks the agent to remember the choice. Whether a grant survives a restart is agent-defined. Devin's ACP grant is documented as durable across sessions. Antigravity's `allow_always` was witnessed offered on `execute` asks, but its persistence across a server restart is unverified. Whether a `GEMINI_HOME`-scoped ACP server reads or shares the standalone CLI's settings files (such as `~/.gemini/antigravity-cli/settings.json` or `~/.config/devin/config.json`) is also unverified. The plugin writes nothing to agent configuration and records no client-side grant.

### Human-controlled modes
Foreign session modes are controlled exclusively by the user:
- Open the mode picker with **ctrl+e** to choose an advertised session mode.
- Open the model picker with **ctrl+m** to switch the active model.
- The plugin emits zero automatic `session/set_mode` or `session/set_config_option` frames without human action: `session/new` binds the advertised catalog as-is (the retired `hosted.<harness>` / `HostedCreationPolicy` block no longer drives writes), and `session/load` continuations are never rewritten (see the [operations guide](./operations.md)).

---

## 4. Keybindings in the hosted view

| Shortcut | Action |
|---|---|
| **ctrl+m** | Open foreign model picker |
| **ctrl+e** | Open foreign session mode picker |
| **ctrl+t** | Open thinking effort picker (Devin) |
| **ctrl+g** | Open foreign commands palette |
| **ctrl+o** | Open session picker to switch or create sessions |
| **ctrl+x** | Cancel in-flight turn |
| **esc** | Return to OpenCode home (session continues in background) |

---

## 5. Documentation and official sources

For further information regarding the underlying platforms and protocols, consult the official documentation:

- [OpenCode V2 documentation](https://opencode.ai/v2/docs)
- [Devin documentation](https://docs.devin.ai)
- [Agent Client Protocol](https://agentclientprotocol.org)
- [Antigravity documentation](https://antigravity.google/docs)
- Repository overview: [README.md](../README.md)
- Licensing: [LICENSE](../LICENSE)
