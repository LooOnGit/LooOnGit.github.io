---
title: "Build Process And Exploring the GNU Toolchain"
date: 2025-12-16 22:49:03 +0800
categories: [Bare Metal STM32]
tags: [Bare Metal STM32]
---

# 🛠️ Build Process And Exploring the GNU Toolchain

## 🔄 Build Process

Quá trình biên dịch của một chương trình mà IDE thực hiện gồm nhiều giai đoạn (stages):

![Process Stage](/assets/Bare_Metal_STM32/GNU/image.png)

---

## 📋 Các giai đoạn biên dịch

### 1️⃣ The Pre-processing Stage

**Quá trình này sẽ làm:**

- 🗑️ **Stripping comments**: Loại bỏ các comment.
- 🔍 **Evaluating preprocessor directives**: Những dòng bắt đầu bằng `#`, như:
  - Macro define: `#define`
  - Conditional compilation: `#if`, `#ifdef`, `#ifndef`, `#else`, `#endif`
  - Include: `#include`
  
  Bộ tiền xử lý (preprocessor) sẽ thay thế các chỉ thị này bằng những giá trị đã được định nghĩa hoặc các đoạn mã tương ứng.

- 📄 **Output generation**: Đầu ra là file `.i` chứa mã nguồn đã được tiền xử lý.

> [!NOTE]
> **Input**: File `.c` (source code)  
> **Output**: File `.i` (preprocessed code)

---

### 2️⃣ The Compilation Stage

- 📥 **Input**: File `.i` của stage tiền xử lý.
- ⚙️ **Process**: Compiler phân tích cấu trúc source code, tối ưu nó cho hiệu năng và space (dung lượng), và tạo ra mã assembly.
- 📤 **Output**: File `.s` chứa mã assembly.

> [!TIP]
> Compiler sẽ kiểm tra cú pháp và tạo ra mã assembly tối ưu dựa trên kiến trúc CPU target.

---

### 3️⃣ The Assembly Stage

- 📥 **Input**: File `.s` của stage biên dịch.
- 🔄 **Process**: Diễn giải mỗi lệnh của hợp ngữ (assembly) và chuyển nó thành mã máy (machine code).
- 📤 **Output**: File `.o` chứa mã máy, file này chứa binary code.

> [!NOTE]
> File `.o` được gọi là **object file** - chứa mã máy nhưng chưa thể thực thi được vì chưa được liên kết.

---

### 4️⃣ The Linking Stage

- 📥 **Input**: Object files (`.o`) và C standard library files.
- 🔗 **Process**: 
  - Liên kết các file `.o`
  - Giải quyết các tham chiếu ký hiệu (symbolic references) và địa chỉ
  - Đảm nhận công việc như phân bổ bộ nhớ cho các biến và hàm
- 📤 **Output**: Giai đoạn này tạo ra một **relocatable file** (tệp có thể tái định vị)

> [!IMPORTANT]
> File relocatable này đầy đủ nội dung nhưng **chưa phải** là file thực thi cuối cùng!

#### 📌 Giải thích về Relocation

Quá trình **relocation** rất quan trọng:
- Là quá trình điều chỉnh trong file relocatable
- Đảm bảo trong quá trình chạy trên target device, mỗi phần của code và data được đặt đúng vị trí trong bộ nhớ
- Trong quá trình này chỉ link các file `.o` và C standard library files
- Địa chỉ vẫn ở dạng **tương đối**
- Cần điều chỉnh thêm ở giai đoạn **locating** cho phù hợp với vi điều khiển

---

### 5️⃣ The Locating Stage

- 📥 **Input**: 
  - Relocatable file (từ giai đoạn linking)
  - Linker script (`.ld` file)
  
- 🗺️ **Process**: 
  - Sử dụng **linker script** để xác định vị trí của các phần của code và data trong bộ nhớ
  - Điều chỉnh các address và offset để phù hợp với sơ đồ của target's memory map
  
- 📤 **Output**: Một **executable binary file**, thường định dạng như:
  - **ELF** (Executable and Linkable Format)
  - **Binary** (định dạng nhị phân thuần)

> [!IMPORTANT]
> **Linker script** là file cực kỳ quan trọng trong embedded systems, nó xác định:
> - Vị trí của Flash memory (nơi chứa code)
> - Vị trí của RAM (nơi chứa data và stack)
> - Entry point của chương trình
> - Các section như `.text`, `.data`, `.bss`, etc.

## GNU tools

