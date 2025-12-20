---
title: "Compiler And Flash STM32"
date: 2025-12-20 14:45:34 +0800
categories: [Bare Metal STM32]
tags: [Bare Metal STM32]
---

# Compiler And Flash STM32
## Compiler
Tải GCC cài đặt ở đây
[https://developer.arm.com/downloads/-/gnu-rm](https://developer.arm.com/downloads/-/gnu-rm)

Tải bản `arm-gnu-toolchain-15.2.rel1-mingw-w64-i686-arm-none-eabi.msi` là bản hiện tại đang dùng có thể sẽ được update bản mới.


Sau khi cài đặt xong, thêm vào môi trường path ở `Edit the system environment variables`.
![Edit the system environment variables](/assets/Bare_Metal_STM32/Flash/image.png)
![Edit the system environment variables](/assets/Bare_Metal_STM32/Flash/image2.png)
Thêm đường dẫn vào trong này và đường dẫn sẽ là nới chứa `arm-none-eabi-gcc.exe`. Là trình biên dịch cho code bare metal.
![Edit the system environment variables](/assets/Bare_Metal_STM32/Flash/image3.png)

Sau đó chỉ cần ở **cmd** chạy lên 
```bash
PS C:\Users\tranc> arm-none-eabi-gcc --version
arm-none-eabi-gcc.exe (Arm GNU Toolchain 15.2.Rel1 (Build arm-15.86)) 15.2.1 20251203
Copyright (C) 2025 Free Software Foundation, Inc.
This is free software; see the source for copying conditions.  There is NO
warranty; not even for MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.
```
Đã thành công set up môi trường biến dịch. 


**Lưu ý:** Chỉ chạy trong **Command Prompt** chỉ thấy chạy được trên này.

## Flash

Ở flash có thể dùng **STM32CubeProgrammer** hoặc **ST-Link Utility** để dùng đến `STM32_Programmer_CLI` hoặc `ST-LINK_CLI`. Để nạp chương trình cho board.


Download **STM32CubeProgrammer** ở đây
[https://www.st.com/en/development-tools/stm32cubeprog.html](https://www.st.com/en/development-tools/stm32cubeprog.html)

Download **ST-Link Utility** ở đây
[https://www.st.com/en/development-tools/stsw-link009.html](https://www.st.com/en/development-tools/stsw-link009.html)


**Ví dụ:** 
Ở đây đang dùng là **STM32CubeProgrammer** thì tìm đến đường dẫn bin và set up môi trường như **gcc** ở trên.
```bash
PS C:\Users\tranc> STM32_Programmer_CLI --version
      -------------------------------------------------------------------
                       STM32CubeProgrammer v2.21.0
      -------------------------------------------------------------------

STM32CubeProgrammer version: 2.21.0
```
Chạy kiếm tra setup thành công rồi.


Muốn nạp code vào trong mcu chạy
```bash
PS C:\Users\tranc> STM32_Programmer_CLI --help
      -------------------------------------------------------------------
                       STM32CubeProgrammer v2.21.0
      -------------------------------------------------------------------


Usage :
STM32_Programmer_CLI.exe [command_1] [Arguments_1][[command_2] [Arguments_2]...]

Generic commands:

 -?, -h, --help         : Show this help
 -c,     --connect      : Establish connection to the device
     <port=<PortName>   : Interface identifier. ex COM1, /dev/ttyS0, usb1,
                          JTAG, SWD, JLINK...)
    USB port optional parameters:
     [sn=<serialNumber>]   : Serial number of the usb dfu
     [PID=<Product ID>]    : Product ID. ex: 0xA38F, etc, default 0xDF11
     [VID=<Vendor ID>]     : Vendor ID. ex: 0x0389, etc, default x0483
    UART port optional parameters:
     [br=<baudrate>]    : Baudrate. ex: 115200, 9600, etc, default 115200
     [P=<parity>]       : Parity bit, value in {NONE,ODD,EVEN}, default EVEN
     [db=<data_bits>]   : Data bit, value in {6, 7, 8} ..., default 8
```
chạy xong `--help` sẽ thấy các flag có thể dùng.

## Chương trình ví dụ

Có chương trình mẫu đây `main.c`.
```c
#include <stdint.h>

#define PERIPH_BASE 		(0x40000000UL)							//1: Define base address for peripherals
#define AHB1PERIPH_OFFSET 	(0x00020000UL)							//2. Offset for AHB1 peripheral bus
#define AHB1PERIPH_BASE 	(PERIPH_BASE + AHB1PERIPH_OFFSET) 		//3. Base address for AHB1 peripherals

#define GPIOD_OFFSET 		(0x00000C00UL)							//4. Offset for GPIOD
#define GPIOD_BASE			(AHB1PERIPH_BASE + GPIOD_OFFSET)		//5. Base address for GPIOD

#define RCC_OFFSET			(0x00003800UL)							//6. Offset for RCC
#define RCC_BASE			(AHB1PERIPH_BASE + RCC_OFFSET)			//7. Base address for RCC

#define AHB1EN_R_OFFSET		(0x30UL) 								//8. Offset for AHB1EN register
#define RCC_AHB1EN_R		(*(volatile unsigned int *)(RCC_BASE + AHB1EN_R_OFFSET)) //9. Address off AHB1EN Register

#define MODER_OFFSET		(0x00UL)								//10. Offset for mode register
#define GPIOD_MODER			(*(volatile unsigned int *)(GPIOD_BASE + MODER_OFFSET)) //11. Address of GPIOD mode register
#define ODR_OFFSET			(0x14UL)								//12. Offset for output data register
#define GPIO_ODR			(*(volatile unsigned int *)(GPIOD_BASE + ODR_OFFSET))	//13. Address of GPIOD output data register
#define GPIOD_EN			(1U << 3)								//14. Bit mask for enabling GPIOD (bit 0)
#define PIN_12				(1U << 12)								//15. Bit mask for GPIOD pin 12
#define LED_1				PIN_12									//16. Alias for PIN12 representing LED pin


int main(void)
{
    RCC_AHB1EN_R |= GPIOD_EN;			//18.Enable clock access to GPIOD

    GPIOD_MODER &= ~(3U << (12 * 2));	//19. Clear bit 24 and 25
    GPIOD_MODER |=  (1U << (12 * 2));	//20. Set bit 24 to 1
    while (1)
    {
        GPIO_ODR |= LED_1;   // PD12 ON
    }
}
```
`Makefile` lưu ý rằng chữ M đầu phải viết hoa.
```makefile
CFLAGS = -mcpu=cortex-m4 -mthumb -mfpu=fpv4-sp-d16 -mfloat-abi=hard -std=gnu11
LDFLAGS = -mcpu=cortex-m4 -mthumb -mfpu=fpv4-sp-d16 -mfloat-abi=hard -specs=nano.specs -specs=nosys.specs
BUILD_DIR = build


All:
	arm-none-eabi-gcc -c main.c $(CFLAGS) -o build/main.o
	arm-none-eabi-gcc -c -x assembler-with-cpp startup_stm32f411vetx.s $(CFLAGS) -o build/startup.o
	arm-none-eabi-gcc build/main.o build/startup.o $(CFLAGS) -T"STM32F411VETX_FLASH.ld" -Wl,-Map="file.map" -Wl,--gc-sections -static -o build/bare_metal.elf
	arm-none-eabi-objcopy -O ihex build/bare_metal.elf build/bare_metal.hex
	arm-none-eabi-objcopy -O binary build/bare_metal.elf build/bare_metal.bin

$(BUILD_DIR):
	if not exist $(BUILD_DIR) mkdir $(BUILD_DIR)

Clean:
	if exist $(BUILD_DIR)\* del /Q $(BUILD_DIR)\*

Flash:
	STM32_Programmer_CLI -c port=SWD reset=SWrst -w build/bare_metal.hex -V -run -rst
```
các flags sẽ được giải thích ở đây 
[GCC](https://gcc.gnu.org/onlinedocs/gcc-15.2.0/gcc/ARM-Options.html)

![Edit the system environment variables](/assets/Bare_Metal_STM32/Flash/run.png)