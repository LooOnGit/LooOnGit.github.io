---
title: "Introduce C++"
date: 2025-11-02 06:57:05 +0800
categories: [C++]
tags: [C++]
---

# 📖 Giới thiệu về C++

Ra đời vào năm 1972, C là ngôn ngữ lập trình mệnh lệnh được Dennis Ritchie phát triển. Ngôn ngữ lập trình C được phát triển để dùng trong hệ điều hành UNIX. Chính vì vậy, ngôn ngữ lập trình C đã lan rộng ra nhiều hệ điều hành khác và trở thành một trong những ngôn ngữ lập trình phổ biến nhất thế giới.

Ngoài ra, C là ngôn ngữ lập trình bậc trung, nó được tạo ra để có thể thuận tiện viết các chương trình lớn mà không có nhiều lỗi và không đặt gánh nặng cho người viết trình dịch C.

C++ là một ngôn ngữ lập trình máy tính có tính năng của ngôn ngữ lập trình C cũng như Simula 67 (một ngôn ngữ hướng đối tượng đầu tiên). Kế thừa và phát triển C, C++ đã giới thiệu khái niệm Class và Objects.

C++ đóng gói các tính năng ngôn ngữ cấp cao và cấp thấp. Vì vậy, nó được xem như một ngôn ngữ cấp độ trung gian. Trước đó ngôn ngữ lập trình C++ được gọi là “C with classes” vì C++ có tất cả các thuộc tính của ngôn ngữ C.

# Objects and variables
Một đối tượng được có tên thì gọi là biến (variable). Việc đặt tên các object để chúng ta tham chiếu đến object đó lần nữa trong chương trình.

# Different forms of initialization
Có 5 dạng khởi tạo (initialization) phổ biến trong C++:
```c
int a;         // default-initialization (khởi tạo mặc định - không có giá trị gán)

// Các dạng khởi tạo truyền thống (Traditional initialization forms):
int b = 5;     // copy-initialization (khởi tạo sao chép - giá trị gán nằm sau dấu bằng)
int c ( 6 );   // direct-initialization (khởi tạo trực tiếp - giá trị gán nằm trong ngoặc đơn)

// Các dạng khởi tạo hiện đại (được khuyên dùng - Modern initialization forms):
int d { 7 };   // direct-list-initialization (khởi tạo danh sách trực tiếp - giá trị gán nằm trong ngoặc nhọn)
int e {};      // value-initialization (khởi tạo giá trị - ngoặc nhọn rỗng, thường tự gán về 0)
```

# Quiz time
## What is data?
Data là bất cứ thông tin có thể di chuyển, xử lý, hoặc lưu trữ bằng computer.
## What is value?
Giá trị là một chữ số, ký tự, hoặc một chuỗi ký tự.
## What is object?
Object là một region trong bộ nhớ (thường là memory) để có thể lưu trữ data.
## What is an indentifier?
Indentifier là một tên dùng để truy cập variable.
## What is a data type used for? 
Data type dùng để xác định value (e.g a number, a character, a string, etc.).
## What is a integer?
Integer là một số có thể được viếc mà không cần dấu chấm thập phân.

## When should I initialize with { 0 } vs {}? 
khi khởi tạo direct-list-initialization thì nên dùng { 0 } thay vì {} để tránh lỗi. Ví dụ:
```c
int a { 0 }; // OK
```
Sử dụng value-initialization {} khi bạn muốn khởi tạo biến với giá trị mặc định (thường là 0 cho số, false cho bool, nullptr cho con trỏ).
```c
int x {};      // value initialization
std::cin >> x; // we're immediately replacing that value so an explicit 0 would be meaningless
```

## What is the difference between initialization and assignment?
Initialization cung cấp cho biến một giá trị ban đầu tại thời điểm tạo ra nó. Assignment cung cấp một variable một giá trị tại thời điểm sau khi nó đã được tạo ra.
```c
int a = 5; // initialization
a = 6;   // assignment
```

## What form of initialization should you prefer when you want to initialize a variable with a specific value?
Direct-list-initialization (aka. direct brace initialization).

## What are default-initialization and value-initialization? What is the behavior of each? Which should you prefer?
- Default-initialization xảy ra khi biến được khai báo không có giá trị khởi tạo (ví dụ: int x;). Trong hầu hết các trường hợp, biến sẽ được để lại với một giá trị không xác định (indeterminate value). 
- Value-initialization xảy ra khi biến được khởi tạo bằng cặp ngoặc nhọn rỗng (ví dụ: int x{};). Trong hầu hết các trường hợp, điều này sẽ thực hiện khởi tạo về 0 (zero-initialization).
- Bạn nên ưu tiên sử dụng value-initialization, vì nó khởi tạo biến về một giá trị nhất quán. 