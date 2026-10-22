# Tham chiếu hỗ trợ và an toàn (Support & Safety)

Ngôn ngữ: [English](./support.md) | **Tiếng Việt**

Tài liệu tham chiếu này mô tả quy trình chẩn đoán sự cố với `hivemind-doctor`, ranh giới bảo mật chỉ xác thực qua nhà cung cấp (vendor-auth-only), cơ chế cấp quyền trực tiếp, hủy lượt chạy, tất toán công cụ gốc và khôi phục phiên trong `hmoc-steward-acp`.

---

## 1. Chẩn đoán và khắc phục sự cố với hivemind-doctor

Bạn có thể chạy chẩn đoán qua hội thoại với `/hivemind-setup` hoặc yêu cầu agent OpenCode gọi `hivemind-doctor`. Trong `0.4.0`, agent truy cập `hivemind-doctor` qua bề mặt `execute` (CodeMode) của OpenCode, không phải công cụ có tên: nó tìm bên trong `execute` một cách đồng bộ rồi gọi đúng đường dẫn trả về. Lệnh bị chặn nêu đúng phạm vi còn thiếu — bản thân `execute`, hoặc đúng công cụ/hành động; plugin không bao giờ tự mở rộng quyền.

| Hành động (`action`) | Cần sự đồng ý | Chức năng thực hiện | Tác động tới trạng thái & mức sẵn sàng |
|---|---|---|---|
| `check` (mặc định `check: "initialize"`) | Không | Kiểm tra đường dẫn `acpBinary`, phiên bản `cliCompanion` tùy chọn (`devin` hoặc `agy` CLI), chạy bắt tay ACP `initialize` có giới hạn thời gian, và đọc trạng thái đăng nhập cùng `authEpoch`. | Báo cáo `readinessTier: "initialize-checked"` (hoặc `"blocked"` nếu lỗi) mà không tạo phiên hay tốn chi phí câu lệnh. |
| `check` với `check: "session"` | Có (`consent: true` và chấp thuận hộp thoại `hivemind-session-probe` của hệ thống) | Chạy kiểm tra chẩn đoán `initialize` -> `session/new` không gửi câu lệnh trong tối đa `<=15s` rồi đóng tiến trình ngay lập tức. | Báo cáo `readinessTier: "session-checked"` và đóng dấu `CatalogEvidence` (`catalog-evidence/<harnessId>`). Nêu rõ rằng `session/new` có thể kích hoạt kiểm tra xác thực qua mạng, khởi chạy MCP server ngoại lai và tạo phiên lưu trữ phía nhà cung cấp. |
| `install` | Có (chấp thuận hộp thoại `hivemind-install` của hệ thống) | Tải và xác minh máy chủ ACP Antigravity `1.4.0` trên macOS/Windows được hỗ trợ, hoặc hướng dẫn cài `devin` CLI. | Giữ nguyên trạng thái nếu hộp thoại bị từ chối hoặc không khả dụng. |
| `sign-in` | Có (chấp thuận hộp thoại `hivemind-sign-in` của hệ thống) | Dùng luồng `authenticate(methodId)` do máy chủ ACP công bố. Devin có thể yêu cầu `devin auth login`. Đăng nhập CLI `agy` riêng không chứng minh máy chủ ACP đã đăng nhập. | Chỉ ghi `signed-in` khi có kết quả xác nhận, rồi tăng `authEpoch`. |
| `sign-out` | Có (chấp thuận hộp thoại `hivemind-sign-out` của hệ thống) | Cập nhật bản ghi đăng nhập cục bộ của plugin (`state: "signed-out"`) và tăng `authEpoch`. | Nếu hộp thoại bị từ chối hoặc không khả dụng, hành động dừng lại trước khi thay đổi trạng thái; khi được chấp thuận, thu hồi các tiến trình engine đang rảnh thuộc thế hệ cũ nhưng vẫn giữ nguyên các lượt chạy đang bận và các yêu cầu cấp quyền đang chờ trả lời. |

