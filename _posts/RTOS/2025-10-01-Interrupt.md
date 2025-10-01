---
title: "INTERRUPT"
date: 2025-10-01 20:34:00 +0800
categories: [RTOS]
tags: [RTOS]
---

# 📚 FreeRTOS Interrupt

## 📌 Giới thiệu về Interrupt trong FreeRTOS
Interrupt (ngắt) là một sự kiện làm gián đoạn quá trình thực thi bình thường của CPU để xử lý một sự kiện quan trọng. Trong FreeRTOS, việc xử lý interrupt được thiết kế để đảm bảo tính real-time và độ tin cậy của hệ thống.

## 🔄 Cấu trúc Interrupt trong FreeRTOS

### 1. 🎯 Cơ chế Deferred Interrupt Processing
FreeRTOS sử dụng cơ chế xử lý ngắt trì hoãn để tối ưu hóa thời gian đáp ứng:

1. **ISR (Interrupt Service Routine)**: 
   - Thực hiện các xử lý tối thiểu và nhanh
   - Đánh dấu sự kiện cần xử lý
   - Đánh thức task xử lý ngắt

2. **Deferred Processing Task**:
   - Xử lý chi tiết sự kiện ngắt
   - Độ ưu tiên cao hơn các task thông thường
   - Thực hiện trong ngữ cảnh task

### 2. ⚡ Các API Interrupt trong FreeRTOS

#### 2.1 🔒 Từ ISR tới Task
```c
BaseType_t xQueueSendFromISR(
    QueueHandle_t xQueue,
    const void *pvItemToQueue,
    BaseType_t *pxHigherPriorityTaskWoken
);

BaseType_t xSemaphoreGiveFromISR(
    SemaphoreHandle_t xSemaphore,
    BaseType_t *pxHigherPriorityTaskWoken
);
```

#### 2.2 🔓 Critical Sections
```c
taskENTER_CRITICAL();     // Vào vùng găng
// Code cần bảo vệ
taskEXIT_CRITICAL();      // Ra vùng găng

// Trong ISR
taskENTER_CRITICAL_FROM_ISR();
// Code cần bảo vệ
taskEXIT_CRITICAL_FROM_ISR();
```

### 3. 🛡️ Interrupt Nesting

FreeRTOS hỗ trợ interrupt nesting (lồng ngắt) thông qua hai cấu hình:

1. **configMAX_SYSCALL_INTERRUPT_PRIORITY**:
   - Định nghĩa mức ưu tiên cao nhất cho ngắt sử dụng API RTOS
   - Ngắt có mức ưu tiên cao hơn không thể sử dụng API RTOS

2. **configKERNEL_INTERRUPT_PRIORITY**:
   - Định nghĩa mức ưu tiên cho ngắt kernel
   - Thường là mức ưu tiên thấp nhất

### 4. 🔍 Best Practices

1. **Tối thiểu hóa ISR**:
   ```c
   void UART_IRQHandler(void) {
       BaseType_t xHigherPriorityTaskWoken = pdFALSE;
       
       // Đọc dữ liệu
       uint8_t data = UART_READ_DATA();
       
       // Gửi tới task xử lý
       xQueueSendFromISR(xUartQueue, &data, &xHigherPriorityTaskWoken);
       
       // Yêu cầu chuyển ngữ cảnh nếu cần
       portYIELD_FROM_ISR(xHigherPriorityTaskWoken);
   }
   ```

2. **Task xử lý ngắt**:
   ```c
   void vUartProcessingTask(void *pvParameters) {
       uint8_t receivedData;
       
       for(;;) {
           // Đợi dữ liệu từ ISR
           if(xQueueReceive(xUartQueue, &receivedData, portMAX_DELAY) == pdPASS) {
               // Xử lý dữ liệu
               ProcessUartData(receivedData);
           }
       }
   }
   ```

### 5. ⚠️ Các lưu ý quan trọng

1. **Không block trong ISR**:
   - Không sử dụng vTaskDelay()
   - Không sử dụng API block khác
   - Sử dụng phiên bản ...FromISR() của API

2. **Độ ưu tiên ngắt**:
   - Cấu hình đúng mức ưu tiên
   - Tuân thủ configMAX_SYSCALL_INTERRUPT_PRIORITY
   - Xem xét latency và đáp ứng real-time

3. **Quản lý tài nguyên**:
   - Sử dụng mutex cho tài nguyên chia sẻ
   - Tránh deadlock trong xử lý ngắt
   - Giảm thiểu thời gian trong critical section

## 🔧 Ví dụ thực tế

### Timer Interrupt Example
```c
// Khởi tạo
void vSetupTimerInterrupt(void) {
    // Tạo queue cho dữ liệu timer
    xTimerQueue = xQueueCreate(5, sizeof(uint32_t));
    
    // Cấu hình timer hardware
    TIMER_Init();
    TIMER_EnableInterrupt();
}

// ISR
void TIMER_IRQHandler(void) {
    BaseType_t xHigherPriorityTaskWoken = pdFALSE;
    uint32_t ulTimerCount = TIMER_GetCount();
    
    // Gửi giá trị timer tới task
    xQueueSendFromISR(xTimerQueue, &ulTimerCount, &xHigherPriorityTaskWoken);
    
    portYIELD_FROM_ISR(xHigherPriorityTaskWoken);
}

// Task xử lý
void vTimerProcessTask(void *pvParameters) {
    uint32_t ulReceivedValue;
    
    for(;;) {
        if(xQueueReceive(xTimerQueue, &ulReceivedValue, portMAX_DELAY) == pdPASS) {
            // Xử lý giá trị timer
            ProcessTimerValue(ulReceivedValue);
        }
    }
}
```

## 📊 Hiệu năng và Tối ưu hóa

1. **Độ trễ ngắt**:
   - Giảm thiểu thời gian trong ISR
   - Sử dụng cơ chế deferred processing
   - Monitor và đo đạc thời gian xử lý

2. **Tối ưu bộ nhớ**:
   - Sử dụng kích thước queue phù hợp
   - Giải phóng tài nguyên khi không cần
   - Tránh allocation động trong ISR

3. **Debug và Troubleshooting**:
   - Sử dụng trace macros
   - Monitor stack usage
   - Kiểm tra overflow và timing
