## Compute > Cloud Functions > 코드 템플릿 가이드 > Java

이 문서는 NHN Cloud의 Cloud Functions 서비스에서 Java를 사용하여 함수를 개발하는 방법을 상세히 설명합니다.

## 템플릿 정보
| 항목              | 값                  |
|-----------------|--------------------|
| **지원 버전**       | 17, 21             |
| **파일명**         | HelloWorld.java    |
| **Entry Point** | example.HelloWorld |

## 기본 템플릿

### Hello World 예시
가장 기본적인 함수 형태입니다.

```java
package example;

import org.springframework.http.RequestEntity;
import org.springframework.http.ResponseEntity;

public class HelloWorld {

    public ResponseEntity<?> call(RequestEntity<?> req) {
        return ResponseEntity.ok("Hello World!");
    }

}
```

### Context 객체(RequestEntity)
Java 함수에서는 Spring의 `RequestEntity`를 통해 HTTP 요청 정보에 접근할 수 있습니다.

```java
package example;

import org.springframework.http.HttpHeaders;
import org.springframework.http.HttpMethod;
import org.springframework.http.RequestEntity;
import org.springframework.http.ResponseEntity;
import java.net.URI;
import java.util.Map;

public class HelloWorld {

    public ResponseEntity<?> call(RequestEntity<Map<String, Object>> req) {
        // HTTP 요청 정보
        HttpMethod method = req.getMethod();
        URI url = req.getUrl();
        HttpHeaders headers = req.getHeaders();
        Map<String, Object> body = req.getBody();

        // 응답 데이터 구성
        String responseBody = String.format(
            "Method: %s\nURL: %s\nHeaders: %s\nBody: %s",
            method, url, headers, body
        );

        return ResponseEntity.ok(responseBody);
    }
}
```

## 템플릿 파일 다운로드 및 활용

### 템플릿 다운로드
Cloud Functions에서 제공하는 Java 템플릿을 다운로드하여 로컬 환경에서 개발할 수 있습니다.

**템플릿 다운로드 링크**: [java.zip](https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_2acdfabf4efe4efc8a04c00b348110c9/cdn_origin/prod_cloud_functions/templates/java/java.zip)

### 템플릿 파일 구조
다운로드한 템플릿은 Maven 프로젝트 구조를 따릅니다.

```
java.zip
├── pom.xml
└── src
    └── main
        └── java
            └── example
                └── HelloWorld.java
```

### 로컬 개발 과정

#### 1. 압축 해제
```bash
# 압축 해제
unzip java.zip -d my-java-function

# 작업 디렉터리 이동
cd my-java-function
```

#### 2. 함수 코드 수정
`src/main/java/example/HelloWorld.java` 파일을 원하는 로직으로 수정합니다.

```java
// HelloWorld.java - 간단한 수정 예시
package example;

import org.springframework.http.RequestEntity;
import org.springframework.http.ResponseEntity;
import java.time.ZonedDateTime;
import java.time.format.DateTimeFormatter;
import java.util.HashMap;
import java.util.Map;
import java.util.Optional;

public class HelloWorld {

    public ResponseEntity<?> call(RequestEntity<Map<String, String>> req) {
        try {
            // 쿼리 파라미터에서 이름 가져오기
            String name = Optional.ofNullable(req.getUrl().getQuery())
                                  .map(q -> q.split("="))
                                  .filter(p -> p.length > 1 && p[0].equals("name"))
                                  .map(p -> p[1])
                                  .orElse("World");

            // POST 요청인 경우 body에서 메시지 가져오기
            String customMessage = "";
            if (req.getMethod() == org.springframework.http.HttpMethod.POST && req.hasBody()) {
                customMessage = req.getBody().getOrDefault("message", "");
            }

            // 응답 데이터 구성
            Map<String, String> response = new HashMap<>();
            response.put("greeting", "Hello, " + name + "!");
            response.put("message", customMessage);
            response.put("method", req.getMethod().toString());
            response.put("timestamp", ZonedDateTime.now().format(DateTimeFormatter.ISO_INSTANT));

            return ResponseEntity.ok(response);

        } catch (Exception e) {
            Map<String, String> errorResponse = new HashMap<>();
            errorResponse.put("error", "Internal Server Error");
            errorResponse.put("details", e.getMessage());
            return ResponseEntity.status(500).body(errorResponse);
        }
    }
}
```

#### 3. ZIP 파일로 압축
수정된 소스 코드를 다시 ZIP 파일로 압축합니다. `pom.xml` 파일과 `src` 디렉터리가 최상위에 포함되도록 압축해야 합니다.

```bash
# Windows (PowerShell)
Compress-Archive -Path .\pom.xml, .\src -DestinationPath my-function.zip

# Windows (7-Zip 사용 시)
7z a my-function.zip pom.xml src

# macOS/Linux
zip -r my-function.zip pom.xml src

# 모든 파일 포함(불필요한 파일 제외)
zip -r my-function.zip . -x "*.git*" "target/*" "*.log"
```

### Cloud Functions 콘솔에서 업로드
- 함수 생성 또는 수정 시, **사용자 로컬 환경** 방식을 선택합니다.
- **파일 선택**을 클릭하여 생성한 `my-function.zip` 파일을 업로드합니다.

