# part5

## 事前準備

[aws_happy_code - GitHub](https://github.com/ymd65536/aws_happy_code.git)をgitコマンドでDesktop上にcloneします。
本章ではDockerコマンドを利用するため、Dockerがインストールされている環境を用意してください。
なお、AWS上で実行できるようになっていますのでローカル上で実行する手順をスキップして動作確認をすることもできます。

### IAMユーザーを作成する

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

### AWS SAM CLIをインストール

AWS SAM CLIを使うことでAWS SAMを使ったサーバーレスアプリケーションを構築できます。
CLIは公式ドキュメントに記載されていますのでインストールしてから次に進むようにしてください。

[インストール方法](https://docs.aws.amazon.com/ja_jp/serverless-application-model/latest/developerguide/install-sam-cli.html)

## ハンズオン

### リポジトリを作成する

`aws_happy_code`リポジトリでターミナルを開き、part3にディレクトリを変更します。

```sh
cd ~/Desktop/aws_happy_code/part4
```

### SAM CLIでアプリケーションを作成する

SAM CLIでサーバーレスアプリケーションを作成するために以下のコマンドを実行します。

```sh
sam init
```

```text
% sam init 

You can preselect a particular runtime or package type when using the `sam init` experience.
Call `sam init --help` to learn more.

Which template source would you like to use?
        1 - AWS Quick Start Templates
        2 - Custom Template Location
Choice: 1


Choose an AWS Quick Start application template
        1 - Hello World Example
        2 - Multi-step workflow
        3 - Serverless API
        4 - Scheduled task
        5 - Standalone function
        6 - Data processing
        7 - Hello World Example With Powertools
        8 - Infrastructure event management
        9 - Serverless Connector Hello World Example
        10 - Multi-step workflow with Connectors
        11 - Lambda EFS example
        12 - DynamoDB Example
        13 - Machine Learning
Template: Template: 1

Use the most popular runtime and package type? (Python and zip) [y/N]: N

Which runtime would you like to use?
        1 - aot.dotnet7 (provided.al2)
        2 - dotnet6
        3 - dotnet5.0
        4 - dotnetcore3.1
        5 - go1.x
        6 - go (provided.al2)
        7 - graalvm.java11 (provided.al2)
        8 - graalvm.java17 (provided.al2)
        9 - java11
        10 - java8.al2
        11 - java8
        12 - nodejs18.x
        13 - nodejs16.x
        14 - nodejs14.x
        15 - nodejs12.x
        16 - python3.9
        17 - python3.8
        18 - python3.7
        19 - python3.10
        20 - ruby2.7
        21 - rust (provided.al2)
Runtime: 19

What package type would you like to use?
        1 - Zip
        2 - Image
Package type: 2

Based on your selections, the only dependency manager available is pip.
We will proceed copying the template using pip.

Would you like to enable X-Ray tracing on the function(s) in your application?  [y/N]: N

Would you like to enable monitoring using CloudWatch Application Insights?
For more info, please view https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/cloudwatch-application-insights.html [y/N]: N

Project name [sam-app]: cicd-app

    -----------------------
    Generating application:
    -----------------------
    Name: cicd-app
    Base Image: amazon/python3.10-base
    Architectures: x86_64
    Dependency Manager: pip
    Output Directory: .
    Configuration file: cicd-app/samconfig.toml

    Next steps can be found in the README file at cicd-app/README.md
    

Commands you can use next
=========================
[*] Create pipeline: cd cicd-app && sam pipeline init --bootstrap
[*] Validate SAM template: cd cicd-app && sam validate
[*] Test Function in the Cloud: cd cicd-app && sam sync --stack-name {stack-name} --watch


SAM CLI update available (1.89.0); (1.78.0 installed)
```

ディレクトリを変更して関数をビルドします。

```sh
cd cicd-app && sam build
```

※以下のようなエラーが発生した場合はDockerのデーモンが起動されていることを確認してください。

```text
Build Failed
Error: Building image for HelloWorldFunction requires Docker. is Docker running?
```

改善しない場合は環境変数`DOCKER_HOST`がセットされていない可能性があります。
`docker.sock`のパスを`DOCKER_HOST`に指定した上で`sam build`を実行します。

以下の例はMacOS上でRancher Desktopを利用している場合になりますが、

```sh
DOCKER_HOST=unix://$HOME/.rd/docker.sock sam build
```

`Build Succeeded`が表示されましたら、ビルドに成功しています。

```text
Build Succeeded

Built Artifacts  : .aws-sam/build
Built Template   : .aws-sam/build/template.yaml

Commands you can use next
=========================
[*] Validate SAM template: sam validate
[*] Invoke Function: sam local invoke
[*] Test Function in the Cloud: sam sync --stack-name {{stack-name}} --watch
[*] Deploy: sam deploy --guided
```

実際に関数を実行するために以下のコマンドを実行します。

```sh
sam local invoke HelloWorldFunction
```

うまくいかない場合は環境変数`DOCKER_HOST`を指定した状態で`sam local invoke HelloWorldFunction`を実行します。

```sh
DOCKER_HOST=unix://$HOME/.rd/docker.sock sam local invoke HelloWorldFunction
```

実行結果

```text
% sam local invoke HelloWorldFunction
Invoking Container created from helloworldfunction:python3.10-v1
Building image.................
Using local image: helloworldfunction:rapid-x86_64.

START RequestId: ffe3462d-df1e-467e-a95d-a8d6b5fd28e8 Version: $LATEST
END RequestId: ffe3462d-df1e-467e-a95d-a8d6b5fd28e8
REPORT RequestId: ffe3462d-df1e-467e-a95d-a8d6b5fd28e8  Init Duration: 1.14 ms  Duration: 545.87 ms     Billed Duration: 546 ms Memory Size: 128 MB     Max Memory Used: 128 MB
{"statusCode": 200, "body": "{\"message\": \"hello world\"}"}
%
```

### CodeCommitのリポジトリをクローンする

以下のコマンドで`codecommit.yml`をCloudFormationで実行します。

```sh
aws cloudformation deploy --stack-name codecommit --template-file ./codecommit.yml --tags Name=cicdhandson --profile cicd_handson
```

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
