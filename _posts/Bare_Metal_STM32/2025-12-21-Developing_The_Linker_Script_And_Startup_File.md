---
title: "Developing The Linker Script And Startup File"
date: 2025-12-21 08:00:09 +0800
categories: [Bare Metal STM32]
tags: [Bare Metal STM32]
---

# 🛠️ Developing The Linker Script And Startup File

## Understanding the STM32 Memory model
Trong **STM32** có nhiều loại bộ nhớ khác nhau, để tập trung tìm hiểu linker script và startup file, ta sẽ xem xét bộ nhớ Flash và RAM.


**Memory density**: Thường thế hiện số bit hoặc byte trên mỗi đơn vị của vùng nhớ vật lý. Ví dụ,số bit trên millimeter hoặc số byte trên centimeter.


**Memory size**: Tổng dung lượng của memory. Chẳng hạn như **KB**, **MB** hay **GB**.

## Understanding the Linker processing
Trong quá trình build, bước liên kết (linking) file object là bước rất quan trọng để tạo ta firmware hoàn chỉnh có thể chạy được. Ví dụ 1 value có thể dùng ở nhiều file thì linker này kết hợp lại với nhau. 

### Section attributes and their implications
Mỗi section trong một file object được xác định bởi **unique name** và **size**.
- **Loadable sections**: Chưa nội dung đã được load vào memory ở runtime. Đóng vai trò cần thiết cho các việc thực thi chương trình, bao gồm chương trình thực thi và khai báo data. Ví dụ như **.text** và **.data**.
- **Allocated sections**: Section này thì không chứa nội dung dữ liệu. Thay vào đó, chúng cho biết rằng một vùng nhớ cần được reserved(dành riêng), thường dành cho dữ liệu chưa được khởi tạo và sẽ được xác định trong quá trình runtime. Ví dụ như **.bss**.
- **Non-allocated, Non-loadable sections**: Section này không thuộc 2 loại trên mà chỉ chứa thông tin debug hoặc metadata. Nó không cần thiết trong giai đoạn runtime.


Với các section thì mỗi **section** có 2 loại address là **virtual memory address (VMA)** vaf **load memory address (LMA)**.
- **VMA**: VMA là địa chỉ biểu thị section sẽ nằm ở đâu trong bộ nhớ khi chương trình được thực thi. Đây là địa chỉ runtime mà CPU/hệ thống sử dụng để truy cập dữ liệu hoặc thực thi lệnh của section đó.
- **LMA**: Ngược lại thì LMA là địa chỉ mà section được nạp (load) vật lý vào bộ nhớ.


**Ví dụ**: 
Giả sử hệ thống có:
- Flash: bắt đầu tại 0x08000000
- SRAM: bắt đầu tại 0x20000000

Khai báo biến:
```bash
int counter = 10;
```
Biến counter nằm trong section .data.
| Giai đoạn             | Bộ nhớ | Địa chỉ                |
| --------------------- | ------ | ---------------------- |
| Khi nạp firmware      | Flash  | `0x08004000` (**LMA**) |
| Khi chương trình chạy | SRAM   | `0x20000000` (**VMA**) |
Tại sao 0x08004000 bởi vì có nhiều section khác nhau mỗi section đều chứa ở vùng địa chỉ mà section `.data` nằm ở vùng địa chỉ `0x08004000`.


**Lưu ý**: Trong hầu hết tình huống thì VMA và LMA thì giống nhau. Tuy nhiên, trong một số trường hợp đặc biệt đáng chú ý là khi một section dữ liệu ban đầu được nạp vào bộ nhớ Flash, nhưng sau đó được sao chép sang SRAM khi hệ thống khởi động.

### Key components of the linker script 

![Key components of the linker script](/assets/Bare_Metal_STM32/Linker_Script/image.png)

### Linker script syntax
#### Memory directive (MEMORY)
Chỉ định này định nghĩa *Memory** có những đặc trưng name, start address, end address, size.
##### Usage template**
```bash
MEMORY
{
    <region_name> ( <attributes> ) : ORIGIN = <origin>, LENGTH = <length>
    ...
}
```
- **name**: Một tên định danh mà muốn đặt cho vùng nhớ.
- **attributes**: Đặc tính này cho phép truy cập cho vùng nhớ, như đọc, ghi, và cho phép thực thi.
- **ORIGIN**: Xác định địa chỉ bắt đầu của vùng nhớ.
- **LENGTH**: Chỉ định kích thước của vùng nhớ.

##### Usage example
```bash
MEMORY
{
    FLASH (rx) : ORIGIN = 0x08000000, LENGTH = 256K
    RAM (rwx) : ORIGIN = 0x20000000, LENGTH = 64K
}
``` 
- **FLASH**: Đánh dấu với read (r) và execute (x) permission (rx). Chỉ định rằng vùng nhớ có thể thực thi code nhung không ghi chương trình thực thi. Nó bắt đầu ở `0x08000000` và có kích thước `256K`.
- **RAM**: Đánh dấu với read (r), write (w) và execute (x) permission (rwx), cho phép nó chứa data và code thực thi có thể sửa trong khi runtime chương trình. Nó bắt đầu ở `0x20000000` và có kích thước `64K`.

##### The entry directive (ENTRY)
Chỉ định entry point của chương trình, nó là phần của code để thực thi reset.

**Usage template**
```bash
ENTRY (Symbol)
```
**Usage example**
```bash
ENTRY (Reset_Handler)
```
Trong cái ví dụ này thì `Reset_Handler` thì được thiết kế entry point (điểm khởi đầu) của chương trình.
Quá trình phát triển firmware thì `Reset_Handler` chịu trách nhiệm khởi động hệ và chuyển sang chương trình chính `main`.

