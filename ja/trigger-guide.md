## Compute > Cloud Functions > トリガーガイド

このドキュメントでは、Cloud Functionsで提供するトリガーの種類と設定方法について説明します。

## トリガーの概要

トリガーは関数を実行させるイベントソースです。Cloud Functionsは様々な種類のトリガーを提供しており、複数の方法で関数を呼び出すことができます。

### サポートするトリガーの種類

| トリガーの種類      | 説明                      | デフォルト提供 |
|-------------|------------------------|-------|
| HTTP        | HTTPリクエストを通じて関数を実行       | O      |
| Timer       | 指定された時間または周期に従って関数を実行 | X      |
| API Gateway | API Gatewayを通じて関数を実行  | X      |

## HTTPトリガー

HTTPトリガーは関数の作成時にデフォルトで提供され、HTTPリクエストを通じて関数を実行できます。

### 特徴

- 関数の作成時に自動的に生成されます。
- 削除することはできず、有効化/無効化のみ可能です。
- GET, POSTメソッドをサポートします。

### HTTPトリガーURL形式

```
https://{userdomain}/{関数名}
```

### 使用例

#### GETリクエスト

```bash
curl -X GET "https://{userdomain}/{関数名}?param1=value1&param2=value2"
```

#### POSTリクエスト

```bash
curl -X POST "https://{userdomain}/{関数名}" \
  -H "Content-Type: application/json" \
  -d '{"key": "value"}'
```

### HTTPトリガーの有効化/無効化

1. Cloud Functionsコンソールで関数を選択します。
2. **トリガー**タブに移動します。
3. HTTPトリガーの有効化/無効化トグルをクリックします。

> **[参考]**
> <br>無効化されたHTTPトリガーにリクエストを送信しても、関数は実行されません。

## Timerトリガー

Timerトリガーは、指定された時間または周期に従って自動的に関数を実行します。Cron式を使用して実行周期を設定できます。

### Timerトリガーの作成

![trigger-guide-05](https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_2acdfabf4efe4efc8a04c00b348110c9/cdn_origin/prod_cloud_functions/2025-11-25/trigger-guide-01.png)

1. Cloud Functionsコンソールで関数を選択します。
2. **トリガー**タブに移動します。
3. **トリガー作成**をクリックします。
4. トリガーの種類で**Timer**を選択します。
5. **Value**にCron式を入力します。
6. **作成**をクリックします。

### Cron式の形式

Cron式は次のような形式に従います。

```
* * * * * *
│ │ │ │ │ │
│ │ │ │ │ └─ 曜日 (0-6 または SUN-SAT、0: 日曜日)
│ │ │ │ └─── 月 (1-12 または JAN-DEC)
│ │ │ └───── 日 (1-31)
│ │ └─────── 時 (0-23)
│ └───────── 分 (0-59)
└─────────── 秒 (0-59)
```

#### Cron式のフィールド

| フィールド              | 必須  | 許容値             | 許容される記号  |
|-----------------|-----|-----------------|-----------|
| 秒(Seconds)       | Yes | 0-59             | * / , -    |
| 分(Minutes)       | Yes | 0-59             | * / , -    |
| 時(Hours)         | Yes | 0-23             | * / , -    |
| 日(Day of month) | Yes | 1-31             | * / , - ? |
| 月(Month)         | Yes | 1-12 または JAN-DEC | * / , -    |
| 曜日(Day of week) | Yes | 0-6 または SUN-SAT  | * / , - ? |

#### 記号の説明

`*`

そのフィールドの全ての値と一致することを示します。例えば、5番目のフィールド(月)に`*`を使用すると、全ての月を意味します。

`/`

範囲の増分を表す際に使用します。例えば、2番目のフィールド(分)に`3-59/15`を使用すると、3分から始まって15分ごとに実行されることを意味します(3、18、33、48分)。`*/...`形式は`first-last/...`形式と同じであり、そのフィールドの最大範囲に対する増分を意味します。`N/...`形式は`N-MAX/...`を意味し、Nから始まってその範囲の終わりまで増分を使用します。範囲を超えた場合、最初に戻ることはありません。

`,`

リストの項目を区切る際に使用します。例えば、6番目のフィールド(曜日)に`MON,WED,FRI`を使用すると、月曜日、水曜日、金曜日を意味します。

`-`

範囲を定義する際に使用します。例えば、3番目のフィールド(時)に`9-17`を使用すると、午前9時から午後5時まで(含む)の全ての時間を意味します。

`?`

日(Day of month)または曜日(Day of week)フィールドを空けておくために、`*`の代わりに使用できます。

### Cron式の例

| Cron式              | 説明                             |
|------------------------|---------------------------------|
| `0 * * * * *`          | 毎分実行(0秒に)                   |
| `0 0 * * * *`          | 毎時0分に実行                       |
| `0 0 0 * * *`          | 毎日午前0時に実行                        |
| `0 0 9 * * MON`        | 毎週月曜日の午前9時に実行                 |
| `0 0 0 1 * *`          | 毎月1日の午前0時に実行                     |
| `0 */5 * * * *`        | 5分ごとに実行                          |
| `0 0 9-18 * * MON-FRI` | 平日(月-金)の午前9時から午後6時まで毎時実行 |
| `30 0 12 * * *`        | 毎日午後12時0分30秒に実行             |
| `0 0 0 1 JAN *`        | 毎年1月1日の午前0時に実行                  |

