---
title: "Operators"
date: 2025-08-13 09:54:24 +0800
categories: [C]
tags: [C]
---
# 🔢 Toán tử trong C

## 📝 Tổng quan

Toán tử trong C được chia thành các nhóm chính:

- Toán tử số học
- Toán tử quan hệ 
- Toán tử logic
- Toán tử bit
- Toán tử gán
- Toán tử điều kiện
- Toán tử đặc biệt

## ➕ Toán tử số học

| Toán tử | Ý nghĩa | Ví dụ |
|---------|---------|--------|
| + | Cộng | a + b |
| - | Trừ | a - b |
| * | Nhân | a * b |
| / | Chia | a / b |
| % | Chia lấy dư | a % b |
| ++ | Tăng 1 đơn vị | a++ hoặc ++a |
| -- | Giảm 1 đơn vị | a-- hoặc --a |

## 🔍 Toán tử quan hệ

| Toán tử | Ý nghĩa | Ví dụ |
|---------|---------|--------|
| == | So sánh bằng | a == b |
| != | So sánh khác | a != b |
| > | Lớn hơn | a > b |
| < | Nhỏ hơn | a < b |
| >= | Lớn hơn hoặc bằng | a >= b |
| <= | Nhỏ hơn hoặc bằng | a <= b |

## 🔄 Toán tử logic

| Toán tử | Ý nghĩa | Ví dụ |
|---------|---------|--------|
| && | AND logic | a && b |
| \|\| | OR logic | a \|\| b |
| ! | NOT logic | !a |

## 🔧 Toán tử bit

| Toán tử | Ý nghĩa | Ví dụ |
|---------|---------|--------|
| & | AND bit | a & b |
| \| | OR bit | a \| b |
| ^ | XOR bit | a ^ b |
| ~ | NOT bit | ~a |
| << | Dịch trái | a << n |
| >> | Dịch phải | a >> n |

## ⚡ Ví dụ thực tế

```c
#include <stdio.h>

int main() {
    int a = 10, b = 3;
    
    // Toán tử số học
    printf("a + b = %d\n", a + b);  // 13
    printf("a - b = %d\n", a - b);  // 7
    printf("a * b = %d\n", a * b);  // 30
    printf("a / b = %d\n", a / b);  // 3
    printf("a %% b = %d\n", a % b); // 1
    
    // Toán tử quan hệ và logic
    printf("a > b là %d\n", a > b);     // 1 (true)
    printf("a < b là %d\n", a < b);     // 0 (false)
    printf("a >= b là %d\n", a >= b);   // 1 (true)
    
    return 0;
}
```

## ⚠️ Lưu ý quan trọng

1. Phân biệt `++a` và `a++`:
   - `++a`: Tăng a trước khi sử dụng
   - `a++`: Sử dụng a trước khi tăng

2. Ưu tiên toán tử:
   - `()` có độ ưu tiên cao nhất
   - Toán tử số học > Toán tử quan hệ > Toán tử logic

3. Phép chia số nguyên:
   - Kết quả luôn là số nguyên
   - Phần thập phân bị cắt bỏ
