# サーバーレスアーキテクチャで翻訳Web APIを構築する

- 今回学ぶAWSサービス
1. Lambda
2. API GATEWAY
3. DynamoDB

<img width="700" src="img/1.png">


# Lambdaの情報

## ハンドラーとは
- ```AWS Lambda```におけるハンドラー（handler）は、**Lambda関数が実行される際に最初に呼び出されるメソッドや関数のこと**を指します。  
簡単に言えば、**Lambda関数のエントリーポイント**です。  
このハンドラーは、イベントデータを受け取り、それを処理するロジックを実行します。  

- 具体的な役割  
ハンドラーは以下のような役割を担います：
1. イベントデータの受け取り: トリガー（例: API Gateway, S3, DynamoDBなど）から送信されたデータを受け取ります。
2. データの処理: 受け取ったデータに基づいて必要な処理を実行します。
3. 結果の返却: 処理結果を返却します（必要に応じて）。

設定例（Pythonの場合）  
例えば、PythonでLambda関数を定義する場合のハンドラーは以下のようになります：  
```
def lambda_handler(event, context):
    # イベントデータの処理
    print("Received event:", event)

    # レスポンスの作成
    return {
        'statusCode': 200,
        'body': 'Hello, Lambda!'
    }
```

- ハンドラー関数の仕組み  
```event``` :Lambda関数に渡されるデータ（JSON形式が多い）。  
```context```: 実行環境に関する情報（例: 実行時間の制限、関数名など）を含むオブジェクト。 

- ハンドラー指定方法  
Lambda関数を作成する際に、AWSコンソールやCLIでハンドラーの名前を指定します。  
例えば、file_name.function_name の形式で指定します。  
上記の例では、ファイル名が lambda_function.py で関数名が lambda_handler なら、以下のように指定します：  
```
lambda_function.lambda_handler
```
## Node.js の関数ハンドラー  
- 関数の設定でハンドラーパラメータを指定することで、呼び出すハンドラーメソッドを Lambda ランタイムに指示できます。  
- Node.js で関数を設定するとき、ハンドラー設定の値は、ファイル名と、エクスポートしたハンドラーモジュールの名前を、ドットで区切ったものになります。  
たとえば、コンソールのデフォルト値は、index.js で exports.handler を呼び出す index.handler です  


IAMロール→AWSリソースにアタッチする。

# AWS Lambda: 同期呼び出しと非同期呼び出し

## 同期呼び出し (Synchronous Invocation)

- **動作**: 呼び出し元はLambda関数の結果が返されるまで待機します。
- **用途**:
  - 結果が即座に必要な場合。
  - 例: API Gatewayを使用してHTTPリクエストを処理。
- **流れ**:
  1. 呼び出し元がLambda関数を実行。
  2. Lambdaが処理を実行し、結果を返却。
  3. 呼び出し元が結果を受け取る。
- **メリット**:
  - 即座に結果を取得可能。
- **デメリット**:
  - 実行が遅れると、呼び出し元の処理も遅延する。

## 非同期呼び出し (Asynchronous Invocation)

- **動作**: 呼び出し元はLambda関数の実行をトリガーしますが、処理結果を待たずに次の処理を進めます。
- **用途**:
  - 結果をすぐに必要としない場合。
  - 例: S3バケットのイベント通知やバッチ処理。
- **流れ**:
  1. 呼び出し元がLambda関数を呼び出す。
  2. 呼び出しがキューに送信され、Lambdaがバックグラウンドで処理。
  3. 必要に応じて結果をCloudWatch Logsや別サービスに出力。
- **メリット**:
  - 呼び出し元は待機せずに他の処理を継続可能。
- **デメリット**:
  - エラー処理が複雑になる可能性あり（DLQの設定が推奨される）。

## 比較表

| 特徴                | 同期呼び出し                           | 非同期呼び出し                        |
|---------------------|-------------------------------------|-------------------------------------|
| **レスポンスの必要性** | 即座に結果を取得する必要がある場合     | 結果が後から利用される場合            |
| **処理の性質**        | ユーザーインタラクションに適している  | バッチ処理やバックグラウンド処理に適している |
| **エラー処理**        | 呼び出し元でエラーを処理可能          | Dead Letter Queue (DLQ) を活用可能  |

***

<br><br><br><br>


# AWS Lambdaはコンテナを使っている？

## **AWS Lambdaのコンテナ利用について**

