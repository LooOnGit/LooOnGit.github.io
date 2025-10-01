---
title: "IPC Socket"
date: 2025-09-28 19:17:24 +0800
categories: [Linux]
tags: [Linux]
---

# 🔌 IPC Socket trong Linux

## 1. Introduction

### 🎯 Khái niệm
Socket là một cơ chế giao tiếp cho phép các processes có thể giao tiếp với nhau, không phân biệt chúng đang chạy trên cùng một thiết bị hay các thiết bị khác nhau.

Đặc điểm chính của Socket:
- 🔹 Socket được đại diện bởi một socket descriptor file
- 🔹 Thông tin trong socket file bao gồm: Domain, Type và Protocol
- 🔹 Hỗ trợ giao tiếp hai chiều (full-duplex)
- 🔹 Linh hoạt và có thể dùng cho cả local và network communication

### 🔄 Các loại Socket
Datagram socket tốc độ nhanh hơn stream bởi vì gửi thôi không cần check kết nối.
1. **Stream Socket (SOCK_STREAM)**
   - Yêu cầu thiết lập kết nối trước khi truyền dữ liệu
   - Đảm bảo dữ liệu được truyền đáng tin cậy
   - Thường dùng trong TCP
   - Nếu có lỗi trong quá trình truyền nhận có lỗi sẽ báo lỗi.
![alt text](/assets/Linux/IPC_SOCKET/stream_Socket.png)
![alt text](/assets/Linux/IPC_SOCKET/applycation_TCP.png)

2. **Datagram Socket (SOCK_DGRAM)**
   - Không yêu cầu thiết lập kết nối
   - Không đảm bảo dữ liệu được truyền thành công
   - Thường dùng trong UDP

3. **Domain Types**
   - Internet Domain (AF_INET): Giao tiếp qua mạng
   - Unix Domain (AF_UNIX): Giao tiếp trên cùng máy

## 2. Work Flow

### STREAM SOCKET 
![Stream Socket Flow](/assets/Linux/IPC_SOCKET/stream_Socket.png)
#### 1. Tạo Socket

**Mục đích:**
- 🎯 Tạo một endpoint mới cho giao tiếp
- 🔌 Giống như tạo ổ cắm điện để chuẩn bị kết nối
- 📝 Trả về một file descriptor để quản lý socket

```c
#include <sys/socket.h>

int socket(int domain, int type, int protocol);
// Returns file descriptor on success, or -1 on error
```

**Tham số:**
- `domain`: Chỉ định họ giao thức (AF_INET cho Internet, AF_UNIX cho local)
- `type`: Kiểu socket (SOCK_STREAM cho TCP, SOCK_DGRAM cho UDP)
- `protocol`: Giao thức cụ thể (thường để 0 cho mặc định)

**Kết quả:**
- Trả về file descriptor nếu thành công (như một ID để quản lý socket)
- Trả về -1 nếu có lỗi (ví dụ: không đủ bộ nhớ, quá nhiều file descriptors)


#### 2. Bind

**Mục đích:**
- 🏠 Gán một địa chỉ cụ thể cho socket
- 📍 Giống như đăng ký địa chỉ nhà cho ổ cắm
- 🌐 Cho phép các client biết cách tìm đến socket này

```c
int bind(int sockfd, const struct sockaddr *addr, socklen_t addrlen);
// Returns 0 on success, -1 on error
```

**Tham số:**
- `sockfd`: Socket file descriptor (ID của socket đã tạo)
- `addr`: Địa chỉ để bind (IP address + port number)
- `addrlen`: Độ dài của cấu trúc địa chỉ

#### 3. Listen

**Mục đích:**
- 👂 Chuyển socket sang trạng thái lắng nghe
- 🚪 Giống như mở cửa và sẵn sàng đón khách
- 📞 Cho phép nhận các yêu cầu kết nối từ client

```c
int listen(int sockfd, int backlog);
// Returns 0 on success, -1 on error
```

**Tham số:**
- `sockfd`: Socket file descriptor
- `backlog`: Số lượng kết nối tối đa có thể xếp hàng chờ

#### 4. Accept

**Mục đích:**
- ✅ Chấp nhận kết nối từ client
- 🤝 Giống như bắt tay với khách khi họ đến
- 🔄 Tạo một socket mới riêng cho kết nối này

```c
int accept(int sockfd, struct sockaddr *addr, socklen_t *addrlen);
// Returns new socket file descriptor on success, -1 on error
```

**Tham số:**
- `sockfd`: Socket file descriptor đang lắng nghe
- `addr`: Lưu địa chỉ của client kết nối đến
- `addrlen`: Độ dài của cấu trúc địa chỉ

**Lưu ý:**
- Hàm này sẽ block cho đến khi có client kết nối
- Trả về một socket mới để giao tiếp riêng với client đó
- Socket gốc vẫn tiếp tục lắng nghe các kết nối mới

#### 5. Connect

