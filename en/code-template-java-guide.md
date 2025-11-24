## Compute > Cloud Functions > Code Template Guide > Java

This document details how to develop functions by using Java from NHN Cloud's Cloud Functions service.

## Template information
| Item               | Value                  |
|-----------------|---------------------|
| **Supported version**       | 17, 21             |
| **File name**          | HelloWorld.java    |
| **Entry Point** | example.HelloWorld |

## Basic template

### Hello World example
A basic form of function.

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

### Context object (RequestEntity)
In a Java function, you can access HTTP request information through `RequestEntity`.

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
        // HTTP request information
        HttpMethod method = req.getMethod();
        URI url = req.getUrl();
        HttpHeaders headers = req.getHeaders();
        Map<String, Object> body = req.getBody();

        // Response data configuration
        String responseBody = String.format(
            "Method: %s\nURL: %s\nHeaders: %s\nBody: %s",
            method, url, headers, body
        );

        return ResponseEntity.ok(responseBody);
    }
}
```

## Download and use template file

### Template download
You can download the Java template provided by Cloud Functions to develop a local environment.

**Template download link**: [java.zip](https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_2acdfabf4efe4efc8a04c00b348110c9/cdn_origin/prod_cloud_functions/templates/java/java.zip)

### Template file structure
The structure of the downloaded template file is as follows:

```
java.zip
├── pom.xml
└── src
    └── main
        └── java
            └── example
                └── HelloWorld.java
```

### Local development process

#### 1. Unzip
```bash
# Unzip
unzip java.zip -d my-java-function

# Move to task directory
cd my-java-function
```

#### 2. Modify function codes
Modify `src/main/java/example/HelloWorld.java` file with the logic you want.

```java
// HelloWorld.java - Simple modification example
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
            // Import name from query parameter
            String name = Optional.ofNullable(req.getUrl().getQuery())
                                  .map(q -> q.split("="))
                                  .filter(p -> p.length > 1 && p[0].equals("name"))
                                  .map(p -> p[1])
                                  .orElse("World");

            // Import message from body if POST request
            String customMessage = "";
            if (req.getMethod() == org.springframework.http.HttpMethod.POST && req.hasBody()) {
                customMessage = req.getBody().getOrDefault("message", "");
            }

            // Response data configuration
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

#### 3. Compress into a ZIP file
Compress the modified source code into ZIP file. You must compress it so that `pom.xml` and `src` are included at the top level.

```bash
# Windows (PowerShell)
Compress-Archive -Path .\pom.xml, .\src -DestinationPath my-function.zip

# Windows (when using 7-Zip)
7z a my-function.zip pom.xml src

# macOS/Linux
zip -r my-function.zip pom.xml src

# Include all files (exclude unnecessary files)
zip -r my-function.zip . -x "*.git*" "target/*" "*.log"
```

### Upload from Cloud Functions console
- When creating or modifying a function, select the **User Local Environment** method.
- Click **Select File** to upload the `my-function.zip` file you created.

### Cautions for upload
- **Upload File**: Upload **ZIP file** which includes source code and `pom.xml`.
- **ZIP File Structure**: The `pom.xml` and `src` directory must be located directly in the root of the ZIP file.
- **File to Exclude**: Do not include unnecessary files such as the `target` directory, `.git` directory, etc.
- **File Size**: ZIP file size is limited to 100 MiB.

## Process by HTTP method

### Process GET request
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

        // Import query parameter
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

### Process POST request
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

        // Processing logic
        Map<String, String> response = new HashMap<>();
        response.put("id", UUID.randomUUID().toString().substring(0, 8));
        response.put("name", name);
        response.put("email", requestBody.getOrDefault("email", "not provided"));
        response.put("processed_at", Instant.now().toString());

        return ResponseEntity.status(201).body(response);
    }
}
```

## Manage package (`pom.xml`)

Use the `pom.xml` file to manage dependencies. When you add required libraries to the `dependencies` section, Cloud Functions automatically downloads the dependencies and includes them in the build when you upload your function.

```xml
<dependencies>
    <!-- Basic Spring Boot dependencies (include Jackson) -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
        <version>3.3.2</version>
        <scope>provided</scope>
    </dependency>

    <!-- Example of adding Apache Commons Lang3 library -->
    <dependency>
        <groupId>org.apache.commons</groupId>
        <artifactId>commons-lang3</artifactId>
        <version>3.12.0</version>
    </dependency>
</dependencies>
```

### Example of using external libraries
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

        // String manipulation using Apache Commons Lang3 StringUtils
        Map<String, Object> response = new HashMap<>();
        response.put("original", originalText);
        response.put("reversed", StringUtils.reverse(originalText));
        response.put("isAlphanumeric", StringUtils.isAlphanumeric(originalText));
        response.put("wordCount", StringUtils.split(originalText, ' ').length);

        return ResponseEntity.ok(response);
    }
}
```

## Entry Point configuration

### Single class
Use `packagename.classname` as an Entry Point.

- Package name: `example`
- Class name: `HelloWorld`
- Entry Point: `example.HelloWorld`
- **Important**: Methods that act as functions must have the signature `public ResponseEntity<?> call(RequestEntity<?> req)`.

### Caution
- **Project structure**: You must maintain the `src/main/java` directory structure.
- **Memory and execution time**: Functions must operate within limited resources, so avoid heavy workloads and optimize the code.
