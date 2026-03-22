---
title: "Git 04 - Remote Repository và Cộng Tác"
date: 2026-03-25 20:00:00 +0700
categories: [Git]
tags: [Git]
---

# GIT 04 - REMOTE REPOSITORY VÀ CỘNG TÁC

## Remote Repository là gì?

Cho đến bây giờ, mọi thứ chúng ta làm đều trên máy tính cá nhân (local). **Remote Repository** là phiên bản repository được lưu trữ trên một máy chủ (server), ví dụ như **GitHub**, **GitLab**, hoặc **Bitbucket**.

Remote repo phục vụ 2 mục đích chính:
1. **Sao lưu (Backup)**: Code của bạn được lưu ở nơi an toàn, không sợ mất khi hỏng máy.
2. **Cộng tác (Collaboration)**: Nhiều người có thể cùng làm việc trên một dự án.

## Quản lý Remote (git remote)

```bash
# Xem danh sách các remote đang kết nối
git remote -v
# Kết quả ví dụ:
# origin  https://github.com/user/repo.git (fetch)
# origin  https://github.com/user/repo.git (push)

# Thêm một remote mới với tên 'origin' (tên mặc định, quy ước)
git remote add origin https://github.com/user/repo.git

# Đổi tên remote
git remote rename origin upstream

# Xóa remote
git remote remove upstream
```

## Đẩy Code lên Remote (git push)

```bash
# Push nhánh hiện tại lên remote 'origin'
# -u: thiết lập upstream (lần đầu cần, sau này chỉ cần 'git push')
git push -u origin main

# Push một nhánh mới lên remote
git push -u origin feature/wifi

# Xóa một nhánh trên remote
git push origin --delete feature/old-branch
```

## Tải code về từ Remote

### git fetch - Tải về nhưng KHÔNG gộp

```bash
# Tải toàn bộ thay đổi từ remote về local nhưng không thay đổi working directory
git fetch origin

# Sau khi fetch, so sánh với nhánh local
git log HEAD..origin/main --oneline
```

`fetch` an toàn hơn vì nó cho bạn xem thay đổi trước, rồi mới quyết định có merge hay không.

### git pull - Tải về VÀ gộp luôn

```bash
# Tương đương: git fetch + git merge
git pull

# pull với rebase thay vì merge (lịch sử gọn hơn)
git pull --rebase
```

> **Lời khuyên**: Nên dùng `git fetch` + `git merge` thay vì `git pull` cho đến khi bạn quen với Git. Điều này cho bạn quyền kiểm soát tốt hơn.

## Luồng làm việc nhóm với Pull Request

**Pull Request (PR)** (trên GitHub) hay **Merge Request (MR)** (trên GitLab) là cơ chế cộng tác cốt lõi. Đây là cách hoạt động:

### Luồng Fork & Pull (Open Source)
Dùng cho dự án mà bạn không có quyền ghi trực tiếp (ví dụ: đóng góp vào Linux kernel).

```
1. Fork repository gốc về account của bạn trên GitHub
2. Clone fork đó về máy:
   git clone https://github.com/YOUR_USER/repo.git

3. Thêm remote trỏ về repo gốc (upstream):
   git remote add upstream https://github.com/ORIGINAL_USER/repo.git

4. Tạo nhánh tính năng:
   git switch -c fix/typo-in-readme

5. Code, commit...
   git commit -m "fix: correct typo in README"

6. Push lên fork của bạn:
   git push -u origin fix/typo-in-readme

7. Mở Pull Request trên GitHub từ fork của bạn vào repo gốc.
```

### Luồng Feature Branch (Team)
Dùng trong team khi mọi người đều có quyền ghi vào cùng một repository.

```
1. Clone repo chính:
   git clone https://github.com/team/project.git

2. Tạo nhánh tính năng mới:
   git switch -c feature/add-mqtt-support

3. Code, commit thường xuyên...

4. Push nhánh lên remote:
   git push -u origin feature/add-mqtt-support

5. Mở Pull Request trên GitHub để review.

6. Sau khi được approve, merge vào main.

7. Xóa nhánh feature sau khi merge:
   git branch -d feature/add-mqtt-support
   git push origin --delete feature/add-mqtt-support
```

## Đồng bộ với Remote thường xuyên

Khi làm việc nhóm, cần cập nhật code mới nhất từ team thường xuyên để tránh conflict lớn:

```bash
# Đang ở nhánh feature/wifi
# Cập nhật main mới nhất về local
git fetch origin

# Rebase feature của bạn lên trên main mới nhất
git rebase origin/main

# Nếu có conflict, giải quyết xong rồi:
git rebase --continue
# Hoặc hủy rebase:
git rebase --abort
```

## Tóm tắt tổng quan các lệnh

| Lệnh | Mô tả |
| :--- | :--- |
| `git remote add origin <url>` | Kết nối repo local với remote |
| `git push -u origin <branch>` | Push và thiết lập upstream |
| `git fetch` | Tải thay đổi về, không merge |
| `git pull` | Tải về và merge |
| `git pull --rebase` | Tải về và rebase |

---
**Bài trước**: [Git 03 - Branching và Merging](/posts/Git_03_Branching_And_Merging)
