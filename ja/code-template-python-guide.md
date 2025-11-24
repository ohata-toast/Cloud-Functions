## Compute > Cloud Functions > コードテンプレートガイド > Python

このドキュメントでは、NHN CloudのCloud FunctionsサービスでPythonを使用して関数を開発する方法を詳しく説明します。

## テンプレート情報
| 項目             | 値               |
|-----------------|------------------|
| **サポートバージョン**       | 3.11, 3.12, 3.13 |
| **ファイル名**         | user.py          |
| **Entry Point** | user.main        |

## 基本テンプレート

### Hello Worldの例
最も基本的な関数の形式です。

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

### Contextオブジェクト
Pythonの関数では、Flaskのrequestオブジェクトを通じてHTTPリクエスト情報にアクセスできます。

```python
from flask import request
import json

def main():
    # HTTPリクエスト情報
    method = request.method
    headers = dict(request.headers)
    args = request.args.to_dict()

    # リクエストボディ(POST/PUTなど)
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

## テンプレートファイルのダウンロードと活用

### テンプレートのダウンロード
Cloud Functionsが提供するPythonテンプレートをダウンロードし、ローカル環境で開発できます。

**テンプレートのダウンロードリンク**: [python.zip](https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_2acdfabf4efe4efc8a04c00b348110c9/cdn_origin/prod_cloud_functions/templates/python/python.zip)

### テンプレートのファイル構造
ダウンロードしたテンプレートファイルの構造は以下の通りです:

```
python.zip
├── user.py          # メイン関数ファイル
└── requirements.txt # 依存関係管理ファイル
```

#### user.py
基本的なYAML処理関数が含まれています。
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

### ローカルでの開発プロセス

#### 1. 解凍
```bash
# 解凍
unzip python.zip -d my-function

# 作業ディレクトリへ移動
cd my-function
```

#### 2. 関数コードの修正
`user.py`ファイルを、目的のロジックに合わせて修正します。

```python
# user.py - 簡単な修正例
from flask import request
import json
from datetime import datetime

def main():
    try:
        # クエリパラメータから名前を取得
        name = request.args.get('name', 'World')

        # POSTリクエストの場合、bodyからメッセージを取得
        custom_message = ''
        if request.method == 'POST':
            try:
                data = request.get_json()
                if data:
                    custom_message = data.get('message', '')
            except Exception:
                # JSONのパースに失敗した場合は無視
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

#### 3. ZIPファイルへ圧縮
修正したコードを、再度ZIPファイルへ圧縮します。

```bash
# Windows (PowerShell)
Compress-Archive -Path .\user.py, .\requirements.txt -DestinationPath my-function.zip

# Windows (7-Zipを使用する場合)
7z a my-function.zip user.py requirements.txt

# macOS/Linux
zip my-function.zip user.py requirements.txt

# 全てのファイルを含める(追加ファイルがある場合)
zip -r my-function.zip . -x "*.git*" "__pycache__/*" "*.pyc" "test.py"
```

### Cloud Functionsコンソールでのアップロード
> 関数を作成または修正する際に、ユーザーのローカル環境にあるファイルをアップロードする場合に使用します。(コンソール利用ガイド参照)

### アップロード時の注意事項

#### ZIPファイルの構造
- ZIPファイルのルートに、直接`.py`ファイルと`requirements.txt`を配置する必要があります。
- 不要なフォルダ構造は避けることを推奨します。

**正しい構造:**
```
my-function.zip
├── user.py
├── requirements.txt
└── utils.py (追加ファイルがある場合)
```

**誤った構造:**
```
my-function.zip
└── my-function/
    ├── user.py
    └── requirements.txt
```

#### ファイルサイズの制限
- ZIPファイルのサイズは100MiB以下に制限されます。
- `__pycache__`フォルダは含めないでください。

#### 除外するファイル
```bash
# .gitignoreと同様に、以下のファイルは除外
zip -r my-function.zip . -x \
  "__pycache__/*" \
  "*.pyc" \
  ".git/*" \
  "*.log" \
  "test.py" \
  ".env" \
  "*.zip"
```

## HTTPメソッド別の処理

### GETリクエストの処理
```python
from flask import request
import json
from datetime import datetime

def main():
    if request.method != 'GET':
        return json.dumps({'error': 'Method Not Allowed'}, ensure_ascii=False), 405

    # クエリパラメータの取得
    name = request.args.get('name', 'World')
    greeting = request.args.get('greeting', 'Hello')

    result = {
        'message': f'{greeting}, {name}!',
        'timestamp': datetime.now().isoformat(),
        'method': 'GET'
    }

    return json.dumps(result, ensure_ascii=False)
```

### POSTリクエストの処理
```python
from flask import request
import json
from datetime import datetime
import uuid

def main():
    if request.method != 'POST':
        return json.dumps({'error': 'Method Not Allowed'}, ensure_ascii=False), 405

    try:
        # JSON形式のrequest bodyをパース
        request_body = request.get_json()

        # 必須フィールドの検証
        if not request_body or 'name' not in request_body:
            return json.dumps({
                'error': 'Missing required field: name'
            }, ensure_ascii=False), 400

        name = request_body['name']
        email = request_body.get('email', 'not provided')
        message = request_body.get('message', 'No message')

        # 処理ロジック
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

## パッケージ管理

### requirements.txtの作成
依存関係の管理には、`requirements.txt`ファイルを作成します。

```txt
pyyaml
requests>=2.28.0
python-dateutil>=2.8.0
```

### 外部API呼び出しの例
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

        # 外部APIの呼び出し
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

### データ処理の例
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

        # データ処理
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

        # データのソートとフィルタリング
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
    """名前を正規化する関数"""
    if not name:
        return ''
    # 空白を除去し、先頭の文字を大文字にする
    return name.strip().title()
```

## エントリーポイントの設定

### 単一関数
`ファイル名.関数名`をエントリーポイントとして使用します。

ファイル名: `user.py`
関数名: `main`
Entry Point: `user.main`

### 複数関数
1つのファイルで複数の関数を定義できます。

```python
# handlers.py
from flask import request
import json

def get_user():
    # ユーザー照会ロジック
    return json.dumps({'message': 'Get user'}, ensure_ascii=False)

def create_user():
    # ユーザー作成ロジック
    return json.dumps({'message': 'User created'}, ensure_ascii=False)

def update_user():
    # ユーザー修正ロジック
    return json.dumps({'message': 'User updated'}, ensure_ascii=False)
```

エントリーポイントの設定:
- `handlers.get_user`
- `handlers.create_user`
- `handlers.update_user`

## 注意事項

### サポートしていないパッケージ
現在、以下の特徴を持つ複雑なパッケージはサポートしていません。

**サポートしていないパッケージの例:**
- `numpy`、`pandas` (C/C++拡張モジュールが必須)
- `scipy` (システムライブラリ依存関係)
- `tensorflow`、`pytorch` (複雑な初期化プロセス)

**サポート可能なパッケージの例:**
- `requests` (HTTPクライアント)
- `pyyaml` (YAML処理)
- `python-dateutil` (日付・時刻処理)
- `pillow` (画像処理 - 基本機能)

### メモリ及び実行時間に関する考慮事項
- 関数は、限られたメモリと実行時間内で動作する必要があります。
- 大容量データを処理する際は、ジェネレータやストリーム処理を検討してください。
- 長時間実行されるタスクは、適切に分割してください。
