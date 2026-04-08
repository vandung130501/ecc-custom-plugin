# 09 - Workflow Thực Tế

## Kết Hợp Tất Cả Lại

File này hướng dẫn cách dùng ECC trong thực tế - kết hợp Skills, Agents, Commands, Hooks và Rules vào một quy trình làm việc mượt mà.

---

## Scenario 1: Thêm Tính Năng Mới (Feature Development)

**Bối cảnh:** Bạn cần thêm chức năng "Đặt lại mật khẩu" cho ứng dụng.

### Bước 1: Lên kế hoạch

```
Bạn gõ: /plan - Thêm chức năng đặt lại mật khẩu qua email
```

ECC làm gì:
- Command `/plan` được đọc từ `commands/plan.md`
- Kích hoạt `planner` agent (`agents/planner.md`)
- Agent phân tích và trả về:

```
Kế hoạch: Password Reset Feature

1. Database
   - Thêm bảng `password_reset_tokens` (token, user_id, expires_at, used)
   - Migration: 001_add_password_reset_tokens.sql

2. Backend API
   - POST /api/auth/forgot-password → Tạo token, gửi email
   - POST /api/auth/reset-password → Validate token, đổi password

3. Email Service
   - Template email đặt lại mật khẩu
   - Integrate với SendGrid/Resend

4. Frontend
   - Trang /forgot-password (form nhập email)
   - Trang /reset-password?token=xxx (form nhập password mới)

5. Tests
   - Unit tests cho token generation/validation
   - Integration tests cho API endpoints
   - E2E test cho full flow

Ước tính: 3-4 hours
```

### Bước 2: Implement theo TDD

```
Bạn gõ: /tdd - Implement password reset token generation và validation
```

ECC làm gì:
- Command `/tdd` kích hoạt `tdd-workflow` skill
- `tdd-guide` agent hướng dẫn từng bước

**RED phase (viết test trước):**
```typescript
// Claude viết test trước
describe('PasswordResetToken', () => {
  it('generates a secure token', () => {
    const token = generateResetToken()
    expect(token).toHaveLength(64)
    expect(token).toMatch(/^[a-f0-9]+$/)
  })

  it('expires after 1 hour', () => {
    const token = createResetRecord(userId)
    expect(token.expiresAt).toBeCloseTo(Date.now() + 3600000, -3)
  })

  it('can only be used once', async () => {
    const token = await createAndUseToken(userId)
    await expect(useToken(token.value)).rejects.toThrow('Token already used')
  })
})
```

**Tests FAIL → GREEN phase (implement code):**
```typescript
// Claude implement minimal code
export function generateResetToken(): string {
  return crypto.randomBytes(32).toString('hex')
}

export async function createResetRecord(userId: string) {
  return db.passwordResetTokens.create({
    data: {
      token: generateResetToken(),
      userId,
      expiresAt: new Date(Date.now() + 3600000),
      used: false,
    }
  })
}
```

**Tests PASS → REFACTOR:**
```
Claude tự động refactor, cleanup, thêm error handling
```

### Bước 3: Commit (Hooks tự động chạy)

```
Bạn gõ: "Commit code vừa viết"
Claude: git commit -m "feat(auth): add password reset token generation"
```

Hooks tự động kích hoạt:
1. `pre:bash:block-no-verify` → Đảm bảo không bypass hooks
2. `pre:bash:commit-quality` → Kiểm tra:
   - ✓ Conventional commit format
   - ✓ Không có `console.log` bị quên
   - ✓ Không có hardcoded secrets
   - ✓ Lint passed

Nếu có vấn đề:
```
[CommitQuality] FAIL: Found console.log in src/auth/token.js:45
[CommitQuality] Remove debug statements before committing.
```

Claude tự fix, rồi commit lại.

### Bước 4: Code Review

```
Bạn gõ: /code-review
```

`code-reviewer` agent kiểm tra:
```
Code Review: Password Reset Feature

ISSUES:
- auth/token.ts:23 - Token không có rate limiting, user có thể spam reset request
- auth/api.ts:45 - Error message "User not found" có thể bị exploit để enumerate users
  → Nên: "If email exists, we'll send a reset link" (ambiguous on purpose)

SUGGESTIONS:
- Thêm index trên cột token trong database cho performance
- Thêm cron job cleanup tokens đã expired

SECURITY:
- Token length 64 chars ✓ (đủ entropy)
- Token hash trong DB ✓ (không lưu plaintext)
- Expiry 1 hour ✓ (reasonable)

OVERALL: Approve với minor fixes
```

### Bước 5: E2E Test

```
Bạn gõ: /e2e - Password reset flow từ đầu đến cuối
```

Claude tạo Playwright test:
```typescript
test('user can reset password via email', async ({ page }) => {
  // 1. Vào trang forgot password
  await page.goto('/forgot-password')

  // 2. Nhập email
  await page.fill('[data-testid=email]', 'user@example.com')
  await page.click('[data-testid=submit]')

  // 3. Kiểm tra thông báo
  await expect(page.locator('[data-testid=success-msg]'))
    .toContainText('Check your email')

  // 4. Simulate click link trong email (dùng token từ DB)
  const token = await getLatestResetToken('user@example.com')
  await page.goto(`/reset-password?token=${token}`)

  // 5. Đặt password mới
  await page.fill('[data-testid=new-password]', 'NewSecurePass123!')
  await page.click('[data-testid=submit]')

  // 6. Verify đăng nhập được với password mới
  await page.goto('/login')
  await page.fill('[data-testid=email]', 'user@example.com')
  await page.fill('[data-testid=password]', 'NewSecurePass123!')
  await page.click('[data-testid=login-btn]')
  await expect(page).toHaveURL('/dashboard')
})
```

