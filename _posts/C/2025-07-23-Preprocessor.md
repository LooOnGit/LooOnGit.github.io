---
title: "Preprocessor"
date: 2025-07-23 09:56:00 +0800
categories: [C]
tags: [C]
---
# Tiền xử lý (Preprocessor) trong C

## 🧠 Tổng quan

Tiền xử lý (Preprocessor) trong C là một bước thực hiện trước khi biên dịch mã nguồn. Tất cả các chỉ thị tiền xử lý đều bắt đầu với ký tự `#`. Các chỉ thị này không phải là lệnh C/C++ nên không cần dấu `;` khi kết thúc.

---

## 🔧 Các nhóm chỉ thị tiền xử lý

### 1. Chỉ thị bao hàm tệp (`#include`)

`#include` cho phép thêm nội dung của một file khác vào file đang viết. Chúng ta đặc biệt sử dụng `#include` để thêm các file `.h` từ thư viện chuẩn (như `stdio.h`, `stdlib.h`, `string.h`, `math.h`) hoặc các file `.h` do bạn tự định nghĩa.

---

#### Cách sử dụng:

1. **Thêm file từ thư viện hệ thống**:
   Sử dụng dấu ngoặc nhọn `< >` để thêm các file `.h` có sẵn trong thư mục cài đặt IDE của bạn.
   ```c
   #include <stdlib.h>
   ```
2. **Thêm file từ thư mục dự án**: 
   Sử dụng dấu ngoặc kép " " để thêm các file .h trong thư mục chứa project của bạn. Nếu không tìm thấy file trong thư mục dự án, bộ tiền xử lý sẽ tiếp tục tìm trong thư viện hệ thống.
   ```c
   #include "myheader.h"
   ```
   **Cách hoạt động**:
   Tất cả nội dung của file .h sẽ được chèn vào file .c tại vị trí của chỉ thị #include.
### Ví dụ 2:

Dưới đây là những gì ta có trong file `.c`:

```c
#include "file.h"

long myFunction(int x, double y) {
    /* Source code of function */
}

void otherFunction(long value) {
    /* Source code of function */
}
```
Và những gì có trong file .h:
```c
long myFunction(int x, double y);
void otherFunction(long value);
```
**Tiền xử lý**:
Khi sử dụng #include "file.h", bộ tiền xử lý sẽ chèn toàn bộ nội dung của file file.h vào file .c tại vị trí của chỉ thị #include. Kết quả sau khi tiền xử lý sẽ như sau:
```c
long myFunction(int x, double y);
void otherFunction(long value);

long myFunction(int x, double y) {
    /* Source code of function */
}

void otherFunction(long value) {
    /* Source code of function */
}
```
### Ví dụ 3: Sử dụng hàm `static`

#### Mô tả:
Hàm `static` trong C được sử dụng để giới hạn phạm vi sử dụng của hàm chỉ trong file `.c` nơi nó được định nghĩa. Điều này giúp tránh xung đột tên hàm khi làm việc với nhiều file trong một dự án.

#### Cách thực hiện:

1. **Tạo file `lib.h`**:
   ```c
   #include <stdio.h>
   void in_so();
   ```

2. **Tạo file `lib.c`**:
   ```c
   #include "lib.h"

   static void so(int num) {
       num = num + 1;
   }

   void in_so() {
       int num = 0;
       so(num);
       printf("Số sau khi tăng: %d\n", num);
   }
   ```

   - Hàm `static void so()` chỉ có thể được sử dụng trong file `lib.c`. Nó không thể được gọi từ các file khác.

3. **Tạo file `main.c`**:
   ```c
   #include "lib.h"

   int main() {
       in_so();
       return 0;
   }
   ```

#### Kết quả:
Khi chạy chương trình, bạn sẽ nhận được kết quả in ra từ hàm `in_so()`.

#### Lưu ý:
- Hàm `static` giúp bảo vệ các hàm nội bộ, tránh việc chúng bị sử dụng ngoài ý muốn từ các file khác.
- Đây là một cách tốt để tổ chức mã nguồn và giảm thiểu xung đột tên trong các dự án lớn.

### 2. Chỉ thị định nghĩa cho tên (`#define`)

#### Mô tả:
Chỉ thị `#define` trong C được sử dụng để định nghĩa một macro, tức là một tên thay thế cho một giá trị hoặc đoạn mã. Macro không chiếm bộ nhớ và được thay thế trực tiếp trong mã nguồn trong quá trình tiền xử lý.

#### Các loại macro:

