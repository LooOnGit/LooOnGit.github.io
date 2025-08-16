---
title: "SEMAPHORE"
date: 2025-08-16 08:39:00 +0800
categories: [RTOS]
tags: [RTOS]
---

# 🔒 Semaphore trong RTOS

## 💡 Semaphore là gì?

Semaphore là một cơ chế đồng bộ hóa được sử dụng trong RTOS để:
- 🛡️ Kiểm soát truy cập vào tài nguyên dùng chung
- 🔄 Quản lý đồng bộ hóa tác vụ
- Như một hàng đợi (Queue), các task có yêu cầu sử dụng tài nguyên sẽ đựợc xếp vào hàng đợi. Khi có Semaphore được give, thì task nào được xếp vào hàng đợi trước sẽ được sử dụng trước.
![alt text](/assets/RTOS/semaphore.png)

## 📑 Các loại Semaphore

1. 🔂 Bin Semaphoree
   - Chỉ có hai trạng thái (0 hoặc 1)
   - Tương tự mutex nhưng không hoàn toàn giống
   - Dùng cho đồng bộ hóa đơn giản

2. 🔢 Couting Semaphore
   - Có thể có nhiều trạng thái
   - Dùng để kiểm soát truy cập vào nhóm tài nguyên
   - Thường dùng trong các tình huống producer-consumer

## ⚙️ Nguyên lý hoạt động
- Task take semaphore sẽ bị block cho tới khi semaphore có sẵn.
- Khi Task hay hàm xử  lý ngắt Give Semaphore, thì sẽ làm tăng số lượng Semaphore lên.

## 🎯 Các Trường hợp Sử dụng Phổ biến

1. 📦 Quản lý Tài nguyên
   - Bảo vệ tài nguyên dùng chung
   - Quản lý nhóm tài nguyên

2. 🔄 Đồng bộ hóa Tác vụ
   - Báo hiệu giữa các tác vụ
   - Xử lý sự kiện

3. 🏭 Producer-Consumer
   - Quản lý bộ đệm
   - Đồng bộ hóa hàng đợi dữ liệu

## 💻 Ví dụ Code

```c
sem_t semaphore;

// Khởi tạo
sem_init(&semaphore, 0, 1);

// Chờ
sem_wait(&semaphore);

// Phần găng
// ...

// Giải phóng
sem_post(&semaphore);

// Hủy
sem_destroy(&semaphore);
```