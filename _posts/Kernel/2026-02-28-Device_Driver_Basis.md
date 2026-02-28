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

Nếu muốn module được load tự động khi boot, tạo file `/etc/modulesload.d/<filename>.conf` 
```bash
echo "mydrv" > /etc/modules-load.d/mydrv.conf
```
mỗi dòng 1 module. `<filename>` nên có ý nghĩa dễ hiểu đối với bạn. Thông thường người ta đặt tên là:
```bash
/etc/modules-load.d/modules.conf
```

#### Auto-loading
`depmod` không chỉ build `modules.dep` mà còn build `modules.dep.bin` file làm nhiều việc hơn.
- Khi một dev kernel viết driver, họ sẽ biết chính xác sẽ support hardware nào, vì vậy họ chịu trách nhiệm cung cấp các product và vendor ID cho tất cả thiết bị được support.
- `depmod` cũng xử lý các file module để trích xuất và tổng hợp thông tin này, sau đó tạo ra file modules.alias, nằm tại:
```bash
/lib/modules/<kernel_release>/modules.alias
```
một `modules.alias` sẽ chứa các thông tin như sau:
```bash
alias usb:v0403pFF1Cd*dc*dsc*dp*ic*isc*ip*in* ftdi_sio
alias usb:v0403pFF18d*dc*dsc*dp*ic*isc*ip*in* ftdi_sio
alias usb:v0403pDAFFd*dc*dsc*dp*ic*isc*ip*in* ftdi_sio
alias usb:v0403pDAFEd*dc*dsc*dp*ic*isc*ip*in* ftdi_sio
alias usb:v0403pDAFDd*dc*dsc*dp*ic*isc*ip*in* ftdi_sio
alias usb:v0403pDAFCd*dc*dsc*dp*ic*isc*ip*in* ftdi_sio
alias usb:v0D8Cp0103d*dc*dsc*dp*ic*isc*ip*in* snd_usb_audio
alias usb:v*p*d*dc*dsc*dp*ic01isc03ip*in* snd_usb_audio
alias usb:v200Cp100Bd*dc*dsc*dp*ic*isc*ip*in* snd_usb_au
```
Ở user space cần hot-plug agent (or device manager), thường `udev` hoặc `mdev` để register với kernel thông báo khi có thiết bị mới được insert vào hệ thống.


Khi đó kernel sẽ gửi device's descriptor (pid, vid, class, device class, device subclass, interface, và tất cả thông tin để nhận diện device) để hot-plug daemond, sau đó daemon sẽ gọi `modprobe` kèm thông tin đó. Tiếp theo `modprobe` sẽ parse `modules.alias` file để tìm module phù hợp và load vào kernel. Nếu module có phụ thuộc vào module khác, `modprobe` sẽ load module phụ thuộc trước.


#### Module unload
Thường sử dụng command `rmmod` để unload module. Nên sử dụng unload cho một module được load bằng `ismod` command. 


- Module unload là một feature để enable/disable module khi cần thiết, theo `CONFIG_MODULE_UNLOAD` config option. 
```bash
CONFIG_MODULE_UNLOAD=y
```

Trong lúc runtime, kernel sẽ ngăn chặn bạn unload module có thể gây lỗi hệ thống. Bởi vì do kernel duy trì một reference count (số lần module được sử dụng), để biết liệu module có đang được sử dụng hay không. Nếu kernel tin rằng việc unload không an toàn để xóa một module, nó sẽ không cho phép bạn unload module đó. Tuy nhiên, bạn có thể thay đổi hành vi như sau:
```bash
MODULE_FORCE_UNLOAD=y
```
Option đầu tiên nên set trong kernel config để force unload một module:
```bash
rmmod -f mymodule
```

Mặt khác, một higher-level command để unload một module bằng cách thông minh hơn là `modprobe -r`.
```bash
modprobe -r mymodule
```
Có thể check xem module có được unload thành công hay không bằng command:
```bash
lsmod
```
## Driver Skeletons
```c
#include <linux/init.h>
#include <linux/module.h>
#include <linux/kernel.h>
static int __init helloworld_init(void) {
    pr_info("Hello world!\n");
    return 0;
}
static void __exit helloworld_exit(void) {
    pr_info("End of the world\n");
}
module_init(helloworld_init);
module_exit(helloworld_exit);
MODULE_AUTHOR("John Madieu <john.madieu@gmail.com>");
MODULE_LICENSE("GPL");
```
### Module entry and exit points
Tất cả các driver trong kernel đều có entry point và exit point.
- Entry point được gọi khi module được load vào kernel (bằng `modprode` hoặc `insmod`).
- Exit point được gọi khi module được unload khỏi kernel (bằng `rmmod` hoặc `modprobe -r`).


So với c/c++ thì có `main()` function ở user space, còn trong kernel thì có `module_init()` và `module_exit()`. Những gì dev cần làm là cho kernel biết function nào là entry point và function nào là exit point. 


**Example**: Với ví dụ trên chương trình trên helloworld driver, `helloworld_init()` là entry point và `helloworld_exit()` là exit point. Sử dụng module_init() và module_exit() macro để cho kernel biết function nào là entry point và function nào là exit point.
```c
module_init(helloworld_init);
module_exit(helloworld_exit);
```
**Note**: Hàm init hoặc exit chỉ được chạy một lần, ngay sau khi module được nạp vào hoặc bị gỡ khỏi kernel.
### Module information
## Errors and messages printing

## Module Parameters

## Building your first module


