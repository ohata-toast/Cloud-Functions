## Compute > Cloud Functions > Code Template Guide > Go

This document details how to develop functions by using Go from NHN Cloud's Cloud Functions service.

## Template information
| Item              | Value                                        |
|-----------------|------------------------------------------|
| **Supported version**       | 1.22 (scheduled for deprecation), 1.23 (scheduled for deprecation), 1.24, 1.25 |
| **File name**    | functions.go                               |
| **Entry Point** | Handler                                  |

## Basic template

### Hello World example
A basic form of function.

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

### Context object
In Go functions, HTTP requests and responses are processed via `http.ResponseWriter` and `*http.Request`.

```go
package main

import (
	"encoding/json"
	"log"
	"net/http"
)

func Handler(w http.ResponseWriter, r *http.Request) {
	// HTTP request information
	method := r.Method
	headers := r.Header
	query := r.URL.Query()

	// Configure response data
	response := map[string]interface{}{
		"method":  method,
		"headers": headers,
		"query":   query,
		"path":    r.URL.Path,
	}

	// Configure JSON response
	w.Header().Set("Content-Type", "application/json")
	w.WriteHeader(http.StatusOK)

	if err := json.NewEncoder(w).Encode(response); err != nil {
		log.Printf("Error encoding response: %v", err)
	}
}
```

## Download and use template file

### Template download
You can download the Go template provided by Cloud Functions to develop a local environment.

**Template download link**: [go.zip](https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_2acdfabf4efe4efc8a04c00b348110c9/cdn_origin/prod_cloud_functions/templates/go/go.zip)

### Template file structure
The structure of the downloaded template file is as follows:

```
go.zip
├── functions.go     # Main function file
└── go.mod          # Dependency management file
```

#### functions.go
Include basic fake data generation functions.
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

### Local development process

#### 1. Unzip
```bash
# Unzip
unzip go.zip -d my-function

# Move to task directory
cd my-function
```

#### 2. Modify function codes
Modify `functions.go` file with the logic you want.

```go
// functions.go - Simple modification example
package main

import (
	"encoding/json"
	"net/http"
	"time"
)

func Handler(w http.ResponseWriter, r *http.Request) {
	// Import name from query parameter
	name := r.URL.Query().Get("name")
	if name == "" {
		name = "World"
	}

	// Import message from body if POST request
	var customMessage string
	if r.Method == "POST" {
		var data map[string]interface{}
		if err := json.NewDecoder(r.Body).Decode(&data); err == nil {
			if msg, ok := data["message"].(string); ok {
				customMessage = msg
			}
		}
	}

	// Configure response data
	response := map[string]interface{}{
		"greeting":  "Hello, " + name + "!",
		"message":   customMessage,
		"method":    r.Method,
		"timestamp": time.Now().Format(time.RFC3339),
	}

	// JSON response
	w.Header().Set("Content-Type", "application/json")
	w.WriteHeader(http.StatusOK)

	if err := json.NewEncoder(w).Encode(response); err != nil {
		http.Error(w, "Internal Server Error", http.StatusInternalServerError)
	}
}
```

#### 3. Compress into a ZIP file
Compress the modified source code into ZIP file.

```bash
# Windows (PowerShell)
Compress-Archive -Path .\functions.go, .\go.mod -DestinationPath my-function.zip

# Windows (when using 7-Zip)
7z a my-function.zip functions.go go.mod

# macOS/Linux
zip my-function.zip functions.go go.mod

# Include all files (if any additional files are available)
zip -r my-function.zip . -x "*.git*" "go.sum" "test.go"
```

### Upload from Cloud Functions console
> Used when uploading files from the user's local environment when creating or modifying a function. (refer to Console Guide)

### Cautions for upload

#### ZIP file structure
- The `.go` file and `go.mod` must be located directly in the root of the ZIP file.
- We recommend avoiding unnecessary folder structures.

**Right structure:**
```
my-function.zip
├── functions.go
├── go.mod
└── utils.go (if there are additional files)
```

**Wrong structure:**
```
my-function.zip
└── my-function/
    ├── functions.go
    └── go.mod
```

#### File size limit
- ZIP file size is limited to 100MiB.
- `go.sum` file should not be included.

#### Files to exclude
```bash
# Similar to .gitignore, exclude the following files:
zip -r my-function.zip . -x \
  "go.sum" \
  ".git/*" \
  "*.log" \
  "test.go" \
  ".env" \
  "*.zip"
```

## Process by HTTP method

### Process GET request
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

	// Import query parameter
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

### Process POST request
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

	// Read request body
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

	// Validate required fields
	if requestBody.Name == "" {
		http.Error(w, "Missing required field: name", http.StatusBadRequest)
		return
	}

	// Generate response data
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
	// Generate simple ID (use UUID, etc. in reality)
	return time.Now().Format("20060102150405")
}

func getOrDefault(value, defaultValue string) string {
	if value == "" {
		return defaultValue
	}
	return value
}
```

## Manage packages

### Write go.mod
Write `go.mod` to manage dependencies.

```go
module example.com/myfunction

go 1.25

require (
    github.com/google/uuid v1.6.0
    github.com/gorilla/mux v1.8.1
)
```

### Example of external API call
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

	// External API call
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

	// Response configuration
	response := map[string]interface{}{
		"user":       userData,
		"fetched_at": time.Now().Format(time.RFC3339),
	}

	w.Header().Set("Content-Type", "application/json")
	json.NewEncoder(w).Encode(response)
}
```

### Data processing example
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

	// Data processing
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

	// Data sorting
	sort.Slice(processedData, func(i, j int) bool {
		return processedData[i].NormalizedName < processedData[j].NormalizedName
	})

	// Filter only vaild data
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

## Configure Entry Point

### Single function
Use the function name as the Entry Point.

Function name: `Handler`
Entry Point: `Handler`

### Multiple functions
You can define multiple functions in a single file.

```go
// handlers.go
package main

import (
	"encoding/json"
	"net/http"
)

func GetUser(w http.ResponseWriter, r *http.Request) {
	// User lookup logic
	response := map[string]string{"message": "Get user"}
	w.Header().Set("Content-Type", "application/json")
	json.NewEncoder(w).Encode(response)
}

func CreateUser(w http.ResponseWriter, r *http.Request) {
	// User creation logic
	response := map[string]string{"message": "User created"}
	w.Header().Set("Content-Type", "application/json")
	w.WriteHeader(http.StatusCreated)
	json.NewEncoder(w).Encode(response)
}

func UpdateUser(w http.ResponseWriter, r *http.Request) {
	// User modification logic
	response := map[string]string{"message": "User updated"}
	w.Header().Set("Content-Type", "application/json")
	json.NewEncoder(w).Encode(response)
}
```

Entry Point configuration:
- `GetUser`
- `CreateUser`
- `UpdateUser`

## Caution

### Go version
Currently, supported Go versions are **1.24** and **1.25**.

**Deprecation Planned**: Go 1.22 and 1.23 are deprecated, and their use will be restricted.

#### Go Version Support Policy
Cloud Functions follows Go's official release policy. Go releases new major versions twice a year (January and July), and each version is supported only up to the latest two major versions.

**Example**:
- Go 1.22 will no longer be supported when Go 1.24 is released.
- Go 1.23 will no longer be supported when Go 1.25 is released.

**Recommendation**: use the latest supported version (1.24 or 1.25) to continue receiving security patches and bug fixes.