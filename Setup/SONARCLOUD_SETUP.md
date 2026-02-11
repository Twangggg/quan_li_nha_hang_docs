# Hướng Dẫn Cấu Hình SonarCloud

## Bước 1: Tạo Tài Khoản SonarCloud

1. Truy cập [sonarcloud.io](https://sonarcloud.io)
2. Click **"Log in"** và chọn **"With GitHub"**
3. Authorize SonarCloud để truy cập GitHub account của bạn

## Bước 2: Import Repository

1. Sau khi đăng nhập, click **"+"** ở góc trên bên phải
2. Chọn **"Analyze new project"**
3. Chọn organization (hoặc tạo mới nếu chưa có)
4. Chọn repository **FoodHub_BE** từ danh sách
5. Click **"Set Up"**

## Bước 3: Lấy Thông Tin Cần Thiết

### 3.1. Lấy Organization Key

- Sau khi import, bạn sẽ thấy **Organization Key** trên dashboard
- Ví dụ: `your-username` hoặc `your-org-name`
- **Lưu lại giá trị này**

### 3.2. Tạo Token

1. Click vào avatar ở góc trên bên phải
2. Chọn **"My Account"** → **"Security"** tab
3. Trong phần **"Generate Tokens"**:
   - **Name**: `FoodHub_BE_CI`
   - **Type**: chọn **"Global Analysis Token"**
   - **Expires in**: chọn thời gian (khuyến nghị: 90 days hoặc No expiration)
4. Click **"Generate"**
5. **Copy token ngay** (bạn sẽ không thể xem lại sau này)

## Bước 4: Cập Nhật Project Key

1. Mở file `sonar-project.properties`
2. Thay đổi dòng đầu tiên:

   ```properties
   sonar.projectKey=foodhub-be
   ```

   Thành project key mà SonarCloud đã tạo cho bạn (thường là `organization-key_repository-name`)

3. Cập nhật organization:
   ```properties
   sonar.organization=your-organization-key
   ```
   Thay `your-organization-key` bằng organization key bạn đã lưu ở bước 3.1

## Bước 5: Thêm Secrets vào GitHub

1. Truy cập repository trên GitHub
2. Vào **Settings** → **Secrets and variables** → **Actions**
3. Click **"New repository secret"**

### Thêm SONAR_TOKEN:

- **Name**: `SONAR_TOKEN`
- **Secret**: paste token bạn đã copy ở bước 3.2
- Click **"Add secret"**

### Thêm SONAR_ORGANIZATION:

- **Name**: `SONAR_ORGANIZATION`
- **Secret**: paste organization key từ bước 3.1
- Click **"Add secret"**

## Bước 6: Test CI Pipeline

1. Commit và push các thay đổi:

   ```bash
   git add .
   git commit -m "feat: integrate SonarCloud analysis"
   git push origin feat/devops
   ```

2. Tạo Pull Request hoặc push lên branch `main`/`develop`

3. Kiểm tra GitHub Actions:
   - Vào tab **Actions** trên GitHub
   - Xem workflow **"CI Pipeline"** đang chạy
   - Đợi đến bước **"SonarCloud Analysis"**

4. Kiểm tra kết quả trên SonarCloud:
   - Truy cập [sonarcloud.io](https://sonarcloud.io)
   - Vào project **FoodHub_BE**
   - Xem các metrics: Bugs, Vulnerabilities, Code Smells, Coverage, Duplications

## Xử Lý Lỗi Thường Gặp

### Lỗi: "Project key already exists"

- Đảm bảo `sonar.projectKey` trong `sonar-project.properties` khớp với project key trên SonarCloud

### Lỗi: "Invalid authentication token"

- Kiểm tra lại `SONAR_TOKEN` trong GitHub Secrets
- Tạo token mới nếu cần

### Lỗi: "Organization not found"

- Kiểm tra lại `SONAR_ORGANIZATION` trong GitHub Secrets
- Đảm bảo organization key đúng (không có khoảng trắng)

## Kết Quả Mong Đợi

Sau khi cấu hình thành công, mỗi lần push code:

- ✅ CI pipeline sẽ tự động chạy SonarCloud analysis
- ✅ Kết quả phân tích sẽ hiển thị trên SonarCloud dashboard
- ✅ Quality Gate status sẽ xuất hiện trong PR checks
- ✅ Code coverage sẽ được track và hiển thị

## Quality Gate

SonarCloud sẽ tự động áp dụng Quality Gate mặc định:

- **Coverage**: ≥ 80%
- **Duplications**: ≤ 3%
- **Maintainability Rating**: A
- **Reliability Rating**: A
- **Security Rating**: A

Bạn có thể tùy chỉnh Quality Gate trong SonarCloud settings.
