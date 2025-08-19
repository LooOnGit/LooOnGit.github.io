---
title: Type Qualifiers
author: Loo
date: 2025-08-04 06:37:29 +0800
categories: [C]
tags: [C]
---

## Tổng quan

Trong C, các **type qualifiers** (bộ định tính kiểu) là các từ khóa được sử dụng để sửa đổi các thuộc tính của các biến và kiểu dữ liệu, ảnh hưởng đến cách mà trình biên dịch xử lý chúng.

```mermaid
    B --> F[Ngăn sửa đổi giá trị]
    C --> G[Ngăn tối ưu hóa]
    D --> H[Tối ưu con trỏ]
    E --> I[Thao tác thread-safe]
```

## 1. const 🔒

Biến được khai báo với const không thể bị thay đổi sau khi được khởi tạo. 

### Các cách sử dụng const

```c
// 1. Giá trị hằng số
const int MAX_SIZE = 100;  // Không thể thay đổi giá trị

// 2. Con trỏ đến dữ liệu hằng
const int* ptr1;          // Không thể thay đổi dữ liệu qua con trỏ
// hoặc
int const* ptr2;          // Tương tự như trên

// 3. Con trỏ hằng
int* const ptr3 = &value; // Con trỏ không thể trỏ đến địa chỉ khác

// 4. Con trỏ hằng đến dữ liệu hằng
const int* const ptr4;    // Cả con trỏ và dữ liệu đều là hằng
```

## 2. Từ khóa volatile ⚡

Từ khóa `volatile` thông báo cho compiler rằng giá trị của biến có thể thay đổi bất cứ lúc nào mà không cần thông qua code. Nó dùng cho biến thay đổi nằm ngoài chương trình, như ngắt khi nhấn nút thì chương trình không biết khi nào nhấn hết.

### Các trường hợp sử dụng phổ biến

```c
// Thanh ghi phần cứng
volatile uint32_t* status_register;

// Tài nguyên chia sẻ trong môi trường đa luồng
volatile int shared_flag;

// Vòng lặp vô hạn với volatile
volatile int flag = 1;
while(flag) {
    // Flag có thể bị thay đổi bởi interrupt
}
```

> ⚠️ **Quan trọng**: Luôn sử dụng `volatile` cho các thanh ghi phần cứng và biến có thể bị truy cập bởi interrupt.

<!-- ## 3. Từ khóa restrict (C99) 🎯

Từ khóa `restrict` được sử dụng với con trỏ để báo cho compiler biết rằng con trỏ là cách duy nhất để truy cập dữ liệu mà nó trỏ đến.

```c
void process_array(int* restrict ptr1, int* restrict ptr2, int size) {
    // Compiler có thể tối ưu vì biết ptr1 và ptr2 không chồng lấn
    for(int i = 0; i < size; i++) {
        ptr1[i] = ptr2[i] * 2;
    }
}
```

## 4. Từ khóa _Atomic (C11) ⚛️

Đảm bảo truy cập nguyên tử (không bị gián đoạn) đến các biến trong lập trình đồng thời.

```c
#include <stdatomic.h>

_Atomic int shared_counter = 0;

// This operation is guaranteed to be atomic
shared_counter++;
```

## Những Thực Hành Tốt Nhất 📝

1. **Sử dụng const một cách rộng rãi**
   - Đánh dấu `const` cho tất cả các biến không nên thay đổi
   - Sử dụng con trỏ const cho tham số hàm không cần sửa đổi

2. **Sử dụng volatile**
   - Dùng cho thanh ghi phần cứng
   - Dùng cho biến được chia sẻ với interrupt
   - Không lạm dụng - nó ngăn cản tối ưu hóa

3. **Hướng dẫn sử dụng restrict**
   - Chỉ sử dụng khi chắc chắn về tính độc quyền của con trỏ
   - Hữu ích cho tối ưu hóa trong tính toán số học

4. **Cân nhắc khi dùng _Atomic**
   - Sử dụng cho biến chia sẻ trong code đa luồng
   - Cân nhắc đến hiệu năng

## Lỗi Thường Gặp ❌

1. **Nhầm lẫn về const**
   ```c
   const int* ptr;      // Không thể sửa đổi dữ liệu qua ptr
   int* const ptr;      // Không thể thay đổi ptr
   ```

2. **Thiếu volatile**
   ```c
   // Sai
   uint32_t* hw_register;
   
   // Đúng
   volatile uint32_t* hw_register;
   ```

3. **Sử dụng restrict không đúng**
   ```c
   // Nguy hiểm nếu mảng có thể chồng lấn
   void copy(int* restrict dest, int* restrict src);
   ``` -->