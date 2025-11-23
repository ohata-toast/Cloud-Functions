## Compute > Cloud Functions > コードテンプレートガイド > Ruby

このドキュメントでは、NHN CloudのCloud FunctionsサービスでRubyを使用して関数を開発する方法を詳しく説明します。

## テンプレート情報
| 項目             | 値       |
|-----------------|----------|
| **サポートバージョン**       | 3.4.5    |
| **ファイル名**         | parse.rb |
| **Entry Point** | handler  |

## 基本テンプレート

### Hello Worldの例
最も基本的な関数の形式です。

```ruby
# frozen_string_literal: true

require 'nokogiri'

def handler(context)
  context.logger.info("Received request")

  doc = Nokogiri::XML(context.request.body.read)
  ele = doc.at_xpath('//message')
  msg = ele.nil? ? 'No Message' : ele.content

  Rack::Response.new([msg, "\n"]).finish
end
```

### Contextオブジェクト
関数に渡される`context`オブジェクトを通じて、ロガーやリクエスト情報にアクセスできます。

```ruby
# frozen_string_literal: true

require 'json'

def handler(context)
  # ロガーの使用
  context.logger.info("Function execution started")

  # HTTPリクエスト情報
  request = context.request
  method = request.request_method # GET, POSTなど
  headers = request.env.select { |k, v| k.start_with?('HTTP_') }
  query_params = request.params
  body = request.body.read

  # レスポンスデータの構成
  response_data = {
    method: method,
    headers: headers,
    query_params: query_params,
    body: body
  }.to_json

  # Rack::Responseを使用してレスポンスを返す
  response = Rack::Response.new([response_data])
  response['Content-Type'] = 'application/json'
  response.finish
end
```

## テンプレートファイルのダウンロードと活用

### テンプレートのダウンロード
Cloud Functionsが提供するRubyテンプレートをダウンロードし、ローカル環境で開発できます。

**テンプレートのダウンロードリンク**: [ruby.zip](https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_2acdfabf4efe4efc8a04c00b348110c9/cdn_origin/prod_cloud_functions/templates/ruby/ruby.zip)

### テンプレートのファイル構造
ダウンロードしたテンプレートファイルの構造は次のとおりです。

```
ruby.zip
├── parse.rb
└── Gemfile
```

### ローカルでの開発プロセス

#### 1. 解凍
```bash
# 解凍
unzip ruby.zip -d my-function

# 作業ディレクトリへ移動
cd my-function
```

#### 2. 関数コードの修正
`parse.rb`ファイルを、目的のロジックに合わせて修正します。

```ruby
# parse.rb - 簡単な修正例
# frozen_string_literal: true

require 'json'
require 'time'

def handler(context)
  begin
    # クエリパラメータから名前を取得
    name = context.request.params['name'] || 'World'

    # POSTリクエストの場合、bodyからメッセージを取得
    custom_message = ''
    if context.request.post? && context.request.body
      begin
        body = JSON.parse(context.request.body.read)
        custom_message = body['message'] || ''
      rescue JSON::ParserError
        # JSONのパースに失敗した場合は無視
      end
    end

    response_data = {
      greeting: "Hello, #{name}!",
      message: custom_message,
      method: context.request.request_method,
      timestamp: Time.now.iso8601
    }.to_json

    Rack::Response.new([response_data], 200, { 'Content-Type' => 'application/json' }).finish
  rescue => e
    error_response = {
      error: 'Internal Server Error',
      details: e.message
    }.to_json
    Rack::Response.new([error_response], 500, { 'Content-Type' => 'application/json' }).finish
  end
end
```

#### 3. ZIPファイルへ圧縮
修正したソースコードを、再度ZIPファイルへ圧縮します。`parse.rb`と`Gemfile`がルートディレクトリに含まれるように圧縮する必要があります。

```bash
zip my-function.zip parse.rb Gemfile Gemfile.lock
```
**参考**: `Gemfile.lock`ファイルはデプロイ段階で自動的に生成されますが、依存関係の正確なバージョンを事前に確認または固定したい場合は、ローカルで`bundle install`を実行して直接生成した後、一緒に圧縮できます。`Gemfile.lock`ファイルが含まれている場合、そのファイルに明記されたバージョンを使用してビルドされます。

