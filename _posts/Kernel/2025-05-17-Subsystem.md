---
title: "Subsystem"
date: 2025-05-17 06:20:00 +0700
categories: [Linux]
tags: [Linux]
---

# 📘 Subsystem

Bài viết này đóng vai trò là bản đồ (roadmap) để tìm hiểu về kiến trúc và cách hoạt động của các **Subsystem (Hệ thống con)** trong Linux Kernel, đặc biệt tập trung vào góc độ lập trình Device Driver cho hệ thống nhúng (Embedded Linux).

---

## 1. 🌟 Tổng quan về Linux Subsystem
- **Subsystem là gì?** Khái niệm và mục đích chia nhỏ Kernel thành các Subsystem (phân quyền quản lý, tái sử dụng code, chuẩn hóa API).
- **Phân tầng kiến trúc:** Sự phân tách giữa *Hardware-specific layer* (tầng giao tiếp phần cứng), *Core layer* (tầng lõi xử lý logic chung) và *User-space interface layer* (tầng cung cấp giao diện cho người dùng).
- **So sánh:** Viết driver trực tiếp (Raw Character Driver) vs Viết driver thông qua một Subsystem.

## 2. 🏗️ Trái tim của Subsystem: Linux Device Model (LDM)
Để hiểu cách các subsystem hoạt động, bắt buộc phải nắm được cách Kernel quản lý các object.
- **Kobject, Kset và Ktype:** Cấu trúc nền tảng định hình nên cây thư mục object trong Kernel.
- **Tứ trụ của LDM:** 
  - **Bus:** Đường truyền logic kết nối thiết bị và driver (VD: `platform_bus`, `i2c_bus`).
  - **Device:** Đại diện cho thiết bị vật lý hoặc ảo.
  - **Driver:** Đoạn code điều khiển thiết bị.
  - **Class:** Phân loại thiết bị theo chức năng (VD: `input`, `tty`, `graphics`) bất kể chúng nằm trên bus nào.
- **Sysfs (`/sys/`):** Cách LDM xuất (export) thông tin của subsystem ra môi trường user-space.

## 3. 🔍 Tìm hiểu các Subsystem phổ biến (Embedded Focus)
Tập trung vào các subsystem thường xuyên làm việc nhất khi viết driver cho board nhúng:

### 3.1. I2C & SPI Subsystem
- **Kiến trúc Core:** `i2c-core`, `spi-core`.
- **Phân tách trách nhiệm:**
  - *Adapter/Master Driver:* Driver điều khiển bộ I2C/SPI controller của SoC.
  - *Client/Device Driver:* Driver điều khiển con chip ngoại vi (sensor, eeprom) cắm vào bus.

### 3.2. GPIO & Pinctrl Subsystem
- **Pinctrl Subsystem:** Quản lý việc cấu hình chân (Pin Muxing) và thuộc tính điện (pull-up, pull-down).
- **GPIO Subsystem:** API mới (Descriptor-based `gpiod_*`) so với API cũ (Integer-based `gpio_*`).

### 3.3. Input Subsystem
- Dùng cho các thiết bị đầu vào: Nút nhấn, bàn phím, chuột, màn hình cảm ứng.
- **Các thành phần:** `input_dev`, Event handler (`evdev`, `joydev`), `/dev/input/eventX`.

### 3.4. IIO (Industrial I/O) Subsystem
- Subsystem chuyên dụng cho các loại cảm biến đo lường: ADC, DAC, Accelerometer (Gia tốc), Gyroscope (Con quay hồi chuyển).
- Cơ chế hoạt động: Kênh (Channels), Bộ đệm (Ring Buffer), và Trigger.

### 3.5. Một số Subsystem quan trọng khác
- **Regulator Subsystem:** Quản lý nguồn điện cấp cho các thiết bị (LDO, Buck, Boost).
- **Clock (CCF - Common Clock Framework):** Quản lý xung nhịp (clock tree) của toàn hệ thống.
- **Interrupt (IRQ) Subsystem:** Quản lý ngắt cứng, ngắt mềm, `request_irq()`, Top-half/Bottom-half.

## 4. 🛠️ Cách viết Driver tích hợp vào một Subsystem
- **Đăng ký (Registration):** Cách đăng ký một driver với subsystem (VD: `module_i2c_driver`, `input_register_device`).
- **Ops Structures:** Triển khai các con trỏ hàm (callbacks) do subsystem yêu cầu.
- **Event Handling:** Cách gửi dữ liệu từ phần cứng lên cho subsystem xử lý để đưa ra User-space.

## 5. 🌳 Mối liên kết giữa Subsystem và Device Tree (DTS)
- Cách subsystem đọc và phân tích cú pháp (parse) các thuộc tính từ file `.dts`.
- Ánh xạ tự động từ Device Tree node thành các thiết bị logic (logical devices) để giao cho Subsystem quản lý.