1. **Macro giống đối tượng**:
   - Macro này hoạt động như một hằng số, thay thế tên macro bằng giá trị được định nghĩa.
   ```c
   #define PI 3.14159
   #define SLOGAN "Học mà chơi - Chơi mà học"
   printf("Giá trị của PI: %f\n", PI);
   printf("%s\n", SLOGAN);
   ```

   - Macro cũng có thể được định nghĩa mà không có giá trị thay thế. Khi xuất hiện trong mã nguồn, nó sẽ bị xóa và không được thay thế bởi bất kỳ giá trị nào.
   ```c
   #define USE_YEN
   #ifdef USE_YEN
   printf("Sử dụng đồng Yên.\n");
   #endif
   ```

   - **Lưu ý**: Macro không có giá trị thay thế thường được sử dụng để kiểm tra điều kiện trong quá trình tiền xử lý.

2. **Macro giống hàm**:
   - Macro này hoạt động giống như một hàm inline, có thể nhận tham số.
   ```c
   #define SQUARE(x) ((x) * (x))
   printf("Bình phương của 5: %d\n", SQUARE(5));
   ```

3. **Xóa định nghĩa macro**:
   Sử dụng `#undef` để xóa một macro đã được định nghĩa.
   ```c
   #define TEMP 100
   #undef TEMP
   ```

#### Ví dụ:

1. **Định nghĩa hằng số**:
   ```c
   #define MAX 100

   int arr[MAX];
   printf("Kích thước mảng: %d\n", MAX);
   ```

2. **Macro với tham số**:
   ```c
   #define MULTIPLY(a, b) ((a) * (b))

   int result = MULTIPLY(3, 4);
   printf("Kết quả: %d\n", result);
   ```

3. **Sử dụng macro để kiểm tra nền tảng**:
   ```c
   #define WINDOWS

   #ifdef WINDOWS
   printf("Chương trình đang chạy trên Windows.\n");
   #endif
   ```

#### Lưu ý:
- Macro không kiểm tra kiểu dữ liệu, vì vậy cần cẩn thận khi sử dụng để tránh lỗi không mong muốn.
- Sử dụng dấu ngoặc trong macro có tham số để đảm bảo tính đúng đắn của biểu thức.
- Macro giúp tăng hiệu suất nhưng có thể làm mã nguồn khó đọc nếu lạm dụng.

4. **So sánh `#define` và `typedef`**:
   - `#define` chỉ đơn giản là thay thế tên bằng một giá trị hoặc đoạn mã khác trong quá trình tiền xử lý.
   - `typedef` được sử dụng để định nghĩa một kiểu dữ liệu mới, giúp mã nguồn dễ đọc và dễ bảo trì hơn.
   ```c
   #define u8 char
   typedef char u8;
   ```
   - **Khác biệt chính**:
     - `#define` không kiểm tra kiểu dữ liệu, chỉ thực hiện thay thế văn bản.
     - `typedef` được xử lý bởi trình biên dịch và cung cấp kiểm tra kiểu dữ liệu.
### 5. Sử dụng `#undef` để hủy bỏ định nghĩa macro

#### Mô tả:
`#undef` được sử dụng để xóa một macro đã được định nghĩa trước đó. Điều này đặc biệt hữu ích khi bạn cần định nghĩa lại một macro với giá trị mới hoặc muốn đảm bảo rằng macro không còn tồn tại trong phạm vi tiếp theo của mã nguồn.

#### Ví dụ:
```c
// Định nghĩa macro DEVIOT
#define LOO "Hello Loo"

// Hủy bỏ định nghĩa macro DEVIOT
#undef LOO

// Định nghĩa lại macro DEVIOT với giá trị mới
#define LOO "Welcome to Loo"

printf("%s\n", L); // Kết quả: Welcome to DEVIOT
```

#### Lưu ý:
- **Phạm vi của macro**: Khi bộ tiền xử lý kết thúc, tất cả các macro được định nghĩa trong file sẽ bị loại bỏ. Điều này có nghĩa là macro chỉ có hiệu lực từ điểm định nghĩa đến cuối file.
- **Sử dụng trong nhiều file**: Nếu làm việc với nhiều file, nên sử dụng `#undef` để tránh xung đột định nghĩa macro giữa các file.
- **Định nghĩa trong file `.h`**: Khi định nghĩa macro trong file `.h`, các macro này sẽ được áp dụng cho tất cả các file `.c` mà file `.h` được `#include`.

#### Ứng dụng thực tế:
Khi làm việc với các dự án lớn, việc sử dụng `#undef` giúp tránh xung đột định nghĩa macro giữa các file khác nhau, đặc biệt khi các file có thể sử dụng cùng một tên macro nhưng với giá trị khác nhau.
