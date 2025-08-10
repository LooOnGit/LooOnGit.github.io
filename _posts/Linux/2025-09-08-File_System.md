---
title: "File System"
date: 2025-08-09 14:06:00 +0800
categories: [Linux]
tags: [Linux]
---
# 🐧 Linux File Management

## 1. 📂 **Tổng quan về File trên Linux**
- Linux quản lý tất cả mọi thứ như một file.

### 📑 Các loại file trên Linux

- **📄 Regular file**: là các file thông thường như text file, executable file.
- **📁 Directories file**: file chứa danh sách các file khác.
- **🖴 Character Device file**: file đại diện cho các thiết bị không có địa chỉ vùng nhớ.
- **💽 Block Device file**: file đại diện cho các thiết bị có địa chỉ vùng nhớ.
Dùng lệnh `ls /dev/` để xem các thiết bị đang connect
- **🔗 Link files**: file đại diện cho một file khác. Giống kiểu shortcut.
- **🔌 Socket file**: file đại diện cho 1 socket.
- **📦 Pipe file**: file đại diện cho 1 pipe.

### Hiển thị thông tin về file
Dùng lệnh `ls -l` tính theo byte
![alt text](/assets/Linux/file_system/ls_l.png)

- **Trường đầu tiên:** loại file, quyền file.
- **Trường thứ 2:** số  hardlink của file.
- **Trường thứ 3:** tên User.
- **Trường thứ 4:** tên group.
- **Truờng thứ 5:** kích thước file (bytes).
- **Trường thứ 6:** thời điêm edit file lần cuối.
- **Trường thứ 7:** tên file.

Dùng lệnh `ls -lh` tính theo kbyte.
![alt text](/assets/Linux/file_system/ls_lh.png)

### User
user `root` là user có quyền cao nhết trong hệ thống.
![alt text](/assets/Linux/file_system/user_root.png)

ở đây khi chạy `sudo su` thì nó chuyển về user root từ user loo

### Hardlink của file
là số  file cùng trỏ đến 1 vùng nhớ.

### Tên group
- file có `-` trước thì là đại diện cho regular file.
![alt text](/assets/Linux/file_system/regular_file.png)

- file có `d` đầu tiên là directory file.
![alt text](/assets/Linux/file_system/directory_file.png)

- file có  `c` charactor device file.
- file có `b` là block device file.
- file có `l` là link files.
- file có `s` là file socket.
- file có `p` là pipe file.

Quyền truy cập file gồm 3 nhóm:
1. **User permission** (quyền của chủ sở hữu file)
2. **Group permission** (quyền của nhóm sở hữu file)
3. **Others permission** (quyền của những người khác mà không phải là root)

Mỗi quyền có 3 loại:
- **r** → read (đọc) → giá trị 4
- **w** → write (ghi) → giá trị 2
- **x** → execute (thực thi) → giá trị 1

**Command**
`chmod [u/g/o/a][+/-/=][r/w/x] file`
- u → user (chủ sở hữu file)
- g → group (nhóm)
- o → others (người khác)
- a → all (tất cả: u+g+o)
- chmod → thay đổi quyền
- chown → thay đổi user, group


**Ví dụ:**
- nếu có `rw` thì là 6 vì 0b110 hệ nhị phân 1 (r) 1 (w) 0 (x).

**Ví dụ:**
![alt text](/assets/Linux/file_system/group_file.png)
regular file có rw-rw-r-- là có quyền rw của `user permission`, rw thứ 2
 có quyền rw của `Group permission`, rw thứ 3 chỉ có quyền r của `others permission`.

**Ví dụ:** dùng lệnh
```bash
chmod u+x file.txt     # Thêm quyền execute cho user
chmod g-w file.txt     # Gỡ quyền write cho group
chmod o=r file.txt     # Chỉ cho others quyền read
chmod a+rw file.txt    # Cho tất cả quyền đọc và ghi
```

