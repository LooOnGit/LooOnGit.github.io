---
title: "Git 01 - Giới Thiệu và Cài Đặt"
date: 2026-03-22 20:00:00 +0700
categories: [Git]
tags: [Git]
---

# GIT 01 - GIỚI THIỆU VÀ CÀI ĐẶT

## Git là gì?

**Git** là một hệ thống quản lý phiên bản phân tán (**Distributed Version Control System - DVCS**), được tạo ra bởi **Linus Torvalds** vào năm 2005 khi phát triển Linux kernel. Git giúp:

- **Lưu lại lịch sử** thay đổi của dự án theo từng mốc thời gian.
- **Nhiều người** có thể cùng làm việc trên một dự án mà không gây xung đột.
- **Phục hồi** lại bất kỳ phiên bản nào trong quá khứ khi cần thiết.

## VCS là gì và tại sao cần nó?

**Version Control System (VCS)** là công cụ giúp theo dõi lịch sử thay đổi của các file. Hãy tưởng tượng bạn đang chỉnh sửa file firmware cho một con MCU. Không có VCS, nếu lỡ tay xóa mất đoạn code quan trọng, bạn sẽ gặp rắc rối lớn. Với Git, mọi thứ đã được "snapshot" lại và có thể phục hồi bất cứ lúc nào.

Có 3 thế hệ VCS:
- **Local VCS**: Lưu trữ toàn bộ trên máy tính cá nhân (dễ mất dữ liệu).
- **Centralized VCS (CVCS)**: Một server trung tâm lưu trữ, nhiều người kết nối vào (SVN, CVS). Nếu server hỏng, mọi người mất quyền làm việc.
- **Distributed VCS (DVCS)**: Mỗi người clone toàn bộ lịch sử về máy. **Git** thuộc loại này.

## Cài đặt Git

### Trên Windows
Tải và cài đặt từ [https://git-scm.com/downloads](https://git-scm.com/downloads). Trong quá trình cài, chọn **Git Bash** để có terminal tốt hơn.

Kiểm tra sau khi cài:
```bash
git --version
# Kết quả ví dụ: git version 2.44.0.windows.1
```

### Trên Linux (Ubuntu/Debian)
```bash
sudo apt update
sudo apt install git
```

### Trên macOS
```bash
brew install git
```

## Cấu hình Git lần đầu

Sau khi cài, điều quan trọng đầu tiên là khai báo **danh tính** của bạn. Thông tin này sẽ được đính kèm vào mỗi commit bạn tạo ra.

```bash
# Thiết lập tên người dùng (--global áp dụng cho toàn máy)
git config --global user.name "Tên Của Bạn"

# Thiết lập email
git config --global user.email "email@example.com"

# Thiết lập editor mặc định (ví dụ: VS Code)
git config --global core.editor "code --wait"
```

Kiểm tra lại cấu hình:
```bash
git config --list
```

### Phạm vi cấu hình

Git có 3 cấp độ cấu hình, cấp thấp hơn sẽ ghi đè cấp cao hơn:

| Cấp độ | Flag | Nơi lưu | Phạm vi |
| :--- | :--- | :--- | :--- |
| System | `--system` | `/etc/gitconfig` | Toàn bộ hệ thống |
| Global | `--global` | `~/.gitconfig` | Toàn bộ user hiện tại |
| Local | `--local` | `.git/config` | Chỉ repository hiện tại |

## Luồng làm việc cơ bản của Git (tổng quan)

Git quản lý dữ liệu theo 3 vùng chính:

```
[Working Directory] ---git add---> [Staging Area] ---git commit---> [Repository (.git)]
```

- **Working Directory**: Thư mục dự án thực tế bạn đang chỉnh sửa.
- **Staging Area (Index)**: Vùng "chuẩn bị", nơi bạn chọn những thay đổi muốn đưa vào commit tiếp theo.
- **Repository**: Cơ sở dữ liệu Git (nằm trong thư mục ẩn `.git`), lưu toàn bộ lịch sử.

---
**Bài tiếp theo**: [Git 02 - Vòng Đời File và Các Lệnh Cơ Bản](/posts/Git_02_Basic_Workflow_And_Commands)
