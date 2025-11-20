## Compute > Cloud Functions > 트리거 가이드

이 문서는 Cloud Functions에서 제공하는 트리거 유형과 설정 방법을 설명합니다.

## 트리거 개요

트리거는 함수를 실행시키는 이벤트 소스입니다. Cloud Functions는 다양한 유형의 트리거를 제공하여 여러 방식으로 함수를 호출할 수 있습니다.

### 지원 트리거 유형

| 트리거 유형      | 설명                     | 기본 제공 |
|-------------|------------------------|-------|
| HTTP        | HTTP 요청을 통해 함수 실행      | O     |
| Timer       | 지정된 시간 또는 주기에 따라 함수 실행 | X     |
| API Gateway | API Gateway를 통해 함수 실행  | X     |

## HTTP 트리거

HTTP 트리거는 함수 생성 시 기본으로 제공되며, HTTP 요청을 통해 함수를 실행할 수 있습니다.

### 특징

- 함수 생성 시 자동으로 생성됩니다.
- 삭제할 수 없으며, 활성화/비활성화만 가능합니다.
- GET, POST 메서드를 지원합니다.

### HTTP 트리거 URL 형식

```
https://{userdomain}/{함수명}
```

### 사용 예시

#### GET 요청

```bash
curl -X GET "https://{userdomain}/{함수명}?param1=value1&param2=value2"
```

#### POST 요청

```bash
curl -X POST "https://{userdomain}/{함수명}" \
  -H "Content-Type: application/json" \
  -d '{"key": "value"}'
```

### HTTP 트리거 활성화/비활성화

1. Cloud Functions 콘솔에서 함수를 선택합니다.
2. **트리거** 탭으로 이동합니다.
3. HTTP 트리거의 활성화/비활성화 토글을 클릭합니다.

> **[참고]**
> <br>비활성화된 HTTP 트리거로 요청을 보내면 함수가 실행되지 않습니다.

## Timer 트리거

Timer 트리거는 지정된 시간 또는 주기에 따라 자동으로 함수를 실행합니다. Cron 표현식을 사용하여 실행 주기를 설정할 수 있습니다.

### Timer 트리거 생성

![trigger-guide-05](https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_2acdfabf4efe4efc8a04c00b348110c9/cdn_origin/prod_cloud_functions/2025-11-25/trigger-guide-01.png)

1. Cloud Functions 콘솔에서 함수를 선택합니다.
2. **트리거** 탭으로 이동합니다.
3. **트리거 생성**을 클릭합니다.
4. 트리거의 유형을 **Timer**로 선택합니다.
5. **Value**에 Cron 표현식을 입력합니다.
6. 생성 버튼을 클릭합니다.

### Cron 표현식 형식

Cron 표현식은 다음과 같은 형식을 따릅니다:

```
* * * * * *
│ │ │ │ │ │
│ │ │ │ │ └─ 요일 (0-6 또는 SUN-SAT, 0: 일요일)
│ │ │ │ └─── 월 (1-12 또는 JAN-DEC)
│ │ │ └───── 일 (1-31)
│ │ └─────── 시 (0-23)
│ └───────── 분 (0-59)
└─────────── 초 (0-59)
```

#### Cron 표현식 필드

| 필드              | 필수  | 허용 값            | 허용 특수 문자  |
|-----------------|-----|-----------------|-----------|
| 초(Seconds)      | Yes | 0-59            | * / , -   |
| 분(Minutes)      | Yes | 0-59            | * / , -   |
| 시(Hours)        | Yes | 0-23            | * / , -   |
| 일(Day of month) | Yes | 1-31            | * / , - ? |
| 월(Month)        | Yes | 1-12 또는 JAN-DEC | * / , -   |
| 요일(Day of week) | Yes | 0-6 또는 SUN-SAT  | * / , - ? |

#### 특수 문자 설명

`*`

해당 필드의 모든 값과 일치함을 나타냅니다. 예를 들어, 5번째 필드(월)에 `*`를 사용하면 모든 월을 의미합니다.

`/`

범위의 증분을 나타낼 때 사용합니다. 예를 들어, 2번째 필드(분)에 `3-59/15`를 사용하면 3분부터 시작해서 15분마다 실행됨을 의미합니다(3, 18, 33, 48분). `*/...` 형태는 `first-last/...` 형태와 동일하며, 해당 필드의 가장 큰 범위에 대한 증분을 의미합니다. `N/...` 형태는 `N-MAX/...`를 의미하며, N부터 시작해서 해당 범위의 끝까지 증분을 사용합니다. 범위를 넘어가면 다시 처음으로 돌아가지 않습니다.

`,`

목록의 항목을 구분할 때 사용합니다. 예를 들어, 6번째 필드(요일)에 `MON,WED,FRI`를 사용하면 월요일, 수요일, 금요일을 의미합니다.

`-`

범위를 정의할 때 사용합니다. 예를 들어, 3번째 필드(시)에 `9-17`을 사용하면 오전 9시부터 오후 5시까지(포함)의 모든 시간을 의미합니다.

`?`

일(Day of month) 또는 요일(Day of week) 필드를 비워두기 위해 `*` 대신 사용할 수 있습니다.

### Cron 표현식 예시

