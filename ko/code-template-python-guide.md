## Compute > Cloud Functions > 코드 템플릿 가이드 > Python

이 문서는 NHN Cloud의 Cloud Functions 서비스에서 Python을 사용하여 함수를 개발하는 방법을 상세히 설명합니다.

## 템플릿 정보
| 항목              | 값                |
|-----------------|------------------|
| **지원 버전**       | 3.11, 3.12, 3.13 |
| **파일명**         | user.py          |
| **Entry Point** | user.main        |

## 기본 템플릿

### Hello World 예시
가장 기본적인 함수 형태입니다.

```python
import sys
import yaml

document = """
  a: 1
  b:
    c: 3
    d: 4
"""

def main():
    return yaml.dump(yaml.safe_load(document))
```

### Context 객체
Python 함수에서는 Flask의 request 객체를 통해 HTTP 요청 정보에 접근할 수 있습니다.

```python
from flask import request
import json

def main():
    # HTTP 요청 정보
    method = request.method
    headers = dict(request.headers)
    args = request.args.to_dict()

    # 요청 본문(POST/PUT 등)
    data = None
    if request.is_json:
        data = request.get_json()
    elif request.data:
        data = request.data.decode('utf-8')

    return {
        'method': method,
        'headers': headers,
        'query_params': args,
        'body': data
    }
```

## 템플릿 파일 다운로드 및 활용

### 템플릿 다운로드
Cloud Functions에서 제공하는 Python 템플릿을 다운로드하여 로컬 환경에서 개발할 수 있습니다.

**템플릿 다운로드 링크**: [python.zip](https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_2acdfabf4efe4efc8a04c00b348110c9/cdn_origin/prod_cloud_functions/templates/python/python.zip)

### 템플릿 파일 구조
다운로드한 템플릿 파일의 구조는 다음과 같습니다.

```
python.zip
├── user.py          # 메인 함수 파일
└── requirements.txt # 의존성 관리 파일
```

#### user.py
기본 YAML 처리 함수가 포함되어 있습니다.
```python
import sys
import yaml

document = """
  a: 1
  b:
    c: 3
    d: 4
"""

def main():
    return yaml.dump(yaml.safe_load(document))
```

#### requirements.txt
```txt
pyyaml
```

### 로컬 개발 과정

#### 1. 압축 해제
```bash
# 압축 해제
unzip python.zip -d my-function

# 작업 디렉터리 이동
cd my-function
```

#### 2. 함수 코드 수정
`user.py` 파일을 원하는 로직으로 수정합니다.

```python
# user.py - 간단한 수정 예시
from flask import request
import json
from datetime import datetime

def main():
    try:
        # 쿼리 파라미터에서 이름 가져오기
        name = request.args.get('name', 'World')

        # POST 요청인 경우 body에서 메시지 가져오기
        custom_message = ''
        if request.method == 'POST':
            try:
                data = request.get_json()
                if data:
                    custom_message = data.get('message', '')
            except Exception:
                # JSON 파싱 실패 시 무시
                pass

        result = {
            'greeting': f'Hello, {name}!',
            'message': custom_message,
            'method': request.method,
            'timestamp': str(datetime.now())
        }

        return json.dumps(result, ensure_ascii=False)

    except Exception as e:
        return json.dumps({
            'error': 'Internal Server Error',
            'details': str(e)
        }, ensure_ascii=False)
```

#### 3. ZIP 파일로 압축
수정된 코드를 다시 ZIP 파일로 압축합니다.

```bash
# Windows (PowerShell)
Compress-Archive -Path .\user.py, .\requirements.txt -DestinationPath my-function.zip

# Windows (7-Zip 사용 시)
7z a my-function.zip user.py requirements.txt

# macOS/Linux
zip my-function.zip user.py requirements.txt

# 모든 파일 포함(추가 파일이 있는 경우)
zip -r my-function.zip . -x "*.git*" "__pycache__/*" "*.pyc" "test.py"
```

### Cloud Functions 콘솔에서 업로드
> 함수를 생성하거나 수정할 때 사용자 로컬 환경의 파일을 업로드 시 사용. (콘솔 사용 가이드 참고)

### 업로드 시 주의사항

#### ZIP 파일 구조
- ZIP 파일의 루트에 직접 `.py` 파일과 `requirements.txt`가 위치해야 합니다.
- 불필요한 폴더 구조는 피할 것을 권장합니다.

**올바른 구조:**
```
my-function.zip
├── user.py
├── requirements.txt
└── utils.py (추가 파일이 있는 경우)
```

**잘못된 구조:**
```
my-function.zip
└── my-function/
    ├── user.py
    └── requirements.txt
```

#### 파일 크기 제한
- ZIP 파일 크기는 100MiB 이하로 제한됩니다.
- `__pycache__` 폴더는 포함하지 마세요.

#### 제외할 파일들
```bash
# .gitignore와 유사하게 다음 파일들은 제외
zip -r my-function.zip . -x \
  "__pycache__/*" \
  "*.pyc" \
  ".git/*" \
  "*.log" \
  "test.py" \
  ".env" \
  "*.zip"
```

## HTTP 메서드별 처리

### GET 요청 처리
```python
from flask import request
import json
from datetime import datetime

def main():
    if request.method != 'GET':
        return json.dumps({'error': 'Method Not Allowed'}, ensure_ascii=False), 405

    # 쿼리 파라미터 가져오기
    name = request.args.get('name', 'World')
    greeting = request.args.get('greeting', 'Hello')

    result = {
        'message': f'{greeting}, {name}!',
        'timestamp': datetime.now().isoformat(),
        'method': 'GET'
    }

    return json.dumps(result, ensure_ascii=False)
```

