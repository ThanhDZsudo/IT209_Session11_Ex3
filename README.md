# IT211_SS5_Ex2 - Spring Boot REST API & CI/CD Demo

Dự án mẫu Spring Boot REST API tích hợp quy trình CI/CD tự động bằng GitHub Actions:
- **Ngôn ngữ:** Java 17
- **Framework:** Spring Boot 4
- **Build tool:** Gradle Wrapper
- **Database:** MySQL (Production) / H2 (Testing)

## Pipeline CI/CD trên GitHub Actions
Pipeline được kích hoạt tự động khi `push` hoặc `pull_request` vào nhánh `main` / `master`, hoặc chạy thủ công qua `workflow_dispatch`.

Quy trình gồm các bước:
1. **Checkout Source Code**
2. **Set up JDK 17** (Eclipse Temurin)
3. **Setup Gradle & Cache dependencies**
4. **Grant execute permission for gradlew** (`chmod +x gradlew`)
5. **Check cú pháp & Compile:** `./gradlew compileJava compileTestJava`
6. **Chạy Unit Test:** `./gradlew test`
7. **Đóng gói file JAR:** `./gradlew bootJar -x test`
8. **Upload Artifact:** Đưa file `.jar` lên GitHub Actions Artifacts để tải về.
