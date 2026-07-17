---
name: slimats-agent
description: Connect to and operate SlimATS via its Agent REST API without MCP. Use when the user mentions SlimATS, ATS API, agent_api, api/agent.php, Bearer key, recruitment workflows, add candidates, upload CV attachments, schedule interviews, send HR emails, test invites, profiles, tests, interviews, jobs, or wants use-case examples for controlling the ATS through prompt-driven HTTP calls.
disable-model-invocation: true
---

# SlimATS Agent

## Mục tiêu

Skill này hướng dẫn agent kết nối và điều khiển SlimATS qua REST API `api/agent.php` bằng Bearer key, không cần MCP.

## Khi dùng skill này

Dùng ngay khi user muốn:

- kết nối AI agent với SlimATS
- thao tác ATS qua prompt bình thường
- đọc/ghi dữ liệu `profiles`, `tests`, `jobs`, `interviews`, `submissions`, `settings`
- viết prompt hệ thống, curl, fetch, Python, n8n, Make để gọi SlimATS

## Kết nối chuẩn

1. Lấy API key trong `Admin -> Cài đặt -> API Agent`
2. Xác định base URL:

```text
https://your-domain.com/.../api/agent.php
```

3. Mọi request phải có header:

```http
Authorization: Bearer <agent_api_key>
```

4. Endpoint dùng query `?path=...`, ví dụ:

```text
GET /api/agent.php?path=health
GET /api/agent.php?path=meta
GET /api/agent.php?path=tests
GET /api/agent.php?path=tests/5&expand=1
GET /api/agent.php?path=profiles
POST /api/agent.php?path=profiles/search
GET /api/agent.php?path=interviews
GET /api/agent.php?path=jobs
PATCH /api/agent.php?path=settings
POST /api/agent.php?path=profiles/{id}/attachments
```

Upload file: `POST profiles/{id}/attachments` với `Content-Type: multipart/form-data` (field `file`, tuỳ chọn `kind`, `is_primary_cv`). Chi tiết: [reference.md](reference.md#upload-file-đính-kèm).

## Quy trình trả lời

1. Xác nhận user muốn kết nối theo cách nào:
   - prompt/system prompt
   - curl
   - JavaScript/fetch
   - Python
   - n8n/Make
2. Luôn cung cấp:
   - base URL
   - header Bearer
   - 2-5 ví dụ request thực tế
3. Nếu user muốn agent “hiểu cách điều khiển SlimATS”, hãy đưa:
   - prompt hệ thống mẫu
   - danh sách endpoint chính
   - quy tắc không suy đoán dữ liệu khi chưa gọi API
4. Nếu user muốn thao tác cập nhật dữ liệu, nhấn mạnh dùng đúng method `POST`, `PATCH`, `DELETE`.

## Endpoint chính

- `health`, `meta`
- `settings`
- `jobs`, `jobs/{id}`
- `tests`, `tests/{id}?expand=1`
- `sections/{id}`, `questions/{id}`
- `profiles`, `profiles/search`, `profiles/{id}`
- `profiles/{id}/email`
- `profiles/{id}/test-invite`
- `profiles/{id}/attachments` (POST upload, GET `.../attachments/{id}` download)
- `submissions`, `submissions/{id}`
- `interviews`, `interviews/{id}`
- `operations/imap-scan`

Đọc thêm:
- [examples.md](examples.md) — prompt copy-paste cho user
- [reference.md](reference.md) — endpoint & curl
- [usecases.md](usecases.md) — use case theo tình huống user/agent

## Cách user dùng skill

1. Lấy API key: **Admin → Cài đặt → API Agent**
2. Mở chat agent, ghi: `Dùng skill slimats-agent`
3. Dán prompt từ [examples.md](examples.md) hoặc mô tả nhiệm vụ tự nhiên
4. Agent gọi REST API `api/agent.php` — **không cần MCP**

## Use case phổ biến (tóm tắt)

| Tình huống | User có thể nói | API chính |
|------------|-----------------|-----------|
| Kiểm tra kết nối | «Test API SlimATS» | `GET health`, `GET meta` |
| Thêm ứng viên | «Thêm 3 UV vào job #2» | `POST profiles/bulk` |
| Upload CV / file | «Upload CV cho profile #15» | `POST profiles/{id}/attachments` |
| Lọc tiềm năng | «Top 10 UV shortlisted job Marketing» | `POST profiles/search` |
| Đổi trạng thái | «Profile 15 → shortlisted» | `PATCH profiles/{id}` |
| Gửi email | «Gửi email mời PV cho #15» | `POST profiles/{id}/email` |
| Link bài test | «Tạo link test cho #8» | `POST profiles/{id}/test-invite` |
| Lên lịch PV | «PV profile 5 ngày 20/7 14h» | `POST interviews` |
| Gửi email lịch PV | «Gửi email mời theo lịch #12» | `POST interviews/{id}/send-email` |
| Xem lịch tuần | «Lịch PV tuần này» | `GET interviews?date_from&date_to` |
| Bài nộp test | «Bài nộp test #3» | `GET submissions` |
| Cấu trúc bài test | «Xem câu hỏi test #5» | `GET tests/{id}?expand=1` |
| Quét email CV | «Quét inbox IMAP» | `POST operations/imap-scan` |

Chi tiết từng bước, body JSON và prompt mẫu: [usecases.md](usecases.md) · [examples.md](examples.md).

## Prompt hệ thống mẫu

Dùng mẫu này khi user muốn agent thường kết nối SlimATS mà không cần MCP:

```text
Bạn đang điều khiển SlimATS qua REST API.

Base URL:
https://your-domain.com/path/api/agent.php

Authorization:
Bearer YOUR_API_KEY

Nguyên tắc:
- Khi cần dữ liệu thật, hãy gọi API thay vì suy đoán.
- Ưu tiên dùng các path health, meta, profiles, profiles/search, profiles/{id}/attachments, tests, tests/{id}?expand=1, interviews, jobs, settings.
- Khi cập nhật dữ liệu, dùng đúng method POST/PATCH/DELETE.
- Tóm tắt kết quả ngắn gọn bằng tiếng Việt.
```

## Smoke test tối thiểu

```bash
curl -H "Authorization: Bearer YOUR_API_KEY" \
  "https://your-domain.com/path/api/agent.php?path=health"
```

Sau đó:

```bash
curl -H "Authorization: Bearer YOUR_API_KEY" \
  "https://your-domain.com/path/api/agent.php?path=meta"
```

## Ghi chú

- Không yêu cầu MCP nếu user không muốn.
- Nếu user hỏi «làm được gì» / «use case», mở [usecases.md](usecases.md) hoặc [examples.md](examples.md).
- Nếu user chỉ cần “dùng bằng prompt”, đưa prompt từ [examples.md](examples.md).
- Nếu user muốn tự động hóa nhiều bước, đề xuất thêm ví dụ `curl`, `fetch`, `Python`, hoặc workflow n8n.
- Trước email hàng loạt hoặc đổi trạng thái hàng loạt: liệt kê đối tượng và xác nhận với user (xem UC-13).
