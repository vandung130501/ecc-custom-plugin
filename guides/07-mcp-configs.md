# 07 - MCP Configs (Model Context Protocol)

## MCP là gì?

**MCP (Model Context Protocol)** là giao thức cho phép Claude kết nối với các công cụ và dịch vụ bên ngoài. Hiểu đơn giản: MCP giúp Claude có thêm "tay" để làm việc - browse web, đọc database, deploy code, quản lý files...

Không có MCP: Claude chỉ chat và đọc/viết files local.

Có MCP: Claude có thể browse internet, query database, deploy lên cloud, gửi Slack message, và nhiều hơn nữa.

## Cấu Trúc File `mcp-configs/mcp-servers.json`

```json
{
  "mcpServers": {
    "tên-server": {
      "command": "npx",
      "args": ["-y", "tên-package"],
      "env": {
        "API_KEY": "YOUR_KEY_HERE"
      },
      "description": "Mô tả server này làm gì"
    }
  }
}
```

## Danh Sách MCP Servers Có Sẵn

### `memory` - Bộ nhớ liên session

**Package:** `@modelcontextprotocol/server-memory`

**Làm gì:** Lưu thông tin quan trọng để Claude nhớ qua các session khác nhau.

**Dùng khi:**
- Muốn Claude nhớ preferences của bạn
- Lưu context dự án quan trọng
- Giữ notes giữa các session

**Cài đặt:**
```json
"memory": {
  "command": "npx",
  "args": ["-y", "@modelcontextprotocol/server-memory"],
  "description": "Persistent memory across sessions"
}
```

**Không cần API key.**

---

### `sequential-thinking` - Chain-of-thought reasoning

**Package:** `@modelcontextprotocol/server-sequential-thinking`

**Làm gì:** Cho Claude khả năng suy nghĩ có cấu trúc, step-by-step cho các vấn đề phức tạp.

**Dùng khi:**
- Giải quyết vấn đề logic phức tạp
- Phân tích multi-step
- Debug phức tạp

**Không cần API key.**

---

### `firecrawl` - Web scraping

**Package:** `firecrawl-mcp`

**Làm gì:** Crawl và scrape web content, chuyển thành text sạch cho Claude đọc.

**Dùng khi:**
- Claude cần đọc documentation online
- Research từ websites
- Extract data từ web pages

**Cài đặt:**
```json
"firecrawl": {
  "command": "npx",
  "args": ["-y", "firecrawl-mcp"],
  "env": {
    "FIRECRAWL_API_KEY": "YOUR_FIRECRAWL_KEY_HERE"
  }
}
```

**Cần:** API key từ firecrawl.dev

---

### `exa-web-search` - Web search

**Package:** `exa-mcp-server`

**Làm gì:** Search web với semantic search thay vì keyword search.

**Dùng khi:**
- Claude cần tìm kiếm thông tin mới nhất
- Research về thư viện, framework, API
- Tìm examples và tutorials

**Cần:** API key từ exa.ai

---

### `vercel` - Deploy lên Vercel

**Type:** HTTP MCP server

**Làm gì:** Quản lý deployments, projects trên Vercel trực tiếp từ chat.

**Dùng khi:**
- Deploy code lên Vercel
- Check deployment status
- Rollback version

**Cài đặt:**
```json
"vercel": {
  "type": "http",
  "url": "https://mcp.vercel.com"
}
```

---

### `railway` - Deploy lên Railway

**Package:** `@railway/mcp-server`

**Làm gì:** Quản lý Railway deployments, databases.

---

### `supabase` - Supabase integration

**Làm gì:** Query database, manage auth, storage trực tiếp.

**Dùng khi:**
- Claude cần đọc/viết database
- Manage Supabase projects
- Debug database issues

---

### `github` - GitHub integration

**Làm gì:** Tạo PRs, manage issues, review code trực tiếp trên GitHub.

**Dùng khi:**
- Claude cần tạo PR tự động
- Comment trên issues
- Review và merge PRs

---

### `omega-memory` - Advanced memory với semantic search

**Package:** `omega-memory`

