---
title: "GPIO Subsystem"
date: 2025-05-17 06:20:00 +0700
categories: [Kernel]
tags: [Kernel]
---

# 📘 GPIO Subsystem

GPIO (General Purpose Input/Output) là các chân đa năng của SoC có thể được cấu hình để đọc tín hiệu (Input) hoặc xuất tín hiệu (Output) theo điều khiển của phần mềm. Linux Kernel quản lý toàn bộ các chân GPIO thông qua một subsystem chuyên biệt với hai thế hệ API.

---

## 1. 🌟 Tổng quan GPIO Subsystem
- GPIO Subsystem cung cấp một lớp trừu tượng (abstraction layer) thống nhất để lập trình viên Driver điều khiển GPIO mà không cần quan tâm đến cách phần cứng SoC cụ thể hoạt động bên dưới.
- Kernel quản lý các chân GPIO thông qua khái niệm **GPIO Controller** (đại diện bởi `struct gpio_chip`). Mỗi Controller quản lý một dải chân nhất định.
- Toàn bộ thông tin GPIO được kết nối với Device Tree qua thuộc tính `gpio-controller` và `#gpio-cells`.

---

## 2. ⚠️ Integer-based GPIO (API Cũ - Legacy)

Đây là thế hệ API đầu tiên, quản lý GPIO bằng một **con số nguyên (integer) toàn cục** đại diện cho từng chân trên toàn hệ thống. API này hiện đã bị **deprecated** (không khuyến khích sử dụng).

### Cơ chế hoạt động
Mỗi chân GPIO được gán một con số duy nhất trên toàn hệ thống (ví dụ: GPIO chip A chiếm số 0–31, GPIO chip B chiếm số 32–63). Lập trình viên phải tự tra cứu và hard-code con số này vào driver.
```bash
debian@arm:~$ cat /sys/class/gpio/gpiochip32/base
32
```

### Các API
```c
#include <linux/gpio.h>

int gpio_request(unsigned gpio, const char *label);

int gpio_direction_input(unsigned gpio);
int gpio_direction_output(unsigned gpio, int value);

int  gpio_get_value(unsigned gpio);
void gpio_set_value(unsigned gpio, int value);

void gpio_free(unsigned gpio);
```

### Nhược điểm
- **Dễ xung đột số:** Con số integer là toàn cục, khi cắm thêm chip mở rộng GPIO qua I2C thì số của các chân có thể bị dịch chuyển, gây lỗi driver.
- **Không hiểu Active-Low/High:** Driver phải tự viết code đảo ngược bit thủ công dựa vào cấu hình phần cứng → dễ sinh bug.
- **Không gắn được với Device Tree:** Phải hard-code số GPIO vào source code, kém linh hoạt khi chuyển sang board khác.

---

## 3. ✅ Descriptor-based GPIO (API Mới - Modern)

Ra đời từ **Kernel 3.13**, thế hệ API này quản lý GPIO thông qua một con trỏ mờ (opaque pointer) kiểu **`struct gpio_desc`** thay vì số nguyên. Đây là cách tiếp cận **bắt buộc** cho mọi driver mới.

### Cơ chế hoạt động
Driver không cần biết số GPIO là bao nhiêu. Thay vào đó, nó xin cấp phát GPIO dựa trên **tên (label/con_id)** được khai báo trong Device Tree, Kernel sẽ tự dò tìm và trả về một `struct gpio_desc *` tương ứng.

### Khai báo trong Device Tree (DTS)
```dts
my_device {
    compatible = "vendor,my-device";
    /* Tên label "reset" và "led", kèm cờ GPIO_ACTIVE_LOW */
    reset-gpios = <&gpio1 5 GPIO_ACTIVE_LOW>;
    led-gpios   = <&gpio2 3 GPIO_ACTIVE_HIGH>;
};
```
 
