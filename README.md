# GitHubUploaderService - Hướng dẫn sử dụng

## Giới thiệu
`GitHubUploaderService` là một service trong Spring Boot giúp tải ảnh lên GitHub Repository. Service này sử dụng thư viện `org.kohsuke.github` để tương tác với GitHub API.

## Cấu trúc thư mục
- `service/GitHubUploaderService.java` - Chứa logic xử lý upload ảnh.
- `exception/AppException.java` - Xử lý lỗi khi file không hợp lệ.
- `utils/FileValidationUtil.java` - Hỗ trợ kiểm tra định dạng file ảnh.

## Cấu hình
Trước khi sử dụng, cần cập nhật `GITHUB_TOKEN`, `REPO_NAME` và `BRANCH` trong `GitHubUploaderService.java`:
```java
private static final String GITHUB_TOKEN = "YOUR_GITHUB_TOKEN";
private static final String REPO_NAME = "your-username/your-repository";
private static final String BRANCH = "main";
```

## Hướng dẫn sử dụng
### 1. Kiểm tra định dạng ảnh hợp lệ
Service chỉ chấp nhận các định dạng ảnh: JPG, PNG, JPEG, GIF. Nếu không đúng định dạng, sẽ ném lỗi:
```java
throw new AppException("Chỉ chấp nhận file JPG, PNG, JPEG, GIF!");
```

### 2. Upload ảnh lên GitHub
Service sẽ kiểm tra xem ảnh đã tồn tại chưa:
- Nếu có, ảnh sẽ được cập nhật với commit message: `Cập nhật ảnh: filename`
- Nếu không, ảnh sẽ được tạo mới với commit message: `Tải ảnh mới: filename`

```java
repository.createContent()
        .content(file.getBytes())
        .path(imagePath)
        .message("Tải ảnh mới: " + file.getOriginalFilename())
        .branch(BRANCH)
        .commit();
```

### 3. Trả về đường dẫn ảnh
Sau khi upload thành công, đường dẫn ảnh sẽ có định dạng:
```
https://raw.githubusercontent.com/{REPO_NAME}/{BRANCH}/images/products/{filename}
```
Ví dụ:
```
https://raw.githubusercontent.com/NgSao/image_springboot/main/images/products/sample.jpg
```

## Ví dụ sử dụng
Dưới đây là ví dụ về cách sử dụng `GitHubUploaderService` trong một controller của Spring Boot:

```java
@RestController
@RequestMapping("/api/upload")
public class UploadController {
    private final GitHubUploaderService gitHubUploaderService;

    public UploadController(GitHubUploaderService gitHubUploaderService) {
        this.gitHubUploaderService = gitHubUploaderService;
    }

    @PostMapping("/image")
    public ResponseEntity<String> uploadImage(@RequestParam("file") MultipartFile file) {
        try {
            String imageUrl = gitHubUploaderService.uploadImage(file);
            return ResponseEntity.ok(imageUrl);
        } catch (IOException e) {
            return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR).body("Lỗi khi tải ảnh lên GitHub");
        }
    }
}
```

## Service
Dưới đây là mã nguồn của `GitHubUploaderService`:
```java
@Service
public class GitHubUploaderService {
    private static final String GITHUB_TOKEN = "YOUR_GITHUB_TOKEN";
    private static final String REPO_NAME = "your-username/your-repository";
    private static final String BRANCH = "main";

    public String uploadImage(MultipartFile file) throws IOException {
        if (!FileValidationUtil.isValidImage(file)) {
            throw new AppException("Chỉ chấp nhận file JPG, PNG, JPEG, GIF!");
        }
        GitHub github = GitHub.connectUsingOAuth(GITHUB_TOKEN);
        GHRepository repository = github.getRepository(REPO_NAME);

        String imagePath = "images/products/" + file.getOriginalFilename();

        try {
            GHContent existingContent = repository.getFileContent(imagePath);
            repository.createContent()
                    .content(file.getBytes())
                    .path(imagePath)
                    .message("Cập nhật ảnh: " + file.getOriginalFilename())
                    .sha(existingContent.getSha())
                    .branch(BRANCH)
                    .commit();
        } catch (Exception e) {
            repository.createContent()
                    .content(file.getBytes())
                    .path(imagePath)
                    .message("Tải ảnh mới: " + file.getOriginalFilename())
                    .branch(BRANCH)
                    .commit();
        }

        return "https://raw.githubusercontent.com/" + REPO_NAME + "/" + BRANCH + "/" + imagePath;
    }
}
```

## Lưu ý bảo mật
**Không commit trực tiếp `GITHUB_TOKEN` vào mã nguồn!** Hãy sử dụng biến môi trường hoặc cấu hình Spring Boot để bảo vệ thông tin đăng nhập.

## Dependencies
Cần thêm thư viện sau vào `pom.xml` để sử dụng GitHub API:
```xml
<dependency>
    <groupId>org.kohsuke</groupId>
    <artifactId>github-api</artifactId>
    <version>1.123</version>
</dependency>
```

## Liên hệ
Nếu gặp vấn đề, hãy liên hệ [NgSao](https://github.com/NgSao).