**Làm gì:** Memory nâng cao hơn `memory` server, có semantic search và multi-agent coordination.

**Cần:** uvx được cài

---

### `playwright` - Browser automation

**Làm gì:** Điều khiển browser, chạy E2E tests, screenshot websites.

**Dùng khi:**
- E2E testing
- Visual regression testing
- Web automation

---

## Cách Cài MCP Server

### Bước 1: Thêm vào config

Chỉnh sửa `mcp-configs/mcp-servers.json` hoặc tạo `.claude/mcp.json` trong project:

```json
{
  "mcpServers": {
    "tên-server": {
      "command": "npx",
      "args": ["-y", "package-name"],
      "env": {
        "API_KEY": "your-key"
      }
    }
  }
}
```

### Bước 2: Set API keys (nếu cần)

```bash
# Trong .env hoặc export trực tiếp
export FIRECRAWL_API_KEY="fc-xxx..."
export EXA_API_KEY="exa-xxx..."
```

### Bước 3: Restart Claude Code

MCP servers cần restart để apply.

### Bước 4: Kiểm tra

Hỏi Claude: "Bạn có tool nào từ MCP server X không?"

## Cách Dùng MCP Trong Chat

Khi MCP server đã cài, Claude tự động có thêm tools. Bạn chỉ cần dùng tự nhiên:

```
"Hãy search web tìm cách implement JWT refresh token trong Node.js"
→ Claude dùng exa-web-search tự động

"Crawl trang docs.anthropic.com/claude và tóm tắt các model có sẵn"
→ Claude dùng firecrawl

"Deploy code hiện tại lên Vercel production"
→ Claude dùng Vercel MCP

"Nhớ giúp tôi: Project này dùng PostgreSQL, Node.js 20, deploy trên Railway"
→ Claude dùng memory MCP để lưu lại
```

## Tạo MCP Server Tùy Chỉnh

Nếu bạn muốn Claude có thể kết nối với hệ thống nội bộ của công ty:

```javascript
// Ví dụ đơn giản: MCP server kết nối với internal API
const { Server } = require('@modelcontextprotocol/sdk/server/index.js')

const server = new Server({
  name: 'my-internal-api',
  version: '1.0.0',
})

// Định nghĩa tool
server.setRequestHandler('tools/list', async () => ({
  tools: [{
    name: 'get_employee_info',
    description: 'Lấy thông tin nhân viên từ HR system',
    inputSchema: {
      type: 'object',
      properties: {
        employeeId: { type: 'string' }
      }
    }
  }]
}))

// Implement tool
server.setRequestHandler('tools/call', async (request) => {
  if (request.params.name === 'get_employee_info') {
    const { employeeId } = request.params.arguments
    const data = await hrApi.getEmployee(employeeId)
    return { content: [{ type: 'text', text: JSON.stringify(data) }] }
  }
})
```

## Security Khi Dùng MCP

**Cẩn thận:**
- Không commit API keys vào git
- Dùng biến môi trường cho credentials
- Giới hạn permissions của MCP servers
- Review permissions khi cài server mới

**File `.gitignore` nên có:**
```
.env
.env.local
mcp-secrets.json
```

## Sự Khác Biệt: MCP vs Tools Có Sẵn

| | Claude's built-in tools | MCP Servers |
|--|------------------------|-------------|
| **Ví dụ** | Read, Write, Bash, Grep | Firecrawl, Vercel, GitHub |
| **Cài đặt** | Tự động có sẵn | Phải cấu hình thêm |
| **Phạm vi** | Local filesystem | External services |
| **Cần config** | Không | Có |
| **Cần API key** | Không | Thường có |

## Lỗi Thường Gặp

**Q: Claude không thấy MCP server đã cài?**

A: Restart Claude Code. MCP servers cần restart để load.

**Q: Tool từ MCP trả lỗi authentication?**

A: Kiểm tra API key đã được set trong environment chưa:
```bash
echo $FIRECRAWL_API_KEY
```

**Q: MCP server crash khi Claude gọi?**

A: Check logs trong Claude Code. Đảm bảo package được cài đúng:
```bash
npx -y package-name --help
```
