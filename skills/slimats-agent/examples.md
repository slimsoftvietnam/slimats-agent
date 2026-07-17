# SlimATS — Prompt mẫu (copy & paste)

Dùng khi **user** muốn nhờ AI thao tác SlimATS bằng prompt thường, **không cần MCP**.

Thay trước khi gửi:
- `BASE` → URL thật, ví dụ `https://slim.vn/tuyendung/phongvan/api/agent.php`
- `YOUR_API_KEY` → key từ **Admin → Cài đặt → API Agent**

---

## Prompt hệ thống (dán vào agent một lần)

```text
Bạn điều khiển SlimATS (phần mềm tuyển dụng) qua REST API.

Base URL: BASE
Authorization: Bearer YOUR_API_KEY

Quy tắc:
1. Khi cần dữ liệu thật, gọi API — không suy đoán.
2. Luôn gọi GET path=health trước khi làm việc phức tạp.
3. Dùng GET path=meta để lấy danh sách jobs, tests, pipeline_statuses.
4. Trước khi gửi email hoặc đổi trạng thái hàng loạt: liệt kê đối tượng và hỏi xác nhận.
5. Trả lời tiếng Việt, ngắn gọn.
```

---

## Prompt user — Thiết lập & kiểm tra (UC-01)

```text
Dùng skill slimats-agent. Kiểm tra kết nối SlimATS:
- Base: BASE
- Key: YOUR_API_KEY
Gọi health và meta, báo có bao nhiêu job, bài test, và các trạng thái pipeline.
```

```text
API SlimATS của tôi trả 401. Hướng dẫn kiểm tra Bearer key và URL đúng.
```

---

## Prompt user — Ứng viên (UC-02, UC-03, UC-04)

```text
Thêm 3 ứng viên sau vào job Marketing (job_posting_id lấy từ meta):
1. Nguyễn Văn A — a@example.com — 0901111111
2. Trần Thị B — b@example.com — 0902222222
3. Lê Văn C — c@example.com — 0903333333
Dùng SlimATS Agent API.
```

```text
Lọc 10 ứng viên tiềm năng nhất cho job #2 (sort=priority, statuses: reviewing, shortlisted).
```

```text
Đánh dấu profile #15 là shortlisted, ghi chú HR: "CV tốt, mời PV tuần sau".
```

```text
Xem chi tiết hồ sơ profile #15 (kèm lịch PV và file đính kèm nếu có).
```

---

## Prompt user — Email & bài test (UC-05, UC-06)

```text
Gửi email mời phỏng vấn cho profile #15:
- Ngày giờ: 20/07/2026 14:00
- Link Meet: https://meet.google.com/xxx
Dùng template interview qua API.
```

```text
Tạo link mời làm bài test cho profile #8, test_id=3.
```

```text
Thêm ứng viên mới và tạo luôn link bài test (create_test_invite) cho job #2.
```

---

## Prompt user — Phỏng vấn (UC-07, UC-08)

```text
Lên lịch phỏng vấn cho profile #5:
- 20/07/2026 14:00
- Online, link Meet: https://meet.google.com/xxx
- Người PV: HR Team
Sau đó gửi email mời PV.
```

```text
Lên lịch PV cho profile #5 và gửi email luôn trong một request (send_email: true).
```

```text
Liệt kê tất cả lịch phỏng vấn tuần này (status=scheduled).
```

```text
Hủy / đổi trạng thái lịch phỏng vấn #12 qua API.
```

---

## Prompt user — Bài nộp & pipeline (UC-09, UC-13)

```text
Liệt kê bài nộp của bài test #3, mới nhất trước.
```

```text
Xem chi tiết submission #42 và đề xuất có nên chuyển sang shortlisted không.
```

```text
Workflow job Marketing:
1) Lọc top 5 ứng viên tiềm năng
2) Mời làm bài test những người chưa nộp
3) Báo cáo ai đã nộp
4) Đề xuất ai nên lên PV
Chờ tôi xác nhận trước khi gửi email.
```

---

## Prompt user — Bài test & tin tuyển dụng (UC-10, UC-11)

```text
Xem cấu trúc bài test #5 (expand=1): có bao nhiêu phần, bao nhiêu câu?
```

```text
Import 5 câu hỏi vào phần #12 của bài test (mỗi dòng một câu, không xóa câu cũ).
```

```text
Liệt kê tin tuyển dụng đang active và URL public của từng tin.
```

---

## Prompt user — IMAP & cài đặt (UC-12)

```text
Quét inbox IMAP để lấy CV ứng viên mới (operations/imap-scan).
```

```text
Xem cài đặt SMTP/IMAP hiện tại qua API settings (không in password).
```

---

## Prompt cho agent khi user chưa có key

```text
Hướng dẫn tôi lấy API key SlimATS và test kết nối bằng curl, không dùng MCP.
```

---

## Liên kết

- Use case chi tiết: [usecases.md](usecases.md)
- Endpoint & curl: [reference.md](reference.md)
- Hướng dẫn agent: [SKILL.md](SKILL.md)
