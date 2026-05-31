---
title: "Enviroment"
date: 2025-11-09 13:56:25 +0800
categories: [Kernel]
tags: [Kernel][Beaglebone]
---

# 🐧 Enviroment Beaglebone

## Toolchain để build cho board cross compile
![alt text](/assets/BeagleBone/Enviroment/image_1.png)

Dùng đến **gcc-arm-10.3-2021.07-x86_64-arm-none-linux-gnueabihf** để biên dịch chạy trên được board.

![alt text](/assets/BeagleBone/Enviroment/image_2.png)

Làm như sau để thấy trình biên dịch trên máy host **gcc** là biên dịch cho c, còn g++ biên dịch cho c++.

Trong linux các thiết bị phần cứng thường được đại diện bằng 1 file nằm trong thư mục `dev/sda` không thể xem trực tiếp mà phải mount về mới xem được.

![alt text](/assets/BeagleBone/Enviroment/image_3.png)

## Giới thiệu về BeagleBone
Dùng chip AM335x tổng cộng có 128 pin chia làm 4 group gpio, mỗi bank chứa 32 pin (4x32=128).
- GPIO bank 0 (0-31)
- GPIO bank 1 (32-63)
- GPIO bank 2 (64-95)
- GPIO bank 3 (96-128)
Links: [AM335x](https://www.ti.com/lit/ug/spruh73q/spruh73q.pdf?ts=1762611603905&ref_url=https%253A%252F%252Fwww.ti.com%252Fproduct%252FAM3358%253F)


