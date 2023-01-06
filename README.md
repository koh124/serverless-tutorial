## ServerLess Frameworkを使ってAWS Lambdaを使ってみる

### 【getting started】
1. npmでserverlessフレームワークをインストールする
```
$ npm install -g serverless
```

2. AWSでIAMユーザーを作成する

設定は、

ユーザー名: serverless_user（任意）

アクセスの種類: 「プログラムによるアクセス」にチェック

アクセス権限: アクセス許可の設定→既存のポリシーを直接アタッチ→AdministratorAccessをチェック

必ずアクセスキーID、シークレットアクセスキーを控える

シークレットアクセスキーはIAMユーザーを作成したときしか確認できない

3. ServerlessフレームワークにIAMユーザーのアクセスキーID、シークレットアクセスキーを設定する

```
$ serverless config credentials --provider aws --key ACCESS_KEY_ID --secret SECRET_ACCESS_KEY
```
設定したアクセスキーID、シークレットアクセスキーは`~/.aws/credentials`に保存される
```
$ cat ~/.aws/credentials

// result
[default]
aws_access_key_id=ACCESS_KEY_ID
aws_secret_access_key=SECRET_ACCESS_KEY
```

4. アプリケーションを作成する

コマンド概要

sls: `serverless`コマンドのエイリアス

-t: 関数に使用する言語やインフラ環境のテンプレートを選択。`--template`オプションの省略形

-p: サービスのディレクトリを指定する。`--path`オプションの省略形

```
$ sls create -t aws-python -p serverless_sample
```

実行後、以下のようなツリーが構成されている。

```
serverless_sample
|＿handler.py
|＿serverless.yml
```

4. serverless.ymlの設定

以下のような初期設定を行う。

```
provider:
  name: aws
  runtime: python3.7
  region: ap-northeast-1
```


5. REST APIを構築する

```
# handler.py
def hello(event, context):
  body = {
      "message": "Go Serverless v1.0! Your function executed successfully!",
      "input": event
  }

  response = {
      "statusCode": 200,
      "body": json.dumps(body)
  }

  return response
  
# serverless.yml
functions:
  hello:
    handler: handler.hello
  events:
  # REST API エンドポイント
    - httpApi:
        path: /users/create
        method: get
```

6. AWSにデプロイする
```
$ serverless deploy
```

表示されたURL（endpoint: /users/create）にブラウザからアクセスすると、JSONレスポンスが返却される

7. webページを作る
```
# handler.py
def index(event, context):
  html = """<!DOCTYPE html>
  <html lang="en">
  <head>
  <meta charset="UTF-8">
  <meta http-equiv="X-UA-Compatible" content="IE=edge">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Document</title>
  </head>
  <body>
  webpage
  </body>
  </html>"""

  response = {
      "statusCode": 200,
      "headers": {
          "Content-Type": "text/html"
      },
      "body": html
  }

  return response

# serverless.yml
functions:
  web:
    handler: handler.index
    events:
      - http:
          path: '/'
          method: get
```

同じようにデプロイしてエンドポイントにアクセスすると、webpageが表示される

### 動作のイメージ

デプロイ後、AWSのCloudFormationからサービス（スタック）がどのように構築されているかを確認できる

アプリケーション名は`serverless.yml`の`service`に記述したものになる（serverless-sample）

アプリケーションに登録した関数を、コマンド一発で呼び出したり、任意のイベントをトリガーに呼び出すことができる

今回はserverless.ymlのfunctionsで指定した`web`と`hello`がAWS Lambdaに関数として登録されている

今回はeventsの記述でhttpリクエストがあった場合に呼び出す指定をしたため、ブラウザからURLを開くとJSONやwebページが返却された
