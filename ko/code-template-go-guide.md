## Compute > Cloud Functions > 코드 템플릿 가이드 > Go

이 문서는 NHN Cloud의 Cloud Functions 서비스에서 Go를 사용하여 함수를 개발하는 방법을 상세히 설명합니다.

## 템플릿 정보
| 항목              | 값                                        |
|-----------------|------------------------------------------|
| **지원 버전**       | 1.22(사용 중단 예정), 1.23(사용 중단 예정), 1.24, 1.25 |
| **파일명**         | functions.go                             |
| **Entry Point** | Handler                                  |

## 기본 템플릿

### Hello World 예시
가장 기본적인 함수 형태입니다.

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

### Context 객체
Go 함수에서는 `http.ResponseWriter`와 `*http.Request`를 통해 HTTP 요청과 응답을 처리합니다.

```go
package main

import (
	"encoding/json"
	"log"
	"net/http"
)

func Handler(w http.ResponseWriter, r *http.Request) {
	// HTTP 요청 정보
	method := r.Method
	headers := r.Header
	query := r.URL.Query()

	// 응답 데이터 구성
	response := map[string]interface{}{
		"method":  method,
		"headers": headers,
		"query":   query,
		"path":    r.URL.Path,
	}

	// JSON 응답 설정
	w.Header().Set("Content-Type", "application/json")
	w.WriteHeader(http.StatusOK)

	if err := json.NewEncoder(w).Encode(response); err != nil {
		log.Printf("Error encoding response: %v", err)
	}
}
```

## 템플릿 파일 다운로드 및 활용

### 템플릿 다운로드
Cloud Functions에서 제공하는 Go 템플릿을 다운로드하여 로컬 환경에서 개발할 수 있습니다.

**템플릿 다운로드 링크**: [go.zip](https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_2acdfabf4efe4efc8a04c00b348110c9/cdn_origin/prod_cloud_functions/templates/go/go.zip)

### 템플릿 파일 구조
다운로드한 템플릿 파일의 구조는 다음과 같습니다.

```
go.zip
├── functions.go     # 메인 함수 파일
└── go.mod          # 의존성 관리 파일
```

#### functions.go
기본 fake 데이터 생성 함수가 포함되어 있습니다.
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

### 로컬 개발 과정

#### 1. 압축 해제
```bash
# 압축 해제
unzip go.zip -d my-function

# 작업 디렉터리 이동
cd my-function
```

#### 2. 함수 코드 수정
`functions.go` 파일을 원하는 로직으로 수정합니다.

```go
// functions.go - 간단한 수정 예시
package main

import (
	"encoding/json"
	"net/http"
	"time"
)

func Handler(w http.ResponseWriter, r *http.Request) {
	// 쿼리 파라미터에서 이름 가져오기
	name := r.URL.Query().Get("name")
	if name == "" {
		name = "World"
	}

	// POST 요청인 경우 body에서 메시지 가져오기
	var customMessage string
	if r.Method == "POST" {
		var data map[string]interface{}
		if err := json.NewDecoder(r.Body).Decode(&data); err == nil {
			if msg, ok := data["message"].(string); ok {
				customMessage = msg
			}
		}
	}

	// 응답 데이터 구성
	response := map[string]interface{}{
		"greeting":  "Hello, " + name + "!",
		"message":   customMessage,
		"method":    r.Method,
		"timestamp": time.Now().Format(time.RFC3339),
	}

	// JSON 응답
	w.Header().Set("Content-Type", "application/json")
	w.WriteHeader(http.StatusOK)

	if err := json.NewEncoder(w).Encode(response); err != nil {
		http.Error(w, "Internal Server Error", http.StatusInternalServerError)
	}
}
```

#### 3. ZIP 파일로 압축
수정된 코드를 다시 ZIP 파일로 압축합니다.

```bash
# Windows (PowerShell)
Compress-Archive -Path .\functions.go, .\go.mod -DestinationPath my-function.zip

# Windows (7-Zip 사용 시)
7z a my-function.zip functions.go go.mod

# macOS/Linux
zip my-function.zip functions.go go.mod

# 모든 파일 포함(추가 파일이 있는 경우)
zip -r my-function.zip . -x "*.git*" "go.sum" "test.go"
```

### Cloud Functions 콘솔에서 업로드
> 함수를 생성하거나 수정할 때 사용자 로컬 환경의 파일을 업로드 시 사용. (콘솔 사용 가이드 참고)

### 업로드 시 주의사항