### Timerトリガーの修正

![trigger-guide-02](https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_2acdfabf4efe4efc8a04c00b348110c9/cdn_origin/prod_cloud_functions/2025-11-25/trigger-guide-02.png)

1. Cloud Functionsコンソールで関数を選択します。
2. **トリガー**タブに移動します。
3. 修正するTimerトリガーを選択します。
4. **トリガー修正**をクリックします。
5. **Value**のCron式を修正します。
6. **修正**をクリックします。

> **[参考]**
> <br>月(Month)と曜日(Day of week)フィールドの値は、大文字と小文字を区別しません。"SUN"、"Sun"、"sun"は全て同じものとして認識されます。

## API Gatewayトリガー

API Gatewayトリガーは、同一プロジェクトのAPI Gatewayサービスを活用して関数を実行できます。API Gatewayを通じて、よりきめ細かいAPI管理と制御が可能になります。

### 事前要件

- 同一プロジェクトでAPI Gatewayサービスが有効になっている必要があります。
  - 有効になっていない場合、トリガー作成時に有効化できます。

### API Gatewayトリガーの作成

![trigger-guide-04](https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_2acdfabf4efe4efc8a04c00b348110c9/cdn_origin/prod_cloud_functions/2025-11-25/trigger-guide-04.png)

1. Cloud Functionsコンソールで関数を選択します。
2. **トリガー**タブに移動します。
3. **トリガー作成**をクリックします。
4. トリガーの種類でAPI Gatewayを選択します。
5. 使用するパスを入力します。
   * 重複するパスは使用できません。
6. **作成**をクリックします。

### API GatewayトリガーURL形式

```
https://{stageurl}/{パス}
```

### 使用例

#### GETリクエスト

```bash
curl -X GET "https://{stageurl}/{パス}?param1=value1&param2=value2"
```

#### POSTリクエスト

```bash
curl -X POST "https://{stageurl}/{パス}" \
  -H "Content-Type: application/json" \
  -d '{"key": "value"}'
```

### API Gatewayトリガーの特徴

- API Gatewayの様々な機能を活用できます。
  - 認証及び権限管理
  - リクエスト/レスポンス変換
  - 使用量プラン及びAPIキー管理
  - リクエスト制限
- API GatewayコンソールでAPI設定を管理できます。
- 自動的に生成された1つのサービスでのみ関数連携が可能です。
- HTTPトリガーと同様にGET,POSTメソッドをサポートします。

### API Gatewayトリガーの修正

![trigger-guide-05](https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_2acdfabf4efe4efc8a04c00b348110c9/cdn_origin/prod_cloud_functions/2025-11-25/trigger-guide-05.png)

1. Cloud Functionsコンソールで関数を選択します。
2. **トリガー**タブに移動します。
3. 修正するAPI Gatewayトリガーを選択します。
4. **トリガー修正**をクリックします。
5. パスを変更します。
6. **修正**をクリックします。

> **[参考]**
> <br>API Gatewayトリガーの修正は、API Gatewayサービス内で既存のトリガーを削除した後、新しく作成する方式で行われます。
> <br>これにより、そのリソースに適用されたプラグインが全て削除されます。
> <br>プラグインが適用されたリソースパスを変更する場合は、修正の代わりに新しいトリガーを追加することを推奨します。

### API Gatewayと併用する

API Gatewayトリガーを作成すると、API Gatewayコンソールで追加設定を行えます。

詳細は[API Gatewayガイド](https://docs.nhncloud.com/ko/Application%20Service/API%20Gateway/ko/overview/)を参照してください。

## トリガーの削除

### トリガーの削除方法

![trigger-guide-03](https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_2acdfabf4efe4efc8a04c00b348110c9/cdn_origin/prod_cloud_functions/2025-11-25/trigger-guide-03.png)

1. Cloud Functionsコンソールで関数を選択します。
2. **トリガー**タブに移動します。
3. 削除するトリガーを選択します。(複数選択可能)
4. **トリガー削除**をクリックします。
5. **トリガー削除**モーダルウィンドウで**削除**をクリックします。

### 注意事項

- HTTPトリガーは削除できません。HTTPトリガーは基本トリガーとして提供され、有効化/無効化のみ可能です。
- トリガーを削除すると、そのイベントを通じて関数を実行できなくなります。
- 削除されたトリガーは復元できません。

## トリガーの制限事項

### API Gatewayトリガーの制限事項

- 同一プロジェクト内のAPI Gatewayのみ接続できます。
- 1つのAPI Gatewayリソースには、1つの関数のみ接続できます。
- 1つの関数には、複数のAPI Gatewayトリガーを登録できます。
- API Gatewayサービスが無効化されたり、接続されたリソースが削除または変更されたりすると、トリガーも動作しません。
  - この場合、トリガーを修正できないため、削除してから再登録する必要があります。
