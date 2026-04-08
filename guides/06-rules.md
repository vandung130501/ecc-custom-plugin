# 06 - Rules

## Rules là gì?

**Rules** là các quy tắc bắt buộc mà Claude **phải luôn tuân theo**, bất kể bạn có nhắc hay không. Khác với Skills (chỉ áp dụng khi được kích hoạt), Rules được áp dụng trong **mọi cuộc trò chuyện**.

Hiểu đơn giản:
- **Skills** = "Khi cần làm X, hãy làm theo cách này"
- **Rules** = "Dù làm gì, KHÔNG BAO GIỜ được làm Y"

## Cấu Trúc Thư Mục Rules

```
rules/
├── common/              # Quy tắc chung cho mọi ngôn ngữ
│   ├── security.md      # Bảo mật
│   ├── coding-style.md  # Style code
│   ├── testing.md       # Testing requirements
│   ├── git-workflow.md  # Git workflow
│   ├── agents.md        # Cách dùng agents
│   ├── hooks.md         # Hook conventions
│   ├── patterns.md      # Design patterns
│   └── performance.md   # Performance guidelines
├── typescript/          # Rules riêng cho TypeScript
│   ├── coding-style.md
│   ├── security.md
│   └── ...
├── golang/              # Rules riêng cho Go
├── php/                 # Rules riêng cho PHP
└── web/                 # Rules riêng cho Web
```

## Rules Quan Trọng Nhất

### 1. Security Rules (`rules/common/security.md`)

**Checklist bắt buộc TRƯỚC MỌI COMMIT:**

```
[ ] Không có hardcoded secrets (API keys, passwords, tokens)
[ ] Tất cả user inputs đều được validate
[ ] Dùng parameterized queries (tránh SQL injection)
[ ] HTML output được sanitize (tránh XSS)
[ ] CSRF protection đã bật
[ ] Authentication/authorization đúng chỗ
[ ] Rate limiting trên tất cả endpoints
[ ] Error messages không leak thông tin nhạy cảm
```

**Quy tắc cứng:**

```javascript
// FAIL: BAD - Không bao giờ hardcode secrets
const apiKey = "sk-abc123xyz789..."

// PASS: GOOD - Luôn dùng environment variables
const apiKey = process.env.OPENAI_API_KEY

// FAIL: BAD - SQL Injection vulnerability
const query = `SELECT * FROM users WHERE id = ${userId}`

// PASS: GOOD - Parameterized query
const query = 'SELECT * FROM users WHERE id = $1'
db.query(query, [userId])
```

**Khi phát hiện vấn đề bảo mật:**
1. DỪNG lại ngay lập tức
2. Gọi `security-reviewer` agent
3. Sửa issue TRƯỚC khi tiếp tục
4. Rotate bất kỳ secret nào đã bị lộ

---

### 2. Coding Style (`rules/common/coding-style.md`)

**Nguyên tắc cốt lõi:**
- Code được đọc nhiều hơn được viết → Ưu tiên rõ ràng
- Tên biến/hàm phải mô tả rõ mục đích
- Hàm ngắn (<50 dòng), làm một việc
- Không magic numbers, dùng constants có tên

**Immutability (QUAN TRỌNG):**

```typescript
// PASS: GOOD - Spread operator
const updated = { ...user, name: 'New Name' }
const newArr = [...items, newItem]

// FAIL: BAD - Mutation trực tiếp
user.name = 'New Name'  // KHÔNG ĐƯỢC
items.push(newItem)     // KHÔNG ĐƯỢC
```

**Đặt tên:**

```typescript
// Variables
const isLoading = true          // boolean: is/has/can/should prefix
const userCount = 42            // number: noun
const userName = "John"         // string: noun

// Functions
function fetchUserById(id)      // async get: fetch prefix
function calculateTotal(items)  // compute: calculate/compute prefix
function isValidEmail(email)    // boolean return: is/has/can prefix
function handleSubmit(event)    // event handler: handle prefix
```

---

### 3. Testing Requirements (`rules/common/testing.md`)

**Bắt buộc:**
- Tests phải được viết **TRƯỚC** code (TDD)
- Coverage tối thiểu **80%** cho mọi code
- Coverage **100%** bắt buộc cho: financial calculations, auth logic, security code
- Test names phải mô tả behavior, không mô tả implementation

```typescript
// PASS: GOOD test names
test('returns empty array when no users match search query', () => {})
test('throws AuthError when token is expired', () => {})
test('applies 10% discount when order total exceeds 100', () => {})

// FAIL: BAD test names
test('works', () => {})
test('test login', () => {})
test('function returns value', () => {})
```