AWS Lambdaは、関数の実行環境としてコンテナ技術を利用しています。LambdaはAWSが管理する軽量なコンテナランタイムを使用し、関数ごとに独立した環境を提供しています。以下にその詳細を説明します。

---

## **1. マネージドなコンテナ環境**
- **AWSによる管理**:
  - Lambda関数は、AWSが自動的にコンテナを作成し、実行環境として利用します。
  - ユーザーがコンテナの管理やスケーリングを直接行う必要はありません。
- **実行の仕組み**:
  - Lambda関数が呼び出されると、必要に応じてコンテナがプロビジョニングされ、その中でコードが実行されます。
  - 再利用可能な場合は、既存のコンテナを再利用することで高速化を実現します（コールドスタートとウォームスタート）。

---

## **2. 独自のコンテナイメージを利用可能**
- **独自のイメージサポート**:
  - ユーザーはDockerコンテナイメージを作成し、それをLambdaで実行できます。
- **Amazon ECRとの統合**:
  - コンテナイメージをAmazon Elastic Container Registry (ECR) にプッシュし、Lambda関数として使用することが可能です。
- **利用用途**:
  - カスタムランタイムや依存関係を持つアプリケーションを実行したい場合に便利です。

---

## **3. セキュリティと隔離**
- **環境の独立性**:
  - 各関数は独立したコンテナ環境内で実行され、他の関数からの干渉を防ぎます。
- **セキュリティ**:
  - AWSは、基盤となるホストやコンテナランタイムのセキュリティを管理し、ユーザーのコード実行を安全に保ちます。

---

## **4. コンテナイメージ利用の具体例**
以下は、Lambdaで独自のコンテナイメージを使用する場合の手順例です：

1. **Dockerfileを作成**:
   ```dockerfile
   FROM public.ecr.aws/lambda/python:3.8
   COPY app.py .
   CMD ["app.lambda_handler"]

2. Dockerイメージをビルド:
```
docker build -t my-lambda-image .
```

3. Amazon ECRにプッシュ:
```
aws ecr create-repository --repository-name my-lambda-image
docker tag my-lambda-image:latest <account-id>.dkr.ecr.<region>.amazonaws.com/my-lambda-image:latest
docker push <account-id>.dkr.ecr.<region>.amazonaws.com/my-lambda-image:latest
```

4. Lambda関数を作成:  
AWS Management ConsoleまたはCLIを使用して、作成したイメージを指定してLambda関数をデプロイします。

***

<br><br><br><br><br><br><br><br>












# AWS Lambda ハンズオン1 Lambdaを単体で使ってみる。

## 関数の作成  
1. 1から作成を選択。  
<img width="1200" src="img/2.png">


### 基本的な情報
1. 関数名に「translate-function」と入力。
2. ランタイムは最新のpython3.13を選択。
3. アーキテクチャ　(Lambda 関数の命令セットアーキテクチャにより、Lambda が関数の実行に使用するコンピュータプロセッサのタイプが決まる。ここではx86_64を選択)
<img width="1200" src="img/3.png">

### デフォルトの実行ロールの変更
- 基本的な Lambda アクセス権限で新しいロールを作成 を選択。
<img width="1200" src="img/4.png">

- 関数の作成ボタンを押下し、正常に作成されたことを確認
<img width="1200" src="img/5.png">


### Lambdaの基本設定を変更する

- 設定タブを開き、一般設定より編集ボタンを押下
<img width="1200" src="img/6.png">

1. メモリの変更
- デフォルトの128MBから、今回は256MBに変更

2. タイムアウトを変更
- デフォルトの3秒から今回は10秒に変更
<img width="1200" src="img/7.png">

- 変更後、保存ボタンを押下

### IAMロールの確認

- 今回は「基本的な Lambda アクセス権限で新しいロールを作成」を選択しているので、Cloudwatch Logsに権限が与えられていることを確認
<img width="1200" src="img/8.png">

<img width="1200" src="img/9.png">

### テスト実行
- ｢テスト｣タブを選択し、イベント名に任意の名前を入力し、テストイベントを作成

<img width="1200" src="img/10.png">

<img width="1200" src="img/11.png">


### ログの確認

- ソースコードを変更
```
import json
import logging

logger = logging.getLogger()
logger.setLevel(logging.INFO)

def lambda_handler(event, context):

    logger.info(event)

    return {
        'statusCode': 200,
        'body': json.dumps('Hello Hands on world!')
    }
