# Hướng dẫn bắt đầu nhanh (Quickstart)

Ngôn ngữ: [English](./quickstart.md) | **Tiếng Việt**

Tài liệu hướng dẫn này giúp bạn đi từ bản cài đặt [OpenCode V2](https://opencode.ai/v2/docs) mới đến phiên hosted đầu tiên (`/devin`, `/antigravity`) và tác vụ ủy quyền gốc (`hivemind-delegate`, `delegate-devin`) với plugin `hmoc-steward-acp`.

---

## 1. Kiểm tra điều kiện tiên quyết theo từng Harness

Bạn cần host OpenCode V2 trong khoảng hỗ trợ: `opencode --version` trả về v2.1.4 trở lên trong nhánh v2 (dưới v3) trên `PATH`. Host V1 không thể nạp điểm vào TUI của plugin, và v2.0.x nằm dưới mức hỗ trợ tối thiểu. Khoảng hỗ trợ công bố không đổi trong `0.4.2`. Bằng chứng runtime có giới hạn: 6 route ủy quyền gốc (Devin và Antigravity giao việc cho self, harness đối ứng, và executor) đã chạy thành công với các marker đạt trên OpenCode 2.2.1. Thử nghiệm cô lập trước đó cũng xác nhận đăng ký plugin, session/RPC và TUI với server 2.1.4. Hosted attach, rebind sidecar và UAT TUI từ chủ sở hữu vẫn đang chờ; Freebuff phía thượng nguồn tiếp tục bị chặn.

Trình cài đặt và luồng thiết lập kiểm tra độc lập từng **Harness**. Bạn có thể chỉ dùng Devin, chỉ dùng Antigravity, hoặc kết hợp cả hai:

| Harness | Provider ID | `acpBinary` bắt buộc (máy chủ ACP stdio) | `cliCompanion` tùy chọn (CLI dòng lệnh) | Xác thực |
|---|---|---|---|---|
| **Devin** | `devin-native` | `devin` trên `PATH` (hoặc `DEVIN_BIN` / `harnesses.devin.bin`) | Cùng CLI; mốc phát triển `3000.11.92` | `devin auth login` |
| **Antigravity** | `antigravity-native` | Máy chủ ACP chính thức `1.4.0`: `agy_acp_server.par` trên macOS hoặc `agy_acp_server.exe` trên Windows; bản cài được quản lý hoặc `harnesses.antigravity.bin` | `agy` CLI `1.4.2` là công cụ riêng, không bắt buộc | Luồng `authenticate` do máy chủ ACP công bố |

Quy tắc phân giải tệp thực thi:
- Thiết lập chỉ dùng Antigravity hoàn tất bình thường mà không yêu cầu cài đặt `devin` CLI trên máy.
- Antigravity theo dõi nguồn gốc `acpBinary` (`agy_acp_server.par`) tách biệt hoàn toàn với `cliCompanion` (`agy`). Công cụ `agy` CLI độc lập không bao giờ được dùng làm phương án dự phòng cho `acpBinary`.
- Đường dẫn ghi đè tường minh (`harnesses.devin.bin`, `harnesses.antigravity.bin`) phải trỏ tới một tệp thực thi hợp lệ hoặc tên lệnh có trên `PATH`; đường dẫn thư mục hoặc tệp không có quyền thực thi sẽ bị từ chối an toàn (fail closed).

---

## 2. Cài đặt hmoc-steward-acp

Tài liệu cho `0.4.2`. Cài nó qua OpenCode. Lệnh này thêm plugin ở phạm vi toàn cục:

```bash
opencode plugin add hmoc-steward-acp@0.4.2
```

`0.4.2` là bản vá trên `0.4.1`, vốn là bản vá trên `0.4.0`, vốn được xây trên `0.3.0`. `0.3.0` mang bản sửa lỗi hỗ trợ OpenCode 2.1.2, bản sửa lỗi bind lần đầu, và delegate Freebuff đã ghim. Các bản phát hành trước đó (`0.2.2`, và `0.2.2-test.5` trên thẻ `testing`) có trước bản sửa 2.1.2 — đừng dùng chúng trên host OpenCode 2.1.2.

Muốn chỉ dùng trong dự án này, thêm `"hmoc-steward-acp@0.4.2"` vào mảng `plugins` trong `<repo>/opencode.json`. Cấu hình toàn cục nằm tại `~/.config/opencode/opencode.json`. Giữ nguyên plugin và thiết lập khác. Khi nâng cấp, thay mục cũ thay vì thêm bản trùng. Bản cài npm không cần tải mã từ GitHub hay chạy trình cài đặt riêng.

Hãy khởi động lại OpenCode hoặc mở phiên làm việc mới sau khi cài đặt để host nạp cả plugin máy chủ và các route TUI.

---

## 3. Khởi đầu miễn phí với model opencode/big-pickle

Nếu bạn mới sử dụng OpenCode V2 và chưa kết nối nhà cung cấp model trả phí, bạn có thể chạy luồng thiết lập có hướng dẫn và điều phối tác vụ bằng model miễn phí không cần đăng nhập mặc định của OpenCode (`opencode/big-pickle`) kết hợp với skill `hivemind-setup` đi kèm.

> [!IMPORTANT]
> **Thông báo bảo mật và quyền riêng tư cho điểm cuối `opencode/big-pickle` miễn phí:** Nội dung câu lệnh (prompts), ngữ cảnh tệp đính kèm và phản hồi gửi qua model miễn phí `opencode/big-pickle` có thể được nhà cung cấp ghi nhật ký và lưu giữ để đánh giá hoặc huấn luyện model. Tuyệt đối **không** gửi mã nguồn bí mật, tài sản độc quyền, dữ liệu nhạy cảm của khách hàng hoặc thông tin xác thực khi đang dùng `opencode/big-pickle`. Hãy chuyển model của OpenCode host sang nhà cung cấp riêng tư hoặc gói doanh nghiệp trước khi làm việc với kho mã nhạy cảm.

---

## 4. Chạy thiết lập có hướng dẫn một lần — và nó được lưu lại

Trong giao diện TUI của OpenCode, chạy `/hivemind-setup` (hoặc chỉ cần yêu cầu bằng tiếng Việt hay tiếng Anh tự nhiên):

- **Tiếng Việt:** `"Chạy /hivemind-setup và giúp tôi chọn mặc định."`
- **Tiếng Anh:** `"Run /hivemind-setup and help me choose my defaults."`

Agent dùng công cụ `question` gốc của OpenCode; bạn chọn. Agent thực hiện thao tác và giải thích từng lựa chọn bằng ngôn ngữ của bạn. Chưa có wizard TUI riêng.

**Định tuyến 0.4.0:** agent truy cập `hivemind-setup` qua bề mặt `execute` của OpenCode, không phải công cụ có tên. Nó tìm bên trong `execute` rồi gọi đúng đường dẫn nhận được. Nếu `execute` hoặc một hành động lá bị từ chối, lệnh nêu đúng phạm vi còn thiếu; plugin không bao giờ tự mở rộng quyền. Việc sửa quyền chỉ diễn ra sau khi bạn chọn phạm vi dự án hoặc toàn cục.

1. **Kiểm tra sức khỏe (`action: "health"`)** — một lượt đọc tĩnh. `ready` nghĩa là thiết lập đã được lưu: agent báo và dừng. `action-required` hoặc `blocked` chỉ ra đúng một cách khắc phục.
2. **Lựa chọn (`action: "next"` rồi `action: "choose"`)** — bật Harness, đặt thứ hạng, và tùy chọn model slot hoặc giới hạn. Mỗi thao tác ghi được bảo vệ bởi dấu vân tay của bước và số phiên bản tài liệu (`expectedVersion`); câu trả lời cũ bị từ chối và bước được phát lại.
3. **Hoàn tất (`action: "health"` lần nữa)** — `ready` chỉ xác nhận thiết lập tĩnh, không chứng minh đăng nhập hay hạn mức còn hiệu lực. `config/delegate` là biên nhận đã lưu. Muốn đổi về sau, nêu rõ yêu cầu, ví dụ `/hivemind-setup đổi thứ tự harness`.
4. **Khắc phục, chỉ khi được chỉ tên** — `hivemind-doctor` check/install/sign-in khi bạn đồng ý tường minh. Kiểm tra engine là hành động của doctor, không phải bước thiết lập: 0.4.0 chỉ công bố `health`, `next` và `choose` trên setup, còn lượt kiểm tra doctor tự nguyện sẽ lưu `catalog-evidence/<harnessId>` kèm phạm vi.

Phạm vi: chính sách ủy quyền là toàn cục theo plugin (`config/delegate`). Quyền của host và mục plugin thuộc từng phạm vi — toàn cục (`~/.config/opencode/opencode.json`) hoặc dự án (`<repo>/opencode.json` hay `.opencode/opencode.json`); hãy yêu cầu agent dùng đúng phạm vi bạn muốn. Quy tắc quyền có thứ tự và kết quả khớp cuối cùng thắng; plugin được nạp theo thứ tự mảng, nên mục sau có thể ghi đè mục trước.

---

## 5. Khởi chạy phiên Hosted đầu tiên

Phiên Hosted cho phép bạn làm việc trực tiếp với Devin hoặc Antigravity ngay trong màn hình TUI của OpenCode:

1. Gõ `/devin` hoặc `/antigravity` trong OpenCode TUI (hoặc mở bảng lệnh và chọn **Devin: open session** / **Antigravity: open session**).
2. Khi hộp thoại **Session picker** xuất hiện, chọn **`+ New session in <cwd>`** ở đầu danh sách để tạo phiên mới trong thư mục làm việc hiện tại, hoặc chọn một phiên có sẵn trong nhóm **Current workspace** hay **Other workspaces** để tiếp tục qua `session/load`.
3. Nhập yêu cầu vào ô **Composer**. Gõ `@` để đính kèm tệp trong thư mục làm việc dưới dạng khối `resource_link`, hoặc gõ `/` để mở danh sách gợi ý có gắn nhãn nguồn gốc (`[foreign command]`, `[client control]`, `[host skill]`, `[host reference]`).
4. Dùng phím tắt **ctrl+m** (đổi model), **ctrl+e** (đổi mode), **ctrl+t** (mức suy luận trên Devin), **ctrl+x** để hủy lượt đang chạy, và **esc** để quay lại màn hình chính của OpenCode trong khi phiên vẫn giữ nguyên ở chế độ nền.

---

## 6. Chạy tác vụ Native delegation đầu tiên

Native delegation cho phép agent chính của OpenCode giao việc cho một phiên con **Native child** (`ses_*`) thực thụ do Devin hoặc Antigravity đảm nhiệm:

- **Ủy quyền theo thứ tự ưu tiên (`hivemind-delegate`):** Yêu cầu agent OpenCode:
  `"Dùng hivemind-delegate kiểm tra thư mục src/ và tóm tắt ranh giới các module."`
  Khi không chỉ định `model`, bước **Preflight** sẽ kiểm tra các Harness đang bật theo đúng thứ tự `rankOrder` và chọn Harness khả dụng có thứ hạng cao nhất.
- **Ủy quyền cố định theo Harness (`delegate-devin`, hoặc `hivemind-delegate` với `harness: "antigravity"`):** Yêu cầu agent OpenCode:
  `"Gọi delegate-devin với description 'Kiểm tra logic thử lại', prompt 'Kiểm tra xử lý lỗi trong src/acp.ts', và background: false."`
- Truyền `background: true` để chạy song song nhiều phiên con độc lập ở chế độ nền, hoặc truyền `sessionID` (`ses_*`) của một phiên con đã tạo để thực hiện **Child continuation** trên cùng hành trình ngoại lai (foreign trajectory) đó.
- **`devin_session` ACP chuẩn** vẫn còn cho các cuộc hội thoại Devin bền vững, gồm cả đầu vào `cloud` và `agent_type`. Nó không tạo Native child của OpenCode và không tham gia định tuyến `rankOrder`.

Harness delegate thứ ba, **Freebuff** (`freebuff-native`), đi kèm như một lane chỉ-deny cần bật tường minh: tác vụ được giao với `harness: "freebuff"` chạy trong một phiên con native thật trên bản Freebuff SDK đã biên dịch ghim sẵn trong gói, với mọi công cụ đều bị từ chối — không có hiệu ứng công cụ nào. Nó không bao giờ tự bật và không bao giờ nhận suất dự phòng trong thứ hạng; chỉ chủ động bật mới được nhận. Trong `0.4.0`, mỗi lượt Freebuff bắt đầu bằng việc mở một session lease do máy chủ cấp với hạn mức chi tiêu ví theo phiên; hủy lượt chạy chỉ dừng lượt đó chứ không thu hồi cửa sổ phía thượng nguồn. Credential của nó chỉ được gửi tới endpoint HTTPS first-party hoặc loopback — hàng rào từ chối trước khi bất kỳ header Authorization nào được đính kèm. `0.3.0` chỉ phân giải credential từ `CODEBUFF_API_KEY`. `0.4.0` dự định đăng nhập bằng CLI Freebuff native với phương án dự phòng device-key đã đăng ký sẵn; khóa môi trường tường minh vẫn hợp lệ, và đường đi này đang chờ đánh giá an toàn. Trạng thái 0.4.1: dịch vụ thượng nguồn từ chối các completion chế độ miễn phí bằng `403 free_mode_cli_required` trên cả dòng catalog miễn phí lẫn trả phí, và điều khoản Freebuff đã công bố giới hạn inference bậc miễn phí cho các client tương tác chính thức. Vì vậy harness Freebuff xuất xưởng ở trạng thái chưa cấu hình/thử nghiệm; lựa chọn headless được hỗ trợ là Codebuff SDK trả phí hoặc BYOK bên trong client chính thức.

---

## 7. Bước tiếp theo và tài liệu liên quan

- Quy trình làm việc hằng ngày, bộ chọn phiên, chính sách khởi tạo, điều chỉnh tùy chọn phiên đang mở và 5 mức độ sẵn sàng: [Hướng dẫn vận hành](./operations.vi.md) ([English](./operations.md))
- Chẩn đoán với doctor, ranh giới xác thực vendor-auth-only, cấp quyền, hủy tác vụ và khôi phục: [Tham chiếu hỗ trợ và an toàn](./support.vi.md) ([English](./support.md))
- Bảng so sánh môi trường và quy ước tiền tố câu lệnh (`/`, `@`, `!`, `#`): [Tham chiếu khả năng Hosted CLI](./hosted-cli.md)
- Tổng quan kho mã và kiến trúc: [README.md](../README.md), [PROJECT.md](../PROJECT.md), [ROADMAP.md](../ROADMAP.md), [GLOSSARY.md](../GLOSSARY.md), [SECURITY.md](../SECURITY.md) và [CHANGELOG.md](../CHANGELOG.md)
