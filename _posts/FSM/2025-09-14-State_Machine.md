---
title: "State Machine"
date: 2025-09-14 09:05:00 +0800
categories: [FSM]
tags: [FSM]
---

# 🔄 State Machine trong Embedded Systems

## 🎯 Tổng quan
### State machine là gì?
State machine là một mô hình tính toán dựa trên một máy giả định được tạo thành từ một hoặc nhiều trạng thái. Nó chuyển từ trạng thái này sang trạng thái khác để thực hiện các hành động khác nhau.

### Khi nào ta nên sử dụng State machine?
Khi chúng ta muốn chia một nhiệm vụ lớn và phức tạp thành những đơn vị nhỏ độc lập.

## Lợi ích của việc dùng state machine?
- Dễ dàng theo dõi quá trình chuyển đổi/ dữ liệu/ sự kiện nào gây ra trạng thái hiện tại của yêu cầu.
- Ngăn chặn các hoạt động câu lệnh trái phép.
- Bảo trì dễ dàng, quá trình chuyển đổi trạng thái đọc lập nhau.
- Ổn iịnh và dễ dàng thay đổi.

## 🏗️ Cấu trúc cơ bản

### 1. Thành phần chính
- **States** 📍: Các trạng thái của hệ thống
- **Events** 🔔: Các sự kiện kích hoạt chuyển đổi
- **Transitions** ➡️: Quy tắc chuyển đổi giữa các trạng thái
- **Actions** ⚡: Hành động thực hiện khi chuyển đổi

## 🔀 Các loại State Machine

### 1. Moore Machine 🔷
- Output phụ thuộc vào state hiện tại
- Đơn giản, dễ dự đoán
- Ít phụ thuộc vào input

### 2. Mealy Machine 🔶
- Output phụ thuộc vào state và input
- Linh hoạt hơn
- Phản ứng nhanh với input

## 💻 Triển khai trong C

### 1. Enum cho States và Events
```c
typedef enum {
    STATE_IDLE,
    STATE_PROCESSING,
    STATE_DONE,
    STATE_ERROR
} State;

typedef enum {
    EVENT_START,
    EVENT_COMPLETE,
    EVENT_FAIL,
    EVENT_RESET
} Event;
```

### 2. Struct cơ bản
```c
typedef struct {
    State current_state;
    void (*state_handlers[NUM_STATES])(Event);
} StateMachine;

### 2. Quản lý chuyển đổi
```c
static const State state_transitions[NUM_STATES][NUM_EVENTS] = {
    // EVENT_START   EVENT_COMPLETE  EVENT_FAIL    EVENT_RESET
    { STATE_PROCESSING, STATE_IDLE,   STATE_ERROR,  STATE_IDLE  },  // STATE_IDLE
    { STATE_PROCESSING, STATE_DONE,   STATE_ERROR,  STATE_IDLE  },  // STATE_PROCESSING
    { STATE_PROCESSING, STATE_DONE,   STATE_ERROR,  STATE_IDLE  },  // STATE_DONE
    { STATE_IDLE,      STATE_ERROR,  STATE_ERROR,  STATE_IDLE  }   // STATE_ERROR
};
```

## 🚀 Ví dụ thực tế

### LED Controller State Machine
```c
typedef enum {
    LED_OFF,
    LED_ON,
    LED_BLINK
} LedState;

typedef enum {
    BTN_PRESS,
    TIMER_TICK,
    ERROR_DETECTED
} LedEvent;

void led_state_machine_init(void) {
    // Khởi tạo state machine
    current_state = LED_OFF;
    led_off();
}

void led_state_machine_run(LedEvent event) {
    switch(current_state) {
        case LED_OFF:
            if(event == BTN_PRESS) {
                led_on();
                current_state = LED_ON;
            }
            break;
            
        case LED_ON:
            if(event == BTN_PRESS) {
                led_start_blink();
                current_state = LED_BLINK;
            }
            break;
            
        case LED_BLINK:
            if(event == BTN_PRESS) {
                led_off();
                current_state = LED_OFF;
            }
            break;
    }
}
```