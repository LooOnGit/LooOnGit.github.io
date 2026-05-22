---
title: "Device Tree - Mô Tả Phần Cứng Cho Linux"
date: 2025-12-13 13:56:25 +0800
categories: [Kernel]
tags: [Kernel]
---

# 🌳 Device Tree

## 📖 Device Tree là gì?

**Device Tree** là một cấu trúc dữ liệu dạng cây (tree) được sử dụng để mô tả phần cứng của hệ thống một cách linh hoạt, tránh việc hardcode thông tin phần cứng trực tiếp vào kernel Linux.

### 🎯 Tại sao cần Device Tree?

Trước khi có Device Tree, các thông tin về phần cứng (địa chỉ thiết bị, interrupt, GPIO pin, v.v.) được **hardcode** trực tiếp vào kernel. Điều này dẫn đến:

- ❌ Phải biên dịch lại kernel khi thay đổi phần cứng
- ❌ Code kernel phình to với hàng nghìn dòng mô tả board
- ❌ Khó bảo trì và mở rộng
- ❌ Mỗi board cần một kernel riêng

Với Device Tree:

- ✅ Tách biệt mô tả phần cứng ra khỏi kernel
- ✅ Một kernel có thể chạy trên nhiều board khác nhau
- ✅ Dễ dàng thêm/sửa thiết bị mà không cần rebuild kernel
- ✅ Code kernel gọn gàng, dễ maintain

---

## 🏗️ Cấu trúc Device Tree

Device Tree sử dụng kiến trúc cây với:

- **Root Node** (`/`): Node gốc chứa toàn bộ hệ thống
- **Child Nodes**: Các node con đại diện cho thiết bị phần cứng
- **Properties**: Các thuộc tính mô tả chi tiết thiết bị

```
/ (root)
├── cpus
│   ├── cpu@0
│   └── cpu@1
├── memory@80000000
├── soc
│   ├── uart@44e09000
│   ├── i2c@44e0b000
│   └── gpio@44e07000
└── leds
    ├── led0
    └── led1
```

---

## 🔄 Quy Trình Build Device Tree

![Quy trình build Device Tree](/assets/Yocto/Device_tree/image.png)

### 📝 1. Device Tree Source (`.dts`)

File nguồn được viết bằng ngôn ngữ mô tả thiết bị, dễ đọc và chỉnh sửa:

```dts
/dts-v1/;

/ {
    compatible = "ti,am335x-bone-black", "ti,am335x-bone", "ti,am33xx";
    model = "TI AM335x BeagleBone Black";

    memory@80000000 {
        device_type = "memory";
        reg = <0x80000000 0x20000000>; /* 512 MB */
    };

    leds {
        compatible = "gpio-leds";
        
        led0 {
            label = "beaglebone:green:usr0";
            gpios = <&gpio1 21 0>;
            linux,default-trigger = "heartbeat";
        };
    };
};
```

### 🔧 2. Device Tree Compiler (DTC)

Sử dụng công cụ `dtc` để biên dịch:

```bash
dtc -I dts -O dtb -o am335x-boneblack.dtb am335x-boneblack.dts
```

**Các tham số:**
- `-I dts`: Input format là Device Tree Source
- `-O dtb`: Output format là Device Tree Blob (binary)
- `-o`: Tên file output

### 📦 3. Device Tree Blob (`.dtb`)

File binary được kernel Linux đọc khi boot:
- 🔐 Định dạng nhị phân, không thể đọc trực tiếp
- 🚀 Được bootloader (U-Boot) load vào RAM
- 💾 Kernel parse và khởi tạo thiết bị

---

## 📚 Cú Pháp Device Tree

### 🏷️ Properties Cơ Bản

| Property | Mô tả | Ví dụ |
|----------|-------|-------|
| `compatible` | Định danh thiết bị | `"ti,am335x-uart"` |
| `reg` | Địa chỉ và kích thước | `<0x44e09000 0x2000>` |
| `interrupts` | Interrupt number | `<72>` |
| `status` | Trạng thái thiết bị | `"okay"` hoặc `"disabled"` |
| `gpios` | GPIO pin | `<&gpio1 21 0>` |

### 🔢 Ví Dụ: UART Device

```dts
uart0: serial@44e09000 {
    compatible = "ti,am3352-uart", "ti,omap3-uart";
    reg = <0x44e09000 0x2000>;
    interrupts = <72>;
    status = "okay";
    clock-frequency = <48000000>;
};
```

