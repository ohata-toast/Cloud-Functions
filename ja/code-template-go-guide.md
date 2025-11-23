## Compute > Cloud Functions > コードテンプレートガイド > Go

このドキュメントでは、NHN CloudのCloud FunctionsサービスでGoを使用して関数を開発する方法を詳しく説明します。

## テンプレート情報
| 項目             | 値                                       |
|-----------------|------------------------------------------|
| **サポートバージョン**       | 1.22(使用中止予定)、1.23(使用中止予定)、1.24、1.25 |
| **ファイル名**         | functions.go                             |
| **Entry Point** | Handler                                  |

## 基本テンプレート

### Hello Worldの例
最も基本的な関数の形式です。

```go
package main

import (
	"fmt"
	"log"
	"net/http"

	"github.com/brianvoe/gofakeit/v6"
)

func Handler(w http.ResponseWriter, r *http.Request) {
	name := gofakeit.Name()
	email := gofakeit.Email()
	city := gofakeit.City()

	msg := fmt.Sprintf("Fake Name: %s\nEmail: %s\nCity: %s\n", name, email, city)
	_, err := w.Write([]byte(msg))
	if err != nil {
		log.Printf("Error writing response: %v", err)
	}
}
```

### Contextオブジェクト
Goの関数では、`http.ResponseWriter`と`*http.Request`を通じてHTTPのリクエストとレスポンスを処理します。

```go
package main

import (
	"encoding/json"
	"log"
	"net/http"
)

func Handler(w http.ResponseWriter, r *http.Request) {
	// HTTPリクエスト情報
	method := r.Method
	headers := r.Header
	query := r.URL.Query()

	// レスポンスデータの構成
	response := map[string]interface{}{
		"method":  method,
		"headers": headers,
		"query":   query,
		"path":    r.URL.Path,
	}

	// JSONレスポンスの設定
	w.Header().Set("Content-Type", "application/json")
	w.WriteHeader(http.StatusOK)

	if err := json.NewEncoder(w).Encode(response); err != nil {
		log.Printf("Error encoding response: %v", err)
	}
}
```

## テンプレートファイルのダウンロードと活用

### テンプレートのダウンロード
Cloud Functionsが提供するGoテンプレートをダウンロードし、ローカル環境で開発できます。

**テンプレートのダウンロードリンク**: [go.zip](https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_2acdfabf4efe4efc8a04c00b348110c9/cdn_origin/prod_cloud_functions/templates/go/go.zip)

### テンプレートのファイル構造
ダウンロードしたテンプレートファイルの構造は次のとおりです。

```
go.zip
├── functions.go     # メイン関数ファイル
└── go.mod          # 依存関係管理ファイル
```

#### functions.go
基本的なfakeデータ生成関数が含まれています。
```go
package main

import (
	"fmt"
	"log"
	"net/http"

	"github.com/brianvoe/gofakeit/v6"
)

func Handler(w http.ResponseWriter, r *http.Request) {
	name := gofakeit.Name()
	email := gofakeit.Email()
	city := gofakeit.City()

	msg := fmt.Sprintf("Fake Name: %s\nEmail: %s\nCity: %s\n", name, email, city)
	_, err := w.Write([]byte(msg))
	if err != nil {
		log.Printf("Error writing response: %v", err)
	}
}
```

#### go.mod
```go
module example.com/ncf

go 1.22

require github.com/brianvoe/gofakeit/v6 v6.28.0
```

### ローカルでの開発プロセス

#### 1. 解凍
```bash
# 解凍
unzip go.zip -d my-function

# 作業ディレクトリへ移動
cd my-function
```

#### 2. 関数コードの修正
`functions.go`ファイルを、目的のロジックに合わせて修正します。

```go
// functions.go - 簡単な修正例
package main

import (
	"encoding/json"
	"net/http"
	"time"
)

func Handler(w http.ResponseWriter, r *http.Request) {
	// クエリパラメータから名前を取得
	name := r.URL.Query().Get("name")
	if name == "" {
		name = "World"
	}

	// POSTリクエストの場合、bodyからメッセージを取得
	var customMessage string
	if r.Method == "POST" {
		var data map[string]interface{}
		if err := json.NewDecoder(r.Body).Decode(&data); err == nil {
			if msg, ok := data["message"].(string); ok {
				customMessage = msg
			}
		}
	}

	// レスポンスデータの構成
	response := map[string]interface{}{
		"greeting":  "Hello, " + name + "!",
		"message":   customMessage,
		"method":    r.Method,
		"timestamp": time.Now().Format(time.RFC3339),
	}

	// JSONレスポンス
	w.Header().Set("Content-Type", "application/json")
	w.WriteHeader(http.StatusOK)

	if err := json.NewEncoder(w).Encode(response); err != nil {
		http.Error(w, "Internal Server Error", http.StatusInternalServerError)
	}
}
```

#### 3. ZIPファイルへ圧縮
修正したコードを、再度ZIPファイルへ圧縮します。

