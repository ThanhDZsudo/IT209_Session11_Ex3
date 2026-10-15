# Báo cáo: Tối ưu tốc độ CI bằng Gradle Dependency Caching (Session 11 - Bài 3)

## 1. Thông tin bài làm
- **Học viên:** Nguyễn Tiến Thành
- **Môn học:** IT211 - Dịch vụ Java / CI-CD Pipeline
- **Bài tập:** Bài 3 - Tối ưu tốc độ CI bằng Gradle Dependency Caching
- **Repository:** [ThanhDZsudo/IT209_Session11_Ex3](https://github.com/ThanhDZsudo/IT209_Session11_Ex3)

---

## 2. Mục tiêu
Bật tính năng Caching cho Gradle Dependencies để:
- Giảm thời gian tải các thư viện / dependencies từ kho chứa Internet (Maven Central) trong mỗi lần luồng CI thực thi.
- Giảm thời gian thực thi của bước `compileJava compileTestJava` và `test`.
- Tối ưu hóa tổng thời gian thực hiện Job trong GitHub Actions pipeline.

---

## 3. Cấu hình kịch bản GitHub Actions (`.github/workflows/ci.yml`)
Trong bước thiết lập môi trường Java, sử dụng `actions/setup-java@v5` kết hợp với tham số `cache: 'gradle'`:

```yaml
name: Java CI with Gradle

on:
  push:
    branches: [ "main", "master" ]
  pull_request:
    branches: [ "main", "master" ]
  workflow_dispatch:

permissions:
  contents: read

jobs:
  ci-pipeline:
    name: Check Syntax, Test & Package JAR
    runs-on: ubuntu-latest

    steps:
      # Bước 1: Checkout mã nguồn
      - name: Checkout source code
        uses: actions/checkout@v4

      # Bước 2: Thiết lập môi trường Java JDK 17 & tự động cache Gradle dependencies
      - name: Thiết lập môi trường Java JDK 17
        uses: actions/setup-java@v5
        with:
          java-version: '17'
          distribution: 'temurin'
          cache: 'gradle'

      # Bước 3: Cấp quyền thực thi cho file gradlew
      - name: Grant execute permission for gradlew
        run: chmod +x gradlew

      # Bước 4: Check cú pháp & Biên dịch mã nguồn
      - name: Check syntax & compile classes
        run: ./gradlew compileJava compileTestJava --info

      # Bước 5: Chạy kiểm thử tự động (Unit Test)
      - name: Run tests
        run: ./gradlew test

      # Bước 6: Đóng gói ứng dụng thành file JAR thực thi
      - name: Package JAR application
        run: ./gradlew bootJar -x test

      # Bước 7: Upload và lưu trữ file JAR làm Artifact
      - name: Upload JAR Artifact
        uses: actions/upload-artifact@v4
        with:
          name: IT211_SS5_Ex2-application-jar
          path: build/libs/*.jar
          retention-days: 7
```

---

## 4. Kết quả thực nghiệm 2 lần Push liên tiếp

*(Dữ liệu chi tiết sẽ được tự động đồng bộ ngay sau khi hoàn thành Run 2)*
