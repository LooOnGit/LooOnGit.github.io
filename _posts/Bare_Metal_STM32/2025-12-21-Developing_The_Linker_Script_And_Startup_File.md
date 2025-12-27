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


Chỉ thị AT dùng để chỉ định địa chỉ tải (LMA – Load Memory Address) cho một section khi địa chỉ tải này khác với địa chỉ thực thi (VMA – Virtual Memory Address) của section đó. Chỉ thị này thường được sử dụng cho các section được nạp ở một vùng nhớ khác trong quá trình khởi tạo, sau đó được copy sang vị trí chạy thực tế khi chương trình bắt đầu thực thi.
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
Khi viết linker script, ta thường phải sử dụng tiền tố và hậu tố. Nó rất quan trong trong xác định memory address và size của vùng nhớ.


![Prefix and suffix](/assets/Bare_Metal_STM32/Linker_Script/image2.png)


### Linker script symbols
Linker symbols, hay còn gọi đơn giản là symbols, là những thành phần nền tảng trong quá trình chuyển đổi mã nguồn thành chương trình có thể thực thi.


Symbols bao gồm 2 thành phần: name và value. Các symbol này được gán giá trị số nguyên, đại diện cho địa chỉ bộ nhớ nơi các biến, hàm, hoặc các thành phần khác của chương trình được lưu trữ trong bộ nhớ của vi điều khiển.

Trong ngữ cảnh của linker symbols, giá trị được gán cho một symbol đại diện cho địa chỉ bộ nhớ nơi biến hoặc hàm tương ứng được lưu trữ.


![Linker script symbols](/assets/Bare_Metal_STM32/Linker_Script/image3.png)


Toán tử giống như toán tử trong C. Toán tử bao gồm `=`, `+=`, `-=`, `*=`, `/=`, `%=`, `&=`, `|=`, `^=`, `<<=`, `>>=`.


![Operations](/assets/Bare_Metal_STM32/Linker_Script/image4.png)


Khi sourc main.c được processing thì nó sẽ tạo ra file symbol table.

![Operations](/assets/Bare_Metal_STM32/Linker_Script/image5.png)
## Writing the linker script and startup file
Dưới đây là một chương trình tôi đã giải thích theo mấy lệnh comment.
```c
/*Specifying the firmware's entry point*/
ENTRY(Reset_Handler)

/*Detailing the available memory*/
MEMORY
{
    FLASH(rx):ORIGIN = 0x08000000, LENGTH = 512KB
    SRAM(rwx):ORIGIN = 0x20000000, LENGHT = 128K
}

_estack = ORIGIN(SRAM) + LENGHT(SRAM);

/*Specifying the necessary heap and stack sizes*/
__max_heap_size = 0x200;
__max_heap_size = 0x400;

/*Define output sections*/
SECTIONS
{
    .text :
    {
        . = ALIGN(4);
        *(.isr_vector_tbl)  /*merge all .isr_vector_tbl section of input files*/
        *(.text)            /*merge all .text section of input files*/
        *(.rodata)          /*merge all .rodata section of input files*/
        . = ALIGN(4);
        _etext = .;         /*create a global symbol to hold end of text section*/
    }> FLASH
    .data :
    {
        . = ALIGN(4);
        _sdata = .;         /*Create a global symbol to hold start of data section*/
        *(.data)
        . = ALIGN(4);
        _edata = .;         /*Create a global symbol to hold end of data section*/
    } > SRAM AT> FLASH /*>(vma) AT> (lma)*/
    .bss :
    {
        . = ALIGN(4);
        _sbss = .;
        *(.bss)
        . = ALIGN(4);
        _ebss = .;
    }> SRAM
}
```
### Writing the startup file
Nhiệm vụ của startup file bao gồm:
- **Implementing the vector table**: Defining the vector table để map interrupts to their handlers, đảm bảo hệ thông có thể phản hồi hiệu quả trước các sự kiện khác nhau.
- **Creating interrupt handlers**: Mỗi interrupt listed trong **vector table**, một interrupt handler sẽ được thực thi define respond sự kiện tương ứng.
- **Establishing the firmware's entry point**: Đề cập đến `Reset_Handler`, như được chỉ định trong linker script. Hàm này đóng vai trò là entry point của chương trình, nó thực thi sau khi **reset** và chịu trách nhiệm thiết lập enviroment cho chương trình chính.
- **Transferring the .data section**: Liên quan đến việc copy `.data` section từ **FLASH** sang **SRAM**.
- **Zeroing the .bss section**: Khởi tạo `.bss` section về giá trị 0, đảm bảo rằng các các variable global và static đều bắt đầu với một trạng thái xác định.