```bash
# Windows (PowerShell)
Compress-Archive -Path .\functions.go, .\go.mod -DestinationPath my-function.zip

# Windows (7-Zipを使用する場合)
7z a my-function.zip functions.go go.mod

# macOS/Linux
zip my-function.zip functions.go go.mod

# 全てのファイルを含める(追加ファイルがある場合)
zip -r my-function.zip . -x "*.git*" "go.sum" "test.go"
```

### Cloud Functionsコンソールでのアップロード
> 関数を作成または修正する際に、ユーザーのローカル環境にあるファイルをアップロードする場合に使用します。(コンソール利用ガイド参照)

### アップロード時の注意事項

#### ZIPファイルの構造
- ZIPファイルのルートに、直接`.go`ファイルと`go.mod`を配置する必要があります。
- 不要なフォルダ構造は避けることを推奨します。

**正しい構造:**
```
my-function.zip
├── functions.go
├── go.mod
└── utils.go (追加ファイルがある場合)
```

**誤った構造:**
```
my-function.zip
└── my-function/
    ├── functions.go
    └── go.mod
```

#### ファイルサイズの制限
- ZIPファイルのサイズは100MiB以下に制限されます。
- `go.sum`ファイルは含めないでください。

#### 除外するファイル
```bash
# .gitignoreと同様に、以下のファイルは除外
zip -r my-function.zip . -x \
  "go.sum" \
  ".git/*" \
  "*.log" \
  "test.go" \
  ".env" \
  "*.zip"
```

## HTTPメソッド別の処理

### GETリクエストの処理
```go
package main

import (
	"encoding/json"
	"net/http"
	"time"
)

func Handler(w http.ResponseWriter, r *http.Request) {
	if r.Method != "GET" {
		http.Error(w, "Method Not Allowed", http.StatusMethodNotAllowed)
		return
	}

	// クエリパラメータの取得
	name := r.URL.Query().Get("name")
	if name == "" {
		name = "World"
	}

	greeting := r.URL.Query().Get("greeting")
	if greeting == "" {
		greeting = "Hello"
	}

	response := map[string]interface{}{
		"message":   greeting + ", " + name + "!",
		"timestamp": time.Now().Format(time.RFC3339),
		"method":    "GET",
	}

	w.Header().Set("Content-Type", "application/json")
	json.NewEncoder(w).Encode(response)
}
```

### POSTリクエストの処理
```go
package main

import (
	"encoding/json"
	"io"
	"net/http"
	"time"
)

type RequestBody struct {
	Name    string `json:"name"`
	Email   string `json:"email"`
	Message string `json:"message"`
}

func Handler(w http.ResponseWriter, r *http.Request) {
	if r.Method != "POST" {
		http.Error(w, "Method Not Allowed", http.StatusMethodNotAllowed)
		return
	}

	// request bodyの読み取り
	body, err := io.ReadAll(r.Body)
	if err != nil {
		http.Error(w, "Error reading body", http.StatusBadRequest)
		return
	}
	defer r.Body.Close()

	var requestBody RequestBody
	err = json.Unmarshal(body, &requestBody)
	if err != nil {
		http.Error(w, "Invalid JSON format", http.StatusBadRequest)
		return
	}

	// 必須フィールドの検証
	if requestBody.Name == "" {
		http.Error(w, "Missing required field: name", http.StatusBadRequest)
		return
	}

	// レスポンスデータの生成
	response := map[string]interface{}{
		"id":           generateID(),
		"name":         requestBody.Name,
		"email":        getOrDefault(requestBody.Email, "not provided"),
		"message":      getOrDefault(requestBody.Message, "No message"),
		"processed_at": time.Now().Format(time.RFC3339),
	}

	w.Header().Set("Content-Type", "application/json")
	w.WriteHeader(http.StatusCreated)
	json.NewEncoder(w).Encode(response)
}

func generateID() string {
	// 簡単なIDの生成(実際にはUUIDなどを使用)
	return time.Now().Format("20060102150405")
}

func getOrDefault(value, defaultValue string) string {
	if value == "" {
		return defaultValue
	}
	return value
}
```

## パッケージ管理

### go.modの作成
依存関係の管理には、`go.mod`ファイルを作成します。

```go
module example.com/myfunction

go 1.25

require (
    github.com/google/uuid v1.6.0
    github.com/gorilla/mux v1.8.1
)
```

