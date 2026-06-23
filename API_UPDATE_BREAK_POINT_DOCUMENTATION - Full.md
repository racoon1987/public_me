# API: Update Break Point (Cập nhật Điểm Dừng)

## Tổng Quan
API này cập nhật vị trí dừng (break point) của người dùng khi xem một nội dung hoặc loạt phim cụ thể. Break point được sử dụng để lưu lại tiến độ xem, cho phép người dùng tiếp tục xem từ vị trí dừng lần trước.

---

## Endpoint

```
PUT https://soft.vnptmedia.vn/api/v1/contents/{contentId}/break-point
```

---

## Request

### Path Parameter
| Tham số | Kiểu | Bắt buộc | Mô tả |
|--------|------|----------|-------|
| `contentId` | String | Có | ID duy nhất của nội dung |

### Request Body
```json
{
  "type_id": "string",      // Kiểu nội dung (bắt buộc)
  "series_id": "string",    // ID loạt phim/tập (bắt buộc)
  "break_point": "integer", // Vị trí dừng tính bằng giây (bắt buộc)
  "member_id": "string"     // ID thành viên/người dùng (bắt buộc)
}
```

### Ví Dụ Request
```bash
curl -X PUT https://soft.vnptmedia.vn/api/v1/contents/123123123/break-point \
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
- Nếu thiếu `series_id` → Exception: "Params required: series_id"  
- Nếu thiếu `break_point` → Exception: "Params required: break_point"
- Nếu thiếu `member_id` → Exception: "Params required: member_id"

---

## Response

### Success Response (200 OK)
```json
{
  "code": "0",
  "message": "Success",
  "data": {}
}
```

### Error Response (400 Bad Request)
```json
{
  "code": "1",
  "message": "Params required: type_id",
  "data": null
}
```

### Error Response - Update Failed
```json
{
  "code": "1",
  "message": "Update Failed",
  "data": null
}
```

---

## Luồng Xử Lý Chi Tiết

### 1. **Validation & Preparation** (ContentApi.java)
- Kiểm tra sự tồn tại của 4 tham số bắt buộc: `type_id`, `series_id`, `break_point`, `member_id`
- Thêm `content_id` từ path parameter vào body
- Gọi `ContentService.updateBreakPoint(body)`

### 2. **DAO Layer - Core Logic** (ContentDaoImpl.java)

#### **Bước 1: Xử Lý Bảng `CONTENT_RECENT`**

Mục đích: Lưu lại rằng người dùng đã xem nội dung này.

- **Tìm kiếm** bản ghi hiện tại:
  ```sql
  SELECT RECENT_ID FROM CONTENT_RECENT 
  WHERE CONTENT_ID = {contentId}
    AND MEMBER_ID = {memberId}
    AND TYPE_ID = {typeId}
    AND PROFILE_ID = {profileId}
  ```

- **Nếu bản ghi tồn tại:**
  - Cập nhật thời gian sửa đổi (lần cuối xem):
    ```sql
    UPDATE CONTENT_RECENT 
    SET MODIFYDATE = sysdate 
    WHERE RECENT_ID = {recentId}
    ```

- **Nếu bản ghi không tồn tại:**
  - Tạo UUID mới
  - Chèn bản ghi mới:
    ```sql
    INSERT INTO CONTENT_RECENT 
    (RECENT_ID, CONTENT_ID, MEMBER_ID, TYPE_ID, PROFILE_ID, MODIFYDATE)
    VALUES ({uuid}, {contentId}, {memberId}, {typeId}, {profileId}, sysdate)
    ```

#### **Bước 2: Xử Lý Bảng `CONTENT_SERIES_RECENT`**

Mục đích: Lưu vị trí dừng (break point) chính xác cho loạt phim.

- **Tìm kiếm** bản ghi hiện tại:
  ```sql
  SELECT SERIES_RECENT_ID FROM CONTENT_SERIES_RECENT
  WHERE SERIES_ID = {seriesId}
    AND MEMBER_ID = {memberId}
    AND PROFILE_ID = {profileId}
  LIMIT 1
  ```

- **Nếu bản ghi tồn tại:**
  - Cập nhật thời gian và vị trí dừng:
    ```sql
    UPDATE CONTENT_SERIES_RECENT 
    SET CREATE_DATE = sysdate, BREAK_POINT = {breakPoint}
    WHERE SERIES_RECENT_ID = {seriesRecentId}
    ```

- **Nếu bản ghi không tồn tại:**
  - Tạo UUID mới
  - Chèn bản ghi mới với break point:
    ```sql
    INSERT INTO CONTENT_SERIES_RECENT 
    (SERIES_RECENT_ID, SERIES_ID, CONTENT_ID, MEMBER_ID, PROFILE_ID, CREATE_DATE, BREAK_POINT)
    VALUES ({uuid}, {seriesId}, {contentId}, {memberId}, {profileId}, sysdate, {breakPoint})
    ```

#### **Bước 3: Hoàn Tất & Trả Về**
- Nếu cả 2 bước thành công → Return `true`
- Nếu có exception → In ra exception, Return `false` → API trả về "Update Failed"

---

## Database Schema

### Bảng `CONTENT_RECENT`
| Cột | Kiểu | Mô Tả |
|-----|------|-------|
| RECENT_ID | VARCHAR(255) | Khóa chính (UUID) |
| CONTENT_ID | VARCHAR(255) | ID nội dung |
| MEMBER_ID | VARCHAR(255) | ID thành viên |
| TYPE_ID | VARCHAR(255) | Kiểu nội dung |
| PROFILE_ID | VARCHAR(255) | ID hồ sơ/profile |
| MODIFYDATE | TIMESTAMP | Thời gian sửa đổi cuối cùng |

### Bảng `CONTENT_SERIES_RECENT`
| Cột | Kiểu | Mô Tả |
|-----|------|-------|
| SERIES_RECENT_ID | VARCHAR(255) | Khóa chính (UUID) |
| SERIES_ID | VARCHAR(255) | ID loạt phim/tập |
| CONTENT_ID | VARCHAR(255) | ID nội dung |
| MEMBER_ID | VARCHAR(255) | ID thành viên |
| PROFILE_ID | VARCHAR(255) | ID hồ sơ/profile |
| CREATE_DATE | TIMESTAMP | Thời gian tạo/cập nhật |
| BREAK_POINT | INTEGER | Vị trí dừng (giây) |

---

## Ví Dụ Thực Tế

### Kịch Bản: Người dùng xem phim, dừng lại sau 20 phút

**Request:**
```bash
PUT /api/v1/contents/movie001/break-point
Content-Type: application/json

