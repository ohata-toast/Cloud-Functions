## Compute > Cloud Functions > コードテンプレートガイド > Java

このドキュメントでは、NHN CloudのCloud FunctionsサービスでJavaを使用して関数を開発する方法を詳しく説明します。

## テンプレート情報
| 項目       | 値                |
|-----------------|--------------------|
| **サポートバージョン** | 17, 21             |
| **ファイル名**    | HelloWorld.java    |
| **Entry Point** | example.HelloWorld |

## 基本テンプレート

### Hello Worldの例
最も基本的な関数の形式です。

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

### Contextオブジェクト(RequestEntity)
Javaの関数では、Springの`RequestEntity`を通じてHTTPリクエスト情報にアクセスできます。

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
        // HTTPリクエスト情報
        HttpMethod method = req.getMethod();
        URI url = req.getUrl();
        HttpHeaders headers = req.getHeaders();
        Map<String, Object> body = req.getBody();

        // レスポンスデータの構成
        String responseBody = String.format(
            "Method: %s\nURL: %s\nHeaders: %s\nBody: %s",
            method, url, headers, body
        );

        return ResponseEntity.ok(responseBody);
    }
}
```

## テンプレートファイルのダウンロードと活用

### テンプレートのダウンロード
Cloud Functionsが提供するJavaテンプレートをダウンロードし、ローカル環境で開発できます。

**テンプレートのダウンロードリンク**: [java.zip](https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_2acdfabf4efe4efc8a04c00b348110c9/cdn_origin/prod_cloud_functions/templates/java/java.zip)

### テンプレートのファイル構造
ダウンロードしたテンプレートは、Mavenのプロジェクト構造に従っています。

```
java.zip
├── pom.xml
└── src
    └── main
        └── java
            └── example
                └── HelloWorld.java
```

### ローカルでの開発プロセス

#### 1. 解凍
```bash
# 解凍
unzip java.zip -d my-java-function

# 作業ディレクトリへ移動
cd my-java-function
```

#### 2. 関数コードの修正
`src/main/java/example/HelloWorld.java`ファイルを、目的のロジックに合わせて修正します。

```java
// HelloWorld.java - 簡単な修正例
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
            // クエリパラメータから名前を取得
            String name = Optional.ofNullable(req.getUrl().getQuery())
                                  .map(q -> q.split("="))
                                  .filter(p -> p.length > 1 && p[0].equals("name"))
                                  .map(p -> p[1])
                                  .orElse("World");

            // POSTリクエストの場合、bodyからメッセージを取得
            String customMessage = "";
            if (req.getMethod() == org.springframework.http.HttpMethod.POST && req.hasBody()) {
                customMessage = req.getBody().getOrDefault("message", "");
            }

            // レスポンスデータの構成
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

#### 3. ZIPファイルへ圧縮
修正したソースコードを、再度ZIPファイルへ圧縮します。`pom.xml`ファイルと`src`ディレクトリがルートディレクトリに含まれるように圧縮する必要があります。

```bash
# Windows (PowerShell)
Compress-Archive -Path .\pom.xml, .\src -DestinationPath my-function.zip

# Windows (7-Zipを使用する場合)
7z a my-function.zip pom.xml src

# macOS/Linux
zip -r my-function.zip pom.xml src

# 全てのファイルを含める(不要なファイルは除く)
zip -r my-function.zip . -x "*.git*" "target/*" "*.log"
```

### Cloud Functionsコンソールでのアップロード
- 関数を作成または修正する際に、**ユーザーローカル環境**方式を選択します。
- **ファイルを選択**をクリックし、作成した`my-function.zip`ファイルをアップロードします。

### アップロード時の注意事項
- **アップロードファイル**:ソースコードと`pom.xml`が含まれた**ZIPファイル**をアップロードする必要があります。
- **ZIPファイルの構造**: ZIPファイルのルートに`pom.xml`と`src`ディレクトリを配置する必要があります。
- **除外するファイル**: `target`ディレクトリや`.git`ディレクトリなど、不要なファイルは含めないでください。
- **ファイルサイズ**: ZIPファイルのサイズは100MiB以下に制限されます。

## HTTPメソッド別の処理

### GETリクエストの処理
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

        // クエリパラメータの取得
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

### POSTリクエストの処理
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

        // 処理ロジック
        Map<String, String> response = new HashMap<>();
        response.put("id", UUID.randomUUID().toString().substring(0, 8));
        response.put("name", name);
        response.put("email", requestBody.getOrDefault("email", "not provided"));
        response.put("processed_at", Instant.now().toString());

        return ResponseEntity.status(201).body(response);
    }
}
```

## パッケージ管理(`pom.xml`)

依存関係の管理には、`pom.xml`ファイルを修正します。`dependencies`セクションに必要なライブラリを追加すると、関数のアップロード時にCloud Functionsが自動で依存関係をダウンロードし、ビルドに含めます。

```xml
<dependencies>
    <!-- 基本Spring Boot依存関係(Jackson含む) -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
        <version>3.3.2</version>
        <scope>provided</scope>
    </dependency>

    <!-- Apache Commons Lang3ライブラリの追加例 -->
    <dependency>
        <groupId>org.apache.commons</groupId>
        <artifactId>commons-lang3</artifactId>
        <version>3.12.0</version>
    </dependency>
</dependencies>
```

### 外部ライブラリの活用例
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

        // Apache Commons Lang3のStringUtilsを使用した文字列操作
        Map<String, Object> response = new HashMap<>();
        response.put("original", originalText);
        response.put("reversed", StringUtils.reverse(originalText));
        response.put("isAlphanumeric", StringUtils.isAlphanumeric(originalText));
        response.put("wordCount", StringUtils.split(originalText, ' ').length);

        return ResponseEntity.ok(response);
    }
}
```

## エントリーポイントの設定

### 単一クラス
`パッケージ名。クラス名`をエントリーポイントとして使用します。

- パッケージ名: `example`
- クラス名: `HelloWorld`
- Entry Point: `example.HelloWorld`
- **重要**:関数として機能するメソッドは、`public ResponseEntity<?> call(RequestEntity<?> req)`というシグネチャを持つ必要があります。

### 注意事項
- **プロジェクト構造**: `src/main/java`のディレクトリ構造を維持する必要があります。
- **メモリと実行時間**:関数は限られたリソース内で動作する必要があるため、重い処理は避け、コードを最適化してください。
