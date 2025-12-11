---
title: "Getting Started with Poky - Yocto Basics"
date: 2025-12-11 22:34:23 +0700
categories: [Yocto]
tags: [Yocto]
---

# 🎁 Poky

Poky là reference distribution của Yocto Project, bao gồm tất cả các tools và metadata cần thiết để build một Linux distribution.

![Poky architecture diagram](/assets/Yocto/image-4.png)

## 📂 Cấu trúc thư mục Poky

```bash
poky/ 
├── bitbake/              # Công cụ build chính của Yocto 
│ 
├── meta/                 # OpenEmbedded-Core (OE-Core) 
│   ├── conf/             # Cấu hình mặc định 
│   ├── recipes-core/     # Các recipe hệ thống cơ bản 
│   ├── recipes-devtools/ # Công cụ phát triển (gcc, make, gdb…) 
│   ├── recipes-kernel/   # Build Linux kernel 
│   └── recipes-support/  # Các gói hỗ trợ hệ thống 
│ 
├── meta-poky/            # Policy cấu hình mặc định của Poky 
│ 
├── meta-yocto-bsp/       # BSP mẫu cho các board tham khảo 
│ 
├── scripts/              # Script hỗ trợ (oe-init-build-env, bitbake…) 
│ 
├── documentation/        # Tài liệu Yocto 
│ 
├── build/                # Thư mục build (sinh ra khi chạy build) 
│ 
├── oe-init-build-env     # Script khởi tạo môi trường build 
│ 
└── LICENSE               # The license under which Poky is distributed (mix of GPLv2 and MIT)
```

---

# 🚀 Using Yocto Project - Basics

## ⚙️ Setup môi trường

Trước khi build, cần khởi tạo môi trường build:

```bash
./oe-init-build-env
```

Script này sẽ:
- ✅ Tạo build directory (mặc định tên là `build`)
- ✅ Thiết lập các biến môi trường cần thiết
- ✅ Sẵn sàng để chạy `bitbake`

## 🎯 Common targets

Các target phổ biến khi build với Yocto:

- 📦 **core-image-minimal**: Image nhỏ gọn để boot device với các command line cơ bản
- 🖥️ **core-image-sato**: Image có giao diện Sato (GNOME mobile-based UI)
- 🔧 **meta-toolchain**: Tạo cross-toolchain ở định dạng có thể cài đặt
- 💻 **meta-ide-support**: Tạo cross-toolchain và các tools bổ sung (gdb, qemu...) cho IDE integration

## 📁 Build directory structure

```bash
build/ 
├── conf/                 # Cấu hình chính của project
│   ├── local.conf        # Cấu hình máy build, package, image
│   └── bblayers.conf     # Danh sách các layer đang sử dụng 
│ 
├── tmp/                  # Tất cả output của build system
│   ├── buildstats/       # Thống kê build (CPU, time…)
│   ├── work/             # Nơi compile từng package
│   ├── deploy/           # Nơi xuất image, kernel, dtb, SDK (output cuối cùng)
│   │   ├── images/       # File .wic, .sdimg, .tar.bz2 (image để flash)
│   │   ├── rpm/          # Gói RPM 
│   │   ├── ipk/          # Gói IPK 
│   │   └── sdk/          # SDK cho host 
│   ├── log/              # Log build 
│   ├── sysroots/         # Thư viện & header cho host + target
│   └── cache/            # Cache build 
│ 
├── sstate-cache/         # Shared state cache (tăng tốc build) 
│ 
└── downloads/            # Source code tải về (tar.gz, git…)
```

## 🔧 Config the build system

Trong thư mục `conf/` có 2 file cấu hình quan trọng:

### 📝 bblayers.conf
- Liệt kê các layer Yocto sử dụng trong quá trình build
- Định nghĩa thứ tự ưu tiên của các layer

### 📝 local.conf
Thiết lập các biến cấu hình cho quá trình build:

- ⚡ **BB_NUMBER_THREADS**: Số lượng task BitBake chạy song song
- 🔄 **PARALLEL_MAKE**: Số tiến trình song song khi compile
- 🎯 **MACHINE**: Chỉ định board target để build (vd: `raspberrypi4`, `beaglebone`)

## 🔨 Build process

### ▶️ Usage:
```bash
bitbake [options] [recipename/target ...]
```

