# SlimATS Agent Reference

Use case & prompt: [usecases.md](usecases.md) · [examples.md](examples.md) · [SKILL.md](SKILL.md)

## Base endpoint

```text
/api/agent.php?path=...
```

## Auth

```http
Authorization: Bearer <agent_api_key>
```

Nếu sai key, API trả `401 Unauthorized`.

## Endpoint thường dùng

### Kiểm tra kết nối

```text
GET health
GET meta
```

`GET meta` trả về: `pipeline_statuses`, `interview_statuses`, `email_templates`, `jobs`, `tests`, `profile_sources`, `endpoints`.

### Ứng viên

```text
GET profiles
POST profiles
POST profiles/bulk
POST profiles/search
GET profiles/{id}
PATCH profiles/{id}
POST profiles/{id}/email
POST profiles/{id}/test-invite
```

### Bài test

```text
GET tests
POST tests
GET tests/{id}
GET tests/{id}?expand=1
PATCH tests/{id}
POST tests/{id}/copy
POST tests/{id}/sections
PATCH/DELETE sections/{id}
POST sections/{id}/questions
POST sections/{id}/import
PATCH/DELETE questions/{id}
```

### Tin tuyển dụng

```text
GET jobs
POST jobs
GET jobs/{id}
PATCH jobs/{id}
POST jobs/{id}/copy
```

### Bài nộp và phỏng vấn

```text
GET submissions
GET submissions/{id}
PATCH submissions/{id}

GET interviews
POST interviews
GET interviews/{id}
PATCH interviews/{id}
DELETE interviews/{id}
POST interviews/{id}/send-email
```

### Cài đặt

```text
GET settings
PATCH settings
POST operations/imap-scan
```

## Ví dụ curl

### Health

```bash
curl -H "Authorization: Bearer YOUR_API_KEY" \
  "https://your-domain.com/path/api/agent.php?path=health"
```

### Danh sách bài test

```bash
curl -H "Authorization: Bearer YOUR_API_KEY" \
  "https://your-domain.com/path/api/agent.php?path=tests"
```

### Chi tiết bài test có phần + câu hỏi

```bash
curl -H "Authorization: Bearer YOUR_API_KEY" \
  "https://your-domain.com/path/api/agent.php?path=tests/5&expand=1"
```

### Tìm ứng viên tiềm năng

```bash
curl -X POST \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"statuses":["reviewing","shortlisted"],"sort":"priority"}' \
  "https://your-domain.com/path/api/agent.php?path=profiles/search"
```

### Tạo lịch phỏng vấn

```bash
curl -X POST \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"profile_id":5,"scheduled_at":"2026-07-20 14:00:00","location":"Văn phòng","meeting_link":"","interviewer":"HR team","status":"scheduled"}' \
  "https://your-domain.com/path/api/agent.php?path=interviews"
```

Thêm `"send_email":true` trong body để gửi email mời PV ngay khi tạo lịch.

### Gửi email mời phỏng vấn (profile)

```bash
curl -X POST \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"template_type":"interview","scheduled_at":"2026-07-20 14:00:00","meeting_link":"https://meet.google.com/xxx"}' \
  "https://your-domain.com/path/api/agent.php?path=profiles/15/email"
```

## Prompt ngắn cho agent ngoài

```text
Bạn đang điều khiển SlimATS qua REST API.
Base URL: https://your-domain.com/path/api/agent.php
Authorization: Bearer YOUR_API_KEY

Khi cần dữ liệu thật, hãy gọi API thay vì suy đoán.
Ưu tiên: health, meta, profiles, profiles/search, tests, tests/{id}?expand=1, interviews, jobs, settings.
Prompt user đầy đủ: examples.md
```
