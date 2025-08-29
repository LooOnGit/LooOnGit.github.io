---
title: "Linked List"
date: 2025-07-29 23:02:00 +0800
categories: [Data Structure]
tags: [Data Structure]
---

# 📝 LINKED LIST (DANH SÁCH LIÊN KẾT)

## Danh sách liên kết là gì?
Danh sách liên kết là một tập hợp các node được phân bổ động, được sắp xếp theo cách mà mỗi node sẽ được chứa một giá trị và một con trỏ trỏ đến node tiếp theo nó. Nếu con trỏ là NULL, thì nó là nút cuối cùng trong danh sách.

Một danh sách được liên kết được giữ ằng cách sử dụng một biến con trỏ trỏ đến mục đầu tiên của danh sách. Nếu con trỏ đó cũng là NULL, thì danh sách được coi là trống.

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

## ⚖️ Ưu và nhược điểm

### Ưu điểm ✅
- Kích thước động
- Dễ dàng thêm/xóa phần tử
- Không cần cấp phát bộ nhớ liên tục

### Nhược điểm ❌
- Không truy cập ngẫu nhiên
- Cần thêm bộ nhớ cho con trỏ
- Không hiệu quả trong cache

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
    
    printf("Danh sách liên kết: ");
    printList(head);
    return 0;
}
```