### Writing the startup file 
```c
extern uint32_t _estack;
extern uint32_t _etext;
extern uint32_t _edata;
extern uint32_t _sdata;
extern uint32_t _sbss;
extern uint32_t _ebss;

void Reset_Handler(void);
int main(void);
void NMI_Handeler(void)__attribute__((weak, alias("Default_Handler")));
void HardFault_Handler(void)__attribute__((weak, alias("Default_Handler")));
void MemManage_Handler(void)__attribute__((weak, alias("Default_Handler")));

uint32_t vector_tbl[] __attribute__((section(".isr_vector"))) = {
    (uint32_t)&_estack,
    (uint32_t)&Reset_Handler,
    (uint32_t)&NMI_Handeler,
    (uint32_t)&HardFault_Handler,
    (uint32_t)&MemManage_Handler,
    (uint32_t)&BusFault_Handler,
    (uint32_t)&UsageFault_Handler,
    0,
    0,
    0,
    0,
    (uint32_t)&SVC_Handler,
    (uint32_t)&DebugMon_Handler,
    0,
    (uint32_t)&PendSV_Handler,
    (uint32_t)&SysTick_Handler,
    (uint32_t)&WWDG_IRQHandler,              			/* Window Watchdog interrupt                                          */
    (uint32_t)&PVD_IRQHandler,               			/* EXTI Line 16 interrupt / PVD through EXTI                          */
    (uint32_t)&TAMP_STAMP_IRQHandler,        			/* Tamper and TimeStamp interrupts through                            */
    (uint32_t)&RTC_WKUP_IRQHandler,          			/* RTC Wakeup interrupt through the EXTI line                         */
    (uint32_t)&FLASH_IRQHandler,             			/* FLASH global interrupt                                             */
    (uint32_t)&RCC_IRQHandler,               			/* RCC global interrupt                                               */
    (uint32_t)&EXTI0_IRQHandler,             			/* EXTI Line0 interrupt                                               */
    (uint32_t)&EXTI1_IRQHandler,             			/* EXTI Line1 interrupt                                               */
    (uint32_t)&EXTI2_IRQHandler,             			/* EXTI Line2 interrupt                                               */
    (uint32_t)&EXTI3_IRQHandler,             			/* EXTI Line3 interrupt                                               */
    (uint32_t)&EXTI4_IRQHandler,             			/* EXTI Line4 interrupt                                               */
    (uint32_t)&DMA1_Stream0_IRQHandler,      			/* DMA1 Stream0 global interrupt                                      */
    (uint32_t)&DMA1_Stream1_IRQHandler,      			/* DMA1 Stream1 global interrupt                                      */
    (uint32_t)&DMA1_Stream2_IRQHandler,      			/* DMA1 Stream2 global interrupt                                      */
    (uint32_t)&DMA1_Stream3_IRQHandler,      			/* DMA1 Stream3 global interrupt                                      */
    (uint32_t)&DMA1_Stream4_IRQHandler,      			/* DMA1 Stream4 global interrupt                                      */
    (uint32_t)&DMA1_Stream5_IRQHandler,      			/* DMA1 Stream5 global interrupt                                      */
    (uint32_t)&DMA1_Stream6_IRQHandler,      			/* DMA1 Stream6 global interrupt                                      */
    (uint32_t)&ADC_IRQHandler,               			/* ADC1 global interrupt                                              */
    0,                            			/* Reserved                                                           */
    0,                            			/* Reserved                                                           */
    0,                            			/* Reserved                                                           */
    0,                            			/* Reserved                                                           */
    (uint32_t)&EXTI9_5_IRQHandler,           			/* EXTI Line[9:5] interrupts                                          */
    (uint32_t)&TIM1_BRK_TIM9_IRQHandler,     			/* TIM1 Break interrupt and TIM9 global interrupt                     */
    (uint32_t)&TIM1_UP_TIM10_IRQHandler,     			/* TIM1 Update interrupt and TIM10 global interrupt                   */
    (uint32_t)&TIM1_TRG_COM_TIM11_IRQHandler,			/* TIM1 Trigger and Commutation interrupts and TIM11 global interrupt */
    (uint32_t)&TIM1_CC_IRQHandler,           			/* TIM1 Capture Compare interrupt                                     */
    (uint32_t)&TIM2_IRQHandler,              			/* TIM2 global interrupt                                              */
    (uint32_t)&TIM3_IRQHandler,              			/* TIM3 global interrupt                                              */
    (uint32_t)&TIM4_IRQHandler,              			/* TIM4 global interrupt                                              */
    (uint32_t)&I2C1_EV_IRQHandler,           			/* I2C1 event interrupt                                               */
    (uint32_t)&I2C1_ER_IRQHandler,           			/* I2C1 error interrupt                                               */
    (uint32_t)&I2C2_EV_IRQHandler,           			/* I2C2 event interrupt                                               */
    (uint32_t)&I2C2_ER_IRQHandler,           			/* I2C2 error interrupt                                               */
    (uint32_t)&SPI1_IRQHandler,              			/* SPI1 global interrupt                                              */
    (uint32_t)&SPI2_IRQHandler,              			/* SPI2 global interrupt                                              */
    (uint32_t)&USART1_IRQHandler,            			/* USART1 global interrupt                                            */
    (uint32_t)&USART2_IRQHandler,            			/* USART2 global interrupt                                            */
    0,                            			/* Reserved                                                           */
    (uint32_t)&EXTI15_10_IRQHandler,         			/* EXTI Line[15:10] interrupts                                        */
    (uint32_t)&RTC_Alarm_IRQHandler,         			/* RTC Alarms (A and B) through EXTI line interrupt                   */
    (uint32_t)&OTG_FS_WKUP_IRQHandler,       			/* USB On-The-Go FS Wakeup through EXTI line interrupt                */
    0,                            			/* Reserved                                                           */
    0,                            			/* Reserved                                                           */
    0,                            			/* Reserved                                                           */
    0,                            			/* Reserved                                                           */
    (uint32_t)&DMA1_Stream7_IRQHandler,      			/* DMA1 Stream7 global interrupt                                      */
    0,                            			/* Reserved                                                           */
    (uint32_t)&SDIO_IRQHandler,              			/* SDIO global interrupt                                              */
    (uint32_t)&TIM5_IRQHandler,              			/* TIM5 global interrupt                                              */
    (uint32_t)&SPI3_IRQHandler,              			/* SPI3 global interrupt                                              */
    0,                            			/* Reserved                                                           */
    0,                            			/* Reserved                                                           */
    0,                            			/* Reserved                                                           */
    0,                            			/* Reserved                                                           */
    (uint32_t)&DMA2_Stream0_IRQHandler,      			/* DMA2 Stream0 global interrupt                                      */
    (uint32_t)&DMA2_Stream1_IRQHandler,      			/* DMA2 Stream1 global interrupt                                      */
    (uint32_t)&DMA2_Stream2_IRQHandler,      			/* DMA2 Stream2 global interrupt                                      */
    (uint32_t)&DMA2_Stream3_IRQHandler,      			/* DMA2 Stream3 global interrupt                                      */
    (uint32_t)&DMA2_Stream4_IRQHandler,      			/* DMA2 Stream4 global interrupt                                      */
    0,                            			/* Reserved                                                           */
    0,                            			/* Reserved                                                           */
    0,                            			/* Reserved                                                           */
    0,                            			/* Reserved                                                           */
    0,                            			/* Reserved                                                           */
    0,                            			/* Reserved                                                           */
    (uint32_t)&OTG_FS_IRQHandler,            			/* USB On The Go FS global interrupt                                  */
    (uint32_t)&DMA2_Stream5_IRQHandler,      			/* DMA2 Stream5 global interrupt                                      */
    (uint32_t)&DMA2_Stream6_IRQHandler,      			/* DMA2 Stream6 global interrupt                                      */
    (uint32_t)&DMA2_Stream7_IRQHandler,      			/* DMA2 Stream7 global interrupt                                      */
    (uint32_t)&USART6_IRQHandler,            			/* USART6 global interrupt                                            */
    (uint32_t)&I2C3_EV_IRQHandler,           			/* I2C3 event interrupt                                               */
    (uint32_t)&I2C3_ER_IRQHandler,           			/* I2C3 error interrupt                                               */
    0,                            			/* Reserved                                                           */
    0,                            			/* Reserved                                                           */
    0,                            			/* Reserved                                                           */
    0,                            			/* Reserved                                                           */
    0,                            			/* Reserved                                                           */
    0,                            			/* Reserved                                                           */
    0,                            			/* Reserved                                                           */
    (uint32_t)&FPU_IRQHandler,               			/* FPU global interrupt                                               */
    0,                            			/* Reserved                                                           */
    0,                            			/* Reserved                                                           */
    (uint32_t)&SPI4_IRQHandler,              			/* SPI 4 global interrupt                                             */
    (uint32_t)&SPI5_IRQHandler              			/* SPI 5 global interrupt */    
};

/* Default handler that enters an infinite loop */

void Default_Handler(void)
{
	while(1)
	{
		
	}
}


/* Reset Handler */
void Reset_Handler(void)
{
	// Calculate the sizes of the .data and .bss sections
	uint32_t data_mem_size =  (uint32_t)&_edata - (uint32_t)&_sdata;
	uint32_t bss_mem_size  =   (uint32_t)&_ebss - (uint32_t)&_sbss;
    
	// Initialize pointers to the source and destination of the .data section
	uint32_t *p_src_mem =  (uint32_t *)&_etext;
	uint32_t *p_dest_mem = (uint32_t *)&_sdata;
	
	/*Copy .data section from FLASH to SRAM*/
	for(uint32_t i = 0; i < data_mem_size; i++  )
	{
		
		 *p_dest_mem++ = *p_src_mem++;
	}
	
	// Initialize the .bss section to zero in SRAM
	p_dest_mem =  (uint32_t *)&_sbss;
	
	for(uint32_t i = 0; i < bss_mem_size; i++)
	{
		 /*Set bss section to zero*/  
		*p_dest_mem++ = 0;
	}
	
	    // Call the application's main function.

	main();
}
```
#### External symbol declarations
```c
extern uint32_t _estack;
extern uint32_t _etext;
extern uint32_t _edata;
extern uint32_t _sdata;
extern uint32_t _sbss;
extern uint32_t _ebss;
```
Những symbol này được khai báo ở linker script. Mỗi symbol là địa chỉ sử dụng trong startup process.
- `_estack`: Địa chỉ top của stack. Giá trị này load vào main stack pointer register ngay từ sớm ở trong quá trình startup process.
- `_etext`: Đánh dấu phần cuối của executable code section và bắt đầu của data section stored in flash memory.
Chúng ta sử dụng nó như một mốc tham chiếu để sao chép dữ liệu đã được khởi tạo từ FLASH sang SRAM.
- `_sdata` và `_edata`: Đại diện cho địa chỉ bắt đầu và kết thúc của initialized data section trong SRAM. Chúng được sử để determine size và đích đến cho data copy từ FLASH to RAM.
- `_sbss` và `_ebss`: Đánh dấu bắt đầu và kết thúc của uninitialized data section trong SRAM. Chúng sử dụng những symbol để clear section, set nó thành 0.