### Cloud Functionsコンソールでのアップロード
- 関数を作成または修正する際に、**ユーザーローカル環境**方式を選択します。
- **ファイルを選択**をクリックし、作成した`my-function.zip`ファイルをアップロードします。

### アップロード時の注意事項
- **アップロードファイル**: `*.rb`、`Gemfile`が含まれた**ZIPファイル**をアップロードする必要があります。
  - `Gemfile.lock`ファイルは任意です。含まれている場合はそのファイルに明記されたバージョンでビルドされ、ない場合はデプロイ段階で自動的に生成されます。
- **ZIPファイルの構造**: ZIPファイルのルートに各ファイルを配置する必要があります。
- **ファイルサイズ**: ZIPファイルのサイズは100MiB以下に制限されます。

## HTTPメソッド別の処理

### GETリクエストの処理
```ruby
# frozen_string_literal: true

require 'json'
require 'time'

def handler(context)
  unless context.request.get?
    return Rack::Response.new(['Method Not Allowed'], 405).finish
  end

  name = context.request.params['name'] || 'World'
  greeting = context.request.params['greeting'] || 'Hello'

  response_data = {
    message: "#{greeting}, #{name}!",
    timestamp: Time.now.iso8601
  }.to_json

  Rack::Response.new([response_data], 200, { 'Content-Type' => 'application/json' }).finish
end
```

### POSTリクエストの処理
```ruby
# frozen_string_literal: true

require 'json'
require 'time'
require 'securerandom'

def handler(context)
  unless context.request.post?
    return Rack::Response.new(['Method Not Allowed'], 405).finish
  end

  begin
    request_body = JSON.parse(context.request.body.read)
    name = request_body['name']

    unless name && !name.empty?
      return Rack::Response.new(['Missing required field: name'], 400).finish
    end

    response_data = {
      id: SecureRandom.hex(4),
      name: name,
      email: request_body['email'] || 'not provided',
      processed_at: Time.now.iso8601
    }.to_json

    Rack::Response.new([response_data], 201, { 'Content-Type' => 'application/json' }).finish
  rescue JSON::ParserError
    Rack::Response.new(['Invalid JSON format'], 400).finish
  end
end
```

## パッケージ管理(`Gemfile`)

依存関係(Gem)の管理には`Gemfile`を使用します。必要なGemを追加し、`bundle install`を実行して`Gemfile.lock`を生成します。

```ruby
# Gemfile
# frozen_string_literal: true

source "https://rubygems.org"

gem "nokogiri", "~> 1.18" # XML/HTMLパーサー
gem "httparty", "~> 0.23" # HTTPクライアント
```

### 外部API呼び出しの例
```ruby
# frozen_string_literal: true

require 'json'
require 'httparty'

def handler(context)
  begin
    user_id = context.request.params['userId']

    api_url = if user_id
                "https://jsonplaceholder.typicode.com/users/#{user_id}"
              else
                "https://jsonplaceholder.typicode.com/users"
              end

    response = HTTParty.get(api_url)

    unless response.success?
      error_message = {
        error: "External API Error",
        status_code: response.code,
        details: response.body
      }.to_json
      return Rack::Response.new([error_message], response.code, { 'Content-Type' => 'application/json' }).finish
    end

    Rack::Response.new([response.body], 200, { 'Content-Type' => 'application/json' }).finish

  rescue => e
    error_response = {
      error: 'Internal Server Error',
      details: e.message
    }.to_json
    Rack::Response.new([error_response], 500, { 'Content-Type' => 'application/json' }).finish
  end
end
```

## エントリーポイントの設定

エントリーポイントには関数名を使用します。

- ファイル名: `parse.rb`
- 関数名: `handler`
- Entry Point: `handler`

### 注意事項
- **Bundlerのバージョン**: `Gemfile.lock`を作成する際には、Bundler 2.6.2以上のバージョンの使用を推奨します。
- **ActiveSupportなど一部のGemの互換性**: `ActiveSupport`のようにC拡張(C extension)に依存したり、内部構造が複雑だったりする一部のGemは、現在のCloud Functions環境と互換性の問題が発生する可能性があります。`activesupport` Gemは内部で`zeitwerk` Gemに依存しており、このGemがファイルシステムを探索する方法が実行環境と競合し、正常に動作しない場合があります。そのため、できるだけ標準ライブラリや外部依存の少ないGemを使用することを推奨します。
