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
#### A single value copy
Khi copy một biến đơn giản như **char** và **int**, nhưng không type quá lớn như **struct** và **array**, kernel cung cấp macro để thực hiện như `put_user(x, ptr)` và `get_user(x, ptr)`:
- `put_user(x, ptr);`: Macro copy một variable từ kernel space tới user space. c đại diện cho giá trị copy từ user space, và `ptr` là address đích nằm trong user space. Return 0 là success, hoặc `-EFAULT` là error. Cùng kiểu dữ liệu.
- `get_user(x, ptr);`: Copy variable từ user space đến kernel space và ruen 0 nếu sucess hoặc `EFAULT` nếu error.
### The open method
`open` method được gọi khi ai đó muốn open device file. Device open luôn luôn thành công trong trừng hợp phương thức này chưa được define. Thông thường sử dụng phương thức này để thực hiện khởi tạo device và data structure, và return negative error code nếu trả về 0 thành công.
```c
int (*open)(struct inode *inode, struct file *filp);
```
#### Per-device data
`release` method được gọi khi device được closed, ngược lại hoàn tòn với method `open`. Bạn phải undeo tất cả những gì đã thực hiện trong bước open.
- 1. Free bất kỳ private memory cấp phát trong suốt bước `open()`.
- 2. Shut down device (nếu support) và discard mọi thứ buffer khi lần đóng cuối cùng (nếu device support multi opening, hoặc driver handle nhiều hơn một device).
```c
static int sram_release(struct inode *inode, struct file *filp)
{
    struct pcf2127 *pcf = NULL;
    pcf = container_of(inode->i_cdev, struct pcf2127, cdev);

    mutex_lock(&device_list_lock);
    filp->private_data = NULL;

    /* last close? */
    pcf2127->users--;
    if (!pcf2127->users) {
        kfree(tx_buffer);
        kfree(rx_buffer);
        tx_buffer = NULL;
        rx_buffer = NULL;
        [...]
        if (any_global_struct)
            kfree(any_global_struct);
    }

    mutex_unlock(&device_list_lock);
    
    return 0;
}   
```
### The write method
`write()` method sử dụng để send data đến device, bất cứ khi nào một user app call `write` function trên device file, kernel thực hiện trong kernel.
```c
ssize_t(*write)(struct file *filp, const char __user *buf, size_t count, off_t *pos);
```
- return value là number là số byte đã written.
- `*buf` đại diện cho data buffer từ user space.
- `count` size của dữ liệu yêu cầu transfer.
- `*pos` Chỉ định vị trí bắt đầu mà data được written vào trong file.
#### Steps to write
Những thao tác có thể thực hiện trong phương thức này:
- 1. Kiểm tra các yêu cầu bad hoặc invalid đến từ user space. Bước này chỉ có ý nghĩa nếu device để lộ (expose) memory của nó  (eeprom, I/O memory, v.v) nó là những thành phần giới hạn sixe.
```c
/* if trying to Write beyond the end of the file, return error.
* "filesize" here corresponds to the size of the device memory (if any)
*/
if ( *pos >= filesize ) return -EINVAL;
```
- 2. Điều chỉnh `count` trên số byte còn lại không vượt quá file size. Bước này không bắt buộc, và điều kiện của nó giống step 1:
```c
/* filesize coerresponds to the size of device memory */
if (*pos + count > filesize)
    count = filesize - *pos;
```
- 3. Tìm ví trí nó start để write. Step này có ý nghĩa nếu device có memory mà `write()` được yêu cầu để write data vào. Tương tự step 1, 2 không bắt buộc:
```c
/* convert pos into valid address */
void *from = pos_to_address( *pos );
```
- 4. Copy data từ user space và write nó vào kernel space:
```c
if (copy_from_user(dev->buffer, buf, count) != 0){
    retval = -EFAULT;
    goto out;
}
/* now move data from dev->buffer to physical device */
```
- 5. Write vào physical device và return một error nếu thất bại:
```c
write_error = device_write(dev->buffer, count);
if ( write_error )
    return -EFAULT;
```
- 6. Tăng vị trí hiện tại của cursor trong file, theo number byte đã ghi. Cuối cùng, return number byte đã copy:
```c
*pos += count;
return count;
```


