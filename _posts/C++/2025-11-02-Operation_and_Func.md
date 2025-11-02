---
title: "Openration And Function"
date: 2025-11-02 07:14:05 +0800
categories: [C++]
tags: [C++]
---

# Operators and Common Functions
## 📊 Variable Types in C++

Ngoài các kiểu dữ liệu bạn đã học như `short`, `int`, `long`, `double`, `bool`, hôm nay chúng ta sẽ tìm hiểu về các kiểu dữ liệu khác trong C++.

| 🏷️ Kiểu (Type) | 📦 Kích thước (Size) | 🎯 Phạm vi (Range) | 💻 Ví dụ (Example) |
|:---|:---|:---|:---|
| `bool` | 1 byte | `0` đến `1` (tương ứng `false` và `true`) | `bool is_active = true;` |
| `char` | 1 byte | `-128` đến `127` | `char initial = 'A';` |
| `unsigned char` | 1 byte | `0` đến `255` | `unsigned char ascii = 65;` |
| `int` | 4 bytes | `-2,147,483,648` đến `2,147,483,647` | `int score = 100;` |
| `unsigned int` | 4 bytes | `0` đến `4,294,967,295` | `unsigned int user_id = 12345;`|
| `short` | 2 bytes | `-32,768` đến `32,767` | `short year = 2024;` |
| `unsigned short` | 2 bytes | `0` đến `65,535` | `unsigned short port = 8080;` |
| `long long` | 8 bytes | `-(2⁶³)` đến `(2⁶³)-1` | `long long population = 8e9;` |
| `unsigned long long` | 8 bytes | `0` đến `18,446,744,073,709,551,615`| `unsigned long long atoms = 1e19;`|
| `float` | 4 bytes | *Chưa xác định* (khoảng 7 chữ số thập phân)| `float pi = 3.14f;` |
| `double` | 8 bytes | *Chưa xác định* (khoảng 15 chữ số thập phân)| `double e = 2.71828;` |

## Data Formats
| 🎯 Định dạng (Specifier) | 📦 Kiểu dữ liệu (Data Type) | 📝 Ví dụ (Example) | 💡 Công dụng |
|:---|:---|:---|:---|
| `%c` | `char` | `"%c"` | In một ký tự đơn. |
| `%s` | `char *` (chuỗi ký tự) | `"%8s"`, `"%-10s"` | In một chuỗi. Có thể định độ rộng, canh lề. |
| `%d`, `%i` | `int`, `short` | `"%-2d"`, `"%03d"` | In số nguyên hệ thập phân (có dấu). |
| `%f` | `float` | `"%5.2f"` | In số thực (dấu phẩy động). |
| `%lf` | `double` | `"%8.3lf"` | In số thực `double`. |
| `%ld` | `long` | `"%-10ld"` | In số nguyên `long`. |
| `%u` | `unsigned int`, `unsigned short` | `"%2u"`, `"%02u"` | In số nguyên hệ thập phân (không dấu). |
| `%o` | `int`, `short`, `unsigned` | `"%06o"`, `"%03o"` | In số nguyên hệ bát phân (octal). |
| `%x`, `%X` | `int`, `short`, `unsigned` | `"%04x"` | In số nguyên hệ thập lục phân (hexadecimal). `%x` chữ thường, `%X` chữ hoa. |
| `%e`, `%E` | `float` | `"%5.3e"` | In số thực dưới dạng khoa học (vd: 1.23e+02). |
| `%g`, `%G` | `float` | `"%g"` | Tự động chọn `%f` hoặc `%e` cho gọn nhất. |
| `%lu` | `unsigned long` | `"%10lu"` | In số nguyên `unsigned long`. |
| `%lo` | `long`, `unsigned long` | `"%12lo"` | In số nguyên `long` hệ bát phân. |
| `%lx`, `%lX` | `long`, `unsigned long` | `"%08lx"` | In số nguyên `long` hệ thập lục phân. |
| `%%` | (Không có) | `"%%"` | In ra ký tự `%`. |

## Cách dùng toán tử
```c
//Dung sai: bool : true(1) : false(0)
bool check = 10 > 1; //gan gia tri cua phép so sánh (10>1) cho bien check
```

## Thư viện iomanip

Trong C++, thư viện `iomanip` cung cấp các hàm "manipulator" cho các lệnh đầu ra (cout) và đầu vào (cin).

## Thư viện bits/stdc++.h

Nó sẽ gọi hết các thư viện chuẩn của C++, bao gồm cả các thư viện của C (ở C++ là <c...>).