**Mục đích:**
- 🔌 Dùng ở phía client để kết nối tới server
- 🚀 Khởi tạo quá trình bắt tay ba bước của TCP
- 🔗 Thiết lập kênh giao tiếp hai chiều

```c
int connect(int sockfd, const struct sockaddr *addr, socklen_t addrlen);
// Returns 0 on success, -1 on error
```

**Tham số:**
- `sockfd`: Socket file descriptor của client
- `addr`: Địa chỉ của server muốn kết nối đến
- `addrlen`: Độ dài của cấu trúc địa chỉ

#### 6. Read/Write

**Mục đích:**
- 📤 Write(): Gửi dữ liệu qua socket
- 📥 Read(): Nhận dữ liệu từ socket
- 🔄 Cho phép giao tiếp hai chiều

```c
ssize_t read(int fd, void *buf, size_t count);
ssize_t write(int fd, const void *buf, size_t count);
// Returns number of bytes read/written on success, -1 on error
```

**Tham số:**
- `fd`: Socket file descriptor
- `buf`: Buffer chứa dữ liệu đọc/ghi
- `count`: Số bytes muốn đọc/ghi

**Lưu ý:**
- Read() có thể đọc ít bytes hơn yêu cầu
- Write() có thể không ghi hết dữ liệu trong một lần
- Cần kiểm tra giá trị trả về để xử lý đúng

#### 7. Close

**Mục đích:**
- 🚫 Đóng kết nối socket
- 🧹 Giải phóng tài nguyên hệ thống
- 🔒 Kết thúc phiên giao tiếp

```c
int close(int fd);
// Returns 0 on success, -1 on error
```

**Tham số:**
- `fd`: Socket file descriptor cần đóng

**Lưu ý:**
- Nên luôn close socket khi không cần nữa
- Đóng cả hai phía server và client
- Có thể dùng shutdown() để đóng một chiều

#### 5. Code mẫu Server cơ bản
```c
#include <sys/socket.h>
#include <netinet/in.h>

int main() {
    int server_fd = socket(AF_INET, SOCK_STREAM, 0);
    struct sockaddr_in address;
    
    address.sin_family = AF_INET;
    address.sin_addr.s_addr = INADDR_ANY;
    address.sin_port = htons(8080);
    
    bind(server_fd, (struct sockaddr *)&address, sizeof(address));
    listen(server_fd, 3);
    
    int client_socket = accept(server_fd, NULL, NULL);
    
    // Read/Write operations
    
    close(client_socket);
    close(server_fd);
    return 0;
}
```

### Datagram sockets
![alt text](/assets/Linux/IPC_SOCKET/datagram_socket.png)

#### **sendto() - Gửi dữ liệu:**
```c
ssize_t sendto(int sockfd, const void *buf, size_t len, int flags,
               const struct sockaddr *dest_addr, socklen_t addrlen);
// Returns number of bytes sent on success, -1 on error
```

**Tham số:**
- `sockfd`: Socket file descriptor
- `buf`: Buffer chứa dữ liệu cần gửi
- `len`: Độ dài dữ liệu
- `flags`: Flags điều khiển (thường là 0)
- `dest_addr`: Địa chỉ đích
- `addrlen`: Độ dài của cấu trúc địa chỉ

#### **recvfrom() - Nhận dữ liệu:**
```c
ssize_t recvfrom(int sockfd, void *buf, size_t len, int flags,
                 struct sockaddr *src_addr, socklen_t *addrlen);
// Returns number of bytes received on success, -1 on error
```

**Tham số:**
- `sockfd`: Socket file descriptor
- `buf`: Buffer để lưu dữ liệu nhận được
- `len`: Kích thước tối đa có thể nhận
- `flags`: Flags điều khiển (thường là 0)
- `src_addr`: Địa chỉ nguồn gửi
- `addrlen`: Độ dài của cấu trúc địa chỉ

## 3. Socket Domains

### 🌐 Internet Domain Socket (AF_INET)
Cho phép giao tiếp giữa 2 processes trên các thiết bị khác nhau qua mạng.

**Đặc điểm:**
- 🔹 Sử dụng IPv4 addressing
- 🔹 Cần địa chỉ IP và port number
- 🔹 Hỗ trợ cả TCP và UDP
- 🔹 Có thể giao tiếp qua Internet/LAN

**Cấu trúc địa chỉ:**
```c
struct sockaddr_in {
    sa_family_t sin_family; //address    // AF_INET
    in_port_t sin_port;         // Port number
    struct in_addr sin_addr;    // IPv4 address
};
```

**Ví dụ khởi tạo:**
```c
struct sockaddr_in addr;
addr.sin_family = AF_INET;
addr.sin_port = htons(8080);
addr.sin_addr.s_addr = inet_addr("192.168.1.100");
```

### 🖥️ Unix Domain Socket (AF_UNIX)
Cho phép giao tiếp giữa các processes trên cùng một máy thông qua filesystem.

**Đặc điểm:**
- 📁 Sử dụng pathname làm địa chỉ
- ⚡ Hiệu năng cao hơn AF_INET
- 🔒 An toàn hơn (không qua network stack)
- 🔄 Hỗ trợ passing file descriptors

