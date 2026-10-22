# Hướng dẫn vận hành (Operations)

Ngôn ngữ: [English](./operations.md) | **Tiếng Việt**

Tài liệu này hướng dẫn quy trình vận hành hằng ngày trên Hosted Views (`/devin`, `/antigravity`), Native delegation (`hivemind-delegate`, `delegate-devin`), chính sách khởi tạo, điều chỉnh tùy chọn phiên đang mở và một lượt kiểm tra sức khỏe tĩnh, đã lưu (preflight theo thứ hạng chỉ còn nội bộ trong định tuyến `hivemind-delegate` — kiểm tra engine trực tiếp chỉ chạy qua `hivemind-doctor`).

---

## 1. Làm việc trong Hosted Views (/devin và /antigravity)

Mỗi route Hosted View (`/devin` hoặc `/antigravity`) kết nối tới một **Lane** thuộc tiến trình TUI được định danh theo cặp `(Harness, cwd)`. Lane tồn tại độc lập với việc chuyển màn hình: nhấn **esc** sẽ đưa bạn về màn hình chính của OpenCode trong khi phiên đang gắn kết vẫn tiếp tục chạy ở chế độ nền.

### Bộ chọn phiên ưu tiên thư mục làm việc (Session picker)
Khi bạn mở một route chưa gắn phiên hoặc nhấn **ctrl+o** trên một lane đang rảnh, **Session picker** sẽ hiển thị danh sách phiên từ `session/list`:
- **`+ New session in <cwd>`:** Luôn cố định ở dòng đầu tiên để bạn tạo ngay phiên mới trong thư mục làm việc hiện tại (`cwd`).
- **Current workspace:** Các phiên hiện có cùng thư mục `cwd` với không gian làm việc hiện tại, được nhóm theo thời gian hoạt động gần nhất.
- **Other workspaces:** Các phiên được tạo ở những thư mục khác trên máy hoặc tài khoản của bạn, hỗ trợ tiếp tục phiên xuyên thư mục (**Continuation** qua `session/load`). Mỗi dòng hiển thị rõ đường dẫn thư mục, thời gian cập nhật, ký hiệu đang gắn kết `●` và biểu tượng khóa `⚿` (`isLocked` / `lockHolderPid`) nếu phiên đang được giữ bởi một client khác.
- **Hiển thị rõ `cwd` trên thanh tiêu đề:** Ngay khi phiên được gắn kết qua `session/new` hoặc `session/load`, thanh tiêu đề của Hosted View hiển thị đường dẫn `cwd` của phiên đó cùng với nhãn model, mode và mức suy luận để bạn không bao giờ nhầm thư mục khi tiếp tục phiên từ nơi khác.

---

## 2. Sử dụng gợi ý / và @ có gắn nhãn nguồn gốc trong Composer

Trong ô nhập liệu **Composer** của Hosted View, khi bạn gõ ký tự `/` ở đầu câu (hoặc nhấn **ctrl+g**) hay gõ `@`, bảng gợi ý **Origin-labelled completion** sẽ gắn nhãn nguồn gốc chính xác cho từng mục:

| Nhãn nguồn gốc | Nguồn dữ liệu | Hành vi khi chọn và gửi |
|---|---|---|
| `[foreign command]` | Được phiên ACP ngoại lai công bố qua `available_commands_update`. | Chèn `/<command> ` vào Composer mà không tự động gửi; khi nhấn gửi sẽ truyền đi dưới dạng khối văn bản `session/prompt` thông thường. Nếu phiên chưa công bố lệnh nào, bảng gợi ý hiển thị `[foreign command] no commands — this session has not reported any`, không tự bịa lệnh và không chặn phím Enter. |
| `[client control]` | Các điều khiển cục bộ của Hosted View (`sessions`, `model`, `mode`, `thinking`, `filter`, `help`). | Mở bộ chọn cục bộ, bộ lọc nhãn hoặc bảng phím tắt ngay trong TUI của OpenCode mà không phát sinh bất kỳ lời gọi `session/prompt` nào trên đường truyền. |
| `[host skill]` | Các tệp `SKILL.md` trong thư mục dự án được tìm thấy tại `.agents/skills`, `.devin/skills`, `.gemini/skills` và `.opencode/skills`. | Chèn thẻ `@<skill-path>` vào Composer và phân giải thành khối `resource_link` (`file://` URI) chuẩn khi gửi. Không bao giờ tự chạy lệnh shell `!` hay giả lập runtime của công cụ `skill` bên phía host. |
| `[host reference]` | Các mục `path` cục bộ khai báo trong `references` của `<workspace>/opencode.json`. | Chèn thẻ `@<reference-path>` vào Composer và phân giải thành khối `resource_link` (`file://` URI) chuẩn khi gửi. |

