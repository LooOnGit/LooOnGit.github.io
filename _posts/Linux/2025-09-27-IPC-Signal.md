---
title: "IPC SIGNAL"
date: 2025-09-27 23:43:24 +0800
categories: [Linux]
tags: [Linux]
---

# 📡 IPC Signals trong Linux

## 1. Giới thiệu
Signal là một cơ chế IPC (Inter-Process Communication) trong Linux/Unix.
Giống như interrupt, không biết nào có signal giống như ngắt.
![alt text](/assets/Linux/IPC_Signal/signal.png)

**VD**:
Khi chạy chương trình nhấn **ctrl + c**, kết thúc process.

## 2. 📋Signal Handler
Bảng dưới mô tả khi nhận signal nó sẽ làm gì.
![alt text](/assets/Linux/IPC_Signal/handler_signal.png)
![alt text](/assets/Linux/IPC_Signal/mean_signal.png)

### Các loại handler:
1. **Default handler**: Hành động mặc định của hệ thống
2. **User-defined handler**: Hàm do người dùng định nghĩa
3. **Ignore signal**: Bỏ qua signal (SIG_IGN)

**VD**: 
Khi dùng `ctrl + c` là SIGINT sẽ thực hiện terminal như bản trên.

Signal hander có thể thay đổi dùng đế system call signal().


```c
void (*signal(int sig, void (*handler)(int)))(int);
```

**VD**: Khi bạn nhấn Ctrl+C, chương trình không thoát mà chạy handler() in ra "Received signal 2" (2 = SIGINT).


```c
#include <stdio.h>
#include <signal.h>
#include <unistd.h>

void handler(int signum) {
    printf("Received signal %d\n", signum);
}

int main() {
    // Gán handler cho SIGINT (Ctrl+C)
    signal(SIGINT, handler);

    printf("Running... Press Ctrl+C to trigger signal handler\n");
    while (1) {
        sleep(1);
    }
    return 0;
}
```


Để hủy 1 **Signal** để không làm gì hết.


```c
#include <stdio.h>
#include <signal.h>
#include <unistd.h>

int main() {
    // Bỏ qua Ctrl+C (SIGINT)
    signal(SIGINT, SIG_IGN);

    printf("Program is running. Try pressing Ctrl+C, it will be ignored!\n");

    while (1) {
        printf("Working...\n");
        sleep(2);
    }

    return 0;
}
```


Chạy chương trình trên, nhấn Ctrl+C thì chương trình không thoát (vì signal `SIGINT` bị ignore). Bạn chỉ có thể thoát bằng cách kill -9 PID (`SIGKILL` thì không thể hủy được).


Khi muốn đưa signal về hàm mặc định.


```c
signal(SIGINT, SIG_IGN); // ignore signal
```


### Signal tương tự như ngắt
![alt text](/assets/Linux/IPC_Signal/Process_singal.png)
Trong lúc chương trình đang chạy hàm xử lý **signal** (signal handler), nếu cùng loại signal đó (hoặc signal khác) tới tiếp thì có được xử lý ngay không?

Khi một signal đang được xử lý, thì mặc định signal đó sẽ bị block (chặn) cho đến khi handler kết thúc.
→ Ví dụ: bạn đang trong handle_sigint(), nếu người dùng bấm Ctrl+C liên tục, thì kernel sẽ nhớ chỉ 1 cái SIGINT pending. Khi handler xong, nó mới được gửi tiếp, nhưng các lần bấm dư sẽ bị gộp lại (chỉ giữ 1).

Nếu một signal khác loại đến (ví dụ đang xử lý SIGINT, thì có SIGTERM tới), thì nó vẫn có thể interrupt handler hiện tại. Điều này tùy thuộc vào thiết lập sa_mask trong sigaction().

Nếu bạn muốn cho phép reentrant handler (tức là handler có thể bị ngắt bởi chính signal đó nhiều lần), bạn phải dùng sigaction() với cờ SA_NODEFER (hoặc SA_NOMASK).

**Lưu ý**:
- Trong 1 **Signal Handler** chỉ xử lý gọn, như chỉ thay đổi giá trị biến toàn cục gửi ra ngoài thì ở ngoài **Stack Machine** đoán chạy 1 hàm riêng bên ngoài thôi.
- Không nên malloc trong **Signal Handler**. Tránh đưa các hàm xử lý chuỗi.
- **Reentrant** và **Non Reentrant** 


**VD:** Non-reentrant (không an toàn vì dùng printf)

```c
#include <stdio.h>
#include <signal.h>
#include <unistd.h>

void handler(int sig) {
    printf(">> Signal caught!\n");  // ❌ Không an toàn (printf không reentrant)
}

int main() {
    signal(SIGINT, handler);  // Đăng ký handler cho Ctrl+C

    while (1) {
        printf("Main loop running...\n"); // Có thể bị gián đoạn
        sleep(1);
    }

    return 0;
}
```


👉 Ở đây, nếu Ctrl+C xảy ra khi printf trong main chưa xong, thì printf trong handler lại chạy → dễ sinh lỗi hoặc in ra loạn dòng.


