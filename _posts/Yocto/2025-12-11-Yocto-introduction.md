---
title: "Yocto introduction"
date: 2025-12-11 22:17:59 +0800
categories: [Yocto]
tags: [Yocto]
---

## 📖 Introduction

### 🏗️ Linux System Architecture
![Linux system architecture diagram](/assets/Yocto/image.png)

### 🚀 Overall Linux Boot Sequence
![Linux boot sequence diagram](/assets/Yocto/image-1.png)

### 💼 Embedded Linux Work
- 🔧 **BSP work**: porting the bootloader and Linux kernel, developing Linux device drivers
- 🔨 **System integration work**: assembling all the user space components needed for the system, configure them, develop the upgrade and recovery mechanisms, etc.
- 💻 **Application development**: write the company-specific applications and libraries

### ⚙️ Build System

#### 📋 Nguyên lý hoạt động (Principle)
![Build system principle diagram](/assets/Yocto/image-2.png)

- 📝 **Building from source** → lot of flexibility
- 🔄 **Cross-compilation** → leveraging fast build machines
- 📜 **Recipes** for building components → easy

#### 🛠️ Tools
A wide range of solutions: Yocto/OpenEmbedded, PTXdist, Buildroot, OpenWRT, and more.

- 🎯 **Yocto/OpenEmbedded**: Builds a complete Linux distribution with binary packages. Powerful, but somewhat complex, and quite steep learning curve.
- 🎯 **Buildroot**: Builds a root filesystem image, no binary packages. Much simpler to use, understand and modify.

### 🌟 Yocto Project

#### 📚 Từ khóa (Lexicon)

- ⚡ **BitBake** là một task **scheduler** giống **make**. Nó đọc configuration files và recipes (gọi là metadata) để set các task để thực hiện **download**, **configure** và build **applications** và **filesystem image**.

- 🏛️ **OpenEmbedded-Core**, set base layers. Bao gồm:
  1. 📦 **Recipes (.bb)**: công thức build phần mềm (busybox, bash, glibc, systemd…)
  2. 🧱 **Base Layers**: các lớp nền tảng
  3. 🧩 **Classes (.bbclass)**: luật build chung (cmake, autotools, kernel, image…)

- 🎁 **Poky**, the reference system. Nó đã bao gồm những project và các tools, bootstrap (khởi tạo) một distribution dựa trên Yocto Project.

![Yocto Project architecture diagram](/assets/Yocto/image-3.png)

- 📄 **Recipes** mô tả fetch, configure, compile và đóng gói application và images như thế nào. Có cú pháp đặc biệt.
- 📂 **Layers** để set các recipes, matching một mục tiêu chung.  
  **VD**: Của Texas Instruments board support, có sử dụng meta-ti layer.
- 🗂️ **Multiple layers** có nhiều layer trong 1 distribution, phụ thuộc vào yêu cầu.
- 🎯 **Architecture support**: ARM, MIPS (32 và 64 bits), PowerPC, RISC-V and x86 (32 and 64 bits)
- 💻 **QEMU support**: hỗ trợ cho QEMU (một trình giả lập)
