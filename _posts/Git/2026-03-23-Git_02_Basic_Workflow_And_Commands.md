---
title: "Git 02 - Vòng Đời File và Các Lệnh Cơ Bản"
date: 2026-03-23 20:00:00 +0700
categories: [Git]
tags: [Git]
---

# GIT 02 - VÒNG ĐỜI FILE VÀ CÁC LỆNH CƠ BẢN

## Khởi tạo Repository

Có 2 cách để bắt đầu làm việc với một Git repo:

### 1. Khởi tạo từ thư mục có sẵn
```bash
cd my_project/
git init
```
Lệnh này tạo ra một thư mục ẩn `.git/` chứa toàn bộ cơ sở dữ liệu của Git.

### 2. Clone một repository có sẵn
```bash
git clone https://github.com/user/repo.git

# Clone và đặt tên thư mục khác
git clone https://github.com/user/repo.git my_folder_name
```

## Vòng đời của một File trong Git

Đây là khái niệm quan trọng nhất cần nắm vững. Mỗi file trong Working Directory của bạn ở trong một trong 2 trạng thái lớn: **tracked** hoặc **untracked**.

```
  Untracked ----git add----> Staged
  Unmodified                 |
      |            <---------+---git commit
  (edit file)                        |
      |                              v
  Modified ----git add----> Staged  Unmodified
```

Chi tiết 4 trạng thái:

| Trạng thái | Mô tả |
| :--- | :--- |
| **Untracked** | File mới, Git chưa biết đến sự tồn tại của file này |
| **Unmodified** | File đã được Git theo dõi, chưa có thay đổi nào so với commit cuối |
| **Modified** | File đã được chỉnh sửa nhưng chưa được đưa vào Staging Area |
| **Staged** | File đã được đánh dấu để đưa vào commit tiếp theo |

## Kiểm tra trạng thái

```bash
git status
```
Đây là lệnh bạn sẽ dùng thường xuyên nhất. Nó báo cáo toàn bộ trạng thái của Working Directory và Staging Area.

Muốn xem ngắn gọn hơn:
```bash
git status -s
# Ví dụ kết quả:
# M  README.md     <- Modified và đã staged
#  M main.c        <- Modified nhưng chưa staged
# ?? newfile.txt   <- Untracked
```

## Thêm file vào Staging Area (git add)

```bash
# Thêm một file cụ thể
git add main.c

# Thêm tất cả file trong thư mục hiện tại
git add .

# Thêm tất cả file có phần mở rộng .c
git add *.c
```

## Bỏ qua file không cần track (.gitignore)

Tạo file `.gitignore` ở thư mục gốc của project để Git tự động bỏ qua các file không cần thiết:

```gitignore
# Bỏ qua file object và binary
*.o
*.bin
build/

# Bỏ qua file cấu hình IDE
.vscode/
*.suo

# Bỏ qua file log
*.log
```

## Tạo Commit (git commit)

Commit là việc "chụp ảnh" (snapshot) lại trạng thái hiện tại của Staging Area và lưu vào lịch sử.

```bash
# Mở editor để viết commit message
git commit

# Viết commit message trực tiếp trên lệnh
git commit -m "feat: add timer initialization function"

# Thêm toàn bộ file đã tracked vào stage và commit luôn (bỏ qua bước git add)
git commit -am "fix: correct timer reload value"
```

### Viết commit message tốt

Một commit message tốt nên ngắn gọn, rõ ràng và mô tả **tại sao** thay đổi, không chỉ **cái gì** thay đổi.

**Format phổ biến (Conventional Commits):**
```
<type>: <description>

[optional body]
```

Các `type` hay dùng:
- `feat`: Thêm tính năng mới
- `fix`: Sửa bug
- `docs`: Cập nhật tài liệu
- `refactor`: Tái cấu trúc code
- `chore`: Công việc bảo trì (update dependency, config,...)

## Xem lịch sử Commit (git log)

```bash
# Xem toàn bộ lịch sử
git log

# Xem ngắn gọn, mỗi commit 1 dòng
git log --oneline

# Vẽ đồ thị nhánh
git log --oneline --graph --all

# Xem N commit gần nhất
git log -5
```

Ví dụ output của `git log --oneline`:
```
a1b2c3d (HEAD -> main) feat: add UART driver
e4f5g6h fix: resolve SPI clock polarity issue
i7j8k9l Initial commit
```

## Bỏ Staged (git restore / git reset)

```bash
# Bỏ stage một file (file vẫn còn thay đổi ở Working Directory)
git restore --staged main.c

# Cú pháp cũ hơn
git reset HEAD main.c

# Huỷ bỏ mọi thay đổi ở Working Directory (CẨNTHẬN: không thể hoàn tác!)
git restore main.c
```

## Xem sự khác biệt (git diff)

```bash
# Xem thay đổi trong Working Directory chưa được staged
git diff

# Xem thay đổi đã được staged (so sánh với commit cuối)
git diff --staged
```

---
**Bài trước**: [Git 01 - Giới Thiệu và Cài Đặt](/posts/Git_01_Introduction_And_Setup/)
**Bài tiếp theo**: [Git 03 - Branch và Merge](/posts/Git_03_Branching_And_Merging/)