**VD:** Reentrant/An toàn (dùng write)
```c
#include <stdio.h>
#include <signal.h>
#include <unistd.h>

void handler(int sig) {
    const char msg[] = ">> Signal caught!\n";
    write(STDOUT_FILENO, msg, sizeof(msg) - 1);  // ✅ An toàn (write là async-signal-safe)
}

int main() {
    signal(SIGINT, handler);  // Đăng ký handler cho Ctrl+C

    while (1) {
        printf("Main loop running...\n");
        sleep(1);
    }

    return 0;
}
```


`write()` là hàm async-signal-safe nên có thể gọi trong signal handler mà không lo deadlock hay hỏng dữ liệu.

### Quay lại signal handler default
#### Dùng con trỏ hàm signal 
```c
#include <stdio.h>
#include <stdlib.h>
#include <pthread.h>
#include <unistd.h>
#include <string.h>
#include <signal.h>

void handlerSingal(int sig)
{
    printf("Receive the signal %d\n", sig);
}

int main()
{
    void (*oldHandler)(int);
    oldHandler = signal(SIGINT, handlerSingal);
    

    while(1)
    {
        printf("Hello World\n");
        for(int i = 0; ; i++)
        {
            sleep(1);
            if(i == 10)
            {
                signal(SIGINT, oldHandler);
                printf("restore the signal handler\n");
            }
        }
    }
    
    return 0;
}
```

#### Dùng MACRO
```c
#include <stdio.h>
#include <stdlib.h>
#include <pthread.h>
#include <unistd.h>
#include <string.h>
#include <signal.h>

void handlerSingal(int sig)
{
    printf("Receive the signal %d\n", sig);
}

int main()
{
    signal(SIGINT, handlerSingal);
    

    while(1)
    {
        printf("Hello World\n");
        for(int i = 0; ; i++)
        {
            sleep(1);
            if(i == 10)
            {
                signal(SIGINT, SIG_DFL);
                printf("restore the signal handler\n");
            }
        }
    }
    
    return 0;
}
``` 

## Non-Reentrant vs Reentrant Functions

Trong C trên Unix/Linux, nhiều hàm **không reentrant** vì dùng buffer static, biến toàn cục, hoặc quản lý tài nguyên chia sẻ.
Một số hàm có phiên bản **reentrant** (hậu tố `_r`) an toàn hơn khi dùng trong signal handler hoặc multithread.

### Reentrant

Là một hàm "an toàn" để có thể bị ngắt (interrupt) khi đang chạy dở dang, và được gọi lại một lần nữa (từ một luồng khác hoặc từ một signal handler) mà không làm sai lệch dữ liệu hay gây lỗi chương trình.

**Đặc điểm:**
- **Không sử dụng biến toàn cục**: Mọi dữ liệu cần thiết đều được truyền qua tham số hoặc được cấp phát riêng cho từng luồng.
- **Không sử dụng bộ nhớ đệm (buffer) chia sẻ**: Nếu cần buffer, mỗi luồng phải tự cấp phát.
- **An toàn với signal handler**: Có thể gọi trong signal handler mà không làm hỏng trạng thái chương trình.

### Non-Reentrant

Hàm non-reentrant là hàm **không an toàn** khi được gọi bởi nhiều luồng cùng lúc hoặc trong signal handler.

**Nguyên nhân:**
- **Sử dụng biến static/global**: Dữ liệu chia sẻ giữa các lần gọi hàm.
- **Sử dụng bộ nhớ đệm chia sẻ**: Ví dụ như `asctime()` sử dụng buffer tĩnh nội bộ.
- **Không đảm bảo thread-safe**: Các hàm thư viện chuẩn cũ (ví dụ: `printf`, `strtok`) thường không thread-safe.

### Bảng so sánh

| Nhóm hàm              | Non-Reentrant (không an toàn)                                                                                                                                               | Reentrant (an toàn)                                           |
| --------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------- |
| **I/O chuẩn (stdio)** | `printf`, `fprintf`, `sprintf`, `snprintf`, `scanf`, `fscanf`, `sscanf`, `getc`, `fgetc`, `getchar`, `putc`, `fputc`, `putchar`, `fgets`, `gets`, `fputs`, `puts`, `perror` | ❌ Không có phiên bản `_r` → tránh dùng trong signal handler   |
| **Chuỗi & Thời gian** | `strtok`, `asctime`, `ctime`, `gmtime`, `localtime`                                                                                                                         | `strtok_r`, `asctime_r`, `ctime_r`, `gmtime_r`, `localtime_r` |
| **Quản lý bộ nhớ**    | `malloc`, `calloc`, `free`, `realloc`                                                                                                                                       | ❌ Không có phiên bản `_r`                                     |
| **Biến môi trường**   | `getenv`, `setenv`                                                                                                                                                          | ❌ Không có `_r`                                               |
| **Thông tin user**    | `getpwnam`, `getpwuid`                                                                                                                                                      | `getpwnam_r`, `getpwuid_r`                                    |
| **Thông tin host**    | `gethostbyname`, `gethostbyaddr`                                                                                                                                            | `gethostbyname_r`, `gethostbyaddr_r`                          |

---

## Lưu ý khi viết Signal Handler
* Càn ngắn càn tốt.
* Chỉ dùng **async-signal-safe functions** (xem `man 7 signal`), ví dụ:

  * `write`, `read`, `signal`, `kill`, `_exit`, `waitpid`
* Tránh tuyệt đối `printf`, `malloc`, `getenv`, hoặc các hàm thao tác buffer static.

---

## Kill
```c
kill(PID, SIGKILL);
```