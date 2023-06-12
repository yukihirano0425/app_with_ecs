# part3

## 事前準備

[aws_happy_code - GitHub](https://github.com/ymd65536/aws_happy_code.git)をgitコマンドでDesktop上にcloneします。

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

### リポジトリを作成する

`aws_happy_code`リポジトリでターミナルを開き、part3にディレクトリを変更します。

```sh
cd ~/Desktop/aws_happy_code/part3
```

以下のコマンドで`codecommit.yml`をCloudFormationで実行します。

```sh
aws cloudformation deploy --stack-name codecommit --template-file ./codecommit.yml --tags Name=cicdhandson --profile cicd_handson
```

### CodeCommitのリポジトリをクローンする

Desktop上にCodeCommitのリポジトリをcloneします。

```sh
cd ~/Desktop
```

```sh
git clone codecommit::ap-northeast-1://cicd_handson@cicdhandson
```

ディレクトリを移動します。

```sh
cd ~/Desktop/cicdhandson
```

### mainブランチを作成

```sh
git checkout -b main
```

```sh
echo "Hello Lambda" > README.md
```

```sh
git add .
git commit -m "part3"
git push -u 
```

### lambda_handsonブランチを切る

新しいブランチでビルドを実行する為にCodeBuild用に新しくブランチを切ります。

```sh
git checkout -b lambda_handson
```

### buildspec.yamlを作成する

CodeBuildで利用する設定ファイル(buildspec.yml)を作成します。
part3ディレクトリにあるbuildspec.ymlを`cicd_handson`リポジトリにコピーします。

```sh
cp ~/Desktop/aws_happy_code/part3/buildspec.yml ~/Desktop/cicdhandson/
```

### dockerfileを作成する

dockerfileを追加します。
`aws_code_happy`リポジトリに戻り、part3ディレクトリにあるdockerfileを`cicd_handson`リポジトリにコピーします。

```sh
cp ~/Desktop/aws_happy_code/part3/dockerfile ~/Desktop/cicdhandson/
```

### app.py　を追加

```sh
cp ~/Desktop/aws_happy_code/part3/app.py ~/Desktop/cicdhandson/
```

### リモートリポジトリを更新する

CodeCommitのリモートリポジトリにdockerfileをpushします。
リモートリポジトリにブランチを追加します。

```sh
git add .
git commit -m "part3"
git push --set-upstream origin lambda_handson
```

### CodeBuild用 S3バケットの作成

`aws_happy_code`リポジトリでターミナルを開き、part3にディレクトリを変更します。

```sh
cd ~/Desktop/aws_happy_code/part3
```

以下のコマンドで`s3.yml`をCloudFormationで実行します。

```sh
aws cloudformation deploy --stack-name s3 --template-file ./s3.yml --tags Name=cicdhandson --profile cicd_handson
```

### ECRリポジトリの作成

以下のコマンドで`ecr.yml`をCloudFormationで実行します。

```sh
aws cloudformation deploy --stack-name ecr --template-file ./ecr.yml --tags Name=cicdhandson --profile cicd_handson
```

### ハンズオンで利用するIAM Roleを作成する

```sh
aws cloudformation deploy --stack-name codebuild-iam-role --template-file ./codebuild-role.yml --tags Name=cicdhandson --capabilities CAPABILITY_NAMED_IAM --profile cicd_handson
```

```sh
aws cloudformation deploy --stack-name event-bridge-iam-role --template-file ./event-bridge-iam-role.yml --tags Name=cicdhandson --capabilities CAPABILITY_NAMED_IAM --profile cicd_handson
```

```sh
aws cloudformation deploy --stack-name pipeline-iam-role --template-file ./pipeline-iam-role.yml --tags Name=cicdhandson --capabilities CAPABILITY_NAMED_IAM --profile cicd_handson
```

### CodeBuildのプロジェクトを作成する

```sh
aws cloudformation deploy --stack-name code-build --template-file ./code-build.yml --tags Name=cicdhandson --profile cicd_handson
```

### CodePipeline の環境構築

CloudFormationでCodePipelineを構築します。
以下のコマンドで`codecommit.yml`をCloudFormationで実行します。

```sh
aws cloudformation deploy --stack-name pipeline --template-file ./pipeline.yml --tags Name=cicdhandson --profile cicd_handson
```

### プルリクエストを作成する

```sh
aws codecommit create-pull-request --title "part3" --description "part3 lambda ci/cd" --targets repositoryName=cicdhandson,sourceReference=lambda_handson --profile cicd_handson
```

```sh
ACCOUNT_ID=`aws sts get-caller-identity --profile cicd_handson --query 'Account' --output text` && echo $ACCOUNT_ID
```

```sh
PULL_REQUEST_ID=`aws codecommit list-pull-requests --profile cicd_handson --pull-request-status OPEN --repository-name cicdhandson --query 'pullRequestIds' --output text` && echo $PULL_REQUEST_ID
```

```sh
REVISIONID=`aws codecommit get-pull-request --pull-request-id $PULL_REQUEST_ID --profile cicd_handson --query 'pullRequest.revisionId' --output text` && echo $REVISIONID
```

```sh
aws codecommit evaluate-pull-request-approval-rules --pull-request-id $PULL_REQUEST_ID --revision-id $REVISIONID --profile cicd_handson
```

```sh
COMMITID=`aws codecommit get-branch --repository-name cicdhandson --branch-name lambda_handson --profile cicd_handson --query 'branch.commitId' --output text` && echo $COMMITID
```

### ブランチをマージする

```sh
aws codecommit merge-pull-request-by-fast-forward --pull-request-id $PULL_REQUEST_ID --source-commit-id $COMMITID --repository-name cicdhandson --profile cicd_handson
```

### Lambdaを関数を作成する

CloudFormationでLambdaを構築します。
以下のコマンドで`lambda.yml`をCloudFormationで実行します。

```sh
aws cloudformation deploy --stack-name lambda --template-file ./lambda.yml --tags Name=cicdhandson --capabilities CAPABILITY_NAMED_IAM --profile cicd_handson
```

### Lambda　関数のテスト

AWS CLIでLambdaを実行します。

```bash
aws lambda invoke --profile cicd_handson --function-name "cicdhandsonFunc" --invocation-type RequestResponse --region "ap-northeast-1" response.json && cat response.json
```

### 新しいバージョンの関数を作成する

```bash
aws lambda update-function-code --profile cicd_handson --function-name "cicdhandsonFunc" --image-uri "{アカウントID}.dkr.ecr.ap-northeast-1.amazonaws.com/cicdhandson:latest" --publish
```

### app.py を修正する

```py
def lambda_handler(event, context):

    return {
        'Hello': 'Test Lambda 2'
    }

```

### エイリアスを作成する

```bash
aws lambda create-alias --profile cicd_handson --function-name "cicdhandsonFunc" --function-version "1" --name deploy
```

### エイリアスを変更する

```bash
aws lambda update-alias --profile cicd_handson --function-name "cicdhandsonFunc" --function-version "2" --name deploy
```

### Lambda　関数のテスト(エイリアスを変更後)

```bash
aws lambda invoke --profile cicd_handson --function-name "cicdhandsonFunc:deploy" --invocation-type RequestResponse --region "ap-northeast-1" response.json && cat response.json
```

## 片付け

### パイプラインを削除

```sh
aws cloudformation delete-stack --stack-name pipline --profile cicd_handson
```

### CodeCommit

```sh
aws cloudformation delete-stack --stack-name codecommit --profile cicd_handson
```

## IAM

```sh
aws cloudformation delete-stack --stack-name pipeline-iam-role --profile cicd_handson
```

```sh
aws cloudformation delete-stack --stack-name code-build --profile cicd_handson
```

```sh
aws cloudformation delete-stack --stack-name code-build-iam-role --profile cicd_handson
```

### S3バケットを空にする

```sh
aws s3 ls --profile cicd_handson | grep cicdhandson | awk '{print $3}'
```

```sh
aws s3 rm s3://cicdhandson-bucket-{アカウント}/ --recursive --profile cicd_handson
```

中身を確認します。

```sh
aws s3 ls s3://cicdhandson-bucket-{アカウントID} --profile cicd_handson
```

```sh
aws s3 rb s3://cicdhandson-bucket-{アカウントID} --force
```

## まとめ

これでハンズオンは以上です。上記の構成でCodeCommit にDockerfileをおくことにより
buildspec.ymlの設定に従ってCodeBuildでイメージをビルドできます。これでイメージをリポジトリにpushしたことをトリガーに
CodeDeployによるデプロイやApp Runnerへのアプリケーションデプロイができます。
