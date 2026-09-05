---
title: BLE INTRODUCTION
date: 2026-08-20 08:21:49 +0700
categories:
  - BLE
tags:
  - BLE
---

# BLE INTRODUCTION

Bluetooth là một tiêu chuẩn công nghệ truyền thông không dây được sử dụng rộng rãi, do Nhóm Lợi ích Đặc biệt Bluetooth (Bluetooth SIG) quản lý. Giao thức Bluetooth Low Energy (LE - Năng lượng thấp) được xem là một giao thức khác biệt so với Bluetooth Classic (Cổ điển) và được thiết kế để truyền lượng dữ liệu nhỏ hơn với tốc độ dữ liệu tương đối thấp hơn, từ đó mang lại mức tiêu thụ điện năng thấp hơn.

## Bluetooth LE features
Data packet tạo ra nhỏ, khoảng 27 - 251 bytes.
Bluetooth LE cũng khác biệt so với Bluetooth Classic ở một số khía cạnh khác, chẳng hạn như các cấu trúc mạng (topologies) và các loại nút mạng (node types) được hỗ trợ. Nguyên nhân là do Bluetooth LE được thiết kế hướng tới các trường hợp sử dụng (use cases) hoàn toàn khác biệt so với Bluetooth Classic, do đó việc sử dụng các cấu trúc mạng khác nhau là điều cần thiết.

|                         Features                         |                                      |
| :------------------------------------------------------: | :----------------------------------: |
|                    **Operating band**                    | 2400 MHz – 2483.5 MHz  <br>~ 2.4 GHz |
|                  **Channel bandwidth**                   |                2 MHz                 |
|                **Number of RF channels**                 |                  40                  |
|                **Maximum transmit power**                |          20 dBm  <br>0.1 W           |
|         **Maximum application data throughput**          |               1.4 Mbps               |
| **Maximum range at reduced data rates (125 & 500 kbps)** |               ~1000 m                |
## Bluetooth LE protocol stack
![alt text](/assets/BLE/protocol_stack.png)
Ở tầng trên cùng là **lớp ứng dụng (application)**. Đây là tầng mà người dùng tương tác, thông qua các API, để sử dụng giao thức Bluetooth LE. Các thành phần quan trọng của tầng ứng dụng bao gồm các hồ sơ (profiles), dịch vụ (services) và đặc tính (characteristic.
### Host
- **Logical Link Control & Adaptation Protocol (L2CAP)** : Cung cấp các service đóng gói data cho các layer tầng trên.
- **Security Manager Protocol (SMP)** : định nghĩa và cung cấp các phương thức để truyền thông an toàn.
- **Attribute Protocol (ATT)** : cho phép một thiết bị hiển thị/cung cấp các phần dữ liệu cụ thể cho một thiết bị khác.
- **Generic Attribute Profile (GATT)** : định nghĩa các quy trình phụ (sub-procedures) cần thiết để sử dụng tầng ATT.
- **Generic Access Profile (GAP)** : giao tiếp trực tiếp với ứng dụng để xử lý việc khám phá thiết bị và các dịch vụ liên quan đến kết nối.
### Controller
- **Physical Layer (PHY)** : Quyết định cách dữ liệu thực tế được điều chế (modulated) lên sóng vô tuyến, cũng như cách nó được truyền và nhận.
- **Link Layer (LL)** : Quản lý trạng thái của khối vô tuyến (radio), được định nghĩa là một trong các trạng thái sau – chờ (standby), quảng bá (advertising), quét (scanning), khởi tạo (initiating), kết nối (connection).
