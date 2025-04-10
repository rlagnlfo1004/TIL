## **🔍 Overview**

파일과 같은 데이터 업로드는 웹/앱에서 가장 많이 다루는 기능 중 하나이다. 사진, 동영상, 문서와 같은 미디어 파일을 업로드하는 프로세스는 주로 아래와 같다.



![출처 : https://mangkyu.tistory.com/18](https://velog.velcdn.com/images/jmjmjmz732002/post/f2e57165-6fae-429e-842e-0ae973ca6781/image.png)

출처 : https://mangkyu.tistory.com/18

1. 사용자가 파일을 어플리케이션 서버에 업로드한다.

2. 어플리케이션 서버는 처리를 위해 업로드를 임시 공간에 저장한다.

3. 파일을 데이터베이스, 파일 서버 또는 영구 저장을 위한 개체 저장소로 전송한다.
<br></br>

보통 파일 업로드는 **권한 설정** 때문에 서버를 경유해야 한다. 영상 파일 같이 컨텐츠의 용량이 높은 경우, 서버에 넘겨준 후 **서버에서 스토리지에 저장하는 이중 작업은 비효율적**일 때가 있다. 네트워크 I/O 및 서버 CPU 사용량이 커지며 속도 지연을 일으킬 수도 있다는 것이다. 파일 크기가 크지 않더라도 서버에서 Multipart file을 받아 S3 버킷에 업로드하면 서버쪽에서 파일을 갖고 있어야 하는 자체로 리소스 낭비가 발생할 수 있다.

그렇다면 클라이언트에서 바로 업로드하면 되지 왜 굳이 서버를 거쳐서 업로드하는 걸까? **보안** 때문이다. 정해둔 규칙 안에서 데이터가 관리되어야 하기 때문에 **권한을 가진 사용자만 S3에 접근해야 한다**. 이를 서버가 수행해주며 일종의 보안 절차 작업을 거치게 되는 것. 위와 같은 단점을 개선하여 서버단의 리소스를 사용하지 않고 클라이언트가 S3에 직접 파일을 업로드할 수 있는 + 보안 절차까지 보장되는 **Presigned URL** 방식을 알아보자.
<br></br><br></br>
## 🔍 Presigned URL 이란?

먼저, Presigned URL이란 직역시 '미리 서명된 URL'이라는 뜻이다. Presigned URL을 사용하면 서버는 URL만 발급해주고, 파일 업로드 관리는 AWS 측에서 진행한다. 서버에서 권한을 검증하여 나온 Presigned URL으로,  해당 URL은 **이미 S3에 지급할 수 있는 권한을 가진 상태**를 의미하기에, 서버에 거쳐 권한 검증을 할 필요가 없다. S3의 Bucket Policy나 acl과 같은 권한 설정과 관계없이 특정 단말(채널)에 특정 유효 기간동안 S3에 PUT, GET이 가능하게 하는 URL이다. 유효 기간 이후에 사라지기 때문에(Access Denied) 외부 유출에도 큰 위험이 없다.
<br></br><br></br>
## 🔍 Presigned URL 을 사용하는 이유

백엔드 서버를 구축할 때, 파일 업로드 구현을 서버가 S3 버킷에 직접 업로드하는 방식으로 하는 경우에는 서버에서 이미지 파일을 직접 전송받고 s3에 전송해 줘야 하기에, 이 과정에서 서버 자원이 많이 소모된다. 하지만 AWS S3에서 제공하는 Presigned URL을 사용한다면 **서버에서 파일을 직접 처리하지 않게** 되기에 서버의 부담이 많이 줄어드는 큰 장점이 있다. 즉, 서버의 네트워크 I/O 비용을 줄이고, 리소스를 절약할 수 있다. 그리고 후술하는 URL 파라미터에서 나오는 것처럼 특정 유효 기간동안만 S3 접근 권한을 지급할 수 있기 때문에 무분별한 S3 접근을 막아 보안을 강화할 수 있다. 또한, AWS에서 Presigned URL을 통해 파일을 업로드하는 경우 처리 속도를 아주 빠르게 제공해 주고 있어서 기존 프로젝트보다 성능이 향상된다.
<br></br><br></br>
## ⚡️ Presigned URL 동작방식


[![출처 : https://ipekogosu.tistory.com/59]([[attachment:13dc636d-6182-4ef1-b135-8d398436dfd8:image.png](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbNW3bx%2FbtsJbsyARHf%2FdaPorrahKn7KtT9JktswlK%2Fimg.png)](https://www.notion.so/image/attachment%3A13dc636d-6182-4ef1-b135-8d398436dfd8%3Aimage.png?table=block&id=1d1a76ff-c9a1-80b8-918c-cdc6fc31552a&spaceId=37875086-0463-4536-bf7b-76a3045c93e9&width=1510&userId=86276022-5496-4e5f-ba0e-6d27cf0a31ca&cache=v2))](https://www.notion.so/image/attachment%3A13dc636d-6182-4ef1-b135-8d398436dfd8%3Aimage.png?table=block&id=1d1a76ff-c9a1-80b8-918c-cdc6fc31552a&spaceId=37875086-0463-4536-bf7b-76a3045c93e9&width=1510&userId=86276022-5496-4e5f-ba0e-6d27cf0a31ca&cache=v2)

출처 : https://ipekogosu.tistory.com/59

Presigned URL을 사용하는 로직은 다음과 같다.

1. client -> server : presigned url을 달라고 요청
2. server -> s3 bucket : presigned url을 새로 생성하도록 요청
3. s3 bucket -> server : url을 생성해서 전달
4. server -> client : 받은 url을 전달
5. client -> s3 bucket : url에 직접 파일을 업로드하면 bucket에 업로드
<br></br>

이 flow를 보면, **Presigned URL이란 결국 S3 bucket으로 접근을 할 수 있게 해 주는 endpoint**라는 것을 알 수 있다. 보통 s3 버킷에는 함부로 접근을 하지 못하게 막아야 보안적으로 바람직하지만, presigned url은 제한된 조건 하에서 AWS credentials이 없어도 접근이 가능하도록 통로를 열어주는 것이다. 여기서 가장 큰 핵심은 **클라이언트에서 직접 S3에 파일을 보내**게 된다는 것이다.
<br></br>
각각의 단계에서의 세부적인 내용을 예시를 통해 알아보자

### 1. 클라이언트 → 백엔드: "파일 올릴래!”

클라이언트가 파일을 올리기 위한 URL을 발급 요청한다.

```
POST /api/files/presigned-urls
Content-Type: application/json

{
  "fileNames": ["image1.jpg", "report.pdf", "profile.png"]
}
```
<br></br>
### 2. AWS S3 → 백엔드: S3 업로드용 URL 발급

AWS는 파일을 올릴 때 사용할 수 있는 URL을 발급해준다. 이때, 해당 URL을 Presigned URL이라고 한다. 뒤에서 다뤄볼 URL 파라미터에서 나오는 것처럼 특정 유효 기간동안만 S3 접근 권한을 지급할 수 있기 때문에 무분별한 S3 접근을 막아 보안을 강화할 수 있다. 즉, 발급된 해당 URL을 통해서는 15분 뒤에 업로드 불가하다.

```json
{
  "preSignedUrls": [
    "https://mybucket.s3.ap-northeast-2.amazonaws.com/uploads/image1.jpg?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20250410T100000Z&X-Amz-SignedHeaders=host&X-Amz-Credential=AKIAIOSFODNN7EXAMPLE%2F20250410%2Fap-northeast-2%2Fs3%2Faws4_request&X-Amz-Expires=600&X-Amz-Signature=abcdef1234567890abcdef1234567890abcdef1234567890abcdef1234567890",
    
    "https://mybucket.s3.ap-northeast-2.amazonaws.com/docs/report.pdf?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20250410T101500Z&X-Amz-SignedHeaders=host&X-Amz-Credential=AKIAIOSFODNN7EXAMPLE%2F20250410%2Fap-northeast-2%2Fs3%2Faws4_request&X-Amz-Expires=900&X-Amz-Signature=1234abcd5678efgh9012ijkl3456mnop7890qrst1234uvwx5678yzab9012cdef",

    "https://mybucket.s3.ap-northeast-2.amazonaws.com/user/profile.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20250410T102500Z&X-Amz-SignedHeaders=host&X-Amz-Credential=AKIAIOSFODNN7EXAMPLE%2F20250410%2Fap-northeast-2%2Fs3%2Faws4_request&X-Amz-Expires=300&X-Amz-Signature=ffeeddccbbaa99887766554433221100aabbccddeeff00112233445566778899"
  ]
}
```
<br></br>
### 3. 클라이언트 → AWS S3 : 파일 업로드

클라이언트는 각 파일을 해당 Presigned URL에 PUT 요청으로 파일을 보내어 업로드 한다. 위의 각각의 주소에 Http 메서드는 PUT이며, Header에 content-type만 파일 타입으로 지정하고, Body에 binary type으로 파일 업로드하면 된다.

```
PUT https://mybucket.s3.ap-northeast-2.amazonaws.com/uploads/image1.jpg?...
Content-Type: image/jpeg

[image1.jpg 파일의 바이너리 데이터]
```

```
PUT https://mybucket.s3.ap-northeast-2.amazonaws.com/docs/report.pdf?...
Content-Type: application/pdf

[report.pdf 파일의 바이너리 데이터]
```

```
PUT https://mybucket.s3.ap-northeast-2.amazonaws.com/user/profile.png?...
Content-Type: image/png

[profile.png 파일의 바이너리 데이터]
```
<br></br>
## 🔍 Presigned URL 파라미터

S3의 **Pre-signed URL**에는 여러 개의 **쿼리 파라미터**가 포함돼 있는데, 이들은 S3에 요청을 보낼 때 AWS 인증과 관련된 정보를 포함해줘야 하기 때문에 붙는 것들이다. 아래의 예시를 통해 자세히 알아보자.

```
https://example-bucket.s3.amazonaws.com/images/2d098b12-5cd7-4f00-835b-a6c998a13617%EB%B8%94%EB%A1%9C%EA%B7%B8%20%EC%8D%B8%EB%84%A4%EC%9D%BC.png
?x-amz-acl=public-read
&X-Amz-Algorithm=AWS4-HMAC-SHA256
&X-Amz-Date=20230903T144326Z
&X-Amz-SignedHeaders=host
&X-Amz-Expires=120
&X-Amz-Credential=<ACCESS_KEY>/20230903/ap-northeast-2/s3/aws4_request
&X-Amz-Signature=<SIGNATURE_VALUE>
```

**x-amz-acl**

Presigned URL을 활용하여 업로드하고 리소스에 대한 GET 요청 시 access denied가 발생한다.

그러므로 Presigned URL을 생성할 때 public-read 권한을 설정해야 한다.

**x-amz-algorithm**

서명 버전과 알고리즘을 식별하고, 서명을 계산하는데 사용한다. 버전 4를 위해서 AWSS4-HMAC-SHA256

**x-amz-date**

날짜는 ISO 8601 형식을 사용한다.

ex) 20240118T000000Z

**x-amz-signedheaders**

서명을 계산하기 위해 사용되는 헤더 목록으로, HTTP host 헤더가 요구된다.

**x-amz-expires**

미리 선언된 URL이 유효한 시간 주기로, 초 단위이며 정수 값이다.

최소 1에서 최대 604800(7일)까지 가능하다.

**x-amz-credential**

Acces Key와 범위 정보(요청 날짜, 리전, 서비스 명)

**x-amz-signature**

요청을 인증하기 위한 서명
<br></br>
## ⚡️ Spring Boot 구현

### 1. AmazonS3Config 파일을 만들어서 Bean으로 S3Presigner를 등록

S3 접근을 위해 필요한 인증 정보를 설정해준다. 해당 예시에서는 백엔드에서 직접 파일 업로드 & 다운로드를 하기 위한 S3Client을 추가로 만들었다.

```java
@Configuration
public class AmazonS3Config {

    @Value("${cloud.aws.credentials.access-key}")
    private String accessKey;

    @Value("${cloud.aws.credentials.secret-key}")
    private String secretKey;

    @Value("${cloud.aws.region.static}")
    private String region;

    // 백엔드에서 직접 파일 업로드 & 다운로드
    @Bean
    public S3Client s3Client() {
        return S3Client.builder()
                .region(Region.of(region))
                .credentialsProvider(StaticCredentialsProvider.create(
                        AwsBasicCredentials.create(accessKey, secretKey)))
                .build();
    }

    // Pre-signed URL 생성 (프론트에서 업로드 가능)
    @Bean
    public S3Presigner s3Presigner() {
        return S3Presigner.builder()
                .region(Region.of(region))
                .credentialsProvider(StaticCredentialsProvider.create(
                        AwsBasicCredentials.create(accessKey, secretKey)))
                .build();
    }
}
```
<br></br>
### 2. Presigned URL을 생성

**파일 메타데이터를 저장**

사실 이부분이 핵심이라 생각한다. AWS S3는 파일을 **폴더 없이 Key 값**으로 관리하기 때문에, 업로드한 **파일의 S3 경로를 데이터베이스에 저장**해야 검색 및 조회가 가능하다. 즉 파일이 어디에 저장되었는지 추적 가능하다는 것이다. 해당 서비스는 **Post, Notice 등 데이터와 연결 가능** 파일이 어떤 게시물(게시글, 공지사항)에 속하는지 식별 가능하도록 하기 위한 구현이다. Pre-signed URL은 일시적이므로, 실제 **S3 저장 위치를 유지할 필요**가 있으며, 파일 삭제시에도 데이터베이스에 기록된 경로를 통해 S3에서 삭제 요청을 보낼 수 있다.


[![image.png]([attachment:25c80a82-934f-44de-9dcf-4ae6153be745:image.png](https://www.notion.so/image/attachment%3Aaa144a10-b425-4227-8e30-866332684eca%3Aimage.png?table=block&id=1bba76ff-c9a1-802e-8865-d76548d02a04&spaceId=37875086-0463-4536-bf7b-76a3045c93e9&width=2000&userId=86276022-5496-4e5f-ba0e-6d27cf0a31ca&cache=v2))](https://www.notion.so/image/attachment%3A25c80a82-934f-44de-9dcf-4ae6153be745%3Aimage.png?table=block&id=1d1a76ff-c9a1-80d1-b8f2-f6920106bf72&spaceId=37875086-0463-4536-bf7b-76a3045c93e9&width=1510&userId=86276022-5496-4e5f-ba0e-6d27cf0a31ca&cache=v2)

S3는 폴더 개념이 없고, 객체 키(Object Key) 기반으로 파일을 관리한다. 하지만, 논리적인 폴더 구조를 만들어서 관리하는 것이 일반적이다. 따라서, Post와 Notice는 구분하여 저장하는 것이 유지보수에 유리하다.

- Post(게시글) 파일 → **`posts/{postId}/{fileName}`**
- Notice(공지사항) 파일 → **`notices/{noticeId}/{fileName}`**
<br></br>
```java
@Entity
@Table(name = "file_metadata")
@Getter
@NoArgsConstructor(access = AccessLevel.PROTECTED)
@AllArgsConstructor
@Builder
public class FileMetadata {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long fileId;

    @Enumerated(EnumType.STRING)
    private FileCategory category; // POST, NOTICE, STUDY, ASSIGNMENT

    private Long entityId; // 특정 Post, Notice, Study, Assignment 의 Id

    private String fileName;

    private String filePath; // S3 저장경로 (uniqueFileName)
}
```

위의 코드와 같이 파일을 통합 file_metadata를 통해 관리한다. 즉 Post, Notice 구분 없이 하나의 테이블에 저장하였다. 각 파일이 속한 엔티티(Post, Notice)를 category + entity_id 로 구분한다. 이에 대한 장점으로는 테이블 중복 없어 유지보수 용이하며, 다양한 유형의 파일을 추가 가능하고, 모든 파일을 한 곳에서 관리 가능하다. 하지만, 단점으로는 특정 파일 유형(Post, Notice 등)에 대한 속성 추가 어렵다. 현재의 서비스에서는 속성의 구분이 없기에 통합 파일 메타데이터를 사용한다.
<br></br><br></br>
```java
@Transactional
    public String generatePresignedUrl(FileCategory category, Long entityId, String fileName) {
        String uniqueFileName = category.name().toLowerCase() + "/" + entityId + "/" +  UUID.randomUUID() + "_" + fileName;

        PutObjectPresignRequest request = PutObjectPresignRequest.builder()
                .signatureDuration(Duration.ofMinutes(10)) // 10분 유효
                .putObjectRequest(builder -> builder
                        .bucket(bucketName)
                        .key(uniqueFileName)
                        .build())
                .build();

        PresignedPutObjectRequest presignedRequest = s3Presigner.presignPutObject(request);

        // file metadata 정보를 데이터베이스에 저장
        FileMetadata fileMetadata = FileMetadata.builder()
                .category(category)
                .entityId(entityId)
                .fileName(fileName)
                .filePath(uniqueFileName)
                .build();
        fileMetadataRepository.save(fileMetadata);

        return presignedRequest.url().toString();
    }
```

위는 generatePresignedUrl 메서드를 통해 PresignedUrl을 생성함과 동시에 file metadata 정보를 데이터베이스에 저장하는 모습이다.
<br></br><br></br>
## 📚 참고

[[AWS, Java, Spring] Presigned Url로 S3에 이미지 업로드하기](https://ipekogosu.tistory.com/59)

[[Packy] Spring Boot에서 Presigned URL로 AWS S3에 파일 업로드하기](https://leeeeeyeon-dev.tistory.com/88)

[[Spring Boot] 서버리스 기반 S3 Presigned URL 적용하기](https://velog.io/@jmjmjmz732002/Spring-Boot-%EC%84%9C%EB%B2%84%EB%A6%AC%EC%8A%A4-%EA%B8%B0%EB%B0%98-S3-Presigned-URL-%EC%A0%81%EC%9A%A9%ED%95%98%EA%B8%B0#aws-s3-presigned-url)