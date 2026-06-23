# API Documentation: Add My List (Xem Sau)

## Tổng Quan
API này cho phép người dùng quản lý danh sách "Xem sau" (My List) của nội dung. Hỗ trợ các hành động: thêm, xóa, hoặc cập nhật trạng thái yêu thích của nội dung.

---

## 1. Thông Tin Endpoint

Domain: https://soft.vnptmedia.vn/

| Thuộc tính | Chi tiết |
|-----------|---------|
| **URL** | `/api/v1/contents/{contentId}/add-my-list` |
| **Method** | `POST` |
| **Tính chất** | Tạo mới / Cập nhật hành động với nội dung |
| **Authentication** | Có (yêu cầu member_id) |

---

## 2. Path Parameters

| Tham số | Kiểu | Bắt buộc | Mô tả |
|---------|------|---------|-------|
| **contentId** | String | ✓ | ID định danh của nội dung |

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

| Trường | Kiểu | Bắt buộc | Mô tả | Ví dụ |
|--------|------|---------|-------|-------|
| **type_id** | String | ✓ | Loại nội dung (2=Film, 5=TV Show) | `"2"` |
| **member_id** | String | ✓ | ID thành viên / tài khoản | `"12345"` |
| **profile_id** | String | Tùy chọn | ID profile trong tài khoản | `"profile_001"` |
| **series** | Integer | Tùy chọn | Số tập (mặc định: 0) | `1` |
| **action** | Integer | Tùy chọn | Hành động:<br>• `1` = Yêu thích (Favorite)<br>• `2` = Xem sau (My List)<br>(mặc định: 0) | `2` |

### Validation & Logic xử lý

```java
// Các trường sẽ được mặc định nếu không truyền
if (body.get("profile_id") == null) {
    body.put("profile", "0");
}
if (body.get("series") == null) {
    body.put("profile", 0);
}
if (body.get("action") == null) {
    body.put("profile", 0);
}

// contentId được thêm vào từ path
body.put("content_id", contentId);
```

---

## 4. Response

### Success Response (HTTP 200)

```json
{
  "code": "00",
  "msg": "Success",
  "data": {
    "type": 3
  }
}
```

### Response Data - Giải thích `type` field

| Giá trị | Ý nghĩa | Hành động đã thực hiện |
|--------|--------|----------------------|
| **1** | DELETE | Xóa record khỏi My List |
| **2** | UPDATE | Cập nhật hành động (thay đổi action) |
| **3** | INSERT | Thêm mới vào My List |
| **0** | ERROR | Lỗi trong quá trình xử lý |

### Error Response

```json
{
  "code": "E001",
  "msg": "Params required: type_id",
  "data": null
}
```

#### Các lỗi có thể xảy ra

| Lỗi | Nguyên nhân | HTTP Status |
|-----|-----------|------------|
| `Params required: type_id` | Thiếu type_id | 200 (nhưng được xem là lỗi logic) |
| `Params required: member_id` | Thiếu member_id | 200 |
| Exception during database operation | Lỗi database hoặc transaction | 200 (với error response) |

---

## 5. Cơ chế Hoạt động Chi tiết

### Quy Trình Xử Lý (Database Transaction)

