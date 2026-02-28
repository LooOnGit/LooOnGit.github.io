---
title: "Device Driver Basis"
date: 2025-02-28 14:12:59 +0800
categories: [Kernel]
tags: [Kernel]
---

# DEVICE DRIVER BASIS

## User space and kernel space
- **Kernel space** và **user space** thì liên quan đến **memory** và **access right** (quyền truy cập).
- **Kernel space** là nơi chứa kernel, driver, và các thành phần khác của hệ điều hành.
- **User space** là nơi chứa các ứng dụng và chương trình.

![User space and kernel space](/assets/Kernel/Device_Driver_Basis/user_space_and_kernel_space.png)

### Sự khác biệt giữa kernel space và user space
#### Kernel space
Đây là nơi địa chỉ nơi **kernel** được lưu trữ (hosted) and run. **Kernel memory** (hay kernel space) là một vùng nhớ được quản lý bởi kernel và được bảo vệ bằng các flag truy cập để ngăn chặn **user space** truy cập vào. Khi vào **kernel mode** thì có thể truy cập toàn bộ hệ thống, vì nó chạy mức ưu tiên cao nhất (cả kernel space và user space).
#### User space
Vùng bộ nhớ dành cho các chương trình người dùng. Bị giới hạn quyền truy cập, không thể trực tiếp truy cập kernel space. Muốn dùng tài nguyên hệ thống phải thông qua system call. Khi chương trình gọi system call, CPU chuyển từ user mode → kernel mode để thực thi. Xong việc, CPU quay lại user mode. 


**Example**: `read`, `write`, `open`, `close` là các system call.

## Driver Skeletons

## Errors and messages printing

## Module Parameters

## Building your first module