### Các API
```c
#include <linux/gpio/consumer.h>

/* Xin cấp phát GPIO theo tên label trong DTS.
 * con_id = "reset" -> Kernel tự tìm thuộc tính "reset-gpios" trong DTS.
 * flags  = GPIOD_OUT_LOW / GPIOD_OUT_HIGH / GPIOD_IN
 */
struct gpio_desc *gpiod_get(struct device *dev,
                            const char *con_id,
                            enum gpiod_flags flags);

/* Cấu hình chiều (Direction) */
int gpiod_direction_input(struct gpio_desc *desc);
int gpiod_direction_output(struct gpio_desc *desc, int value);

/* Đọc / Ghi giá trị theo mức LOGIC (không phải điện áp vật lý) */
int  gpiod_get_value(struct gpio_desc *desc);
void gpiod_set_value(struct gpio_desc *desc, int value);

/* Giải phóng chân GPIO sau khi dùng xong */
void gpiod_put(struct gpio_desc *desc);
```

### Ưu điểm vượt trội

| Tiêu chí | Integer-based (Cũ) | Descriptor-based (Mới) |
|---|---|---|
| Nhận diện chân | Hard-code số nguyên toàn cục | Lấy theo tên label từ DTS |
| Xử lý Active-Low | Tự viết code đảo bit | **Kernel tự xử lý tự động** |
| Tích hợp DTS | Kém, thủ công | **Hoàn hảo, tự động ánh xạ** |
| An toàn kiểu dữ liệu | Thấp (chỉ là int) | Cao (opaque pointer) |
| Trạng thái | ❌ Deprecated | ✅ Khuyến khích dùng |

### Điểm nổi bật nhất: Tự động xử lý Active-Low
```c
/* DTS đã khai báo: reset-gpios = <&gpio1 5 GPIO_ACTIVE_LOW>; */
struct gpio_desc *reset = gpiod_get(dev, "reset", GPIOD_OUT_LOW);

/* Lập trình viên chỉ cần nghĩ theo logic: 1 = BẬT reset */
gpiod_set_value(reset, 1);
/* → Kernel tự biết chân này Active-Low, sẽ xuất 0V ra chân vật lý */

gpiod_set_value(reset, 0);
/* → Kernel xuất 3.3V ra chân vật lý để TẮT reset */
```
Code driver hoàn toàn trong sáng, không phụ thuộc vào thiết kế phần cứng!

---

## 4. 🛠️ Ví dụ hoàn chỉnh: Điều khiển LED bằng Descriptor-based
```c
#include <linux/module.h>
#include <linux/platform_device.h>
#include <linux/gpio/consumer.h>

struct my_priv {
    struct gpio_desc *led;
};

static int my_probe(struct platform_device *pdev)
{
    struct my_priv *priv;

    priv = devm_kzalloc(&pdev->dev, sizeof(*priv), GFP_KERNEL);
    if (!priv)
        return -ENOMEM;

    /* Lấy GPIO "led" từ DTS, cấu hình là output mức thấp ban đầu */
    priv->led = devm_gpiod_get(&pdev->dev, "led", GPIOD_OUT_LOW);
    if (IS_ERR(priv->led))
        return PTR_ERR(priv->led);

    /* Bật LED theo mức logic (Kernel tự xử lý Active-Low/High) */
    gpiod_set_value(priv->led, 1);

    platform_set_drvdata(pdev, priv);
    return 0;
}

static int my_remove(struct platform_device *pdev)
{
    struct my_priv *priv = platform_get_drvdata(pdev);
    gpiod_set_value(priv->led, 0); /* Tắt LED */
    /* devm_gpiod_get tự động giải phóng, không cần gọi gpiod_put */
    return 0;
}
```
> **Lưu ý:** Sử dụng `devm_gpiod_get()` thay vì `gpiod_get()` để Kernel tự động giải phóng tài nguyên khi driver bị gỡ, tránh memory leak.