Quy tắc đính kèm tệp bằng `@`:
- Gõ `@<đường-dẫn-tương-đối>` để gợi ý các tệp bên trong thư mục gốc của workspace (liên kết tượng trưng symlink được kiểm tra `realpath` để chặn truy cập vượt ra ngoài thư mục).
- Gõ đường dẫn tuyệt đối tường minh bắt đầu bằng `@/` tới một tệp hoặc thư mục có thật trên máy sẽ tạo khối `resource_link` (`file://` URI), nhưng quyền đọc thực tế vẫn do chính sách bảo mật của agent ngoại lai quyết định.

---

## 3. Ủy quyền tác vụ với hivemind-delegate và delegate-devin

Agent chính của OpenCode có thể giao việc qua hai công cụ ủy quyền gốc trong khi công cụ `subagent` tích hợp sẵn vẫn giữ nguyên không đổi:

| Công cụ | Hành vi định tuyến | Trường hợp sử dụng |
|---|---|---|
| `hivemind-delegate` | Chạy **Preflight** theo thứ tự ưu tiên **Harness rank** khi bỏ trống `model`, hoặc định tuyến thẳng tới provider ghi trong `model` (`devin-native/*` hoặc `antigravity-native/*`). Trường `harness` tùy chọn (ví dụ `harness: "antigravity"`) ghim lời gọi vào một Harness đã đăng ký. | Ủy quyền tổng quát theo thứ tự ưu tiên Harness bạn đã cấu hình, hoặc ghim thẳng vào một Harness. |
| `delegate-devin` | **Pinned alias** cố định vào `devin-native`. | Tác vụ chỉ định rõ cho Devin. |

Quy tắc thực thi và tiếp tục phiên con:
- **Chạy trực tiếp (foreground) và chạy nền (background):** `background: false` (mặc định) chờ lượt chạy của phiên con hoàn tất và trả kết quả về ngay; `background: true` khởi chạy một **Native child** (`ses_*`) ở chế độ nền và gửi kết quả về phiên cha khi hoàn thành.
- **Chạy song song và giới hạn tùy chọn:** Các phiên con độc lập có thể chạy chồng lấp ở cả hai chế độ mà không bị giới hạn số lượng mặc định. Nếu bạn bật giới hạn theo chế độ (`limits.foreground`, `limits.background` trong `config/delegate`), các lời gọi vượt ngưỡng sẽ dừng sớm (fail fast) trước khi tạo phiên con.
- **Tiếp tục phiên con và kiểm kê:** Truyền `sessionID` (`ses_*`) của phiên con OpenCode để thực hiện **Child continuation** trên cùng hành trình ngoại lai đã gắn kết. Dùng `devin-children` với `action: "list"` và `action: "inspect"` để tra cứu danh sách phiên con và nguồn gốc gắn kết `foreign.sessionID`.

---

## 4. Cấu hình chính sách ủy quyền phiên con mới

Plugin lưu cấu hình mặc định cho phiên con ủy quyền trong tương lai tại `config/delegate` (quản lý qua `/hivemind-setup`):

### Chính sách khởi tạo phiên con tương lai (`delegate.<harness>`)
- Quản lý `models` tùy chọn (chuỗi **Model slot chain** gồm 1 vị trí chính và tối đa 2 vị trí dự phòng) và giới hạn `limits`. Dùng `model#variant` được công bố để chọn mức suy luận của model đó. Chính sách `thinking` riêng đã ngừng dùng; bỏ trống models để giữ mặc định của phiên ngoại lai.
- Chỉ áp dụng cho các Native child **mới tạo** trong khoảng thời gian **Bound-but-unprompted child** (sau khi `session/new` và lưu bản ghi gắn kết, trước khi gửi `session/prompt` đầu tiên).
- Nếu model ở vị trí chính không có trong danh mục `session/new` vừa trả về, quá trình nhận việc sẽ chuyển sang vị trí kế tiếp trong chuỗi; nếu mọi vị trí đều không hợp lệ, lời gọi sẽ dừng an toàn trước khi gửi câu lệnh.
- Các phiên con đang chạy, đã tồn tại hoặc được tiếp tục (`session/load`) không bao giờ bị ghi đè cấu hình hay đổi sang Harness khác.

