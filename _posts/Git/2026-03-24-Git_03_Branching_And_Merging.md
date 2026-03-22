---
title: "Git 03 - Branching và Merging"
date: 2026-03-24 20:00:00 +0700
categories: [Git]
tags: [Git]
---

# GIT 03 - BRANCHING VÀ MERGING

## Branch là gì?

**Branch (nhánh)** là một trong những tính năng mạnh nhất của Git. Branch cho phép bạn tách ra một "dòng phát triển" độc lập từ dòng chính mà không ảnh hưởng gì đến nhau.

**Ví dụ thực tế**: Bạn đang làm firmware cho một board MCU (nhánh `main`). Khách hàng muốn thêm tính năng WiFi, bạn tạo nhánh `feature/wifi`. Trong khi làm, có bug cấp bách trên bản production, bạn tạo nhánh `hotfix/uart-bug` từ `main` để sửa. Tất cả diễn ra song song, không ảnh hưởng nhau.

Trong Git, một branch thực chất chỉ là một **con trỏ nhẹ** (lightweight pointer) trỏ đến một commit cụ thể. Con trỏ đặc biệt `HEAD` luôn trỏ đến branch bạn đang làm việc.

## Làm việc với Branch

### Tạo Branch

```bash
# Xem danh sách tất cả các branch (dấu * là branch hiện tại)
git branch

# Tạo branch mới (chưa chuyển sang)
git branch feature/wifi

# Tạo branch mới VÀ chuyển sang luôn (cách phổ biến hơn)
git switch -c feature/wifi
# hoặc cú pháp cũ hơn:
git checkout -b feature/wifi
```

### Chuyển Branch

```bash
# Chuyển sang branch khác
git switch main
# hoặc cú pháp cũ:
git checkout main
```

### Xóa Branch

```bash
# Xóa branch đã được merge (an toàn)
git branch -d feature/wifi

# Xóa branch dù chưa được merge (CẢNH BÁO: mất code!)
git branch -D feature/wifi
```

## Ví dụ luồng làm việc với Branch

```bash
# Đang ở nhánh main, tạo nhánh tính năng mới
git switch -c feature/adc-driver

# ... code, code, code ...
git add drivers/adc.c
git commit -m "feat: add ADC initialization"

# ... code tiếp ...
git add drivers/adc.c
git commit -m "feat: implement ADC read function"

# Xong tính năng, quay về main để merge
git switch main
```

## Merge - Gộp Nhánh

Sau khi hoàn thành công việc trên một nhánh, bạn cần đưa các thay đổi đó trở về nhánh chính bằng lệnh `merge`.

```bash
# Đang ở nhánh main
git merge feature/adc-driver
```

### Fast-forward Merge

Xảy ra khi nhánh `main` không có commit mới nào kể từ khi tạo ra `feature/adc-driver`. Git đơn giản chỉ "tua nhanh" con trỏ `main` lên commit mới nhất của `feature/adc-driver`.

```
Trước merge:
  main: A --- B
               \
  feature:      C --- D

Sau fast-forward merge:
  main: A --- B --- C --- D
```

### 3-Way Merge

Xảy ra khi cả `main` và nhánh `feature` đều có commit mới. Git tạo ra một **merge commit** mới để kết hợp cả hai.

```
Trước merge:
  main: A --- B --- E
               \
  feature:      C --- D

Sau 3-way merge:
  main: A --- B --- E --- M (merge commit)
               \         /
  feature:      C --- D
```

## Giải quyết Xung đột (Conflict Resolution)

Xung đột xảy ra khi cả 2 nhánh cùng chỉnh sửa **cùng một dòng** trong cùng một file. Git sẽ dừng lại và yêu cầu bạn tự giải quyết.

```bash
git merge feature/wifi
# OUTPUT: CONFLICT (content): Merge conflict in src/config.h
# Automatic merge failed; fix conflicts and then commit the result.
```

Kiểm tra file bị xung đột:
```bash
git status
# Both modified: src/config.h
```

Mở file `src/config.h`, bạn sẽ thấy các marker như sau:

```c
<<<<<<< HEAD          // Nội dung trên nhánh bạn đang ở (main)
#define WIFI_SSID "NetworkA"
=======               // Phân cách
#define WIFI_SSID "NetworkB"
>>>>>>> feature/wifi  // Nội dung từ nhánh đang merge vào
```

**Cách giải quyết**: Sửa file thủ công (xóa các marker `<<<<<<<`, `=======`, `>>>>>>>`), giữ lại nội dung đúng:

```c
// Giữ lại cái đúng, xóa hết markers
#define WIFI_SSID "NetworkA_Final"
```

Sau khi sửa xong, đánh dấu đã giải quyết và commit:
```bash
git add src/config.h
git commit -m "merge: resolve wifi SSID conflict"
```

## Rebase - Thay thế cho Merge

`rebase` là một cách khác để tích hợp thay đổi, tạo ra lịch sử commit **tuyến tính** và gọn gàng hơn. Thay vì tạo merge commit, rebase "xếp lại" các commit của bạn lên trên đỉnh của nhánh đích.

```bash
# Đang ở feature/wifi, rebase lên main mới nhất
git rebase main
```

> **QUAN TRỌNG**: Không dùng `rebase` trên các nhánh **đã được push lên remote** và **đang có người khác dùng**. Vì rebase viết lại lịch sử, sẽ gây rối loạn cho người khác.

---
**Bài trước**: [Git 02 - Vòng Đời File và Các Lệnh Cơ Bản](/posts/Git_02_Basic_Workflow_And_Commands/)
**Bài tiếp theo**: [Git 04 - Remote Repository và Cộng tác](/posts/Git_04_Remote_Repositories_And_Collaboration/)
