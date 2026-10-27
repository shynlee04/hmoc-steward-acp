# Roadmap

This is the end-user roadmap for `hmoc-steward-acp`. See
[PROJECT.md](PROJECT.md) for the vision, public scope and installation direction.
Milestones below are planned outcomes, not released feature claims or promised
dates. The npm registry confirms `0.1.0`; `0.2.0` is the packaging/setup repair.

## npm MVP order

These are gaps in the existing release, not new Harness phases.

1. **Now, 0.2.0.** Load and stream from the npm-installed TUI. Save setup once,
   finish with foreign defaults, and accept an explicit change to saved choices.
   Keep English/Vietnamese instructions aligned with actual supported behavior.
2. **Next.** Move occasional setup/doctor/hosted management actions behind
   OpenCode `execute` and skill-led discovery. Keep delegation directly usable.
   Preserve each action's permission checks and give one clear next step on error.
3. **Later.** Add a dedicated native-dialog wizard only if the agent-led
   `question` flow is unreliable. Consider more Harnesses after the existing
   npm journey passes owner UAT. Neither is required to repair this MVP.

GitHub is the front page and user docs. npm is the installation channel.
Tests stay in private development; they are not a public contribution workflow.

## Current baseline

Devin and Antigravity have hosted-session and native-delegation implementations.
The new named production release still needs safety, guided-workflow and
distribution hardening. Claude Code and Cursor are not currently supported.

## Milestones

| Order | Milestone | User outcome | Completion condition |
|---|---|---|---|
| 0 | Safe release baseline | Changing options affects the intended session; cancelling work stops it; valid installation choices are understood. | Session-swap settings and cancellation regressions are repaired and verified. The new public name preserves existing settings and session continuity. |
| 1 | Guided English/Vietnamese workflows | Ask naturally in OpenCode TUI for setup, doctor, configuration, quick preflight and the next supported journey. | Published skills/commands match real behavior. The agent explains scope, requests needed consent, handles failures and gives equivalent essential guidance in both languages. |
| 2 | npm distribution | Install the supported product from npm through OpenCode. | Verify the frozen tarball, registry version and public readback. GitHub carries user docs, not source/test releases. Private development content and credentials are excluded. |
| 3 | Claude Code | Use Claude Code through hosted sessions and native delegation in OpenCode. | Both streams work through a verified supported integration path, including setup, real capabilities, permissions, cancellation, continuation and failure reporting. |
| 4 | Cursor | Use Cursor through the same two streams and guided workflows. | Both streams meet the same user-facing acceptance conditions after the Claude Code milestone. |

Claude Code is the next new Harness. Cursor follows Claude Code. A single
working stream does not complete either Harness milestone. Model names, commands
and sessions will be shown only when genuinely supported by that integration.

## What production verification covers

- A first-time user starts from the distributed package, not a private developer
  checkout, and receives a clear guided setup flow.
- English and Vietnamese prompts reach the intended workflow and expose choices
  through the available OpenCode interaction surfaces.
- The doctor explains missing executables, unavailable integration paths and
  sign-in requirements without silently installing or authenticating.
- Confirmed configuration changes preserve unrelated settings and distinguish
  future-child policy from current hosted-session options.
- Preflight distinguishes static evidence, initialized-engine evidence, blocked
  states and unknown facts. Known failures stop before task dispatch.
- The npm package includes the promised skills, commands and guides.
- Hosted and delegated work cover start, continuation, cancellation, errors and
  the supported permission behavior of each Harness.
- Updates preserve saved settings and existing session ownership.
- The final public payload excludes private planning, captured account data,
  internal development skills, local machine assumptions and credentials.

Selected production tests exist to prove the released experience. They are not
a contribution tutorial. Green development tests alone do not establish these
outcomes, and a health check must not hide a paid task or claim unobserved facts.

## Scope discipline

No contribution programme, standalone control application or performance-based
Harness selector is part of this release direction. Quick preflight respects
your explicit selection and configured rank. Future integrations must preserve
the same consent and capability-honesty rules rather than imitate unavailable
vendor features.

## Tiếng Việt

Đây là lộ trình dành cho người dùng cuối của `hmoc-steward-acp`. Xem
[PROJECT.md](PROJECT.md) để biết tầm nhìn, phạm vi công khai và cách cài đặt.
Các mốc là mục tiêu dự kiến, không phải tính năng đã phát hành hay cam kết về
ngày hoàn tất. npm đã có `0.1.0`; `0.2.0` sửa đóng gói TUI và thiết lập.

### Thứ tự MVP npm

1. **Bây giờ, 0.2.0.** TUI cài từ npm nạp được và hiển thị phản hồi trực tiếp.
   Lưu thiết lập một lần, hoàn tất khi giữ mặc định ngoại lai và cho phép đổi
   lựa chọn đã lưu. Sửa hướng dẫn hai ngôn ngữ theo hành vi thực tế.
2. **Tiếp theo.** Đưa thao tác quản lý ít dùng vào `execute` và skill, giữ
   delegation dễ gọi trực tiếp. Không bỏ kiểm tra quyền; lỗi phải nêu bước kế tiếp.