```
Client POST request
         ↓
[ContentApi.addMyList()]
   ├─ Validate tham số bắt buộc (type_id, member_id)
   ├─ Gán content_id từ path vào body
   │
   └─→ [ContentService.saveContentFavourite(body)]
       │
       └─→ [ContentDaoImpl.saveContentFavourite(body)]
           │
           ├─ Kiểm tra record tồn tại trong PROFILE_CONTENT_FAVOURITE
           │
           ├─ TH1: Record tồn tại
           │  └─ So sánh old_action vs new_action (p_action)
           │     ├─ Nếu giống nhau → DELETE (return 1)
           │     └─ Nếu khác nhau → UPDATE (return 2)
           │
           └─ TH2: Record không tồn tại
              └─ INSERT record mới (return 3)
```

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
POST /api/v1/contents/content123/add-my-list
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
  "code": "00",
  "msg": "Success",
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
  "code": "00",
  "msg": "Success",
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
  "code": "00",
  "msg": "Success",
  "data": {
    "type": 2
  }
}
```
→ **Kết quả:** Record UPDATE, ACTION thay đổi từ 2 → 1

---

## 7. Trạng thái Nội dung

### Kiểm tra trạng thái nội dung

API này liên kết với endpoint `/api/v1/contents/{contentId}` (GET) để kiểm tra:

```json
{
  "IS_FAVORITE": 1,      // 1 = Đã yêu thích, 0 = Chưa
  "IS_MYLIST": 1         // 1 = Trong "Xem sau", 0 = Chưa
}
```

Logic kiểm tra:
```java
// Từ ContentService.getDetail()
result.put("IS_FAVORITE", 
    contentDao.checkFavoriteContentOfMember(contentId, memberId) ? 1 : 0
);
result.put("IS_MYLIST", 
    contentDao.checkContentIsMyList(contentId, memberId)
);
```

---

## 8. Liên Kết với Các API Khác

| API | Mô tả | Liên hệ |
|-----|------|--------|
| `GET /api/v1/contents/{contentId}` | Lấy chi tiết nội dung | Có trường `IS_FAVORITE`, `IS_MYLIST` |
| `PUT /api/v1/contents/{contentId}/break-point` | Cập nhật vị trí xem | Cùng database table |
| `GET /api/v1/contents/categories` | Danh sách danh mục | Lọc nội dung từ My List |

---

## 9. Best Practices & Lưu Ý

### Do's ✓
- Luôn gửi **type_id** và **member_id** (các tham số bắt buộc)
- Sử dụng toggle pattern: gửi cùng dữ liệu lần 2 để xóa
- Cached IS_MYLIST result từ detail endpoint khi có thể

### Don'ts ✗
- Không gửi request mà không có type_id
- Không mix-up action values (1 vs 2) nếu không hiểu logic
- Không assume thứ tự thực hiện nếu gửi nhiều request liên tiếp

### Performance Considerations
- Database transaction được bảo vệ: `@Transactional(rollbackFor = Exception.class)`
- Nên cache kết quả IS_MYLIST khoảng 5-10 phút
- Batch operations: nên group updates để tránh N+1 queries

---

## 10. Error Handling

### Kiểm tra lỗi từ Response

```javascript
// Client-side example
if (response.data.type === 0) {
    // Lỗi xảy ra, kiểm tra msg
    console.error("Lỗi:", response.data.msg);
} else if (response.data.type === 1) {
    console.log("Đã xóa khỏi My List");
} else if (response.data.type === 2) {
    console.log("Đã cập nhật");
} else if (response.data.type === 3) {
    console.log("Đã thêm vào My List");
}
```

### Xử lý Exception

```java
catch (Exception e) {
    e.printStackTrace();
    return Common.errorResponse(e, "addMyList");
    // Trả về error message với exception details
}
```

---

## 11. Security

- ✓ Yêu cầu **member_id** (xác định người dùng)
- ✓ Yêu cầu **type_id** (xác định loại nội dung)
- ✓ Transaction-safe (Rollback nếu lỗi)
- ⚠️ Không có rate limiting (nên implement ở Gateway)
- ⚠️ Không validate profile ownership (nên thêm check)

---

## 12. Versioning

| Phiên bản | Ngày | Ghi chú |
|----------|------|--------|
| v1 | 2024 | Initial version |
| v1 (hiện tại) | 2026 | Endpoint: `/api/v1/contents/{contentId}/add-my-list` |

---

## 13. Test Cases

### Test 1: Add to My List (Thêm vào "Xem sau")
```
Điều kiện: Member chưa add nội dung này
Kết quả mong muốn: return type = 3 (INSERT)
```

### Test 2: Remove from My List (Xóa khỏi "Xem sau")
```
Điều kiện: Member đã add nội dung này (action=2)
Kết quả mong muốn: return type = 1 (DELETE)
```

### Test 3: Change Action (Thay đổi action)
```
Điều kiện: Member đã add (action=2), gửi action=1
Kết quả mong muốn: return type = 2 (UPDATE)
```

### Test 4: Validation Error
```
Điều kiện: Gửi request mà thiếu type_id
Kết quả mong muốn: Error message "Params required: type_id"
```

---

## Tài liệu Liên Quan

- [updateBreakPoint.md](updateBreakPoint.md) - Cập nhật vị trí xem
- [updateBreakPoint - Full.md](updateBreakPoint%20-%20Full.md) - Tài liệu chi tiết break point