---

## Scenario 2: Sửa Bug Khẩn Cấp

**Bối cảnh:** Production có bug - users không login được.

```
Bạn: "Bug production! Login API trả về 500 mà không có error message rõ ràng"
```

### Bước 1: Build Fix

```
Bạn: /build-fix
```

`build-error-resolver` agent đọc logs, phân tích:
```
Phân tích lỗi...

Stack trace:
TypeError: Cannot read properties of undefined (reading 'id')
  at verifyToken (auth/jwt.ts:34)
  at middleware/auth.ts:12

Root cause: JWT secret bị undefined trong production environment.
Biến JWT_SECRET chưa được set trên Railway deployment.

Fix:
1. Thêm JWT_SECRET vào Railway environment variables
2. Thêm validation khi app start: process.env.JWT_SECRET || throw new Error(...)
3. Improve error message: "Authentication service unavailable" thay vì expose stack trace
```

### Bước 2: Fix và Test

Claude sửa code, commit:
```
fix(auth): add JWT_SECRET validation at startup

- Add required env var validation
- Improve error messages for production
- Add integration test for missing env vars
```

Hook `pre:bash:commit-quality` kiểm tra → PASS.

---

## Scenario 3: Refactor Lớn

**Bối cảnh:** Cần migrate từ REST API sang GraphQL.

```
Bạn: /plan - Migrate API từ REST sang GraphQL, giữ backward compatibility
```

`architect` agent (model: opus) phân tích:

```
Architecture Plan: REST → GraphQL Migration

Strategy: Strangler Fig Pattern
- Không thay thế toàn bộ cùng lúc
- Thêm GraphQL endpoint /graphql song song với REST
- Migrate từng phần, test từng bước
- Tắt REST endpoints dần dần

Phase 1 (Week 1): Setup GraphQL
  - Cài Apollo Server / pothos
  - Expose schema đơn giản, 1-2 queries
  - Add monitoring

Phase 2 (Week 2-3): Migrate Queries
  - Users, Products queries
  - Giữ REST, test song song

Phase 3 (Week 4): Migrate Mutations
  - Create/Update/Delete operations
  - Test regression

Phase 4 (Week 5): Deprecate REST
  - Chuyển clients sang GraphQL
  - Sunset REST endpoints

Trade-offs:
- PRO: Type safety, flexible queries, subscription support
- CON: Learning curve, cần tooling update, caching phức tạp hơn
```

```
Bạn: "Bắt đầu Phase 1"

Claude:
1. /tdd - Setup Apollo Server với first query
2. Implement schema
3. /code-review - Review schema design
4. /e2e - Test GraphQL endpoint
5. Commit với message: "feat(api): add GraphQL endpoint with Apollo Server"
```

---

## Tips Workflow Hiệu Quả

### Rule Bắt Đầu Ngày

```
Mỗi ngày bắt đầu:
1. /plan cho task hôm nay
2. Chia thành chunks nhỏ (<2h mỗi chunk)
3. Implement từng chunk theo TDD
4. /code-review trước khi merge
```

### Rule Trước Khi Commit

```
Checklist tự động (hooks làm):
✓ Conventional commit message
✓ No console.log/debugger
✓ No secrets/API keys
✓ Lint passed

Checklist thủ công (bạn làm):
✓ Tests đang pass
✓ Code review đã xong
✓ Documentation updated nếu cần
```

### Rule Khi Bị Stuck

```
1. /build-fix - Nếu có lỗi compile/build
2. Đọc error message đầy đủ
3. Hỏi Claude giải thích error
4. Nếu vẫn stuck: "Explain the root cause và suggest 3 approaches"
```

### Rule Khi Code Phức Tạp

```
"Complexity budget": Nếu cần >30 phút để hiểu một đoạn code,
thì cần refactor.

1. /code-review - Identify complex areas
2. /tdd - Viết tests cho existing code trước
3. Refactor từng phần nhỏ
4. Verify tests vẫn pass sau mỗi refactor
```

---

## Bảng Tóm Tắt: Khi Nào Dùng Gì

| Tình huống | Tool dùng |
|-----------|----------|
| Tính năng mới | `/plan` → `/tdd` → `/code-review` |
| Bug cần fix | `/build-fix` → fix → `/code-review` |
| Thiết kế hệ thống | `/plan` → `architect` agent |
| Review trước merge | `/code-review` |
| Viết E2E tests | `/e2e` |
| Tổng kết session | `/learn` |
| Code chạy lâu | Hook tự nhắc dùng tmux |
| Commit không đúng format | Hook tự chặn |
| Lộ secrets | Hook tự phát hiện |
| Cần documentation | `/docs [topic]` |
| Task phức tạp, nhiều files | `/orchestrate` |

---

## Quick Reference Card

```
PHÁT TRIỂN:
  /plan [feature]      → Lên kế hoạch
  /tdd [feature]       → Implement với TDD
  /feature-dev [name]  → Full workflow

REVIEW:
  /code-review         → Review thay đổi hiện tại
  /e2e [flow]          → E2E tests

DEBUG:
  /build-fix           → Sửa build errors

GIT:
  /checkpoint          → Lưu trạng thái
  commit               → Hooks tự chạy

KNOWLEDGE:
  /learn               → Học từ session
  /docs [topic]        → Tra cứu docs
  /skill-create        → Tạo skill mới
```