> [!NOTE]
> **Ranh giới của `sign-out` cục bộ:** Khi gọi `hivemind-doctor` với `action: "sign-out"`, plugin trước tiên hỏi sự chấp thuận của bạn qua hộp thoại đồng ý `hivemind-sign-out` của hệ thống, sau đó chỉ cập nhật bản ghi cục bộ và làm mới bể tiến trình đang rảnh. Hành động này không bao giờ gọi RPC `logout` chưa được kiểm chứng trên đường truyền ACP và không xóa hay sửa đổi thông tin xác thực do `devin` hoặc `agy` lưu trên máy của bạn.

---

## 2. Ranh giới xác thực vendor-auth-only và cấu hình không chứa bí mật

Plugin thực thi nghiêm ngặt ranh giới **vendor-auth-only** (`REQ-HARN-25` và `RC-01`):
- **Không trích xuất hay lưu trữ thông tin xác thực:** Plugin không bao giờ đọc, trích xuất, nhập hoặc lưu trữ dữ liệu từ macOS Keychain, cơ sở dữ liệu SQLite, OAuth access/refresh token, khóa API hay địa chỉ email tài khoản. Toàn bộ thông tin xác thực nằm hoàn toàn trong kho lưu trữ chính thức của nhà cung cấp (`devin auth login`, `agy auth login`, hoặc bước bắt tay `authenticate` của ACP).
- **Bộ đếm thế hệ `authEpoch` không lộ danh tính:** Mỗi khi trạng thái xác thực thay đổi (`sign-in`, `sign-out` cục bộ, hoặc gặp lỗi thuộc lớp `auth` trên đường truyền), plugin tăng số nguyên đơn điệu `authEpoch`. Các tiến trình rảnh và bản chụp `CatalogEvidence` mang dấu `authEpoch` cũ sẽ tự động bị đánh dấu hết hạn mà không cần lưu bất kỳ dấu vân tay thông tin xác thực nào.
- **Cấu hình không phải kho bí mật (`RC-01`):** Bộ phân tích từ chối một số tên trường thông tin xác thực, không thể nhận diện mọi bí mật hay tên trường thay thế. Không nhập thông tin xác thực vào `config/delegate`, tùy chọn plugin, câu trả lời thiết lập hay prompt. Tuân thủ [thông báo bảo mật](./quickstart.vi.md) khi dùng `opencode/big-pickle`.

---

## 3. Trung gian cấp quyền và phạm vi lưu quyền

Khi agent ngoại lai cần thực hiện thao tác nhạy cảm, nó sẽ gửi yêu cầu cấp quyền qua ACP bằng `session/request_permission`:

### Thẻ yêu cầu cấp quyền trực tiếp trong Hosted View
- **Chỉ do người dùng quyết định:** Các lane Hosted không bao giờ tự động phê duyệt quyền. Mọi yêu cầu đều hiển thị thành thẻ **Permission Request** ngay trong Entry Log với các lựa chọn đánh số (`1..9` để chọn, **esc** để hủy).
- **Hiển thị nguyên vẹn lựa chọn và cảnh báo của agent:** Thẻ hiển thị chính xác các lựa chọn do agent ngoại lai công bố (`allow_once`, `reject_once`, `allow_always`) cùng cảnh báo bảo mật nguyên bản (như `agy.security.warning` của Antigravity), không bao giờ tự tạo thêm tùy chọn lưu vĩnh viễn.
- **Thông báo phạm vi ngoại lai:** Mỗi thẻ đều nêu rõ rằng **Persistence scope** (phạm vi theo lượt, theo phiên hay lưu qua các lần khởi động lại) hoàn toàn do tiến trình agent ngoại lai kiểm soát, không phải do plugin lưu hay áp đặt.

### Cầu nối cấp quyền trong Native delegation và tool-lane
- **Native delegation (`hivemind-delegate`, `delegate-devin`):** Các phiên con chạy trong **Delegated mode** của Harness (`bypass` với Devin, `yolo` với Antigravity) và chuyển tiếp mọi yêu cầu `session/request_permission` phát sinh sang `Permission.Service` gốc của OpenCode (sự kiện `permission.asked`), trong đó `reject` sẽ từ chối còn `once` / `always` sẽ trả lời theo chính sách phiên.
- **Tool-lane `devin_session`:** Mặc định đặt `autoApprove: false` (`auto_approve: false`), tự động trả lời `cancelled` trừ khi bạn chủ động truyền `auto_approve: true` cho tác vụ đã tin cậy. Khi bật `auto_approve: true`, các nhãn mang nghĩa từ chối sẽ bị loại trừ trước và các tập lựa chọn mơ hồ sẽ trả về `cancelled`.