**AAA Pattern bắt buộc:**

```typescript
test('calculates discount correctly', () => {
  // Arrange - Setup
  const order = { total: 150, items: [...] }

  // Act - Execute
  const discount = calculateDiscount(order)

  // Assert - Verify
  expect(discount).toBe(15)
})
```

---

### 4. Git Workflow (`rules/common/git-workflow.md`)

**Conventional Commits - PHẢI THEO:**

```
<type>(<scope>): <description>

type: feat, fix, docs, style, refactor, test, chore
scope: tên module/component (optional)
description: ngắn gọn, tiếng Anh, không dấu chấm cuối

# Ví dụ đúng:
feat(auth): add OAuth2 login with Google
fix(cart): correct discount calculation for bundle items
docs(api): update authentication endpoint docs
test(user): add unit tests for email validation

# Ví dụ sai:
Fixed some bugs
update auth
WIP
```

**Quy tắc branch:**
- `main` / `master` → production code, không push trực tiếp
- `develop` → integration branch
- `feature/tên-tính-năng` → tính năng mới
- `fix/mô-tả-bug` → sửa bug
- `hotfix/mô-tả` → emergency fix trên production

**Quy tắc commit:**
- Mỗi commit là một unit of work logic
- Không commit code bị broken
- Không commit secrets/credentials
- Không dùng `--no-verify` để skip hooks

---

### 5. Agents Rules (`rules/common/agents.md`)

Khi Claude gọi agents:
- Luôn truyền conventions từ skill vào agent prompt
- Dùng `opus` cho tasks phức tạp (architecture, security)
- Dùng `haiku` cho tasks lặp đi lặp lại (formatting, simple checks)
- Không gọi agent khi task đơn giản có thể làm trực tiếp

---

### 6. Performance Guidelines (`rules/common/performance.md`)

```typescript
// PASS: Parallel execution khi có thể
const [users, orders] = await Promise.all([
  fetchUsers(),
  fetchOrders()
])

// FAIL: Sequential khi không cần thiết
const users = await fetchUsers()
const orders = await fetchOrders()

// Database: Chỉ select columns cần thiết
const { data } = await db.from('users').select('id, name, email')

// Không select * trong production
const { data } = await db.from('users').select('*')  // BAD
```

## Rules Theo Ngôn Ngữ

### TypeScript Rules (`rules/typescript/`)

- Không dùng `any` type - luôn define proper types
- Dùng `interface` cho object shapes
- Dùng `type` cho unions và intersections
- Enable strict mode trong tsconfig

```typescript
// FAIL: BAD
function process(data: any): any { }

// PASS: GOOD
interface UserData {
  id: string
  email: string
  role: 'admin' | 'user'
}
function processUser(data: UserData): ProcessedUser { }
```

### Go Rules (`rules/golang/`)

- Error handling explicit, không dùng `_` để bỏ qua errors
- Goroutines phải có cơ chế cleanup
- Dùng context.Context cho cancellation

```go
// FAIL: BAD
result, _ := someFunction()

// PASS: GOOD
result, err := someFunction()
if err != nil {
  return fmt.Errorf("someFunction failed: %w", err)
}
```

## Cách Rules Được Áp Dụng

Rules được đặt trong thư mục `.claude/rules/` của project → Claude đọc tự động khi bắt đầu session.

Để thêm rule cho project của bạn:

```bash
# Tạo thư mục nếu chưa có
mkdir -p .claude/rules

# Tạo file rule
touch .claude/rules/my-project-rules.md
```

Nội dung:

```markdown
# My Project Rules

## Mandatory

- Luôn dùng TypeScript strict mode
- Không được thay đổi database schema mà không có migration file
- Mọi API endpoint phải có rate limiting
```

## Sự Khác Biệt Rules vs Skills

| | Rules | Skills |
|--|-------|--------|
| **Khi áp dụng** | Luôn luôn, mọi session | Chỉ khi được kích hoạt |
| **Tính chất** | Bắt buộc, không được vi phạm | Hướng dẫn tốt nhất |
| **Ví dụ** | "Không hardcode secrets" | "Cách implement TDD đúng cách" |
| **Vị trí** | `rules/` hoặc `.claude/rules/` | `skills/*/SKILL.md` |

## Lỗi Thường Gặp

**Q: Claude vẫn commit code không theo conventional commits format?**

A: Kiểm tra hook `pre:bash:commit-quality` có đang hoạt động không. Cũng đảm bảo rule `git-workflow.md` được load.

**Q: Làm sao biết rules nào đang được áp dụng?**

A: Hỏi Claude: "Liệt kê các rules bạn đang tuân theo trong project này."
