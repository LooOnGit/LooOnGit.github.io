---
title: Quá trình biên dịch một chương trình C
author: Loo
date: 2025-08-06
categories: [C]
tags: [C]
---

# Quá trình biên dịch một chương trình C
![alt text](/assets/C/Compiling.png)
## Quá trình biên dịch chương trình C gồm 4 giai đoạn:

### 1. **Giai đoạn tiền xử lý** (Preprocessor)
- Loại bỏ các comment
- Chỉ thị tiền xử lý (#)
- Nhận mã nguồn
Sau khi được xử lý hết ở giai đoạn này sinh ra `file.i`

**Ví dụ:** chỉ thị #include cho phép ghép thêm mã chương trình của một tệp tiêu để vào mã nguồn cần dịch thay toàn bộ nội dung của file được include. Các hằng số được định nghĩa bằng #define sẽ được thay thế bằng giá trị cụ thể tại mỗi nơi sử dụng trong chương trình.

### 2. Biên dịch (compilation)
- Phân tích cú pháp (syntax) của mã nguồn NNBC
- Chuyển chúng sang dạng mã Assembly là một ngôn ngữ bậc thấp (hợp ngữ) gần với tập lệnh của bộ vi xử lý.
Đầu ra của giai đoạn này là từ: `file.i` sang `file.s` chứa mã assembly.

### 3. Assemble
- Biên dịch `file.s` => Sang mã máy 0 và 1
- Một tệp mã máy (.obj) sinh ra trong hệ thống sau đó.
Kết quả giai đoạn này sẽ sinh ra: `file.o`

### 4. Linking (Trình liên kết)
- Trong giai đoạn này mã máy của một chương trình dịch từ nhiều nguồn (file .c hoặc file thư viện .lib) được liên kết lại với nhau để tạo thành chương trình đích duy nhất.
- Mã máy của các hàm thư viện gọi trong chương trình cũng được đưa vào chương trình cuối trong giai đoạn này.
- Chính vì vậy mà các lỗi liên quan đến việc gọi hàm hay sử dụng biến tổng thể mà không tồn tại sẽ bị phát hiện. Kể cả lỗi viết chương trình chính không có hàm main() cũng được phát hiện trong liên kết.
Kết thúc quá trình này các tạo ra file thực thi `.exe`