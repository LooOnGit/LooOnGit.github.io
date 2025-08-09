---
title: "File System"
date: 2025-08-09 14:06:00 +0800
categories: [Linux]
tags: [Linux]
---
# 🐧 Linux File Management

1. 📂 **Tổng quan về File trên Linux**
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

- Trường đầu tiên: loại file, quyền file.
- Trường thứ 2: số  hardlink của file.
- Trường thứ 3: tên User.
- Trường thứ 4: tên group.
- Truờng thứ 5: kích thước file (bytes).
- Trường thứ 6: thời điêm edit file lần cuối.
- Trường thứ 7: tên file.

Dùng lệnh `ls -lh` tính theo kbyte.
![alt text](/assets/Linux/file_system/ls_lh.png)

### User
user `root` là user có quyền cao nhết trong hệ thống.
![alt text](/assets/Linux/file_system/user_root.png)

ở đây khi chạy `sudo su` thì nó chuyển về user root từ user loo

2. 📝 **Đọc ghi File trong Linux**

3. 🗃️ **Quản lý File trên Linux**
4. 🔒 **File Locking**
5. ⚡ **Đọc ghi File bất đồng bộ**
