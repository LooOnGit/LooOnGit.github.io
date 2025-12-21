---
title: "Developing The Linker Script And Startup File"
date: 2025-12-21 08:00:09 +0800
categories: [Bare Metal STM32]
tags: [Bare Metal STM32]
---

# 🛠️ Developing The Linker Script And Startup File

## Understanding the STM32 Memory model
Trong **STM32** có nhiều loại bộ nhớ khác nhau, để tập trung tìm hiểu linker script và startup file, ta sẽ xem xét bộ nhớ Flash và RAM.


**Memory density**: Thường thế hiện số bit hoặc byte trên mỗi đơn vị của vùng nhớ vật lý. Ví dụ,số bit trên millimeter hoặc số byte trên centimeter.


**Memory size**: Tổng dung lượng của memory. Chẳng hạn như **KB**, **MB** hay **GB**.

## Understanding the Linker processing
Trong quá trình build, bước liên kết (linking) file object là bước rất quan trọng để tạo ta firmware hoàn chỉnh có thể chạy được. Ví dụ 1 value có thể dùng ở nhiều file thì linker này kết hợp lại với nhau. 

### Section attributes and their implications
Mỗi section trong một file object được xác định bởi **unique name** và **size**.
- **Loadable sections**: Chưa nội dung đã được load vào memory ở runtime. Đóng vai trò cần thiết cho các việc thực thi chương trình, bao gồm chương trình thực thi và khai báo data. Ví dụ như **.text** và **.data**.
- **Allocated sections**: Section này thì không chứa nội dung dữ liệu. Thay vào đó, chúng cho biết rằng một vùng nhớ cần được reserved(dành riêng), thường dành cho dữ liệu chưa được khởi tạo và sẽ được xác định trong quá trình runtime. Ví dụ như **.bss**.
- **Non-allocated, Non-loadable sections**: Section này không thuộc 2 loại trên mà chỉ chứa thông tin debug hoặc metadata. Nó không cần thiết trong giai đoạn runtime.

## Understanding the Startup File