3. **Về sau.** Chỉ thêm wizard hộp thoại gốc nếu luồng `question` qua agent
   thiếu tin cậy. Chỉ xét Harness mới sau UAT của hành trình npm hiện tại.

GitHub là trang giới thiệu và tài liệu; npm là kênh cài đặt. Bộ kiểm thử vẫn
thuộc kho phát triển riêng, không phải quy trình đóng góp công khai.

### Hiện trạng

Devin và Antigravity đã có triển khai phiên hosted và native delegation.
Bản production mang tên mới vẫn cần hoàn thiện an toàn, workflow được hướng
dẫn và phân phối. Claude Code và Cursor hiện chưa được hỗ trợ.

### Các mốc

| Thứ tự | Mốc | Kết quả cho người dùng | Điều kiện hoàn tất |
|---|---|---|---|
| 0 | Nền tảng phát hành an toàn | Đổi tùy chọn đúng phiên; hủy công việc thực sự dừng việc; hiểu các lựa chọn cài đặt hợp lệ. | Sửa và kiểm tra lỗi cấu hình khi chuyển phiên cùng lỗi hủy tác vụ. Tên công khai mới giữ cấu hình và khả năng tiếp tục phiên đã có. |
| 1 | Workflow tiếng Anh/tiếng Việt | Yêu cầu tự nhiên trong TUI OpenCode để thiết lập, doctor, cấu hình, preflight nhanh và chọn hành trình tiếp theo. | Skill/lệnh công bố khớp hành vi thực tế. Agent giải thích phạm vi, xin sự đồng ý khi cần, xử lý lỗi và cung cấp hướng dẫn thiết yếu tương đương ở hai ngôn ngữ. |
| 2 | Phân phối npm | Cài sản phẩm từ npm qua OpenCode. | Xác minh tarball đã chốt, phiên bản registry và gói tải lại công khai. GitHub chỉ chứa tài liệu, không phát hành mã/kiểm thử. Không lộ nội dung riêng hay thông tin xác thực. |
| 3 | Claude Code | Dùng Claude Code qua phiên hosted và native delegation trong OpenCode. | Cả hai luồng hoạt động qua cách tích hợp được hỗ trợ và đã xác minh, gồm thiết lập, khả năng thực, quyền, hủy, tiếp tục phiên và báo lỗi. |
| 4 | Cursor | Dùng Cursor qua cùng hai luồng và workflow được hướng dẫn. | Cả hai luồng đạt điều kiện chấp nhận tương đương sau mốc Claude Code. |

Claude Code là Harness mới tiếp theo. Cursor đi sau Claude Code. Chỉ hoàn tất
một luồng chưa đủ để hoàn tất mốc của Harness. Model, lệnh và phiên chỉ được
hiển thị khi tích hợp thực sự hỗ trợ.

### Kiểm tra production

- Người dùng mới bắt đầu từ gói được phân phối, không phụ thuộc kho phát triển
  riêng, và nhận luồng thiết lập rõ ràng.
- Yêu cầu tiếng Anh và tiếng Việt đi đúng workflow; lựa chọn được trình bày qua
  các bề mặt tương tác có sẵn của OpenCode.
- Doctor giải thích thiếu executable, cách tích hợp chưa có hoặc yêu cầu đăng
  nhập; không âm thầm cài đặt hay xác thực.
- Thay đổi cấu hình đã xác nhận giữ giá trị không liên quan và phân biệt chính
  sách cho phiên con mới với tùy chọn của phiên hosted hiện tại.
- Preflight phân biệt bằng chứng tĩnh, bằng chứng khởi tạo engine, trạng thái bị
  chặn và điều chưa biết. Lỗi đã biết dừng trước khi giao tác vụ.
- Hai phương thức cài đặt có cùng skill, lệnh và hướng dẫn đã cam kết.
- Công việc hosted/delegation bao gồm bắt đầu, tiếp tục, hủy, báo lỗi và hành vi
  cấp quyền được hỗ trợ của từng Harness.
- Cập nhật giữ cấu hình đã lưu và quyền sở hữu phiên hiện có.
- Nội dung công khai không chứa kế hoạch riêng, dữ liệu tài khoản đã ghi lại,
  skill phát triển nội bộ, giả định riêng của máy hoặc thông tin xác thực.

Các kiểm thử production đã chọn dùng để chứng minh trải nghiệm được phát hành,
không phải bài hướng dẫn đóng góp. Chỉ có kiểm thử phát triển xanh là chưa đủ.
Kiểm tra sức khỏe không được giấu một tác vụ tính phí hoặc khẳng định điều chưa
quan sát được.

### Giới hạn phạm vi

Định hướng này không gồm chương trình đóng góp, ứng dụng điều khiển riêng hay
bộ chọn Harness dựa trên tốc độ. Preflight nhanh tuân theo lựa chọn trực tiếp và
thứ tự ưu tiên của bạn. Tích hợp mới giữ cùng quy tắc đồng ý và trung thực về
khả năng, không mô phỏng tính năng nhà cung cấp chưa có.
