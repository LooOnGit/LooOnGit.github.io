---
title: "Factory Method"
date: 2026-04-02 03:39:05 +0800
categories: [Design Patterns]
tags: [Design Patterns][Creational patterns]
---

# FACTORY METHOD

## Intent
**Factory Method** là một creatial design pattern để cung cấp giao diện để tạo ra các đối tượng trong một lớp cha (superclass), nhung cho phép các lớp con (subclasses) thay đổi loại đối tượng sẽ được tạo ra.

## Problem
Giả sử tọa ra một logistic app để vận chuyển, thì ban đầu trên cạn thì vận chuyển bằng xe tải đi thì tạo ok rồi. Nhưng một ngày khách yêu cầu thêm vận tải biển, thì xe tải không thể phải tạo thêm vận tải đường biển vì vậy phải sửa source thêm các điều kiện các thứ làm có code trở nên rất tệ.

![alt text](/assets/Design_Patterns/Factory_method/image1.png)

## Solution
**Factory Method** pattern đề xuất rằng nên gọi direct object construction (sử dụng `new` operation). Các object được return bằng factory method được gọi là product.

![alt text](/assets/Design_Patterns/Factory_method/image2.png)

## Structure

## Pseudocode

## Applicability

## How to Implement

## Pros and Cons

## Relations with Other Patterns