# SlimATS Agent Skill

Repo **public** — Cursor/Codex skill điều khiển [SlimATS](https://github.com/slimsoftvietnam) qua Agent REST API (không cần MCP).

**Cài nhanh:** [`QUICKSTART.md`](QUICKSTART.md)

## Cấu trúc

```
slimats-agent/
├── skills/slimats-agent/
│   ├── SKILL.md        # Hướng dẫn agent chính
│   ├── examples.md     # Prompt copy-paste cho user
│   ├── usecases.md     # UC-01 → UC-13 chi tiết
│   └── reference.md    # Endpoint & curl
└── QUICKSTART.md
```

## Cài đặt nhanh (Cursor)

1. Clone repo:

```bash
git clone https://github.com/slimsoftvietnam/slimats-agent.git
```

2. Copy skill vào project hoặc thư mục skills của Cursor:

```bash
cp -r slimats-agent/skills/slimats-agent .cursor/skills/
```

Windows (PowerShell):

```powershell
Copy-Item -Recurse slimats-agent\skills\slimats-agent .cursor\skills\
```

3. Lấy API key trong SlimATS: **Admin → Cài đặt → API Agent**

4. Trong chat agent, ghi: `Dùng skill slimats-agent` và dán prompt từ [`examples.md`](skills/slimats-agent/examples.md).

## Agent API

| | |
|---|---|
| Endpoint | `{BASE_URL}/api/agent.php?path=...` |
| Auth | `Authorization: Bearer <agent_api_key>` |
| Key | Admin → Cài đặt → API Agent |

**Smoke test:**

```bash
curl -s -H "Authorization: Bearer YOUR_API_KEY" \
  "https://your-domain.com/tuyendung/api/agent.php?path=health"
```

**Luôn gọi trước phiên làm việc:**

```http
GET ?path=health
GET ?path=meta
```

Chi tiết endpoint: [`reference.md`](skills/slimats-agent/reference.md) · use case: [`usecases.md`](skills/slimats-agent/usecases.md).

## Use case phổ biến

- Thêm / lọc / cập nhật ứng viên (`profiles`, `profiles/search`, `profiles/bulk`)
- Gửi email HR, link bài test (`profiles/{id}/email`, `test-invite`)
- Lên lịch phỏng vấn (`interviews`)
- Quản lý tin tuyển dụng, bài test, câu hỏi (`jobs`, `tests`)
- Quét email IMAP (`operations/imap-scan`)

## License

Skill và tài liệu API — SlimSoft Vietnam.
