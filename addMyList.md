# API Documentation: Add My List (Xem Sau)

## Tổng Quan

API này cho phép người dùng quản lý danh sách "Xem sau" (My List) của nội dung. Hỗ trợ các hành động: thêm, xóa, hoặc cập nhật trạng thái yêu thích của nội dung.

---

## 1. Thông Tin Endpoint

Domain: https://soft.vnptmedia.vn/

| Thuộc tính         | Chi tiết                                           |
| ------------------ | -------------------------------------------------- |
| **URL**            | `/service/api/v1/contents/{contentId}/add-my-list` |
| **Method**         | `POST`                                             |
| **Tính chất**      | Tạo mới / Cập nhật hành động với nội dung          |
| **Authentication** | Có (yêu cầu member_id)                             |

---

## 2. Path Parameters

| Tham số       | Kiểu   | Bắt buộc | Mô tả                     |
| ------------- | ------ | -------- | ------------------------- |
| **contentId** | String | ✓        | ID định danh của nội dung |

---

## 3. Request Body

**Content-Type:** `application/json`

### Cấu trúc Body

```json
{
  "type_id": "2",
  "member_id": "12345",
  "profile_id": "profile_001",
  "series": 1,
  "action": 2
}
```

### Chi tiết các trường

| Trường         | Kiểu    | Bắt buộc | Mô tả                                                                                    | Ví dụ           |
| -------------- | ------- | -------- | ---------------------------------------------------------------------------------------- | --------------- |
| **type_id**    | String  | ✓        | Loại nội dung (2=Film, 5=TV Show)                                                        | `"2"`           |
| **member_id**  | String  | ✓        | ID thành viên / tài khoản                                                                | `"12345"`       |
| **profile_id** | String  | Tùy chọn | ID profile trong tài khoản                                                               | `"profile_001"` |
| **series**     | Integer | Tùy chọn | Số tập (mặc định: 0)                                                                     | `1`             |
| **action**     | Integer | Tùy chọn | Hành động:<br>• `1` = Yêu thích (Favorite)<br>• `2` = Xem sau (My List)<br>(mặc định: 0) | `2`             |

---

## 4. Response

### Success Response (HTTP 200)

```json
{
  "success": true,
  "message": "Thành công",
  "data": {
    "type": 3
  }
}
```

### Response Data - Giải thích `type` field

| Giá trị | Ý nghĩa | Hành động đã thực hiện               |
| ------- | ------- | ------------------------------------ |
| **1**   | DELETE  | Xóa record khỏi My List              |
| **2**   | UPDATE  | Cập nhật hành động (thay đổi action) |
| **3**   | INSERT  | Thêm mới vào My List                 |
| **0**   | ERROR   | Lỗi trong quá trình xử lý            |

### Error Response

```json
{
  "success": false,
  "message": "System error: Params required: type_id",
  "data": null
}
```

---

### Database Tables Liên quan

#### 1. **PROFILE_CONTENT_FAVOURITE** (Bảng chính)

```sql
CREATE TABLE PROFILE_CONTENT_FAVOURITE (
    CONTENT_ID       VARCHAR2(50),     -- ID nội dung
    PROFILE_ID       VARCHAR2(50),     -- ID profile
    MEMBER_ID        VARCHAR2(50),     -- ID thành viên
    TYPE_ID          VARCHAR2(50),     -- Loại nội dung
    ACTION           INTEGER,          -- 1=Favorite, 2=MyList, ...
    SERIES           INTEGER,          -- Số tập
    CREATEDATE       TIMESTAMP         -- Ngày tạo
);
```

**Logic:**

- `ACTION = 1`: Nội dung yêu thích
- `ACTION = 2`: Nội dung trong danh sách "Xem sau"
- Khi add-my-list với cùng `action` → DELETE (toggle off)
- Khi thay đổi `action` → UPDATE

---

## 6. Ví dụ Sử dụng

### Ví dụ 1: Thêm nội dung vào "Xem sau"

**Request:**

```bash
POST /service/api/v1/contents/content123/add-my-list
Content-Type: application/json

{
  "type_id": "2",
  "member_id": "user456",
  "profile_id": "profile_001",
  "series": 0,
  "action": 2
}
```

**Response (Lần đầu tiên):**

```json
{
  "success": true,
  "message": "Thành công",
  "data": {
    "type": 3
  }
}
```

→ **Kết quả:** Record mới được INSERT vào PROFILE_CONTENT_FAVOURITE với ACTION=2

---

### Ví dụ 2: Xóa nội dung khỏi "Xem sau"

**Request:** (Giống như ví dụ 1, send lại cùng dữ liệu)

```json
{
  "type_id": "2",
  "member_id": "user456",
  "profile_id": "profile_001",
  "series": 0,
  "action": 2
}
```

**Response (Lần thứ 2):**

```json
{
  "success": true,
  "message": "Thành công",
  "data": {
    "type": 1
  }
}
```

→ **Kết quả:** Record bị DELETE (Toggle OFF). Nội dung xóa khỏi "Xem sau"

---

### Ví dụ 3: Thay đổi từ "Xem sau" thành "Yêu thích"

**Request:**

```json
{
  "type_id": "2",
  "member_id": "user456",
  "profile_id": "profile_001",
  "series": 0,
  "action": 1
}
```

**Response:**

```json
{
  "success": true,
  "message": "Thành công",
  "data": {
    "type": 2
  }
}
```

→ **Kết quả:** Record UPDATE, ACTION thay đổi từ 2 → 1
