---
title: "Makefile Advance"
date: 2025-07-27 12:51:00 +0800
categories: [Linux]
tags: [Linux]
---

# 🐧 Linux_Learn

## 📚 Agenda

### I. 📝 Giới thiệu về Makefile

#### 1.1 Khái niệm
- Makefile là một tập tin script chứa các chỉ thị (rules) để build project
- Được sử dụng để tự động hóa quá trình biên dịch và liên kết các file trong project
- Chương trình `make` sẽ đọc nội dung trong Makefile và thực thi theo các rule được định nghĩa

#### 1.2 Cấu trúc của Makefile

```
====== Cấu trúc của một Rule trong Makefile ======

hellomake: hello.c main.c             # Rule header
|         |                           # 
|         +-> Dependencies            # Các file cần có
|             (file phụ thuộc)        # để tạo target
|
+-> Target (tên file đích)

    gcc -o hellomake hello.c main.c   -I .# Command line
    ^   ^          ^                  # 
    |   |          |                  #
    |   |          +-> Source files   # File nguồnFixed 
    |   |
    |   +-> Output flag              # Flag output
    |
    +-> Compiler command            # Lệnh biên dịch
    (phải có Tab ở đầu dòng)
```

Một rule trong Makefile bao gồm 4 phần chính:

1. **Target** 🎯:
   - Tên file cần tạo ra (ví dụ: `hellomake`)
   - Hoặc tên hành động cần thực hiện (ví dụ: `clean`, `all`)

2. **Dependencies** 🔗:
   - Các file cần có để tạo ra target
   - Trong ví dụ: `hello.c` và `main.c`
   - Nếu các file này thay đổi, target sẽ được build lại

3. **Tab** ➡️:
   - Bắt buộc phải có dấu tab ở đầu dòng lệnh
   - Không thể thay thế bằng dấu cách
   - Đây là quy tắc cú pháp bắt buộc của Makefile

4. **Action** ⚡:
   - Lệnh thực thi để tạo ra target
   - Ví dụ: `gcc -o hellomake hello.c main.c`
   - Có thể có nhiều lệnh, mỗi lệnh phải bắt đầu bằng tab

Cú pháp tổng quát:
```makefile
target: dependencies
[tab]  commands
```

#### 1.3 Các thành phần cơ bản
- **Biến (Variables)**: `CC = gcc`, `CFLAGS = -Wall`
- **Comments**: bắt đầu bằng dấu #
- **Phony Targets**: `.PHONY: clean all`
- **Pattern Rules**: `%.o: %.c`

#### 1.4 Ví dụ đơn giản
```makefile
CC = gcc
CFLAGS = -Wall

program: main.o utils.o
    $(CC) $(CFLAGS) -o program main.o utils.o

main.o: main.c
    $(CC) $(CFLAGS) -c main.c

utils.o: utils.c
    $(CC) $(CFLAGS) -c utils.c

clean:
    rm -f *.o program
```

#### 1.3 Cấu trúc của Makefile


### II. 🔄 Quá trình biên dịch một chương trình C
- Các bước tiền xử lý (Preprocessing)
- Biên dịch (Compilation)
- Assembling
- Linking
- Demo với ví dụ cụ thể

### III. 📚 Static Lib và Shared Lib

#### 3.1 Khái niệm cơ bản 📖

- **Thư viện** là một tập hợp các đoạn mã được biên dịch sẵn để có thể được sử dụng lại trong một chương trình.
- Quá trình tạo thư viện:
  ```
  Source Files → Object Files → Library → Executable Program
  ```

#### 3.2 Phân loại 🔄

Được chia ra làm 2 loại chính:

1. **Static Library (.a)**
   - Được link trực tiếp vào chương trình khi biên dịch
   - File thư viện có đuôi `.a` (archive)
   - Làm tăng kích thước file thực thi
   - Không phụ thuộc vào thư viện bên ngoài khi chạy

2. **Shared Library (.so)**
   - Được load động khi chương trình chạy
   - File thư viện có đuôi `.so` (shared object)
   - Tiết kiệm bộ nhớ vì nhiều chương trình có thể dùng chung
   - Cần có thư viện trong hệ thống khi chạy chương trình

#### 3.3 Quy trình tạo và sử dụng thư viện 🛠️

```
┌─────────────┐     ┌─────────────┐     ┌──────────┐     ┌──────────────┐
│ Source File │ ──▶ │ Object File │ ──▶ │ Runtime  │ ──▶ │  Executable  │
│    (.c)     │     │    (.o)     │     │ Library  │     │   Program    │
└─────────────┘     └─────────────┘     └──────────┘     └──────────────┘
                           ▲                   ▲
                           │                   │
                    ┌─────────────┐    ┌──────────┐
                    │   Linker    │ ◀─ │  Static/ │
                    │             │    │  Shared  │
                    └─────────────┘    │   Libs   │
                                      └──────────┘
```

#### 3.4 So sánh Static Library và Shared Library 📊

| Thuộc tính | Static Library | Shared Library |
|------------|---------------|----------------|
| **Linking time** | - Tất cả các modules trong thư viện sẽ được copy vào trong file thực thi tại thời điểm biên dịch (compile time)<br>- Khi chương trình được load vào bộ nhớ, OS chỉ đặt một file thực thi duy nhất bao gồm source code và thư viện được link (Static linking) | - Được sử dụng trong quá trình link khi mà cả file thực thi và file thư viện đều được load vào bộ nhớ (runtime)<br>- Một shared lib có thể được nhiều chương trình sử dụng (Dynamic linking) |
| **Size** | Sử dụng static lib tốn nhiều bộ nhớ hơn shared lib | Sử dụng shared lib tốn ít bộ nhớ hơn vì chỉ có duy nhất một bản sao trong bộ nhớ |
| **External File changes** | File thực thi cần phải recompile lại bất cứ khi nào có sự thay đổi trong static lib | Đối với shared lib, không cần phải biên dịch lại file thực thi |
| **Time** | Mất nhiều thời gian hơn để thực thi | Sử dụng shared lib tốn ít thời gian hơn để thực thi vì thư viện nằm sẵn trong bộ nhớ |

**Tóm tắt:**
- **Static Library (.a)**:
  - ✅ Độc lập, không phụ thuộc thư viện ngoài
  - ✅ Hiệu suất tốt khi chạy
  - ❌ File lớn, tốn bộ nhớ
  - ❌ Cần biên dịch lại khi lib thay đổi

- **Shared Library (.so)**:
  - ✅ Tiết kiệm bộ nhớ, nhiều chương trình dùng chung
  - ✅ Cập nhật lib không cần biên dịch lại
  - ❌ Phụ thuộc vào lib trong hệ thống
  - ❌ Có thể gặp vấn đề về version

## 🛠 Công cụ cần chuẩn bị
- GCC Compiler
- Make utility
- Text editor (VSCode, Vim, etc.)
- Git

## 📖 Tài liệu tham khảo
- GNU Make Manual
- GCC Documentation
- Linux Programming Interface