### Ranh giới cấp quyền CodeMode (0.4.0)
- **Hai lớp cổng:** lệnh CodeMode cần cả quyền `execute` bên ngoài lẫn quyền công cụ/hành động lá có tên. Việc cho phép một thẻ quyền gốc không bao giờ cấp một trong hai cổng này.
- **Deny tường minh thắng:** thẻ cho phép đã lưu không thể ghi đè `deny` tường minh trong `opencode.json`. Plugin không bao giờ tự mở rộng quyền và không hứa tự sửa quyền của chính nó.
- **Sửa cấu hình theo đồng ý:** khi người dùng yêu cầu sửa quyền, agent hỏi đúng phạm vi sửa dự án hoặc toàn cục và dùng công cụ sẵn có của host. Các bước thủ công chỉ được đưa ra khi những công cụ đó bị từ chối.
- **Đúng đường dẫn:** agent tìm bên trong `execute` một cách đồng bộ và gọi đúng đường dẫn trả về. Không có namespace công cụ cứng, tên đoán mò hay đường eval.

---

## 4. Hủy lượt chạy, tất toán công cụ gốc và khôi phục

### Hủy lượt chạy (**ctrl+x** và tín hiệu hủy)
- **Trong Hosted View (`ctrl+x`):** Nhấn **ctrl+x** sẽ gửi `session/cancel` cho lượt đang chạy, chờ phiên ngoại lai dừng hẳn và giữ nguyên kết nối của lane để bạn gửi câu lệnh tiếp theo hoặc mở Session picker (**ctrl+o**). Trong lúc `session/load` (**Pending binding**) đang diễn ra, các câu lệnh mới và thao tác ghi `session/set_config_option` đều bị khóa chặn để không bao giờ ghi nhầm cấu hình sang phiên khác.
- **Trong Native delegation:** Khi hủy một tác vụ phiên con hoặc kích hoạt `AbortSignal`, plugin sẽ gửi `session/cancel` tới hành trình ngoại lai đang gắn kết và chờ lượt chạy tất toán xong trước khi giải phóng quyền sở hữu (**Ownership release**).

### Tất toán trung thực công cụ và suy luận gốc (`flushUnsettled`)
- **Tất toán rõ ràng công cụ chưa kết thúc (`flushUnsettled`):** Các lời gọi công cụ ngoại lai (`tool_call` và `tool_call_update`) được ghi nhận là quan sát do nhà cung cấp thực thi. Nếu lượt ủy quyền kết thúc khi một công cụ ngoại lai vẫn chưa báo trạng thái kết thúc (terminal), `flushUnsettled` sẽ đóng công cụ đó ngay cuối lượt kèm thông báo lỗi thiếu trạng thái kết thúc rõ ràng, không để vòng quay chờ treo vô hạn và không tự bịa kết quả thành công.
- **Không tạo suy luận giả:** Chỉ các khối `agent_thought_chunk` thực sự do agent ngoại lai gửi về mới được đưa vào luồng suy luận. Nếu Harness không phát sinh khối suy luận nào trong lượt chạy, trình điều khiển sẽ không chèn bất kỳ văn bản suy luận giả tạo nào.

### Khôi phục sau sự cố hoặc khởi động lại
- Mỗi Native child lưu bản ghi gắn kết tại `bindings/<childSessionID>` cùng dấu vết `sentText` và `foreignAck`.
- Nếu mất kết nối sau khi đã gửi câu lệnh, cơ chế khôi phục sẽ nạp lại phiên ngoại lai (`session/load`) và đối chiếu lịch sử phát lại (Replay): nếu thấy bản vọng câu lệnh của người dùng khớp với `sentText`, plugin xác nhận đây là **Accepted turn** (lượt đã được nhận) mà không làm giả `foreignAck`, tiếp tục theo dõi trên cùng hành trình ngoại lai đó thay vì gửi lặp lại câu lệnh hoặc tạo phiên mới.

---

## 5. Phân loại lỗi và bảng xử lý sự cố thường gặp

