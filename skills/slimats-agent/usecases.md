# SlimATS — Use case cho User & Agent

Tài liệu này giúp **user** biết có thể nhờ AI làm gì, và giúp **agent** biết gọi API nào cho từng tình huống.

Thay `BASE` = `https://your-domain.com/.../api/agent.php` và `KEY` = API key từ **Cài đặt → API Agent**.

Prompt copy-paste: [examples.md](examples.md) · Endpoint chi tiết: [reference.md](reference.md) · Hướng dẫn agent: [SKILL.md](SKILL.md).

---

## Bảng tra nhanh

| User muốn… | Agent làm gì | API |
|------------|--------------|-----|
| Kiểm tra kết nối | Gọi `health` | `GET path=health` |
| Xem tổng quan hệ thống | Gọi `meta` (jobs, tests, trạng thái) | `GET path=meta` |
| Thêm 1 ứng viên | `POST profiles` | body: full_name, email, job_posting_id… |
| Thêm nhiều ứng viên | `POST profiles/bulk` | body: `{ "profiles": [...] }` |
| Tìm ứng viên tiềm năng | `POST profiles/search` | sort=priority, statuses |
| Đổi trạng thái ứng viên | `PATCH profiles/{id}` | body: status, hr_note |
| Gửi email mời PV / từ chối | `POST profiles/{id}/email` | body: `template` hoặc `template_type`, `scheduled_at`… |
| Tạo link bài test | `POST profiles/{id}/test-invite` | body: test_id (tuỳ chọn) |
| Upload CV / file đính kèm | `POST profiles/{id}/attachments` | multipart: file, kind, is_primary_cv |
| Tải file đính kèm | `GET profiles/{id}/attachments/{attachment_id}` | Bearer auth |
| Xem bài nộp test | `GET submissions` hoặc `GET submissions/{id}` | filter test_id, status |
| Lên lịch phỏng vấn | `POST interviews` | profile_id, scheduled_at… |
| Gửi email mời PV theo lịch | `POST interviews/{id}/send-email` | — |
| Xem lịch PV tuần này | `GET interviews` | date_from, date_to, status |
| Danh sách tin tuyển dụng | `GET jobs` | — |
| Tạo / sửa bài test | `GET/PATCH tests/{id}?expand=1` | sections, questions |
| Import câu hỏi hàng loạt | `POST sections/{id}/import` | body: text nhiều dòng |
| Quét email IMAP thêm hồ sơ | `POST operations/imap-scan` | cần cấu hình IMAP trước |

---

## UC-01: Thiết lập lần đầu

**User nói:** «Kết nối agent với SlimATS» / «Test API key»

**Agent:**
1. Hướng user lấy key: Admin → Cài đặt → API Agent
2. `GET BASE?path=health` — phải trả `ok: true`
3. `GET BASE?path=meta` — lấy danh sách jobs, tests, pipeline statuses

**Prompt mẫu cho user:**
> Kiểm tra kết nối SlimATS với API key của tôi, gọi health và meta, tóm tắt có bao nhiêu job và bài test.

---

## UC-02: Nhập ứng viên từ danh sách

**User nói:** «Thêm 3 ứng viên sau vào vị trí Marketing: …»

**Agent:**
```http
POST BASE?path=profiles/bulk
Content-Type: application/json

{
  "profiles": [
    {
      "full_name": "Nguyễn Văn A",
      "email": "a@example.com",
      "phone": "0901234567",
      "job_posting_id": 2,
      "source": "agent",
      "status": "new"
    }
  ]
}
```

**Lưu ý:** `job_posting_id` lấy từ `GET meta` hoặc `GET jobs`.

---

## UC-03: Lọc ứng viên tiềm năng

**User nói:** «Cho tôi 10 ứng viên tiềm năng nhất job #2»

**Agent:**
```http
POST BASE?path=profiles/search
Content-Type: application/json

{
  "job_posting_id": 2,
  "statuses": ["reviewing", "shortlisted", "interview_scheduled"],
  "sort": "priority",
  "limit": 10
}
```

