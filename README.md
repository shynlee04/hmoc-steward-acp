# hmoc-steward-acp

**Devin and Antigravity inside OpenCode.** · [English](#english) · [Tiếng Việt](#tiếng-việt)

## English

Give your OpenCode agent a colleague. This plugin opens a real Devin or Antigravity session in your TUI when you want to watch, and hands work to one in the background when you do not.

Install `hmoc-steward-acp@0.4.0` from npm, run setup once, then work normally.

### Install

Documentation for `0.4.0`:

```bash
opencode plugin add hmoc-steward-acp@0.4.0
```

`0.4.0` builds on `0.3.0`, which carries the OpenCode 2.1.2 support repair, the first-run bind fix, and the pinned Freebuff delegate; earlier `0.2.x` releases (including `0.2.2` and `0.2.2-test.5` on the `testing` tag) remain installable but predate the 2.1.2 repair — do not use them on an OpenCode 2.1.2 host.

Reload OpenCode afterwards. The plugin loads on startup, so a running session keeps the old code until you restart.

Project-only install: add `hmoc-steward-acp@0.4.0` to the `plugins` array of that project's `opencode.json` and skip the global command.

### Setup (once)

```text
/hivemind-setup
```

In `0.4.0`, the three situational tools — `hivemind-setup`, `hivemind-doctor` and `hivemind-hosted` — are reached through OpenCode's `execute` (CodeMode) surface rather than as named tools: the agent searches inside `execute` synchronously and calls the exact path returned. A blocked call names the missing scope — `execute` itself, or the specific tool and action. The plugin never widens permissions on its own. The `/hivemind-setup` command and its question guidance stay unchanged.

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

A third delegate Harness, **Freebuff** (`freebuff-native`), ships as an opt-in deny-only lane: a task delegated to it runs inside a genuine native child session on the compiled pinned-source Freebuff SDK bundled in the package, with every tool denied — zero tool effects. It never auto-enables and never takes the rank fallback; an explicit owner opt-in — enabling it in the `config/delegate` document, or selecting `harness: "freebuff"` — is the only admission. It is delegation-only: no `/freebuff` hosted view and no full coding executor. In `0.4.0`, each Freebuff run starts by opening a server-issued session lease with a session-scoped wallet spend limit; cancelling a turn stops the turn but does not revoke that upstream window. Its credential may egress only to a first-party HTTPS endpoint or loopback — the fence refuses before any Authorization header otherwise. `0.3.0` resolved the credential from `CODEBUFF_API_KEY` alone. `0.4.0` intends native Freebuff CLI login with an existing registered device-key fallback; an explicit environment key remains valid, and that path is safety-review pending. Its live entitled turn is unverified: there is no live entitled-prompt or billing proof yet.

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

- **OpenCode V2** (`opencode` v2.1.2 or later within v2, below v3, on PATH). A V1 host cannot load the TUI entry, and v2.0.x or v3 is outside the supported range.
- **Devin:** the `devin` CLI on PATH, or `DEVIN_BIN` pointing at it, signed in with `devin auth login`.
- **Antigravity:** the official Antigravity ACP server. The plugin can install the pinned build after you approve it, and signs in through ACP `authenticate`.

The plugin checks these. It does not silently install or sign you in.

### If the plugin fails to load

OpenCode caches npm plugin installs under `~/.cache/opencode/npm/`. After upgrading, if the error still names an old version's files, delete that package's cache directory and restart. An old cached install keeps running old code regardless of what the registry serves.

To turn the plugin off without uninstalling, disable it by plugin id:

```json
{ "plugins": ["-hivemind-steward-oc-devin"] }
```

### Safety

- Configuration accepts no credentials. Do not paste tokens, keys or passwords into setup answers or plugin options.
- Permission prompts from the foreign agent are shown to you; `auto_approve` on a delegated session defaults to false.
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

Cài `hmoc-steward-acp@0.4.0` từ npm, chạy thiết lập một lần, rồi làm việc bình thường.

### Cài đặt

Tài liệu cho `0.4.0`:

```bash
opencode plugin add hmoc-steward-acp@0.4.0
```

`0.4.0` được xây trên `0.3.0`, bản đã có bản sửa lỗi hỗ trợ OpenCode 2.1.2, bản sửa lỗi bind lần đầu, và delegate Freebuff đã ghim; các bản `0.2.x` trước đó (gồm `0.2.2` và `0.2.2-test.5` trên thẻ `testing`) vẫn cài được nhưng có trước bản sửa 2.1.2 — đừng dùng chúng trên host OpenCode 2.1.2.

Sau đó nạp lại OpenCode. Plugin chỉ nạp lúc khởi động, nên phiên đang chạy vẫn giữ code cũ cho tới khi bạn khởi động lại.

Chỉ dùng trong một dự án: thêm `hmoc-steward-acp@0.4.0` vào mảng `plugins` trong `opencode.json` của dự án đó, bỏ qua lệnh cài toàn cục.

### Thiết lập (một lần)

```text
/hivemind-setup
```

Trong `0.4.0`, ba công cụ tình huống — `hivemind-setup`, `hivemind-doctor` và `hivemind-hosted` — được truy cập qua bề mặt `execute` (CodeMode) của OpenCode, không phải công cụ có tên: agent tìm bên trong `execute` một cách đồng bộ rồi gọi đúng đường dẫn nhận được. Lệnh bị chặn sẽ nêu rõ phạm vi còn thiếu — bản thân `execute`, hoặc đúng công cụ/hành động. Plugin không bao giờ tự mở rộng quyền. Lệnh `/hivemind-setup` và hướng dẫn bằng câu hỏi vẫn giữ nguyên.

Agent kiểm tra thiết lập đã lưu trước. Nếu đã ổn nó dừng ngay — không hỏi, không ghi, không có nghi thức đầu phiên. Nếu còn thiếu, nó hỏi từng bước bằng công cụ `question` của OpenCode:

1. **Bật harness nào**, và thứ tự dự phòng. Devin và Antigravity là hai harness hosted mà setup có thể bật. Freebuff là lane ủy quyền riêng, chỉ được nhận khi chủ động opt-in bởi chủ sở hữu — bật trong tài liệu `config/delegate` hoặc chọn `harness: "freebuff"` — và không bao giờ nhận suất dự phòng. Harness id không được hỗ trợ sẽ bị từ chối chứ không âm thầm bỏ qua.
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

Harness delegate thứ ba, **Freebuff** (`freebuff-native`), đi kèm như một lane chỉ-deny cần bật tường minh: tác vụ được giao cho nó chạy trong một phiên con native thật trên bản Freebuff SDK đã biên dịch ghim sẵn trong gói, với mọi công cụ đều bị từ chối — không có hiệu ứng công cụ nào. Nó không bao giờ tự bật và không bao giờ nhận suất dự phòng trong thứ hạng; chỉ chủ động bật — trong tài liệu `config/delegate` hoặc chọn `harness: "freebuff"` — mới được nhận. Nó chỉ dùng để ủy quyền: không có Hosted View `/freebuff` và không phải executor lập trình đầy đủ. Trong `0.4.0`, mỗi lượt Freebuff bắt đầu bằng việc mở một session lease do máy chủ cấp với hạn mức chi tiêu ví theo phiên; hủy lượt chạy chỉ dừng lượt đó chứ không thu hồi cửa sổ phía thượng nguồn. Credential của nó chỉ được gửi tới endpoint HTTPS first-party hoặc loopback — hàng rào từ chối trước khi bất kỳ header Authorization nào được đính kèm. `0.3.0` chỉ phân giải credential từ `CODEBUFF_API_KEY`. `0.4.0` dự định đăng nhập bằng CLI Freebuff native với phương án dự phòng device-key đã đăng ký sẵn; khóa môi trường tường minh vẫn hợp lệ, và đường đi này đang chờ đánh giá an toàn. Lượt chạy thật có quyền vẫn chưa được kiểm chứng: chưa có bằng chứng về prompt có quyền chạy thật hoặc tính phí.

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

- **OpenCode V2** (`opencode` từ v2.1.2 trở lên trong nhánh v2, dưới v3, trên PATH). Host V1 không nạp được entry TUI; v2.0.x và v3 nằm ngoài khoảng hỗ trợ.
- **Devin:** CLI `devin` trên PATH, hoặc `DEVIN_BIN` trỏ tới nó, đã đăng nhập bằng `devin auth login`.
- **Antigravity:** ACP server chính thức của Antigravity. Plugin có thể cài bản đã ghim sau khi bạn duyệt, và đăng nhập qua ACP `authenticate`.

Plugin chỉ kiểm tra. Nó không tự cài hay tự đăng nhập.

### Nếu plugin không nạp được

OpenCode cache bản cài plugin npm trong `~/.cache/opencode/npm/`. Sau khi nâng cấp, nếu lỗi vẫn nhắc tới tệp của phiên bản cũ, hãy xoá thư mục cache của gói đó rồi khởi động lại. Bản cache cũ vẫn chạy code cũ bất kể registry phục vụ gì.

Muốn tắt plugin mà không gỡ cài, tắt theo plugin id:

```json
{ "plugins": ["-hivemind-steward-oc-devin"] }
```

### An toàn

- Cấu hình không nhận credential. Đừng dán token, khoá hay mật khẩu vào câu trả lời thiết lập hoặc tuỳ chọn plugin.
- Prompt quyền từ agent gốc được hiển thị cho bạn; `auto_approve` trên phiên được giao mặc định là false.
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