### arm-none-eabi-gcc
- **arm**: target architecture
- **none**: chỉ hệ điều hành mà trình biên dịch hướng tới. Ở đây, none chỉ là thiết kế cho môi trường bare-metal, tức là chương trình chạy trực tiếp trên phần cứng mà không có hệ điều hành bên dưới.
- **eabi**: Embedded Application Binary Interface. EABI là một standard định nghĩa cho binary layout của hệ thống và user programs, library. Nó chắc chắn rằng trình biên dịch hoạt động trên bất kì bộ xử lý arm nào theo tiêu chuẩn EABI.
-**gcc**: Viết tắt của GNU Compiler Collection.

```bash
arm-none-eabi-gcc -c main.c -o main.o
```
Trong đó:
- `arm-none-eabi-gcc`: Command
- `main.c`: source file
- `-o`: compiler flag
- `main.o`: output file

### Some common compiler flags
Có nhiều command:
- `-c`: Flag này sử dụng để compile và assemble nhưng không link. Khi chạy run flag này nó xử lý tới assembly stage nhưng stop trước khi linking.
- `-o` file: Chỉ định output file.
- `-g`: Tạo debug information trong file thực thi. 
- `-Wall`: Cho phép tất cả các warning message, xác định issue trong code.
- `-Werror`: Xem xét tất cả warning và error, đảm bảo chất lượng code và ổn định.
- `-I` [DIR]: Bao gồm một thư mục để tìm kiếm các header file, thường cho tổ chức project lớn.
- `ansi` and `-std=STANDARD`: tùy chọn flag của trình biên dịch dùng để chỉ định phiên bản (chuẩn) của ngôn ngữ C.
- `-v`: flag này cho trình biên dịch hiển thị thông tin chi tiết về quá trình biên dịch.

![Process Stage](/assets/Bare_Metal_STM32/GNU/image1.png)
![Process Stage](/assets/Bare_Metal_STM32/GNU/image2.png)

### Some architecture-specific flags
- `-mcpu=[NAME]`: Chỉ định CPU target trong quá trình biên dịch.
- `-march=[NAME]`: Chỉ định kiến trúc ARM, nó config cho trình biên dịch biết kiến trúc ARM nào để biên dịch.
- `mtune=[NAME]`: Tương tự `mcpu`, nhưng nó chỉ tối ưu hóa cho CPU nhất định.
- `thumb`: Cấu hình compiler để tạo ra code theo tập lệnh Thumb, nó là compressed(nén) version của ARM instruction set. Để tăng code density và nâng cao hiệu quả sử dụng bộ nhớ.
- `marm`: Hướng dẫn cho compiler tạo ra code theo tập lệnh ARM.
- `mlittle-endian/-mbig-endian`: Chỉ định endianness tạo ra code. Little-endian là hầu như là format chung của ARM. 
![Process Stage](/assets/Bare_Metal_STM32/Flash/image4.png)

### Other Commands in the GNU Toolchain for Arm
- `arm-none-eabi-nm`: Liệt kê các symbol trong file object.Trong ngữ cảnh các chương trình khác nhau, chẳng hạn như tên hàm, tên biến và các hằng số. 
- `arm-none-eabi-size`: Cung cấp lượng thông tin chi tiết về dung lượng bộ nhớ mà các phần khác của chương trình chiếm dụng.
- `arm-none-eabi-objdump`: Công cụ này được sử dụng để trích xuất và hiển thị thông tin chi tiết từ các file object. Nó cung cấp cái nhìn sâu về các lệnh máy, khiến nó trở thành một công cụ rất quan trọng cho việc phân tích kỹ lưỡng các file object. Các chức năng của nó bao gồm: disassemble (dịch ngược) mã máy, hiển thị header của các section và trình bày bảng symbol. Công cụ này đặc biệt hữu ích khi chúng ta cần đi sâu vào chi tiết của mã đã biên dịch, giúp làm rõ cấu trúc, nội dung và cơ chế hoạt động của file, từ đó hỗ trợ hiệu quả cho việc debug và tối ưu code.
- `arm-none-eabi-readelf`: Tool này cung cấp thông tin chi tiết về file ELF, bao gồm header, program header và symbol table. Nó tiện ích khi làm việc với ELF.
- `arm-none-eabi-objcopy`: Sử dụng để convert object file từ 1 format sang format khác, thường là convert từ file object sang file binary.
![Process Stage](/assets/Bare_Metal_STM32/GNU/image3.png)



