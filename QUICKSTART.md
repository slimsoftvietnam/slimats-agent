# Quickstart — SlimATS Agent Skill

## 1. Clone & cài skill

```bash
git clone https://github.com/slimsoftvietnam/slimats-agent.git
cd slimats-agent
```

**Cursor** — copy vào workspace:

```bash
cp -r skills/slimats-agent /path/to/your-project/.cursor/skills/
```

**Codex / Claude Code** — copy vào thư mục skills tương ứng (ví dụ `~/.codex/skills/`).

## 2. Lấy API key

Trong SlimATS: **Admin → Cài đặt → API Agent** → copy key.

Base URL ví dụ:

```text
http://localhost/tuyendung/api/agent.php
https://slim.vn/tuyendung/phongvan/api/agent.php
```

## 3. Dùng trong chat

Ghi:

```text
Dùng skill slimats-agent.
Base URL: https://your-domain/.../api/agent.php
API key: YOUR_API_KEY
```

Hoặc dán prompt từ [`skills/slimats-agent/examples.md`](skills/slimats-agent/examples.md).

## 4. Kiểm tra kết nối

```bash
curl -H "Authorization: Bearer YOUR_API_KEY" \
  "https://your-domain/.../api/agent.php?path=health"
```

Kết quả `ok: true` → sẵn sàng dùng.
