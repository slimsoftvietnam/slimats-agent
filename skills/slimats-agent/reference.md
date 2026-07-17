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
POST profiles/{id}/attachments          (multipart/form-data)
GET profiles/{id}/attachments/{attachment_id}
```

#### Upload file đính kèm

```http
POST /api/agent.php?path=profiles/{id}/attachments
Authorization: Bearer <agent_api_key>
Content-Type: multipart/form-data
```

| Field | Bắt buộc | Mô tả |
|-------|----------|--------|
| `file` | Có* | File upload (field thay thế: `cv_file`, `attachment`) |
| `kind` | Không | `cv`, `portfolio`, `image`, `test`, `document`, `other` — tự suy từ đuôi file nếu bỏ trống |
| `is_primary_cv` | Không | `1`/`0`/`true`/`false` — mặc định `true` khi `kind=cv`; cập nhật `profile.cv_link` nếu là CV chính |

**Định dạng file:** PDF, DOC, DOCX, PNG, JPG, JPEG, WEBP  
**Giới hạn:** mặc định 10MB/file — setting `profile_upload_max_mb`

**Response `201`:**

```json
{
  "ok": true,
  "attachment": {
    "id": 12,
    "profile_id": 15,
    "filename": "cv.pdf",
    "mime_type": "application/pdf",
    "file_size": 245760,
    "kind": "cv",
    "admin_url": "https://.../admin.php?page=attachment&id=12",
    "download_url": "https://.../api/agent.php?path=profiles/15/attachments/12",
    "created_at": "2026-07-17 10:00:00"
  },
  "cv_link_updated": true,
  "profile": { "id": 15, "cv_link": "...", "...": "..." },
  "attachments": [ "..."]
}
```

**Lỗi thường gặp:** `404` hồ sơ không tồn tại · `422` thiếu file / sai định dạng / vượt dung lượng

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

### Upload file đính kèm ứng viên

```bash
curl -X POST \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -F "file=@/path/to/cv.pdf" \
  -F "kind=cv" \
  -F "is_primary_cv=1" \
  "https://your-domain.com/path/api/agent.php?path=profiles/15/attachments"
```

Field `file` (hoặc `cv_file`, `attachment`). `kind`: `cv`, `portfolio`, `image`, `test`, `document`, `other`. Khi `is_primary_cv=1` và `kind=cv`, API cập nhật `profile.cv_link`.

Định dạng: PDF, DOC, DOCX, PNG, JPG, JPEG, WEBP. Giới hạn mặc định 10MB/file (`settings.profile_upload_max_mb`).

### Tải file đính kèm

```bash
curl -H "Authorization: Bearer YOUR_API_KEY" \
  -o cv.pdf \
  "https://your-domain.com/path/api/agent.php?path=profiles/15/attachments/12"
```

## Prompt ngắn cho agent ngoài

```text
Bạn đang điều khiển SlimATS qua REST API.
Base URL: https://your-domain.com/path/api/agent.php
Authorization: Bearer YOUR_API_KEY

Khi cần dữ liệu thật, hãy gọi API thay vì suy đoán.
Ưu tiên: health, meta, profiles, profiles/search, profiles/{id}/attachments, tests, tests/{id}?expand=1, interviews, jobs, settings.
Prompt user đầy đủ: examples.md
```