**Ví dụ:** đổi group
```bash
chown username file.txt          # đổi user
chown username:groupname file.txt # đổi user và group
```

**Ví dụ**: thực tế
```bash
chmod 644 file.txt  # user: rw-, group: r--, others: r--
chmod 600 file.txt  # chỉ user có quyền đọc/ghi
chmod 700 script.sh # chỉ user có quyền đọc/ghi/thực thi
chmod u=rw,g=r,o= file.txt # user: rw-, group: r--, others: ---
```
## 2. 📝 **Đọc ghi File trong Linux**
Kernel cung cấp một bộ **system call** cơ bản để thực hiện việc đọc ghi và thao tác với file, bao gồm:
- `open()`
- `read()`
- `write()`
- `lseek()`
- `close()`

### Cú pháp các hàm:

```c
int open(const char *pathname, int flags, mode_t mode);

ssize_t read(int fd, void *buffer, size_t count);

ssize_t write(int fd, void *buffer, size_t count);

off_t lseek(int fd, off_t offset, int whence);

int close(int fd);
```
### Tại sao gọi là system call
![alt text](/assets/Linux/file_system/system_call.png)

Có 2 không gian là không gian là `user_space` và `kernel_space` để ghi được nội dung vào 1 file thì `c lib` không có khả năng vì vậy nó gọi `system call` để  làm việc đó để  chuyển đổi từ `user space` qua `kernel space`.

**Note**: ưu tiên dùng `system call` hạn chế  dùng lib như fopen của thư viện C để chạy nhanh đỡ phải từ `user space` sang `kernel space`.


## 3. 🗃️ **Quản lý File trên Linux**
Kernel điều khiển việc tương tác giữa các tiến trình và fiile thông qua ba bảng:
- File descriptor table
- Open file table
- I-node table

![alt text](/assets/Linux/file_system/file_manager.png)

Mỗi phần tử trong `I-node table` đại diện cho 1 file và chứa thông tin 1 file.
`Open file table` khi mở file sẽ điền vào file này.


![alt text](/assets/Linux/file_system/system_tabel.png)
seek_set đưa con trỏ đến trong nội dung 3 lần giống bấm mũi tên di chuyển vị trí nội dung vậy.

### Ví dụ chương trình C sử dụng I/O system calls

```c
// C program to illustrate
// I/O system calls
#include <stdio.h>
#include <string.h>
#include <unistd.h>
#include <fcntl.h>

int main(void) {
    int fd;
    int numb_read, numb_write;
    char buf1[12] = "hello world\n";

    // assume foobar.txt is already created
    fd = open("hello.txt", O_RDWR | O_CREAT, 0667);
    if (-1 == fd)
        printf("open() hello.txt failed\n");

    numb_write = write(fd, buf1, strlen(buf1));
    printf("Write %d bytes to hello.txt\n", numb_write);

    lseek(fd, 2, SEEK_SET);
    write(fd, "AAAAAAAAAA", strlen("AAAAAAAAAA"));

    close(fd);

    return 0;
}
```
khi mà open thì chiều từ phải sang trái. Khi đọc và ghi thì ngược lại(file descriptor -> Open file table -> I-node table)

# Quản lý file trong Kernel: File Descriptor Table, Open File Table và I-node Table

## 1. Giới thiệu
Trong hệ điều hành kiểu Unix/Linux, khi một tiến trình mở một file (`open()`), kernel cần biết:
- Tiến trình nào đang mở file gì.
- Trạng thái đọc/ghi đến đâu.
- Quyền truy cập file.
- Thông tin vật lý của file trên ổ đĩa.

Để quản lý điều đó một cách hiệu quả, kernel sử dụng **ba bảng** liên kết với nhau:

---

## 2. Ba bảng quản lý file

