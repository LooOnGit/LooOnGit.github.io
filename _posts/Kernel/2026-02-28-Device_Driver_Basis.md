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
#### `__init` and `__exit` attributes
`__init` và `__exit` là kernel macros, được định nghĩa trong `include/linux/init.h`.
```c
#define __init __section(.init.text)
#define __exit __section(.exit.text)
```
- `__init` yêu cầu **linker** đặt code vào section riêng biệt trong file object kernel. Section này được kernel giải phóng sau khi module được nạp và hàm init được thực thi. Điều này chỉ áp dụng cho các driver được biên dịch trực tiếp vào kernel (built-in drivers), không áp dụng cho các module có thể nạp động (loadable modules). Kernel sẽ chạy hàm init của driver lần đầu tiên trong quá trình khởi động (boot).
- `__exit` tương tự code này bị loại khi module được compiled statically vào kernel hoặc khi module được unload khỏi kernel không được enable. Bởi cả 2 trường hợp đó hàm **exit** không bao giờ được gọi. `__exit` chỉ áp dụng cho các module có thể nạp động (loadable modules). 


Tất cả attributes trên liên quan đến các file đối tượng có định dạng **Executable and Linkable Format (ELF)**. ELF là một định dạng file được sử dụng rộng rãi trong các hệ điều hành dựa trên Unix để lưu trữ các file thực thi, file object, thư viện chia sẻ và file core dump.


Có thể run `objdump -h module.ko` để in ra các section khác nhau để tạo thành module.ko kernel module.

![ELF file](/assets/Kernel/Device_Driver_Basis/ELF.png)


Chỉ có một vài section trong hình minh họa là các section chuẩn của ELF:
- `.text`: chứa code thực thi của chương trình.
- `.data`: chứa các biến toàn cục được khởi tạo.
- `.bss`: chứa các biến toàn cục không được khởi tạo.
- `.rodata`: chứa các hằng số (read-only data).
- `.comment`: chứa thông tin comment.  

Các section khác được thêm vào theo nhu cầu riêng của kernel. 
- `.modinfo`: chứa thông tin về module.
- `.init.text`: chứa code thực thi của hàm init.
- `.exit.text`: chứa code thực thi của hàm exit.
- `.modsym`: chứa thông tin về symbol.
- `.modstr`: chứa thông tin về string.
- `.modrel`: chứa thông tin về relocation.
### Module information
Mỗi kernel module có một section đặc biệt tên là `.modinfo`.


**Ví dụ thông tin có thể lưu:**
- Tác giả
- Mô tả
- License
- Tham số

Các macro MODULE_* sẽ ghi thông tin vào `.modinfo`:
- `MODULE_DESCRIPTION()` → mô tả module
- `MODULE_AUTHOR()` → tác giả
- `MODULE_LICENSE()` → loại license


Có thể dump nội dùng của `.modinfo` bằng command:
```bash
objdump -d -j .modinfo module.ko
```

![modinfo](/assets/Kernel/Device_Driver_Basis/modinfo.png)


Có thể dùng đển `modinfo` command để xem thông tin của module:

![modinfo2](/assets/Kernel/Device_Driver_Basis/modinfo2.png)

### Licensing
License được defined bằng `MODULE_LICENSE()` macro.


Điều này ảnh hưởng đến hành vi của module, vì nếu giấy phép không tương thích với GPL, thì module của bạn sẽ không thể sử dụng các dịch vụ/hàm mà kernel xuất ra thông qua macro `EXPORT_SYMBOL_GPL()`. Macro này chỉ xuất các symbol cho những module tương thích GPL.
Điều này trái ngược với EXPORT_SYMBOL(), macro này xuất các hàm cho module với bất kỳ loại giấy phép nào.

Các loại liences được kernel hỗ trợ:
- "GPL"
- "GPL v2"
- "GPL and additional rights"
- "Dual BSD/GPL"
- "Dual MIT/GPL"
- "Proprietary"
- "Unrestricted"
- "Dual MPL/GPL"

### Module author(s)
`MODULE_AUTHOR()` thông báo tác giả của module. Có thể nhiều tác giả


**Example**:
```c
MODULE_AUTHOR("John Madieu <[EMAIL_ADDRESS]>");
MODULE_AUTHOR("Lorem Ipsum <[EMAIL_ADDRESS]>");
```

### Module description
`MODULE_DESCRIPTION()` mô tả module.


**Example**:
```c
MODULE_DESCRIPTION("Hello world driver");
```

## Errors and messages printing
### Error handling
Kernel hoặc chương trình user space có thể:
- Hiểu sai tình huống
- Đưa ra quyết định sai
- Gây hành vi không mong muốn


