# Phân tích yêu cầu — vai Provider

- Cặp đàm phán:Pair-03 Core ↔ Access Gate
- Product: A / B Smart Campus Access Control
- Provider service: Access Gate Service
- Consumer service:Core Management Service
- Người viết: Lê Hải Đăng
- Ngày:18/05/2026


---

## 1. Resource chính

| Resource      | Mô tả                          | Thuộc tính bắt buộc                  | Thuộc tính tùy chọn |
| ------------- | ------------------------------ | ------------------------------------ | ------------------- |
| AccessRequest | Yêu cầu xác thực quyền mở cổng | requestId, userId, gateId, timestamp | vehiclePlate        |
| AccessEvent   | Nhật ký truy cập sau khi xử lý | eventId, result, createdAt           | imageUrl            |
| GateStatus    | Trạng thái thiết bị cổng       | gateId, status                       | lastMaintenanceAt   |


---

## 2. Action/API dự kiến

| Method | Path                    | Mục đích                 | Consumer gọi khi nào? |
| ------ | ----------------------- | ------------------------ | --------------------- |
| POST   | `/access-requests`      | Gửi yêu cầu mở cổng      | Khi user quét thẻ     |
| GET    | `/access-requests/{id}` | Lấy trạng thái request   | Polling sau khi gửi   |
| GET    | `/access-events/recent` | Lấy log gần nhất         | Dashboard realtime    |
| GET    | `/gates/{id}/status`    | Kiểm tra trạng thái cổng | Trước khi gửi request |


---

## 3. Error case

Tối thiểu 5 case.

| Status | Tình huống                  | Response body dự kiến |
| -----: | --------------------------- | --------------------- |
|    400 | JSON thiếu field bắt buộc   | `Problem`             |
|    401 | Thiếu Bearer token          | `Problem`             |
|    403 | User không có quyền mở cổng | `Problem`             |
|    404 | Gate ID không tồn tại       | `Problem`             |
|    409 | Cổng đang được sử dụng      | `Problem`             |
|    422 | User bị blacklist           | `Problem`             |
|    429 | Gửi request quá nhanh       | `Problem`             |
|    503 | Thiết bị gate offline       | `Problem`             |


## 4. Giả định bổ sung

Ghi rõ những điểm user story chưa nói nhưng Provider cần giả định.

- Giả định 1:
  Mỗi gate có ID duy nhất trong toàn hệ thống.

- Giả định 2:
  Consumer đã xác thực JWT trước khi gọi API.

- Giả định 3:
  Access event chỉ lưu tối đa 90 ngày.

- Giả định 4:
  Timestamp sử dụng chuẩn ISO-8601 UTC.
---

## 5. Câu hỏi cho Consumer

1. Consumer cần polling hay webhook callback?

2. Consumer có cần pagination cho access-events không?

3. Có cần filter theo gateId hoặc userId không?

4. Timeout tối đa Consumer chấp nhận là bao nhiêu?

5. Có cần batch API không?
---

## 6. Rủi ro tích hợp

| Rủi ro                  | Tác động             | Đề xuất xử lý                |
| ----------------------- | -------------------- | ---------------------------- |
| Naming không thống nhất | Parse lỗi            | Chốt camelCase trong OpenAPI |
| Timestamp sai timezone  | Sai log audit        | Dùng ISO-8601 UTC            |
| Payload quá lớn         | Timeout Prism        | Giới hạn response size       |
| Enum không cố định      | Consumer mapping lỗi | Dùng enum schema             |
| Retry không kiểm soát   | Duplicate event      | Idempotency key              |
| API thay đổi version    | Consumer break       | VERSIONING.md                |

