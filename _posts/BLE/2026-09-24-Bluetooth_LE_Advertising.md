---
title: Bluetooth_LE_Advertising
date: 2026-09-24 08:00:58 +0700
categories:
  - BLE
tags:
  - BLE
---
# Bluetooth LE Advertising
# Overview 
Advertising trong BLE sử dụng cho mục đích broadcast (phát sóng) dữ liệu đến các thiết bị lân cận hoặc thông báo sự hiện diện của nó để một thiết bị khác có thể kết nối vào.

# Advertising process
## Advertising and discovery 
Khi device ở trạng thái advertising state, nó sẽ phát ra các gói quảng bá (advertising packets) để thông báo sự hiện diện của mình và tiềm năng kết nối với một thiết bị khác. Các gói tin này quảng bá này được gửi đi định kỳ theo các khoảng thời gian quảng bá (advertising intervals).


- **Advertising intervals**:  Khoảng thời gian mà advertising packet gửi, trong khoảng từ 20ms đến 10,24s, với mức tăng theo step là 0,625ms.

## Advertisement channels


BLE communicate thông qua 40 channel khác nhau. Trong đó được chia thành 3 primary channel và 37 channel phụ. Mỗi channel rộng 2MHz.
- **Primary channel**: được sử dụng trong mục đích advertisement.
- **Secondary channel**: Đôi lúc cũng được sử dụng cho mục đích advertisement, nhưng chủ yếu dử dụng để truyền dữ liệu sau khi đã thiết lập kết nối.
![alt text](/assets/BLE/advertising_channels.png)
Để đảm bảo một mức độ dự phòng, advertising packet được gửi trên 3 primary advertising channels, channels 37, 38, 39. Device cũng sẽ scan 3 channels này để tìm kiếm advertising devices.


3 channel 37, 38, 39 này đóng vai trò thiết yêu trong việc thiết lập kết nối. 3 channel này không liên tiếp nhau như hình trên. 3 channel này cách xa nhau để tránh nhiễu từ băng tần lân cận, ít bị ảnh hưởng bởi các device dùng công nghệ khác sử dụng băng tần ISM, chẳng hạn như wifi.
## Scan interval and scan window

- **Scan interval** là khoản thời gian giữa các lần scan tìm các advertisement packet.
- **Scan window** là khoảng thời gian mà thiết bị quét thực sự dành ra để quét tìm gói tin. 


**Ví dụ:** **Hãy tưởng tượng bạn là người bảo vệ, cần canh xem có ai đến gõ cửa không.** Nếu bạn đứng nhìn ra cửa suốt cả ngày thì chắc chắn không bỏ sót ai, nhưng rất mệt. Nên bạn quyết định: cứ mỗi 10 phút, bạn ra đứng nhìn cửa 3 phút, 7 phút còn lại thì ngồi nghỉ.


Trong ví dụ này:
- **Scan interval = 10 phút**: là "chu kỳ lặp lại", cứ bao lâu thì bắt đầu một lượt canh mới.
- **Scan window = 3 phút**: là thời gian bạn _thực sự_ đứng canh trong mỗi chu kỳ đó.
- **Duty cycle = 3/10 = 30%**: là tỷ lệ thời gian bạn làm việc so với tổng thời gian.



Bởi vì device advertise trên các channel khác nhau, scanner sẽ rotate xung quanh các channel, bằng cách switch các channel sau mỗi scan interval.


![alt text](/assets/BLE/scan_interval_window.png)
Scanner sẽ tiêu tốn năng lượng hơn việc advertising.

## Scan request and respone
Khi một peripheral là advertising, một central có thể chọn gửi scan request đến peripheral, hỏi thêm thông tin không có trong advertisement packets. Nếu scan request được chấp nhận, peripheral sẽ reponse qua 3 primary channels.
![alt text](/assets/BLE/scan_request_response.png)
Đây là cách để thiết bị ngoại vi gửi thêm dữ liệu mà không cần phải thêm thiết lập kết nối với thiết bị central first. Ngoài ra, peripheral có thể chọn gửi lại respone empty nếu nó không còn thông tin nào để cung cấp.

# Advertising types
Có nhiều cách khác nhau để peripheral có thể advertise.
- **Connectable vs non-connectable:** Xác định liệu central có thể connect peripheral hoặc không.
- **Connectable vs non-connectable:** Xác định nếu peripheral chấp nhận scan request từ một scanner.
- **Directed vs undirected:** Xác định rằng liệu advertisement packets có được gửi đến scanner hoặc không.


|                    | Connectable | Scannable | Directed |
|--------------------|:-----------:|:---------:|:--------:|
| ADV_IND            |      x      |     x     |          |
| ADV_DIRECT_IND     |      x      |           |     x    |
| ADV_SCAN_IND       |             |     x     |          |
| ADV_NONCONN_IND    |             |           |          |

# Bluetooth address 
Mỗi BLE device được định danh bằng address 48-bit. Bluetooth address được phân loại public hay random. Random address thì có resolvable or non-resolvable.


![alt text](/assets/BLE/bluetooth_address_types.png)

Một Bluetooth LE devie sử dụng ít nhất một address type:
- Public address.
- Random static address.
- Random private resolvable.
- Random private non-resolvate.