**Giải thích:**
- `uart0`: Label để tham chiếu
- `serial@44e09000`: Tên node (serial ở địa chỉ 0x44e09000)
- `compatible`: Driver nào sẽ xử lý thiết bị này
- `reg`: UART nằm ở địa chỉ `0x44e09000`, kích thước `0x2000` bytes
- `interrupts`: Sử dụng interrupt số 72

### 📌 Ví Dụ: GPIO LED

```dts
leds {
    compatible = "gpio-leds";
    pinctrl-names = "default";
    pinctrl-0 = <&user_leds_s0>;

    led0 {
        label = "beaglebone:green:usr0";
        gpios = <&gpio1 21 GPIO_ACTIVE_HIGH>;
        linux,default-trigger = "heartbeat";
        default-state = "off";
    };

    led1 {
        label = "beaglebone:green:usr1";
        gpios = <&gpio1 22 GPIO_ACTIVE_HIGH>;
        linux,default-trigger = "mmc0";
        default-state = "off";
    };
};
```

---

## 🔍 Device Tree Overlays

**Device Tree Overlays** cho phép thay đổi/thêm cấu hình phần cứng **không cần reboot**!

### 💡 Khi nào dùng Overlays?

- 🔌 Thêm cape (expansion board) vào BeagleBone
- 🎛️ Enable/disable peripheral (I2C, SPI, UART)
- 🔧 Thay đổi pin muxing
- 📡 Thêm sensor hoặc module mới

### 📄 Ví Dụ Overlay: Enable I2C2

```dts
/dts-v1/;
/plugin/;

/ {
    compatible = "ti,beaglebone", "ti,beaglebone-black";

    fragment@0 {
        target = <&am33xx_pinmux>;
        __overlay__ {
            bb_i2c2_pins: pinmux_bb_i2c2_pins {
                pinctrl-single,pins = <
                    0x178 0x73  /* P9.18, I2C2_SDA */
                    0x17C 0x73  /* P9.17, I2C2_SCL */
                >;
            };
        };
    };

    fragment@1 {
        target = <&i2c2>;
        __overlay__ {
            status = "okay";
            pinctrl-names = "default";
            pinctrl-0 = <&bb_i2c2_pins>;
            clock-frequency = <100000>;
        };
    };
};
```

### 🚀 Load Overlay

```bash
# Biên dịch overlay
dtc -@ -I dts -O dtb -o BB-I2C2-00A0.dtbo BB-I2C2-00A0.dts

# Load overlay vào hệ thống
echo BB-I2C2 > /sys/devices/platform/bone_capemgr/slots

# Kiểm tra overlays đã load
cat /sys/devices/platform/bone_capemgr/slots
```

---

## 🛠️ Làm Việc Với Device Tree

### 📂 Vị Trí Files

```
📁 arch/arm/boot/dts/
├── am335x-boneblack.dts          # BeagleBone Black main DTS
├── am335x-bone-common.dtsi       # Common cho tất cả BBB variants
├── am33xx.dtsi                   # AM335x SoC definitions
└── am33xx-clocks.dtsi            # Clock definitions
```

### 🔎 Xem Device Tree Runtime

```bash
# Xem device tree hiện tại đang chạy
ls /proc/device-tree/

# Xem thông tin một property
cat /proc/device-tree/model

# Decompile DTB về DTS
dtc -I dtb -O dts -o current.dts /boot/dtbs/am335x-boneblack.dtb
```

### ✏️ Sửa Đổi Device Tree

1. **Sửa file `.dts`** trong kernel source
2. **Biên dịch lại:**
   ```bash
   make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- dtbs
   ```
3. **Copy file `.dtb` đến boot partition:**
   ```bash
   cp arch/arm/boot/dts/am335x-boneblack.dtb /boot/dtbs/
   ```
4. **Reboot để load device tree mới**

---

## 🎯 Ví Dụ Thực Hành: Blink LED với Platform Driver

Ví dụ này minh họa cách tạo một **Platform Driver** điều khiển LED bằng Device Tree trên BeagleBone Black.

### 📋 Bước 1: Thêm Device Tree Node

#### 1️⃣ Sửa file `am33xx.dtsi`

Tìm file trong thư mục: `.../kernelbuildscripts/KERNEL/arch/arm/boot/dts/am33xx.dtsi`