#### ZIP 파일 구조
- ZIP 파일의 루트에 직접 `.go` 파일과 `go.mod`가 위치해야 합니다.
- 불필요한 폴더 구조는 피할 것을 권장합니다.

**올바른 구조:**
```
my-function.zip
├── functions.go
├── go.mod
└── utils.go (추가 파일이 있는 경우)
```

**잘못된 구조:**
```
my-function.zip
└── my-function/
    ├── functions.go
    └── go.mod
```

#### 파일 크기 제한
- ZIP 파일 크기는 100MiB 이하로 제한됩니다.
- `go.sum` 파일은 포함하지 마세요.

#### 제외할 파일들
```bash
# .gitignore와 유사하게 다음 파일들은 제외
zip -r my-function.zip . -x \
  "go.sum" \
  ".git/*" \
  "*.log" \
  "test.go" \
  ".env" \
  "*.zip"
```

## HTTP 메서드별 처리

### GET 요청 처리
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

	// 쿼리 파라미터 가져오기
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

### POST 요청 처리
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

	// request body 읽기
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

	// 필수 필드 검증
	if requestBody.Name == "" {
		http.Error(w, "Missing required field: name", http.StatusBadRequest)
		return
	}

	// 응답 데이터 생성
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
	// 간단한 ID 생성(실제로는 UUID 등 사용)
	return time.Now().Format("20060102150405")
}

func getOrDefault(value, defaultValue string) string {
	if value == "" {
		return defaultValue
	}
	return value
}
```

## 패키지 관리

### go.mod 작성
의존성 관리를 위해 `go.mod` 파일을 작성합니다.

```go
module example.com/myfunction

go 1.25

require (
    github.com/google/uuid v1.6.0
    github.com/gorilla/mux v1.8.1
)
```

### 외부 API 호출 예시
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

	// 외부 API 호출
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

	// 응답 구성
	response := map[string]interface{}{
		"user":       userData,
		"fetched_at": time.Now().Format(time.RFC3339),
	}

	w.Header().Set("Content-Type", "application/json")
	json.NewEncoder(w).Encode(response)
}
```

### 데이터 처리 예시
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

	// 데이터 처리
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

	// 데이터 정렬
	sort.Slice(processedData, func(i, j int) bool {
		return processedData[i].NormalizedName < processedData[j].NormalizedName
	})

	// 유효한 데이터만 필터링
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

## Entry Point 설정

### 단일 함수
함수명을 Entry Point로 사용합니다.

함수명: `Handler`
Entry Point: `Handler`

### 다중 함수
하나의 파일에서 여러 함수를 정의할 수 있습니다.

```go
// handlers.go
package main

import (
	"encoding/json"
	"net/http"
)

func GetUser(w http.ResponseWriter, r *http.Request) {
	// 사용자 조회 로직
	response := map[string]string{"message": "Get user"}
	w.Header().Set("Content-Type", "application/json")
	json.NewEncoder(w).Encode(response)
}

func CreateUser(w http.ResponseWriter, r *http.Request) {
	// 사용자 생성 로직
	response := map[string]string{"message": "User created"}
	w.Header().Set("Content-Type", "application/json")
	w.WriteHeader(http.StatusCreated)
	json.NewEncoder(w).Encode(response)
}

func UpdateUser(w http.ResponseWriter, r *http.Request) {
	// 사용자 수정 로직
	response := map[string]string{"message": "User updated"}
	w.Header().Set("Content-Type", "application/json")
	json.NewEncoder(w).Encode(response)
}
```

Entry Point 설정:
- `GetUser`
- `CreateUser`
- `UpdateUser`

## 주의사항

### Go 버전
현재 지원되는 Go 버전은 **1.24**와 **1.25**입니다.

**지원 중단 예정**: Go 1.22와 1.23은 지원이 중단되어 사용이 제한될 예정입니다.

#### Go 버전 지원 정책
Cloud Functions는 Go의 공식 릴리즈 정책을 따릅니다. Go는 연 2회(1월, 7월) 새로운 메이저 버전을 릴리즈하며, 각 버전은 최신 2개 메이저 버전까지만 지원됩니다.

**예시**:
- Go 1.24가 릴리즈되면 Go 1.22는 지원 종료
- Go 1.25가 릴리즈되면 Go 1.23은 지원 종료

**권장사항**: 최신 지원 버전(1.24 또는 1.25)을 사용하여 보안 패치와 버그 수정을 계속 받을 수 있도록 합니다.
