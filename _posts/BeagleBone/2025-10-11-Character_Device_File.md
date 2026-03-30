---
title: "CHARACTER DEVICE FILE"
date: 2025-11-09 19:29:00 +0700
categories: [BeagleBone]
tags: [BeagleBone]
---

# 🐧 CHARACTER DEVICE FILE

## Device file
Các thiết bị như usb, ổ cứng, ... .Đại diện trong linux dưới dạng là file gọi là **device file**.

Dùng `lsblk` để search ra.

![alt text](/assets/BeagleBone/Character_device_file/img1.png)

Để tạo ra **device file** cần bước tạo ra cặp số **device number**. **Device file** đại diện cho phần cứng **device number** đại diện cho **device file**.

![alt text](/assets/BeagleBone/Character_device_file/img2.png)
có 2 cột số là cặp số của **device number**.

1. **Major number**: Là số xác định liên kết giữa driver và device. Một major number có thể được chia sẻ giữa nhiều device driver.
2. **Minor number**: Là số để phân biệt các thiết bị vật lý riêng lẻ.

## Cấp phát tĩnh và cấp phát động cho device number
Để cấp phat **device number** có 2 cách cấp phát tĩnh và cấp phát động.

### Cấp phát động
```c
struct m_foo_dev
    {
        dev_t dev_num; /* dev_t dev = MKDEV(170, 0) */
    }mdev;

if(alloc_chrdev_region(&mdev.dev_num, 0, 1, "m-cdev") < 0){
    pr_err("Failed to alloc chrdev region\n");
    return -1;
}
```
`alloc_chrdev_region`có đối số thứ 2 là số bắt đầu từ số bao nhiêu và đối số thứ 3 là số lượng được cấp như mẫu trên thì bắt đầu từ 0 và số lượng là 4 là `0 1 2 3` ví dụ cho major number là 255 thì minor được cấp là `0 1 2 3`.

### Cấp phát tĩnh
Phải dò tìm device number chưa được dùng rồi cấp phát.
```c
registerZ_chrdev_region(&mdev.dev_num, 1, "m-cdev");
```

## Thực thi và kiểm tra
```bash
looonlinux@looonlinux-VMware-Virtual-Platform:~/word/Linux_Learn/major-minor$ sudo dmesg | tail
[14319.336803] audit: type=1400 audit(1763194163.595:206): apparmor="DENIED" operation="sendmsg" class="net" profile="/snap/snapd/25202/usr/lib/snapd/snap-confine" pid=5243 comm="snap-confine" family="unix" sock_type="stream" protocol=0 requested_mask="send" denied_mask="send"
[14319.578886] audit: type=1400 audit(1763194163.837:207): apparmor="DENIED" operation="sendmsg" class="net" profile="/snap/snapd/25202/usr/lib/snapd/snap-confine" pid=5270 comm="snap-confine" family="unix" sock_type="stream" protocol=0 requested_mask="send" denied_mask="send"
[14319.578891] audit: type=1400 audit(1763194163.837:208): apparmor="DENIED" operation="sendmsg" class="net" profile="/snap/snapd/25202/usr/lib/snapd/snap-confine" pid=5270 comm="snap-confine" family="unix" sock_type="stream" protocol=0 requested_mask="send" denied_mask="send"
[14319.765476] audit: type=1400 audit(1763194164.023:209): apparmor="DENIED" operation="sendmsg" class="net" profile="/snap/snapd/25202/usr/lib/snapd/snap-confine" pid=5297 comm="snap-confine" family="unix" sock_type="stream" protocol=0 requested_mask="send" denied_mask="send"
[14570.860381] workqueue: psi_avgs_work hogged CPU for >10000us 7 times, consider switching to WQ_UNBOUND
[15125.348140] workqueue: blk_mq_run_work_fn hogged CPU for >10000us 4 times, consider switching to WQ_UNBOUND
[15273.317848] workqueue: blk_mq_run_work_fn hogged CPU for >10000us 5 times, consider switching to WQ_UNBOUND
[15392.112571] exam: loading out-of-tree module taints kernel.
[15392.112575] exam: module verification failed: signature and/or required key missing - tainting kernel
[15392.129944] Hello World! Major number allocated: 240
```
Cấp phát major là 240 đó, tiếp theo chạy
```bash
cat /proc/devices
```
thì tìm 240 thôi.
```bash
ls /dev/ | grep 240
```
Tới bước này rồi thì check không ra gì hết bởi vì chỉ mới tạo ra **device number** thôi chưa tạo **device file**.

