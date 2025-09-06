---
title: "Ring Buffer"
date: 2025-09-06 09:30:01 +0800
categories: [Data Structure]
tags: [Data Structure]
---

# 🔄 Tìm hiểu thêm về Ring Buffer

## 📝 Ring Buffer là gì?

Ring Buffer (còn gọi là circular queue) là một cấu trúc dữ liệu dạng FIFO (First In First Out) có kích thước cố định, được thiết kế để:
- Lưu trữ dữ liệu tạm thời
- Quản lý bộ nhớ hiệu quả
- Xử lý dữ liệu streaming

## So với linear queue

Đối với queue thông thường việc thêm và xóa đi phần tử, sẽ có khoảng trống không sử dụng được.

## 💡Circular Queue hoạt động như thế nào?
1. Khởi tạo array có size là n, trong đó n là số lượng phần tử tối đã của queue.
2. Khởi tạo 3 giá trị (size, capacity and front).
3. **Enqueue:** Để enqueue một phần từ x vào queue, làm như sau:
    1. Kiểm tra nếu size ==  capacity (queue là full), display "Queue if full".
    2. Nếu không full: tính `rear = (front + size) % capacity` và đưa vào giá trị ở phần tử thứ rear. Tăng size lên 1.
4. **Dequeue:** 
    1. Kiểm tra nếu size == 0 (queue là rỗng).
    2. Nếu không rỗng: Lấy phần tử ở front index và di chuyển đến front = (front + 1) % capacity. Ngoài ra, giảm size xuống 1 đơn vị.

![Queue Example](/assets/DataStructure/RingBuffer/page1.png)
![Queue Example](/assets/DataStructure/RingBuffer/page2.png)
![Queue Example](/assets/DataStructure/RingBuffer/page3.png)
![Queue Example](/assets/DataStructure/RingBuffer/page4.png)
![Queue Example](/assets/DataStructure/RingBuffer/page5.png)
![Queue Example](/assets/DataStructure/RingBuffer/page6.png)

![Queue Example](/assets/DataStructure/RingBuffer/RingBuffer.png)

## Code

```c
#include <stdio.h>
#include <stdbool.h>
#define MAX_SIZE 4

//defining the Queue structure
typedef struct {
    int items[MAX_SIZE];
    int front;
    int size;
    int capacity;
}Queue;

// Function to initialize the queue
void initializeQueue(Queue* q) {
    q->capacity = MAX_SIZE;
    q->size = 0;
    q->front = 0;
}

//Function to check if the queue is empty
bool isEmpty(Queue* q) { return (q->size == 0); }

//Function to check if the queue is full
bool isFull(Queue* q) { return (q->size == q->capacity); }


//Function to add an element to the queue (Enqueue operation)
void enqueue(Queue* q, int value) {
    if (isFull(q)) {
        printf("Queue is full\n");
        return;
    }
    int rear = (q->front + q->size) % q->capacity;
    q->items[rear] = value;
    q->size++;
}

//Function to remove an element to the queue (Dequeue operation)
int dequeue(Queue* q) {
    if (isEmpty(q)) {
        printf("Queue is empty\n");
        return -1;
    }
    int res =  q->items[q->front];
    q->front = (q->front + 1) % q->capacity;
    q->size--;
    return res;
}

// Get the front element
int getFront(Queue q) {

    // Queue is empty
    if (q.size == 0)
        return -1;
    return q.items[q.front];
}

// Get the rear element
int getRear(Queue q) {

    // Queue is empty
    if (q.size == 0)
        return -1;
    int rear = (q.front + q.size - 1) % q.capacity;
    return q.items[rear];
}

int main(void) {
    Queue q;
    initializeQueue(&q);

    enqueue(&q, 30);
    printf("%d\t %d\n", getFront(q), getRear(q));

    enqueue(&q, 40);
    printf("%d\t %d\n", getFront(q), getRear(q));

    enqueue(&q, 80);
    printf("%d\t %d\n", getFront(q), getRear(q));

    enqueue(&q, 90);
    printf("%d\t %d\n", getFront(q), getRear(q));

    dequeue(&q);
    printf("%d\t %d\n", getFront(q), getRear(q));

    dequeue(&q);
    printf("%d\t %d\n", getFront(q), getRear(q));

    enqueue(&q, 1000);
    printf("%d\t %d\n", getFront(q), getRear(q));
    return 0;
}
```