Hoặc GET:
```http
GET BASE?path=profiles&job_posting_id=2&status=shortlisted&limit=10
```

---

## UC-04: Cập nhật trạng thái + ghi chú HR

**User nói:** «Đánh dấu ứng viên #15 là shortlisted, ghi chú: CV tốt, mời PV»

**Agent:**
```http
PATCH BASE?path=profiles/15
Content-Type: application/json

{
  "status": "shortlisted",
  "hr_note": "CV tốt, mời PV"
}
```

Trạng thái hợp lệ lấy từ `meta` (pipeline_statuses).

---

## UC-05: Gửi email cho ứng viên

**User nói:** «Gửi email mời phỏng vấn cho profile 15»

**Agent:** Cần SMTP đã cấu hình trong SlimATS.

```http
POST BASE?path=profiles/15/email
Content-Type: application/json

{
  "template_type": "interview",
  "scheduled_at": "2026-07-20 14:00:00",
  "location": "Online",
  "meeting_link": "https://meet.google.com/xxx"
}
```

`template` hoặc `template_type` (cùng nghĩa). Giá trị thường dùng lấy từ `meta.email_templates`: `interview`, `rejection`, `more_info`, `test_invite`, `offer`. Có thể truyền `subject`/`body` tùy chỉnh; nếu bỏ trống API render từ mẫu HR.

---

## UC-06: Tạo link mời làm bài test

**User nói:** «Tạo link bài test cho ứng viên #8»

**Agent:**
```http
POST BASE?path=profiles/8/test-invite
Content-Type: application/json

{ "test_id": 3 }
```

Bài test phải **đang mở**. Trả về object `invite` có URL làm bài.

---

## UC-14: Upload file đính kèm ứng viên

**User nói:** «Upload CV cho profile #15» / «Gắn portfolio vào hồ sơ ứng viên»

**Agent — upload CV chính:**
```http
POST BASE?path=profiles/15/attachments
Content-Type: multipart/form-data

file=@/path/to/cv.pdf
kind=cv
is_primary_cv=1
```

**Field hỗ trợ:** `file` (hoặc `cv_file`, `attachment`).

**`kind`:** `cv`, `portfolio`, `image`, `test`, `document`, `other` (tự suy từ đuôi file nếu bỏ trống).

**Định dạng:** PDF, DOC, DOCX, PNG, JPG, JPEG, WEBP.

**Giới hạn:** mặc định 10MB/file — cấu hình qua `settings.profile_upload_max_mb` (`PATCH settings`).

**Khi `is_primary_cv=1` và `kind=cv`:** API cập nhật `profile.cv_link` trỏ tới file trong admin.

**Response gồm:** `attachment`, `attachments` (toàn bộ), `cv_link_updated`, `profile`.

**Tải file:**
```http
GET BASE?path=profiles/15/attachments/12
Authorization: Bearer KEY
```

**Xem danh sách file:** `GET BASE?path=profiles/15` — field `attachments` kèm `admin_url`, `download_url`, `kind`, `mime_type`, `file_size`.

---

## UC-07: Lên lịch phỏng vấn + gửi email

**User nói:** «Lên lịch PV cho profile 5 ngày 20/07/2026 14h, link Meet …, gửi email luôn»

**Agent bước 1 — tạo lịch:**
```http
POST BASE?path=interviews
Content-Type: application/json

{
  "profile_id": 5,
  "scheduled_at": "2026-07-20 14:00:00",
  "location": "Online",
  "meeting_link": "https://meet.google.com/xxx",
  "interviewer": "HR Team",
  "status": "scheduled",
  "notes": "Vòng 1 — technical",
  "send_email": true
}
```

Nếu `send_email: true`, API gửi email mời PV ngay sau khi tạo lịch (trả về field `email` trong response).