```

- 「コード」タブより、ソースコードを変更
<img width="1200" src="img/12.png">

- 「テスト」タブより、テストを実行

<img width="1200" src="img/13.png">

- 「モニタリング」タブより、Cloudwatchログを表示を選択。

<img width="1200" src="img/14.png">

- 「ログストリーム」タブよりログが確認できるので、最新のログをクリック。

<img width="1200" src="img/15.png">

- ログが出力されていることが確認できる。

<img width="1200" src="img/16.png">

# Lambdaハンズオン2　他のサービスを呼び出してみる

- 「コード」タブより、ソースコードを変更  
ログの出力はもう必要ないので、該当する部分を削除
<img width="1200" src="img/20.png">

aws ptyhon sdkと検索

<img width="1200" src="img/17.png">

APIリファレンスを選択

<img width="1200" src="img/18.png">

Translateと検索

<img width="1200" src="img/19.png">

```
import boto3

client = boto3.client('translate')
```
上記のコードを貼り付ける  
今回は分かりやすくするため、clientの部分をtranslateに書き換える。  

<img width="1200" src="img/21.png">

SDKのサイトに戻り、下部の｢translate_text｣を選択

Request Syntaxよりコードをコピー
```
response = client.translate_text(
    Text='string',
    TerminologyNames=[
        'string',
    ],
    SourceLanguageCode='string',
    TargetLanguageCode='string',
    Settings={
        'Formality': 'FORMAL'|'INFORMAL',
        'Profanity': 'MASK',
        'Brevity': 'ON'
    }
)
```

<img width="1200" src="img/23.png">

- インデントを揃える

<img width="1000" src="img/24.png">

- 10~12行目は必要ないので削除する

<img width="1000" src="img/25.png">

- `input_text = 'おはよう'`と追加

<img width="1000" src="img/26.png">


- `Text=input_text,`と変更

<img width="1000" src="img/27.png">

-  `SourceLanguageCode='jp',` `TargetLanguageCode='en',` に変更

<img width="1000" src="img/28.png">

- `response = client.translate_text(`を`response = translate.translate_text(`に変更

<img width="1000" src="img/29.png">

- ` output_text = response.get('TranslatedText')`を追加

<img width="1000" src="img/30.png">

- bodyでoutput_textを表示させる

`            
'output_text': output_text
        })
    }
`
<img width="1000" src="img/31.png">

## IAMロールに許可を追加する
- 設定タブより既存のロールの項目を確認し、「IAM コンソールで translate-function-role-7q9pdvlz ロールを表示 します。」をクリック

<img width="1000" src="img/35.png">

許可ポリシーより、ポリシーをアタッチを選択。

<img width="1000" src="img/33.png">

translateと検索し、TranslateFullAccsessを選択し、許可を追加する。

<img width="1000" src="img/34.png">

Lambdaのコードへの変更を更新するため、コードソースよりDeployを選択。

<img width="1000" src="img/36.png">

Deployが完了した後テストを実行し、問題なく実行できるか確認する。

<img width="1000" src="img/37.png">

# APIGATEWAYを単体で使ってみる

- Mockデータを返すAPIを作成する。

APIGATEWAYより、RESTAPIを選択し、構築する。

<img width="1000" src="img/38.png">

新しいAPIを選択し、API名は｢transtale-api｣と入力。APIエンドポイントタイプはリージョンを選択。

<img width="1000" src="img/39.png">

リソースを作成ボタンを押下

<img width="1000" src="img/40.png">

リソース名に｢sample｣と入力し、リソースを作成する。

<img width="1000" src="img/41.png">

/sampleを選択した状態で、メソッドを作成ボタンを押下

<img width="1000" src="img/42.png">

メソッドタイプはGETを選択し、統合タイプはMOCKを選択する

<img width="1000" src="img/43.png">

以下のように、GETメソッドが作成される。

<img width="1000" src="img/44.png">

統合レスポンスより、編集ボタンを押下


<img width="1000" src="img/45.png">

マッピングテンプレートのコンテンツタイプを`application/json`とし、テンプレート本文に以下のコードを記入し、保存する

```
{
    "statusCode": 200,
    "body": {
        {
            "report_id": 5,
            "report_title" : "Hello, world"
        },
        {
            "report_id": 7,
            "report_title" : "Good morning!"
        }
    }
}
```

<img width="1000" src="img/46.png">

APIをデプロイボタンを押下し、新しいステージを選択し、ステージ名にdevと入力し、デプロイする。

<img width="1000" src="img/47.png">

APIがデプロイされるので｢URL を呼び出す｣を選択する。

<img width="1000" src="img/48.png">

以下のようにAPIが呼び出されることを確認する

<img width="1000" src="img/49.png">

# API GatewayにおけるMOCK

## **MOCKとは**
MOCKは、API Gatewayの統合タイプの1つで、バックエンドサービスが未完成または未提供の状態で仮のレスポンスを返却する機能を指します。フロントエンドとバックエンドの開発を並行して進める際に活用されます。

---

## **主な特徴**
1. **バックエンド不要**:
   - MOCK統合を使用すると、バックエンドの実装がなくてもAPIから仮のレスポンスを提供可能。
2. **開発の効率化**:
   - フロントエンドの開発者はMOCKで仮のレスポンスを受け取ることで、バックエンドを待たずに進められる。
3. **シンプルな設定**:
   - API Gatewayのコンソール内で簡単に設定可能。

# API Gatewayにおけるステージとは

## **ステージの概要**
ステージは、AWS API GatewayでデプロイされたAPIのバージョンや環境を管理するための機能です。主に開発環境、テスト環境、本番環境などの異なる環境ごとに設定を分けるために利用されます。

---

## **ステージの主な特徴**
1. **環境の分離**
   - 開発 (`dev`)、テスト (`test`)、本番 (`prod`) など、異なる環境を区別して管理可能。

2. **URLでの識別**
   - 各ステージには一意のURLが割り当てられます。
     例: `https://{api-id}.execute-api.{region}.amazonaws.com/{stage-name}/`

