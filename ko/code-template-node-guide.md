## Compute > Cloud Functions > 코드 템플릿 가이드 > Node.js

이 문서는 NHN Cloud의 Cloud Functions 서비스에서 Node.js를 사용하여 함수를 개발하는 방법을 상세히 설명합니다.

## 템플릿 정보
| 항목              | 값               |
|-----------------|-----------------|
| **지원 버전**       | 20.16.0, 22.5.0 |
| **파일명**         | hello.js        |
| **Entry Point** | hello           |

## 기본 템플릿
### Hello World 예시
가장 기본적인 함수 형태입니다.

```javascript
module.exports = async (context) => {
    return {
        status: 200,
        body: "Hello, World!\n"
    };
}
```

### Context 객체
함수에 전달되는 `context` 객체는 다음과 같은 정보를 포함합니다.

```javascript
module.exports = async (context) => {
    // HTTP 요청 정보
    console.log('Method:', context.request.method);
    console.log('Headers:', context.request.headers);
    console.log('Query:', context.request.query);
    console.log('Body:', context.request.body);

    return {
        status: 200,
        body: JSON.stringify({
            message: "Context information logged"
        })
    };
}
```

## 템플릿 파일 다운로드 및 활용

### 템플릿 다운로드
Cloud Functions에서 제공하는 Node.js 템플릿을 다운로드하여 로컬 환경에서 개발할 수 있습니다.

**템플릿 다운로드 링크**: [nodejs.zip](https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_2acdfabf4efe4efc8a04c00b348110c9/cdn_origin/prod_cloud_functions/templates/nodejs/nodejs.zip)

### 템플릿 파일 구조
다운로드한 템플릿 파일의 구조는 다음과 같습니다.

```
nodejs.zip
├── hello.js          # 메인 함수 파일
└── package.json      # 의존성 관리 파일
```

#### hello.js
기본 Hello World 함수가 포함되어 있습니다.
```javascript
module.exports = async (context) => {
    return {
        status: 200,
        body: "hello, world!\n"
    };
}
```

#### package.json
```json
{}
```

### 로컬 개발 과정

#### 1. 압축 해제
```bash
# 압축 해제
unzip nodejs.zip -d my-function

# 작업 디렉터리 이동
cd my-function
```

#### 2. 함수 코드 수정
`hello.js` 파일을 원하는 로직으로 수정합니다.

```javascript
// hello.js - 간단한 수정 예시
module.exports = async (context) => {
    try {
        // 쿼리 파라미터에서 이름 가져오기
        const { name = 'World' } = context.request.query;

        // POST 요청인 경우 body에서 메시지 가져오기
        let customMessage = '';
        if (context.request.method === 'POST' && context.request.body) {
            const body = context.request.body;
            customMessage = body.message || '';
        }

        return {
            status: 200,
            headers: {
                'Content-Type': 'application/json'
            },
            body: JSON.stringify({
                greeting: `Hello, ${name}!`,
                message: customMessage,
                method: context.request.method,
                timestamp: new Date().toISOString()
            })
        };

    } catch (error) {
        return {
            status: 500,
            body: JSON.stringify({
                error: 'Internal Server Error',
                details: error.message
            })
        };
    }
}
```

#### 3. ZIP 파일로 압축
수정된 코드를 다시 ZIP 파일로 압축합니다.

```bash
# Windows (PowerShell)
Compress-Archive -Path .\hello.js, .\package.json -DestinationPath my-function.zip

# Windows (7-Zip 사용 시)
7z a my-function.zip hello.js package.json

# macOS/Linux
zip my-function.zip hello.js package.json

# 모든 파일 포함(추가 파일이 있는 경우)
zip -r my-function.zip . -x "*.git*" "node_modules/*" "test.js"
```

### Cloud Functions 콘솔에서 업로드
> 함수를 생성하거나 수정할 때 사용자 로컬 환경의 파일을 업로드 시 사용. (콘솔 사용 가이드 참고)

### 업로드 시 주의사항

#### ZIP 파일 구조
- ZIP 파일의 루트에 직접 `.js` 파일과 `package.json`이 위치해야 합니다.
- 불필요한 폴더 구조는 피할 것을 권장합니다.

**올바른 구조:**
```
my-function.zip
├── hello.js
├── package.json
└── utils.js (추가 파일이 있는 경우)
```

**잘못된 구조:**
```
my-function.zip
└── my-function/
    ├── hello.js
    └── package.json
```

#### 파일 크기 제한
- ZIP 파일 크기는 100MiB 이하로 제한됩니다.
- `node_modules` 폴더는 포함하지 마세요. (의존성은 `package.json`으로 관리)

#### 제외할 파일들
```bash
# .gitignore와 유사하게 다음 파일들은 제외
zip -r my-function.zip . -x \
  "node_modules/*" \
  ".git/*" \
  "*.log" \
  "test.js" \
  ".env" \
  "*.zip"
```

## HTTP 메서드별 처리

### GET 요청 처리
```javascript
module.exports = async (context) => {
    if (context.request.method !== 'GET') {
        return {
            status: 405,
            body: JSON.stringify({ error: 'Method Not Allowed' })
        };
    }

    // 쿼리 파라미터 가져오기
    const { name = 'World', greeting = 'Hello' } = context.request.query;

    return {
        status: 200,
        headers: {
            'Content-Type': 'application/json'
        },
        body: JSON.stringify({
            message: `${greeting}, ${name}!`,
            timestamp: new Date().toISOString()
        })
    };
}
```

