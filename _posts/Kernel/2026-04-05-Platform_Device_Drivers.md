---
title: "Platform Device Drivers"
date: 2026-04-05 15:12:59 +0700
categories: [Kernel]
tags: [Kernel]
---

# Platform Device Drivers
Có nhiều loại device có thể plug và play như USB, PCI, v.v. Nhưng đó là những device mà kernel có thể nhận diện được, nhưng có những device mà kernel không thể nhận diện được, ví dụ như I2C, SPI, v.v. 


Các bus như USB, I2S, I2C, UART, SPI, PCI, SATA, v.v. Những bus như vậy là những hardware device gọi là những controller. Bởi vì trúng là 1 phần của SOC, chúng không thể bị tháo rời, không có khả năng tự phát hiện và cũng được gọi là platform device.


Kernel xem root devices và đã kết nối mọi thứ. Đó là lúc **pseudo platform bus** xuất hiện. Pseudo platform bus được gọi là platform bus, là một kernel virtual bus cho device không nằm trên một bus physical mà kernel đã biết. 


Xử lý platform device driver bao gồm 2 phần: 
- **1**. Register một platform device.
- **2**. Register cho platform device với name đã dùng cho driver, kèm theo các thông tin resource, nhằm thông báo cho kernel biết sự hiện diện của thiết bị.
## Platform drivers
Platform device được đại diện trong kernel như là instance của `struct platform_device`, define trong `include/linux/platform_device.h`.
```c
struct platform_device {
    const char *name;
    int id;
    struct device dev;
    u32 num_resources;
    struct resource *resource;
    // [...]
};
```

Platform driver, trước khi driver và device match, `name` field của `struct platform_device` và `static struct platform_driver.driver.name` phải giống nhau. Chỉ cần nhớ rằng, `resource` và `platform data` là những thông tin mà driver cần để hoạt động. là một array, `num_resources` phải chứa size và array. 
### Resource and platform data
Trái ngược hoàn toàn với các loại thiết bị có khả năng cắm nóng (hot-pluggable), kernel hoàn toàn không biết có những thiết bị nào đang hiện diện trên hệ thống điện toán của bạn, khả năng chuyên biệt của chúng tới đâu, hay chúng cần những cấu hình gì để có thể hoạt động chính xác. Do hệ thống không hề có quá trình thương lượng tự động (auto-negotiation process) giữa phần cứng và hệ điều hành, thế nên bất kỳ thông tin nào được chủ động cung cấp cho kernel đều rất quan trọng.


Hiện tại, có hai phương pháp để thông báo cho kernel về các cấu hình tài nguyên (như ngắt IRQ, kênh DMA, vùng nhớ, các cổng I/O, số cổng bus) cũng như phần dữ liệu (bất kỳ cấu trúc dữ liệu tùy chỉnh và riêng tư nào mà bạn có ý định truyền xuống cho driver) mà thiết bị đang yêu cầu. Cụ thể cả hai phương pháp này sẽ được thảo luận ở phần dưới.
### Device provisioning - the new and recommended way
Phương pháp này chủ yếu được sử dụng với các phiên bản kernel đời trước, do thời điểm đó hệ thống chưa hỗ trợ cấu trúc Device Tree (Cây thiết bị). Với phương pháp này, các hàm của driver vẫn mang tính chất tổng quát (generic), và sự tồn tại của thiết bị sẽ được khai báo cứng (hard-code) trực tiếp bên trong các file mã nguồn C riêng biệt quy định cho từng loại bo mạch (board-related source files).
#### Resource
**Resource (Tài nguyên)** đại diện cho tất cả các thành tố cấu thành nên một thiết bị dưới góc độ của phần cứng (hardware point of view). Thiết bị cần phải gọi đến những tài nguyên này để có thể được khởi tạo và chạy chính xác. Kernel chỉ định nghĩa sẵn 6 loại tài nguyên, tất cả đều được liệt kê ở trong tệp `include/linux/ioport.h`, và chúng được sử dụng dưới dạng các cờ (flags) để mô phỏng loại của tài nguyên đó:
```c
#define IORESOURCE_IO 0x00000100 /* PCI/ISA I/O ports */
#define IORESOURCE_MEM 0x00000200 /* Memory regions */
#define IORESOURCE_REG 0x00000300 /* Register offsets */
#define IORESOURCE_IRQ 0x00000400 /* IRQ line */
#define IORESOURCE_DMA 0x00000800 /* DMA channels */
#define IORESOURCE_BUS 0x00001000 /* Bus */
```
Một resource được đại diện trong kernel như là một instance `struct resource`:
```c
struct resource {
    resource_size_t start;
    resource_size_t end;
    const char *name;
    unsigned long flags;
};
```
Từng thành phần trong structure:
- `start/end`: Đại diện cho vị trí begin/end của resource.  
#### Platform data
#### Where to declare platform devices?

## Devices, drivers and bus matching
### How can platform devices and platform drivers match?
#### Kernel devices and drivers-matching function
##### OF style and ACPI match
##### ID table matching
##### Per device-specific data on ID table matching
##### Name matching - platform device name matching
