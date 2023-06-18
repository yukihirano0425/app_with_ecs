# Part6

## 事前準備

### パッケージを作成

## CodeArtifactの操作

### ドメインを作成する

AWS CLIでドメインを作成します。

```sh
aws codeartifact create-domain --domain cicd-handson-domain
```

### 　リポジトリを作成する

```sh
aws codeartifact create-repository --domain cicd-handson-domain --domain-owner $AWS_ACCOUNT_ID --repository cicd
```

### npm-storeを作成する

```sh
aws codeartifact create-repository --domain cicd-handson-domain --domain-owner $AWS_ACCOUNT_ID --repository npm-store
```

### リポジトリとnpm-store を接続する

```sh
aws codeartifact associate-external-connection --domain cicd-handson-domain --domain-owner $AWS_ACCOUNT_ID --repository npm-store --external-connection "public:npmjs"
```

### リポジトリを更新する

```sh
aws codeartifact update-repository --repository cicd --domain cicd-handson-domain --domain-owner $AWS_ACCOUNT_ID --upstreams repositoryName=npm-store
```

### CodeArtifactにログイン

```sh
echo $AWS_DOMAIN
echo $AWS_ACCOUNT_ID
aws codeartifact login --tool npm --domain $AWS_DOMAIN --region $AWS_DEFAULT_REGION --domain-owner $AWS_ACCOUNT_ID --repository handson
aws codeartifact get-repository-endpoint --domain $AWS_DOMAIN --domain-owner $AWS_ACCOUNT_ID --repository handson --format npm
```

## npmの設定

```sh
export CODEARTIFACT_URL="https://$AWS_DOMAIN-$AWS_ACCOUNT_ID.d.codeartifact.$AWS_DEFAULT_REGION.amazonaws.com/npm/handson/" && echo $CODEARTIFACT_URL

yarn config set npmRegistryServer "$CODEARTIFACT_URL"
CODEARTIFACT_AUTH_TOKEN=`aws codeartifact get-authorization-token --domain --region $AWS_DEFAULT_REGION $AWS_DOMAIN --domain-owner $AWS_ACCOUNT_ID --query authorizationToken --output text`
yarn config set 'npmRegistries["$CODEARTIFACT_URL"].npmAuthToken' "${CODEARTIFACT_AUTH_TOKEN}" 
yarn config set 'npmRegistries["$CODEARTIFACT_URL"].npmAlwaysAuth' "true"
```

## パッケージを登録

## 登録されたパッケージの一覧を表示する

```sh
aws codeartifact list-packages --domain cicd-handson-domain --repository cicd
```

## 片付け

### リポジトリを削除する

```sh
aws codeartifact delete-repository --domain cicd-handson-domain --domain-owner $AWS_ACCOUNT_ID --repository cicd
```

### ドメインを削除する

```sh
zaws codeartifact delete-domain --domain cicd-handson-domain --domain-owner $AWS_ACCOUNT_ID
```