| Triệu chứng hoặc mã lỗi | Nguyên nhân | Cách khắc phục |
|---|---|---|
| `ENOENT` / `blocked` tại `acpBinary` | Thiếu tệp thực thi `acpBinary` (`devin` hoặc `agy_acp_server.par`) hoặc tệp không có quyền chạy. | Chạy `/hivemind-setup` hoặc `hivemind-doctor` (`action: "check"`), sau đó đồng ý chạy `action: "install"` hoặc cấu hình đường dẫn hợp lệ tại `harnesses.<id>.bin`. Lưu ý rằng `agy` (`cliCompanion`) không thể thay thế cho `agy_acp_server.par` (`acpBinary`). |
| `Failure class: "auth"` hoặc `signed-out` | Xác thực nhà cung cấp thất bại hoặc bản ghi cục bộ là `signed-out`. | Yêu cầu `hivemind-doctor` với `action: "sign-in"`, rồi đồng ý hộp thoại gốc. Làm theo luồng của máy chủ ACP đã chọn; Devin có thể yêu cầu `devin auth login`. |
| `Failure class: "quota"` | Tài khoản nhà cung cấp đã hết hạn mức sử dụng hoặc tín dụng. | Cơ chế thử lại tự động dừng ngay lập tức. Hãy kiểm tra hạn mức tài khoản nhà cung cấp hoặc đổi thứ tự ưu tiên **Harness rank** qua `/hivemind-setup`. |
| `Failure class: "invalid"` khi tạo phiên | Lựa chọn yêu cầu không có trong danh mục mới của phiên. | Dùng `/hivemind-setup đổi chính sách model` để chọn model hoặc `model#variant` được công bố. Có thể đồng ý kiểm tra phiên để làm mới `CatalogEvidence`. Phiên Hosted không tự áp dụng mặc định. |
| `tui-owner-absent` | Không có phiên Hosted TUI phù hợp đang gắn kết. | Mở `/devin` hoặc `/antigravity`, gắn phiên rồi dùng bộ chọn hoặc yêu cầu thay đổi có mục tiêu. Không có chính sách mặc định `hosted.<harness>` tự động. |
| `ambiguous-owner-conflict` | Có nhiều cửa sổ TUI đang cùng gắn kết vào một khe `(Harness, cwd)`. | Đóng bớt cửa sổ TUI trùng lặp hoặc nhấn trực tiếp **ctrl+m**, **ctrl+e**, **ctrl+t** bên trong màn hình Hosted View bạn muốn đổi. |
| `expired` / `owner-gone` / `unconfirmed` | Yêu cầu đổi tùy chọn phiên Hosted quá hạn (`15s` TTL) hoặc TUI bị nạp lại trong lúc chờ. | Kiểm tra lại nhãn trạng thái trên thanh tiêu đề Hosted View (`expired`/`owner-gone` đã từ chối trước khi ghi; `unconfirmed` báo hiệu lệnh ghi đã gửi đi trước khi mất kết nối). |

### Thiếu lệnh `/devin` hoặc `/antigravity` trong palette

Nếu cả palette lẫn danh sách gợi ý `/` đều không hiển thị `/devin` hay `/antigravity`, và `/plugins` cũng không báo lỗi, thì có thể TUI đã không nạp được plugin này. Nguyên nhân thường gặp là mở TUI ở ngoài thư mục dự án chứa mục plugin. Trong OpenCode `2.1.2`, TUI dò plugin từ thư mục bạn khởi chạy nó và chốt kết quả dò ngay tại thời điểm khởi chạy.

- Bản cài **theo dự án** chỉ được tìm thấy khi bạn mở TUI từ chính thư mục dự án đó. Khởi chạy ở nơi khác sẽ để palette trống dù plugin đã được cài đúng.
- Bản cài **toàn cục** (`~/.config/opencode/plugins/`) được tìm thấy từ mọi thư mục khởi chạy, nên triệu chứng này thường thuộc về một mục cài theo dự án.

Xem [Hướng dẫn bắt đầu nhanh](./quickstart.vi.md) để biết mục cài theo dự án.

Hãy mở TUI từ dự án chứa mục plugin, rồi gắn lại đúng phiên bạn đang có:

```sh
cd /path/to/project && opencode -s <session-id>
```

