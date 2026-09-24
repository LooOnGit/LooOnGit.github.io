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
Advertising trong BLE sử dụng cho mục đích broadcast (phát sóng) dữ liệu đến các thiết bị lân cận hoặc thông báo sự hiện diwjw của nó để một thiết bị khác có thể kết nối vào.

# Advertising process
## Advertising and discovery 
Khi device ở trạng thái advertising state, nó sẽ phát ra các gói quảng bá (advertising packets) để thông báo sự hiện diện của mình và tiềm năng kết nối với một thiết bị khác. Các gói tin này quảng bá này được gửi đi định kỳ theo các khoảng thời gian quảng bá (advertising intervals).


- **Advertising intervals**:  Khoảng thời gian mà advertising packet gửi, trong khoảng từ 20ms đến 10,24s, với mức tăng theo step là 0,625ms.

## Advertisement channels


BLE communicate thông qua 40 channel khác nhau. Trong đó được chia thành 3 primary channel và 37 channel phụ. Mỗi channel rộng 2MHz.
- **Primary channel**: được sử dụng trong mục đích advertisement.
- **Secondary channel**: Đôi lúc cũng được sử dụng cho mục đích advertisement, nhưng chủ yếu dử dụng để truyền dữ liệu sau khi đã thiết lập kết nối.
![](Pasted%20image%2020260924084155.png)
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


![](Pasted%20image%2020260924093537.png)
Scanner sẽ tiêu tốn năng lượng hơn việc advertising.

## Scan request and respone
Khi một peripheral là advertising, một central có thể chọn gửi scan request đến peripheral, hỏi thêm thông tin không có trong advertisement packets. Nếu scan request được chấp nhận, peripheral sẽ reponse qua 3 primary channels.
![](Pasted%20image%2020260924094657.png)
Đây là cách để thiết bị ngoại vi gửi thêm dữ liệu mà không cần phải thêm thiết lập kết nối với thiết bị central first. Ngoài ra, peripheral có thể chọn gửi lại respone empty nếu nó không còn thông tin nào để cung cấp.