**Hoặc tách 2 bước** — tạo lịch không `send_email`, rồi:
```http
POST BASE?path=interviews/{id}/send-email
```

---

## UC-08: Xem lịch phỏng vấn tuần này

**User nói:** «Có bao nhiêu buổi PV tuần này?»

**Agent:**
```http
GET BASE?path=interviews&date_from=2026-07-14&date_to=2026-07-20&status=scheduled
```

---

## UC-09: Xem và chấm bài nộp test

**User nói:** «Liệt kê bài nộp bài test #3 tuần này» / «Xem chi tiết submission 42»

**Agent:**
```http
GET BASE?path=submissions&test_id=3&limit=50
GET BASE?path=submissions/42
```

Đổi trạng thái bài nộp:
```http
PATCH BASE?path=submissions/42
Content-Type: application/json

{ "status": "shortlisted", "hr_note": "Điểm cao phần logic" }
```

---

## UC-10: Quản lý bài test qua API

**User nói:** «Xem cấu trúc bài test #5» / «Thêm 10 câu hỏi vào phần 12»

**Agent xem cấu trúc:**
```http
GET BASE?path=tests/5&expand=1
```

**Import nhiều câu (mỗi dòng một câu):**
```http
POST BASE?path=sections/12/import
Content-Type: application/json

{
  "text": "Câu 1: Mô tả project gần nhất?\nCâu 2: Kinh nghiệm với PHP?\nCâu 3: ...",
  "replace": false
}
```

**Lưu ý:** Export/import JSON file bài test hiện là thao tác **admin UI** (`page=tests`), chưa có path Agent API riêng — agent có thể hướng user dùng nút Xuất/Nhập JSON trên admin.

---

## UC-11: Tin tuyển dụng (careers)

**User nói:** «Tạo tin tuyển dụng mới» / «Liệt kê job đang public»

**Agent:**
```http
GET BASE?path=jobs
POST BASE?path=jobs
PATCH BASE?path=jobs/{id}
```

Chi tiết field xem response `GET jobs/{id}` hoặc admin Careers.

---

## UC-12: Quét email IMAP

**User nói:** «Quét inbox lấy CV ứng viên mới»

**Điều kiện:** IMAP đã cấu hình trong Cài đặt → Email.

**Agent:**
```http
POST BASE?path=operations/imap-scan
```

---

## UC-13: Workflow tuyển dụng điển hình (multi-step)

**User nói:** «Xử lý pipeline cho job Marketing: lọc top 5, mời test, sau đó lên PV những người đã nộp»

**Agent nên làm tuần tự:**

1. `POST profiles/search` — top 5 theo priority
2. Với mỗi profile chưa có submission: `POST profiles/{id}/test-invite`
3. `GET submissions?test_id=X&status=new` — ai đã nộp
4. Với người đạt: `PATCH profiles/{id}` status → shortlisted
5. `POST interviews` + `POST interviews/{id}/send-email`

Luôn báo cáo từng bước cho user, không gửi email hàng loạt nếu user không xác nhận.

---

## Prompt mẫu User có thể copy

Xem đầy đủ theo từng tình huống trong [examples.md](examples.md).

---

## Lỗi thường gặp

| Mã / triệu chứng | Nguyên nhân | Cách xử lý |
|------------------|-------------|------------|
| 401 Unauthorized | Sai/thiếu Bearer key | Kiểm tra Cài đặt → API Agent |
| 404 Endpoint | Sai `path` | Gọi `meta`, đối chiếu bảng trên |
| test-invite lỗi | Bài test chưa mở | Mở bài test trong admin |
| upload attachments lỗi | Sai định dạng / vượt dung lượng | PDF,DOC,DOCX,PNG,JPG,JPEG,WEBP; max `profile_upload_max_mb` MB |
| email lỗi | Chưa cấu hình SMTP | Cài đặt → Email |
| imap-scan lỗi | Chưa bật IMAP / extension | Cấu hình + PHP imap |