### 外部API呼び出しの例
```go
package main

import (
	"encoding/json"
	"fmt"
	"net/http"
	"time"
)

func Handler(w http.ResponseWriter, r *http.Request) {
	userID := r.URL.Query().Get("userId")
	if userID == "" {
		http.Error(w, "userId is required", http.StatusBadRequest)
		return
	}

	// 外部APIの呼び出し
	resp, err := http.Get(fmt.Sprintf("https://jsonplaceholder.typicode.com/users/%s", userID))
	if err != nil {
		http.Error(w, "External API error", http.StatusInternalServerError)
		return
	}
	defer resp.Body.Close()

	if resp.StatusCode == 404 {
		http.Error(w, "User not found", http.StatusNotFound)
		return
	}

	if resp.StatusCode != 200 {
		http.Error(w, "External API error", http.StatusInternalServerError)
		return
	}

	var userData map[string]interface{}
	if err := json.NewDecoder(resp.Body).Decode(&userData); err != nil {
		http.Error(w, "Failed to parse response", http.StatusInternalServerError)
		return
	}

	// レスポンスの構成
	response := map[string]interface{}{
		"user":       userData,
		"fetched_at": time.Now().Format(time.RFC3339),
	}

	w.Header().Set("Content-Type", "application/json")
	json.NewEncoder(w).Encode(response)
}
```

### データ処理の例
```go
package main

import (
	"encoding/json"
	"io"
	"net/http"
	"sort"
	"strings"
	"time"

	"github.com/google/uuid"
)

type DataItem struct {
	Name string `json:"name"`
}

type ProcessedItem struct {
	ID             string `json:"id"`
	Name           string `json:"name"`
	ProcessedAt    string `json:"processed_at"`
	NormalizedName string `json:"normalized_name"`
}

func Handler(w http.ResponseWriter, r *http.Request) {
	body, err := io.ReadAll(r.Body)
	if err != nil {
		http.Error(w, "Error reading body", http.StatusBadRequest)
		return
	}
	defer r.Body.Close()

	var request struct {
		Data []DataItem `json:"data"`
	}

	if err := json.Unmarshal(body, &request); err != nil {
		http.Error(w, "Invalid JSON format", http.StatusBadRequest)
		return
	}

	if request.Data == nil {
		http.Error(w, "Data must be an array", http.StatusBadRequest)
		return
	}

	// データ処理
	var processedData []ProcessedItem
	for _, item := range request.Data {
		if item.Name != "" {
			processed := ProcessedItem{
				ID:             uuid.New().String()[:8],
				Name:           item.Name,
				ProcessedAt:    time.Now().Format(time.RFC3339),
				NormalizedName: normalizeName(item.Name),
			}
			processedData = append(processedData, processed)
		}
	}

	// データのソート
	sort.Slice(processedData, func(i, j int) bool {
		return processedData[i].NormalizedName < processedData[j].NormalizedName
	})

	// 有効なデータのみをフィルタリング
	validData := make([]ProcessedItem, 0)
	for _, item := range processedData {
		if item.NormalizedName != "" {
			validData = append(validData, item)
		}
	}

	response := map[string]interface{}{
		"total_processed": len(processedData),
		"valid_items":     len(validData),
		"data":           validData,
	}

	w.Header().Set("Content-Type", "application/json")
	json.NewEncoder(w).Encode(response)
}

func normalizeName(name string) string {
	if name == "" {
		return ""
	}
	return strings.Title(strings.TrimSpace(name))
}
```

## エントリーポイントの設定

### 単一関数
関数名をエントリーポイントとして使用します。

関数名: `Handler`
Entry Point: `Handler`

### 複数関数
1つのファイルで複数の関数を定義できます。

```go
// handlers.go
package main

import (
	"encoding/json"
	"net/http"
)

func GetUser(w http.ResponseWriter, r *http.Request) {
	// ユーザー照会ロジック
	response := map[string]string{"message": "Get user"}
	w.Header().Set("Content-Type", "application/json")
	json.NewEncoder(w).Encode(response)
}

func CreateUser(w http.ResponseWriter, r *http.Request) {
	// ユーザー作成ロジック
	response := map[string]string{"message": "User created"}
	w.Header().Set("Content-Type", "application/json")
	w.WriteHeader(http.StatusCreated)
	json.NewEncoder(w).Encode(response)
}

func UpdateUser(w http.ResponseWriter, r *http.Request) {
	// ユーザー修正ロジック
	response := map[string]string{"message": "User updated"}
	w.Header().Set("Content-Type", "application/json")
	json.NewEncoder(w).Encode(response)
}
```

エントリーポイントの設定:
- `GetUser`
- `CreateUser`
- `UpdateUser`

## 注意事項

### Goのバージョン
現在サポートされているGoバージョンは、**1.24**と**1.25**です。

**サポート終了予定**: Go 1.22と1.23はサポートが終了し、使用が制限される予定です。

#### Goバージョンサポートポリシー
Cloud Functionsは、Goの公式リリース方針に従います。Goは年2回(1月、7月)新しいメジャーバージョンをリリースしており、各バージョンは最新の2つのメジャーバージョンまでサポートされます。

**例**:
- Go 1.24がリリースされると、Go 1.22はサポート終了
- Go 1.25がリリースされると、Go 1.23はサポート終了

**推奨事項**: セキュリティパッチやバグ修正を引き続き適用できるよう、最新のサポートバージョン(1.24または1.25)を使用することを推奨します。