### POST 요청 처리
```python
from flask import request
import json
from datetime import datetime
import uuid

def main():
    if request.method != 'POST':
        return json.dumps({'error': 'Method Not Allowed'}, ensure_ascii=False), 405

    try:
        # JSON 형태의 request body 파싱
        request_body = request.get_json()

        # 필수 필드 검증
        if not request_body or 'name' not in request_body:
            return json.dumps({
                'error': 'Missing required field: name'
            }, ensure_ascii=False), 400

        name = request_body['name']
        email = request_body.get('email', 'not provided')
        message = request_body.get('message', 'No message')

        # 처리 로직
        response = {
            'id': str(uuid.uuid4())[:8],
            'name': name,
            'email': email,
            'message': message,
            'processed_at': datetime.now().isoformat()
        }

        return json.dumps(response, ensure_ascii=False)

    except Exception as e:
        return json.dumps({
            'error': 'Invalid JSON format',
            'details': str(e)
        }, ensure_ascii=False), 400
```

## 패키지 관리

### requirements.txt 작성
의존성 관리를 위해 `requirements.txt` 파일을 작성합니다.

```txt
pyyaml
requests>=2.28.0
python-dateutil>=2.8.0
```

### 외부 API 호출 예시
```python
from flask import request
import json
import requests
from datetime import datetime

def main():
    try:
        user_id = request.args.get('userId')

        if not user_id:
            return json.dumps({'error': 'userId is required'}, ensure_ascii=False), 400

        # 외부 API 호출
        response = requests.get(f'https://jsonplaceholder.typicode.com/users/{user_id}')

        if response.status_code == 404:
            return json.dumps({'error': 'User not found'}, ensure_ascii=False), 404

        response.raise_for_status()
        user_data = response.json()

        result = {
            'user': user_data,
            'fetched_at': datetime.now().isoformat()
        }

        return json.dumps(result, ensure_ascii=False)

    except requests.RequestException as e:
        return json.dumps({
            'error': 'External API error',
            'details': str(e)
        }, ensure_ascii=False), 500
    except Exception as e:
        return json.dumps({
            'error': 'Internal server error',
            'details': str(e)
        }, ensure_ascii=False), 500
```

### 데이터 처리 예시
```python
from flask import request
import json
from datetime import datetime
import uuid
import re

def main():
    try:
        request_body = request.get_json()
        data = request_body.get('data', [])

        if not isinstance(data, list):
            return json.dumps({'error': 'Data must be an array'}, ensure_ascii=False), 400

        # 데이터 처리
        processed_data = []
        for item in data:
            if 'name' in item:
                processed_item = {
                    'id': str(uuid.uuid4())[:8],
                    **item,
                    'processed_at': datetime.now().isoformat(),
                    'normalized_name': normalize_name(item['name'])
                }
                processed_data.append(processed_item)

        # 데이터 정렬 및 필터링
        valid_data = [item for item in processed_data if item['normalized_name']]
        valid_data.sort(key=lambda x: x['normalized_name'])

        result = {
            'total_processed': len(processed_data),
            'valid_items': len(valid_data),
            'data': valid_data
        }

        return json.dumps(result, ensure_ascii=False)

    except Exception as e:
        return json.dumps({
            'error': 'Data processing failed',
            'details': str(e)
        }, ensure_ascii=False), 400

def normalize_name(name):
    """이름 정규화 함수"""
    if not name:
        return ''
    # 공백 제거 및 첫 글자 대문자화
    return name.strip().title()
```

## Entry Point 설정

### 단일 함수
`파일명.함수명`을 Entry Point로 사용합니다.

파일명: `user.py`
함수명: `main`
Entry Point: `user.main`

### 다중 함수
하나의 파일에서 여러 함수를 정의할 수 있습니다.

```python
# handlers.py
from flask import request
import json

def get_user():
    # 사용자 조회 로직
    return json.dumps({'message': 'Get user'}, ensure_ascii=False)

def create_user():
    # 사용자 생성 로직
    return json.dumps({'message': 'User created'}, ensure_ascii=False)

def update_user():
    # 사용자 수정 로직
    return json.dumps({'message': 'User updated'}, ensure_ascii=False)
```

Entry Point 설정:
- `handlers.get_user`
- `handlers.create_user`
- `handlers.update_user`

## 주의사항

### 지원하지 않는 패키지
현재 아래 특징이 있는 복잡한 패키지는 지원하지 않습니다.

**지원하지 않는 패키지 예시:**
- `numpy`, `pandas` (C/C++ 확장 모듈 필수)
- `scipy` (시스템 라이브러리 의존성)
- `tensorflow`, `pytorch` (복잡한 초기화 과정)

**지원 가능한 패키지 예시:**
- `requests` (HTTP 클라이언트)
- `pyyaml` (YAML 처리)
- `python-dateutil` (날짜/시간 처리)
- `pillow` (이미지 처리 - 기본 기능)

### 메모리 및 실행 시간 고려사항
- 함수는 제한된 메모리와 실행 시간 내에서 동작해야 합니다.
- 대용량 데이터 처리 시 제너레이터나 스트림 처리를 고려하세요.
- 장시간 실행되는 작업은 적절히 분할하세요.