{
  "type_id": "1",
  "series_id": "episode_001",
  "break_point": 1200,
  "member_id": "user_12345"
}
```

**Quá trình xử lý:**

1. API kiểm tra toàn bộ tham số → ✓ Hợp lệ
2. Service gọi DAO
3. DAO kiểm tra `CONTENT_RECENT`:
   - **Lần đầu:** Chèn bản ghi mới với MODIFYDATE = hiện tại
   - **Lần sau:** Cập nhật MODIFYDATE = hiện tại
4. DAO kiểm tra `CONTENT_SERIES_RECENT`:
   - **Lần đầu:** Chèn bản ghi mới với BREAK_POINT = 1200
   - **Lần sau:** Cập nhật BREAK_POINT = 1200 (hoặc giá trị mới)
5. API trả về:
```json
{
  "code": "0",
  "message": "Success",
  "data": {}
}
```

---

## HTTP Status Codes

| Status | Mô Tả |
|--------|-------|
| 200 OK | Cập nhật thành công |
| 400 Bad Request | Tham số bắt buộc bị thiếu hoặc lỗi |

---

## Ghi Chú Thêm

- **Break Point Format:** Thường tính bằng **giây** (seconds)
- **Profile ID:** Cho phép multi-profile trên một tài khoản (ví dụ: cha/mẹ/con)

