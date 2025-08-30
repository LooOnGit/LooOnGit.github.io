---
title: "Linked List"
date: 2025-08-29 23:02:00 +0800
categories: [Data Structure]
tags: [Data Structure]
---

# 📝 LINKED LIST (DANH SÁCH LIÊN KẾT)

## 🎯 Danh sách liên kết là gì?
Danh sách liên kết là một cấu trúc dữ liệu tuyến tính hoạt động như một mảng có thể phát triển và thu nhỏ khi cần thiết, từ bất kỳ điểm nào trong mảng.

## 🌟 Đặc điểm cơ bản

1. **Cấu trúc cơ bản**
   - Một tập hợp các node được phân bổ động
   - Mỗi node chứa một giá trị và một con trỏ trỏ đến node tiếp theo
   - Node cuối cùng trỏ đến NULL

2. **Quản lý danh sách**
   - Sử dụng một biến con trỏ (head) trỏ đến phần tử đầu tiên
   - Danh sách rỗng khi con trỏ head là NULL

## ✨ Ưu điểm so với mảng thông thường

1. **Tính linh hoạt cao**
   - Có thể thêm hoặc bớt các phần tử từ bất kỳ vị trí nào
   - Không cần xác định kích thước ban đầu
   - Tối ưu việc sử dụng RAM - cấp phát khi cần

## ⚠️ Nhược điểm và hạn chế

1. **Hạn chế truy cập**
   - Không hỗ trợ truy cập ngẫu nhiên
   - Phải duyệt từ đầu danh sách để tìm phần tử
   - Tốc độ truy cập chậm hơn so với mảng

2. **Vấn đề về bộ nhớ**
   - Cần thêm bộ nhớ cho con trỏ
   - Phức tạp trong quản lý bộ nhớ
   - Nguy cơ rò rỉ bộ nhớ cao hơn (cấp phát động quên free thì tăng rò rỉ bộ nhớ).

3. **Hiệu suất cache**
   - Không liên tục trong bộ nhớ
   - Hiệu quả cache thấp hơn mảng
   - Tốc độ xử lý có thể bị ảnh hưởng

![alt text](/assets/DataStructure/LinkedList/Node_LinkedList.png)

Tối ưu hóa dùng RAM mỗi lần dùng thì bắt đầu tạo ra phần từ. Không cần tạo ra 1 lượt, đỡ bị phân mãnh.


## 🔄 Các loại Linked List

1. **Singly Linked List** 🔜
   - Mỗi node chứa dữ liệu và con trỏ next
   - Chỉ có thể duyệt theo một chiều

2. **Doubly Linked List** ↔️
   - Mỗi node chứa dữ liệu và hai con trỏ (prev & next)
   - Có thể duyệt theo cả hai chiều

3. **Circular Linked List** 🔄
   - Node cuối trỏ về node đầu
   - Có thể là singly hoặc doubly

## 💡 Cấu trúc Node

```c
// Singly Linked List Node
struct Node {
    int data;           // Dữ liệu
    struct Node* next;  // Con trỏ đến node tiếp theo
};

// Doubly Linked List Node
struct DNode {
    int data;           // Dữ liệu
    struct DNode* prev; // Con trỏ đến node trước
    struct DNode* next; // Con trỏ đến node tiếp theo
};
```

## 🔍 Độ phức tạp

| Thao tác | Thời gian |
|----------|-----------|
| Truy cập | O(n) |
| Tìm kiếm | O(n) |
| Thêm đầu | O(1) |
| Thêm cuối | O(n) |
| Xóa đầu | O(1) |
| Xóa cuối | O(n) |

## 💻 Code mẫu hoàn chỉnh

