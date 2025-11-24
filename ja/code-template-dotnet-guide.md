## Compute > Cloud Functions > コードテンプレートガイド > .NET

このドキュメントでは、NHN CloudのCloud Functionsサービスで.NETを使用して関数を開発する方法を詳しく説明します。

## テンプレート情報
| 項目       | 値                |
|-----------------|---------|
| **サポートバージョン** | 8                  |
| **ファイル名**    | func.cs            |
| **Entry Point** | func             |

## 基本テンプレート

### Hello Worldの例
最も基本的な関数の形式です。`Nhn.DotNetCore.Api`名前空間の`NhnContext`を使用して、ロガーやリクエスト情報にアクセスします。

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

            // CsvHelperの例: CSVの読み取り
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

### Contextオブジェクト
`NhnContext`オブジェクトを通じて、ロガー、リクエスト(Request)、レスポンス(Response)オブジェクトにアクセスできます。

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
        // ロガーの使用
        context.Logger.WriteInfo("Function execution started");

        // HTTPリクエスト情報
        var request = context.Request;
        var method = request.Method;
        var headers = request.Headers;

        // クエリパラメータはcontext.Argumentsから直接取得します。
        var queryParams = context.Arguments;

        string body;
        using (var reader = new StreamReader(request.Body))
        {
            body = reader.ReadToEnd();
        }

        // レスポンスデータの構成
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

## テンプレートファイルのダウンロードと活用

### テンプレートのダウンロード
Cloud Functionsが提供する.NETテンプレートをダウンロードし、ローカル環境で開発できます。

**テンプレートのダウンロードリンク**: [dotnet.zip](https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_2acdfabf4efe4efc8a04c00b348110c9/cdn_origin/prod_cloud_functions/templates/dotnet/dotnet.zip)

### テンプレートのファイル構造
ダウンロードしたテンプレートファイルの構造は次のとおりです。

```
dotnet.zip
├── exclude.txt
├── func.cs       # メイン関数ファイル
└── nuget.txt     # 依存関係管理ファイル
```

### ローカルでの開発プロセス

#### 1. 解凍
```bash
# 解凍
unzip dotnet.zip -d my-function

# 作業ディレクトリへ移動
cd my-function
```

#### 2. 関数コードの修正
`func.cs`ファイルを、目的のロジックに合わせて修正します。

```csharp
// func.cs - 簡単な修正例
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
            // クエリパラメータから名前を取得(ToString()で明示的に変換)
            string name = context.Arguments.ContainsKey("name") ? context.Arguments["name"].ToString() : "World";

            // POSTリクエストの場合、bodyからメッセージを取得
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

#### 3. ZIPファイルへ圧縮
修正したソースコードを、再度ZIPファイルへ圧縮します。`func.cs`と`nuget.txt`がルートディレクトリに含まれるように圧縮する必要があります。

```bash
zip my-function.zip func.cs nuget.txt
```

### Cloud Functionsコンソールでのアップロード
- 関数を作成または修正する際に、**ユーザーローカル環境**方式を選択します。
- **ファイルを選択**をクリックし、作成した`my-function.zip`ファイルをアップロードします。

## HTTPメソッド別の処理

### GETリクエストの処理
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

### POSTリクエストの処理
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

## パッケージ管理(`nuget.txt`)

依存関係(NuGetパッケージ)の管理には、`nuget.txt`ファイルを使用します。必要なパッケージを1行に1つずつ記述してください。

```
# nuget.txt
Microsoft.Extensions.Configuration:8.0.0
```

### 設定管理の例
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

            // 簡単な設定データを作成
            var configData = new Dictionary<string, string>
            {
                {"Database:Host", "localhost"},
                {"Database:Port", "5432"},
                {"App:Name", "NHN Cloud Demo"},
                {"App:Version", "1.0.0"},
                {"Cache:Enabled", "true"},
                {"Cache:TTL", "300"}
            };

            // 設定ビルダーで構成
            var config = new ConfigurationBuilder()
                .AddInMemoryCollection(configData)
                .Build();

            // 設定値を取得
            var dbHost = config["Database:Host"];
            var appName = config["App:Name"];
            var cacheEnabled = config["Cache:Enabled"];

            // 設定セクションを取得
            var dbSection = config.GetSection("Database");
            var dbPort = dbSection["Port"];

            // JSONレスポンスを作成
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
                        "config[\"Database:Host\"] - キーへの直接アクセス",
                        "config.GetSection(\"Database\") - セクションへのアクセス",
                        "AddInMemoryCollection() - メモリ設定"
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

## エントリーポイントの設定

エントリーポイントは**ファイル名**です。(拡張子を除く)

- ファイル名: `func.cs`
- Entry Point: `func`

**重要**:クラス名は`NhnFunction`である必要があり、関数として機能するメソッドは`public string Execute(NhnContext context)`というシグネチャを持つ必要があります。

### 注意事項
- **パッケージバージョン**: `nuget.txt`でパッケージのバージョンを明記できます。(例: `Newtonsoft.Json:9.0.1`)
- **タイプの競合**:依存関係を追加する際にタイプの競合が発生する可能性があるため、可能な限り.NETの基本ライブラリを使用するか、互換性のあるバージョンのパッケージを使用することを推奨します。