### 2.1. File Descriptor Table (per-process table)
- **Mỗi tiến trình** có **một bảng riêng** lưu các **file descriptor** (FD).
- Mỗi FD là một số nguyên (0, 1, 2, …) trỏ tới **một entry** trong **Open File Table**.
- 3 FD mặc định khi tạo tiến trình:
  - `0` → **stdin** (standard input)
  - `1` → **stdout** (standard output)
  - `2` → **stderr** (standard error)
- Cho phép nhiều FD khác nhau trỏ cùng một entry (vd: khi dùng `dup()`).

**Ví dụ:**

| FD | trỏ tới entry trong Open File Table |
|----|-------------------------------------|
| 0  | 5                                   |
| 1  | 6                                   |
| 2  | 7                                   |
| 3  | 8                                   |

---

### 2.2. Open File Table (system-wide, shared)
- Lưu thông tin về **một lần mở file**.
- Có thể được **chia sẻ giữa nhiều tiến trình** (vd: khi fork, hoặc khi dup FD).
- Mỗi entry chứa:
  - **File offset** (đang đọc/ghi tới đâu).
  - **Cờ** (O_RDONLY, O_WRONLY, O_RDWR, O_APPEND...).
  - **Pointer đến I-node Table** của file tương ứng.

**Ví dụ:**

| Offset | Mode     | trỏ tới I-node entry |
|--------|----------|----------------------|
| 1024   | O_RDWR   | 15                   |

---

### 2.3. I-node Table (system-wide, shared)
- Lưu thông tin **về bản thân file** trên đĩa.
- Thông tin gồm:
  - Kích thước file.
  - Quyền truy cập (read/write/execute).
  - UID/GID.
  - Thời gian tạo/sửa.
  - Con trỏ đến các block dữ liệu trên đĩa.

**Ví dụ:**

| File size | Permissions | UID | Blocks ... |
|-----------|-------------|-----|------------|
| 4 KB      | rw-r--r--   | 1000| ...        |

---

## 3. Mối quan hệ giữa 3 bảng
```
[File Descriptor Table của tiến trình]
         ↓
[Open File Table (thông tin mở file)]
         ↓
[I-node Table (thông tin file trên đĩa)]
```

**Ví dụ minh họa:**
1. Tiến trình **A** gọi `fd = open("test.txt", O_RDWR)`
2. Kernel tạo:
   - Một **entry** trong Open File Table (offset=0, mode=O_RDWR).
   - Entry này trỏ tới I-node entry của "test.txt".
   - Trong File Descriptor Table của tiến trình A, gán `fd=3` trỏ tới entry vừa tạo.
3. Nếu A gọi `dup(3)` → FD mới (vd: 4) cũng trỏ **cùng entry** trong Open File Table → offset được chia sẻ.
4. Nếu tiến trình **B** mở `"test.txt"` → kernel có thể tạo **entry Open File Table riêng** nhưng **cùng trỏ** tới I-node Table.

---

## 4. Tóm tắt

| Bảng              | Phạm vi          | Chứa gì                              | Mục đích |
|-------------------|-----------------|---------------------------------------|----------|
| File Descriptor   | Mỗi tiến trình  | Map FD → Open File Table entry        | Liên kết FD của tiến trình với file |
| Open File Table   | Toàn hệ thống   | Offset, mode, pointer tới I-node      | Quản lý trạng thái mở file |
| I-node Table      | Toàn hệ thống   | Thông tin file vật lý trên đĩa        | Quản lý metadata của file |


Một process có thể có nhiều FDs cùng tham chiếu vào một vị trí trong OFD,
Sủ dụng dup(), dup2().
![alt text](/assets/Linux/file_system/file_manager_1.png)

Hai process cùng mở một file, tham chiếu tới cùng một OFD.
sử dụng fork().
![alt text](/assets/Linux/file_system/file_manager_2.png)

Hai process cùng mở một file, tham chiếu tới cùng một inode.
![alt text](/assets/Linux/file_system/file_manager_3.png)



## 4. 🔒 **File Locking**
## 5. ⚡ **Đọc ghi File bất đồng bộ**
