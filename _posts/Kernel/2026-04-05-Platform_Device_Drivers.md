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
#### Resource
#### Platform data
#### Where to declare platform devices?
### Device provisioning - the new and recommended way
## Devices, drivers and bus matching
### How can platform devices and platform drivers match?
#### Kernel devices and drivers-matching function
##### OF style and ACPI match
##### ID table matching
##### Per device-specific data on ID table matching
##### Name matching - platform device name matching
