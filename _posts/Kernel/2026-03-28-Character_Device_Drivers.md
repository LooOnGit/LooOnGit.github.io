---
title: "Character Device Drivers"
date: 2026-03-28 23:52:59 +0800
categories: [Kernel]
tags: [Kernel]
---

# Character Device Drivers

Khái niệm cơ bản của Linux là tất cả mọi thứ đều là file. Character device driver là một loại device driver cơ bản trong kernel source. Character device thì được đại diện trong kernel như là instance của `struct cdev`, define trong `include/linux/cdev.h`.

```c
struct cdev {
    struct kobject kobj;
    struct module *owner;
    const struct file_operations *ops;
    struct list_head list;
    dev_t dev;
    unsigned int count;
}
```
## The concept behind major and minor
Character device được đặt trong `/dev`. 


![Character device](/assets/Kernel/Character_Device_Drivers/img1.png)


Cột đầu tiên để xác định file type, có thể có tiền tố (preceding) là:
- `c`: Character device file
- `d`: Directory
- `b`: Block device file
- `l`: Symbolic link
- `p`: FIFO (named pipe)
- `s`: Socket


Trước cột date theo dạng `<X,Y>` là major và minor number. Đây là cách nhận identifer của device ở user space.


Kernel lưu trữ các indentify một device trong biến type là `dev_t`. Chúng là kiểu **u32**(32-bit unsigned long). **Major** được đại diện 12 bit đầu tiên, **minor** được đại diện 20 bit sau.


Kernel cung cấp macro để lấy major và minor number:
- `MAJOR(dev_t dev)`: Lấy major number
- `MINOR(dev_t dev)`: Lấy minor number
- `MKDEV(unsigned int major, unsigned int minor)`: Tạo dev_t từ major và minor number
```c
#define MINORBITS 20
#define MINORMASK ((1U << MINORBITS) - 1)
#define MAJOR(dev) ((unsigned int) ((dev) >> MINORBITS))
#define MINOR(dev) ((unsigned int) ((dev) & MINORMASK))
#define MKDEV(ma,mi) (((ma) << MINORBITS) | (mi))
```


Device được đăng ký với một major number để nhận diện device, và một minor, nó có thể sử dụng như một index của array tối một local list device. Do một instance của cùng một trình điều khiển driver có thể điều khiển nhiều thiết bị, trong khi các trình điều khiển khác nhau có thể điều khiển các thiết bị khác nhau của cùng một loại.
### Device number allocation and freeing
Có 2 cách cấp phát device number:
- **Static allocation**: Đoán 1 major number chưa được driver sử dụng , thông qua hàm `register_chrdev_region()`. Tránh dùng function này càng nhiều càng tốt.
```c
int register_chrdev_region(dev_t first, unsigned int count, char *name);
```
Return 0 nếu thành công, ngược lại thì error code thì faild. 
- `first`: Tạo major number, cùng minor đầu tiên của range mong muốn. Nên sử dụng `MKDEV(ma, mi)` để tạo major number.
- `count`: Số lượng device number muốn cấp phát.
- `name`: Tên của device or driver.


