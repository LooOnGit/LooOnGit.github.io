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

**Khác cấp phát động và Cấp phát tĩnh:**


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

## 🛠️ Các thao tác cơ bản

### 1. Thêm node mới (Insert) ➕

```c
// Thêm vào đầu danh sách
void insertAtBeginning(struct Node** head, int data) {
    struct Node* newNode = (struct Node*)malloc(sizeof(struct Node));
    newNode->data = data;
    newNode->next = *head;
    *head = newNode;
}
```

### 2. Xóa node (Delete) ❌

```c
void deleteNode(struct Node** head, int key) {
    struct Node *temp = *head, *prev = NULL;
    
    if (temp != NULL && temp->data == key) {
        *head = temp->next;
        free(temp);
        return;
    }
    
    while (temp != NULL && temp->data != key) {
        prev = temp;
        temp = temp->next;
    }
    
    if (temp == NULL) return;
    
    prev->next = temp->next;
    free(temp);
}
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