### Phiên Hosted không ghi cấu hình tự động
- `HostedCreationPolicy` / khối `hosted.<harness>` đã ngừng áp dụng: `session/new` gắn kết danh mục mà phiên công bố nguyên trạng và **không** phát sinh lời gọi `session/set_config_option` hay `session/set_mode` nào. Một khối `hosted.<harness>` cũ trong `config/delegate` được giữ nguyên như dữ liệu người dùng và không bao giờ kích hoạt ghi.
- Đổi `model`, `mode` hay `thinking` của phiên đang gắn kết qua các bộ chọn tương tác hoặc yêu cầu có mục tiêu qua agent ở mục 5.
- Việc mở lại một phiên cũ qua `session/load` (**Continuation**) cũng không ghi và không ghi đè thiết lập sẵn có của phiên đó.

---

## 5. Điều chỉnh tùy chọn phiên Hosted đang mở một cách an toàn

Để xem hoặc đổi `model`, `mode` hay `thinking` trên một phiên Hosted **đang gắn kết**, bạn có hai cách được hỗ trợ:

1. **Phím tắt trực tiếp trong TUI:** Khi đang ở trong `/devin` hoặc `/antigravity`, nhấn **ctrl+m** (chọn model), **ctrl+e** (chọn mode) hoặc **ctrl+t** (chọn mức suy luận trên Devin) để chọn từ danh sách `configOptions` do phiên đó công bố.
2. **Yêu cầu có mục tiêu qua agent (`/hivemind-setup`):** Yêu cầu agent OpenCode bằng tiếng Việt hoặc tiếng Anh, ví dụ:
   - `"Đổi model trong phiên Antigravity hiện tại"`
   - `"Change the model in my current Antigravity session"`

Cơ chế bảo vệ phiên đang mở khi điều chỉnh qua agent:
- Mỗi Lane trên TUI đăng ký một thẻ năng lực có xác thực (`registerOwnerHandle`) tại địa chỉ dịch vụ RPC của plugin (`pluginLocation`) kèm theo thư mục `cwd` của Lane trong dữ liệu gửi, tách biệt mã công khai `handleId` với mã bí mật phản hồi `handleSecret`.
- Khi bạn yêu cầu agent xem hoặc đổi tùy chọn phiên hiện tại, máy chủ chỉ gửi yêu cầu tới đúng `targetHandleId` khớp điều kiện. Các cửa sổ TUI khác sẽ tự động bỏ qua sự kiện này.
- Nếu không có TUI Lane nào đang gắn kết, công cụ từ chối ngay với mã `tui-owner-absent`. Nếu có nhiều cửa sổ TUI cùng giữ một khe `(Harness, cwd)`, công cụ từ chối với mã `ambiguous-owner-conflict`.
- Trước khi ghi bất kỳ thay đổi nào xuống phiên ngoại lai, cửa sổ TUI đích luôn yêu cầu bạn xác nhận trực tiếp qua hộp thoại (`ui.dialog.confirm`) và kiểm tra chốt chặn cuối cùng đối với trạng thái gắn kết, `catalogFingerprint` và trạng thái chưa bị hủy.

---

## 6. Chạy một lượt health, rồi hiểu 5 mức độ sẵn sàng

Bạn không cần kiểm tra đầu mỗi phiên: thiết lập đã lưu chính là trạng thái. Khi có gì đó trục trặc, `hivemind-setup` với `action: "health"` trả lời trong một lượt đọc tĩnh — `ready`, `action-required` hoặc `blocked` kèm đúng một cách khắc phục — và dùng lại thiết lập đã lưu thay vì đi lại từ đầu. `hivemind-doctor` (`action: "check"`) vẫn là chẩn đoán khi bạn muốn xem chi tiết sẵn sàng từng harness:
- **Định tuyến dừng sớm giữ nguyên thứ hạng (nội bộ):** Preflight nội bộ của `hivemind-delegate` đánh giá các Harness đang bật theo đúng thứ tự `rankOrder` đã cấu hình và dừng lại ở Harness khả dụng có thứ hạng cao nhất. Nó không bao giờ chạy đo tốc độ (benchmark), không tự ý đảo thứ hạng theo độ trễ, và không tạo phiên con thử nghiệm hay gửi câu lệnh tốn phí. Tiếp xúc engine trực tiếp vẫn thuộc `hivemind-doctor`: kiểm tra initialize có giới hạn, còn `depth: "session"` yêu cầu `consent: true`.

