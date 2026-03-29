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
### So sánh `struct inode` và `struct file` trong Linux Kernel

Hai cấu trúc dữ liệu cơ bản nhưng đóng vai trò hoàn toàn khác nhau trong việc quản lý tập tin của Linux:

| Tiêu chí | `struct inode` (Index Node) | `struct file` (Open File) |
| :--- | :--- | :--- |
| **Định nghĩa** | Đại diện cho **bản thân tập tin tĩnh** nằm trên thiết bị lưu trữ. | Đại diện cho một **tập tin đang được mở** bởi một tiến trình (process). |
| **Số lượng tồn tại** | Chỉ có **MỘT** `inode` duy nhất cho mỗi tập tin trên đĩa. | Có thể có **NHIỀU** `struct file` đại diện cho cùng một tập tin (nếu nó được mở nhiều lần bởi các tiến trình khác nhau). |
| **Nội dung lưu trữ** | Chứa thông tin về hệ thống tập tin (loại file, vị trí block trên đĩa cứng, quyền truy cập, chủ sở hữu, kích thước...). | Chứa các thông tin về trạng thái mở (vị trí con trỏ đọc/ghi `f_pos`, chế độ mở read/write,... và con trỏ `f_inode` trỏ ngược về `inode`). |
| **Theo dõi vị trí (Cursor)** | **Không** theo dõi vị trí đọc/ghi hiện tại bên trong tập tin. | **Có** theo dõi vị trí đọc/ghi hiện tại (thông qua trường `f_pos`). Mỗi lệnh read/write sẽ làm thay đổi giá trị này. |
| **Chức năng thao tác** | Cung cấp cho OS cách tìm kiếm nội dung vật lý dưới ổ đĩa. | Cung cấp các thao tác (operations) như open, read, write, seek... thông qua con trỏ `f_op` (file_operations). |
| **Khi nào được tạo ra?** | Khi tập tin được tạo/tồn tại trên thiết bị lưu trữ. | Khi người dùng (user space) gọi lệnh hệ thống (system call) `open()`. |
| **Khi nào bị xóa?** | Khi tập tin đó bị xóa cứng (hard link count = 0) khỏi ổ đĩa. | Bị hủy khỏi RAM khi gọi lệnh `close()`. |

**Tóm lại (Triết lý Everything is a file):**
* `struct inode`: Chính là "cái xác" (dữ liệu phần cứng) của tập tin nằm ở dưới kernel/ổ cứng.
* `struct file`: Chính là "linh hồn" (trạng thái mở) của tập tin khi nó đang được tương tác/sử dụng bởi các chương trình. Nhiều `struct file` có thể cùng tham chiếu về một `struct inode` duy nhất.

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