#### Function prototypes and attributes
```c
/* Function prototypes */
void Reset_Handler(void);
int main(void);

/* Exception and Interrupt Handlers */
void NMI_Handler					(void)__attribute__((weak,alias("Default_Handler")));
void HardFault_Handler 				(void) __attribute__ ((weak, alias("Default_Handler")));
void MemManage_Handler 				(void) __attribute__ ((weak, alias("Default_Handler")));
void BusFault_Handler 				(void) __attribute__ ((weak, alias("Default_Handler")));
void UsageFault_Handler 			(void) __attribute__ ((weak, alias("Default_Handler")));
.....
```
`__attribute__ ((weak, alias("Default_Handler")))` thuộc tính này tạo ra handler **weak** và **alias** tới function có tên là `Default_Handler`. Nó cho phép overridden bởi những handler được define tường minh với tên ở những vị trí khác trong ứng dụng.

- `__attribute__`: Nó thông báo cho compiler rằng khai báo mà nó được áp dụng cho function có những thuộc tính đặc biệt. Ảnh hưởng đến cách compiler xử lý function đó và trong một số trường hợp, lúc runtime. Các **attribute** có thể được sử dụng control optimization, generate code, aligment và liên quan đến discussion - đặc tính liên kết (`linkage characteristic`).
- `weak`: Nó có nghĩa là không ngăn cản linker sử dụng một symbol khác cùng tên nhưng có liên kết mạnh hơn (stronger linkage). Chúng ta dùng `Default_Handler` và có thể overidden. Trong context của interrupt handler, việc đánh dấu chúng là weak cho phép chúng ta định nghĩa các `default handler` trong startup file, các handler đặc thù của ứng dụng có thể override không cần sửa thông qua startup file.
- `alias("Default_Handler")`: Nó tạo ra một alias cho symbol khác, trong case này nó có tên là `Default_Handler`. Nó có nghĩa là symbol (e.g., NMI_Handler) không chỉ là weak, mà còn là một alias trỏ tới hàm `Default_Handler`. Do đó khi một ngắt xảy ra, và một handler cụ thể (chẳng hạn NMI_Handler) không được định nghĩa ở nơi khác trong ứng dụng với linkage (non-weak). Chương trình sẽ sử dụng `Default_Handler`thay thế. Cách làm này đảm bảo các ngắt đều có handler, tránh việc hện thống bị treo hoặc sập do các các sự kiện ngắt xảy ra mà không có handler để xử lý.