### POST 요청 처리
```javascript
module.exports = async (context) => {
    if (context.request.method !== 'POST') {
        return {
            status: 405,
            body: JSON.stringify({ error: 'Method Not Allowed' })
        };
    }

    // JSON 형태의 request body 사용
    const requestBody = context.request.body;

    // 필수 필드 검증
    if (!requestBody.name) {
        return {
            status: 400,
            body: JSON.stringify({
                error: 'Missing required field: name'
            })
        };
    }

    const { name, email, message } = requestBody;

    // 처리 로직
    const response = {
        id: Math.random().toString(36).substr(2, 9),
        name: name,
        email: email || 'not provided',
        message: message || 'No message',
        processed_at: new Date().toISOString()
    };

    return {
        status: 201,
        headers: {
            'Content-Type': 'application/json'
        },
        body: JSON.stringify(response)
    };

}
```

## 패키지 관리

### package.json 작성
의존성 관리를 위해 `package.json` 파일을 작성합니다.

```json
{
  "name": "my-cloud-function",
  "version": "1.0.0",
  "description": "NHN Cloud Functions example",
  "main": "index.js",
  "dependencies": {
    "axios": "^1.6.0",
    "lodash": "^4.17.21",
    "moment": "^2.29.4",
    "uuid": "^9.0.1"
  }
}
```

### 외부 API 호출 예시
```javascript
const axios = require('axios');

module.exports = async (context) => {
    try {
        const { userId } = context.request.query;

        if (!userId) {
            return {
                status: 400,
                body: JSON.stringify({ error: 'userId is required' })
            };
        }

        // 외부 API 호출
        const response = await axios.get(`https://jsonplaceholder.typicode.com/users/${userId}`);

        return {
            status: 200,
            headers: {
                'Content-Type': 'application/json'
            },
            body: JSON.stringify({
                user: response.data,
                fetched_at: new Date().toISOString()
            })
        };

    } catch (error) {
        console.error('API call failed:', error.message);

        if (error.response && error.response.status === 404) {
            return {
                status: 404,
                body: JSON.stringify({ error: 'User not found' })
            };
        }

        return {
            status: 500,
            body: JSON.stringify({
                error: 'Internal server error',
                details: error.message
            })
        };
    }
}
```

### 데이터 처리 예시
```javascript
const _ = require('lodash');
const moment = require('moment');
const { v4: uuidv4 } = require('uuid');

module.exports = async (context) => {
    try {
        const requestBody = context.request.body;
        const { data } = requestBody;

        if (!Array.isArray(data)) {
            return {
                status: 400,
                body: JSON.stringify({ error: 'Data must be an array' })
            };
        }

        // 데이터 처리
        const processedData = data.map(item => ({
            id: uuidv4(),
            ...item,
            processed_at: moment().format('YYYY-MM-DD HH:mm:ss'),
            normalized_name: _.capitalize(_.trim(item.name))
        }));

        // 데이터 정렬 및 필터링
        const sortedData = _.orderBy(processedData, ['normalized_name'], ['asc']);
        const validData = sortedData.filter(item => item.normalized_name);

        return {
            status: 200,
            headers: {
                'Content-Type': 'application/json'
            },
            body: JSON.stringify({
                total_processed: processedData.length,
                valid_items: validData.length,
                data: validData
            })
        };

    } catch (error) {
        return {
            status: 400,
            body: JSON.stringify({
                error: 'Data processing failed',
                details: error.message
            })
        };
    }
}
```

## Entry Point 설정

### 단일 함수
파일명을 Entry Point로 사용합니다.

파일명: `hello.js`
Entry Point: `hello`

### 다중 함수
하나의 파일에서 여러 함수를 내보낼 수 있습니다.

```javascript
// users.js
module.exports.getUser = async (context) => {
    // 사용자 조회 로직
    return {
        status: 200,
        body: JSON.stringify({ message: "Get user" })
    };
}

module.exports.createUser = async (context) => {
    // 사용자 생성 로직
    return {
        status: 201,
        body: JSON.stringify({ message: "User created" })
    };
}

module.exports.updateUser = async (context) => {
    // 사용자 수정 로직
    return {
        status: 200,
        body: JSON.stringify({ message: "User updated" })
    };
}
```

Entry Point 설정:
- `users.getUser`
- `users.createUser`
- `users.updateUser`

### Entry Point 제한 사항
- **하위 디렉터리 지정 불가**: 루트 디렉터리의 하위 디렉터리에 있는 파일은 Entry Point로 지정할 수 없습니다.

**올바른 Entry Point:**
```
hello.js → hello
users.js → users.getUser
```

**잘못된 Entry Point:**
```
lib/utils.js → lib.utils ❌
src/handlers.js → src.handlers ❌
modules/auth.js → modules.auth ❌
```

모든 함수 파일은 ZIP 파일의 루트 레벨에 위치해야 합니다.

## 주의사항

### CommonJS vs ES Modules
현재 Cloud Functions는 CommonJS 방식만 지원합니다.

**사용 가능(CommonJS):**
```javascript
const axios = require('axios');
module.exports = async (context) => {
    // 함수 로직
};
```

**사용 불가(ES Modules):**
```javascript
import axios from 'axios';  // ❌ 지원하지 않음
export default async (context) => {  // ❌ 지원하지 않음
    // 함수 로직
};
```

### 메모리 및 실행 시간 고려사항
- 함수는 제한된 메모리와 실행 시간 내에서 동작해야 합니다.
- 대용량 데이터 처리 시 스트림 처리를 고려하세요.
- 장시간 실행되는 작업은 적절히 분할하세요.