Cờ `-s` chọn phiên cần tiếp tục trên máy chủ; cờ này không thêm ngữ cảnh plugin của dự án khởi chạy vào một TUI đã mở ở nơi khác. Đổi thư mục làm việc của phiên hay chọn dự án khác ngay trong TUI cũng không thay đổi việc dò plugin, vì vị trí dò plugin của TUI đã cố định từ lúc khởi chạy. Khởi động lại service nền hay thay bản build plugin cũng không giúp gì vì cùng lý do.

Tiếp tục phiên bằng `-s` vẫn giữ nguyên lịch sử trò chuyện, nên đừng cài lại plugin và đừng tạo phiên mới.

Việc thiếu `/freebuff` là đúng như thiết kế: Freebuff chỉ được đăng ký cho delegation và không khai báo Hosted View, nên nó không có lệnh palette hay route hosted nào.

---

## 6. Giới hạn nền tảng và công cụ thế hệ cũ

- **Các Harness được hỗ trợ:** Devin (`devin-native`) và Antigravity (`antigravity-native`) là hai Harness hosted trong `0.3.0` và `0.4.0`. Freebuff (`freebuff-native`) là lane ủy quyền riêng ở chế độ deny, không có Hosted View. Claude Code và Cursor là các cột mốc tiếp theo trong [ROADMAP.md](../ROADMAP.md) và chưa thể kích hoạt hay định tuyến.
- **Nền tảng cài đặt Antigravity được quản lý:** Máy chủ `1.4.0` có gói đã đo cho macOS arm64/x64 và Windows arm64/x64 (`agy_acp_server.par` hoặc `agy_acp_server.exe`). Bản này chưa hỗ trợ cài được quản lý trên Linux. Chỉ định đường dẫn riêng không tự chứng minh khả năng tương thích.
- **Tool-lane `devin_session`:** Phiên Devin bền vững qua ACP vẫn được giữ, gồm cả đầu vào `cloud` và `agent_type`, nhưng không tạo ra Native child (`ses_*`) của OpenCode và không tham gia định tuyến đa Harness theo `rankOrder`. Trong `0.4.0`, công cụ một lần `devin_run` và công cụ trùng lặp `devin_doctor` bị gỡ bỏ, không có alias; `hivemind-doctor` là bề mặt doctor duy nhất. Hãy ưu tiên dùng `hivemind-delegate` và `delegate-devin` cho quy trình làm việc với phiên con gốc.
- **Freebuff admission (0.4.0):** Một lượt Freebuff chỉ bắt đầu sau khi chủ động bật: bật trong tài liệu `config/delegate` hoặc chọn `harness: "freebuff"`. Mỗi lượt chạy mở một session lease do máy chủ cấp với hạn mức chi tiêu ví theo phiên; hủy lượt chạy chỉ dừng lượt đó chứ không thu hồi cửa sổ phía thượng nguồn. Lượt chạy thật có quyền vẫn chưa được kiểm chứng: chưa có bằng chứng về prompt có quyền chạy thật hoặc tính phí. `0.3.0` chỉ phân giải credential từ `CODEBUFF_API_KEY`. `0.4.0` dự định đăng nhập bằng CLI Freebuff native với phương án dự phòng device-key đã đăng ký sẵn; khóa môi trường tường minh vẫn hợp lệ, và đường đi này đang chờ đánh giá an toàn.

---

## 7. Tài liệu liên quan và báo cáo bảo mật

- Cài đặt lần đầu và lưu ý quyền riêng tư của model miễn phí `opencode/big-pickle`: [Hướng dẫn bắt đầu nhanh](./quickstart.vi.md) ([English](./quickstart.md))
- Quy trình làm việc hằng ngày với Hosted View và Native delegation: [Hướng dẫn vận hành](./operations.vi.md) ([English](./operations.md))
- Bảng so sánh môi trường và quy ước tiền tố câu lệnh (`/`, `@`, `!`, `#`): [Tham chiếu khả năng Hosted CLI](./hosted-cli.md)
- Chính sách báo cáo lỗ hổng bảo mật và mô hình mối đe dọa: [SECURITY.md](../SECURITY.md)
- Tổng quan dự án, lộ trình, bảng thuật ngữ và nhật ký thay đổi: [README.md](../README.md), [PROJECT.md](../PROJECT.md), [ROADMAP.md](../ROADMAP.md), [GLOSSARY.md](../GLOSSARY.md) và [CHANGELOG.md](../CHANGELOG.md)
