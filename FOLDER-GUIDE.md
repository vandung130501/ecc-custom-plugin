# Hướng dẫn cấu trúc thư mục — Everything Claude Code

> Tài liệu này giải thích từng thư mục trong project theo ngôn ngữ đơn giản, dễ tiếp cận.

---

## Tổng quan nhanh

```
everything-claude-code/
├── agents/        ← Trợ lý con chuyên biệt (Claude thuê thêm "nhân viên")
├── commands/      ← Lệnh tắt /slash bạn gõ trong Claude Code
├── skills/        ← Cẩm nang kỹ năng cho từng lĩnh vực
├── hooks/         ← Tự động hóa chạy trước/sau mỗi hành động
├── rules/         ← Luật bất di bất dịch Claude phải tuân theo
├── mcp-configs/   ← Kết nối Claude với công cụ ngoài (Jira, GitHub…)
├── scripts/       ← Công cụ Node.js hỗ trợ vận hành
├── contexts/      ← Bối cảnh dự án để Claude hiểu đúng ngữ cảnh
├── examples/      ← File mẫu CLAUDE.md cho từng loại dự án
├── docs/          ← Tài liệu nội bộ & hướng dẫn kiến trúc
├── tests/         ← Bộ kiểm thử tự động
├── manifests/     ← Danh sách cấu hình cài đặt
├── schemas/       ← Định nghĩa cấu trúc dữ liệu (JSON Schema)
├── plugins/       ← Hệ thống plugin mở rộng
├── research/      ← Ghi chép nghiên cứu nội bộ
├── assets/        ← Hình ảnh, tài nguyên tĩnh
└── ecc2/          ← Phiên bản 2.0 viết bằng Rust (thử nghiệm)
```

---

## Chi tiết từng thư mục

---

### `agents/` — Trợ lý con chuyên biệt
**30 agents**

**Hình dung:** Claude chính là quản lý. Khi gặp việc phức tạp, nó "thuê" thêm nhân viên chuyên môn từ thư mục này.

Mỗi file `.md` là một agent với vai trò riêng:

| Agent | Làm gì |
|---|---|
| `planner.md` | Lập kế hoạch chi tiết trước khi code |
| `code-reviewer.md` | Review code, tìm lỗi, gợi ý cải tiến |
| `tdd-guide.md` | Hướng dẫn phát triển theo TDD |
| `build-error-resolver.md` | Chuyên sửa lỗi build |
| `architect.md` | Thiết kế kiến trúc hệ thống |
| `database-reviewer.md` | Kiểm tra schema và query database |

**Cấu trúc một file agent:**
```yaml
---
name: planner
description: Chuyên gia lập kế hoạch...
tools: ["Read", "Grep", "Glob"]
model: opus
---
Nội dung hướng dẫn hành vi của agent...
```

---

### `commands/` — Lệnh slash `/`
**63 commands**

**Hình dung:** Đây là bàn phím tắt. Thay vì viết cả đoạn dài, bạn chỉ cần gõ `/plan` và Claude biết phải làm gì.

Mỗi file `.md` = một lệnh `/tên-file`:

| Lệnh | Tác dụng |
|---|---|
| `/plan` | Phân tích yêu cầu, lập kế hoạch, hỏi xác nhận trước khi code |
| `/tdd` | Viết test trước, code sau (Test-Driven Development) |
| `/code-review` | Review code local hoặc GitHub PR |
| `/build-fix` | Tự động tìm và sửa lỗi build |
| `/e2e` | Tạo và chạy E2E tests |
| `/feature-dev` | Workflow phát triển tính năng có cấu trúc |
| `/go-build` | Build dự án Go |
| `/flutter-test` | Chạy test Flutter |
| `/prp-plan` | Lập kế hoạch chi tiết kèm PRD |

**Cấu trúc một file command:**
```yaml
---
description: Mô tả ngắn gọn hiển thị khi gợi ý lệnh
---
# Nội dung hướng dẫn Claude thực hiện...
```

---

### `skills/` — Cẩm nang kỹ năng
**112 skills**

**Hình dung:** Thư viện sách tham khảo. Khi làm việc trong lĩnh vực nào, Claude mở đúng cuốn sách để làm đúng cách.

