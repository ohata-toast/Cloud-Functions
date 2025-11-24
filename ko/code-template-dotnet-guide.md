## Compute > Cloud Functions > 코드 템플릿 가이드 > .NET

이 문서는 NHN Cloud의 Cloud Functions 서비스에서 .NET을 사용하여 함수를 개발하는 방법을 상세히 설명합니다.

## 템플릿 정보
| 항목              | 값       |
|-----------------|---------|
| **지원 버전**       | 8       |
| **파일명**         | func.cs |
| **Entry Point** | func    |

## 기본 템플릿

### Hello World 예시
가장 기본적인 함수 형태입니다. `Nhn.DotNetCore.Api` 네임스페이스의 `NhnContext`를 사용하여 로거 및 요청 정보에 접근합니다.

```csharp
using System;
using System.IO;
using System.Globalization;
using CsvHelper;
using Nhn.DotNetCore.Api;

public class NhnFunction
{
    public string Execute(NhnContext context)
    {
        string respo = "initial value";
        try
        {
            context.Logger.WriteInfo("Starting..... ");

            // CsvHelper 예제: CSV 읽기
            var csvData = "Name,Age\nJohn,30\nJane,25";
            using (var reader = new StringReader(csvData))
            using (var csv = new CsvReader(reader, CultureInfo.InvariantCulture))
            {
                var records = csv.GetRecords<dynamic>();
                respo = "CSV Data Processed!";
            }
        }
        catch (Exception ex)
        {
            context.Logger.WriteError(ex.Message);
            respo = ex.Message;
        }
        context.Logger.WriteInfo("Done!");
        return respo;
    }
}
```

### Context 객체
`NhnContext` 객체를 통해 로거, 요청(Request), 응답(Response) 객체에 접근할 수 있습니다.

```csharp
using System;
using System.IO;
using System.Collections.Generic;
using Nhn.DotNetCore.Api;
using Newtonsoft.Json;

public class NhnFunction
{
    public string Execute(NhnContext context)
    {
        // 로거 사용
        context.Logger.WriteInfo("Function execution started");

        // HTTP 요청 정보
        var request = context.Request;
        var method = request.Method;
        var headers = request.Headers;

        // 쿼리 파라미터는 context.Arguments에서 직접 가져옵니다.
        var queryParams = context.Arguments;

        string body;
        using (var reader = new StreamReader(request.Body))
        {
            body = reader.ReadToEnd();
        }

        // 응답 데이터 구성
        var responseData = new Dictionary<string, object>
        {
            { "method", method },
            { "headers", headers },
            { "queryParams", queryParams },
            { "body", body }
        };

        return JsonConvert.SerializeObject(responseData);
    }
}
```

## 템플릿 파일 다운로드 및 활용

### 템플릿 다운로드
Cloud Functions에서 제공하는 .NET 템플릿을 다운로드하여 로컬 환경에서 개발할 수 있습니다.

**템플릿 다운로드 링크**: [dotnet.zip](https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_2acdfabf4efe4efc8a04c00b348110c9/cdn_origin/prod_cloud_functions/templates/dotnet/dotnet.zip)

### 템플릿 파일 구조
다운로드한 템플릿 파일의 구조는 다음과 같습니다.

```
dotnet.zip
├── exclude.txt
├── func.cs       # 메인 함수 파일
└── nuget.txt     # 의존성 관리 파일
```

### 로컬 개발 과정

#### 1. 압축 해제
```bash
# 압축 해제
unzip dotnet.zip -d my-function

# 작업 디렉터리 이동
cd my-function
```

#### 2. 함수 코드 수정
`func.cs` 파일을 원하는 로직으로 수정합니다.

```csharp
// func.cs - 간단한 수정 예시
using System;
using System.IO;
using System.Collections.Generic;
using Nhn.DotNetCore.Api;
using Newtonsoft.Json;
using Newtonsoft.Json.Linq;

public class NhnFunction
{
    public string Execute(NhnContext context)
    {
        try
        {
            // 쿼리 파라미터에서 이름 가져오기(ToString()으로 명시적 변환)
            string name = context.Arguments.ContainsKey("name") ? context.Arguments["name"].ToString() : "World";

            // POST 요청인 경우 body에서 메시지 가져오기
            string customMessage = "";
            if (context.Request.Method == "POST")
            {
                using (var reader = new StreamReader(context.Request.Body))
                {
                    string body = reader.ReadToEnd();
                    if (!string.IsNullOrEmpty(body))
                    {
                        JObject jObject = JObject.Parse(body);
                        customMessage = jObject["message"]?.ToString() ?? "";
                    }
                }
            }

            var responseData = new Dictionary<string, object>
            {
                { "greeting", $"Hello, {name}!" },
                { "message", customMessage },
                { "method", context.Request.Method },
                { "timestamp", DateTime.UtcNow.ToString("o") }
            };

            return JsonConvert.SerializeObject(responseData);
        }
        catch (Exception ex)
        {
            context.Logger.WriteError($"Error: {ex.Message}");
            return $"{{ \"error\": \"Internal Server Error\", \"details\": \"{ex.Message}\" }}";
        }
    }
}
```

#### 3. ZIP 파일로 압축
수정된 소스 코드를 다시 ZIP 파일로 압축합니다. `func.cs`와 `nuget.txt`가 최상위에 포함되도록 압축해야 합니다.

```bash
zip my-function.zip func.cs nuget.txt
```

### Cloud Functions 콘솔에서 업로드
- 함수 생성 또는 수정 시, **사용자 로컬 환경** 방식을 선택합니다.
- **파일 선택**을 클릭하여 생성한 `my-function.zip` 파일을 업로드합니다.

