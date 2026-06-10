# Tóm tắt bài Lab: Data Cleaning Sprint

## Mục tiêu
Thực hành làm sạch dữ liệu (Data Sanitization) từ một file dữ liệu thô bị lỗi (`toxic_sample.json`) để đảm bảo tính an toàn (Privacy) và độ tin cậy (Reliability) trước khi đưa dữ liệu vào hệ thống AI. 

Dữ liệu thô này chứa các lỗi:
- **PII (Personally Identifiable Information)**: Tên và email thật của người dùng (cần ẩn/xóa để bảo mật).
- **Zombies**: Các bản ghi bị trùng lặp (duplicate).
- **Outliers & Garbage**: Giá trị bất thường (ví dụ: giá lớn hơn $5000 hoặc nhỏ hơn 0).

---

## Các bước thực hiện

### 1. Chuẩn bị môi trường (Tùy chọn)
- Nên sử dụng môi trường ảo (virtual environment) với Python 3.x.
- Thư viện cần thiết: `json` (đã có sẵn trong Python).

### 2. Thực hành lập trình (File `cleaning.py`)
Mở file `cleaning.py` và hoàn thiện các phần được đánh dấu `# TODO` theo thứ tự sau:

1. **Hàm `mask_email(email)`**: 
   - Che giấu địa chỉ email bằng cách chỉ giữ lại chữ cái đầu tiên của phần tên người dùng, và thêm `***` phía trước phần tên miền.
   - Ví dụ: `vana@gmail.com` -> `v***@gmail.com`.

2. **Đọc dữ liệu**:
   - Mở và đọc nội dung từ file `input_file` (được truyền vào là `toxic_sample.json`) dưới dạng JSON.

3. **Xử lý từng bản ghi trong vòng lặp**:
   - **Loại bỏ trùng lặp (Deduplication)**: Kiểm tra xem `id` của bản ghi đã tồn tại trong tập hợp `seen_ids` chưa. Nếu rồi thì bỏ qua.
   - **Loại bỏ Outliers**: Kiểm tra trường `price` (giá trị). Nếu giá > 5000 thì bỏ qua bản ghi này.
   - **Sanity Check**: Nếu giá < 0 thì cũng bỏ qua bản ghi này.
   - **Xử lý PII (Privacy)**: 
     - Xóa hoàn toàn trường `name` khỏi bản ghi.
     - Cập nhật trường `email` bằng kết quả của hàm `mask_email` đã viết ở trên.

4. **Lưu dữ liệu đã làm sạch**:
   - Ghi danh sách `sanitized_data` ra file `output_file` (`sanitized_sample.json`) với định dạng JSON (dùng `json.dump` với `indent=4` để dễ nhìn).

### 3. Chạy thử nghiệm và kiểm tra
- Mở terminal/command prompt và chạy lệnh:
  ```bash
  python cleaning.py
  ```
- Mở file `sanitized_sample.json` vừa được tạo ra để kiểm tra xem:
  - Có còn trường `name` không? (Phải không còn)
  - Các email đã được che giấu đúng định dạng chưa?
  - Không còn các bản ghi bị lặp `id` hay giá trị âm/lớn hơn 5000.