## Tạo device file
### Tạo struct 
Để tạo class thì có struct class.
```c
struct m_foo_dev {
        dev_t dev_num;
        struct class *m_class;
    }
```
### Tạo struct clas
```c
if((mdev.m_class = class_create(THIS_MODULE, "m_class")) == NULL)
{
    pr_err("Cannot create the struct class for my device\n");
    goto rm_device_numb;
}
```
`THIS_MODULE` để chỉ thị rằng module đang viết thuộc class là `m_class`.

### Tạo device file
```c
if(device_create(mdev.m_class, NULL, mdev.dev_num, NULL, "m_device")) == NULL)
{
    pr_err("Cannot create my device\n");
    goto rm_class;
}
```
Cấp device number cho device file.

Khi destroy mấy function thì nào tạo ra sau thì phải xóa trước trong destroy.
```c
/* Destructor */
static void  __exit chdev_exit(void)
{
    device_destroy(mdev.m_class, mdev.dev_num);             /* 3.0 */
    class_destroy(mdev.m_class);                            /* 2.0 */
    unregister_chrdev_region(mdev.dev_num, 1);              /* 1.0 */
    pr_info("Goodbye\n");
}
```
cái class được tạo cuối cùng.

Chạy file thực thi và check kết quả:
![alt text](/assets/BeagleBone/Character_device_file/result_class.png)

Bước tiếp theo để đọc ghi dữ liệu từ thiết bị thì phải dùng device file để ghi xuống thanh ghi.

## Register File operation
- **struct file_operat** là một phần tử của struct cdev.
- Định nghĩa toàn bộ các hoạt động của file (open/read/write/close/mmap/ioctl).
![alt text](/assets/BeagleBone/Character_device_file/file_operation.png)

- Struct cdev là một phần tử của struct inode.
- Là struct đại điện cho character device.
![alt text](/assets/BeagleBone/Character_device_file/struct_cdev.png)

Để đọc ghi được vào device file thì phải đăng kí read/write, ...
### Các bước để thêm file operation
#### 1. Tạo protype cho các hàm
```c
#include <linux/cdev.h>     /* Define cdev_init(), cdev_add()*/
static int      m_open(struct inode *inode, struct file *file);
static int      m_release(struct inode *inode, struct file *file);
static ssize_t  m_read(struct file *filp, char __user *user_buf, size_t size,loff_t * offset);
static ssize_t  m_write(struct file *filp, const char *user_buf, size_t size, loff_t * offset);


/* This function will be called when we open the Device file */
static int m_open(struct inode *inode, struct file *file)
{
    pr_info("System call open() called...!!!\n");
    return 0;
}

/* This function will be called when we close the Device file */
static int m_release(struct inode *inode, struct file *file)
{
    pr_info("System call close() called...!!!\n");
    return 0;
}

/* This function will be called when we read the Device file */
static ssize_t m_read(struct file *filp, char __user *user_buf, size_t size, loff_t *offset)
{
    pr_info("System call read() called...!!!\n");
    return 0;
}

/* This function will be called when we write the Device file */
static ssize_t m_write(struct file *filp, const char __user *user_buf, size_t size, loff_t *offset)
{
    pr_info("System call write() called...!!!\n");
    return size;
}
```
#### 2. Tạo struct file operation
```c
static struct file_operations fops =
{
    .owner      = THIS_MODULE,
    .read       = m_read,
    .write      = m_write,
    .open       = m_open,
    .release    = m_release,
};
```
Chỉ các tên field `.open`, `.read`, `.write`, `.release` là bắt buộc đúng—tên hàm thì tùy bạn.
#### 3. Đăng ký thiết bị (register char device)
```c
/* 4.0 Creating cdev structure */
cdev_init(&mdev.m_cdev, &fops);

/* 4.1 Adding character device to the system */
if ((cdev_add(&mdev.m_cdev, mdev.dev_num, 1)) < 0) {
    pr_err("Cannot add the device to the system\n");
    goto rm_device;
}
```
#### 4. Gỡ đăng ký khi rmmod (module_exit)
```c
cdev_del(&mdev.m_cdev);
```

#### 5. Biên dịch và chạy
![alt text](/assets/BeagleBone/Character_device_file/echo.png)
Có thể thấy echo sẽ ghi vào thì nó gọi ra systemcall như `open`, `write`, `close`.
![alt text](/assets/BeagleBone/Character_device_file/echo.png)
Có thể thấy cat sẽ ghi vào thì nó gọi ra systemcall như `open`, `read`, `close`.

### Write và Read đã đọc ghi được sao lại thêm ioctl
