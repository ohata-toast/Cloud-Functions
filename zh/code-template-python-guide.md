## Compute > Cloud Functions > Code Template Guide > Python

This document details how to develop functions by using Python from NHN Cloud's Cloud Functions service.

## Template information
| Item              | Value                  |
|-----------------|------------------|
| **Supported version**       | 3.11, 3.12, 3.13 |
| **File name**         | user.py            |
| **Entry Point** | user.main        |

## Basic template

### Hello World example
A basic form of function.

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

### Context object
Python functions can access HTTP request information through Flask's request object.

```python
from flask import request
import json

def main():
    # HTTP request information
    method = request.method
    headers = dict(request.headers)
    args = request.args.to_dict()

    # Request body (such as POST/PUT)
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

## Download and use template file

### Template download
You can download the Python template provided by Cloud Functions to develop a local environment.

**Template download link**: [python.zip](https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_2acdfabf4efe4efc8a04c00b348110c9/cdn_origin/prod_cloud_functions/templates/python/python.zip)

### Template file structure
The structure of the downloaded template file is as follows:

```
python.zip
├── user.py          # Main function file
└── requirements.txt # Dependency management file
```

#### user.py
Include basic YAML processing functions.
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

### Local development process

#### 1. Unzip
```bash
# Unzip
unzip python.zip -d my-function

# Move to task directory
cd my-function
```

#### 2. Modify function codes
Modify `user.py` file with the logic you want.

```python
# user.py - Simple modification example
from flask import request
import json
from datetime import datetime

def main():
    try:
        # Import name from query parameter
        name = request.args.get('name', 'World')

        # Import message from body if POST request
        custom_message = ''
        if request.method == 'POST':
            try:
                data = request.get_json()
                if data:
                    custom_message = data.get('message', '')
            except Exception:
                # Ignore if JSON parsing fails
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

#### 3. Compress into a ZIP file
Compress the modified source code into ZIP file.

```bash
# Windows (PowerShell)
Compress-Archive -Path .\user.py, .\requirements.txt -DestinationPath my-function.zip

# Windows (when using 7-Zip)
7z a my-function.zip user.py requirements.txt

# macOS/Linux
zip my-function.zip user.py requirements.txt

# Include all files (if any additional files are available)
zip -r my-function.zip . -x "*.git*" "__pycache__/*" "*.pyc" "test.py"
```

### Upload from Cloud Functions console
> Used when uploading files from the user's local environment when creating or modifying a function. (refer to Console Guide)

### Cautions for upload

#### ZIP file structure
- The `.py` file and `requirements.txt` must be located directly in the root of the ZIP file.
- We recommend avoiding unnecessary folder structures.

**Right structure:**
```
my-function.zip
├── user.py
├── requirements.txt
└── utils.py (if there are additional files)
```

**Wrong structure:**
```
my-function.zip
└── my-function/
    ├── user.py
    └── requirements.txt
```

#### File size limit
- ZIP file size is limited to 100MiB.
- `__pycache__` file should not be included.

#### Files to exclude
```bash
# Similar to .gitignore, exclude the following files:
zip -r my-function.zip . -x \
  "__pycache__/*" \
  "*.pyc" \
  ".git/*" \
  "*.log" \
  "test.py" \
  ".env" \
  "*.zip"
```

## Process by HTTP method

### Process GET request
```python
from flask import request
import json
from datetime import datetime

def main():
    if request.method != 'GET':
        return json.dumps({'error': 'Method Not Allowed'}, ensure_ascii=False), 405

    # Import query parameter
    name = request.args.get('name', 'World')
    greeting = request.args.get('greeting', 'Hello')

    result = {
        'message': f'{greeting}, {name}!',
        'timestamp': datetime.now().isoformat(),
        'method': 'GET'
    }

    return json.dumps(result, ensure_ascii=False)
```

### Process POST request
```python
from flask import request
import json
from datetime import datetime
import uuid

def main():
    if request.method != 'POST':
        return json.dumps({'error': 'Method Not Allowed'}, ensure_ascii=False), 405

    try:
        # Parse request body as a JSON format
        request_body = request.get_json()

        # Validate required fields
        if not request_body or 'name' not in request_body:
            return json.dumps({
                'error': 'Missing required field: name'
            }, ensure_ascii=False), 400

        name = request_body['name']
        email = request_body.get('email', 'not provided')
        message = request_body.get('message', 'No message')

        # Processing logic
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

## Manage packages

### Write requirements.txt
Write `requirements.txt` to manage dependencies.

```txt
pyyaml
requests>=2.28.0
python-dateutil>=2.8.0
```

### Example of external API call
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

        # External API call
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

### Data processing example
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

        # Data processing
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

        # Data sorting and filtering
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
    """name normalization function"""
    if not name:
        return ''
    # Remove spaces and capitalize the first letter
    return name.strip().title()
```

## Configure Entry Point

### Single function
Use `filename.functionname` as the Entry Point.

File name: `user.py`
Function name: `main`
Entry Point: `user.main`

### Multiple functions
You can define multiple functions in a single file.

```python
# handlers.py
from flask import request
import json

def get_user():
    # User lookup logic
    return json.dumps({'message': 'Get user'}, ensure_ascii=False)

def create_user():
    # User creation logic
    return json.dumps({'message': 'User created'}, ensure_ascii=False)

def update_user():
    # User modification logic
    return json.dumps({'message': 'User updated'}, ensure_ascii=False)
```

Entry Point Configuration:
- `handlers.get_user`
- `handlers.create_user`
- `handlers.update_user`

## Caution

### Unsupported packages
Complex packages with the following features are not currently supported:

**Examples of unsupported packages:**
- `numpy`, `pandas` (C/C++ extension module required)
- `scipy` (system library dependencies)
- `tensorflow`, `pytorch` (complex initialization process)

**Examples of supported packages:**
- `requests` (HTTP client)
- `pyyaml` (YAML processing)
- `python-dateutil` (date/time processing)
- `pillow` (image processing - basic feature)

### Considerations for Memory and execution time
- Functions must operate within limited memory and execution time.
- Consider generator and stream processing when handling large-scale data.
- Split long-running tasks appropriately.
