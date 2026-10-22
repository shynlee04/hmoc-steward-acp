# hmoc-steward-acp

Hivemind's OpenCode plugin lineage. Steward family. ACP lane.

## Release status

`hmoc-steward-acp` is distributed through npm. Version `0.1.0` is published;
`0.2.0` repairs its TUI packaging. See the changelog for the candidate's scope.
Publication, engineering checks and owner acceptance are separate facts.

The current code supports Devin and Antigravity through hosted sessions and
native delegation. Release hardening is still required. Claude Code is the next
planned Harness for both streams. Cursor follows.

## Vision

Stay in OpenCode to work with supported coding agents. Ask in plain English or
Vietnamese. Your OpenCode agent helps you set up the plugin, diagnose problems,
change settings, check the selected lane and choose what to do next.

You should not need to learn internal configuration formats or switch to a
separate control application for ordinary plugin workflows. Vendor installation
and sign-in can still require their supported external steps and your consent.

## The two ways to work

- **Hosted sessions.** Talk to a supported Harness in an OpenCode TUI view.
  Choose or create a session, continue it, review streamed work and answer its
  permission requests.
- **Native delegation.** Ask your OpenCode agent to give a task to a supported
  Harness through a genuine OpenCode child. Follow its result, continue that
  child or cancel its work.

Sessions, models, modes and commands must reflect the Harness's actual support.
A planned integration is not an available choice. Existing children keep their
Harness when you change defaults for future work.

## Guided workflows in the next release

| Ask for | What the agent should do |
|---|---|
| Initial setup | Show detected Harnesses, ask which to enable and rank, explain choices, save confirmed policy and verify it. |
| Doctor | Explain the observed problem and remedy. Ask before installation or sign-in and stop cleanly if you decline. |
| Configuration | Distinguish future delegation defaults from the current hosted session's options. Explain the scope before saving a change. |
| Quick preflight | Check the selected lane before work, fail early on known problems and label what was checked or remains unknown. |
| What next | Show supported hosted/delegation journeys, useful examples and the relevant limitations. |

Examples of the intended English and Vietnamese experience:

- "Set this plugin up and explain my choices."
- "Thiết lập plugin này và giải thích các lựa chọn cho tôi."
- "Check my selected lane before starting. Do not install or sign in without asking."
- "Kiểm tra lane đã chọn trước khi bắt đầu. Hãy hỏi tôi trước khi cài đặt hoặc đăng nhập."

The current setup entry is `/hivemind-setup`. The workflow family uses
`hivemind-*` names. The complete guided doctor, configuration and preflight
experience belongs to the release roadmap. New command names will be documented
when their interfaces are implemented, not advertised as working commands now.

Quick preflight means a short, fail-fast health check. It does not mean secretly
benchmarking agents or selecting a different Harness by measured speed. Explicit
selection and your configured rank remain authoritative. A static check is not
proof of authentication, quota, current models or a successful task.

## Installation direction

Install the npm package through OpenCode. Exact version and project/global
instructions are in [Quickstart](docs/quickstart.md). GitHub is the front page
and user documentation, not a package installation source.

## Public scope

This is an end-user product, not a contribution programme. The npm payload
contains the runtime and essential guides. The public GitHub surface is the
front page, guides, roadmap and safety information, not the development source
or test suite. Existing private development tests remain in the working repo.

Private planning, agent orchestration, research captures, internal development
skills, local machine instructions, accounts and credentials are not release
content. A public roadmap describes outcomes and support. It does not publish the
private development process. The npm runtime necessarily exposes its shipped
implementation; this is a distribution boundary, not source-code secrecy.

## Release promises

- Ask before actions that install software, authenticate or expand permissions.
- Do not save credentials as configuration values.
- Do not change the wrong session or continue cancelled work.
- Preserve settings and session continuity across a public-name change.
- Explain whether a change affects future children or the current hosted session.
- Report blocked or unknown health honestly. Do not invent a successful check.
- Offer equivalent essential guidance in English and Vietnamese.
- Verify the distributed artifact and supported journeys before publication.

See [ROADMAP.md](ROADMAP.md) for milestone order and completion conditions.

## Tiếng Việt

### Tình trạng và tầm nhìn

`hmoc-steward-acp` được phân phối qua npm. Bản `0.1.0` đã được công bố;
`0.2.0` sửa cách đóng gói TUI. Xem changelog để biết phạm vi bản ứng viên.
Đã công bố, đã kiểm tra kỹ thuật và được chủ dự án chấp nhận là ba việc riêng.

Mã hiện tại hỗ trợ Devin và Antigravity qua cả phiên hosted và native delegation.
Vẫn cần hoàn thiện trước khi phát hành. Harness tiếp theo là Claude Code cho cả
hai luồng. Sau đó là Cursor.

