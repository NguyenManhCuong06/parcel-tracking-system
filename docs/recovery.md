# Hướng dẫn khôi phục và xử lý sự cố Git

Tài liệu này mô tả các quy trình khôi phục dữ liệu, hoàn tác thay đổi và xử lý sự cố phổ biến khi làm việc với repository Parcel Tracking System.

## Mục lục
1. [Xem lịch sử thay đổi](#1-xem-l%E1%BB%8Bch-s%E1%BB%AD-thay-%C4%91%E1%BB%95i)
2. [Hoàn tác commit đã push](#2-ho%C3%A0n-t%C3%A1c-commit-%C4%91%C3%A3-push)
3. [Khôi phục file từ commit cũ](#3-kh%E1%BB%8Fi-ph%E1%BB%A5c-file-t%E1%BB%AB-commit-c%E1%BB%AD)
4. [Quay về tag checkpoint](#4-quay-v%E1%BB%81-tag-checkpoint)
5. [Sử dụng git reflog](#5-s%E1%BB%AD-d%E1%BB%A5ng-git-reflog)
6. [Xử lý file bị xóa nhầm](#6-x%E1%BB%AD-l%C3%BD-file-b%E1%BB%8B-x%C3%B3a-nh%E1%BA%A7m)
7. [Xử lý merge conflict](#7-x%E1%BB%AD-l%C3%BD-merge-conflict)
8. [Kiểm tra file nhạy cảm trong lịch sử](#8-ki%E1%BB%83m-tra-file-nh%E1%BA%Aạy-c%E1%BA%A3m-trong-l%E1%BB%8Bch-s%E1%BB%AD)

---

## 1. Xem lịch sử thay đổi

```bash
# Xem lịch sử commit dạng đồ họa
git log --oneline --graph --decorate -n 20

# Xem lịch sử chi tiết của một file
git log --oneline -- <đường-dẫn-file>

# Xem thay đổi của một commit cụ thể
git show <hash-commit>

# Xem diff giữa hai commit
git diff <hash-cũ> <hash-mới> -- <đường-dẫn-file>
```

## 2. Hoàn tác commit đã push

### 2.1 Tạo commit đảo ngược (git revert) - AN TOÀN
```bash
# Hoàn tác 1 commit cụ thể
git revert <hash-commit>

# Hoàn tác nhiều commit liên tiếp
git revert <hash-cũ>..<hash-mới>

# Hoàn tác commit merge (cần chỉ định parent)
git revert -m 1 <hash-merge-commit>
```
> **Lưu ý**: `git revert` tạo commit mới đảo ngược thay đổi, giữ nguyên lịch sử. Phù hợp cho commit đã push lên remote.

### 2.2 Khôi phục nhánh về trạng thái trước (CHỈ DÙNG KHI CHƯA PUSH)
```bash
# MỀM: giữ thay đổi trong working tree
git reset --soft HEAD~1

# CỨNG: xóa cả thay đổi trong working tree
git reset --hard HEAD~1
```
> **CẢNH BÁO**: Không dùng `git reset --hard` trên commit đã push. Nếu cần, dùng `git revert`.

## 3. Khôi phục file từ commit cũ

```bash
# Xem nội dung file tại commit cũ
git show <hash-commit>:<đường-dẫn-file>

# Khôi phục file về trạng thái commit cũ (ghi đè file hiện tại)
git checkout <hash-commit> -- <đường-dẫn-file>

# Khôi phục file và stage luôn
git restore --staged --source=<hash-commit> -- <đường-dẫn-file>
```

### Ví dụ thực tế
```bash
# Khôi phục architecture.md từ commit 861c529
git show 861c529:architecture.md > docs/architecture.md

# Khôi phục README.md từ commit 6eabd63
git checkout 6eabd63 -- README.md
```

## 4. Quay về tag checkpoint

```bash
# Liệt kê các tag checkpoint
git tag -l "checkpoint/*"

# Checkout về tag checkpoint (detached HEAD)
git checkout checkpoint/m0-start

# Tạo nhánh mới từ tag để làm việc tiếp
git checkout -b fix/something checkpoint/m0-start
```

## 5. Sử dụng git reflog

`git reflog` ghi lại mọi thay đổi HEAD (commit, checkout, reset, rebase, merge...). Dùng để khôi phục khi lỡ `git reset --hard` hoặc `git checkout` nhầm.

```bash
# Xem reflog
git reflog

# Xem reflog của nhánh cụ thể
git reflog main

# Khôi phục về trạng thái trước reset
git reset --hard HEAD@{1}  # HEAD@{1} là trạng thái trước lệnh reset cuối

# Tìm commit đã mất trong reflog
git reflog | grep "commit message fragment"
```

### Ví dụ khôi phục sau `git reset --hard` nhầm
```bash
# 1. Xem reflog tìm commit muốn quay về
git reflog
# Output ví dụ:
# a1b2c3d HEAD@{0}: reset: moving to HEAD~1
# e4f5g6h HEAD@{1}: commit: feat: add new feature

# 2. Quay về commit trước reset
git reset --hard e4f5g6h
# hoặc
git reset --hard HEAD@{1}
```

## 6. Xử lý file bị xóa nhầm

```bash
# File chưa commit (chỉ có trong working tree)
git checkout -- <đường-dẫn-file>

# File đã stage (git add)
git restore --staged <đường-dẫn-file>
git checkout -- <đường-dẫn-file>

# File đã commit rồi bị xóa trong commit sau
git checkout <hash-commit-trước-khi-xóa> -- <đường-dẫn-file>
```

## 7. Xử lý merge conflict

```bash
# 1. Khi merge bị conflict
git merge feature/xyz
# CONFLICT (content): Merge conflict in <file>

# 2. Xem file conflict
git status
# Unmerged paths: both modified: <file>

# 3. Mở file, tìm dấu >>>>>>> ======= <<<<<<<, sửa thủ công
# Hoặc dùng tool: git mergetool

# 4. Sau khi sửa xong
git add <file>
git commit
```

### Chiến lược merge an toàn (--no-ff)
```bash
# Giữ lịch sử nhánh, dễ revert cả nhánh
git merge --no-ff feature/xyz
```

## 8. Kiểm tra file nhạy cảm trong lịch sử

### Tìm file nhạy cảm từng bị commit
```bash
# Tìm file .env, .pem, .key từng bị commit
git log --all --full-history --name-only -- "*.env" "*.pem" "*.key" "*.pfx" "*.p12" "secrets.json" | grep -E "^\w" | sort -u

# Tìm secret pattern trong lịch sử (cần cài gitleaks hoặc truffleHog)
# Gitleaks: https://github.com/gitleaks/gitleaks
```

### Nếu phát hiện secret đã lọt vào lịch sử
1. **DỪNG LẠI** - không tự ý rewrite history
2. Báo ngay cho chủ dự án
3. Thu hồi/đổi secret **TRƯỚC KHI** bàn cách xử lý lịch sử
4. Cân nhắc: `git filter-repo`, BFG Repo-Cleaner, hoặc rotate secret

---

## Tag checkpoint quan trọng

| Tag | Mô tả | Commit |
|-----|-------|--------|
| `checkpoint/m0-start` | Trước khi bắt đầu M0 | `df4cb0c` |
| `checkpoint/m0-done` | Sau khi hoàn tất M0 | (sẽ tạo) |

---

## Lệnh hữu ích nhanh

```bash
# Xem file đang bị ignore
git check-ignore -v <file>

# Xem file đang được track
git ls-files

# Xem file modified/staged
git status -sb

# Xem diff staged
git diff --staged

# Xem diff working tree
git diff

# Xóa file khỏi cache (giữ file local)
git rm --cached <file>

# Xóa thư mục khỏi cache
git rm -r --cached <thư-mục>

# Clean working tree (CHỈ DÙNG KHI CHẮC CHẮN)
git clean -fd  # xóa untracked files và directories
```

---

## Quy tắc an toàn bắt buộc

1. **KHÔNG** `git push --force` / `--force-with-lease` khi chưa có sự đồng ý
2. **KHÔNG** `git rebase` commit đã push
3. **KHÔNG** `git commit --amend` commit đã push
4. **KHÔNG** `git filter-repo`, `git branch -D`, xóa tag khi chưa được phép
5. **LUÔN** dùng `git revert` cho commit đã push
6. **LUÔN** tạo tag checkpoint trước khi làm việc rủi ro
7. **LUÔN** chạy `dotnet build` trước khi commit

---

*Cập nhật: 2026-11-21 - Phiên bản M0*