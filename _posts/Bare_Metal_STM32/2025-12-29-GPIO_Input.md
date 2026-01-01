---
title: "GPIO Input"
date: 2025-12-29 06:43:52 +0700
categories: [Bare Metal STM32]
tags: [Bare Metal STM32]
---

# The General-Purpose Input/Output (GPIO) Peripheral
## Understanding the GPIO peripheral
Trong STM32F411 thì có đến 6 port: **PORTA**, **PORTB**, **PORTC**, **PORTD**, **PORTE** và **PORTH**.


Port này cung cấp đa dạng feature, bao gồm:
- **I/O control**: Nó cho phép để quản lý đến 16 input/output mỗi port.
- **Output states**: Pins có thể cấu hình cho push-pull hoặc open-drain.
- **Output data**: Output data thì điều khiển bằng `GPIOx_ODR` register khi pin cấu hình như là tạo ra output data. Đối với cấu hình**alternate function**, liên kết với peripheral điều khiển output data.
- **Speed selection**: Để cấu set operating speed cho mỗi pin.
- **Input states**: Cấu hình cho pin chẳng hạn như floating, pull-up, pull-down or analog input.
- **Input data**: Data can be read from the GPIOx_IDR register or an associated peripheral when configured for alternate function input.
- **Configuration locking**: The GPIOx_LCKR register is used to lock the I/O configuration preventing accidental changes.
- **Alternate function selection**: Up to 16 alternate functions per I/O pin can be configured, providing flexibility for connecting peripherals.

## The STM32 GPIO register
Each GPIO port include a set of 32-bit register essential for configuration and control. The configuration register comprise the following:
- **GPIOx_MODER** (mode register)
- **GPIOx_TYPER** (output type register)
- **GPIOx_OSPEEDR** (output speed register)
- **GPIOx_PUPDR** (pull-up/pull-down register)


The data register inclue the following:
- **GPIOx_IDR** (input data register)
- **GPIOx_ODR** (output data register)


**GPIOx_BSRR** (the bit-set/ reset register) and GPIOx_LCKR (the locking register) are used to control pin state and access. 


**GPIOx_AFRL** and **GPIOx_AFRH** (the alternate function low and high register) are used to select alternate function for each pin.
### The GPIO mode register (GPIOx_MODER)
This register allows us to set each pin in different modes, such as **input**, **output**, **alternate function**, or **analog**.


It is a 32-bit register divided into 16 pairs of bits. Each pair of bits correspond to a specific in the GPIO port, allowing the individual configuration of each pin.


![GPIO mode register](/assets/Bare_Metal_STM32/GPIO/GPIO_mode.png)

### The GPIO output data register (GPIOx_ODR) and the GPIO input data register (GPIOx_IDR)
GPIOx_ODR is a 32-bit register, but only the lower 16 bits are used to control the output state of the pins.


![GPIO data output register](/assets/Bare_Metal_STM32/GPIO/GPIO_Output_data.png)


GPIOx_IDR is used to read the current state of the GPIO pins configured as inputs. By reading from this register, we can determine whether each input is at a high or low logic level.


![GPIO data input register](/assets/Bare_Metal_STM32/GPIO/GPIO_Input_data.png)

### The GPIO bit set/reset register (GPIOx_BSRR)
Controlling the state of GPIO pins. It provides atomic bitwise operations to set or reset individual bits, which ensures that no interrupt can disrupt the operation, maintaining data integrity during the modification.


GPIOx_BSRR is a 32-bit register divided into two 16-bit sections


![GPIO BSRR](/assets/Bare_Metal_STM32/GPIO/GPIO_BSRR.png)

### The GPIO alternate function register (GPIOx_AFRL and GPIOx_AFRH)
The GPIO **alternate function register** (AFRs) are important for configuring the alternate functions of the GPIO pins in STM32 micontrollers. These register allow each pin to be assigned a specific peripheral function, enhancing the versatility of the microcontroller.


GPIOx_AFRL is a 32-bit register, divided into eight 4-bit fields. Each 4-bit field correspond to one pin in the range of pin 0 to 7 in the GPIO port.


![GPIO AFRL](/assets/Bare_Metal_STM32/GPIO/GPIO_AFRL.png)


GPIOx_AFRH is also a 32-bit register, divided into eight 4-bit fields. Each 4-bit field correspond to one pin in the range of pins 8 to 15 in the GPIO port.


![GPIO AFRH](/assets/Bare_Metal_STM32/GPIO/GPIO_AFRH.png)


![GPIO AFRH](/assets/Bare_Metal_STM32/GPIO/Alternate_function.png)


## Developing input and output drivers

### The GPIO output driver using the BSRR
- **Step1**: Set clock
- **Step2**: Set Mode output 
- **Step3**: Set/Reset register BSSR

**Link code**: [BSSR GPIO](https://github.com/LooOnGit/Bare_Metal_STM/blob/406978499279e0f892d24866a4ba76f388e51e8e/Src/main.c)


### The GPIO input driver
- **Step1**: Set clock
- **Step2**: Set Mode input
- **Step3**: Read register IDR

**Link code**: [Read GPIO](https://github.com/LooOnGit/Bare_Metal_STM/blob/f8e3cd201d8e29d6b89bd87afbd1badd101e1322/Src/main.c)

