# [Bài tập 2]: Thiết kế REST API hệ thống mượn sách

## I. Mục tiêu
Thiết kế các endpoint RESTful cho chức năng quản lý Sách (Book) và Thẻ mượn (Loan).
* **Mỗi sách gồm:** `id`, `title`, `author`, `year`, `quantity`.
* **Mỗi thẻ mượn gồm:** `id`, `bookId`, `borrowerName`, `borrowDate`, `returnDate`.

## II. Bảng thiết kế REST API

| Chức năng | Method | URL | Query params (nếu có) | Status thành công | Status lỗi (ví dụ) |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Lấy danh sách sách** (có phân trang & lọc theo tác giả) | `GET` | `/books` | `?page=1&limit=10`<br>`&author=...` | `200 OK` | `400 Bad Request` (sai tham số) |
| **Lấy thông tin chi tiết một cuốn sách** | `GET` | `/books/{id}` | *(Không)* | `200 OK` | `404 Not Found` (sách không tồn tại) |
| **Thêm sách mới** | `POST` | `/books` | *(Không)* | `201 Created` | `400 Bad Request` (thiếu dữ liệu bắt buộc như title, author...) |
| **Cập nhật số lượng sách** (chỉ cập nhật 1 trường) | `PATCH` | `/books/{id}` | *(Không)* | `200 OK` | `404 Not Found` (không tìm thấy sách có id này) |
| **Xóa sách** | `DELETE` | `/books/{id}` | *(Không)* | `204 No Content` | `404 Not Found` (sách không tồn tại) |
| **Lấy danh sách thẻ mượn của một sách** (sub-resource) | `GET` | `/books/{id}/loans` | *(Không)* | `200 OK` | `404 Not Found` (không tìm thấy sách gốc) |
| **Tạo thẻ mượn mới** | `POST` | `/loans` | *(Không)* | `201 Created` | `400 Bad Request` (thiếu `bookId` hoặc dữ liệu gửi lên sai) |
| **Lấy thông tin chi tiết một thẻ mượn** | `GET` | `/loans/{id}` | *(Không)* | `200 OK` | `404 Not Found` (thẻ mượn không tồn tại) |
| **Trả sách** (cập nhật ngày trả) | `PATCH` | `/loans/{id}` | *(Không)* | `200 OK` | `404 Not Found` (không tìm thấy thẻ mượn), `400 Bad Request` (sách đã được trả) |

---

## III. Giải thích chi tiết các lựa chọn thiết kế

1. **Quy tắc đặt tên (Naming Convention):** Tất cả các URL đều sử dụng danh từ số nhiều (`/books`, `/loans`) đại diện cho các tài nguyên của hệ thống, tuyệt đối không chứa động từ hành động theo đúng chuẩn RESTful.
2. **Sử dụng PATCH thay vì PUT:** * Đối với chức năng **Cập nhật số lượng sách** và **Trả sách** (chỉ cập nhật trường `returnDate`), phương thức `PATCH` (Partial Update) là lựa chọn chuẩn xác nhất vì nó chỉ thay đổi cục bộ giá trị của một hoặc một vài trường cụ thể được gửi lên trong request body, giữ nguyên các thông tin còn lại của đối tượng.
3. **Quan hệ Sub-resource (`/books/{id}/loans`):** Thẻ mượn luôn gắn liền với một cuốn sách cụ thể. Đường dẫn dạng phân cấp sub-resource này giúp thể hiện rõ ràng mối quan hệ cha-con (sách - các lượt mượn của cuốn sách đó), làm cho cấu trúc API trở nên tường minh, dễ hiểu.
4. **Mã trạng thái trả về (HTTP Status Codes):**
    * `201 Created`: Trả về khi tạo mới thành công Sách hoặc Thẻ mượn.
    * `204 No Content`: Sử dụng cho phương thức `DELETE` khi xóa thành công, vì hệ thống đã loại bỏ tài nguyên và không cần trả về thêm dữ liệu nào ở phần body.