### 업로드 시 주의사항
- **업로드 파일**: 소스 코드와 `pom.xml`이 포함된 **ZIP 파일**을 업로드해야 합니다.
- **ZIP 파일 구조**: ZIP 파일의 루트에 `pom.xml`과 `src` 디렉터리가 위치해야 합니다.
- **제외할 파일**: `target` 디렉터리, `.git` 디렉터리 등 불필요한 파일은 포함하지 마세요.
- **파일 크기**: ZIP 파일 크기는 100MiB 이하로 제한됩니다.

## HTTP 메서드별 처리

### GET 요청 처리
```java
package example;

import org.springframework.http.RequestEntity;
import org.springframework.http.ResponseEntity;
import java.net.URLDecoder;
import java.nio.charset.StandardCharsets;
import java.time.Instant;
import java.util.HashMap;
import java.util.Map;
import java.util.Optional;

public class GetHandler {

    public ResponseEntity<?> call(RequestEntity<Void> req) {
        if (req.getMethod() != org.springframework.http.HttpMethod.GET) {
            return ResponseEntity.status(405).body("Method Not Allowed");
        }

        // 쿼리 파라미터 가져오기
        String name = Optional.ofNullable(req.getUrl().getQuery())
                              .map(q -> URLDecoder.decode(q, StandardCharsets.UTF_8))
                              .map(q -> q.split("=")[1])
                              .orElse("World");

        Map<String, String> response = new HashMap<>();
        response.put("message", "Hello, " + name + "!");
        response.put("timestamp", Instant.now().toString());
        response.put("method", "GET");

        return ResponseEntity.ok(response);
    }
}
```

### POST 요청 처리
```java
package example;

import org.springframework.http.RequestEntity;
import org.springframework.http.ResponseEntity;
import java.time.Instant;
import java.util.HashMap;
import java.util.Map;
import java.util.UUID;

public class PostHandler {

    public ResponseEntity<?> call(RequestEntity<Map<String, String>> req) {
        if (req.getMethod() != org.springframework.http.HttpMethod.POST) {
            return ResponseEntity.status(405).body("Method Not Allowed");
        }

        if (!req.hasBody()) {
            return ResponseEntity.badRequest().body("Request body is missing");
        }

        Map<String, String> requestBody = req.getBody();
        String name = requestBody.get("name");

        if (name == null || name.isEmpty()) {
            return ResponseEntity.badRequest().body("Missing required field: name");
        }

        // 처리 로직
        Map<String, String> response = new HashMap<>();
        response.put("id", UUID.randomUUID().toString().substring(0, 8));
        response.put("name", name);
        response.put("email", requestBody.getOrDefault("email", "not provided"));
        response.put("processed_at", Instant.now().toString());

        return ResponseEntity.status(201).body(response);
    }
}
```

## 패키지 관리(`pom.xml`)

의존성 관리를 위해 `pom.xml` 파일을 수정합니다. `dependencies` 섹션에 필요한 라이브러리를 추가하면, 함수 업로드 시 Cloud Functions가 자동으로 의존성을 다운로드하여 빌드에 포함합니다.

```xml
<dependencies>
    <!-- 기본 Spring Boot 의존성(Jackson 포함) -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
        <version>3.3.2</version>
        <scope>provided</scope>
    </dependency>

    <!-- Apache Commons Lang3 라이브러리 추가 예시 -->
    <dependency>
        <groupId>org.apache.commons</groupId>
        <artifactId>commons-lang3</artifactId>
        <version>3.12.0</version>
    </dependency>
</dependencies>
```

### 외부 라이브러리 활용 예시
```java
package example;

import org.apache.commons.lang3.StringUtils;
import org.springframework.http.RequestEntity;
import org.springframework.http.ResponseEntity;
import java.util.HashMap;
import java.util.Map;

public class StringUtilsHandler {

    public ResponseEntity<?> call(RequestEntity<Map<String, String>> req) {
        if (!req.hasBody()) {
            return ResponseEntity.badRequest().body("Request body is missing");
        }

        String originalText = req.getBody().get("text");
        if (StringUtils.isEmpty(originalText)) {
            return ResponseEntity.badRequest().body("Body must contain a 'text' field.");
        }

        // Apache Commons Lang3 StringUtils를 사용한 문자열 조작
        Map<String, Object> response = new HashMap<>();
        response.put("original", originalText);
        response.put("reversed", StringUtils.reverse(originalText));
        response.put("isAlphanumeric", StringUtils.isAlphanumeric(originalText));
        response.put("wordCount", StringUtils.split(originalText, ' ').length);

        return ResponseEntity.ok(response);
    }
}
```

## Entry Point 설정

### 단일 클래스
`패키지명.클래스명`을 Entry Point로 사용합니다.

- 패키지명: `example`
- 클래스명: `HelloWorld`
- Entry Point: `example.HelloWorld`
- **중요**: 함수 역할을 하는 메서드는 `public ResponseEntity<?> call(RequestEntity<?> req)` 시그니처를 가져야 합니다.

### 주의사항
- **프로젝트 구조**: `src/main/java` 디렉터리 구조를 유지해야 합니다.
- **메모리 및 실행 시간**: 함수는 제한된 리소스 내에서 동작해야 하므로, 무거운 작업은 피하고 코드를 최적화하세요.
