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
### File representation in the kernel
## Allocating and registering a character device
## Writing file operations
### Exchanging data between kernel space and user space
### The open method
### The release method
### The write method
### The read method
### The llseek method
### The poll method
### The ioctl method
### Filling the file operations structure