## HTTP 메서드별 처리

### GET 요청 처리
```csharp
using System;
using Nhn.DotNetCore.Api;
using Newtonsoft.Json;
using System.Collections.Generic;

public class NhnFunction
{
    public string Execute(NhnContext context)
    {
        if (context.Request.Method != "GET")
        {
            return "{ \"error\": \"Method Not Allowed\" }";
        }

        string name = context.Arguments.ContainsKey("name") ? context.Arguments["name"].ToString() : "World";

        var responseData = new Dictionary<string, object>
        {
            { "message", $"Hello, {name}!" },
            { "timestamp", DateTime.UtcNow.ToString("o") }
        };

        return JsonConvert.SerializeObject(responseData);
    }
}
```

### POST 요청 처리
```csharp
using System;
using System.IO;
using Nhn.DotNetCore.Api;
using Newtonsoft.Json;
using Newtonsoft.Json.Linq;
using System.Collections.Generic;

public class NhnFunction
{
    public string Execute(NhnContext context)
    {
        if (context.Request.Method != "POST")
        {
            return "{ \"error\": \"Method Not Allowed\" }";
        }

        try
        {
            string requestBodyString;
            using (var reader = new StreamReader(context.Request.Body))
            {
                requestBodyString = reader.ReadToEnd();
            }

            JObject requestBody = JObject.Parse(requestBodyString);
            string name = requestBody["name"]?.ToString();

            if (string.IsNullOrEmpty(name))
            {
                return "{ \"error\": \"Missing required field: name\" }";
            }

            var responseData = new Dictionary<string, object>
            {
                { "id", Guid.NewGuid().ToString("N").Substring(0, 8) },
                { "name", name },
                { "email", requestBody["email"]?.ToString() ?? "not provided" },
                { "processed_at", DateTime.UtcNow.ToString("o") }
            };

            return JsonConvert.SerializeObject(responseData);
        }
        catch (JsonException)
        {
            return "{ \"error\": \"Invalid JSON format\" }";
        }
    }
}
```

## 패키지 관리(`nuget.txt`)

의존성(NuGet 패키지) 관리를 위해 `nuget.txt` 파일을 사용합니다. 필요한 패키지를 한 줄에 하나씩 작성합니다.

```
# nuget.txt
Microsoft.Extensions.Configuration:8.0.0
```

### 설정 관리 예시
```csharp
using System;
using System.Collections.Generic;
using Microsoft.Extensions.Configuration;
using Newtonsoft.Json;
using Nhn.DotNetCore.Api;

public class NhnFunction
{
    public string Execute(NhnContext context)
    {
        try
        {
            context.Logger.WriteInfo("Simple configuration example started");

            // 간단한 설정 데이터 생성
            var configData = new Dictionary<string, string>
            {
                {"Database:Host", "localhost"},
                {"Database:Port", "5432"},
                {"App:Name", "NHN Cloud Demo"},
                {"App:Version", "1.0.0"},
                {"Cache:Enabled", "true"},
                {"Cache:TTL", "300"}
            };

            // 설정 빌더로 구성
            var config = new ConfigurationBuilder()
                .AddInMemoryCollection(configData)
                .Build();

            // 설정 값들 가져오기
            var dbHost = config["Database:Host"];
            var appName = config["App:Name"];
            var cacheEnabled = config["Cache:Enabled"];

            // 설정 섹션 가져오기
            var dbSection = config.GetSection("Database");
            var dbPort = dbSection["Port"];

            // JSON 응답 생성
            var responseData = new Dictionary<string, object>
            {
                { "message", "Microsoft Configuration Demo" },
                { "package", "Microsoft.Extensions.Configuration:8.0.0" },
                { "results", new Dictionary<string, object>
                    {
                        { "databaseHost", dbHost },
                        { "databasePort", dbPort },
                        { "appName", appName },
                        { "cacheEnabled", bool.Parse(cacheEnabled) },
                        { "configCount", configData.Count }
                    }
                },
                { "usage", new[]
                    {
                        "config[\"Database:Host\"] - 직접 키 접근",
                        "config.GetSection(\"Database\") - 섹션 접근",
                        "AddInMemoryCollection() - 메모리 설정"
                    }
                },
                { "timestamp", DateTime.UtcNow.ToString("o") }
            };

            return JsonConvert.SerializeObject(responseData, Formatting.Indented);
        }
        catch (Exception ex)
        {
            context.Logger.WriteError($"Error: {ex.Message}");
            var errorResponse = new Dictionary<string, object>
            {
                { "error", ex.Message },
                { "timestamp", DateTime.UtcNow.ToString("o") }
            };
            return JsonConvert.SerializeObject(errorResponse);
        }
    }
}
```

## Entry Point 설정

Entry Point는 **파일명**입니다. (확장자 제외)

- 파일명: `func.cs`
- Entry Point: `func`

**중요**: 클래스 이름은 `NhnFunction`이어야 하며, 함수 역할을 하는 메서드는 `public string Execute(NhnContext context)` 시그니처를 가져야 합니다.

### 주의사항
- **패키지 버전**: `nuget.txt`에서 패키지 버전을 명시할 수 있습니다. (예: `Newtonsoft.Json:9.0.1`)
- **타입 충돌**: 의존성 추가 시 타입 충돌이 발생할 수 있으므로, 가능한 .NET 기본 라이브러리를 사용하거나 호환되는 버전의 패키지를 사용하는 것을 권장합니다.
