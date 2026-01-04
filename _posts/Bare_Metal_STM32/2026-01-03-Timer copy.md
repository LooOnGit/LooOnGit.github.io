---
title: "General Purpose Timers (TIM)"
date: 2026-01-03 15:52:24 +0700
categories: [Bare Metal STM32]
tags: [Bare Metal STM32]
---

# GENERAL-PURPOSE TIMERS (TIM)

## Introduction to timers and their uses
Timer là ngoại vi phần cứng có trong vi điều khiển, để đếm xung clock. Những xung này có thể sử dụng đo khoảng thời gian, tạo delay chính xác, hoặc trigger event ở khoảng thời gian cụ thể. Timer có thể hoạt động ở nhiều mode, bao gồm couting up, couting down, và tạo ra độ rộng xung (PWM).


Để lựa chọn giữa sử dụng SysTick timer hay general purpose timer trên STM32, cần phụ thuộc vào độ phức tạp và linh hoạt thời gian mà bạn cần. Trong khi **SysTick** rất tốt trong các tác vụ thời gian như tạo ngắt định kỳ hoặc delay, nó giới hạn tính linh hoạt và tính năng, do thường được dành riêng cho **RTOS**. 

 
Mặc khác thì các timer có tính linh hoạt hơn, với nhiều khả năng xử lý tác vụ phức tạp như tạo PWM, input capture, output compare. Chúng hỗ trợ nhiều kênh cho nhiều sự kiện cùng lúc và cung cấp các chức năng nâng cao chẳng hạn đo tần số chính xác và chế độ tiết kiệm năng lượng.
## Common uses cases of timers
- **Time interval measurement**: vi điều khiển gửi một tín hiệu trigger tới ultrasonic sensor, dừng nó phát ra xung phản hồi sau khi gặp vật cản. Timer bắt đầu đếm khi xung phát ra và dừng lại khi nhận được xung phản hồi. 
- **Delay generation**
- **Event trigger**

## STM32 timers
### Introduction to general-purpose timers and advanced timers
Trong STM32F411 thì **Timer1** là timer nâng cao, còn lại là các timer thông dụng.


Các tính năng thông dụng bao gồm:
- **Counter size**: Chúng có bộ đếm 16-bit counter đối với **TIM3**, **TM4** và 32-bit counter cho **TIM2** và **TIM5**, chứa hoạt động đếm lên, xuống, hoặc up/down auto-reload.
- **Prescaler**: Được trang bị bộ chia tần lập trình được 16-bit, cho phép chia tần số xung clock của bộ đếm với bất kỳ hệ số nào từ 1 đến 65.536.
- **Channels**: Cung cấp tối đa bốn kênh độc lập.
- **Interrupt/direct memory access (DMA) generation**: Các timer có thể đồng bộ với tín hiệu bên ngoài và liên kết với nhiều timer khác, giúp tăng tính linh hoạt và khả năng phối hợp trong các ứng dụng phức tạp.
- **Encoder support**: Timer có thể tạo ngắt hoặc yêu cầu DMA dựa trên nhiều sự kiện khác nhau, bao gồm tràn/thiếu bộ đếm, khởi tạo bộ đếm, sự kiện kích hoạt, bắt xung đầu vào (input capture) và so sánh đầu ra (output compare).


Timer nâng cao **TIM1** có kích thước bộ đếm 16-bit. Ngoài sự khác biệt về kích thước bộ đếm, nó kế thừa toàn bộ các tính năng của timer đa dụng và bổ sung thêm các tính năng sau:
- **Complementary outputs**: Hỗ trợ đầu ra bổ sung với khả năng chèn dead-time có thể lập trình.
- **Repetition counter**: Cho phép cập nhật các thanh ghi của timer chỉ sau một số chu kỳ đếm nhất định.
- **Break input**: Có khả năng đưa các tín hiệu đầu ra của timer về trạng thái reset hoặc trạng thái xác định khi được kích hoạt.

### How STM32 timers work
The time-base unit bao gồm register sau:
- **Counter register (TIMx_CNT)**
- **Prescaler register (TIMx_PSC)**
- **Auto-reload register (TIMx_ARR)**


Prescaler register (TIMx_PSC) chia tần số clock của timer với hệ số giá trị từ 1 đến 65,536. Cho phép tốc độ đếm chậm để đáp ứng nhu cầu đo thời gian dài.


Auto-reload register (TIMx_ARR) xác định giá trị mà bộ counter reset về 0. Được dùng để thiết lập chu kỳ hoạt động của timer.


