# 08 - Scripts

## Scripts là gì?

**Scripts** là các file Node.js tiện ích hỗ trợ hoạt động của hooks, setup, và các tác vụ tự động hóa. Chúng là "engine" chạy phía sau mọi tự động hóa trong ECC.

```
scripts/
├── hooks/           # Scripts cho hooks
│   ├── run-with-flags.js       ← Wrapper quan trọng nhất
│   ├── pre-bash-commit-quality.js
│   ├── pre-bash-git-push-reminder.js
│   ├── pre-bash-tmux-reminder.js
│   ├── auto-tmux-dev.js
│   └── ...
├── lib/             # Thư viện dùng chung
│   ├── utils.js              ← Utilities chung
│   ├── package-manager.js    ← Detect package manager
│   └── ...
├── install-plan.js  # Lên kế hoạch cài đặt
├── install-apply.js # Áp dụng cài đặt
├── doctor.js        # Kiểm tra health của ECC
├── status.js        # Xem trạng thái hiện tại
├── skill-create-output.js  # Tạo skill mới
└── ...
```

## Scripts Quan Trọng Nhất

### `scripts/hooks/run-with-flags.js` - Wrapper cho Hooks

Đây là file quan trọng nhất. Tất cả hooks đều chạy qua wrapper này.

**Cú pháp:**
```bash
node scripts/hooks/run-with-flags.js "hook-id" "path/to/hook.js" "profile"
```

**Tính năng:**
1. **Disable hook theo ID:** Đọc `ECC_DISABLED_HOOKS` environment variable
2. **Profile gating:** Đọc `ECC_HOOK_PROFILE` (strict/relaxed)
3. **Parse input:** Tự động parse JSON từ stdin
4. **Error handling:** Đảm bảo hook không crash toàn bộ Claude

**Ví dụ trong hooks.json:**
```json
{
  "command": "node \"${CLAUDE_PLUGIN_ROOT}/scripts/hooks/run-with-flags.js\" \"pre:bash:commit-quality\" \"scripts/hooks/pre-bash-commit-quality.js\" \"strict\""
}
```

**Cách hoạt động:**
```javascript
// run-with-flags.js logic đơn giản
const hookId = process.argv[2]
const scriptPath = process.argv[3]
const profile = process.argv[4]

// Kiểm tra hook có bị disable không
const disabledHooks = process.env.ECC_DISABLED_HOOKS?.split(',') || []
if (disabledHooks.includes(hookId)) {
  process.exit(0)  // Bỏ qua hook này
}

// Kiểm tra profile
const currentProfile = process.env.ECC_HOOK_PROFILE || 'relaxed'
if (profile === 'strict' && currentProfile !== 'strict') {
  process.exit(0)  // Chỉ chạy ở strict mode
}

// Chạy hook thực sự
const hook = require(scriptPath)
hook.run(rawInput)
```

---

### `scripts/lib/package-manager.js` - Detect Package Manager

Tự động phát hiện npm, pnpm, yarn, hoặc bun của project.

**Logic phát hiện (theo thứ tự ưu tiên):**
1. Biến môi trường `CLAUDE_PACKAGE_MANAGER`
2. File lock trong project (`pnpm-lock.yaml`, `yarn.lock`, `bun.lockb`)
3. Config trong `package.json`
4. Fallback về `npm`

**Dùng trong scripts khác:**
```javascript
const { detectPackageManager } = require('./lib/package-manager')

const pm = detectPackageManager()
// pm = 'npm' | 'pnpm' | 'yarn' | 'bun'

const runCmd = pm === 'npm' ? 'npm run' :
               pm === 'yarn' ? 'yarn' :
               pm  // pnpm và bun dùng trực tiếp

exec(`${runCmd} test`)
```

---

### `scripts/lib/utils.js` - Utilities Chung

Các utility functions được dùng ở nhiều nơi:

```javascript
// Ví dụ các functions trong utils.js

// Check xem lệnh có tồn tại không
function commandExists(cmd) {
  try {
    execSync(`which ${cmd}`, { stdio: 'ignore' })
    return true
  } catch {
    return false
  }
}

// Parse JSON an toàn (không throw)
function safeParseJSON(str) {
  try {
    return JSON.parse(str)
  } catch {
    return null
  }
}

// Log với prefix
function log(prefix, message) {
  console.error(`[${prefix}] ${message}`)
}
```

---

### `scripts/doctor.js` - Health Check

Kiểm tra ECC có được cài đúng không.

**Chạy:**
```bash
node scripts/doctor.js
```

**Kiểm tra:**
- Node.js version đủ chưa
- Dependencies đã cài chưa
- Hooks đã được register chưa
- MCP servers có accessible không
- File permissions đúng chưa

**Output ví dụ:**
```
[ECC Doctor] Checking installation...
✓ Node.js v20.10.0 (>= 18 required)
✓ Dependencies installed
✓ Hooks registered (12 hooks)
✗ MCP server 'firecrawl' - FIRECRAWL_API_KEY not set
⚠ tmux not installed (optional, needed for auto-tmux hooks)

Status: 1 error, 1 warning
```

---

### `scripts/install-plan.js` + `scripts/install-apply.js` - Cài Đặt

**`install-plan.js`:** Tạo kế hoạch cài đặt (dry-run, không thay đổi gì):
```bash
node scripts/install-plan.js
# Output: Danh sách những gì sẽ được cài
```