- **Dynamic allocation**: Để kernel làm, sử dụng `alloc_chrdev_region()` function.
```c
int alloc_chrdev_region(dev_t *dev, unsigned int firstminor, unsigned int count, char *name);
```
Return 0 nếu thành công, ngược lại thì error code thì faild. 
- `dev`: Là param output, đại diện cho first number mà kernel đã gán.
- `firstminor`: Minor đầu tiên của range mong muốn.
- `count`: Số lượng device number muốn cấp phát.
- `name`: Tên của device or driver.
## Introduction to device file operations
Các operation được định nghĩa tronng kernel dưới dạng instance của `struct file_operations`. `struct file_operations` tập hợp các **callback** để handler bất cứ **system call** nào từ user space trên một file. Ví dụ, nếu bạn muốn cho phép user thực hiện lệnh write trên một file đại diện cho device của bạn, phải thực thi callback tương ứng với `write` function đó và thêm nó vào `struct file_operations` cấu trúc này sẽ được liên kết với device của bạn.
```c
struct file_operations {
    struct module *owner;
    loff_t (*llseek) (struct file *, loff_t, int);
    ssize_t (*read) (struct file *, char __user *, size_t, loff_t *);
    ssize_t (*write) (struct file *, const char __user *, size_t, loff_t *);
    unsigned int (*poll) (struct file *, struct poll_table_struct *);
    int (*mmap) (struct file *, struct vm_area_struct *);
    int (*open) (struct inode *, struct file *);
    long (*unlocked_ioctl) (struct file *, unsigned int, unsigned long);
    int (*release) (struct inode *, struct file *);
    int (*fsync) (struct file *, loff_t, loff_t, int datasync);
    int (*fasync) (int, struct file *, int);
    int (*lock) (struct file *, int, struct file_lock *);
    int (*flock) (struct file *, int, struct file_lock *);
    // [...]
};
```
Một số method quan trọng, Có thể tìm thấy mô tả trong `include/linux/fs.h` trong kernel source. 
- Mỗi callback trong cấu trúc file_operations sẽ tương ứng với một system call.
- Không bắt buộc phải khai báo toàn bộ các callback này.
- Cơ chế hoạt động: Khi User gọi một system call trên file, Kernel sẽ tìm đến struct file_operations của Driver quản lý file đó:
    - Nếu callback tương ứng đã được định nghĩa: Kernel sẽ thực thi nó.
    - Nếu callback chưa được định nghĩa: Kernel sẽ tự động trả về một mã lỗi (error code) tùy theo loại lệnh (ví dụ: thiếu mmap trả về -ENODEV, thiếu write trả về -EINVAL).
### File representation in the kernel
Kernel mô tả các file như là instance của cấu trúc `struct inode` (chứ không phải `struct file`), define trong `include/linux/fs.h`.
```c
struct inode {
    // [...]
    struct pipe_inode_info *i_pipe; /* Set and used if this is a linux kernel pipe */
    struct block_device *i_bdev;    /* Set and used if this is a block device */
    struct cdev *i_cdev;            /* Set and used if this is a character device */
    // [...]
};
```
`struct inode` là một filesystem data structure giữ thông tin, nó chỉ liên quan tới OS, về một (type của nó: character, block, pipe, v.v.) hoặc directory (từ góc độ của kernel, một thư mục cũng chỉ là một tập tin mà các mục nhập (entry) của nó trỏ đến các tập tin khác) trên ổ đĩa.


Cấu trúc `struct file` được define `include/linux/fs.h`, thực chất là một mức độ mô tả tập tin cao hơn, đại diện cho một tập tin đang được mở (open file) bên trong kernel, và nó dựa trên (phụ thuộc vào) cấu trúc dữ liệu struct inode ở mức thấp hơn.
```c
struct file {
    [...]
    struct path f_path; /* Path to the file */
    struct inode *f_inode; /* inode associated to this file */
    const struct file_operations *f_op;/* operations that can be
            * performed on this file
            */
    loff_t f_pos; /* Position of the cursor in
            * this file */
    /* needed for tty driver, and maybe others */
    void *private_data; /* private data that driver can set
            * in order to share some data between file
            * operations. This can point to any data
            * structure.
            */
    [...]
}
```
**inode** không track được vị trí hiện tại trong file (cursor position) hay mode hiện tại. Nó chỉ chừa thông tin giúp OS tìm nội dung cấu trúc file (pipe, directory, regular disk file, block/character device file, and so on).


**struct file** được sử dụng như một structure chung (thực tế nó giữ một con trỏ trỏ tới một cấu trúc `struct inode`) điều này đại diện cho một tập tin đang được mở và nó cung cấp một bộ các hàm (functions) liên quan đến các method `open`, `write`, `seek`, `read`, `select`, v.v.


`struct inode` đại diện cho một file trong kernel, và một `struct file` mô tả file nó thực sự đang được mở. Có thể có nhiều file descriptor khác nhau địa diện cho cùng một tập trinh được mở đi mở lại nhiều lần, nhưng những file descriptor này sẽ trỏ tới cùng một `inode` duy nhất.

## Allocating and registering a character device
Character device đại diện trong kernel dưới dạng instance của `struct cdev`. Khi viết một character device driver, mục tiêu create và register một instance có liên kết với cấu trúc `struct file_operations`, thao tác này liên kết với `struct file_operation` việc này giúp cho user space có thể thực hiện các thao tác với device.