#### The section directive (SECTIONS)
Chỉ thị này ánh xạ và sắp xếp các section từ input file vào output file.

**Usage example**
```bash
SECTIONS
{
  .output_section_name address :
  {
    input_section_information
  } > memory_region [AT>load_address] [ALIGN (expression)] [:phdr_expression] [=fill_expression]
}
```
- **output_section_name**: Đây là tên được gán cho section trong output file. Các tên phổ biến bao gồm `.text`, `.data`, `.bss`, `.bss`.
- **address**: Địa chỉ bắt đầu của section trong memory. Điều này thường phần này được để cho linker tự quyết định, dựa vào các section được khai báo trong script.
- **input_section_information**: Phần xác định những đầu vào section (từ object file của compiler) sẽ được đưa vào section đầu ra này. Willcard (kí tự đại diện) chẳng hạn như `*(.text)` có thể sử dụng bao gồm tất cả .text section từ tất cả input file. 
- **>memory_region**: Phần này được giao nhiệm vụ để gán section vào một vùng nhớ cụ thể đã được định nghĩa trong MEMORY block của linker script. Chúng ta để cho linker biết mục tiêu memory map sẽ nằm ở đâu ví dụ  như FLASH hoặc RAM.
- **[AT>load_address]**: Phần này **tùy chọn** và **chỉ định**  load address của section. Nó xử dụng trong trường hợp địa chỉ thực thi khác địa chỉ nạp.
- **[ALIGN (expression)]**: Phần này **tùy chọn** và **căn chỉnh** bắt đầu của section để một địa chỉ để nó là**bội số của giá trị được định nghĩa** bởi `expression`. Điều đặc biệt này hữu ích để đảm bảo rằng section bắt đầu ở địa chỉ để yêu cầu căn chỉnh để nó có thể cải thiện cho **tốc độ truy cập** và **tương thích**.
- **:phdr_expression**: Thành phần **tùy chọn** và **liên kết** section với một program header. Những chương trình header là một phần của structure của **Executable and Linkable Format (ELF)**; Chúng cung cấp cho **system loader** với thông tin về cách nạp và chạy các segment khác nhau của chương trình như thế nào. 
- **=fill_expression**: Phần này **tùy chọn** và dùng để chỉ định một byte value để fill khoảng trống giữa các section hoặc cuối section nhằm đạt được một mức căn chỉnh nhất định. Điều này hữu ích để khai báo vùng memory để biết trạng thái.

**Ví dụ**: 
```bash
SECTIONS
{
  .text 0x08000000 :
  {
    *(.text)
  } > FLASH
}
```
Định nghĩa 1 file output có tên là `.text` và đặt địa chỉ bắt đầu của nó tại `0x08000000`. Tất cả các section có tên `.text` từ tất cả input được đưa vào `.text` output. 
#### Other commonly used directives
**1.The KEEP directive**


Chỉ định **KEEP** đảm bảo rằng section hoặc symbol không bị linker loại bỏ trong quá trình tối ưu, thậm chí có vẻ như không được sử dụng. Điều này đặt biệt quan trọng đối với bảng vector (vector table) và hàm khởi tạo.  Vì những thành phần này bắt buộc phải tồn tại trong file nhị phân cuối cùng.
```bash
KEEP(section)
```
Ví dụ:
```bash
KEEP(*(.isr_vector))
``` 


**2.The >region directive**


Chỉ định **>region** dùng để yêu cầu đặt một section cụ thể vào một vùng nhớ xác định. Các vùng nhớ có thể được khai báo trước trong khối chỉ thị MEMORY của linker script.
```bash
section >region
```
**Ví dụ**:
```bash
.data :
{
  *(.data)
} > RAM
```
Trong ví dụ này đặt `.data` vào trong SRAM memory.


**3.The ALIGN directive**


Chỉ định ALIGN đóng vai trò quan trọng trong linker script, dùng để điều chỉnh location counter sao cho nó được align theo biên bộ nhớ xác định. 

**Location counter** là một built-in variable, địa chỉ hiện tại trong bộ nhớ nơi linker đang đặt các section hoặc các phần của file output trong quá trình linking. Trong linker script, nó được ký hiệu bằng dấu chấm **(.)**.
```bash
. = ALIGN(expression);
```
**Ví dụ:**
```bash
. = ALIGN(4);
```
Trong ví dụ này, địa chỉ hiện tại sẽ được căn chỉnh theo biên 4 byte. `.` là địa chỉ hiện tại mà linker đang đặt dữ liệu code. ALIGN(4) đảm bảo nó sẽ luôn nhảy địa chỉ theo bội số của 4 byte.


**4.The PROVIDE directive**


Chỉ định **PROVIDE** cho phép define symbols để linker sẽ inclue trong file output. Nếu không define thì nó sẽ sử dụng mặc định cho symbols các giá trị này có thể ghi đè (override) bởi những module khác nếu cần.
```bash
PROVIDE (symbol = expression)
```
**Ví dụ**:
```bash
PROVIDE(_stack_end = ORIGIN(RAM) + LENGTH(RAM))
```
Trong ví dụ này, `PROVIDE` định nghĩa một symbol tên là `_stack_end` và gán giá trị là `ORIGIN(RAM) + LENGTH(RAM)`, tức là giá trị của `_stack_end` sẽ là địa chỉ bắt đầu của SRAM + kích thước của SRAM.


**5.AT Directive**


Chỉ thị **AT** dùng để chỉ định LMA cho 1 section khi địa chỉ tải này khác địa chỉ thực thi VMA.
```bash
section AT> lma_region
```
**Ví dụ**:
```bash
.data : AT> FLASH
{
  *(.data)
} >RAM
```
## Understanding constants in linker scripts

## Writing the linker script and startup file