#### Vector table definition
```c
/* Vector Table */
uint32_t vector_tbl[] __attribute__((section(".isr_vector"))) = {
    (uint32_t)&_estack,
    (uint32_t)&Reset_Handler,
    (uint32_t)&NMI_Handeler
};
```
Mảng này khác báo interrupt vector table của Microcontroller, đặt ở `.isr_vector_tbl` section được khai báo trong linker script. 


Set `&_estack` symbol như phần tử đầu tiên trong vector table để khai báo địa chỉ top của stack trong memory. Trong ARM Cortex-M microcontroller, first word (32bit) của vector table phải chứa initial value của **main stack pointer (MSP)**. Ngay khi reset xảy ra, xử lý load giá trị này vào MSP register để set up stack pointer chính xác trước khi thực thi bất kì code nào.


Tiếp theo xác định address của `Reset_Handler`, sau đó chúng tôi xử lý list address cho NMI_Handler và các interrupt handlers tiếp theo. Có thể xem vector table trong RM của STM32F411, có những vùng địa chỉ được mô tả là `Reserved` những vùng đặt vậy đều có chủ đích, vector table được thế kế cho ARM Cortex-M thiết kế cho nhiều biến thể MCU khác ở đây mình dùng STM32F411 vì ở MCU này có nhiều interrupt không được hổ trợ trên MCU này nên để reserved. Điều này bắt chúng ta phải tuân theo không thể nào thay đổi vì đây là một architecture của MCU.


![RM](/assets/Bare_Metal_STM32/Linker_Script/image6.png)


`__attribute__((section(".isr_vector")))` cáu này để nói cho linker biết rằng phải đặt mảng `vector_tbl` vào trong section cụ thể được đầu ra có tên file là .isr_vector_tbl.
 
#### Reset handler implementation