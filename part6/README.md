# Part6

## 事前準備

## IAMユーザーを作成する

part3ディレクトリにある`iam_user.yml`を使ってハンズオンで利用するIAMユーザーを作成します。
※すでにAdministratorAccessで権限を作成されている場合はこの動作は不要です。

`I acknowledge that AWS CloudFormation might create IAM resources with custom names.`にチェックを入れて`Submmit`をクリックします。

問題なくスタックが作成できましたら`Outputs`からアクセスキーとシークレットアクセスきーをコピーします。

### AWS　CLIをにIAMユーザーを記録する

AWS CLIを設定する為に以下のコマンドを実行します。

```sh
aws configure --profile cicd_handson
```

いくつか質問がなされるので順番に回答します。ここで先ほどのアクセスキーとシークレットアクセスキーを利用します。
`AWS Access Key Id`にアクセスキー、`AWS Secret Access Key`にシークレットアクセスキーを入力します。
リージョンはap-northeast-1、出力形式はjsonで問題ありません。

![9.png](./img/9.png)

最後に設定されているかどうかを確認する為、`credentials`をチェックします。`[cicd_handson]`という項目が追加されていれば問題ありません。

```sh
cat ~/.aws/credentials 
```

## CodeArtifactの操作

### ドメインを作成する

AWS CLIでドメインを作成します。

```sh
aws codeartifact create-domain --domain cicd-handson-domain --profile cicd_handson
```

### 環境変数のセット

```sh
export AWS_DOMAIN="cicd-handson-domain" && echo $AWS_DOMAIN
export AWS_ACCOUNT_ID=`aws sts get-caller-identity --profile cicd_handson --query 'Account' --output text` &&  
echo $AWS_ACCOUNT_ID
export AWS_DEFAULT_REGION="ap-northeast-1" && echo $AWS_DEFAULT_REGION
export CODEARTIFACT_URL="https://$AWS_DOMAIN-$AWS_ACCOUNT_ID.d.codeartifact.$AWS_DEFAULT_REGION.amazonaws.com/npm/handson/" && echo $CODEARTIFACT_URL

```

### 　リポジトリを作成する

```sh
aws codeartifact create-repository --domain cicd-handson-domain --domain-owner $AWS_ACCOUNT_ID --repository cicd --profile cicd_handson
```

### npm-storeを作成する

```sh
aws codeartifact create-repository --domain cicd-handson-domain --domain-owner $AWS_ACCOUNT_ID --repository npm-store --profile cicd_handson
```

### リポジトリとnpm-store を接続する

```sh
aws codeartifact associate-external-connection --domain cicd-handson-domain --domain-owner $AWS_ACCOUNT_ID --repository npm-store --external-connection "public:npmjs" --profile cicd_handson
```

### リポジトリを更新する

```sh
aws codeartifact update-repository --repository cicd --domain cicd-handson-domain --domain-owner $AWS_ACCOUNT_ID --upstreams repositoryName=npm-store --profile cicd_handson
```

### CodeArtifactにログイン

```sh
aws codeartifact login --tool npm --domain $AWS_DOMAIN --region $AWS_DEFAULT_REGION --domain-owner $AWS_ACCOUNT_ID --repository cicd --profile cicd_handson
```

```sh
aws codeartifact get-repository-endpoint --domain $AWS_DOMAIN --domain-owner $AWS_ACCOUNT_ID --repository cicd --format npm --profile cicd_handson
```

### CODEARTIFACT_AUTH_TOKENの発行

```sh
export CODEARTIFACT_AUTH_TOKEN=`aws codeartifact get-authorization-token --domain $AWS_DOMAIN --region $AWS_DEFAULT_REGION --domain-owner $AWS_ACCOUNT_ID --query authorizationToken --output text --profile cicd_handson`
```

## npmの設定

```sh
yarn config set npmRegistryServer "$CODEARTIFACT_URL"
yarn config set 'npmRegistries["$CODEARTIFACT_URL"].npmAuthToken' "${CODEARTIFACT_AUTH_TOKEN}"
yarn config set 'npmRegistries["$CODEARTIFACT_URL"].npmAlwaysAuth' "true"
```

## パッケージを登録

パッケージが登録されていないことを確認します。

```sh
aws codeartifact list-packages --domain cicd-handson-domain --repository cicd --query 'packages' --output text --profile cicd_handson
```

```sh
cd ~/Desktop/aws_happy_code/part6/sample-package
```

CodeArtifactにパッケージを登録します。

```sh
npm publish
```

登録されたパッケージの一覧を表示します。

```sh
aws codeartifact list-packages --domain cicd-handson-domain --repository cicd --profile cicd_handson
```

## CodeArtifactに登録したパッケージをsample-appに読み込む

```sh
cd ~/Desktop/aws_happy_code/part6/sample-app
```

```sh
npm install sample-package@1.0.0
```

```sh
node index.js
```

## 片付け

### リポジトリを削除する

```sh
aws codeartifact delete-repository --domain cicd-handson-domain --domain-owner $AWS_ACCOUNT_ID --output text --repository cicd --profile cicd_handson
```

### ドメインを削除する

```sh
zaws codeartifact delete-domain --domain cicd-handson-domain --domain-owner $AWS_ACCOUNT_ID
```

```sh
yarn config set npmRegistryServer ""
yarn config set 'npmRegistries["$CODEARTIFACT_URL"].npmAlwaysAuth' "false"
npm config set registry ""
```