Skills được tổ chức theo chủ đề, mỗi thư mục con là một lĩnh vực:

| Thư mục skill | Nội dung |
|---|---|
| `backend-patterns/` | Các pattern phổ biến cho backend |
| `api-design/` | Thiết kế API chuẩn REST/GraphQL |
| `browser-qa/` | Kiểm thử trình duyệt |
| `agentic-engineering/` | Xây dựng hệ thống agent |
| `architecture-decision-records/` | Ghi lại quyết định kiến trúc |
| `article-writing/` | Viết tài liệu kỹ thuật |

Skills khác với commands: **commands** = "làm việc X", **skills** = "kiến thức để làm X đúng cách".

---

### `hooks/` — Tự động hóa theo sự kiện

**Hình dung:** Giống như trigger trong database — khi sự kiện A xảy ra, tự động làm việc B.

Có 2 file chính:
- `hooks.json` — Định nghĩa toàn bộ hooks
- `README.md` — Hướng dẫn sử dụng

**Các loại hook:**

| Loại | Khi nào chạy | Ví dụ dùng |
|---|---|---|
| `PreToolUse` | Trước khi dùng tool | Chặn `--no-verify` trong git commit |
| `PostToolUse` | Sau khi dùng tool | Auto-format code sau khi Edit |
| `Notification` | Khi Claude gửi thông báo | Gửi alert qua Slack |
| `Stop` | Khi Claude kết thúc | Lưu session, tổng kết |

**Ví dụ hook thực tế:**
```json
{
  "matcher": "Bash",
  "hooks": [{ "command": "npx block-no-verify@1.1.2" }],
  "description": "Chặn bỏ qua git hooks"
}
```

---

### `rules/` — Luật bắt buộc tuân theo

**Hình dung:** Nội quy công ty. Claude LUÔN phải theo, không cần ai nhắc.

Được tổ chức theo ngôn ngữ/stack:

```
rules/
├── common/          ← Luật chung cho mọi dự án
│   ├── security.md      (không hardcode secrets, tránh injection...)
│   ├── coding-style.md  (đặt tên biến, cấu trúc code...)
│   ├── testing.md       (yêu cầu test coverage...)
│   ├── git-workflow.md  (conventional commits, PR flow...)
│   └── performance.md   (tránh N+1 queries, tối ưu...)
├── typescript/      ← Luật riêng cho TypeScript
├── golang/          ← Luật riêng cho Go
├── php/             ← Luật riêng cho PHP
└── web/             ← Luật riêng cho Web frontend
```

---

### `mcp-configs/` — Kết nối công cụ ngoài

**Hình dung:** Cổng cắm USB — giúp Claude "cắm" thêm công cụ bên ngoài vào.

File duy nhất: `mcp-servers.json`

MCP (Model Context Protocol) cho phép Claude tích hợp với:
- GitHub / GitLab
- Jira / Linear
- Slack / Discord
- Database (PostgreSQL, SQLite...)
- Filesystem, Browser...

Bạn chọn server nào cần, khai báo vào file này, Claude sẽ có thêm khả năng tương tác với hệ thống đó.

---

### `scripts/` — Công cụ vận hành

**Hình dung:** Hộp đồ nghề — các script Node.js phục vụ cài đặt, quản lý, và vận hành ECC.

Các script quan trọng:

| Script | Chức năng |
|---|---|
| `ecc.js` | CLI chính để quản lý ECC |
| `install-plan.js` | Lên kế hoạch cài đặt |
| `install-apply.js` | Thực hiện cài đặt |
| `uninstall.js` | Gỡ cài đặt |
| `doctor.js` | Chẩn đoán lỗi cấu hình |
| `skills-health.js` | Kiểm tra tình trạng skills |

Thư mục con:
- `scripts/lib/` — Thư viện dùng chung (package manager detection, utils...)
- `scripts/hooks/` — Script chạy bởi hooks
- `scripts/codemaps/` — Tạo bản đồ codebase

---

### `contexts/` — Bối cảnh dự án

**Hình dung:** Briefing trước cuộc họp — cung cấp ngữ cảnh để Claude hiểu đúng tình huống.

