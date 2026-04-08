# 02 - Skills

## Skills là gì?

**Skills** là các file Markdown chứa kiến thức chuyên sâu về một domain cụ thể. Khi bạn kích hoạt một skill, Claude sẽ đọc nội dung đó và áp dụng hướng dẫn vào công việc hiện tại.

Nghĩ đơn giản: Skill giống như **tờ gian lận (cheat sheet)** hoặc **hướng dẫn nội bộ** mà bạn đưa cho Claude trước khi bắt đầu làm việc.

## Cấu Trúc Một Skill

```
skills/
└── tdd-workflow/
    └── SKILL.md    ← File chính
```

Mỗi `SKILL.md` thường có cấu trúc:

```markdown
---
name: tdd-workflow
description: Mô tả ngắn về skill này
origin: ECC
---

# Tên Skill

## When to Activate (Khi nào dùng)
- Trường hợp 1
- Trường hợp 2

## How It Works (Cách hoạt động)
Bước 1: ...
Bước 2: ...

## Examples (Ví dụ thực tế)
...
```

## Cách Kích Hoạt Skill

### Cách 1: Qua Command (phổ biến nhất)

```
/tdd          ← Kích hoạt skill tdd-workflow
/code-review  ← Kích hoạt skill code-reviewer
/plan         ← Kích hoạt skill planner
```

### Cách 2: Nhắc trong chat

```
"Hãy dùng skill coding-standards để review code này"
"Apply tdd-workflow để viết test cho hàm calculateTax"
```

### Cách 3: Claude tự nhận diện

Claude tự động chọn skill phù hợp dựa trên ngữ cảnh câu hỏi của bạn.

## Danh Sách Skills Quan Trọng

### Nhóm Coding & Architecture

| Skill | Khi nào dùng | Ví dụ |
|-------|-------------|-------|
| `coding-standards` | Review code, onboarding, refactor | "Review code này theo chuẩn" |
| `frontend-patterns` | Viết React, hooks, state management | "Tạo component đúng chuẩn" |
| `backend-patterns` | API, service layer, repository | "Thiết kế API endpoint" |
| `api-design` | Thiết kế REST API, response format | "Design API cho user module" |
| `database-migrations` | Thay đổi schema database | "Thêm cột vào bảng users" |

### Nhóm Testing

| Skill | Khi nào dùng |
|-------|-------------|
| `tdd-workflow` | Phát triển theo TDD (test trước) |
| `e2e-testing` | Viết Playwright/Cypress tests |
| `python-testing` | Test với pytest |
| `golang-testing` | Test với Go testing |

### Nhóm DevOps & Infrastructure

| Skill | Khi nào dùng |
|-------|-------------|
| `docker-patterns` | Viết Dockerfile, docker-compose |
| `deployment-patterns` | CI/CD, release strategy |
| `postgres-patterns` | Query optimization, indexing |

### Nhóm Security

| Skill | Khi nào dùng |
|-------|-------------|
| `security-review` | Audit code bảo mật |

### Nhóm AI/Agent

| Skill | Khi nào dùng |
|-------|-------------|
| `claude-api` | Gọi Claude API, tool use, prompt engineering |
| `agentic-engineering` | Xây dựng multi-agent systems |
| `eval-harness` | Đánh giá chất lượng AI output |

## Ví Dụ Thực Tế: Skill `coding-standards`

**Kịch bản:** Junior developer viết code và muốn Claude review theo chuẩn.

```
Bạn: "Review đoạn code này theo coding-standards skill"

function getData(x) {
  var result = []
  for(var i=0; i<x.length; i++) {
    result.push(x[i].name)
  }
  return result
}
```

**Claude (với skill `coding-standards`) sẽ chỉ ra:**

```
FAIL: Tên hàm không mô tả rõ (getData → extractNames)
FAIL: Dùng var thay vì const/let
FAIL: Loop truyền thống thay vì map()
FAIL: Tham số x không có type annotation

Gợi ý:
const extractNames = (items: Item[]): string[] => items.map(item => item.name)
```

## Ví Dụ Thực Tế: Skill `tdd-workflow`

```
Bạn: "/tdd - Viết hàm tính giá sau thuế VAT 10%"

Claude sẽ:
1. Viết test TRƯỚC (RED phase)
   test('tính đúng giá sau VAT', () => {
     expect(calculateVAT(100)).toBe(110)
   })

2. Chạy test - xác nhận FAIL

3. Viết code tối thiểu (GREEN phase)
   function calculateVAT(price: number): number {
     return price * 1.1
   }

4. Chạy test - xác nhận PASS

5. Refactor nếu cần (REFACTOR phase)
```

## Cách Tạo Skill Mới

Dùng command `/skill-create` hoặc tạo thủ công:

```bash
mkdir skills/my-new-skill
touch skills/my-new-skill/SKILL.md
```

Nội dung `SKILL.md`:

```markdown
---
name: my-new-skill
description: Mô tả skill này làm gì
origin: custom
---

# Tên Skill Của Tôi

## When to Activate
- Tình huống 1 dùng skill này
- Tình huống 2 dùng skill này

## How It Works
Bước 1: ...
Bước 2: ...

## Examples
Ví dụ cụ thể...
```

## Nơi Lưu Skills

| Loại | Vị trí | Mục đích |
|------|--------|---------|
| Skills chính thức ECC | `skills/*/SKILL.md` | Built-in, ai cũng dùng được |
| Skills cá nhân | `~/.claude/skills/` | Chỉ bạn dùng, không commit |
| Skills dự án | `.claude/skills/` | Cả team dùng, commit vào repo |

> **Tip:** Đọc `docs/SKILL-PLACEMENT-POLICY.md` để biết nên đặt skill ở đâu.

## Lỗi Thường Gặp

**Q: Tôi dùng `/tdd` nhưng Claude không theo đúng quy trình?**

A: Kiểm tra xem skill `tdd-workflow` có được cài chưa:
```bash
ls skills/tdd-workflow/SKILL.md
```

**Q: Skill của tôi không được Claude áp dụng?**

A: Đảm bảo file có frontmatter đúng format (phần `---` ở đầu file) và được đặt đúng thư mục.