Các lỗi nàu được định nghĩa trong `include/uapi/asm-generic/errno-base.h` và `include/uapi/asm-generic/errno.h`. Một số liệt kê dưới đây:
```c
#define EPERM 1 /* Operation not permitted */
#define ENOENT 2 /* No such file or directory */
#define ESRCH 3 /* No such process */
#define EINTR 4 /* Interrupted system call */
#define EIO 5 /* I/O error */
#define ENXIO 6 /* No such device or address */
#define E2BIG 7 /* Argument list too long */
#define ENOEXEC 8 /* Exec format error */
#define EBADF 9 /* Bad file number */
#define ECHILD 10 /* No child processes */
#define EAGAIN 11 /* Try again */
#define ENOMEM 12 /* Out of memory */
#define EACCES 13 /* Permission denied */
#define EFAULT 14 /* Bad address */
#define ENOTBLK 15 /* Block device required */
#define EBUSY 16 /* Device or resource busy */
#define EEXIST 17 /* File exists */
#define EXDEV 18 /* Cross-device link */
#define ENODEV 19 /* No such device */
#define ENOTDIR 20 /* Not a directory */
#define EISDIR 21 /* Is a directory */
#define EINVAL 22 /* Invalid argument */
#define ENFILE 23 /* File table overflow */
#define EMFILE 24 /* Too many open files */
#define ENOTTY 25 /* Not a typewriter */
#define ETXTBSY 26 /* Text file busy */
#define EFBIG 27 /* File too large */
#define ENOSPC 28 /* No space left on device */
#define ESPIPE 29 /* Illegal seek */
#define EROFS 30 /* Read-only file system */
#define EMLINK 31 /* Too many links */
#define EPIPE 32 /* Broken pipe */
#define EDOM 33 /* Math argument out of domain of func */
#define ERANGE 34 /* Math result not representable */
```
Khi đổi mặt với error, bạn phải undo mọi thứ trước khi error xảy ra thường bằng cách dùng `goto`.


**Example**:
```c
ptr = kmalloc(sizeof (device_t));
if(!ptr) {
    ret = -ENOMEM;
    goto err_alloc;
}

dev = init(&ptr);

if(dev) {
    ret = -EIO
    goto err_init;
}

return 0;

err_init:
    free(ptr);

err_alloc:
    return ret;
```


Tại sao lại dùng `goto` thì ví dụ có 5 step thì khi chạy đến step5 bị lỗi, thì phải clean các operation (step 4, 3, 2, 1) theo thứ tự ngược lại thay vì tạo ra nhiều operation checking.

### Handling null pointer errors
Khi trả lỗi từ các hàm thì hay dùng đến return NULL. Cách này thì được phép làm nhưng vô nghĩa không biết lý do tại sao NULL. 


Vì vậy kernel cung cấp 3 hàm để xử lý lỗi:
- `PTR_ERR()`: trả về lỗi dưới dạng số nguyên
- `IS_ERR()`: kiểm tra xem có lỗi không
- `ERR_PTR()`: trả về con trỏ từ số nguyên

**Example**:
```c
void *ERR_PTR(long error);
long IS_ERR(const void *ptr);
long PTR_ERR(const void *ptr);
```

#### PTR_ERR()
Trả về lỗi dưới dạng con trỏ
```c
ERR_PTR(-ENOMEM);
```

#### IS_ERR()
Kiểm tra xem giá trị trả về có phải là một con trỏ lỗi hay không
```c
if (IS_ERR(foo))
```

#### ERR_PTR()
Trả về error code.
```c
return PTR_ERR(foo);
```

**Example**: Dùng 3 function trên.
```c
static struct iio_dev *indiodev_setup() {
    [...]
    struct iio_dev *indio_dev;

    indio_dev = devm_iio_device_alloc(&data->client->dev, sizeof(data));
    if (!indio_dev)
        return ERR_PTR(-ENOMEM);

    [...]
    return indio_dev;
}

static int foo_probe([...]) {
    [...]
    struct iio_dev *my_indio_dev = indiodev_setup();

    if (IS_ERR(my_indio_dev))
        return PTR_ERR(data->acc_indio_dev);

    [...]
}
```

**Note**: 
Đây là một điểm cộng trong việc xử lý lỗi; đồng thời cũng là một phần trong phong cách lập trình của kernel, trong đó quy định rằng:
- Nếu tên hàm là một hành động hoặc mệnh lệnh, thì hàm đó nên trả về một số nguyên là mã lỗi.


**Ví dụ**: `add_work()` là một lệnh, và hàm này trả về 0 nếu thành công hoặc -EBUSY nếu thất bại.


- Nếu tên hàm là một mệnh đề điều kiện (predicate), thì hàm nên trả về giá trị Boolean thành công/thất bại.


**Ví dụ**: pci_dev_present() là một mệnh đề kiểm tra, và hàm này trả về 1 nếu tìm thấy thiết bị phù hợp, hoặc 0 nếu không tìm thấy.

### Message printing - printk()
- `printk()`: dùng trong kernel.
- `printf()`: dùng trong user space.

Dùng `printk()` thì những line được in ra phải xem bằng `dmesg` command. Có thể chọn log level để in ra gồm có 8 level log được define trong `include/linux/kern_levels.h`, level 0 có mức độ ưu tiên cao nhất và level 7 có mức độ ưu tiên thấp nhất.


