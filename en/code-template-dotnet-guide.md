## Compute > Cloud Functions > Code Template Guide > .NET

This document details how to develop functions by using .NET from NHN Cloud's Cloud Functions service.

## Template information
| Item              | Value                  |
|-----------------|---------|
| **Supported version**       | 8       |
| **File name**         | func.cs |
| **Entry point** | func             |

## Basic template

### Hello World example
A basic form of function. Use `NhnContext` of `Nhn.DotNetCore.Api` namespace to access the logger and request information.

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

            // CsvHelper example: read CSV
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

### Context object
With the `NhnContext` object, you can access the logger, request, and response.

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
        // Use logger
        context.Logger.WriteInfo("Function execution started");

        // HTTP request information
        var request = context.Request;
        var method = request.Method;
        var headers = request.Headers;

        // Query parameters are taken directly from context.Arguments.
        var queryParams = context.Arguments;

        string body;
        using (var reader = new StreamReader(request.Body))
        {
            body = reader.ReadToEnd();
        }

        // Configure response data
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

## Download and use template file

### Template download
You can download the .NET template provided by Cloud Functions to develop a local environment.

**Template download link**: [dotnet.zip](https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_2acdfabf4efe4efc8a04c00b348110c9/cdn_origin/prod_cloud_functions/templates/dotnet/dotnet.zip)

### Template file structure
The structure of the downloaded template file is as follows:

```
dotnet.zip
├── exclude.txt
├── func.cs       # Main function file
└── nuget.txt     # Dependency management file
```

### Local development process

#### 1. Unzip
```bash
# Unzip
unzip dotnet.zip -d my-function

# Move to task directory
cd my-function
```

#### 2. Modify function codes
Modify `func.cs` file with the logic you want.

```csharp
// func.cs - Simple modification example
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
            // Import name from query parameter (explicit conversion with ToString())
            string name = context.Arguments.ContainsKey("name") ? context.Arguments["name"].ToString() : "World";

            // Import message from body if POST request
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

#### 3. Compress into a ZIP file
Compress the modified source code into ZIP file. You must compress it so that `func.cs` and `nuget.txt` are included at the top level.

```bash
zip my-function.zip func.cs nuget.txt
```

### Upload from Cloud Functions console
- When creating or modifying a function, select the **User Local Environment** method.
- Click **Select File** to upload the `my-function.zip` file you created.

## Process by HTTP method

### Process GET request
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

### Process POST request
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

## Manage package (`nuget.txt`)

Use the `nuget.txt` file to manage dependencies (NuGet packages). Write the required packages one per line.

```
# nuget.txt
Microsoft.Extensions.Configuration:8.0.0
```

### Example of configuration management
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

            // Generate simple configuration data
            var configData = new Dictionary<string, string>
            {
                {"Database:Host", "localhost"},
                {"Database:Port", "5432"},
                {"App:Name", "NHN Cloud Demo"},
                {"App:Version", "1.0.0"},
                {"Cache:Enabled", "true"},
                {"Cache:TTL", "300"}
            };

            // Configure with configuration builder
            var config = new ConfigurationBuilder()
                .AddInMemoryCollection(configData)
                .Build();

            // Import configuration values
            var dbHost = config["Database:Host"];
            var appName = config["App:Name"];
            var cacheEnabled = config["Cache:Enabled"];

            // Import configuration sections
            var dbSection = config.GetSection("Database");
            var dbPort = dbSection["Port"];

            // Generate JSON response
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
                        "config[\"Database:Host\"] - Direct key access",
                        "config.GetSection(\"Database\") - Section access",
                        "AddInMemoryCollection() - Memory configuration"
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

## Configure Entry Point

Entry Point is **file name**. (excluding extention)

- File name: `func.cs`
- Entry Point: `func`

**Important**: The class name must be `NhnFunction`, and the method acting as a function must have the signature `public string Execute(NhnContext context)`.

### Caution
- **Package version**: You can specify the package version in `nuget.txt`. (example: `Newtonsoft.Json:9.0.1`)
- **Type conflict**: Since type conflicts can occur when adding dependencies, we recommend using the .NET native library whenever possible or using a compatible version of the package.