### ▶️ Build một target:
```bash
bitbake [target]
```

### ▶️ Ví dụ - Build minimal image:
```bash
bitbake core-image-minimal
```

Lệnh này sẽ chạy full build cho target đã chọn.

---

# 🎓 Using Yocto Project - Advanced Usage

## 🔀 Methods and Conditions

Tất cả các biến có thể được ghi đè và chỉnh sửa trong `local.conf`:

### 1️⃣ Append - Thêm vào cuối
```bash
_append
```
Thêm giá trị **sau** các giá trị đã được định nghĩa (không tự động thêm khoảng trắng).

**Ví dụ:** 
```bash
IMAGE_INSTALL_append = " dropbear"
```
Thêm package `dropbear` vào image.

### 2️⃣ Prepend - Thêm vào đầu
```bash
_prepend
```
Thêm giá trị **trước** những giá trị đã được định nghĩa (không tự động thêm khoảng trắng).

**Ví dụ:** 
```bash
FILESEXTRAPATHS_prepend := "${THISDIR}/${PN}:"
```
Thêm folder vào đầu danh sách paths.

### 3️⃣ Remove - Loại bỏ
```bash
_remove
```
Loại bỏ tất cả các lần xuất hiện của một giá trị.

**Ví dụ:** 
```bash
IMAGE_INSTALL_remove = "i2c-tools"
```
Xóa package `i2c-tools` khỏi image.

### 4️⃣ Machine-specific - Chỉ định cho machine cụ thể
```bash
VARIABLE_<machine>
```
Override biến chỉ cho một machine cụ thể (khớp với MACHINEOVERRIDES, MACHINE, SOC_FAMILY).

**Ví dụ:** 
```bash
KERNEL_DEVICETREE_beaglebone = "am335x-bone.dtb"
```
Chỉ sử dụng device tree này khi machine là `beaglebone`.

### 5️⃣ Ví dụ kết hợp

**Case 1: Append với machine-specific**
```bash
IMAGE_INSTALL = "busybox mtd-utils"
IMAGE_INSTALL_append = " dropbear"
IMAGE_INSTALL_append_beaglebone = " i2c-tools"
```

Kết quả:
- ✅ Machine `beaglebone`: `IMAGE_INSTALL = "busybox mtd-utils dropbear i2c-tools"`
- ✅ Machine khác: `IMAGE_INSTALL = "busybox mtd-utils dropbear"`

**Case 2: Độ ưu tiên override**
```bash
IMAGE_INSTALL_beaglebone = "busybox mtd-utils i2c-tools"
IMAGE_INSTALL = "busybox mtd-utils"
```

Kết quả:
- ✅ Machine `beaglebone`: `IMAGE_INSTALL = "busybox mtd-utils i2c-tools"`
- ✅ Machine khác: `IMAGE_INSTALL = "busybox mtd-utils"`

## 🎛️ Assignment Operators

Các operators dùng để gán giá trị cho configuration variables:

| Operator | Mô tả | Ví dụ |
|----------|-------|-------|
| `=` | Expand value khi sử dụng biến | `VAR = "value"` |
| `:=` | Expand ngay lập tức | `VAR := "value"` |
| `+=` | Append (có space) | `VAR += "value"` |
| `=+` | Prepend (có space) | `VAR =+ "value"` |
| `.=` | Append (không có space) | `VAR .= "value"` |
| `=.` | Prepend (không có space) | `VAR =. "value"` |
| `?=` | Gán nếu chưa được gán trước đó | `VAR ?= "value"` |
| `??=` | Giống `?=` nhưng ưu tiên thấp hơn | `VAR ??= "value"` |

### ⚠️ Lưu ý quan trọng

**Tránh sử dụng** `+=`, `=+`, `.=` và `=.` trong `local.conf` vì:

- ❌ Nếu `+=` được parse trước `?=`, thì `?=` sau đó sẽ bị bỏ qua
- ✅ Nên dùng `_append` và `_prepend` vì chúng luôn hoạt động đúng bất kể thứ tự parse

**Ví dụ vấn đề:**
```bash
# Có thể gây lỗi
VAR += "value1"
VAR ?= "value2"  # Sẽ bị ignore

# Nên dùng
VAR ?= "value2"
VAR_append = " value1"  # Luôn hoạt động đúng
```