**`install-apply.js`:** Thực sự áp dụng cài đặt:
```bash
node scripts/install-apply.js
# Copy hooks, rules, skills vào ~/.claude/
```

---

### `scripts/status.js` - Xem Trạng Thái

```bash
node scripts/status.js
```

**Output:**
```
ECC Status
══════════
Hooks: 12 active, 2 disabled
Skills: 45 installed
Agents: 8 configured
MCP Servers: 3 configured, 2 connected
Profile: strict
```

---

### `scripts/skill-create-output.js` - Tạo Skill

Được gọi bởi `/skill-create` command. Phân tích git history hoặc session để tạo skill mới.

## Cách Viết Hook Script

Pattern chuẩn cho một hook script:

```javascript
// scripts/hooks/my-hook.js
'use strict'

// Export run function để run-with-flags.js có thể gọi
module.exports = { run }

async function run(rawInput) {
  // 1. Parse input
  let input
  try {
    input = JSON.parse(rawInput)
  } catch {
    // Lỗi parse - bỏ qua và cho qua
    process.exit(0)
  }

  // 2. Kiểm tra điều kiện - chỉ can thiệp khi cần
  const command = input?.tool_input?.command || ''
  if (!command.includes('git commit')) {
    process.exit(0)  // Không phải commit, bỏ qua
  }

  // 3. Logic chính
  try {
    const result = await doSomething(command)

    if (result.hasIssues) {
      // CHẶN lại: log lý do ra stderr
      console.error('[MyHook] Phát hiện vấn đề: ' + result.message)
      process.exit(1)
    }
  } catch (err) {
    // Lỗi trong hook - KHÔNG chặn, chỉ log
    console.error('[MyHook] Lỗi không mong đợi:', err.message)
    process.exit(0)  // Cho qua, không block Claude
  }

  // 4. Mọi thứ OK
  process.exit(0)
}

async function doSomething(command) {
  // Logic của bạn ở đây
}
```

**Quy tắc vàng:**
- `process.exit(1)` = CHẶN Claude lại (chỉ dùng khi có vấn đề thực sự)
- `process.exit(0)` = Cho qua (dùng trong mọi trường hợp khác)
- **Luôn** `exit(0)` khi gặp lỗi không mong đợi trong hook

## Cách Viết Lib Script

Scripts trong `scripts/lib/` là utilities được chia sẻ:

```javascript
// scripts/lib/git-utils.js
'use strict'

const { execSync } = require('child_process')

/**
 * Lấy commit message từ lệnh git commit
 * @param {string} command - Lệnh git commit đầy đủ
 * @returns {string|null} - Commit message hoặc null nếu không tìm thấy
 */
function extractCommitMessage(command) {
  const match = command.match(/-m\s+["']([^"']+)["']/)
  return match ? match[1] : null
}

/**
 * Kiểm tra có phải Conventional Commits format không
 * @param {string} message - Commit message
 * @returns {boolean}
 */
function isConventionalCommit(message) {
  const pattern = /^(feat|fix|docs|style|refactor|test|chore)(\(.+\))?: .+/
  return pattern.test(message)
}

/**
 * Lấy danh sách files staged
 * @returns {string[]}
 */
function getStagedFiles() {
  try {
    const output = execSync('git diff --cached --name-only', { encoding: 'utf-8' })
    return output.trim().split('\n').filter(Boolean)
  } catch {
    return []
  }
}

module.exports = {
  extractCommitMessage,
  isConventionalCommit,
  getStagedFiles,
}
```

## Chạy Tests

```bash
# Chạy tất cả tests
node tests/run-all.js

# Test scripts lib
node tests/lib/utils.test.js
node tests/lib/package-manager.test.js

# Test hooks
node tests/hooks/hooks.test.js
```

**Quy tắc testing:**
- Mỗi file trong `scripts/lib/` phải có file test tương ứng trong `tests/lib/`
- Mỗi hook mới phải có ít nhất một integration test trong `tests/hooks/`

## Ví Dụ: Tạo Utility Script Mới

```bash
# Tạo utility
touch scripts/lib/my-util.js

# Tạo test
touch tests/lib/my-util.test.js
```

`scripts/lib/my-util.js`:
```javascript
'use strict'

function formatBytes(bytes) {
  const units = ['B', 'KB', 'MB', 'GB']
  let i = 0
  while (bytes >= 1024 && i < units.length - 1) {
    bytes /= 1024
    i++
  }
  return `${bytes.toFixed(1)} ${units[i]}`
}

module.exports = { formatBytes }
```

`tests/lib/my-util.test.js`:
```javascript
'use strict'

const assert = require('assert')
const { formatBytes } = require('../../scripts/lib/my-util')

// Test cases
assert.strictEqual(formatBytes(500), '500.0 B')
assert.strictEqual(formatBytes(1024), '1.0 KB')
assert.strictEqual(formatBytes(1048576), '1.0 MB')

console.log('my-util.test.js: All tests passed!')
```

## Lỗi Thường Gặp

**Q: Hook script không chạy được?**

A: Kiểm tra file permissions:
```bash
chmod +x scripts/hooks/my-hook.js
```

**Q: Script báo lỗi `require` module không tìm thấy?**

A: Chạy `npm install` để cài dependencies.

**Q: Muốn debug hook script?**

A: Thêm `console.error('[Debug]', JSON.stringify(input))` vào đầu function `run()`, stderr output sẽ hiện trong Claude Code logs.