```c
#define KERN_SOH "\001"        /* ASCII Start Of Header */
#define KERN_SOH_ASCII '\001'

#define KERN_EMERG   KERN_SOH "0" /* hệ thống không thể sử dụng */
#define KERN_ALERT   KERN_SOH "1" /* cần hành động ngay lập tức */
#define KERN_CRIT    KERN_SOH "2" /* tình trạng nghiêm trọng */
#define KERN_ERR     KERN_SOH "3" /* điều kiện lỗi */
#define KERN_WARNING KERN_SOH "4" /* điều kiện cảnh báo */
#define KERN_NOTICE  KERN_SOH "5" /* điều kiện bình thường nhưng đáng chú ý */
#define KERN_INFO    KERN_SOH "6" /* thông tin */
#define KERN_DEBUG   KERN_SOH "7" /* thông điệp mức debug */
```

**Example**:
```c
printk(KERN_INFO "Hello world\n");
```

Nếu không có level log thì mặc định là kernel sẽ gán múc theo cấu hình `CONFIG_DEFAULT_MESSAGE_LOGLEVEL`.


Thực tế thì có thể dùng đến macro `pr_info()`, `pr_err()`, `pr_warn()`, `pr_debug()` thay vì dùng `printk()`.


**Example**:
```c
pr_info("Hello world\n");
```


Bạn có thể kiểm tra các tham số mức log bằng lệnh:
```bash
cat /proc/sys/kernel/printk

4 4 1 7
```
**Trong đó:**
- Giá trị đầu tiên (4) là mức log hiện tại.
- Giá trị thứ hai (4) là mức mặc định theo cấu hình CONFIG_DEFAULT_MESSAGE_LOGLEVEL.
- Các giá trị còn lại ....


**Note**: printk() là hàm in log của kernel, tương tự printf() trong user space. Nó không bao giờ block nên an toàn khi gọi cả trong atomic context. Khi được gọi, printk() sẽ cố khóa console để in thông điệp; nếu không khóa được, nó ghi thông điệp vào buffer rồi trả về ngay, và console sẽ in các thông điệp đó sau. Ngoài printk(), kernel còn hỗ trợ các phương pháp debug khác như dùng `#define DEBUG`hoặc cơ chế dynamic debug.
## Module Parameters
Kernel cũng có thể nhận tham số từ command line. Sử dụng macro `module_param()` để khai báo tham số.

```c
module_param(name, type, perm);
```

**Trong đó:**
- `name`: Tên của biến dùng làm tham số.
- `type`: Kiểu dữ liệu của tham số (bool, charp, byte, short, ushort, int, uint, long, ulong), trong đó charp là con trỏ char.
- `perm`: Quyền truy cập của tham số, tương tự như quyền truy cập file trong `/sys/module/<module>/parameters/<param>`. Một số quyền gồm `S_IWUSR`, `S_IRUSR`, `S_IXUSR`, `S_IRGRP`, `S_WGRP`, `S_IRUGO`.
    - `S_I` is just a prefix
    - `R`: read, `W`: write, `X`: execute
    - `USR`: user, `GRP`: group, `UGO`: user, group, others

**Example**:
```c
module_param(my_param, int, 0644);
```


Khi sử dụng tham số module, bạn nên dùng macro `MODULE_PARM_DESC` để mô tả từng tham số. Macro này sẽ thêm mô tả của từng tham số vào section thông tin module (`.modinfo`).


**Example**:
```c
#include <linux/moduleparam.h>
[...]

static char *mystr = "hello";
static int myint = 1;
static int myarr[3] = {0, 1, 2};

module_param(myint, int, S_IRUGO);
module_param(mystr, charp, S_IRUGO);
module_param_array(myarr, int, NULL, S_IWUSR | S_IRUSR);

MODULE_PARM_DESC(myint, "this is my int variable");
MODULE_PARM_DESC(mystr, "this is my char pointer variable");
MODULE_PARM_DESC(myarr, "this is my array of int");

static int foo()
{
    pr_info("mystring is a string: %s\n", mystr);
    pr_info("Array elements: %d\t%d\t%d", myarr[0], myarr[1], myarr[2]);
    return myint;
}
```
Sau đó thì load module:
```bash
insmod hellomodule-params.ko mystring="packtpub" myint=15 myArray=1,2,3
```
Có thể sử dụng lệnh `modinfo` trước khi nạp module để hiển thị mô tả các tham số mà module hỗ trợ:
```bash
$ modinfo ./helloworld-params.ko

filename: /home/jma/work/tutos/sources/helloworld/./helloworld-params.ko
license: GPL
author: John Madieu <john.madieu@gmail.com>
srcversion: BBF43E098EAB5D2E2DD78C0
depends:
vermagic: 4.4.0-93-generic SMP mod_unload modversions
parm: myint:this is my int variable (int)
parm: mystr:this is my char pointer variable (charp)
parm: myarr:this is my array of int (array of int)
```
- `parm`: hiển thị tên tham số.
- Phần sau dấu `:` là mô tả đã khai báo bằng `MODULE_PARM_DESC`.
- Trong ngoặc là kiểu dữ liệu của tham số.

## Building your first module


