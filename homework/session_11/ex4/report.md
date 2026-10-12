# Báo cáo Bài 4: Khai báo GitHub Secrets và cấu hình Đăng nhập Container Registry

## 1. Mục tiêu
- Tạo tệp mã hóa an toàn (**Repository Secret**) lưu trữ Token cá nhân trên GitHub nhằm bảo mật thông tin xác thực.
- Cấu hình bước xác thực đăng nhập tự động vào **GitHub Container Registry (GHCR)** trong pipeline GitHub Actions bằng action `docker/login-action@v3`.
- Đảm bảo quy trình CI/CD có thể sẵn sàng cho các bước đóng gói và phát hành (push) container image lên GHCR mà không bị lộ mật khẩu/token trong mã nguồn.

---

## 2. Kịch bản CI hoàn chỉnh (`.github/workflows/ci.yml`)

Kịch bản workflow đã được khai báo và cấu hình tại [.github/workflows/ci.yml](../../../.github/workflows/ci.yml):

```yaml
name: Java CI with Gradle Caching and Registry Login

on:
  push:
    branches: [ "main", "master" ]
  pull_request:
    branches: [ "main", "master" ]
  workflow_dispatch:

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout mã nguồn
        uses: actions/checkout@v4

      - name: Thiết lập môi trường Java JDK 17
        uses: actions/setup-java@v5
        with:
          java-version: '17'
          distribution: 'temurin'
          cache: 'gradle'

      - name: Cấp quyền thực thi cho Gradle wrapper
        run: chmod +x gradlew

      - name: Chạy kiểm thử tự động với Gradle
        run: ./gradlew test --no-daemon

      - name: Đăng nhập vào GitHub Container Registry
        uses: docker/login-action@v3
        with:
          registry: ghcr.io
          username: ${{ github.actor }}
          password: ${{ secrets.REGISTRY_TOKEN }}
```

---

## 3. Các bước thực hiện

### Bước 1: Khởi tạo Personal Access Token (Classic)
1. Truy cập vào GitHub cá nhân -> **Settings** -> **Developer settings** -> **Personal access tokens** -> **Tokens (classic)**.
2. Chọn **Generate new token (classic)**.
3. Đặt tên Token (Note): `REGISTRY_TOKEN_GHCR`.
4. Chọn phạm vi quyền (Scopes):
   - Đánh dấu tích vào quyền **`write:packages`** (Upload packages to GitHub Package Registry). Khi chọn quyền này, hệ thống sẽ tự động kích hoạt kèm quyền **`read:packages`**.
5. Nhấn **Generate token** và sao chép mã Token hiển thị.

### Bước 2: Cấu hình Repository Secret
1. Truy cập vào Repository: `AnHaiDung/Dev_SS11_B4`.
2. Vào **Settings** -> Chọn **Secrets and variables** -> **Actions**.
3. Tại phần **Repository secrets**, chọn **New repository secret**.
4. Nhập thông tin:
   - **Name**: `REGISTRY_TOKEN`
   - **Secret**: Dán Personal Access Token vừa tạo ở Bước 1.
5. Nhấn **Add secret**.

### Bước 3: Đẩy mã nguồn lên GitHub & Kiểm tra Pipeline
1. Thực hiện commit và push mã nguồn lên nhánh `main` của repository `Dev_SS11_B4`.
2. Truy cập tab **Actions** trên GitHub repository, chọn workflow đang chạy.
3. Mở xem chi tiết log của bước **"Đăng nhập vào GitHub Container Registry"** và kiểm tra kết quả:
   - Thông báo hiển thị: `Login Succeeded`.

---

## 4. Minh chứng kết quả (Screenshots)

### 4.1. Minh chứng cấu hình Repository Secrets (`REGISTRY_TOKEN`)
*(Đính kèm ảnh chụp màn hình mục Settings -> Secrets and variables -> Actions hiển thị biến `REGISTRY_TOKEN`)*

![Repository Secret REGISTRY_TOKEN](secret.png)

### 4.2. Minh chứng log thực thi bước Đăng nhập Registry thành công
*(Log chi tiết hiển thị kết quả từ GitHub Actions runner)*

```text
Run docker/login-action@v3
  with:
    registry: ghcr.io
    username: AnHaiDung
    password: ***
    logout: true
Logging into ghcr.io...
Login Succeeded!
```
