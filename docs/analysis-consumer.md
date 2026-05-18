# Phân tích yêu cầu — vai Consumer

- Cặp đàm phán: Pair-03 Core ↔ Access Gate
- Product: A / B Smart Campus Access Control
- Consumer service:  Core Management Service
- Provider service:  Access Gate Service
- Người viết: Lê Hải Đăng
- Ngày: 18/05/2026

---

## 1. Resource Consumer cần nhận/gửi

| Resource       | Consumer dùng để làm gì?  | Field bắt buộc với Consumer | Field có thể tùy chọn |
| -------------- | ------------------------- | --------------------------- | --------------------- |
| AccessRequest  | Gửi yêu cầu mở cổng       | requestId, userId, gateId   | vehiclePlate          |
| AccessDecision | Hiển thị kết quả xác thực | status, decisionAt          | reason                |
| AccessEvent    | Dashboard realtime        | eventId, createdAt, result  | imageUrl              |


## 2. API Consumer cần gọi

| Method | Path                    | Lúc nào gọi?          | Kỳ vọng response         |
| ------ | ----------------------- | --------------------- | ------------------------ |
| POST   | `/access-requests`      | Khi user scan QR      | 201 + requestId          |
| GET    | `/access-requests/{id}` | Polling trạng thái    | status + decision        |
| GET    | `/access-events/recent` | Dashboard refresh     | Danh sách event mới nhất |
| GET    | `/gates/{id}/status`    | Trước khi gửi request | ONLINE/OFFLINE           |


## 3. Error case Consumer cần xử lý

Tối thiểu 5 case.

| Status | Consumer hiểu là gì? | Consumer sẽ xử lý thế nào?  |
| -----: | -------------------- | --------------------------- |
|    400 | Payload sai format   | Validate lại request        |
|    401 | Token hết hạn        | Refresh token               |
|    403 | User không có quyền  | Disable action              |
|    404 | Gate không tồn tại   | Hiển thị “Gate unavailable” |
|    409 | Gate đang bận        | Retry sau 5s                |
|    422 | User bị blacklist    | Hiển thị lý do cụ thể       |
|    429 | Quá nhiều request    | Backoff retry               |
|    503 | Gate service offline | Circuit breaker/fallback    |


## 4. Giả định bổ sung

- Giả định 1:
  Provider luôn trả timestamp theo ISO-8601 UTC.

- Giả định 2:
  requestId là duy nhất toàn hệ thống.

- Giả định 3:
  API response time dưới 3 giây.

- Giả định 4:
  Provider hỗ trợ pagination cho list endpoint.

## 5. Câu hỏi cho Provider

1. Provider có hỗ trợ idempotency key không?

2. API có pagination strategy nào?

3. Có giới hạn rate limit không?

4. Provider có đảm bảo ordering của event không?

5. Có field nào nullable không?

## 6. Rủi ro tích hợp

| Rủi ro                        | Tác động             | Đề xuất xử lý             |
| ----------------------------- | -------------------- | ------------------------- |
| Provider đổi enum             | Consumer mapping lỗi | Freeze enum contract      |
| Nullable field không document | Crash parser         | Dùng union type + example |
| API timeout                   | Dashboard treo       | Retry + timeout           |
| Provider đổi version          | Consumer break       | Versioning policy         |
| Thiếu correlation ID          | Khó trace lỗi        | Bổ sung requestId         |
| Không thống nhất timezone     | Sai analytics        | UTC ISO-8601              |

