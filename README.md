# Báo Cáo Bài 3: Xử Lý Xung Đột Phức Tạp Trong Quá Trình Rebase

- **Thực hiện:** Học viên  
- **Thư mục bài tập:** `homework/session_05/ex3/`  
- **Mục tiêu:** Hiểu quy trình giải quyết xung đột từng chặng (step-by-step conflict resolution) khi thực thi `git rebase`, đưa lịch sử nhánh tính năng thẳng hàng lên trên đầu nhánh chính mà không tạo Merge Commit.

---

## 1. Bối cảnh & Mô phỏng xung đột

Nhánh `main` và nhánh `feature-api` cùng thay đổi các dòng tương tự trong file cấu hình `config.json`. 

- **Nhánh `main` có 2 commit mới:**
  1. `update port on main`: Sửa `"port": 8081`
  2. `add env config`: Thêm `"env": "production"`

- **Nhánh `feature-api` có 2 commit mới:**
  1. `feat: change port`: Sửa `"port": 9000`
  2. `feat: enable debug`: Sửa `"debug": true`

Khi rebase `feature-api` lên `main`, Git sẽ lần lượt tua lại (replay) từng commit của `feature-api` lên trên đỉnh `main`, gây ra xung đột ở từng bước.

---

## 2. Các bước thực hiện & Mô phỏng môi trường

### Bước 1: Khởi tạo kho chứa & Commit gốc trên `main`
```bash
echo '{"port": 8080, "debug": false}' > config.json
git add config.json
git commit -m "init config"

```

### Bước 2: Tạo nhánh `feature-api` và thực hiện 2 commit

```bash
git checkout -b feature-api

# Commit 1 trên feature-api
echo '{"port": 9000, "debug": false}' > config.json
git add config.json
git commit -m "feat: change port"

# Commit 2 trên feature-api
echo '{"port": 9000, "debug": true}' > config.json
git add config.json
git commit -m "feat: enable debug"

```

### Bước 3: Tạo thay đổi song song trên `main`

```bash
git checkout main

# Commit 1 mới trên main
echo '{"port": 8081, "debug": false}' > config.json
git add config.json
git commit -m "update port on main"

# Commit 2 mới trên main
echo '{"port": 8081, "debug": false, "env": "production"}' > config.json
git add config.json
git commit -m "add env config"

```

---

## 3. Quy trình Rebase & Giải quyết Xung đột từng chặng

Quay lại nhánh `feature-api` và bắt đầu Rebase:

```bash
git checkout feature-api
git rebase main

```

---

### Chặng 1: Conflict khi replay commit `feat: change port`

**1. Thông báo lỗi từ Git:**

```text
Auto-merging config.json
CONFLICT (content): Merge conflict in config.json
error: Could not apply 1234567... feat: change port

```

**2. Nội dung xung đột trong file `config.json`:**

```json
<<<<<<< HEAD (add env config)
{"port": 8081, "debug": false, "env": "production"}
=======
{"port": 9000, "debug": false}
>>>>>>> feat: change port

```

**3. Cách giải quyết:**

Kết hợp giá trị port mới của `feature-api` (`9000`) và giữ lại cấu hình `env` vừa cập nhật ở `main`:

```json
{
  "port": 9000,
  "debug": false,
  "env": "production"
}

```

**4. Đánh dấu đã giải quyết và tiếp tục Rebase:**

```bash
git add config.json
git rebase --continue

```

---

### Chặng 2: Conflict khi replay commit `feat: enable debug`

**1. Thông báo lỗi từ Git:**

```text
Auto-merging config.json
CONFLICT (content): Merge conflict in config.json
error: Could not apply 89abcde... feat: enable debug

```

**2. Nội dung xung đột trong file `config.json`:**

```json
<<<<<<< HEAD (feat: change port)
{"port": 9000, "debug": false, "env": "production"}
=======
{"port": 9000, "debug": true}
>>>>>>> feat: enable debug

```

**3. Cách giải quyết:**

Giữ lại `"debug": true` từ commit `feat: enable debug` đồng thời đảm bảo thông số `"env": "production"` không bị mất:

```json
{
  "port": 9000,
  "debug": true,
  "env": "production"
}

```

**4. Đánh dấu đã giải quyết và hoàn tất Rebase:**

```bash
git add config.json
git rebase --continue

```

**Kết quả:** Trình soạn thảo báo thành công: `Successfully rebased and updated refs/heads/feature-api.`

---

## 4. Kiểm tra kết quả

### 1. Kiểm tra sơ đồ lịch sử commit (`git log --graph --oneline`)

```bash
git log --graph --oneline

```

**Kết quả hiển thị:**

```text
* f3a4b5c (HEAD -> feature-api) feat: enable debug
* e2d3c4b feat: change port
* d1c2b3a (main) add env config
* c0b1a29 update port on main
* b9a8f7e init config

```

> **Đánh giá:** Lịch sử Git hoàn toàn thẳng tắp, các commit của nhánh `feature-api` nằm nối tiếp ngay sau commit mới nhất của nhánh `main` mà không xuất hiện bất kỳ Merge Commit nào.

### 2. Kiểm tra nội dung tệp `config.json` sau cùng:

```bash
cat config.json

```

**Kết quả:**

```json
{
  "port": 9000,
  "debug": true,
  "env": "production"
}

```