Public address được assign tới device lấy ra từ kho của IEEE cùng nhóm với MAC, do đó giới thiệu như là Blutooth MAC address.
## Public address
Một public address đã được fixed trong device lúc sản xuất. Dịa chỉ này được đăng ký với IEEE, và nó duy nhất trên toàn câu đối với thiết bị đó, không thể thay đổi trong suôt vòng đời của thiết bị. Có một khoản phí liên quan đến việc có được loại địa chỉ này.
## Random address
Random address được  sử dụng phổ biến không yêu cầu đăng kí với IEEE. Được lập trình hoặc được tạo ra trong thời gian runtime. Có thể là static address hoặc private address.
### Random static address
Có thể được allocate và fixed trong vòng đời của device. Nó có thể được thay thế lúc bootup, nhưng không trong lúc runtime.
### Random private address
Có thể được sử dụng khi một thiết bị muốn protect privacy của nó. Địa chỉ có thể thay đổi theo chu kỳ để ẩn danh tính device và theo dõi device.
#### Resolvable random private address 
Resolvable private address đúng như tên gọi nó có thẻ resolvable, vì chung có một khóa share trước (pre-share key) để xác định địa chỉ mới mỗi khi thay đổi. Key này là Indentity Resolving Key - IRK), được dùng để vừa generate and resolve địa chỉ random.


IRK cho phép bên còn lại chuyển đổi địa chỉ riêng từ ngẫu nghiên thành địa chỉ Bluetooth LE thực của thiết bị.
#### Non-resolvable random private address
Là loại address mà các device khác không resolvable được, và chỉ nhằm mục đích ngăn chặn việc theo dõi. Loại địa chỉ này không được sử dụng phổ biến.
# Advertisement packet
BLE packet, phần chính gọi là Protocol Data Unit (PDU). PDU bao gồm data PDU (hay gọi là data channel PDU) và advertising PDU (advertising channel PDU), tùy thuộc vào advertisement hoặc data transmission.
![alt text](/assets/BLE/ble_packet_pdu_structure.png)
Advertising PDU bao gồm header và payload, phần header của advertising bao gồm:
![alt text](/assets/BLE/advertising_pdu_header.png)
- **PDU Type**: Xác định advertisement type, ví dụ `ADV_IND`.
- **RFU**: Reserved for future use.
- **ChSel**: Set 1 nếu LE Channel Selection Algorithm #2.
- **TxAdd** (Tx Address): 0 or 1, phụ thuộc vào transmitter address là public hay random.
- **RxAdd** (Rx Address): 0 or 1, phụ thuộc vào receiver address là public hay random.
- **Lenght**: Lenght của payload.

Payload của advertising PDU thì được chia làm 2 section, 6 byte đầu đại diệ cho advertiser address (AdvA) và phần còn lại là advertisement data (AdvData).

![alt text](/assets/BLE/advertising_pdu_payload.png)
- **AdvA**: Bluetooth address của advertising device.
- **AdvData**: Advertisement data packet.


Payload structure phụ thuộc vào advertising. Khi directed advertisement (`ADV_DIRECT_IND`) cần thêm space để chỉ định thêm address của receiver. Do đó, AdvData field thì đã thay thế bằng receiver address field có size bằng 6. Advertisement packet của type (`ADV_DIRECT_IND`) không bao gồm payload.


**Advertisement data section** thì được mô tả như hình dưới:


![alt text](/assets/BLE/advertisement_data_structure.png)
Advertisement data packet thì tạo ra nhiều structure được gọi là advertisement data structure (AD structures). Mỗi AD structure có 1 length field, 1 field cho type (AD type), 1 field cho data (AD Data).


Một số AD type thường được dùng:
- **Complete local name** (`BT_DATA_NAME_COMPLETE`): Tên thiết bị khi quét.
- **Shortened local name** (`BT_DATA_NAME_SHORTENED`): Tên nhưng ngắn hơn.
- **Uniform Resource Identifier** (`BT_DATA_URI`): Được sử dụng để quảng bá một URI, chẳng hạn như địa chỉ trang web (URL). 
- **Service UUID**: Là số duy nhất trên toàn cầu để cho 1 service cụ thể. 
- **Manufacturer Specific** **Data** (`BT_DATA_MANUFACTURER_DATA`): Đây là một loại rất phổ biến, cho phép các công ty tự do định nghĩa các dữ liệu quảng bá tùy chỉnh của riêng họ. Công nghệ iBeacon của Apple hoạt động hoàn toàn dựa trên loại dữ liệu này.
- **Flags**: là các biến có kích thước 1-bit dùng để đánh dấu (bật/tắt) một thuộc tính hoặc chế độ hoạt động cụ thể của thiết bị (ví dụ: báo cho biết thiết bị này có hỗ trợ kết nối Bluetooth cổ điển hay chỉ hỗ trợ BLE).



![alt text](/assets/BLE/ad_flags_example.png)
## Flags
Advertisement flag thì 1 bit nhưng đóng gói thì 1 byte, có nghĩa là 8 flags có thể được set. Một số flag:
- `BT_LE_AD_LIMITED`: Mở kết nối trong _thời gian ngắn_ (tự tắt quảng bá sau một khoảng thời gian để tiết kiệm pin).
- `BT_LE_AD_GENERAL`: Mở kết nối _liên tục dài hạn_ (không tự động tắt, timeout = 0).
- `BT_LE_AD_NO_BREDR`: Thiết bị _chỉ dùng BLE_, không hỗ trợ sóng Bluetooth classic (BR/EDR).
- 