**Bề mặt công cụ (CodeMode):** `hivemind-setup`, `hivemind-doctor` và `hivemind-hosted` được khám phá qua công cụ `execute` của host, không còn là tool có tên trên provider — agent chạy `search({query})` bên trong `execute` rồi gọi đúng path trả về. Khi bị chặn, lỗi nêu rõ scope/action còn thiếu (`execute` chính nó, hoặc tool/action bên trong nó); rule `deny` tường minh luôn thắng thẻ allow đã lưu. Mọi sửa đổi quyền trong `opencode.json` chỉ áp dụng khi bạn đồng ý, ở phạm vi project hoặc global bạn chọn, giữ nguyên các rule và thứ tự plugin khác — không bao giờ dùng allow đại trà `hivemind-*`.

Mọi báo cáo health, bước tiếp theo và `hivemind-doctor` đều phân loại mức độ sẵn sàng vào 5 cấp độ trung thực (`ReadinessTier`):

| Cấp độ sẵn sàng | Nội dung đã kiểm tra | Điều chưa được chứng minh |
|---|---|---|
| `static-checked` | Tệp `acpBinary` tồn tại và có quyền thực thi trên ổ đĩa; đọc cấu hình cục bộ và trạng thái đăng nhập đã ghi mà không khởi chạy tiến trình con. | Chưa chứng minh tệp thực thi khởi chạy được, bắt tay ACP thành công, đã xác thực với nhà cung cấp hay còn hạn mức (quota). |
| `initialize-checked` | Tiến trình con đã khởi chạy và hoàn tất bắt tay ACP `initialize` trong thời gian giới hạn, không tạo phiên. | Chưa chứng minh tài khoản còn hiệu lực đăng nhập trực tuyến, danh mục model khả dụng hay còn hạn mức câu lệnh. |
| `session-checked` | Đã được người dùng đồng ý tường minh (`consent: true`) để chạy kiểm tra `initialize` -> `session/new` không câu lệnh trong `<=15s`, đóng tiến trình ngay và đóng dấu `CatalogEvidence` (`catalog-evidence/<harnessId>`). | Chứng minh tạo được `session/new` và đọc danh mục tại thời điểm `observedAt`, nhưng không chứng minh còn hạn mức câu lệnh hay tình trạng của các máy chủ MCP bên ngoài. |
| `blocked` | Thiếu hoặc không chạy được `acpBinary`, trạng thái đăng nhập đang là `signed-out`, hoặc cấu hình sai định dạng. | Không thể sử dụng cho đến khi khắc phục qua thiết lập hoặc `hivemind-doctor`. |
| `unknown` | Harness chưa có hồ sơ hoặc chưa được theo dõi. | Chưa có thông tin xác minh. |

**Bằng chứng danh mục có đóng dấu phạm vi (`CatalogEvidence`):** Khi bước kiểm tra `session-checked` (có sự đồng ý) hoặc một phiên thực tế quan sát được danh mục trên đường truyền, plugin lưu lại gợi ý `CatalogEvidence` không mang tính áp đặt kèm dấu phạm vi `(harness, acpBinary, authEpoch, cwd, observedAt)`. Luồng thiết lập và doctor chỉ dùng gợi ý này khi dấu phạm vi khớp với tệp thực thi, thế hệ xác thực `authEpoch` và thư mục hiện tại; lúc tạo phiên thực tế, plugin vẫn luôn kiểm tra trực tiếp với danh mục trên đường truyền.

---

## 7. Tài liệu liên quan

- Cài đặt lần đầu, thiết lập với `/hivemind-setup` và lưu ý quyền riêng tư của `opencode/big-pickle`: [Hướng dẫn bắt đầu nhanh](./quickstart.vi.md) ([English](./quickstart.md))
- Xử lý sự cố với doctor, ranh giới xác thực vendor-auth-only, cấp quyền, hủy tác vụ và khôi phục: [Tham chiếu hỗ trợ và an toàn](./support.vi.md) ([English](./support.md))
- Bảng so sánh môi trường và quy ước tiền tố câu lệnh (`/`, `@`, `!`, `#`): [Tham chiếu khả năng Hosted CLI](./hosted-cli.md)
- Tổng quan kho mã, bảng thuật ngữ và chính sách bảo mật: [README.md](../README.md), [PROJECT.md](../PROJECT.md), [ROADMAP.md](../ROADMAP.md), [GLOSSARY.md](../GLOSSARY.md) và [SECURITY.md](../SECURITY.md)
