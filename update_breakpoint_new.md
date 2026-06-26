# API UPDATE BREAKPOINT

---

### 1. Thông tin API

- **Mô tả:** API dùng để lưu hoặc cập nhật điểm dừng xem (break point) của người dùng cho một nội dung/episode cụ thể khi họ dừng xem.
- **Từ hệ thống:** Mobile/Web client
- **Đến hệ thống:** Content API backend
- **Scheme:** HTTP/HTTPS
- **Xác thực:** Không yêu cầu
- **Định dạng message:** JSON
- **URL:** `/service/api/v1/contents/{contentId}/break-point`
- **Example URL:** `http://10.144.13.61/service/api/v1/contents/123/break-point`
- **Phương thức:** PUT

---

### 2. Mô tả nghiệp vụ

API này dùng để ghi nhận tiến độ xem của người dùng tại một thời điểm cụ thể. Sau khi gọi API, hệ thống sẽ lưu/ cập nhật thông tin break point cho nội dung và tập/series đang xem, nhằm phục vụ cho việc tiếp tục xem từ vị trí đã dừng trước đó.

| Nội dung                        | Mô tả                                                                   |
| :------------------------------ | :---------------------------------------------------------------------- |
| **Chức năng liên quan**         | Lưu tiến độ xem / điểm dừng trên ứng dụng nội dung                      |
| **Actor sử dụng**               | User                                                                    |
| **Điều kiện trước khi gọi API** |                                                                         |
| **Kết quả sau xử lý**           | Hệ thống lưu hoặc cập nhật break point cho nội dung và series tương ứng |

---

### 3. Request Contract

#### 3.1 Header

| Key          | Mandatory | Value mẫu          | Mô tả                               |
| :----------- | :-------: | :----------------- | :---------------------------------- |
| Content-Type |     M     | `application/json` | Định dạng dữ liệu gửi lên           |
| Accept       |     O     | `application/json` | Định dạng dữ liệu mong muốn nhận về |

#### 3.2 Parameter

| STT | Field       | Mandatory (M/O) | Data type | Description                                    |
| :-: | :---------- | :-------------: | :-------- | :--------------------------------------------- |
| 1.  | contentId   |        M        | String    | ID của nội dung cần cập nhật, truyền trong URL |
| 2.  | type_id     |        M        | String    | Loại nội dung, ví dụ: `2`, `5`                 |
| 3.  | series_id   |        M        | String    | ID của series/tập đang được xem                |
| 4.  | break_point |        M        | Integer   | Vị trí dừng, thường là thời gian xem theo giây |
| 5.  | member_id   |        M        | String    | ID người dùng                                  |
| 6.  | profile_id  |        O        | String    | ID profile người dùng, nếu có                  |

#### 3.3 Request Body mẫu

```json
{
  "type_id": "2",
  "series_id": "series_001",
  "break_point": 1200,
  "member_id": "12345",
  "profile_id": "profile_001"
}
```

#### 3.4 Rule validate request

| Field       | Rule validate               | Message lỗi mong đợi           | QA note                      |
| :---------- | :-------------------------- | :----------------------------- | :--------------------------- |
| contentId   | Bắt buộc, truyền trong path | Không có message cố định       | Test bỏ trống / không truyền |
| type_id     | Bắt buộc                    | `Params required: type_id`     | Test bỏ trống                |
| series_id   | Bắt buộc                    | `Params required: series_id`   | Test bỏ trống                |
| break_point | Bắt buộc                    | `Params required: break_point` | Test bỏ trống                |
| member_id   | Bắt buộc                    | `Params required: member_id`   | Test bỏ trống                |
| profile_id  | Không bắt buộc              | Không có                       | Test bỏ trống                |

---

### 4. Response Contract

#### 4.1 Response Parameter

| STT | Field | Mandatory (M/O) | Data type | Description                                |
| :-: | :---- | :-------------: | :-------- | :----------------------------------------- |
| 1.  | data  |        M        | Object    | Trả về object rỗng khi cập nhật thành công |

#### 4.2 Success Response mẫu

```json
{
  "success": true,
  "message": "Thành công",
  "data": {}
}
```

---

### 5. Response lỗi / Error code

| Status                    | Mô Tả                                            |
| :------------------------ | :----------------------------------------------- |
| 200 OK                    | Cập nhật break point thành công                  |
| 500 Internal server error | Lỗi xử lý phía backend khi lưu dữ liệu           |


