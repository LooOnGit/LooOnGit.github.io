---
title: "GPIO Subsystem"
date: 2025-05-17 06:20:00 +0700
categories: [Linux]
tags: [Linux]
---

# 📘 GPIO Subsystem

Bài viết này đóng vai trò là bản đồ (roadmap) để tìm hiểu về kiến trúc và cách hoạt động của các **Subsystem (Hệ thống con)** trong Linux Kernel, đặc biệt tập trung vào góc độ lập trình Device Driver cho hệ thống nhúng (Embedded Linux).

---

## 1. 🌟 Tổng quan về Linux Subsystem
- **Subsystem là gì?** Khái niệm và mục đích chia nhỏ Kernel thành các Subsystem (phân quyền quản lý, tái sử dụng code, chuẩn hóa API).
- **Phân tầng kiến trúc:** Sự phân tách giữa *Hardware-specific layer* (tầng giao tiếp phần cứng), *Core layer* (tầng lõi xử lý logic chung) và *User-space interface layer* (tầng cung cấp giao diện cho người dùng).
- **So sánh:** Viết driver trực tiếp (Raw Character Driver) vs Viết driver thông qua một Subsystem.

## Interget Based and Descriptor Based
### Interget Based

### Descriptor Based