Các bước thực hiện:
- 1. Cấp phát một major và một range của minor với `alloc_chrdev_region()`.
- 2. Create một class cho device với `class_create()`, hiển thị trong `/sys/class/`.
- 3. Set up `struct file_operation` (để truyền vào `cdev_init()`), và mỗi device cần create, call `cdev_init()` và `cdev_add()` để register device.
- 4. Sau đó, create một `device_create()` cho mỗi device, với một proper name (tên phù hợp), kết quả là device sẽ được create vào trong `/dev` directory:
```c
#define EEP_NBANK 8
#define EEP_DEVICE_NAME "eep-mem"
#define EEP_CLASS "eep-class"

struct class *eep_class;
struct cdev eep_cdev[EEP_NBANK];
dev_t dev_num;

static int __init my_init(void)
{
    int i;
    dev_t curr_dev;

    /* Request the kernel for EEP_NBANK devices */
    alloc_chrdev_region(&dev_num, 0, EEP_NBANK, EEP_DEVICE_NAME);

    /* Let's create our device's class, visible in /sys/class */
    eep_class = class_create(THIS_MODULE, EEP_CLASS);

    /* Each eeprom bank represented as a char device (cdev) */
    for (i = 0; i < EEP_NBANK; i++) {
        /* Tie file_operations to the cdev */
        cdev_init(&eep_cdev[i], &eep_fops);
        eep_cdev[i].owner = THIS_MODULE;

        /* Device number to use to add cdev to the core */
        curr_dev = MKDEV(MAJOR(dev_num), MINOR(dev_num) + i);

        /* Now make the device live for the users to access */
        cdev_add(&eep_cdev[i], curr_dev, 1);

        /* create a device node each device /dev/eep-mem0, /dev/eep-mem1,
         * With our class used here, devices can also be viewed under
         * /sys/class/eep-class.
         */
        device_create(eep_class,
                      NULL,             /* no parent device */
                      curr_dev,
                      NULL,             /* no additional data */
                      EEP_DEVICE_NAME "%d", i); /* eep-mem[0-7] */
    }
    return 0;
}
```
## Writing file operations
### Exchanging data between kernel space and user space
- `write()` phương thức bao gồm đọc data từ user space và sau đó được xử lý bởi kernel.
- `read()` phương thức bao gồm đọc data từ kernel đến user space.


`__user` để đánh dấu cho công cụ sparse (một công cụ kiểm tra được kernel sử dụng để tìm kiếm các lỗi lập trình tiềm ẩn), nhằm đảm bảo cho developer họ chuẩn bị thao tác sai cách với một con trở không đáng tin cậy (unstrusted pointer - một con trỏ có thể không hợp lệ ở dạng map địa chỉ ảo hiện tại). Developer tuyệt đối không được pheps deference (nhảy thẳng vào truy xuất dữ liệu/ giá trị) trên con trỏ này, mà bắt buộc phải sử dungj các hàm chuyên dụng của kernel mỗi khi có nhu cầu truy xuất vào vùng nhớ mà con trỏ đó đang trỏ tới.


Để tương tác an toàn với các biến mang nhãn `__user`, kernel cung cấp một bộ các hàm chuyên dụng, bao gồm:
- `copy_from_user()`: Copy dữ liệu từ user space sang kernel space.
- `copy_to_user()`: Copy dữ liệu từ kernel space sang user space.
```c
unsigned long copy_from_user(void *to, const void __user *from, unsigned long n)
unsigned long copy_to_user(void __user *to, const void *from, unsigned long n)
```
Trong cả hai trường hợp, những con trỏ có tiền tố `__user` là các con trỏ trỏ tới vùng nhớ của không gian người dùng (loại vùng nhớ không đáng tin cậy - untrusted). n đại diện cho số lượng byte cần được copy chép đi. Tham số from đại diện cho địa chỉ nguồn, và to là địa chỉ đích dể nhận dữ liệu. Cả hai hàm này đều trả về số lượng byte đã KHÔNG THỂ copy thành công. Trong trường hợp mọi thứ hoàn toàn suôn sẻ, giá trị trả về của 2 hàm sẽ là số 0.

Cần hết sức lưu ý rằng đối với hàm `copy_to_user()`, nếu có một số lượng dữ liệu nào đó không thể copy thành công ra user space, hàm này sẽ tự động lấp đầy phần dung lượng bị hụt đó (pad data) cho đủ với kích thước mà bạn đã yêu cầu (số n kia kìa) bằng các byte 0.



### The open method
### The release method
### The write method
### The read method
### The llseek method
### The poll method
### The ioctl method
### Filling the file operations structure