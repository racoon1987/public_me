# MẪU API CONTRACT 
## DEV BÀN GIAO QA TEST

Tài liệu kết hợp giữa format Interface/API Spec truyền thống và API Contract phục vụ QA kiểm thử.

---

### 1. Thông tin API
* **Mô tả:** [API dùng để lấy/tạo/cập nhật/xóa dữ liệu gì, phục vụ chức năng nào trên hệ thống]
* **Từ hệ thống:** [Tên hệ thống gửi request]
* **Đến hệ thống:** [Tên hệ thống nhận request]
* **Scheme:** HTTP/HTTPS
* **Xác thực:** Bearer Token / API Key / Session / Không yêu cầu
* **Định dạng message:** JSON
* **URL:** `/api/v1/[resource]`
* **Example URL:** `https://domain.com[resource]`
* **Phương thức:** GET / POST / PUT / PATCH / DELETE

---

### 2. Mô tả nghiệp vụ
[Mô tả ngắn gọn API xử lý nghiệp vụ gì, được dùng ở màn hình/chức năng nào, điều kiện để API được gọi, dữ liệu ảnh hưởng sau khi xử lý.]

| Nội dung | Mô tả |
| :--- | :--- |
| **Chức năng liên quan** | [Tên màn hình / menu / module] |
| **Actor sử dụng** | [Admin / User / System job / External system] |
| **Điều kiện trước khi gọi API** | [User đã đăng nhập, dữ liệu đã tồn tại, trạng thái hợp lệ...] |
| **Kết quả sau xử lý** | [Dữ liệu được tạo/cập nhật/xóa/trả về...] |

---

### 3. Request Contract

#### 3.1 Header

| Key | Mandatory | Value mẫu | Mô tả |
| :--- | :---: | :--- | :--- |
| Authorization | M | `Bearer {token}` | Token xác thực người dùng |
| Content-Type | M | `application/json` | Định dạng dữ liệu gửi lên |
| Accept | O | `application/json` | Định dạng dữ liệu mong muốn nhận về |

#### 3.2 Parameter

| STT | Field | Mandatory (M/O) | Data type | Description |
| :---: | :--- | :---: | :--- | :--- |
| **Request** | | | | |
| 1. | scUnitMapId | M | String | ID của tổ chức đơn vị |
| 2. | deviceId | O | String | ID của thiết bị muốn ẩn |
| 3. | serviceId | O | String | Hiện tại chỉ có 1 giá trị là 1 |

#### 3.3 Request Body mẫu
```json
{

}
```

#### 3.4 Rule validate request

| Field | Rule validate | Message lỗi mong đợi | QA note |
| :--- | :--- | :--- | :--- |
| scUnitMapId | Bắt buộc, | | Test null, empty |
| deviceId | Nếu truyền sai mã hoặc mã không tồn tại thì sao | | Test bỏ trống, sai format, mã không tồn tại |
| serviceId | Không truyền hoặc truyền sai thì mã trả về là gì? | | Test bỏ trống, mã sai |

---

### 4. Response Contract

#### 4.1 Response Parameter

| STT | Field | Mandatory (M/O) | Data type | Description |
| :---: | :--- | :---: | :--- | :--- |
| **Response** | | | | |
| 1. | | | | |
| 2. | | | | |
| 3. | | | | |
| 4. | | | | |
| 5. | | | | |
| 6. | | | | |
| 7. | | | | |
| 8. | | | | |
| 9. | | | | |
| 10. | | | | |
| 11. | | | | |
| 12. | | | | |
| 13. | | | | |
| 14. | | | | |
| 15. | | | | |
| 16. | | | | |

#### 4.2 Success Response mẫu
```json
{
  "success": true,
  "message": "Success",
  "data": [
    {
      "CATE_ID": "808080809ec952aa019ec9723ada0020",
      "CATE_NAME": "aaa",
      "CATE_CODE": "banner",
      "CATE_TYPE": "1",
      "TYPE_ID": "2",
      "POSTER_LAYOUT": "4e5f6a7b8c9d4a4b9e8f7a6b5c4d3e05",
      "VIEW_MORE": 0,
      "ITEM_COUNT": 1,
      "GET_DATA": null,
      "DATA": null,
      "SESSION": null,
      "CONTENT_OBJ": null,
      "POSTER_LAYOUT_OBJ": {
        "NAME": "Contentn ngang (lớn)",
        "DESC": null,
        "POSTER_TYPE": 1
      }
    }
  ]
}
```

---

### 5. Response lỗi / Error code

| HTTP Status | responseCode | Trường hợp | Response message mẫu | QA cần test |
| :---: | :--- | :--- | :--- | :--- |
| 400 | VALIDATION_ERROR | Dữ liệu request không hợp lệ | Dữ liệu không hợp lệ | Thiếu field bắt buộc, sai format, sai enum |
| 401 | UNAUTHORIZED | Không có token hoặc token hết hạn | Phiên đăng nhập không hợp lệ | Gọi API không truyền token |
| 403 | FORBIDDEN | User không có quyền | Bạn không có quyền thực hiện chức năng này | User khác role/permission |
| 404 | NOT_FOUND | Không tìm thấy dữ liệu | Không tìm thấy dữ liệu | Truyền feeScheduleCode không tồn tại |
| 500 | SYSTEM_ERROR | Lỗi hệ thống | Có lỗi xảy ra, vui lòng thử lại sau | Kiểm tra hệ thống không trả stacktrace |
a