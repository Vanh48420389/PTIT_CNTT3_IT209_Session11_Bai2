# BÀI TẬP 2: BIÊN DỊCH ĐÓNG GÓI TỆP ỨNG DỤNG .JAR VỚI GRADLE VÀ LƯU TRỮ ARTIFACTS

**Học phần:** Điện toán đám mây / CI-CD & DevOps (IT209)  
**Session:** 11 - CI/CD Pipeline với GitHub Actions  
**Thư mục bài làm:** `Lession11/Bai02`  
**GitHub Submission Path:** `homework/session_11/ex2/`

---

## 1. Mục tiêu bài thực hành
- Mở rộng quy trình CI (Continuous Integration) từ Bài 1 để tự động đóng gói ứng dụng Spring Boot.
- Thực thi tác vụ đóng gói thành tệp thực thi `.jar` độc lập (fat JAR/uber JAR) bằng Gradle: `./gradlew bootJar`.
- Lưu trữ thành phẩm biên dịch (`build/libs/*.jar`) lên hệ thống lưu trữ của GitHub Actions bằng action chính thức `actions/upload-artifact@v4`.
- Đặt tên định danh cho Artifact là `app-jar`, cho phép xem và tải tệp về tại phần **Summary** của Workflow Run.

---

## 2. Cấu trúc thư mục dự án

```text
Bai02/
├── .github/
│   └── workflows/
│       └── ci.yml               # Workflow kịch bản CI tích hợp đóng gói & lưu Artifact
├── gradle/
│   └── wrapper/                 # Gradle Wrapper
├── src/
│   ├── main/java/               # Mã nguồn Spring Boot
│   └── test/java/               # Các bài Unit Test JUnit 5
├── build.gradle                 # Cấu hình Gradle build & toolchain Java 17
├── settings.gradle              # Metadata cấu hình dự án
├── gradlew                      # Gradle wrapper script cho Linux / macOS
├── gradlew.bat                  # Gradle wrapper script cho Windows
└── README.md                    # Hướng dẫn chi tiết & báo cáo kiểm thử
```

---

## 3. Nội dung tệp kịch bản `.github/workflows/ci.yml`

```yaml
name: Java Spring Boot CI (Gradle)

on:
  push:
    branches: [ "main" ]

jobs:
  test:
    name: Run Unit Tests
    runs-on: ubuntu-latest

    steps:
      - name: Checkout mã nguồn
        uses: actions/checkout@v5

      - name: Thiết lập môi trường Java JDK 17
        uses: actions/setup-java@v5
        with:
          java-version: '17'
          distribution: 'temurin'

      - name: Cấp quyền cho Gradle Wrapper
        run: chmod +x gradlew

      - name: Thực thi JUnit Test với Gradle
        run: ./gradlew test

      - name: Đóng gói tệp JAR với Gradle
        run: ./gradlew bootJar

      - name: Lưu trữ thành phẩm JAR Artifact
        uses: actions/upload-artifact@v4
        with:
          name: app-jar
          path: build/libs/*.jar
```

---

## 4. Phân tích kỹ thuật các bước bổ sung

### 4.1. Bước: Đóng gói tệp JAR với Gradle (`bootJar`)
```yaml
- name: Đóng gói tệp JAR với Gradle
  run: ./gradlew bootJar
```
- **Ý nghĩa:** Task `bootJar` do plugin `org.springframework.boot` cung cấp.
- **Cơ chế:** Biên dịch toàn bộ mã nguồn `src/main/java`, đóng gói tất cả các thư viện phụ thuộc (embedded Tomcat, Spring framework,...) thành một tệp duy nhất **Executable JAR** nằm tại thư mục `build/libs/`.
- Tệp `.jar` này có thể chạy độc lập ở bất kỳ môi trường nào có Java Runtime Environment (JRE) bằng lệnh: `java -jar <filename>.jar`.

### 4.2. Bước: Lưu trữ thành phẩm JAR Artifact (`actions/upload-artifact@v4`)
```yaml
- name: Lưu trữ thành phẩm JAR Artifact
  uses: actions/upload-artifact@v4
  with:
    name: app-jar
    path: build/libs/*.jar
```
- **Ý nghĩa:** GitHub Actions runner là môi trường tạm thời (ephemeral runner). Khi workflow kết thúc, toàn bộ dữ liệu trong runner sẽ bị xóa hoàn toàn. Action `actions/upload-artifact@v4` sẽ upload tệp JAR lên kho lưu trữ dài hạn của GitHub.
- **`name: app-jar`**: Tên hiển thị của Artifact trên giao diện GitHub. Khi người dùng tải về, GitHub sẽ đóng gói thành file `app-jar.zip`.
- **`path: build/libs/*.jar`**: Mẫu đường dẫn tìm kiếm tệp JAR đầu ra cần lưu trữ.

---

## 5. Hướng dẫn kiểm tra và nộp bài

### Bước 1: Khởi tạo Git & Đẩy commit lên nhánh `main`
```bash
git init
git add .
git commit -m "feat: add bootJar packaging and upload-artifact to CI workflow"
git branch -M main
git remote add origin <URL_REPOSITORY_GITHUB_CUA_BAN>
git push -u origin main
```

### Bước 2: Kiểm tra kết quả trên GitHub Actions
1. Mở Repository trên GitHub, chuyển sang tab **Actions**.
2. Chọn lần chạy workflow mới nhất kích hoạt bởi sự kiện push vào nhánh `main`.
3. Kiểm tra:
   - Job **Run Unit Tests** có dấu tích xanh (**Success**).
   - Tất cả các bước gồm `Checkout`, `Setup Java`, `gradlew test`, `gradlew bootJar`, và `Upload Artifact` đều hoàn thành không lỗi.

### Bước 3: Xác nhận Artifacts trong phần Summary
1. Nhấn vào mục **Summary** ở cột bên trái của workflow run.
2. Cuộn xuống phần **Artifacts** ở cuối trang.
3. Xác nhận xuất hiện tệp có tên **`app-jar`** với dung lượng cụ thể (khoảng ~30 - 45 MB) và có thể nhấn vào để tải về máy tính.

### Bước 4: Minh chứng nộp bài
- Chụp ảnh màn hình toàn bộ trang **Summary** của lần chạy hiển thị:
  - Tên Workflow `Java Spring Boot CI (Gradle)`
  - Trạng thái `Success` (tích xanh)
  - Khối **Artifacts** có tệp `app-jar`
- Nộp tệp `.github/workflows/ci.yml` và ảnh chụp màn hình lên `homework/session_11/ex2/`.

---

## 6. Kết quả thực thi đóng gói JAR thực tế (Local Verification)

```bash
./gradlew bootJar
```
- **Trạng thái:** `BUILD SUCCESSFUL`
- **Tệp xuất xưởng:** `build/libs/student-0.0.1-SNAPSHOT.jar` (~48.8 MB)
- **Tương thích Artifacts:** Khớp chính xác với đường dẫn `build/libs/*.jar` cấu hình trong GitHub Actions.

