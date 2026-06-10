# Báo cáo Hoàn thành (Data Cleaning Sprint)

## 1. Problem Statement (Vấn đề cần giải quyết)
Trong bối cảnh xây dựng hệ thống AI, việc sử dụng dữ liệu thô (raw data) trực tiếp có thể gây ra nhiều rủi ro nghiêm trọng. Tập dữ liệu thô ban đầu (`toxic_sample.json`) gặp phải các vấn đề "độc hại" (toxic) sau:
- **Vi phạm quyền riêng tư (Privacy Risks / PII):** Dữ liệu chứa thông tin định danh cá nhân (Personally Identifiable Information) như tên thật và email của khách hàng. Việc đưa các thông tin này vào Vector DB hoặc hệ thống AI là một rủi ro lớn về bảo mật.
- **Dữ liệu trùng lặp (Zombies):** Các bản ghi bị lặp lại, khiến AI bị nhầm lẫn hoặc học sai trọng số dữ liệu.
- **Dữ liệu ngoại lệ và rác (Extreme Outliers & Garbage):** Chứa các giá trị vô lý như một chiếc bút chì giá $99,999 hoặc điện thoại có giá trị âm (-$50).

## 2. Solution (Giải pháp)
Để giải quyết các vấn đề trên, chúng ta sử dụng phương pháp **ETL (Extract - Transform - Load)** — làm sạch dữ liệu *trước khi* lưu trữ vào hệ thống đích. 

Cụ thể, một tập lệnh Python (`cleaning.py`) được xây dựng để quét qua toàn bộ tập dữ liệu thô và áp dụng các quy tắc "Sanitization" (làm sạch):
- **Bảo mật PII:** Loại bỏ hoàn toàn tên người dùng; che giấu (mask) email.
- **Khử trùng lặp:** Đảm bảo mỗi `id` sản phẩm chỉ xuất hiện duy nhất 1 lần.
- **Lọc giá trị:** Đặt ngưỡng giới hạn để loại bỏ các sản phẩm có giá trị lỗi (giá < 0) hoặc quá vô lý (giá > $5000).
- **Kết quả:** Xuất ra một tập dữ liệu mới (`sanitized_sample.json`) hoàn toàn an toàn, tinh gọn và thực tế để AI có thể sử dụng.

## 3. Quá trình Triển khai (Implementation Details)
Quá trình làm sạch được lập trình bằng ngôn ngữ Python, sử dụng thư viện `json` có sẵn, cụ thể như sau:

*   **Tách và che giấu Email (Masking Email):** 
    - Viết hàm `mask_email(email)`. Sử dụng hàm `split("@", 1)` để tách email thành `username` và `domain`.
    - Lấy chữ cái đầu tiên của `username`, sau đó ghép nối với chuỗi `***@` và phần `domain` gốc (VD: biến `vana@gmail.com` thành `v***@gmail.com`).
*   **Xóa hoàn toàn Tên người dùng:**
    - Sử dụng lệnh `del item['name']` để xóa hẳn thuộc tính `name` ra khỏi cấu trúc dữ liệu của bản ghi đó.
*   **Khử trùng lặp (Deduplication) hiệu năng cao:**
    - Khởi tạo một tập hợp `seen_ids = set()`. 
    - Khi duyệt qua danh sách, kiểm tra xem `item_id` đã nằm trong tập hợp chưa. Nếu rồi thì lệnh `continue` được gọi để bỏ qua bản ghi. Nếu chưa, ghi nhận `id` đó bằng cách dùng `seen_ids.add(item_id)`.
*   **Lọc giá trị (Outliers & Sanity check):**
    - Lấy giá trị của trường price một cách an toàn bằng `item.get('price', 0)`.
    - Đặt điều kiện rẽ nhánh: `if price > 5000` hoặc `if price < 0` thì gọi lệnh `continue` để bỏ qua bản ghi.
*   **Trích xuất và Lưu trữ:**
    - Dùng `json.load()` để đọc file thô.
    - Tạo danh sách `sanitized_data` để chứa các bản ghi đã được làm sạch.
    - Cuối cùng, dùng `json.dump()` với tham số `indent=4` để lưu danh sách này ra file `sanitized_sample.json`.

## 4. Dẫn chứng và So sánh Trước/Sau (Before vs After)

Sau khi chạy tập lệnh, kết quả tổng quan cho thấy số lượng bản ghi giảm từ **6 bản ghi (ban đầu)** xuống chỉ còn **2 bản ghi (hợp lệ)**. Chi tiết từng thay đổi:

### A. Xử lý quyền riêng tư (PII Masking)
*   **Trước (toxic_sample.json):** 
    ```json
    {
        "id": "A001",
        "name": "Nguyen Van A",
        "email": "vana@gmail.com",
        "product": "Laptop OLED"
        ...
    }
    ```
*   **Sau (sanitized_sample.json):** 
    ```json
    {
        "id": "A001",
        "email": "v***@gmail.com",
        "product": "Laptop OLED"
        ...
    }
    ```
*   **Kết quả:** Thuộc tính `"name"` đã bị xóa sổ hoàn toàn. Email gốc `vana@gmail.com` được mã hóa thành `v***@gmail.com`.

### B. Loại bỏ dữ liệu trùng lặp (Deduplication)
*   **Trước:** Có **2 bản ghi** cùng mang `"id": "A001"` (Laptop OLED của Nguyen Van A).
*   **Sau:** Chỉ còn **đúng 1 bản ghi** `"id": "A001"`. Bản ghi copy (Zombies) thứ hai đã bị thuật toán nhận diện và loại trừ thành công.

### C. Lọc giá trị ngoại lệ và rác (Outliers & Garbage)
*   **Trước:** Tồn tại ba bản ghi có giá trị lỗi/rác:
    - `"id": "A003"`: Bút chì ("Mechanical Pencil") giá `$99,999` (Outlier).
    - `"id": "A004"`: Smartphone giá `-$50` (Negative price).
    - `"id": "A005"`: Smartwatch bị bỏ trống danh mục `"category": ""` (Missing category).
*   **Sau:** Cả ba bản ghi `"id": "A003"`, `"id": "A004"`, và `"id": "A005"` đều **không xuất hiện** trong file kết quả. Tập dữ liệu sau khi làm sạch chỉ giữ lại những sản phẩm có dữ liệu hoàn thiện và giá trị hợp lý (ví dụ: `2500`, `300`).