**Cấu trúc địa chỉ:**
```c
struct sockaddr_un {
    sa_family_t sun_family; //address     // AF_UNIX
    char sun_path[108];         // Pathname
};
```

**Ví dụ khởi tạo:**
```c
struct sockaddr_un addr;
addr.sun_family = AF_UNIX;
strncpy(addr.sun_path, "/tmp/socket", sizeof(addr.sun_path)-1);
```
**path name** mỗi lần bind chưa tồn tại trong hệ thống.



### So sánh hai loại:

| Tiêu chí | Internet Socket | Unix Socket |
|----------|----------------|-------------|
| Phạm vi | Toàn mạng | Cùng máy |
| Địa chỉ | IP + Port | Pathname |
| Hiệu năng | Thấp hơn | Cao hơn |
| Bảo mật | Cần thêm encryption | Tự bảo mật |
| Use case | Web services, Network apps | Database, IPC |

### 🔄 Byte Order trong Network Programming

#### Vấn đề về Byte Order
Các máy tính khác nhau có thể lưu trữ số đa byte theo thứ tự khác nhau:

1. **Big-endian (Network byte order)**:
   - MSB (Most Significant Byte) ở địa chỉ thấp nhất
   - Ví dụ: số 0x1234
     - Địa chỉ N: 0x12 (MSB)
     - Địa chỉ N+1: 0x34 (LSB)

2. **Little-endian (Nhiều CPU hiện đại)**:
   - LSB (Least Significant Byte) ở địa chỉ thấp nhất
   - Ví dụ: số 0x1234
     - Địa chỉ N: 0x34 (LSB)
     - Địa chỉ N+1: 0x12 (MSB)

#### Các hàm chuyển đổi Byte Order

```c
#include <arpa/inet.h>

// Host to Network Short (16-bit)
uint16_t htons(uint16_t hostshort);

// Host to Network Long (32-bit)
uint32_t htonl(uint32_t hostlong);

// Network to Host Short (16-bit)
uint16_t ntohs(uint16_t netshort);

// Network to Host Long (32-bit)
uint32_t ntohl(uint32_t netlong);
```

#### Ví dụ sử dụng:
```c
struct sockaddr_in addr;
addr.sin_port = htons(8080);        // Chuyển port number
addr.sin_addr.s_addr = htonl(IP);   // Chuyển IP address
```

### Lưu ý quan trọng:

**Internet Socket:**
- 🔒 Cần xem xét vấn đề bảo mật
- 🔄 Luôn chuyển đổi byte order khi:
  - Gán giá trị cho port number
  - Xử lý IP address
  - Truyền nhận số liệu đa byte
- 🚧 Kiểm tra firewall settings
- 📡 Xử lý network errors

**Unix Socket:**
- 📁 Cleanup socket file sau khi dùng
- 🔐 Kiểm tra file permissions
- 💾 Xử lý filesystem errors
- 🔄 Kiểm tra pathname length

### 📋 Chi tiết cấu trúc Socket Address

![alt text](/assets/Linux/IPC_SOCKET/network.png)

#### 1. Cấu trúc sockaddr_in cho IPv4
```c
struct sockaddr_in {
    sa_family_t sin_family;     // Address family: AF_INET (IPv4)
    in_port_t sin_port;         // Port number (network byte order)
    struct in_addr sin_addr;    // IPv4 address (network byte order)
    char sin_zero[8];          // Padding (không sử dụng, chỉ để căn chỉnh)
};
```

**VD:**
Địa chỉ là **192.168.1.12** thì giống địa chỉ nhà, trong 1 nhà nhiều người nhều phòng thì đại diện là **port** đại diện service đang chạy.

**Lưu ý:** 
- Phải chạy lệnh `sudo ss -tuln` để kiểm tra máy mình đang dùng **port** nào để né.

**Chi tiết các thành phần:**
- 🏷️ `sin_family`: Luôn là AF_INET cho IPv4
- 🔢 `sin_port`: Port number (16-bit)
- 📍 `sin_addr`: Địa chỉ IP (32-bit)
- 📏 `sin_zero`: Padding bytes (set to zero)

**Ví dụ khởi tạo đầy đủ:**
```c
struct sockaddr_in server_addr;
memset(&server_addr, 0, sizeof(server_addr));  // Clear structure
server_addr.sin_family = AF_INET;
server_addr.sin_port = htons(8080);            // Convert to network byte order
server_addr.sin_addr.s_addr = inet_addr("192.168.1.100");
```

#### 2. Các hàm hỗ trợ địa chỉ IP

```c
// Chuyển đổi IPv4 string sang binary
inet_addr("192.168.1.100");

// Chuyển đổi binary sang string
char ip[INET_ADDRSTRLEN];
inet_ntop(AF_INET, &(addr.sin_addr), ip, INET_ADDRSTRLEN);

// Chuyển đổi hostname sang IP
struct hostent *host = gethostbyname("www.example.com");
```

