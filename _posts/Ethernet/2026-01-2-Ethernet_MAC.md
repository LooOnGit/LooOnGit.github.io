---
title: "ETHERNET MAC"
date: 2026-01-26 19:53:00 +0700
categories: [ETHERNET]
tags: [ETHERNET][STM32F7]
---

# ETHERNET MAC

## Interview
- Tuân thủ hoàn toàn tiêu chuẩn **IEEE 802.3**.
- Có tích hợp DMA riêng để tự động điều khiển flow dữ liệu.
- Hỗ trợ cả 2 giao tiếp vật lý**MII** và **RMII**.
- Hỗ trợ cả 2 hoạt động **full-duplex** và **half-duplex** ở tốc độ 10 hoặc 100 Mbit/s.
- Sử dụng **CSMA/CD** làm phương thức truy cập.

### PHY Control Interface
- PHY bên ngoài thì điều khiển ngoại vi thông qua **Station Management Interface (SMI)** để cho phép đọc và ghi các register của PHY.
- Interface này hỗ trợ MDIO protocol trên pair wire.

![Interaction](/assets/Ethernet/connector.png)

### RMII Interface
- Yêu cầu 7 đường tín hiệu.
- Shared clock cho TX và RX.
- Tiết kiệm GPIO pins.


![Interaction](/assets/Ethernet/RMII.png)


### MII Interface
- Yêu cầu 16 đường tín hiệu.
- Không có shared clock cho TX và RX.


![Interaction](/assets/Ethernet/MII.png)


## Trong STM32F7
### Pin 
![Interaction](/assets/Ethernet/pin_ethernet.png)
### Architecture
![Interaction](/assets/Ethernet/System_architecture.png)


- **Ethernet peripheral** bao gồm **MAC 802.3** và **DMA controller**.
- Có thể chọn giao tiếp **MII** hoặc **RMII**. Có thể set **SYSCFG_PMC**.
- **DMA controller** giao tiếp với Core và memories thông qua AHB Master và Slave.
- AHB Master điều khiển truyền dữ liệu trong khi AHB Slave dùng để truy cập thanh ghi control và status.
- AHB clock frequency tối thiểu là 25MHz khi dùng Ethernet.
- **Transmit FIFO (Tx FIFO)** lưu đệm dữ liệu được DMA đọc từ bộ nhớ hệ thống trước khi được MAC Core truyền đi.
- **Receive FIFO (Rx FIFO)** lưu đệm dữ liệu nhận được từ PHY trước khi được DMA ghi vào bộ nhớ hệ thống.

#### SMI (Station Management Interface)
Dùng để giao tiếp với external PHY. Cho phép cấu hình thanh ghi vào của PHY thông qua hai đường tín hiệu gồm clock (MDC) và data (MDIO).
- **MDC** (Management Data Clock): Cung cấp timing tham chiếu cho data truyền ở frequency tối đa 2.5MHz.
- **MDIO** (Management Data Input/Output): đường dữ liệu vào/ra dạng nối tiếp, dùng để truyền các thông tin trạng thái và cấu hình giữa MAC và thiết bị lớp vật lý (PHY), hoạt động đồng bộ với tín hiệu clock MDC.


![SMI](/assets/Ethernet/MII_pinout.png)
### MII (Media Independent Interface)
MII được định nghĩa cách kết nối giữa MAC sublayer và PHY cho truyền dữ liệu ở 10Mbits/s và 100 Mbit/s.
- **MII_TX_CLK**: Clock cho TX.
- **MII_RX_CLK**: Clock cho RX.
- **MII_TXD[3:0]**: Data cho TX.
- **MII_RXD[3:0]**: Data cho RX.
- **MII_TX_EN**: Enable cho TX.
- **MII_RX_DV**: Đây là tín hiệu dữ liệu nhận hợp lệ (receive data valid). Nó cho biết PHY đang đưa ra các nibble dữ liệu đã được khôi phục và giải mã trên giao diện MII để MAC nhận
- **MII_CRS**: Carrier Sense.Tín hiệu này được PHY kích hoạt khi môi trường truyền hoặc nhận đang hoạt động (không ở trạng thái idle). PHY sẽ ngắt tín hiệu này khi cả quá trình truyền và nhận đều ở trạng thái idle.
- **MII_COL**: Tín hiệu này được PHY kích hoạt khi phát hiện xảy ra va chạm dữ liệu.
- **MII_RX_ER**: Error cho RX.
- **MII_TX_ER**: Error cho TX.


Để tạo ra cả hai tín hiệu clock TX_CLK và RX_CLK, PHY bên ngoài phải được cấp một nguồn clock ngoài 25 MHz. Thay vì cấp thêm bộ bên ngoài thì STM32F7 có thể cấp clock 25MHz thông qua chân MCO pin.


![MII](/assets/Ethernet/clock_external.png)

### RMII (Reduced Media Independent Interface)
![RMII](/assets/Ethernet/RMII_pinout.png)
- Nó giảm số pin sử dụng còn 7pin so với MII là 16pin.
- Khi dùng RMII thì clock phải được gấp đôi từ 25MHz lên 50MHz.
- Có thể cấp clock ngoài là 50MHz, hoặc sử dụng PLL để tạo ra tần số 50MHz.


![RMII_clock](/assets/Ethernet/clock_rmii.png)
## Các thuật ngữ
- **CSMA/CD** (Carrier Sense Multiple Access with Collision Detection): Đa truy cập cảm nhận sóng mang với phát hiện va chạm.
- **MAC** (Media Access Control): Điều khiển truy cập môi trường truyền thông - thực hiện tầng liên kết dữ liệu theo chuẩn Ethernet **IEEE 802.3**.
- **MII** (Media independent interface): Giao tiếp độc lập môi trường.
- **RMII** (Reduced Media Independent Interface): Giao tiếp độc lập môi trường rút gọn.