```c
#include <stdio.h>
#include <stdlib.h>

struct Node {
    int data;
    struct Node* next;
};

void push(struct Node** head, int data) {
    struct Node* newNode = (struct Node*)malloc(sizeof(struct Node));
    newNode->data = data;
    newNode->next = *head;
    *head = newNode;
}

void printList(struct Node* node) {
    while (node != NULL) {
        printf("%d -> ", node->data);
        node = node->next;
    }
    printf("NULL\n");
}

int main() {
    struct Node* head = NULL;
    push(&head, 5);
    push(&head, 4);
    push(&head, 3);
    
    printf("List: ");
    printList(head);
    return 0;
}
```
**Kết quả**
```bash
List: 3 -> 4 -> 5 -> NULL
```

**Dưới đây là quản lý data structure:**
![alt text](/assets/DataStructure/LinkedList/Do1.png)

## 🛠️ Các thao tác với Linked List

### 1. Thao tác cơ bản với Node đầu (Head) 🎯

```c
// Thêm node vào đầu
void insertAtHead(Node** head, int data) {
    Node* newNode = (Node*)malloc(sizeof(Node));
    newNode->data = data;
    newNode->next = *head;
    *head = newNode;
}

// Xóa node đầu
void deleteHead(Node** head) {
    if (*head == NULL) return;
    Node* temp = *head;
    *head = (*head)->next;
    free(temp);
}
```

### 2. Thao tác với Node cuối (Tail) 🔚

```c
// Thêm node vào cuối
void insertAtTail(Node** head, int data) {
    Node* newNode = (Node*)malloc(sizeof(Node));
    newNode->data = data;
    newNode->next = NULL;
    
    if (*head == NULL) {
        *head = newNode;
        return;
    }
    
    Node* last = *head;
    while (last->next != NULL)
        last = last->next;
    last->next = newNode;
}

// Xóa node cuối
void deleteTail(Node** head) {
    if (*head == NULL) return;
    
    if ((*head)->next == NULL) {
        free(*head);
        *head = NULL;
        return;
    }
    
    Node* second_last = *head;
    while (second_last->next->next != NULL)
        second_last = second_last->next;
        
    free(second_last->next);
    second_last->next = NULL;
}
```

### 3. Thao tác tìm kiếm và đếm 🔍

```c
// Tìm kiếm phần tử
Node* search(Node* head, int key) {
    Node* current = head;
    while (current != NULL) {
        if (current->data == key)
            return current;
        current = current->next;
    }
    return NULL;
}

// Đếm số node
int countNodes(Node* head) {
    int count = 0;
    Node* current = head;
    while (current != NULL) {
        count++;
        current = current->next;
    }
    return count;
}
```

### 4. Thao tác đảo ngược danh sách 🔄

```c
// Đảo ngược danh sách
void reverse(Node** head) {
    Node *prev = NULL, *current = *head, *next = NULL;
    while (current != NULL) {
        next = current->next;
        current->next = prev;
        prev = current;
        current = next;
    }
    *head = prev;
}
```

### 5. Thao tác với vị trí bất kỳ 📍

```c
// Chèn vào vị trí bất kỳ
void insertAt(Node** head, int position, int data) {
    if (position < 0) return;
    
    Node* newNode = (Node*)malloc(sizeof(Node));
    newNode->data = data;
    
    if (position == 0) {
        newNode->next = *head;
        *head = newNode;
        return;
    }
    
    Node* current = *head;
    for (int i = 0; i < position - 1 && current != NULL; i++)
        current = current->next;
        
    if (current == NULL) return;
    
    newNode->next = current->next;
    current->next = newNode;
}
```

### 6. Kiểm tra và xử lý lỗi ⚠️

```c
// Kiểm tra danh sách rỗng
bool isEmpty(Node* head) {
    return head == NULL;
}

// Kiểm tra vòng lặp trong danh sách
bool hasLoop(Node* head) {
    Node *slow = head, *fast = head;
    while (fast != NULL && fast->next != NULL) {
        slow = slow->next;
        fast = fast->next->next;
        if (slow == fast)
            return true;
    }
    return false;
}
```