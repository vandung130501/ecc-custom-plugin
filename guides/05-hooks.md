# 05 - Hooks

## Hooks là gì?

**Hooks** là các đoạn code tự động chạy khi có sự kiện xảy ra trong Claude Code — giống như middleware. Bạn không cần làm gì, chúng tự kích hoạt.

Ví dụ thực tế:
- Mỗi khi Claude sắp chạy `git commit` → hook kiểm tra commit message đúng format chưa
- Mỗi khi Claude chỉnh sửa file → hook tự format code
- Mỗi khi Claude sắp `git push` → hook nhắc nhở review lại

## Loại Hooks

| Loại | Khi nào chạy | Ví dụ dùng |
|------|-------------|-----------|
| `PreToolUse` | **Trước** khi Claude dùng tool | Chặn lệnh nguy hiểm, validation |
| `PostToolUse` | **Sau** khi Claude dùng tool | Format code sau khi sửa, logging |
| `Notification` | Khi có thông báo từ Claude | Gửi alert, log session |
| `Stop` | Khi Claude dừng | Lưu session, cleanup |

## Cấu Trúc File hooks.json

File `hooks/hooks.json` chứa tất cả hooks:

```json
{
  "$schema": "...",
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "node scripts/hooks/my-hook.js"
          }
        ],
        "description": "Mô tả hook này làm gì",
        "id": "pre:bash:my-hook"
      }
    ],
    "PostToolUse": [...],
    "Notification": [...],
    "Stop": [...]
  }
}
```

**Giải thích:**
- `matcher`: Tool nào sẽ trigger hook này (`Bash`, `Edit`, `Write`, `Read`, `*` = tất cả)
- `type`: Loại hook (`command` = chạy shell command)
- `command`: Lệnh thực tế được chạy
- `id`: ID duy nhất để có thể tắt hook cụ thể

## Danh Sách Hooks Có Sẵn

### PreToolUse - Hooks Chạy Trước

#### `pre:bash:block-no-verify`
**Mục đích:** Chặn Claude dùng `git commit --no-verify` (vì sẽ bỏ qua pre-commit hooks)

**Tại sao quan trọng:** `--no-verify` bỏ qua lint, test, commit message check → có thể commit code xấu.

```
Bạn gõ: "Commit code đi"
Hook chặn: "Không được dùng --no-verify!"
Claude phải: Sửa lỗi lint/test thật sự thay vì bypass
```

---

#### `pre:bash:commit-quality`
**Mục đích:** Kiểm tra trước khi commit:
- Lint staged files
- Validate commit message format (Conventional Commits)
- Detect `console.log`, `debugger`, secrets bị quên

**Ví dụ chặn:**
```bash
# Claude định commit với message này:
git commit -m "fix stuff"

# Hook chặn và yêu cầu:
"Message phải theo format: fix(auth): fix login bug"
```

---

#### `pre:bash:git-push-reminder`
**Mục đích:** Nhắc nhở review lại trước khi push code

**Output:**
```
[GitPushReminder] Bạn sắp push code lên remote.
Hãy kiểm tra:
- [ ] Đã review diff chưa?
- [ ] Tests đang pass không?
- [ ] Có file nhạy cảm nào không?
```

---

#### `pre:bash:tmux-reminder`
**Mục đích:** Nhắc dùng tmux cho lệnh chạy lâu (server, build...)

**Ví dụ:**
```
Claude định chạy: npm run dev
Hook nhắc: "Lệnh này chạy lâu, hãy dùng tmux để giữ session"
```

---

#### `pre:bash:auto-tmux-dev`
**Mục đích:** Tự động start dev server trong tmux session riêng

---

### PostToolUse - Hooks Chạy Sau

#### `post:edit:format`
**Mục đích:** Tự động format code sau khi Claude chỉnh sửa file

**Hoạt động:**
- Sau khi Edit tool chạy
- Tự động chạy Prettier/ESLint/gofmt tùy ngôn ngữ

---

#### `post:edit:session-summary`
**Mục đích:** Lưu tóm tắt session sau mỗi thay đổi

---

## Cách Hook Scripts Hoạt Động

Hầu hết hooks được viết dưới dạng Node.js scripts trong `scripts/hooks/`:

```javascript
// scripts/hooks/pre-bash-commit-quality.js
const { run } = require('./run-with-flags')

async function run(rawInput) {
  const input = JSON.parse(rawInput)

  // Kiểm tra nếu là lệnh git commit
  if (!input.tool_input?.command?.includes('git commit')) {
    return  // Không phải commit, bỏ qua
  }

  // Kiểm tra commit message
  const message = extractCommitMessage(input.tool_input.command)
  if (!isConventionalCommit(message)) {
    console.error('Commit message phải theo Conventional Commits format!')
    process.exit(1)  // Chặn lại
  }
}
```

