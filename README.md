# hmoc-steward-acp

**Devin and Antigravity inside OpenCode.** · [English](#english) · [Tiếng Việt](#tiếng-việt)

## English

Give your OpenCode agent a colleague. This plugin opens a real Devin or Antigravity session in your TUI when you want to watch, and hands work to one in the background when you do not.

Install `hmoc-steward-acp` 0.6.2, run setup once, then work normally.

### Install

Documentation for `0.6.2`:

```bash
opencode plugin add hmoc-steward-acp@0.6.2
```

`0.6.2` is the non-attested release candidate carrying four wiring-gap closures (ND-44 permission-handler error attribution, SET-19 linked-worktree scope, T08/T09 hosted sink and composer), the Devin 3000.12.32 baseline alignment, and the cursor-shared model evidence, including `usage_update` context-bound learning (ND-42) and JSON-RPC `-32603` outage attribution (ND-43) from `0.6.1`. `0.6.1` is a patch on `0.6.0` carrying scoped test and seam hardening — no new capability; `0.6.0` is a minor on `0.5.0` that adds the opt-in Cursor harness; `0.5.0` is a minor on `0.4.2`, a patch on `0.4.1`, which is a patch on `0.4.0`, which builds on `0.3.0`. `0.3.0` carries the OpenCode 2.1.2 support repair, the first-run bind fix, and the pinned Freebuff delegate. Earlier `0.2.x` releases (including `0.2.2` and `0.2.2-test.5` on the `testing` tag) remain installable but predate the 2.1.2 repair — do not use them on an OpenCode 2.1.2 host.

Project-only install: add the package to the `plugins` array of that project's `opencode.json` and skip the global command.

### Setup (once)

```text
/hivemind-setup
```

In 0.4.0, `hivemind-setup`, `hivemind-doctor` and `hivemind-hosted` are reached through OpenCode's `execute` (CodeMode) surface, not as named tools: the agent searches inside `execute` and calls the exact path it receives. A blocked call names the missing scope — `execute` itself, or the specific tool/action. The plugin never widens permissions on its own.

The agent checks your saved setup first. If it is already healthy it stops immediately — no questions, no writes, no session-start ritual. If something is missing it asks, one step at a time, using OpenCode's own `question` tool:

1. **Which harnesses to enable**, and their fallback order. Devin and Antigravity are the two hosted harnesses setup can enable. Freebuff is a separate delegation-only lane enabled only by an explicit owner opt-in — enabling it in the `config/delegate` document, or selecting `harness: "freebuff"` — and it never takes a fallback slot. Unsupported harness ids are rejected rather than silently ignored.
2. **Which model each harness uses.** Keep the foreign default, or pick `<model>` or `<model>#<variant>` from that harness's advertised catalog.

Answers are saved to OpenCode's plugin-scoped `config/delegate` document. Setup does not run again on later sessions unless you ask for a change:

```text
/hivemind-setup switch antigravity to the default model
```

That reissues only the affected step and keeps everything else.

### Daily use

| Do this | Get this |
|---|---|
| `/devin` or `/antigravity` | A real agent session opened in the TUI, with continuation, file mentions, slash commands, permissions and model/mode/thinking pickers. |
| Ask your agent to call `hivemind-delegate` | A task handed to a harness in a genuine child session. Foreground waits; background returns and reports later. |
| `/devin_delegate <task>` | The same handoff through a command instead of a tool call. |

Sessions, history and permissions stay in the foreign agent's own world. This plugin is the bridge, not a second agent.

A third delegate Harness, **Freebuff** (`freebuff-native`), ships as an opt-in deny-only lane: a task delegated to it runs inside a genuine native child session on the compiled pinned-source Freebuff SDK bundled in the package, with every tool denied — zero tool effects. It never auto-enables and never takes the rank fallback; an explicit owner opt-in — enabling it in the `config/delegate` document, or selecting `harness: "freebuff"` — is the only admission. It is delegation-only: no `/freebuff` hosted view and no full coding executor. In 0.4.0, each Freebuff run starts by opening a server-issued session lease with a session-scoped wallet spend limit; cancelling a turn stops the turn but does not revoke that upstream window. Its credential may egress only to a first-party HTTPS endpoint or loopback — the fence refuses before any Authorization header otherwise. `0.3.0` resolved the credential from `CODEBUFF_API_KEY` alone. `0.4.0` intends native Freebuff CLI login with an existing registered device-key fallback; an explicit environment key remains valid, and that path is safety-review pending. 0.4.1 status: the upstream service refuses free-mode completions with `403 free_mode_cli_required` on both free and paid catalog rows, and the published Freebuff terms restrict free-tier inference to the official interactive clients. The Freebuff harness therefore ships unconfigured/experimental; supported headless alternatives are the paid Codebuff SDK or BYOK inside the official client.

A fourth delegate Harness, **Claude** (`claude-native`), ships as a default-off preview lane: it drives the pinned `@agentclientprotocol/claude-agent-acp` 0.99.1 adapter (Claude Agent SDK 0.3.312, bundled Claude Code 2.1.312); authentication work in this integration is account-OAuth only, and API-key/API-billing authentication is outside scope. Verified evidence is limited to an `initialize` handshake and an empty-session catalog on the default model; task execution, config writes, account eligibility and human UAT are unverified. The free Claude App admission route is not established and remains BLOCKED, and no paid or API-key substitution is offered. Treat it as a preview, not a supported harness.

A fifth delegate Harness, **Cursor** (`cursor-native`), ships opt-in: it drives your installed `cursor-agent acp` (sign in with `cursor-agent login`; the plugin never installs Cursor or reads its credentials) for native delegation, as a main-session model provider, and as a hosted Cursor view. It stays off until you enable `harnesses.cursor.enabled` or select `harness: "cursor"`, and it never takes a fallback slot. Models are the signed-in account's live catalog, verbatim — including `default[]` (Auto). Cursor has no separate thinking control, so a thinking setting or `#variant` for Cursor is refused by name. Shell commands ask through OpenCode's permission prompt; Cursor applies file edits itself without asking, so agent roles that restrict tools are refused before Cursor starts. Verified on macOS arm64 with Cursor `2026.11.05-86047df`; other platforms and owner UAT are pending.

### Project options

A project can tune delegation in that project's `opencode.json`, under the plugin's `options`. Every key is optional. Options are parsed once when the plugin loads; OpenCode watches `opencode.json` and re-activates the plugin automatically when you save it, so no manual restart is needed. A malformed value refuses the plugin load by name — nothing is coerced, and a bad value never widens permissions.

```jsonc
{
  "plugins": [
    {
      "package": "hmoc-steward-acp",
      "options": {
        "harnesses": {
          "devin": {
            // Catalog-advertised id; "#max" is that model's effort variant.
            "models": ["swe-2#max"],
            // Opt-in, default false: lets this session call back (see below).
            "bridge": true
          },
          "antigravity": {
            "models": ["gemini-4-argon#high"]
            // bridge absent: stays off. Permission asks follow your
            // OpenCode permissions, not a plugin key.
          }
        }
      }
    }
  ]
}
```

- `models` is an ordered list of 1–3 catalog-advertised ids; `#<variant>` names that model's advertised effort. Slots are validated when the child session is created, and an unadvertised slot falls to the next rather than silently defaulting. The old `thinking` key is retired — setting it refuses new delegations by name.
- `bridge` is a strict boolean (`"true"`/`"false"` as strings refuse the load). Default is `false`. With `true`, that harness's delegated session may call back into OpenCode with exactly two capabilities, `delegate` and `child_status`. The grant is issued once per new session binding, so enabling it never reaches a session already running. Disabling it stops further bridge callbacks on existing bindings after the saved options reload, without rewriting the stored record.
- Foreign permission asks follow your own OpenCode permissions — the global `permissions` list and `agents.<id>.permissions` rows, merged last-match-wins — not a per-harness plugin key. `harnesses.<id>.permissionPolicy` is retired: setting it refuses the load by name and names the host permission instead.
- `delegate_approve` is the one residual plugin option, global to every harness, and it answers only an ask still open after your permission rules and the plugin's floor decide. `"deny"` refuses that ask by name; `"allow_once"` answers it once with an option the foreign agent itself offered, cancelling when none is offered; any other value — or no key — leaves the host card to ask you. It never overrides a decided verdict: host denies stand, the floor still goes first (protected anchors such as credentials and host state refuse; project edits and reads pass its own allowance), and a human-only gate such as a paid admission never carries it. Nothing is saved or standing — there is no "always" vocabulary.

Other accepted keys: `harnesses.<id>.bin`, `harnesses.<id>.limits`, `harness_rank`, `delegate_approve`, and per-harness flat aliases reported as deprecated. The retired `permissionPolicy` and any other unlisted facet inside `harnesses.<id>` refuse by name, except the retired `hosted` subtree, which stays tolerated as inert data; unknown top-level keys are tolerated and reported.

### Check or repair

```text
/hivemind-setup health
```

One static read: `ready`, `action-required`, or `blocked`, plus the single next thing to do. No engine spawn.

`hivemind-doctor` goes further than the static health read when you explicitly want contact. In `0.3.0` it was a named tool; in `0.4.0` it is reached through the `execute` surface: `initialize` handshake by default, `session` depth only with consent, and `install` / `sign-in` only after an approval prompt.

### What changed in 0.4.0

- `devin_run` and the duplicate `devin_doctor` are removed with no aliases. `hivemind-doctor` is the only doctor surface.
- The standard ACP `devin_session` tool remains, including its `cloud` and `agent_type` inputs, alongside the `/devin_delegate` command, the `delegate-devin` tool and `hivemind-delegate`.
- Situational tools (`hivemind-setup`, `hivemind-doctor`, `hivemind-hosted`) move behind `execute` as described above. One static `health` read is enough; `ready` stops setup.
- Delegation policy persists globally per plugin in `config/delegate`. Project plugin options in `opencode.json` take precedence for that project; scope stays explicit. Do not rerun setup or preflight for each new session.

### Requirements

- **OpenCode V2** (`opencode` v2.1.4 or later within v2, below v3, on PATH). A V1 host cannot load the TUI entry, and v2.0.x or v3 is outside the supported range. The declared OpenCode peer range is unchanged in 0.5.0; the OpenTUI peer floor moved from `>=0.6.8` to `>=0.8.0`. The pinned development and test baseline is OpenCode 2.2.6. Previously observed runtime evidence stays historical: 6 native-delegation routes (Devin and Antigravity each routing to self, other harness, and executor) passed on OpenCode 2.2.1, and earlier bounded tests passed registration, session/RPC, and TUI surfaces with a 2.1.4 server — no full peer-range certification is claimed. Hosted attach, older inference ACP, sidecar rebind, and human TUI UAT remain pending; Freebuff upstream remains blocked.
- **Devin:** the `devin` CLI on PATH, or `DEVIN_BIN` pointing at it, signed in with `devin auth login`.
- **Antigravity:** the official Antigravity ACP server. The plugin can install the pinned build after you approve it, and signs in through ACP `authenticate`.

The plugin checks these. It does not silently install or sign you in.

### If the plugin fails to load

Before changing configuration, record the installed OpenCode version, the plugin version the host actually selected, and the exact error text. Diagnose the artifact the host resolved rather than assuming a cache deletion or restart is required.

To turn the plugin off without uninstalling, disable it by plugin id:

```json
{ "plugins": ["-hivemind-steward-oc-devin"] }
```

### Safety

- Configuration accepts no credentials. Do not paste tokens, keys or passwords into setup answers or plugin options.
- Permission prompts from the foreign agent follow your OpenCode permission rules and are shown to you by default; `auto_approve` on a delegated session defaults to false.
- **0.4.0:** CodeMode calls need both the outer `execute` permission and the named leaf tool/action permission. An explicit `deny` wins over any saved allow card, and the plugin never broadens permissions automatically. Any `opencode.json` permission repair asks for the exact project or global scope first.
- Delegated runs execute the foreign agent's own tools under its own policy. Review [SECURITY.md](SECURITY.md) before enabling auto-approval.

### Guides

- [Quickstart](docs/quickstart.md) — first session, step by step
- [Operations](docs/operations.md) — health, doctor, recovery
- [Support](docs/support.md) — supported harnesses, limits, known gaps
- [Hosted CLI reference](docs/hosted-cli.md) — capability comparison and prompt syntax

This package is distributed through npm. The GitHub repository is the project page and documentation source, not an installation channel.

### Support the project

The plugin is free to install and use. Nothing here unlocks features; these links are optional.

**Free credits and coding plans (referral links)**

- Freebuff — free coding agent with daily credits: [freebuff.com/?ref=…](https://freebuff.com/?ref=ref-6c980d54-a29a-4270-a160-35f13e62a2ff)
- Zap GLM — GLM coding subscription: [z.ai/subscribe?ic=…](https://z.ai/subscribe?ic=LTWUT4H2RS)

**Donate**

- Ko-fi: [ko-fi.com/shynlee04](https://ko-fi.com/shynlee04)
- Buy Me a Coffee: [buymeacoffee.com/shynlee04l](https://buymeacoffee.com/shynlee04l)

**Watch**

- Video: [youtu.be/tgTFweV7bLo](https://youtu.be/tgTFweV7bLo)
- Channel: [youtube.com/@Elecosys](https://www.youtube.com/@Elecosys)

Project page: [hivemind-portal.vercel.app](https://hivemind-portal.vercel.app/vi)

Referral links and donations never affect what the plugin does. Referral sign-ups benefit the author; the plugin works the same without them.

## Tiếng Việt

Cho agent OpenCode của bạn một đồng nghiệp. Plugin mở phiên Devin hoặc Antigravity thật ngay trong TUI khi bạn muốn xem, và giao việc chạy nền khi bạn không ngồi đó.

Cài `hmoc-steward-acp` 0.6.2, chạy thiết lập một lần, rồi làm việc bình thường.

### Cài đặt

Tài liệu cho `0.6.2`:

```bash
opencode plugin add hmoc-steward-acp@0.6.2
```

`0.6.2` là bản 0.6.2 không xác thực chính thức mang bốn đóng wiring-gap (attribution lỗi ND-44, scope worktree liên kết SET-19, sink và composer hosted T08/T09), alignment baseline Devin 3000.12.32, và bằng chứng model dùng chung cursor, gồm học bound ngữ cảnh từ `usage_update` (ND-42) và attribution lỗi `-32603` (ND-43) từ `0.6.1`. `0.6.1` là bản vá trên `0.6.0`, củng cố test và seam ủy quyền — không có năng lực mới; `0.6.0` là bản minor trên `0.5.0`, bổ sung harness Cursor cần bật tường minh; `0.5.0` là bản minor trên `0.4.2` — một bản vá trên `0.4.1`, vốn là bản vá trên `0.4.0`, vốn được xây trên `0.3.0`. `0.3.0` mang bản sửa lỗi hỗ trợ OpenCode 2.1.2, bản sửa lỗi bind lần đầu, và delegate Freebuff đã ghim. Các bản `0.2.x` trước đó (gồm `0.2.2` và `0.2.2-test.5` trên thẻ `testing`) vẫn cài được nhưng có trước bản sửa 2.1.2 — đừng dùng chúng trên host OpenCode 2.1.2.

Chỉ dùng trong một dự án: thêm gói vào mảng `plugins` trong `opencode.json` của dự án đó, bỏ qua lệnh cài toàn cục.

### Thiết lập (một lần)

```text
/hivemind-setup
```

Trong `0.4.0`, `hivemind-setup`, `hivemind-doctor` và `hivemind-hosted` được truy cập qua bề mặt `execute` (CodeMode) của OpenCode, không phải công cụ có tên: agent tìm bên trong `execute` rồi gọi đúng đường dẫn nhận được. Lệnh bị chặn sẽ nêu rõ phạm vi còn thiếu — bản thân `execute`, hoặc đúng công cụ/hành động. Plugin không bao giờ tự mở rộng quyền.

Agent kiểm tra thiết lập đã lưu trước. Nếu đã ổn nó dừng ngay — không hỏi, không ghi, không có nghi thức đầu phiên. Nếu còn thiếu, nó hỏi từng bước bằng công cụ `question` của OpenCode:

1. **Bật harness nào**, và thứ tự dự phòng. Devin và Antigravity là hai harness hosted mà setup có thể bật. Freebuff là lane ủy quyền riêng, chỉ được nhận khi chủ động bật — bật trong tài liệu `config/delegate` hoặc chọn `harness: "freebuff"` — và không bao giờ nhận suất dự phòng. Harness id không được hỗ trợ sẽ bị từ chối chứ không âm thầm bỏ qua.
2. **Mỗi harness dùng model nào.** Giữ mặc định của harness, hoặc chọn `<model>` hay `<model>#<variant>` từ catalog mà harness công bố.

Câu trả lời được lưu vào tài liệu `config/delegate` theo phạm vi plugin của OpenCode. Các phiên sau không phải thiết lập lại, trừ khi bạn yêu cầu đổi:

```text
/hivemind-setup đổi antigravity sang model mặc định
```

Nó chỉ mở lại đúng bước liên quan và giữ nguyên phần còn lại.

### Dùng hằng ngày

| Làm thế này | Nhận được |
|---|---|
| `/devin` hoặc `/antigravity` | Phiên agent thật mở trong TUI, có tiếp tục phiên cũ, mention tệp, slash command, quyền, và bộ chọn model/mode/thinking. |
| Bảo agent gọi `hivemind-delegate` | Việc được giao cho harness trong một phiên con thật. Chế độ foreground chờ kết quả; background trả về ngay rồi báo sau. |
| `/devin_delegate <task>` | Cùng việc giao đó, qua lệnh thay vì gọi công cụ. |

Phiên, lịch sử và quyền nằm trong thế giới của agent gốc. Plugin này là cầu nối, không phải agent thứ hai.

Harness delegate thứ ba, **Freebuff** (`freebuff-native`), đi kèm như một lane chỉ-deny cần bật tường minh: tác vụ được giao cho nó chạy trong một phiên con native thật trên bản Freebuff SDK đã biên dịch ghim sẵn trong gói, với mọi công cụ đều bị từ chối — không có hiệu ứng công cụ nào. Nó không bao giờ tự bật và không bao giờ nhận suất dự phòng trong thứ hạng; chỉ chủ động bật — trong tài liệu `config/delegate` hoặc chọn `harness: "freebuff"` — mới được nhận. Nó chỉ dùng để ủy quyền: không có Hosted View `/freebuff` và không phải executor lập trình đầy đủ. Trong `0.4.0`, mỗi lượt Freebuff bắt đầu bằng việc mở một session lease do máy chủ cấp với hạn mức chi tiêu ví theo phiên; hủy lượt chạy chỉ dừng lượt đó chứ không thu hồi cửa sổ phía thượng nguồn. Credential của nó chỉ được gửi tới endpoint HTTPS first-party hoặc loopback — hàng rào từ chối trước khi bất kỳ header Authorization nào được đính kèm. `0.3.0` chỉ phân giải credential từ `CODEBUFF_API_KEY`. `0.4.0` dự định đăng nhập bằng CLI Freebuff native với phương án dự phòng device-key đã đăng ký sẵn; khóa môi trường tường minh vẫn hợp lệ, và đường đi này đang chờ đánh giá an toàn. Trạng thái 0.4.1: dịch vụ thượng nguồn từ chối các completion chế độ miễn phí bằng `403 free_mode_cli_required` trên cả dòng catalog miễn phí lẫn trả phí, và điều khoản Freebuff đã công bố giới hạn inference bậc miễn phí cho các client tương tác chính thức. Vì vậy harness Freebuff xuất xưởng ở trạng thái chưa cấu hình/thử nghiệm; lựa chọn headless được hỗ trợ là Codebuff SDK trả phí hoặc BYOK bên trong client chính thức.

Harness delegate thứ tư, **Claude** (`claude-native`), đi kèm như một lane preview mặc-định-tắt: chạy trên adapter ghim `@agentclientprotocol/claude-agent-acp` 0.99.1 (Claude Agent SDK 0.3.312, Claude Code 2.1.312 bundled); xác thực trong tích hợp này chỉ qua OAuth tài khoản, và xác thực API-key/API-billing nằm ngoài phạm vi. Bằng chứng đã kiểm chứng giới hạn ở handshake `initialize` và catalog của một phiên rỗng trên model mặc định; chạy task, ghi config, điều kiện tài khoản và UAT của chủ sở hữu vẫn chưa được kiểm chứng. Đường dùng Claude App miễn phí chưa được thiết lập và vẫn BỊ CHẶN, và không có phương án thay thế trả phí hay API-key. Hãy coi nó là preview, chưa phải harness được hỗ trợ.

Harness delegate thứ năm, **Cursor** (`cursor-native`), cần bật tường minh: nó chạy `cursor-agent acp` bạn đã cài (đăng nhập bằng `cursor-agent login`; plugin không bao giờ cài Cursor hay đọc credential của nó) cho ủy quyền native, làm provider model cho phiên chính, và làm Hosted View Cursor. Nó tắt cho đến khi bạn bật `harnesses.cursor.enabled` hoặc chọn `harness: "cursor"`, và không bao giờ nhận suất dự phòng. Danh sách model lấy nguyên văn từ catalog trực tiếp của tài khoản đã đăng nhập — gồm cả `default[]` (Auto). Cursor không có điều khiển thinking riêng, nên thiết lập thinking hoặc `#variant` cho Cursor sẽ bị từ chối kèm tên. Lệnh shell hỏi qua lời nhắc quyền của OpenCode; Cursor tự áp dụng chỉnh sửa file mà không hỏi, nên các agent role giới hạn công cụ sẽ bị từ chối trước khi Cursor khởi động. Đã kiểm chứng trên macOS arm64 với Cursor `2026.11.05-86047df`; các nền tảng khác và UAT của chủ sở hữu vẫn đang chờ.

### Tuỳ chọn theo dự án

Một dự án có thể tinh chỉnh ủy quyền trong `opencode.json` của chính nó, dưới khối `options` của plugin. Mọi khoá đều tuỳ chọn. Tuỳ chọn chỉ được parse một lần khi plugin nạp; OpenCode theo dõi `opencode.json` và tự động nạp lại plugin khi bạn lưu tệp, không cần khởi động lại thủ công. Giá trị sai định dạng làm plugin từ chối nạp kèm tên khoá — không có ép kiểu, và giá trị sai không bao giờ mở rộng quyền.

```jsonc
{
  "plugins": [
    {
      "package": "hmoc-steward-acp",
      "options": {
        "harnesses": {
          "devin": {
            // Id do catalog công bố; "#max" là biến thể effort của model đó.
            "models": ["swe-2#max"],
            // Opt-in, mặc định false: cho phiên này gọi ngược lại (xem dưới).
            "bridge": true
          },
          "antigravity": {
            "models": ["gemini-4-argon#high"]
            // Không khai bridge: vẫn tắt. Yêu cầu quyền đi theo luật
            // OpenCode của bạn, không phải khoá plugin.
          }
        }
      }
    }
  ]
}
```

- `models` là danh sách có thứ tự gồm 1–3 id do catalog công bố; `#<variant>` chỉ effort mà model đó quảng bá. Các slot được kiểm khi phiên con được tạo, và một slot không được công bố sẽ nhường cho slot kế tiếp chứ không âm thầm dùng mặc định. Khoá `thinking` cũ đã ngừng — đặt nó sẽ làm các ủy quyền mới bị từ chối kèm tên.
- `bridge` là boolean nghiêm ngặt (chuỗi `"true"`/`"false"` làm từ chối nạp). Mặc định là `false`. Khi `true`, phiên được ủy quyền của harness đó có thể gọi ngược lại OpenCode với đúng hai năng lực, `delegate` và `child_status`. Grant được cấp một lần cho mỗi binding phiên mới, nên bật nó không chạm tới phiên đang chạy. Tắt nó sẽ dừng mọi lượt gọi ngược bridge tiếp theo trên cả binding đang tồn tại sau khi tuỳ chọn được nạp lại, không ghi đè bản ghi đã lưu.
- Mọi yêu cầu quyền từ agent gốc đi theo quyền OpenCode của chính bạn — danh sách `permissions` toàn cục và các dòng `agents.<id>.permissions`, hợp nhất theo nguyên tắc khớp-cuối-thắng — chứ không phải khoá plugin theo harness. Khoá `harnesses.<id>.permissionPolicy` đã ngừng: đặt nó sẽ làm plugin từ chối nạp kèm tên và chỉ đường tới thiết lập quyền của host.
- `delegate_approve` là tuỳ chọn plugin còn lại duy nhất, áp dụng toàn cục cho mọi harness, và chỉ trả lời yêu cầu vẫn mở sau khi luật quyền của bạn và floor của plugin quyết định. `"deny"` từ chối yêu cầu đó kèm tên; `"allow_once"` trả lời đúng một lần bằng tuỳ chọn mà chính agent gốc đưa ra, huỷ nếu yêu cầu không có tuỳ chọn đó; giá trị khác — hoặc không khai — để thẻ host hỏi bạn. Nó không bao giờ ghi đè quyết định đã chốt: deny của host vẫn thắng, floor vẫn chạy trước (các anchor được bảo vệ như credential và trạng thái host từ chối; sửa đổi và đọc trong dự án đi qua allowance của nó), và cổng chỉ-người như nhập trả phí không bao giờ chịu nó. Không gì được lưu hay thường trực — không có từ vựng "luôn cho phép".

Các khoá khác được chấp nhận: `harnesses.<id>.bin`, `harnesses.<id>.limits`, `harness_rank`, `delegate_approve`, và các alias phẳng theo harness được báo là deprecated. `permissionPolicy` đã ngừng, và mọi facet không nằm trong danh sách bên trong `harnesses.<id>` đều bị từ chối kèm tên — riêng nhánh `hosted` đã ngừng vẫn được giữ như dữ liệu trơ; khoá lạ ở cấp cao nhất được chấp nhận và báo cáo.

### Kiểm tra hoặc sửa

```text
/hivemind-setup health
```

Một lần đọc tĩnh: `ready`, `action-required`, hoặc `blocked`, kèm đúng một việc cần làm tiếp. Không khởi động engine.

`hivemind-doctor` đi xa hơn lượt đọc tĩnh `health` khi bạn thật sự muốn liên hệ engine. Trong `0.3.0` nó là công cụ có tên; trong `0.4.0` nó được truy cập qua bề mặt `execute`: mặc định chỉ bắt tay `initialize`, độ sâu `session` chỉ khi bạn đồng ý, và `install` / `sign-in` chỉ sau khi bạn duyệt.

### Thay đổi trong 0.4.0

- `devin_run` và `devin_doctor` trùng lặp bị gỡ bỏ, không có alias. `hivemind-doctor` là bề mặt doctor duy nhất.
- Công cụ ACP chuẩn `devin_session` vẫn còn, gồm cả đầu vào `cloud` và `agent_type`, cùng lệnh `/devin_delegate`, công cụ `delegate-devin` và `hivemind-delegate`.
- Các công cụ tình huống (`hivemind-setup`, `hivemind-doctor`, `hivemind-hosted`) chuyển vào sau `execute` như mô tả ở trên. Một lượt đọc tĩnh `health` là đủ; `ready` thì dừng thiết lập.
- Chính sách ủy quyền lưu toàn cục theo plugin trong `config/delegate`. Tùy chọn plugin trong `opencode.json` của dự án được ưu tiên cho dự án đó; phạm vi luôn tường minh. Không chạy lại setup hay preflight cho từng phiên mới.

### Yêu cầu

- **OpenCode V2** (`opencode` từ v2.1.4 trở lên trong nhánh v2, dưới v3, trên PATH). Host V1 không nạp được entry TUI; v2.0.x và v3 nằm ngoài khoảng hỗ trợ. Khoảng peer OpenCode công bố không đổi trong 0.5.0; sàn peer OpenTUI nâng từ `>=0.6.8` lên `>=0.8.0`. Baseline phát triển và kiểm thử được ghim là OpenCode 2.2.6. Bằng chứng runtime đã quan sát trước đây giữ nguyên là lịch sử: 6 route ủy quyền gốc (Devin và Antigravity giao việc cho self, harness đối ứng, và executor) đã đạt trên OpenCode 2.2.1, và các thử nghiệm cô lập trước đó cũng vượt qua đăng ký plugin, session/RPC và TUI với server 2.1.4 — không có tuyên bố chứng nhận toàn dải peer. Hosted attach, inference ACP cũ, rebind sidecar và UAT TUI từ chủ sở hữu vẫn đang chờ; Freebuff phía thượng nguồn tiếp tục bị chặn.
- **Devin:** CLI `devin` trên PATH, hoặc `DEVIN_BIN` trỏ tới nó, đã đăng nhập bằng `devin auth login`.
- **Antigravity:** ACP server chính thức của Antigravity. Plugin có thể cài bản đã ghim sau khi bạn duyệt, và đăng nhập qua ACP `authenticate`.

Plugin chỉ kiểm tra. Nó không tự cài hay tự đăng nhập.

### Nếu plugin không nạp được

Trước khi đổi cấu hình, ghi lại phiên bản OpenCode đang cài, phiên bản plugin mà OpenCode thực sự chọn, và nguyên văn lỗi. Chẩn đoán đúng artifact mà host đã resolve thay vì mặc định phải xoá cache hay khởi động lại.

Muốn tắt plugin mà không gỡ cài, tắt theo plugin id:

```json
{ "plugins": ["-hivemind-steward-oc-devin"] }
```

### An toàn

- Cấu hình không nhận credential. Đừng dán token, khoá hay mật khẩu vào câu trả lời thiết lập hoặc tuỳ chọn plugin.
- Prompt quyền từ agent gốc đi theo luật quyền OpenCode của bạn và mặc định hiển thị cho bạn; `auto_approve` trên phiên được giao mặc định là false.
- **0.4.0:** Lệnh CodeMode cần cả quyền `execute` bên ngoài lẫn quyền công cụ/hành động lá có tên. `deny` tường minh thắng mọi thẻ cho phép đã lưu, và plugin không bao giờ tự mở rộng quyền. Mọi sửa quyền trong `opencode.json` đều hỏi đúng phạm vi dự án hoặc toàn cục trước.
- Phiên được giao chạy công cụ của agent gốc theo chính sách của nó. Đọc [SECURITY.md](SECURITY.md) trước khi bật tự động duyệt.

### Hướng dẫn

- [Bắt đầu nhanh](docs/quickstart.vi.md) — phiên đầu tiên, từng bước
- [Vận hành](docs/operations.vi.md) — health, doctor, khôi phục
- [Hỗ trợ](docs/support.vi.md) — harness hỗ trợ, giới hạn, khoảng trống đã biết
- [Tham chiếu Hosted CLI](docs/hosted-cli.md) — so sánh năng lực và cú pháp prompt

Gói này phát hành qua npm. Kho GitHub là trang dự án và nguồn tài liệu, không phải kênh cài đặt.

### Ủng hộ dự án

Plugin miễn phí để cài và dùng. Những link dưới đây là tuỳ chọn, không mở khoá tính năng nào.

**Ưu đãi và gói lập trình (link giới thiệu)**

- Freebuff — coding agent miễn phí kèm credit hằng ngày: [freebuff.com/?ref=…](https://freebuff.com/?ref=ref-6c980d54-a29a-4270-a160-35f13e62a2ff)
- Zap GLM — gói thuê bao GLM: [z.ai/subscribe?ic=…](https://z.ai/subscribe?ic=LTWUT4H2RS)

**Ủng hộ tác giả**

- Ko-fi: [ko-fi.com/shynlee04](https://ko-fi.com/shynlee04)
- Buy Me a Coffee: [buymeacoffee.com/shynlee04l](https://buymeacoffee.com/shynlee04l)

**Xem**

- Video: [youtu.be/tgTFweV7bLo](https://youtu.be/tgTFweV7bLo)
- Kênh: [youtube.com/@Elecosys](https://www.youtube.com/@Elecosys)

Trang dự án: [hivemind-portal.vercel.app](https://hivemind-portal.vercel.app/vi)

Link giới thiệu và ủng hộ không làm thay đổi cách plugin hoạt động. Đăng ký qua link giới thiệu mang lại lợi ích cho tác giả; plugin vẫn chạy y như vậy nếu không dùng.
