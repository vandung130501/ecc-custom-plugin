# 03 - Agents

## Agents là gì?

**Agents** là các AI subagent chuyên biệt được Claude gọi để thực hiện task cụ thể. Khác với Skills (là kiến thức/hướng dẫn), Agents là các "nhân vật AI" độc lập với vai trò rõ ràng.

Ví dụ thực tế:
- **Bạn** = Project Manager → giao task
- **Claude chính** = Tech Lead → điều phối
- **Agents** = Các chuyên gia → làm task cụ thể (architect, reviewer, tester...)

## Cấu Trúc Một Agent File

```markdown
---
name: architect
description: Chuyên gia kiến trúc phần mềm. Dùng khi thiết kế hệ thống mới.
tools: ["Read", "Grep", "Glob"]
model: opus
---

Bạn là senior software architect...

## Vai trò của bạn
- Thiết kế kiến trúc hệ thống
- Đánh giá trade-offs kỹ thuật
...
```

**Giải thích các trường:**

| Trường | Ý nghĩa |
|--------|---------|
| `name` | Tên agent |
| `description` | Khi nào Claude chọn agent này |
| `tools` | Tools agent được phép dùng |
| `model` | Model AI dùng (opus = mạnh nhất, haiku = nhanh nhất) |

## Danh Sách Agents và Khi Nào Dùng

### `architect` - Kiến trúc sư phần mềm

**Model:** opus (mạnh nhất)

**Dùng khi:**
- Thiết kế hệ thống mới từ đầu
- Refactor codebase lớn
- Cần phân tích trade-off kỹ thuật
- Lên kế hoạch scale hệ thống

**Ví dụ:**
```
"Tôi cần thiết kế hệ thống thanh toán cho 100K users/ngày.
Hãy dùng architect agent để phân tích."
```

**Kết quả:** Sơ đồ kiến trúc, ADR (Architecture Decision Records), phân tích pros/cons.

---

### `code-reviewer` - Người review code

**Dùng khi:**
- Review pull request
- Kiểm tra code quality trước merge
- Tìm bug tiềm ẩn

**Ví dụ:**
```
/code-review
(Claude tự gọi code-reviewer agent để review các thay đổi)
```

---

### `tdd-guide` - Hướng dẫn TDD

**Dùng khi:**
- Phát triển tính năng mới theo TDD
- Cần hướng dẫn từng bước RED→GREEN→REFACTOR
- Viết test cho code hiện có

**Ví dụ:**
```
/tdd - Implement hàm xác thực email
```

---

### `planner` - Lập kế hoạch

**Dùng khi:**
- Bắt đầu tính năng lớn, cần phân tích trước
- Chia nhỏ task phức tạp thành steps
- Cần estimate công việc

**Ví dụ:**
```
/plan - Thêm authentication với JWT vào app
```

---

### `security-reviewer` - Chuyên gia bảo mật

**Dùng khi:**
- Phát hiện có vấn đề bảo mật
- Audit code trước release
- Review authentication/authorization code

**Ví dụ:**
```
"Hãy dùng security-reviewer để kiểm tra đoạn code xử lý payment này"
```

---

### `build-error-resolver` - Sửa lỗi build

**Dùng khi:**
- Build thất bại và không biết tại sao
- Có nhiều lỗi phức tạp liên quan nhau

**Ví dụ:**
```
/build-fix
(tự động gọi build-error-resolver khi có lỗi)
```

---

### `database-reviewer` - Review database

**Dùng khi:**
- Review schema database
- Kiểm tra migration an toàn
- Tối ưu query

---

### `doc-updater` - Cập nhật documentation

**Dùng khi:**
- Code thay đổi nhưng docs chưa được update
- Cần generate docs từ code

---

### `e2e-runner` - Chạy E2E tests

**Dùng khi:**
- Generate và chạy Playwright/Cypress tests
- Kiểm tra user flow đầu-cuối

---

### `loop-operator` - Vận hành autonomous loops

**Model:** haiku (nhanh, tiết kiệm)

**Dùng khi:**
- Chạy các task lặp đi lặp lại tự động
- Monitoring và polling

## Cách Claude Gọi Agents

Claude tự động chọn agent dựa trên `description` trong file agent. Bạn cũng có thể chỉ định:

```
"Dùng architect agent để thiết kế module payments"
"Gọi security-reviewer để audit file auth.ts"
```

## Sự Khác Biệt: Agent vs Skill

| | Agent | Skill |
|--|-------|-------|
| **Là gì** | AI subagent độc lập | Kiến thức/hướng dẫn |
| **Chạy như thế nào** | Chạy riêng biệt với tools riêng | Claude chính đọc và áp dụng |
| **Model** | Có thể khác model chính | Cùng model với Claude chính |
| **Ví dụ** | `architect`, `reviewer` | `coding-standards`, `tdd-workflow` |
| **Khi dùng** | Task phức tạp, cần chuyên môn sâu | Hướng dẫn làm đúng cách |

## Ví Dụ Workflow Thực Tế

```
Bạn: "/plan - Xây dựng hệ thống notification realtime"

Claude chính:
  └─ Gọi [planner agent]
      └─ Phân tích requirements
      └─ Đề xuất tech stack (WebSocket vs SSE vs Polling)
      └─ Chia thành 5 tasks nhỏ
      └─ Trả kết quả về

Bạn: "OK, bắt đầu implement task 1"

Claude chính:
  └─ Gọi [architect agent]
      └─ Thiết kế schema database
      └─ Định nghĩa API contracts
  └─ Gọi [tdd-guide agent]
      └─ Viết tests trước
      └─ Implement code
  └─ Gọi [code-reviewer agent]
      └─ Review code vừa viết
      └─ Suggest improvements
```

## Tạo Agent Mới

```bash
# Tạo file agent
touch agents/my-specialist.md
```

Nội dung:

```markdown
---
name: my-specialist
description: Chuyên gia X. Dùng khi cần Y. Tự động kích hoạt khi Z.
tools: ["Read", "Grep", "Bash"]
model: sonnet
---

Bạn là chuyên gia về X với 10 năm kinh nghiệm.

## Vai trò
...

## Quy trình làm việc
1. Bước 1
2. Bước 2
...
```

> **Tip về model:**
> - `opus` = Mạnh nhất, chậm nhất, đắt nhất → Dùng cho task phức tạp (architect, security)
> - `sonnet` = Cân bằng → Dùng cho hầu hết tasks
> - `haiku` = Nhanh nhất, rẻ nhất → Dùng cho task đơn giản, lặp lại