| Cron 표현식               | 설명                              |
|------------------------|---------------------------------|
| `0 * * * * *`          | 매 분마다 실행 (0초에)                  |
| `0 0 * * * *`          | 매 시간 정각에 실행                     |
| `0 0 0 * * *`          | 매일 자정에 실행                       |
| `0 0 9 * * MON`        | 매주 월요일 오전 9시에 실행                |
| `0 0 0 1 * *`          | 매월 1일 자정에 실행                    |
| `0 */5 * * * *`        | 5분마다 실행                         |
| `0 0 9-18 * * MON-FRI` | 평일(월-금) 오전 9시부터 오후 6시까지 매 시간 실행 |
| `30 0 12 * * *`        | 매일 오후 12시 0분 30초에 실행            |
| `0 0 0 1 JAN *`        | 매년 1월 1일 자정에 실행                 |

### Timer 트리거 수정

![trigger-guide-02](https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_2acdfabf4efe4efc8a04c00b348110c9/cdn_origin/prod_cloud_functions/2025-11-25/trigger-guide-02.png)

1. Cloud Functions 콘솔에서 함수를 선택합니다.
2. 트리거 탭으로 이동합니다.
3. 수정할 Timer 트리거를 선택합니다.
4. 트리거 수정 버튼을 클릭합니다.
5. **Cron 표현식**을 수정합니다.
6. 수정 버튼을 클릭합니다.

> **[참고]**
> <br>월(Month)과 요일(Day of week) 필드 값은 대소문자를 구분하지 않습니다. "SUN", "Sun", "sun"은 모두 동일하게 인식됩니다.

## API Gateway 트리거

API Gateway 트리거는 동일 프로젝트의 API Gateway 서비스를 활용하여 함수를 실행할 수 있습니다. API Gateway를 통해 더욱 세밀한 API 관리와 제어가 가능합니다.

### 사전 요구사항

- 동일 프로젝트에 API Gateway 서비스가 활성화되어 있어야 합니다.
  - 활성화 되어 있지 않는 경우 트리거 생성시 활성화 가능합니다.

### API Gateway 트리거 생성

![trigger-guide-04](https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_2acdfabf4efe4efc8a04c00b348110c9/cdn_origin/prod_cloud_functions/2025-11-25/trigger-guide-04.png)

1. Cloud Functions 콘솔에서 함수를 선택합니다.
2. 트리거 탭으로 이동합니다.
3. 트리거 생성 버튼을 클릭합니다.
4. 트리거 유형에서 API Gateway를 선택합니다.
5. 사용할 경로를 입력합니다.
   * 중복된 경로는 사용 불가능합니다.
6. 생성 버튼을 클릭합니다.

### API Gateway 트리거 URL 형식

```
https://{stageurl}/{경로}
```

### 사용 예시

#### GET 요청

```bash
curl -X GET "https://{stageurl}/{경로}?param1=value1&param2=value2"
```

#### POST 요청

```bash
curl -X POST "https://{stageurl}/{경로}" \
  -H "Content-Type: application/json" \
  -d '{"key": "value"}'
```

### API Gateway 트리거 특징

- API Gateway의 다양한 기능을 활용할 수 있습니다.
  - 인증 및 권한 관리
  - 요청/응답 변환
  - 사용량 계획 및 API 키 관리
  - 요청 제한
- API Gateway 콘솔에서 API 설정을 관리할 수 있습니다.
- 자동으로 생성된 하나의 서비스에서만 함수 연동이 가능합니다.
- HTTP 트리거와 동일하게 GET, POST 메서드를 지원합니다.

### API Gateway 트리거 수정

![trigger-guide-05](https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_2acdfabf4efe4efc8a04c00b348110c9/cdn_origin/prod_cloud_functions/2025-11-25/trigger-guide-05.png)

1. Cloud Functions 콘솔에서 함수를 선택합니다.
2. 트리거 탭으로 이동합니다.
3. 수정할 API Gateway 트리거를 선택합니다.
4. 트리거 수정 버튼을 클릭합니다.
5. 경로를 변경합니다.
6. 수정 버튼을 클릭합니다.

### API Gateway와 함께 사용하기

API Gateway 트리거를 생성하면, API Gateway 콘솔에서 추가 설정을 할 수 있습니다:

자세한 내용은 [API Gateway 가이드](https://docs.nhncloud.com/ko/Application%20Service/API%20Gateway/ko/overview/)를 참고하십시오.

## 트리거 삭제

### 트리거 삭제 방법

![trigger-guide-03](https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_2acdfabf4efe4efc8a04c00b348110c9/cdn_origin/prod_cloud_functions/2025-11-25/trigger-guide-03.png)

1. Cloud Functions 콘솔에서 함수를 선택합니다.
2. 트리거 탭으로 이동합니다.
3. 삭제할 트리거를 선택합니다. (여러 개 선택 가능)
4. 트리거 삭제 버튼을 클릭합니다.
5. 확인 대화상자에서 삭제 버튼을 클릭합니다.

### 주의사항

- **HTTP 트리거는 삭제할 수 없습니다.** HTTP 트리거는 기본 트리거로 제공되며, 활성화/비활성화만 가능합니다.
- 트리거를 삭제하면 해당 이벤트를 통해 함수를 실행할 수 없습니다.
- 삭제된 트리거는 복구할 수 없습니다.

## 트리거 제한사항

### API Gateway 트리거 제한사항

- 동일 프로젝트 내의 API Gateway만 연결할 수 있습니다.
- 하나의 API Gateway Resource는 하나의 함수만 연결할 수 있습니다.
- 하나의 함수에는 여러 API Gateway 트리거를 등록할 수 있습니다.
- API Gateway 서비스가 비활성화되거나 연결된 리소스가 삭제 또는 변경되면 트리거도 동작하지 않습니다.
  - 이 경우 트리거를 수정할 수 없으므로 삭제한 후 다시 등록해야 합니다. 