3 file:
- `dev.md` — Bối cảnh khi đang phát triển
- `review.md` — Bối cảnh khi đang review
- `research.md` — Bối cảnh khi đang nghiên cứu

---

### `examples/` — File mẫu CLAUDE.md

**Hình dung:** Template sẵn sàng dùng — copy về và chỉnh sửa cho phù hợp dự án của bạn.

| File mẫu | Dành cho |
|---|---|
| `saas-nextjs-CLAUDE.md` | Dự án Next.js / SaaS |
| `go-microservice-CLAUDE.md` | Microservice bằng Go |
| `django-api-CLAUDE.md` | API bằng Django |
| `rust-api-CLAUDE.md` | API bằng Rust |
| `laravel-api-CLAUDE.md` | API bằng Laravel |
| `user-CLAUDE.md` | Cài đặt cá nhân cho user |

---

### `docs/` — Tài liệu nội bộ

**Hình dung:** Kho lưu trữ quyết định thiết kế và hướng dẫn nâng cao.

Các tài liệu đáng chú ý:
- `SKILL-PLACEMENT-POLICY.md` — Skill nên để ở đâu
- `SKILL-DEVELOPMENT-GUIDE.md` — Cách viết skill mới
- `ECC-2.0-REFERENCE-ARCHITECTURE.md` — Kiến trúc ECC 2.0
- `TROUBLESHOOTING.md` — Xử lý sự cố thường gặp
- Thư mục dịch: `zh-CN/`, `ko-KR/`, `ja-JP/`, `pt-BR/`...

---

### `tests/` — Bộ kiểm thử tự động

**Hình dung:** Hệ thống QA — đảm bảo các scripts và hooks hoạt động đúng.

```
tests/
├── run-all.js        ← Chạy tất cả tests: node tests/run-all.js
├── lib/              ← Tests cho scripts/lib/
├── hooks/            ← Tests cho hooks
├── integration/      ← Tests tích hợp
└── scripts/          ← Tests cho scripts
```

---

### `manifests/` — Cấu hình cài đặt

**Hình dung:** Danh sách hàng hóa — định nghĩa "gói" nào gồm những gì.

3 file JSON:
- `install-profiles.json` — Các profile cài đặt (minimal, standard, full)
- `install-modules.json` — Các module có thể cài
- `install-components.json` — Các component riêng lẻ

---

### `schemas/` — Định nghĩa cấu trúc dữ liệu

**Hình dung:** Bản vẽ kỹ thuật — quy định chính xác các file config phải có cấu trúc thế nào.

Dùng để validate các file JSON trong project:
- `hooks.schema.json` — Cấu trúc hợp lệ của hooks.json
- `install-profiles.schema.json` — Cấu trúc hợp lệ của manifest
- `plugin.schema.json` — Cấu trúc hợp lệ của một plugin

---

### `plugins/` — Hệ thống plugin

**Hình dung:** App Store mini — cho phép mở rộng ECC bằng các plugin bên thứ ba.

Hiện chỉ có `README.md` mô tả plugin system đang được phát triển.

---

### `assets/` — Tài nguyên tĩnh

Chứa thư mục `images/` — logo, ảnh minh họa dùng trong tài liệu.

---

### `ecc2/` — ECC phiên bản 2.0 (Rust)

**Hình dung:** Phòng R&D — đang thử nghiệm viết lại CLI bằng Rust để nhanh hơn.

Dự án Rust riêng biệt với `Cargo.toml`, chưa phải bản production.

---

## Luồng hoạt động tổng thể

```
Bạn gõ /plan trong Claude Code
        │
        ▼
commands/plan.md     ← Đọc hướng dẫn thực hiện
        │
        ▼
agents/planner.md    ← Khởi động agent chuyên lập kế hoạch
        │
        ├── rules/   ← Luôn tuân theo (security, style...)
        ├── skills/  ← Tham khảo kiến thức chuyên môn
        └── hooks/   ← Tự động chạy trước/sau (logging, format...)
```

---

> Tài liệu này được tạo thủ công. Nếu cấu trúc thư mục thay đổi, cập nhật lại file này.
