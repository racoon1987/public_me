# API: Update Break Point (Cập nhật Điểm Dừng)

## Tổng Quan

API này cập nhật vị trí dừng (break point) của người dùng khi xem một nội dung hoặc loạt phim cụ thể. Break point được sử dụng để lưu lại tiến độ xem, cho phép người dùng tiếp tục xem từ vị trí dừng lần trước.

---

## Endpoint

Domain: https://soft.vnptmedia.vn/

```
PUT /service/api/v1/contents/{contentId}/break-point
```

---

## Request

### Path Parameter

| Tham số     | Kiểu   | Bắt buộc | Mô tả                    |
| ----------- | ------ | -------- | ------------------------ |
| `contentId` | String | Có       | ID duy nhất của nội dung |

### Request Body

```json
{
  "type_id": "string", // Kiểu nội dung (bắt buộc)
  "series_id": "string", // ID loạt phim/tập (mặc định = "0")
  "break_point": "integer", // Vị trí dừng tính bằng giây (bắt buộc)
  "member_id": "string", // ID thành viên/người dùng (bắt buộc)
  "profile_id": "string" // ID profile trong tài khoản (mặc định = "0")
}
```

### Ví Dụ Request

```bash
curl -X PUT https://soft.vnptmedia.vn/service/api/v1/contents/123123123/break-point \
  -H "Content-Type: application/json" \
  -d '{
    "type_id": "1",
    "series_id": "series456",
    "break_point": 1200,
    "member_id": "user789"
  }'
```

### Validation (Kiểm Chứng)

API sẽ trả về lỗi nếu thiếu bất kỳ tham số bắt buộc nào:

- Nếu thiếu `type_id` → Exception: "Params required: type_id"
- Nếu thiếu `break_point` → Exception: "Params required: break_point"
- Nếu thiếu `member_id` → Exception: "Params required: member_id"

---

## Response

### Success Response (200 OK)

```json
{
  "success": true,
  "message": "Thành công",
  "data": {}
}
```

### Error Response (500 Interner server error)

```json
{
  "success": false,
  "message": "System error: Params required: type_id",
  "data": null
}
```

---

## Database Schema

### Bảng `CONTENT_RECENT`

| Cột        | Kiểu         | Mô Tả                       |
| ---------- | ------------ | --------------------------- |
| RECENT_ID  | VARCHAR(255) | Khóa chính (UUID)           |
| CONTENT_ID | VARCHAR(255) | ID nội dung                 |
| MEMBER_ID  | VARCHAR(255) | ID thành viên               |
| TYPE_ID    | VARCHAR(255) | Kiểu nội dung               |
| PROFILE_ID | VARCHAR(255) | ID hồ sơ/profile            |
| MODIFYDATE | TIMESTAMP    | Thời gian sửa đổi cuối cùng |

### Bảng `CONTENT_SERIES_RECENT`

| Cột              | Kiểu         | Mô Tả                  |
| ---------------- | ------------ | ---------------------- |
| SERIES_RECENT_ID | VARCHAR(255) | Khóa chính (UUID)      |
| SERIES_ID        | VARCHAR(255) | ID loạt phim/tập       |
| CONTENT_ID       | VARCHAR(255) | ID nội dung            |
| MEMBER_ID        | VARCHAR(255) | ID thành viên          |
| PROFILE_ID       | VARCHAR(255) | ID hồ sơ/profile       |
| CREATE_DATE      | TIMESTAMP    | Thời gian tạo/cập nhật |
| BREAK_POINT      | INTEGER      | Vị trí dừng (giây)     |

---

## HTTP Status Codes

| Status          | Mô Tả                              |
| --------------- | ---------------------------------- |
| 200 OK          | Cập nhật thành công                |
| 400 Bad Request | Tham số bắt buộc bị thiếu hoặc lỗi |

---

## Ghi Chú

- **Break Point Format:** Thường tính bằng **giây** (seconds)
- **Profile ID:** Cho phép multi-profile trên một tài khoản (ví dụ: cha/mẹ/con)
