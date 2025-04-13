```markdown
## 📦 파일 업로드 & 다운로드 정리

---

### 1. 서블릿과 파일 업로드 1

- `multipart/form-data` 설정 필요
- `HttpServletRequest.getParts()` 사용
- `Part`를 통해 파일 메타데이터와 바이너리 접근 가능

```java
Collection<Part> parts = request.getParts();
for (Part part : parts) {
    log.info("name={}, submittedFileName={}, size={}",
             part.getName(), part.getSubmittedFileName(), part.getSize());
    InputStream inputStream = part.getInputStream();
    part.write("/저장경로/" + part.getSubmittedFileName());
}
```

---

### 2. 서블릿과 파일 업로드 2

- 한글, 공백, 특수문자 파일명 → sanitize 필요
- 디렉토리 없으면 생성 (`mkdirs()`)
- 파일명 중복 방지 → UUID 사용

```java
String originalFilename = part.getSubmittedFileName();
String ext = originalFilename.substring(originalFilename.lastIndexOf("."));
String uuid = UUID.randomUUID().toString();
String storedFileName = uuid + ext;
```

---

### 3. 스프링과 파일 업로드

- `MultipartFile` 사용하여 간편 처리
- `transferTo()` 메서드로 저장

```java
@PostMapping("/upload")
public String upload(@RequestParam MultipartFile file) throws IOException {
    String originalFilename = file.getOriginalFilename();
    String fullPath = fileDir + originalFilename;
    file.transferTo(new File(fullPath));
    return "upload-form";
}
```

- application.properties 설정

```properties
file.dir=/Users/kkwon/file/
```

---

### 4. 예제로 구현하는 파일 업로드, 다운로드

- 업로드 시 DB에 업로드 파일명, 저장 파일명 저장
- 다운로드 시 `Content-Disposition` 헤더로 파일명 지정

```java
@GetMapping("/attach/{id}")
public ResponseEntity<Resource> download(@PathVariable Long id) {
    UploadFile file = fileService.findFile(id);
    Resource resource = new UrlResource("file:" + file.getStoredFilename());

    String encodedName = UriUtils.encode(file.getUploadFilename(), StandardCharsets.UTF_8);
    String contentDisposition = "attachment; filename=\"" + encodedName + "\"";

    return ResponseEntity.ok()
        .header(HttpHeaders.CONTENT_DISPOSITION, contentDisposition)
        .body(resource);
}
```
```
