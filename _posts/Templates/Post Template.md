<%*
// Chuẩn front matter của repo LooOnGit.github.io (Jekyll - theme Chirpy)
// category/tag lấy theo tên folder chứa note (ví dụ: _posts/Kernel/... -> categories: [Kernel])
const category = tp.file.folder(false);
const originalTitle = tp.file.title;
const dateStr = tp.date.now("YYYY-MM-DD");

// Đổi tên file thành YYYY-MM-DD-Ten-Bai.md theo đúng convention Jekyll của repo
await tp.file.rename(`${dateStr}-${originalTitle}`);
-%>
---
title: "<% originalTitle %>"
date: <% tp.date.now("YYYY-MM-DD HH:mm:ss") %> +0700
categories: [<% category %>]
tags: [<% category %>]
---

# <% originalTitle %>
