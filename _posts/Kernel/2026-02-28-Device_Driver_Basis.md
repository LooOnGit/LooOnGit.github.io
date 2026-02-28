---
title: "Device Driver Basis"
date: 2026-02-28 14:12:59 +0800
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

### The concept of modules
Một **module** giống như phần **plugin** của kernel. Nó được load vào kernel khi cần thiết và được unload khi không cần thiết. Thêm vào để thêm chức năng mới vào kernel khi hệ thống đang chạy, không cần restart máy.
- Để support modules thì kernel phải được built với `CONFIG_MODULES=y`.

### Module dependencies
Trong **linux**, một module có thể cung cấp các function hoặc variable và export chúng bằng macro `EXPORT_SYMBOL`, chúng được gọi là symbols. Dependency của module B vào module A nghĩa là module B dùng symbols của module A.

#### depmod utility
**depmode** là một tool được run trong quá trình build kernel để tạo ra module dependency files. 
- Nó hoạt động bằng cách đọc mỗi module trong `/lib/modules/<kernel-version>/` và tìm các symbols mà nó export. Result của process này được ghi vào `modules.dep` file và version binary của `modules.dep.bin`.

### Module Loading and Unloading
Để module có thể hoạt động, cần load và kernel. 
- `lsmod` : list all modules
- `insmod` : insert module
- `rmmod` : remove module
- `modprobe` : command này thông minh hơn được ưu tiên sử dụng trong các hệ thống production.

#### Manual loading
Có 2 phương thức manual load:
- `insmod` : đây là cách load module ở low-level.


**Example**:
```bash
insmod /path/to/mydrv.ko
```

- `modprobe` : thường được các quản trị viên hệ thống (sysadmin) sử dụng hoặc trong môi trường production. modprobe là một lệnh thông minh vì nó phân tích file modules.dep để nạp các module phụ thuộc trước, rồi mới nạp module được yêu cầu. Nó tự động xử lý phụ thuộc giữa các module, giống như cách một trình quản lý gói (package manager) hoạt động:


**Example**:
```bash
modprobe mydrv
```
#### Auto-loading
#### Module unload

## Driver Skeletons

## Errors and messages printing

## Module Parameters

## Building your first module


