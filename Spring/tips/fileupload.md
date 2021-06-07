### 클라이언트가 보내는 파일+데이터 request 받기
- Request를 처리하는 Dto를 만든다.
- 파일은 Spring의 MultipartFile로 받을 수 있다.
```
public static class CreateReview {
        private String content;
        private Double score;
        List<MultipartFile> files;
}
```
- controller에서 만든 dto class로 request를 받을 수 있다.

### S3 파일업로드
- AWS S3 버킷을 만들고 IAM을 만들고 엑세스 키와 시크릿 엑세스 키를 받는다.
- 인텔리제이 환경변수나 시스템환경변수에 엑세스 키와 시크릿키를 등록한다.
    - 시스템 환경변수 설정
        - AWS_ACCESS_KEY_ID 
        - AWS_SECRET_ACCESS_KEY

- S3Client를 사용하기 위해 build.gradle 의존성 추가
    ```
        implementation platform('software.amazon.awssdk:bom:2.13.33')
        implementation 'software.amazon.awssdk:s3'
    ```

- S3Client를 DI를 받기 위해서 Bean 등록 
    ```
    @Configuration
    public class AWSS3Config {
        @Bean
        public S3Client getS3Client() {
            return S3Client.builder()
                    .region(Region.AP_NORTHEAST_2)
                    .credentialsProvider(EnvironmentVariableCredentialsProvider.create())
                    .build();
        }
    }
    ```

- 버킷이름과 S3 url 설정
    - application.yml
        ```
        storage:
        s3Bucket: 버킷이름
        s3Public: 퍼블릭주소
        ```
    - yml 설정 값을 사용하기 위한 class 생성
        ```
        @Setter
        @Getter
        @Configuration
        @ConfigurationProperties("storage")
        public class StorageProps {
            String s3Bucket;
            String s3Public;
        }
        ```

- 파일업로드 class 구현
    - S3Client를 DI 받아서 사용한다.
    - s3FileName은 S3에 저장될 이름인데 랜덤으로 생성하였다.
    - publicUrl은 S3에 저장된 객체 url로 파일을 다운로드받을 수 있고 보여줄 수 있다.
    - s3client.putObject 부분에서 S3에 업로드를 한다.
    ```
    @Service
    @RequiredArgsConstructor
    public class FileS3Uploader {
        private final S3Client s3client;
        private final StorageProps storageProps;

        public List<SavedFile> uploadFileList(List<MultipartFile> files) {
            return files.stream().map(this::upload).collect(Collectors.toList());
        }

        private SavedFile upload(MultipartFile file) {
            String originalName = file.getOriginalFilename();
            String extension = Optional.ofNullable(originalName)
                    .filter(s -> s.contains("."))
                    .map(s -> s.substring(originalName.lastIndexOf(".") + 1))
                    .orElse(null);

            String s3FileName = Optional.ofNullable(RandomStringBuilder.generateAlphaNumeric(60)).orElseThrow() + "." + extension;
            String publicUrl = storageProps.getS3Public() + "/" + s3FileName;

            boolean isImage = this.isImage(extension);
            Integer width = null;
            Integer height = null;
            try {
                PutObjectResponse response = s3client.putObject(
                        PutObjectRequest.builder()
                                .key(s3FileName)
                                .bucket(storageProps.getS3Bucket())
                                .build(),
                        RequestBody.fromBytes(file.getBytes()));
                if (isImage) {
                    BufferedImage image = ImageIO.read(file.getInputStream());
                    width = image.getWidth();
                    height = image.getHeight();
                }
            } catch (IOException e) {
                throw new S3FileUploadException();
            }
            SavedFile.SavedFileBuilder savedFileBuilder = SavedFile.builder()
                    .name(s3FileName)
                    .extension(extension)
                    .fileServer(FileServer.S3)
                    .originalName(originalName)
                    .size(file.getSize())
                    .isImage(isImage)
                    .publicUrl(publicUrl)
                    .width(width)
                    .height(height);
            if (isImage) {
                return savedFileBuilder.fileType(FileType.IMAGE).build();
            }
            return savedFileBuilder.fileType(FileType.FILE).build();
        }

        private boolean isImage(String extension) {
            return Optional.ofNullable(extension)
                    .map(s -> s.toLowerCase().matches("png|jpeg|jpg|bmp|gif|svg"))
                    .orElse(false);
        }
    }
    ```
### 파일 업로드 테스트
- ClassPathResource로 해당 파일 InputStream을 가져온다.
- MockMultipartFile을 만들어서 fileUpload의 file에 넣어준다.
```
InputStream is1 = new ClassPathResource("mock/images/enjoy.png").getInputStream();
InputStream is2 = new ClassPathResource("mock/images/enjoy2.png").getInputStream();
MockMultipartFile mockFile1 = new MockMultipartFile("file1", "mock_file1.jpg", "image/jpg", is1.readAllBytes());
MockMultipartFile mockFile2 = new MockMultipartFile("file2", "mock_file2.jpg", "image/jpg", is2.readAllBytes());

ResultActions results = mockMvc.perform(
     fileUpload("/route/{id}/review", 1)
            .file(mockFile1)
            .file(mockFile2)
            .param("content", "테스트 리뷰 내용")
            .param("score", "5")
             .contentType(MediaType.MULTIPART_FORM_DATA)
             .characterEncoding("UTF-8")
);
```

### 파일 수정 기능 구현시 주의할 점
- 프론트에서는 File 엔티티를 MultipartFile 형태로 다시 바꿀 수 없다. 따라서 수정할 때 request data로 File을 받아올 수 없기 때문에 파일이 바뀌었는 지 boolean 값을 받아와서 기능을 구현하는 방식으로 할 수 있다.