Thêm node LED vào cuối file (trước dấu `};` cuối cùng):

```dts
hello_led: sample_led0 {
	compatible = "gpio-leds";
	#address-cells = <0x01>;
	#size-cells = <0x01>;
	reg = <0x44E07000 0x1000>;  /* GPIO1 base address */
	led_config = <0xa 0x2>;      /* Custom property (optional) */
	led_regs = <0x80000000 0x194 0x190 0x134 0x13c>;  /* Custom (optional) */
};
```

**Giải thích:**
- `hello_led:` - Label để reference
- `compatible = "gpio-leds"` - String match với driver
- `reg = <0x44E07000 0x1000>` - GPIO1 memory region (0x44E07000-0x44E08000)

#### 2️⃣ Sửa file `am335x-boneblack.dtsi`

Tìm file: `.../kernelbuildscripts/KERNEL/arch/arm/boot/dts/am335x-boneblack.dtsi`

Thêm vào cuối file:

```dts
&hello_led {
	birth_date = <0x1 0x9 0x7c5>;  /* 1/9/1989 (custom property) */
	status = "okay";                /* Enable this device */
};
```

**⚠️ Quan trọng:** `status = "okay"` bắt buộc phải có, không có sẽ không probe!

---

### 🔨 Bước 2: Build Device Tree Blob

```bash
# Chạy từ thư mục kernelbuildscripts
cd kernelbuildscripts

# Build DTB files
./tools/rebuild.sh
```

Sau khi build xong, file `.dtb` sẽ nằm ở:
```
KERNEL/arch/arm/boot/dts/am335x-boneblack.dtb
```

---

### 📤 Bước 3: Copy DTB vào Board

**Cách 1: Qua SSH/SCP**
```bash
scp KERNEL/arch/arm/boot/dts/am335x-boneblack.dtb debian@192.168.7.2:/tmp/
ssh debian@192.168.7.2
sudo cp /tmp/am335x-boneblack.dtb /boot/dtbs/$(uname -r)/
sudo reboot
```

**Cách 2: Copy vào SD card**
```bash
# Mount boot partition
sudo mount /dev/sdb1 /mnt
sudo cp am335x-boneblack.dtb /mnt/dtbs/
sudo umount /mnt
```

---

### 💻 Bước 4: Viết Platform Driver

Tạo file `blink_led.c`:

```c
#include <linux/module.h>
#include <linux/platform_device.h>
#include <linux/of.h>
#include <linux/io.h>
#include <linux/timer.h>

/* GPIO1 Register Offsets (từ TRM AM335x) */
#define GPIO_SETDATAOUT_OFFSET          0x194  /* Set data output */
#define GPIO_CLEARDATAOUT_OFFSET        0x190  /* Clear data output */
#define GPIO_OE_OFFSET                  0x134  /* Output Enable */

/* GPIO1_31 = USR3 LED trên BeagleBone Black */
#define LED                             ~(1 << 31)  /* Clear bit 31 to enable output */
#define DATA_OUT                        (1 << 31)   /* Set bit 31 */

MODULE_LICENSE("GPL");
MODULE_AUTHOR("Your Name");
MODULE_DESCRIPTION("BeagleBone LED Blink Platform Driver");
MODULE_VERSION("1.0");

/* Global variables */
void __iomem *base_addr;        /* Virtual address của GPIO1 */
struct timer_list my_timer;     /* Kernel timer */
unsigned int count = 0;         /* Blink counter */

/**
 * timer_function - Timer callback để blink LED
 * @data: Timer data (unused)
 */
static void timer_function(struct timer_list *data)
{
	if ((count % 2) == 0) {
		pr_info("LED ON\n");
		writel_relaxed(DATA_OUT, base_addr + GPIO_SETDATAOUT_OFFSET);
	}
	else {
		pr_info("LED OFF\n");
		writel_relaxed(DATA_OUT, base_addr + GPIO_CLEARDATAOUT_OFFSET);
	}

	count++;
	/* Re-arm timer: 1 second (1 * HZ) */
	mod_timer(&my_timer, jiffies + HZ);
}

/**
 * Device Tree matching table
 */
static const struct of_device_id blink_led_of_match[] = {
	{ .compatible = "gpio-leds" },  /* Match với compatible trong DTS */
	{},  /* Sentinel */
};
MODULE_DEVICE_TABLE(of, blink_led_of_match);

/**
 * blink_led_probe - Driver probe khi tìm thấy device trong DT
 */
static int blink_led_probe(struct platform_device *pdev)
{
	uint32_t reg_data = 0;
	struct resource *res = NULL;

	pr_info("🚀 LED Driver Probe Started\n");

	/* 1. Lấy memory resource từ Device Tree (reg property) */
	res = platform_get_resource(pdev, IORESOURCE_MEM, 0);
	if (!res) {
		pr_err("❌ Failed to get memory resource\n");
		return -ENODEV;
	}

	pr_info("📍 GPIO Memory: 0x%08x - 0x%08x\n", res->start, res->end);

	/* 2. Map physical address → virtual address */
	base_addr = ioremap(res->start, resource_size(res));
	if (!base_addr) {
		pr_err("❌ Failed to ioremap\n");
		return -ENOMEM;
	}

	/* 3. Cấu hình GPIO làm output */
	reg_data = readl_relaxed(base_addr + GPIO_OE_OFFSET);
	pr_info("🔧 GPIO_OE before: 0x%08x\n", reg_data);
	reg_data &= LED;  /* Clear bit 31 = output mode */
	writel_relaxed(reg_data, base_addr + GPIO_OE_OFFSET);
	
	reg_data = readl_relaxed(base_addr + GPIO_OE_OFFSET);
	pr_info("🔧 GPIO_OE after: 0x%08x\n", reg_data);

	/* 4. Setup kernel timer */
	timer_setup(&my_timer, timer_function, 0);
	mod_timer(&my_timer, jiffies + HZ);  /* Start after 1 second */

	pr_info("✅ LED Driver Probe Success!\n");
	return 0;
}

/**
 * blink_led_remove - Cleanup khi rmmod
 */
static int blink_led_remove(struct platform_device *pdev)
{
	pr_info("👋 LED Driver Remove\n");

	/* Stop timer trước khi unload (rất quan trọng!) */
	del_timer_sync(&my_timer);

	/* Unmap memory */
	if (base_addr) {
		iounmap(base_addr);
		base_addr = NULL;
	}

	pr_info("✅ Cleanup Complete\n");
	return 0;
}

/**
 * Platform driver structure
 */
static struct platform_driver blink_led_driver = {
	.probe		= blink_led_probe,
	.remove		= blink_led_remove,
	.driver		= {
		.name	= "blink_led",
		.of_match_table = blink_led_of_match,
	},
};

/* Register platform driver */
module_platform_driver(blink_led_driver);
```

---

### 📝 Bước 5: Tạo Makefile

```makefile
obj-m += blink_led.o

KERNEL_DIR := /path/to/kernelbuildscripts/KERNEL
CROSS_COMPILE := arm-linux-gnueabihf-

all:
	make ARCH=arm CROSS_COMPILE=$(CROSS_COMPILE) -C $(KERNEL_DIR) M=$(PWD) modules

clean:
	make -C $(KERNEL_DIR) M=$(PWD) clean

deploy:
	scp blink_led.ko debian@192.168.7.2:/tmp/
```

---

### 🚀 Bước 6: Build và Chạy

```bash
# Build module
make

# Copy sang board
make deploy

# Trên board (SSH vào)
ssh debian@192.168.7.2

# Load module
sudo insmod /tmp/blink_led.ko

# Xem log
dmesg | tail -20

# Kiểm tra module đã load
lsmod | grep blink

# Unload module
sudo rmmod blink_led
```

---

## 🐛 Debugging Guide

### ✅ Kiểm tra Device Tree đã load chưa

```bash
# Xem device tree runtime
ls /proc/device-tree/sample_led0/
cat /proc/device-tree/sample_led0/compatible
cat /proc/device-tree/sample_led0/status

# Kiểm tra platform devices
ls /sys/bus/platform/devices/
cat /sys/bus/platform/devices/*/of_node/compatible
```

**Nếu không thấy node:**
- ❌ Chưa copy `.dtb` mới vào board
- ❌ `status` không phải `"okay"`
- ❌ Chưa reboot sau khi copy DTB

---

### ✅ Kiểm tra Driver có match không

```bash
# Xem kernel log khi insmod
sudo dmesg -w  # Chạy trước khi insmod

# Trong terminal khác
sudo insmod blink_led.ko
```

**Nếu không thấy "Probe Started":**
- ❌ `compatible` string không khớp
- ❌ Thiếu `MODULE_DEVICE_TABLE(of, ...)`
- ❌ Kernel version không support platform driver

