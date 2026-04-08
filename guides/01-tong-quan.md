# 01 - Tổng Quan Dự Án

## Everything Claude Code là gì?

**Everything Claude Code (ECC)** là một **plugin/bộ công cụ** dành cho Claude Code - IDE AI của Anthropic. Nó cung cấp tập hợp các workflow, quy tắc, và tự động hóa được thiết kế sẵn để giúp developer làm việc hiệu quả hơn với AI.

Hiểu đơn giản: Thay vì phải gõ lại hướng dẫn cho Claude mỗi lần làm việc, ECC đã đóng gói sẵn những best practice, quy trình phát triển phần mềm, và tự động hóa vào các file cấu hình.

## Cấu Trúc Folder

```
everything-claude-code/
│
├── agents/          # Các AI agent chuyên biệt (kiến trúc sư, reviewer...)
├── skills/          # Bộ kỹ năng/workflow theo domain (TDD, API design...)
├── commands/        # Slash commands người dùng gõ (/tdd, /plan, /e2e...)
├── hooks/           # Tự động hóa kích hoạt theo sự kiện (trước/sau khi dùng tool)
├── rules/           # Quy tắc cứng Claude phải tuân theo (security, style...)
├── mcp-configs/     # Cấu hình MCP servers - thêm tools bên ngoài cho Claude
├── scripts/         # Script Node.js tiện ích cho hooks và setup
├── tests/           # Test suite cho scripts
├── contexts/        # Các ngữ cảnh làm việc (dev, research, review mode)
├── manifests/       # Danh sách cài đặt ECC
└── guides/          # (Thư mục này) Hướng dẫn tiếng Việt
```

## Luồng Hoạt Động Cơ Bản

```
Bạn gõ lệnh hoặc chat
        |
        v
   Hooks kích hoạt (trước)
        |
        v
   Claude đọc Rules (luôn luôn áp dụng)
        |
        v
   Claude dùng Skills/Agents nếu phù hợp
        |
        v
   Claude thực thi Commands
        |
        v
   Hooks kích hoạt (sau)
        |
        v
   Kết quả trả về cho bạn
```

## Cài Đặt

### Yêu Cầu

- Node.js >= 18
- Claude Code (claude.ai/code) đã cài
- npm/pnpm/yarn/bun

### Cài Đặt Cơ Bản

```bash
# Clone repo về máy
git clone https://github.com/anthropics/everything-claude-code
cd everything-claude-code

# Cài dependencies
npm install

# Chạy setup
node scripts/install-apply.js
```

### Kiểm Tra Cài Đặt

```bash
# Chạy toàn bộ tests
node tests/run-all.js

# Chạy test riêng lẻ
node tests/lib/utils.test.js
node tests/hooks/hooks.test.js
```

## Biến Môi Trường Quan Trọng

| Biến | Mô tả | Ví dụ |
|------|-------|-------|
| `CLAUDE_PACKAGE_MANAGER` | Package manager ưu tiên | `npm`, `pnpm`, `yarn`, `bun` |
| `CLAUDE_PLUGIN_ROOT` | Đường dẫn gốc plugin | Tự động set khi cài |
| `ECC_HOOK_PROFILE` | Profile hooks (strict/relaxed) | `strict` |
| `ECC_DISABLED_HOOKS` | Tắt hook cụ thể | `pre:bash:tmux-reminder` |

## Ví Dụ Thực Tế

Không có ECC:
```
Bạn: "Hãy viết code theo TDD cho hàm tính thuế"
Claude: (tự quyết định, có thể quên viết test trước, quên các bước...)
```

Có ECC:
```
Bạn: "/tdd"
Claude: (tự động theo đúng quy trình RED→GREEN→REFACTOR, viết test trước,
         kiểm tra coverage, commit đúng format...)
```

## Các Khái Niệm Cần Nhớ

| Khái niệm | Vai trò | File cấu hình |
|-----------|---------|---------------|
| **Skills** | Kiến thức domain, hướng dẫn cách làm | `skills/*/SKILL.md` |
| **Agents** | AI chuyên biệt làm task cụ thể | `agents/*.md` |
| **Commands** | Lệnh tắt người dùng gọi | `commands/*.md` |
| **Hooks** | Tự động hóa theo sự kiện | `hooks/hooks.json` |
| **Rules** | Quy tắc bắt buộc Claude tuân thủ | `rules/**/*.md` |
| **MCP** | Tools bên ngoài (browser, memory...) | `mcp-configs/*.json` |

## Đọc Tiếp

- Xem [02-skills.md](./02-skills.md) để hiểu cách Skills hoạt động
- Xem [09-workflow-thuc-te.md](./09-workflow-thuc-te.md) để xem ví dụ workflow hoàn chỉnh
