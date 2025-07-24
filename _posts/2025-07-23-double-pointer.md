---
title: "DOUBLE POINTER"
date: 2025-07-23 09:17:00 +0800
categories: [C]
tags: [C]
---

# 💡 Hướng dẫn về Con trỏ cấp 2 trong C

ví dụ thực tế và giải thích rõ ràng về **con trỏ cấp 2 (double pointer)** trong lập trình C.

## 🧠 Tổng quan

Con trỏ cấp 2 là **con trỏ trỏ đến một con trỏ khác**. Nó giúp chúng ta thao tác sâu hơn với bộ nhớ, thay đổi địa chỉ con trỏ trong hàm, quản lý mảng 2 chiều, và nhiều tình huống nâng cao khác trong lập trình C.

---

## 🏷️ Con trỏ cấp 2 là gì?

- Con trỏ cấp 2 là một biến lưu **địa chỉ của một con trỏ khác**.
- Ký hiệu: `**`, ví dụ: `int **pptr;`
- Có thể được hiểu như:  
  `**pptr` → giá trị của biến gốc  
  `*pptr` → con trỏ cấp 1  
  `pptr` → con trỏ cấp 2

---

## ⚙️ Cách khai báo và sử dụng

### Khai báo:

```c
int a = 5;
int *p = &a;       // con trỏ cấp 1
int **pp = &p;     // con trỏ cấp 2
```
## 🔍 Ví dụ cơ bản
![alt text](/assets/C/Double_pointer.png)
![alt text](/assets/C/Double_pointer_2.png)

Con trỏ cấp 1 lưu trữ địa chỉ của một biến còn con trỏ cấp 2 lưu trữ địa chỉ của 1 con trỏ.

## ⚙️ Con trỏ cấp 2 cấp phát động
### Ví dụ 1:
```c
#include <stdio.h>

void capphat(int **ptt, int n)
{
    int **p = (int **)ptt;
    *p = calloc(n, sizeof(int));
}

int main()
{
    int *pt = NULL;
    capphat(&pt, 10);
    pt[0] = 100;
    printf("%d", pt[0]);
    return 0;
}
```

## 🛠️ Ứng dụng của con trỏ cấp 2

### Ví dụ 2:
```c
#include <stdio.h>
#define KICHTHUOC 3
const int KICHCO = 4;

void xuatmang(int *pt, int sophantu)
{
    for (int i = 0; i < sophantu; i++)
    {
        printf("%d ", pt[i]);
    }
}

int main()
{
    int pt1[5] = {1, 2, 3, 4, 5};
    int pt2[5] = {11, 12, 13, 14, 15};
    int pt3[5] = {21, 22, 23, 24, 25};

    int *contro[KICHTHUOC];
    contro[0] = pt1;
    contro[1] = pt2;
    contro[2] = pt3;

    char *hotensv[] = {
        "Tran Hung Cuong",
        "Ho Ngoc Ha",
        "Nguyen Son Tung",
        "Dam Vinh Hung",
    };

    for (int i = 0; i < KICHCO; i++)
    {
        printf("gia tri cua hotensv[%d] = %s\n", i, hotensv[i]);
    }

    // contro[0][0] = 111; // thay doi gia tri
    xuatmang(contro[0], 5);

    return 0;
}
```
- Tác dụng thứ nhất là thay đổi địa chỉ của con trỏ 1 chiều.
- Tác dụng thứ 2 cấp phát động 1 mảng con trỏ.

### Ví dụ 3:
```c
#include <stdio.h>

void xuatmang(int *pt, int sophantu)
{
    for (int i = 0; i < sophantu; i++)
    {
        printf("%d ", pt[i]);
    }
}

int main()
{
    int mang1[5] = {1, 2, 3, 4, 5};
    int mang2[5] = {11, 12, 13, 14, 15};
    int mang3[5] = {21, 22, 23, 24, 25};
    int mang4[5] = {31, 32, 33, 34, 35};
    int mang5[5] = {41, 42, 43, 44, 45};

    int *pt0 = (int *)calloc(5, sizeof(int *));
    xuatmang(pt0, 5);

    return 0;
}
```
- Vì hàm calloc trả về kiểu void là không có kiểu nên (int **) là để ép kiểu.

### Ví dụ 4: Tác dụng thứ 3 cấp phát động mảng 2 chiều
```c
#include <stdio.h>
#include <stdlib.h>

void NhapMaTran(int **a, int dong, int cot)
{
    int i, j;
    for (i = 0; i < dong; i++)
    for (j = 0; j < cot; j++)
    {
        printf("a[%d][%d] - ", i, j);
        scanf("%d", &a[i][j]);
    }
}

void XuatMaTran(int **a, int dong, int cot)
{
    int i, j;
    for (i = 0; i < dong; i++)
    {
        for (j = 0; j < cot; j++)
            printf("%5d ", a[i][j]);
        printf("\n");
    }
}

int main()
{
    int **a = NULL, dong, cot;
    printf("Nhap vao so dong - ");
    scanf("%d", &dong);
    printf("Nhap vao so cot - ");
    scanf("%d", &cot);

    // Cấp phát mảng chỉ mục cho từng dòng 1
    a = (int **)malloc(dong * sizeof(int *));
    for (i = 0; i < dong; i++)
    {
        // Cấp phát cho từng dòng 1
        a[i] = (int *)malloc(cot * sizeof(int));
    }
    NhapMaTran(a, dong, cot);
    XuatMaTran(a, dong, cot);

    // Giải phóng con trỏ mảng lý các dòng
    for (i = 0; i < dong; i++)
    {
        free(a[i]);
    }
    // Giải phóng con trỏ mảng
    free(a);

    return 0;
}
```