## Wrapper `run-with-flags.js`

Tất cả hooks dùng wrapper này để:

```javascript
node scripts/hooks/run-with-flags.js "hook-id" "scripts/hooks/my-hook.js" "strict"
```

**Tính năng của wrapper:**
1. **Runtime gating** qua biến môi trường `ECC_DISABLED_HOOKS`
2. **Profile switching** qua `ECC_HOOK_PROFILE`
3. **Parse JSON input** tự động
4. **Exit 0 gracefully** khi có lỗi không nghiêm trọng

## Tắt Hook Tạm Thời

```bash
# Tắt một hook cụ thể
export ECC_DISABLED_HOOKS="pre:bash:tmux-reminder"

# Tắt nhiều hooks
export ECC_DISABLED_HOOKS="pre:bash:tmux-reminder,pre:bash:git-push-reminder"

# Bật lại
unset ECC_DISABLED_HOOKS
```

## Đổi Profile Hooks

```bash
# Chế độ strict (tất cả hooks)
export ECC_HOOK_PROFILE="strict"

# Chế độ relaxed (chỉ hooks quan trọng)
export ECC_HOOK_PROFILE="relaxed"
```

## Tạo Hook Mới

### Bước 1: Viết script

```javascript
// scripts/hooks/my-custom-hook.js
module.exports = { run }

async function run(rawInput) {
  const input = JSON.parse(rawInput)

  // Logic của bạn ở đây
  console.error('[MyHook] Đang kiểm tra...')

  // Nếu muốn CHẶN: process.exit(1) + log lý do
  // Nếu muốn CHO QUA: return hoặc process.exit(0)
}
```

### Bước 2: Đăng ký vào hooks.json

```json
{
  "PreToolUse": [
    {
      "matcher": "Bash",
      "hooks": [
        {
          "type": "command",
          "command": "node \"${CLAUDE_PLUGIN_ROOT}/scripts/hooks/run-with-flags.js\" \"pre:bash:my-hook\" \"scripts/hooks/my-custom-hook.js\" \"strict\""
        }
      ],
      "description": "My custom hook mô tả",
      "id": "pre:bash:my-hook"
    }
  ]
}
```

### Bước 3: Test hook

```bash
node tests/hooks/hooks.test.js
```

## Quy Tắc Viết Hook (Quan Trọng!)

1. **Luôn `exit 0` khi lỗi không nghiêm trọng** → Không được block Claude vì bug trong hook
2. **PreToolUse hooks phải nhanh** (<200ms) → Không được gọi API, không I/O nặng
3. **Log ra stderr** với prefix `[HookName]`:
   ```javascript
   console.error('[MyHook] Thông báo này ra stderr')
   ```
4. **Async hooks** cần đánh dấu `"async": true` và timeout ≤30s
5. **Đừng dùng network calls** trong blocking hooks

## Ví Dụ Thực Tế: Hook Ngăn Leak API Key

```javascript
// scripts/hooks/pre-edit-secret-scan.js
module.exports = { run }

const SECRET_PATTERNS = [
  /sk-[a-zA-Z0-9]{48}/,        // OpenAI key
  /AKIA[0-9A-Z]{16}/,          // AWS key
  /ghp_[a-zA-Z0-9]{36}/,      // GitHub token
]

async function run(rawInput) {
  const input = JSON.parse(rawInput)
  const content = input.tool_input?.new_string || ''

  for (const pattern of SECRET_PATTERNS) {
    if (pattern.test(content)) {
      console.error('[SecretScan] CẢNH BÁO: Phát hiện API key trong code!')
      console.error('[SecretScan] Hãy dùng biến môi trường thay vì hardcode.')
      process.exit(1)  // Chặn lại
    }
  }
}
```

## Lỗi Thường Gặp

**Q: Hook của tôi chặn mọi lệnh, kể cả lệnh bình thường?**

A: Kiểm tra logic điều kiện. Dùng `return` (không phải `process.exit(1)`) cho trường hợp không cần can thiệp.

**Q: Hook chạy quá chậm?**

A: PreToolUse hooks phải <200ms. Nếu cần xử lý nặng, đánh dấu `"async": true`.

**Q: Muốn tắt hook vĩnh viễn?**

A: Comment out hoặc xóa hook entry trong `hooks.json`.
