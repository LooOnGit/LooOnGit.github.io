---
title: "Memory Segmentation"
date: 2025-07-23 08:12:00 +0800
categories: [C]
tags: [C]
---

# 💡 Phân vùng Bộ nhớ trong C

Tài liệu này cung cấp kiến thức cơ bản, ví dụ minh họa và giải thích rõ ràng về **phân vùng bộ nhớ (memory segmentation)** trong ngôn ngữ lập trình C.

---

## 📚 Mục lục

- [Tổng quan](#Tổng-quan)
- [Các vùng bộ nhớ chính](#các-vùng-bộ-nhớ-chính)
- [Chi tiết từng phân vùng](#chi-tiết-từng-phân-vùng)
- [Ví dụ minh họa](#ví-dụ-minh-họa)

---

## 🧠 Tổng quan

Khi một chương trình C được biên dịch và chạy, bộ nhớ được chia thành nhiều **vùng (segment)** khác nhau. Việc phân vùng bộ nhớ giúp hệ điều hành và chương trình quản lý tài nguyên hiệu quả hơn, đảm bảo an toàn và ổn định.

---

## 🧩 Các vùng bộ nhớ chính

Bộ nhớ trong chương trình C thường được chia thành các vùng sau:

1. **Text Segment (Code Segment)** – Chứa mã máy của chương trình.
2. **Data Segment (initialized data segment)** – Chứa biến toàn cục, biến static được khởi tạo.
3. **BSS Segment (uninitialized data segment)** – Chứa biến toàn cục, biến static chưa được khởi tạo hoặc khởi tạo bằng 0.
4. **Heap  (free srote segment)** – Dành cho cấp phát bộ nhớ động (malloc, calloc, realloc).
5. **Stack (Stack segment)** – Dùng cho biến cục bộ, lời gọi hàm, địa chỉ trả về,…
![alt text](/assets/C/memory_segment.png)
---

## 🔍 Chi tiết từng phân vùng

### 📄 Text Segment
- Chứa mã nguồn đã được biên dịch.
- Thường là vùng **read-only**.
- Ví dụ: nội dung của các hàm `main()`, `printf()`, v.v.

### 🧾 Data Segment
- Biến toàn cục/static có giá trị khởi tạo.
- gồm 2 vùng nhớ là const segment và initialized data segment

**Ví dụ:**
```c
  int global_var = 10;
  static int count = 0;
```
### ❓ BSS Segment
- Biến toàn cục/static chưa khởi tạo.
- Được khởi tạo mặc định là 0.

**Ví dụ:**
```c
int x;
static int y;
```
### 📦 Heap
- Dùng cho cấp phát động: malloc, calloc,…
- Quản lý thủ công (cần free() sau khi dùng).
- Vùng nhớ Heap thì khả răng mở rộng hướng lên nghĩa là cứ cấp phát bộ nhớ động là địa chỉ bộ cấp phát sẽ tăng lên đần.

**Ví dụ:**
```c
int *arr = (int *)malloc(10 * sizeof(int));
```

### 🌀 Stack 
- Dành cho biến cục bộ, lời gọi hàm.
- Quản lý theo cơ chế LIFO (Last In, First Out).
- Tự động thu hồi sau khi thoát khỏi khối lệnh.
- Vùng nhớ Stack sẽ cấp phát theo xu hướng giảm người là cấp phát từ địa chỉ cao đến thấp.

## 🧪 Ví dụ minh họa

```c
#include <stdio.h>
#include <stdlib.h>

int global_var = 100;           // Data segment
static int static_var = 200;    // Data segment
int bss_var;                    // BSS segment

int main() {
    int local_var = 10;         // Stack
    int *heap_var = malloc(4);  // Heap

    printf("Code (text): %p\n", main);
    printf("Data: %p\n", &global_var);
    printf("BSS: %p\n", &bss_var);
    printf("Heap: %p\n", heap_var);
    printf("Stack: %p\n", &local_var);

    free(heap_var);
    return 0;
}
```