3. **個別の設定**
   - ステージごとにキャッシュ設定、ステージ変数、CloudWatch Logsの有効化などが設定可能。

4. **バージョン管理**
   - 複数のステージを利用することで、APIの異なるバージョンを同時に運用できます。


# APIGATEWAYとLambdaを組み合わせる

<img width="1000" src="img/50.png">

/直下にリソースを作成する

<img width="1000" src="img/51.png">

リソース名にtranslateと入力し、リソースを作成

<img width="1000" src="img/52.png">

メソッドを作成を押下

<img width="1000" src="img/53.png">

- メソッドタイプはGETを選択し、統合タイプはLambda関数を選択。また、Lambdaプロキシ統合のチェックを入れる  
- Lambda関数は前回作成した、translate-functionを選択し、メソッドを作成する

<img width="1000" src="img/54.png">

メソッドリクエストの設定→編集を押下

<img width="1000" src="img/55.png">

URLクエリ文字列パラメータより、名前に`input_text`と入力し、必須にチェックを入れて保存する

<img width="1000" src="img/56.png">

Lambda関数の編集画面に移り、以下のコードを追加する

```
        'isBase64Encoded': False,
        'headers': {}
```

全体

```
import json
import boto3

translate = boto3.client(service_name='translate')

def lambda_handler(event, context):

    input_text = event['queryStringParameters']['input_text']

    response = translate.translate_text(
        Text=input_text,
        SourceLanguageCode="ja",
        TargetLanguageCode="en"
    )

    output_text = response.get('TranslatedText')

    return {
        'statusCode': 200,
        'body': json.dumps({
            'output_text': output_text
        }),
        'isBase64Encoded': False,
        'headers': {}
    }
```

<img width="1000" src="img/57.png">

テストイベントより、新しいイベントを作成  
イベント名はapicallと入力  
テンプレートｰオプションより、APIと検索し、apigateway-aws-proxyを選択  

<img width="1000" src="img/58.png">

 `"queryStringParameters": `の出力を`input_text:"こんにちは"`に変更

<img width="800" src="img/59.png">

<img width="800" src="img/60.png">

Lambda関数の、input_textの出力を`input_text = event['queryStringParameters']['input_text']`に変更

<img width="500" src="img/61.png">

<img width="500" src="img/62.png">

Lambda関数をデプロイし、テスト実行してみる

<img width="1200" src="img/63.png">

/translateより、APIをデプロイ

<img width="1200" src="img/64.png">

ステージはdevを選択

<img width="800" src="img/65.png">

URLを呼び出すより、URLをコピーして移動する

<img width="1200" src="img/66.png">

URL末尾にクエリ文字列パラメータを追加する必要があるため、`?input_text=こんばんは`と入力し、適切に動作するか確認する

<img width="1200" src="img/67.png">