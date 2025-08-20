---
title: "Storage Class Specifiers "
date: 2025-07-23 08:54:37 +0800
categories: [C]
tags: [C]
---
# 💡 Storage Class Specifiers trong C/C++

## 🧠 Tổng quan

Trong C và C++, Storage Class Specifiers (chỉ định lớp lưu trữ) được sử dụng để xác định **thời gian sống**, **phạm vi truy cập** và **vị trí trong bộ nhớ** của các biến và hàm. Việc hiểu rõ về các specifiers này giúp tối ưu hóa việc quản lý bộ nhớ và kiểm soát phạm vi của các biến.

---

## 🏷️ Storage Class Specifiers là gì?

Storage Class Specifiers là các từ khóa dùng để xác định tính chất của các biến hoặc hàm. Các specifiers này ảnh hưởng đến **phạm vi (scope)**, **liên kết (linkage)**, và **thời gian sống (lifetime)** của các biến. Các specifiers chính gồm:

- **auto**
- **register**
- **static**
- **extern**

---

## ⚙️ Các loại Storage Class Specifiers
![alt text](/assets/C/Storage_Class_Specifiers.png)
### auto

- **Phạm vi**: Biến cục bộ.
- **Liên kết**: Không có liên kết.
- **Thời gian sống**: Biến chỉ tồn tại trong khối mà nó được khai báo.
- **Mặc định**: Các biến cục bộ sẽ tự động có kiểu `auto` nếu không được khai báo rõ ràng.

#### Nếu biến cục bộ tự động là auto thì tại sao sinh ra auto để làm gì?
Mặc dù auto không cần thiết trong thực tế code C, nó vẫn hữu ích cho những người viết trình biên dịch vì:

Giúp đơn giản hóa cấu trúc cú pháp ngôn ngữ (grammar): mọi biến đều có thể có một storage class specifier (ví dụ: auto, register, static, extern), nên người viết compiler chỉ cần xử lý một quy tắc cú pháp duy nhất cho tất cả khai báo biến.

Từ khóa auto tồn tại như một phần của thiết kế ngôn ngữ thống nhất và hình thức.

#### Ví dụ:
```c
void example() {
    auto int num = 10; // 'num' là biến cục bộ
    printf("%d\n", num);
}
```

### register
- Phạm vi: Biến cục bộ.
- Liên kết: Không có liên kết.
- Thời gian sống: Biến chỉ tồn tại trong khối mà nó được khai báo.
- Sử dụng: Khuyến khích trình biên dịch lưu biến trong thanh ghi của CPU, giúp truy xuất nhanh hơn (không thể truy xuất tới địa chỉ của biến này (vì không có địa chỉ bộ nhớ).

```c
void example() {
    register int count = 0; // Biến được lưu trữ trong thanh ghi
    count++;
    printf("%d\n", count);
}
```

### static
- Phạm vi: Biến có thể là biến cục bộ hoặc toàn cục.
- Liên kết: Nếu khai báo trong hàm, biến sẽ không bị hủy và giữ lại giá trị qua các lần gọi hàm. Nếu khai báo ngoài hàm, biến chỉ có thể truy cập trong cùng một file.
- Thời gian sống: Biến tồn tại suốt chương trình.
```c
void example() {
    static int count = 0;  // Biến giữ giá trị qua các lần gọi hàm
    count++;
    printf("%d\n", count);
}
```

### extern
- Phạm vi: Biến hoặc hàm có thể truy cập từ các file khác.
- Liên kết: Biến hoặc hàm được định nghĩa ở nơi khác, có thể truy cập từ nhiều file.
- Thời gian sống: Biến tồn tại suốt chương trình.
```c
// file1.c
extern int globalVariable;  // Biến được khai báo là extern

void example() {
    printf("%d\n", globalVariable);
}

// file2.c
int globalVariable = 10;  // Biến toàn cục được định nghĩa
```

## 🛠️ Ứng dụng của Storage Class Specifiers
Các Storage Class Specifiers giúp chúng ta:
- Kiểm soát phạm vi truy cập của các biến, ví dụ như biến toàn cục chỉ có thể truy cập trong một file khi sử dụng static.
- Tối ưu hóa hiệu suất bằng cách sử dụng register để yêu cầu trình biên dịch lưu trữ các biến trong thanh ghi của CPU.
- Giữ lại giá trị của biến cục bộ qua các lần gọi hàm với static.
