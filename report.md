# Báo Cáo Thực Hành: Tối Ưu Tốc Độ CI Bằng Gradle Dependency Caching

- **Học phần:** IT211 - Dịch vụ Java / CI-CD Pipeline
- **Bài tập:** Bài 3 - Tối ưu tốc độ CI bằng Gradle Dependency Caching
- **Học viên:** Nguyễn Tiến Thành
- **Kho lưu trữ (Repository):** [https://github.com/ThanhDZsudo/IT209_Session11_Ex3](https://github.com/ThanhDZsudo/IT209_Session11_Ex3)

---

## 1. Mục Tiêu Bài Tập
1. Kích hoạt tính năng **Caching** cho Gradle Dependencies trong pipeline GitHub Actions bằng tham số `cache: 'gradle'` ở action `actions/setup-java@v5`.
2. Giảm thiểu việc phải tải lại các thư viện/phụ thuộc từ mạng Internet (Maven Central) ở mỗi lần chạy CI.
3. Đo lường và so sánh sự chênh lệch thời gian thực thi của bước `setup-java`, bước biên dịch/kiểm thử (`./gradlew compileJava compileTestJava` & `./gradlew test`), và toàn bộ quy trình CI giữa 2 lần push liên tiếp (Lần 1 chưa có cache vs Lần 2 đã khôi phục cache).

---

## 2. Kịch Bản CI Đã Tối Ưu (`.github/workflows/ci.yml`)

Nội dung tệp cấu hình `.github/workflows/ci.yml` sử dụng `actions/setup-java@v5` với `cache: 'gradle'`:

```yaml
name: Java CI with Gradle

on:
  push:
    branches: [ "main", "master" ]
  pull_request:
    branches: [ "main", "master" ]
  workflow_dispatch: # Cho phép kích hoạt thủ công từ giao diện GitHub Actions

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

      # Bước 3: Cấp quyền thực thi cho file gradlew (tránh lỗi permission denied trên Linux runner)
      - name: Grant execute permission for gradlew
        run: chmod +x gradlew

      # Bước 4: Check cú pháp & Biên dịch mã nguồn (Syntax & Compilation check)
      - name: Check syntax & compile classes
        run: ./gradlew compileJava compileTestJava --info

      # Bước 5: Chạy kiểm thử tự động (Unit Test)
      - name: Run tests
        run: ./gradlew test

      # Bước 6: Đóng gói ứng dụng thành file JAR thực thi (Package JAR)
      - name: Package JAR application
        run: ./gradlew bootJar -x test

      # Bước 7: Upload và lưu trữ file JAR làm Artifact để tải về
      - name: Upload JAR Artifact
        uses: actions/upload-artifact@v4
        with:
          name: IT211_SS5_Ex2-application-jar
          path: build/libs/*.jar
          retention-days: 7
```

---

## 3. Kết Quả Đo Lường 2 Lần Push Liên Tiếp

### 3.1. Bảng so sánh chi tiết thời gian thực thi

| Bước thực hiện (Step) | Lần 1: Chưa Cache (Run #4 - Commit `9a09285`) | Lần 2: Đã Cache (Run #5 - Commit `29ed6bf`) | Chênh lệch (Hiệu quả tối ưu) |
| :--- | :---: | :---: | :---: |
| **Set up job** | 1s | 1s | Giữ nguyên |
| **Checkout source code** | 1s | 1s | Giữ nguyên |
| **Thiết lập môi trường Java JDK 17** *(setup-java)* | 0s *(chưa có cache để khôi phục)* | 5s *(tải & giải nén cache từ GitHub Storage)* | +5s |
| **Grant execute permission for gradlew** | 1s | 0s | Giữ nguyên |
| **Check syntax & compile classes** *(./gradlew compileJava...)* | **52s** *(phải tải tất cả dependencies từ Maven Central)* | **13s** *(sử dụng dependencies đã cache trong runner)* | **Nhanh hơn 39s (Giảm 75%)** |
| **Run tests** *(./gradlew test)* | **8s** | **7s** | Nhanh hơn 1s |
| **Package JAR application** | 2s | 1s | Nhanh hơn 1s |
| **Upload JAR Artifact** | 4s | 3s | Giữ nguyên |
| **Post Thiết lập môi trường Java JDK 17** | **6s** *(lưu cache lên GitHub Actions Cache)* | **0s** *(cache hit, không cần lưu lại)* | Nhanh hơn 6s |
| **Tổng thời gian toàn bộ Job (Total Duration)** | **75 giây (1m 15s)** | **31 giây (0m 31s)** | **Giảm 44 giây (~58.7%)** |

---

## 4. Kiểm Tra Bằng Chứng Khôi Phục Cache Thành Công

### 4.1. Cache Key Được Tạo và Lưu Trữ
Khi bước `setup-java@v5` chạy lần đầu, nó tính toán hash dựa trên cấu hình build (`build.gradle`, `gradle-wrapper.properties`) và tạo các entry cache:
- `setup-java-Linux-x64-gradle-3abb4d7db50a36d19e6318dcdff58cff5c06d0af4d16acf89a80fcbbc30b28f1`
- `setup-java-Linux-x64-gradle-wrapper-de6cdbcbd1e1f8f25cd2f4c951f2672b46708f2ee46299399a257dc866cd90a6`

### 4.2. Log Khôi Phục Cache ở Lần Push Thứ 2
Ở lần push thứ 2 (Run #5), bước `Thiết lập môi trường Java JDK 17` phát hiện cache key trùng khớp và khôi phục thành công:
```text
Received 145982123 of 145982123 (100.0%), 72.1 MBs/sec
Cache restored successfully
Cache restored successfully from key: setup-java-Linux-x64-gradle-3abb4d7db50a36d19e6318dcdff58cff5c06d0af4d16acf89a80fcbbc30b28f1
```

---

## 5. Đánh Giá và Kết Luận

1. **Hiệu suất tăng vượt bậc:** Tổng thời gian chạy luồng CI giảm từ **75 giây** xuống chỉ còn **31 giây** (tiết kiệm **58.7%** thời gian). Đặc biệt bước biên dịch giảm từ **52 giây** xuống còn **13 giây** (nhanh gấp **4 lần**).
2. **Tiết kiệm tài nguyên và băng thông:** Các thư viện bên ngoài không còn bị tải lặp đi lặp lại qua mạng Internet, giúp giảm tải đường truyền và tránh rủi ro pipeline bị nghẽn mạng hay lỗi timeout từ kho lưu trữ trung gian.
3. **Phản hồi nhanh (Fast Feedback Loop):** Quá trình kiểm thử và tích hợp liên tục (CI) được đẩy nhanh tốc độ đáng kể, giúp nhóm phát triển phát hiện lỗi sớm và tự tin release sản phẩm.
