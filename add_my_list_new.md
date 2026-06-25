# API ADD MY LIST

---

### 1. Thông tin API

- **Mô tả:** API dùng để thêm, cập nhật hoặc xóa nội dung khỏi danh sách "Xem sau" (My List) của người dùng.
- **Từ hệ thống:** Mobile/Web client
- **Đến hệ thống:** Content API backend
- **Scheme:** HTTP/HTTPS
- **Xác thực:** Không yêu cầu
- **Định dạng message:** JSON
- **URL:** `/service/api/v1/contents/{contentId}/add-my-list`
- **Example URL:** `https://soft.vnptmedia.vn/service/api/v1/contents/123/add-my-list`
- **Phương thức:** POST

---

### 2. Mô tả nghiệp vụ

API này dùng để quản lý nội dung mà người dùng muốn lưu vào danh sách xem sau. Sau khi gọi API, hệ thống sẽ kiểm tra bản ghi tương ứng trong bảng `PROFILE_CONTENT_FAVOURITE` và thực hiện các thao tác sau:

| Nội dung                        | Mô tả                                                                               |
| :------------------------------ | :---------------------------------------------------------------------------------- |
| **Chức năng liên quan**         | Quản lý My List / Xem sau trên ứng dụng nội dung                                    |
| **Actor sử dụng**               | User                                                                                |
| **Điều kiện trước khi gọi API** | Người dùng đã đăng nhập và có `member_id` hợp lệ                                    |
| **Kết quả sau xử lý**           | Nội dung được thêm mới, cập nhật hoặc xóa khỏi My List tùy theo trạng thái hiện tại |

---

### 3. Request Contract

#### 3.1 Header

| Key          | Mandatory | Value mẫu          | Mô tả                               |
| :----------- | :-------: | :----------------- | :---------------------------------- |
| Content-Type |     M     | `application/json` | Định dạng dữ liệu gửi lên           |
| Accept       |     O     | `application/json` | Định dạng dữ liệu mong muốn nhận về |

#### 3.2 Parameter

| STT | Field      | Mandatory (M/O) | Data type | Description                                                        |
| :-: | :--------- | :-------------: | :-------- | :----------------------------------------------------------------- |
| 1.  | contentId  |        M        | String    | ID của nội dung cần thao tác, truyền trong URL                     |
| 2.  | type_id    |        M        | String    | Loại nội dung, ví dụ: `2`, `5`                                     |
| 3.  | member_id  |        M        | String    | ID người dùng                                                      |
| 4.  | profile_id |        O        | String    | ID profile người dùng, nếu không truyền sẽ mặc định là `"0"`       |
| 5.  | series     |        O        | Integer   | Số tập/phân đoạn của nội dung, nếu không truyền sẽ mặc định là `0` |
| 6.  | action     |        O        | Integer   | Hành động cần thực hiện, nếu không truyền sẽ mặc định là `0`       |

#### 3.3 Request Body mẫu

```json
{
  "type_id": "2",
  "member_id": "12345",
  "profile_id": "profile_001",
  "series": 0,
  "action": 2
}
```

#### 3.4 Rule validate request

| Field      | Rule validate                     | Message lỗi mong đợi         | QA note                      |
| :--------- | :-------------------------------- | :--------------------------- | :--------------------------- |
| contentId  | Bắt buộc, truyền trong path       | Không có message cố định     | Test bỏ trống / không truyền |
| type_id    | Bắt buộc                          | `Params required: type_id`   | Test bỏ trống                |
| member_id  | Bắt buộc                          | `Params required: member_id` | Test bỏ trống                |
| profile_id | Nếu không truyền thì tự gán `"0"` | Không có                     | Test bỏ trống                |
| series     | Nếu không truyền thì tự gán `0`   | Không có                     | Test bỏ trống                |
| action     | Nếu không truyền thì tự gán `0`   | Không có                     | Test bỏ trống                |

---

### 4. Response Contract

#### 4.1 Response Parameter

| STT | Field     | Mandatory (M/O) | Data type | Description                                                            |
| :-: | :-------- | :-------------: | :-------- | :--------------------------------------------------------------------- |
| 1.  | data.type |        M        | Integer   | Kết quả thao tác: `1` = xóa, `2` = cập nhật, `3` = thêm mới, `0` = lỗi |

#### 4.2 Success Response mẫu

```json
{
  "success": true,
  "message": "Thành công",
  "data": {
    "type": 3
  }
}
```

---

### 5. Response lỗi / Error code

| Status                    | Mô Tả                              |
| ------------------------- | ---------------------------------- |
| 200 OK                    | Cập nhật thành công                |
| 500 Internal server error | Tham số bắt buộc bị thiếu hoặc lỗi |
