# 04 - Commands (Slash Commands)

## Commands là gì?

**Commands** là các lệnh tắt bắt đầu bằng `/` mà bạn gõ vào chat Claude. Mỗi command là một file Markdown định nghĩa hành động Claude sẽ thực hiện.

Ví dụ: Thay vì gõ "Hãy theo quy trình TDD, viết test trước, rồi implement, rồi refactor...", bạn chỉ cần gõ `/tdd`.

## Cấu Trúc Một Command File

```markdown
---
description: Mô tả command này làm gì
---

# Tên Command

Nội dung hướng dẫn Claude thực hiện khi command được gọi.

$ARGUMENTS  ← Placeholder cho tham số bạn truyền vào
```

Ví dụ file `commands/tdd.md`:
```markdown
---
description: Legacy slash-entry shim for the tdd-workflow skill.
---

# TDD Command

Apply the `tdd-workflow` skill.
- Stay strict on RED -> GREEN -> REFACTOR.
- Keep tests first, coverage explicit.

$ARGUMENTS
```

## Danh Sách Commands Quan Trọng

### Phát Triển Tính Năng

#### `/tdd [mô tả tính năng]`
**Mục đích:** Phát triển theo Test-Driven Development

**Quy trình:**
1. Viết test thất bại (RED)
2. Implement code tối thiểu (GREEN)
3. Cải thiện code (REFACTOR)
4. Kiểm tra coverage

**Ví dụ:**
```
/tdd - Hàm validate email address
/tdd - Component UserCard hiển thị avatar và tên
```

---

#### `/plan [mô tả tính năng]`
**Mục đích:** Lên kế hoạch trước khi code

**Kết quả bạn nhận được:**
- Phân tích requirements
- Proposed approach
- Danh sách files cần tạo/sửa
- Các bước implementation

**Ví dụ:**
```
/plan - Thêm OAuth2 login với Google
/plan - Tối ưu query database cho trang dashboard
```

**Khi nào dùng:** Trước khi bắt đầu bất kỳ tính năng nào có độ phức tạp trung bình trở lên.

---

#### `/feature-dev [tên tính năng]`
**Mục đích:** Full workflow phát triển tính năng (plan → implement → test → review)

**Ví dụ:**
```
/feature-dev - Shopping cart với persistence
```

---

### Review & Quality

#### `/code-review`
**Mục đích:** Review code đang có thay đổi (staged/unstaged files)

**Kết quả:**
- Danh sách issues tìm được
- Gợi ý cải thiện
- Security concerns

**Ví dụ:**
```
/code-review           ← Review tất cả thay đổi hiện tại
/code-review src/auth  ← Review thư mục cụ thể
```

---

#### `/build-fix`
**Mục đích:** Phân tích và sửa lỗi build/compile

**Khi nào dùng:** Khi `npm run build` hoặc `tsc` có lỗi.

**Ví dụ:**
```
/build-fix
```

---

#### `/e2e [mô tả flow]`
**Mục đích:** Tạo và chạy E2E tests với Playwright

**Ví dụ:**
```
/e2e - Flow đăng ký tài khoản mới
/e2e - Checkout flow từ giỏ hàng đến thanh toán
```

---

### Git & Session

#### `/checkpoint`
**Mục đích:** Lưu trạng thái hiện tại của session

**Khi nào dùng:** Trước khi thực hiện thay đổi lớn, hoặc muốn đánh dấu milestone.

---

#### `/learn`
**Mục đích:** Trích xuất patterns và bài học từ session hiện tại để cải thiện workflow

---

### AI & Orchestration

#### `/orchestrate [task phức tạp]`
**Mục đích:** Chia nhỏ task lớn và chạy nhiều agents song song

**Ví dụ:**
```
/orchestrate - Migrate toàn bộ auth system từ JWT sang OAuth2
```

---

#### `/eval [mô tả]`
**Mục đích:** Đánh giá chất lượng output AI, chạy eval harness

---

#### `/evolve`
**Mục đích:** Cải thiện ECC configuration dựa trên session học được

---

### Tiện Ích

#### `/skill-create`
**Mục đích:** Tạo skill mới từ git history hoặc conversation hiện tại

**Ví dụ:**
```
/skill-create - Tạo skill cho Django REST framework patterns
```

---

#### `/context-budget`
**Mục đích:** Kiểm tra và quản lý context window đang dùng

---

#### `/docs [topic]`
**Mục đích:** Tra cứu documentation

**Ví dụ:**
```
/docs - React useEffect cleanup
/docs - PostgreSQL indexing strategies
```

## Cách Truyền Tham Số

```
/command tham-so-1 tham-so-2

# Ví dụ cụ thể:
/tdd hàm tính thuế VAT cho đơn hàng quốc tế
/plan thêm notification system với WebSocket
/e2e user login flow với 2FA
```

Tham số được nhúng vào vị trí `$ARGUMENTS` trong command file.

## Tạo Command Mới

```bash
touch commands/my-command.md
```

Nội dung:

```markdown
---
description: Mô tả command của tôi
---

# My Command

Hướng dẫn cho Claude khi command này được gọi.

## Bước thực hiện
1. Đầu tiên làm X
2. Sau đó làm Y
3. Cuối cùng làm Z

$ARGUMENTS
```

Sau đó dùng:
```
/my-command tham số của tôi
```

## Sự Khác Biệt: Command vs Skill vs Agent

```
/tdd (Command)
  └─ Kích hoạt tdd-workflow (Skill) → Hướng dẫn quy trình
  └─ Gọi tdd-guide (Agent) → Thực thi từng bước
```

| | Command | Skill | Agent |
|--|---------|-------|-------|
| **Ai kích hoạt** | Người dùng gõ `/` | Claude đọc tự động | Claude gọi khi cần |
| **Vai trò** | Entry point, điều phối | Kiến thức, hướng dẫn | Thực thi chuyên biệt |
| **Ví dụ** | `/tdd` | `tdd-workflow` | `tdd-guide` |

## Tips Sử Dụng Hiệu Quả

1. **Bắt đầu bằng `/plan`** cho mọi tính năng mới → tránh xây sai hướng
2. **Dùng `/tdd`** thay vì gõ hướng dẫn TDD thủ công
3. **`/code-review` trước mỗi commit** → catch bugs sớm
4. **`/build-fix` khi bị stuck** → thay vì tự debug mãi
5. **`/checkpoint` định kỳ** → có thể rollback nếu sai hướng