Ngoại vi timer còn bao gồm các thanh ghi điều khiển (**TIM_CR1**, **TIMx_CR2**, ...) để cấu hình các tham số hoạt động, chẳng hặn cho phép counter, setting trược tiếp bộ đếm, và cấu hình update event (**UEV**). Thanh ghi trạng thái (**Status Register - SR**)(**TIMx_SR**) cho biết trạng thái timer. Chẳng hạn như **UEV** xảy ra hay chưa xảy ra.


Timer có 2 mode là: **Up-counting mode** và **Down-counting mode**.


**UEV là gì?**
- Nói 1 cách đơn giản, UEV là thời điểm xảy ra timer.
- Khi counter đạt tới giá trị auto-reload hoặc 0 (theo chế độ count mode), một UEV sẽ xảy ra.
- Sự kiện này có thể trigger một interrupt hoặc DMA request và set **update interrupt flag**(**UIF**) trong **SR**(TIMx_SR).
- UEV cũng có thể tạo ra thủ công bằng setting **UG bit** trong register tạo ra sự kiện (**TIMx_EGR**).


#### Timer clock prescaling

![Prescaler](/assets/Bare_Metal_STM32/Timer/image.png)


- **System clock (SYSCLK)**: Trong hệ thông clock chính của vi điều khiển. Nó phục vụ khai báo clock source cho tất cả hoạt động phụ.
- **Advance High-performance Bus (AHB) prescaler**: Trong bộ chia đầu tiên trong chuỗi này, nó không chia tần số **SYSCLK** theo hệ số **1,2,4,8,16,64,128,256,or 512**. Tín hiệu xung nhịp thu được sau khi chia được gọi là **HCLK** (AHB Block).
- **High-performance Bus Clock (HCLK)**: Tín hiệu được tạo ra sau AHB prescaler. Nó được sử dụng điều khiển core và hệ thống bus.
- **Advanced Peripheral Bus (APB) prescaler**:


**1. APB1 prescaler**: Bộ chia này tiếp tục chia HCLK để tạo ra xung nhịp cho bus ngoại vi APB1.
Các hệ số chia có thể sử dụng là 1, 2, 4, 8 và 16.


**2. APB2 prescaler**: Tương tự như bộ chia APB1 nhưng được dùng cho bus ngoại vi APB2.
Nó cũng chia HCLK theo các hệ số 1, 2, 4, 8 và 16.

- **Timer prescaler** (TIMx_PSC): Mỗi timer có bộ prescaler tiếng, nó chia tiếp clock từ **APB1** hoặc **APB2** theo bất kì giá trị nào từ 1 đến 65,536. Sự linh hoạt này cho phép điều khiển chính xác tốc độ đếm của timer.
- **Timer counter** (TIMx_CNT): Đây là bộ đếm cuối cùng, sử dụng xung nhịp được tạo ra từ TIMx_PSC. Tốc độ mà bộ đếm này tăng hoặc giảm phụ thuộc vào tổng hiệu ứng chia tần của các prescaler trước đó.


#### Computing a UEV

```c
Update Event Frequency = (Timer Clock)/[(Prescaler + 1)*(ARR + 1)]
```
- **Timer Clock**: Tần số clock của timer (APB1 hoặc APB2).Tần số của APB1 và APB2 bus thì bằng **SYSCLK** khi sử dụng clock cấu hình mặc định.
- **Prescaler**: Giá trị này được set trong register **TIMx_PSC**.
- **Period**: Giá trị set ở trong register **TIMx_ARR**.


Để ví dụ sau:
- **Timer clock (APB1 clock)**: 16MHz
- **Prescaler**: 15999
- **Period (TIMx_ARR value)**: 499


```c
Update Event Frequency = (Timer Clock)/[(Prescaler + 1)*(ARR + 1)]
Update Event Frequency = (16MHz)/[(15999 + 1)*(499 + 1)] = 16MHz/8MHz = 2Hz
```
Vậy thì UEV sẽ xảy ra với tần số 2Hz, tức là mỗi giây 2 lần.
## Developing the timer driver
- **Step1**: Enable the clock for TIMx
- **Step2**: Sets prescaler value to divide the input clock to 10kHz.
- **Step3**: Sets an auto-reaload value to make the timer count up to 10,000, creating a 1-second period.
- **Step4**: Clears the timer counter.
- **Step5**: Enable the timer.