**Example**: 
```c
ssize_t eeprom_write(struct file *filp, const char __user *buf, size_t count, loff_t *f_pos)
{
    struct eeprom_dev *eep = filp->private_data;
    ssize_t retval = 0;

    /* step (1) */
    if (*f_pos >= eep->part_size)
    /* Writing beyond the end of a partition is not allowed. */
    return -EINVAL;

    /* step (2) */
    if (*pos + count >= eep->part_size)
    count = eep->part_size - *pos;

    /* step (3) */
    int part_origin = PART_SIZE * eep->part_index;
    int register_address = part_origin + *pos;

    /* step(4) */
    /* Copy data from user space to kernel space */
    if (copy_from_user(eep->data, buf, count) != 0)
        return -EFAULT;

    /* step (5) */
    /* perform the write to the device */
    if (write_to_device(register_address, buff, count) < 0){
        pr_err("ee24lc512: i2c_transfer failed\n");
        return -EFAULT;
    }
    /* step (6) */
    *f_pos += count;
    return count;
}
```
### The read method
Prototype cho `read()` method:
```c
ssize_t (*read) (struct file *filp, char __user *buf, size_t count, loff_t *pos);
```
Return value là size read. 
- `*buf`: là buffer mà nhận được từ user space.
- `count`: là size yêu cầu transfer (size của user buffer).
- `*pos`: Dữ liệu bắt đầu mà nên được đọc trong file.
#### Steps to read
- **1.** Ngăn chặn việc đọc vượt quá size, và return end-of-life:
```c
if (*pos >= filesize)
    return 0; /* 0 means EOF */
```
- **2.** Number of bytes có thể đọc không thể vượt quá file size. Điều chỉnh `count` cách hợp lý:
```c
if (*pos + count > filesize)
    count = filesize - (*pos);
```
- **3.** Tìm kiếm vị trí mà bắt đầu đọc:
```c
void *from = pos_to_address (*pos); /* convert pos into valid address */
```
- **4.** Copy data vào user space buffer và return một error nếu thất bại:
```c
sent = copy_to_user(buf, from, count);
if (sent)
    return -EFAULT;
```
- **5.** Tăng vị trí hiện tại lên tương ứng với số byte đã đọc, và return number byte đã copy:
```c
*pos += count;
return count;
```


**Example**:
```c
ssize_t eep_read(struct file *filp, char __user *buf, size_t count, loff_t *f_pos)
{
    struct eeprom_dev *eep = filp->private_data;
    
    if (*f_pos >= EEP_S IZE) /* EOF */
        return 0;
    
    if (*f_pos + count > EEP_SIZE)
        count = EEP_SIZE - *f_pos;
    
    /* Find location of next data bytes */
    int part_origin = PART_SIZE * eep->part_index;
    int eep_reg_addr_start = part_origin + *pos;
    
    /* perform the read from the device */
    if (read_from_device(eep_reg_addr_start, buff, count) < 0){
        pr_err("ee24lc512: i2c_transfer failed\n");
        return -EFAULT;
    }

    /* copy from kernel to user space */
    if(copy_to_user(buf, dev->data, count) != 0)
        return -EIO;

    *f_pos += count;
    return count;
}
```
### The llseek method
`llseek` function thì được gọi khi bạn move cursor trong file. Entry point phương thức này ở user space là `lseek()` system call. Xem ở man page.
```c
loff_t(*llseek) (struct file *filp, loff_t offset, int whence);
```
Tham số:
- return value là new position trong file.
- `loff_t` là offset, xác định vị trí sẽ bị thay đôi (di chuyển) bao nhiêu.
- `whence` xác định vị trí bắt đầu di chuyển: 
    - `SEEK_SET`: di chuyển từ đầu file.
    - `SEEK_CUR`: di chuyển từ vị trí hiện tại.
    - `SEEK_END`: di chuyển từ cuối file.
### The poll method
### The ioctl method
### Filling the file operations structure