Bạn làm việc ngay trong OpenCode và yêu cầu bằng tiếng Anh hoặc tiếng Việt.
Agent giúp thiết lập plugin, chẩn đoán lỗi, đổi cấu hình, kiểm tra lane đã chọn
và trình bày các hành trình tiếp theo. Các bước cài đặt hoặc đăng nhập của nhà
cung cấp có thể vẫn cần thao tác bên ngoài và sự đồng ý của bạn.

### Hai cách làm việc

- **Phiên hosted.** Trò chuyện với Harness trong giao diện TUI của OpenCode.
  Chọn hoặc tạo phiên, tiếp tục phiên, xem công việc đang chạy và trả lời yêu cầu
  cấp quyền.
- **Native delegation.** Yêu cầu agent OpenCode giao việc cho Harness qua một
  phiên con OpenCode thực sự. Xem kết quả, tiếp tục phiên con hoặc hủy công việc.

Phiên, model, mode và lệnh phải phản ánh khả năng thực tế của Harness. Tích hợp
còn trong kế hoạch không phải lựa chọn đang dùng được. Đổi mặc định cho công
việc mới không thay Harness của phiên con đã tồn tại.

### Workflow được hướng dẫn

| Yêu cầu | Agent cần thực hiện |
|---|---|
| Thiết lập ban đầu | Hiển thị Harness phát hiện được, hỏi lựa chọn và thứ tự ưu tiên, giải thích, lưu cấu hình đã xác nhận rồi kiểm tra. |
| Doctor | Giải thích lỗi và cách khắc phục. Hỏi trước khi cài đặt hoặc đăng nhập; dừng an toàn nếu bạn từ chối. |
| Cấu hình | Phân biệt mặc định delegation cho công việc mới với tùy chọn của phiên hosted hiện tại. Giải thích phạm vi trước khi lưu. |
| Preflight nhanh | Kiểm tra lane đã chọn trước khi làm việc, dừng sớm khi gặp lỗi đã biết và nêu rõ phần đã kiểm tra hoặc chưa xác định. |
| Hành trình tiếp theo | Trình bày các hành trình hosted/delegation được hỗ trợ, ví dụ và giới hạn liên quan. |

Điểm vào thiết lập hiện tại là `/hivemind-setup`. Các workflow dùng cùng nhóm
tên `hivemind-*`. Trải nghiệm doctor, cấu hình và preflight đầy đủ vẫn thuộc lộ
trình phát hành. Lệnh mới chỉ được công bố khi giao diện của nó được triển khai.

Preflight nhanh không tự benchmark hoặc chuyển sang Harness có tốc độ đo được
cao hơn. Lựa chọn trực tiếp và thứ tự ưu tiên của bạn vẫn quyết định. Kiểm tra
tĩnh không chứng minh đăng nhập, quota, model hiện tại hay một tác vụ thành công.

### Cài đặt và phạm vi công khai

Cài gói npm qua OpenCode. [Hướng dẫn bắt đầu nhanh](docs/quickstart.vi.md)
nêu phiên bản và phạm vi toàn cục/dự án. GitHub chỉ là trang giới thiệu và tài
liệu người dùng, không phải nguồn cài đặt gói.

Gói npm chứa runtime và hướng dẫn thiết yếu. GitHub công khai chỉ giữ trang
giới thiệu, tài liệu, lộ trình và thông tin an toàn, không phải mã phát triển hay
bộ kiểm thử. Các kiểm thử riêng vẫn được giữ trong kho làm việc. Không công bố
kế hoạch riêng, dữ liệu nghiên cứu, skill nội bộ, tài khoản hay thông tin xác
thực. Runtime npm vẫn bộc lộ phần triển khai được phân phối.

### Cam kết phát hành

- Hỏi trước khi cài đặt, đăng nhập hoặc mở rộng quyền.
- Không lưu thông tin xác thực vào giá trị cấu hình.
- Không đổi nhầm phiên hoặc tiếp tục công việc đã bị hủy.
- Giữ cấu hình và khả năng tiếp tục phiên khi đổi tên công khai.
- Giải thích thay đổi áp dụng cho phiên con mới hay phiên hosted hiện tại.
- Báo rõ trạng thái bị chặn hoặc chưa xác định; không tạo kết quả kiểm tra giả.
- Cung cấp hướng dẫn thiết yếu tương đương bằng tiếng Anh và tiếng Việt.
- Kiểm tra artifact phân phối và hành trình được hỗ trợ trước khi công bố.

Xem [ROADMAP.md](ROADMAP.md) để biết thứ tự và điều kiện hoàn tất các mốc.