---

### ✅ Kiểm tra Memory Mapping

```bash
# Xem memory regions
cat /proc/iomem | grep gpio

# Expected:
# 44e07000-44e07fff : gpio@44e07000
```

**Nếu ioremap fail:**
- ❌ `reg` property sai địa chỉ
- ❌ Memory region bị conflict
- ❌ Permission denied (cần run as root)

---

### ✅ Kiểm tra GPIO

```bash
# Xem GPIO status
sudo cat /sys/kernel/debug/gpio

# Kiểm tra LED có blink không
# USR3 LED nằm trên board, gần USB connector
```

**Nếu LED không blink:**
- ❌ Sai GPIO number (check schematic)
- ❌ GPIO bị mux về chức năng khác
- ❌ Timer không chạy (check dmesg)

---

### 🔍 Debug Commands Hữu Ích

```bash
# 1. Xem toàn bộ device tree
dtc -I fs -O dts /proc/device-tree > /tmp/current.dts
less /tmp/current.dts

# 2. Trace platform driver matching
echo 'file drivers/base/platform.c +p' > /sys/kernel/debug/dynamic_debug/control
dmesg -w

# 3. Kiểm tra timer
cat /proc/timer_list | grep -A 10 timer_function

# 4. Monitor kernel messages realtime
sudo dmesg -wH

# 5. Xem module info
modinfo blink_led.ko
```

---

## ⚠️ Common Errors & Solutions

### ❌ Error: "probe sample_led" không xuất hiện

**Nguyên nhân:**
- Compatible string không match
- Device Tree không được load
- `status = "disabled"`

**Solution:**
```bash
# Verify DT matching
cat /sys/bus/platform/drivers/blink_led/uevent
ls /sys/bus/platform/drivers/blink_led/
```

---

### ❌ Error: "Unable to handle kernel paging request"

**Nguyên nhân:**
- `ioremap()` failed nhưng vẫn dùng `base_addr`
- Địa chỉ GPIO sai

**Solution:**
```c
base_addr = ioremap(res->start, resource_size(res));
if (!base_addr) {
    pr_err("ioremap failed\n");
    return -ENOMEM;  /* ← Quan trọng! */
}
```

---

### ❌ Error: Kernel panic khi rmmod

**Nguyên nhân:**
- Timer vẫn chạy khi module bị unload

**Solution:**
```c
/* Phải dùng del_timer_sync(), KHÔNG dùng del_timer() */
del_timer_sync(&my_timer);  /* Wait for timer to finish */
```

---

## 📊 Expected Output

```bash
# dmesg output khi insmod
[  123.456789] 🚀 LED Driver Probe Started
[  123.456790] 📍 GPIO Memory: 0x44e07000 - 0x44e08000
[  123.456791] 🔧 GPIO_OE before: 0x7fffffff
[  123.456792] 🔧 GPIO_OE after: 0x7fffffff
[  123.456793] ✅ LED Driver Probe Success!
[  124.456794] LED ON
[  125.456795] LED OFF
[  126.456796] LED ON
...

# dmesg output khi rmmod
[  200.123456] 👋 LED Driver Remove
[  200.123457] ✅ Cleanup Complete
```

---

## 📖 Tài Liệu Tham Khảo

### 🌐 Official Resources

- 📘 [Device Tree Specification](https://www.devicetree.org/specifications/) - Chuẩn chính thức
- 📙 [BeagleBone Device Tree Guide](https://elinux.org/Beagleboard:BeagleBoneBlack_Debian#Loading_custom_capes) - Hướng dẫn specific cho BBB

### 🎓 Learning Resources

- [Device Tree for Dummies](https://elinux.org/Device_Tree_Usage) - Tutorial cho người mới
- [AM335x Technical Reference](https://www.ti.com/lit/ug/spruh73q/spruh73q.pdf) - Chi tiết về SoC

---

## 💡 Tips & Best Practices

> 💡 **Pro Tip**: Luôn backup file `.dtb` gốc trước khi sửa đổi!

> ⚠️ **Warning**: Sai cấu hình Device Tree có thể khiến board không boot được!

> ✨ **Best Practice**: Sử dụng overlays thay vì sửa trực tiếp main device tree

> 🔧 **Debug**: Dùng `dmesg | grep -i dtb` để kiểm tra lỗi khi load device tree

---
