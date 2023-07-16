# aws_happy_code

## CodeシリーズでハッピーなCI/CDを構築しよう

## 章立て

- (Part0) はじめにと本書で扱うサービス説明
- (Part1) CodeCommitを使ってみる
- (Part2) Docker コンテナのイメージを自動でビルドしてレジストリに保存する構成
- (Part3) Elastic Container Registry のイメージを使ってLambdaをデプロイする構成
- (Part4) Elastic Container Registry のイメージを使ってLambdaをデプロイする構成 - CodeDeployを利用
- (Part5) App Runnerを使ってCI/CDを構成
- (Part6) CodeArtifactを利用する構成
- (Part7) CodeCatalystを利用する構成
- (Part8) CodeGuru ReviewerでCodeCommitを自動レビューする構成
- (Part9) 総括

## リソースを削除する手順

1. CodePipeline
2. EventBridge
3. CodeBuild
4. IAMロール
5. S3
6. ECR
7. CodeCommit

App Runnerを使っている場合(Part5)はApp Runnerの削除も忘れずにやること。

## CodePipelineリソースの削除

```sh
CF_STACK_NAME=pipeline && CF_STACK_STATUS=`aws cloudformation describe-stacks --stack-name $CF_STACK_NAME --query "Stacks[0].StackStatus" --output json --profile cicd_handson` && echo $CF_STACK_STATUS && aws cloudformation delete-stack --stack-name $CF_STACK_NAME --profile cicd_handson
```

```sh
aws cloudformation describe-stacks --query "Stacks[*].StackName" --output json --profile cicd_handson
```

## CodeBuildリソースの削除

```sh
CF_STACK_NAME=code-build && CF_STACK_STATUS=`aws cloudformation describe-stacks --stack-name $CF_STACK_NAME --query "Stacks[0].StackStatus" --output json --profile cicd_handson` && echo $CF_STACK_STATUS && aws cloudformation delete-stack --stack-name $CF_STACK_NAME --profile cicd_handson
```

```sh
aws cloudformation describe-stacks --query "Stacks[*].StackName" --output json --profile cicd_handson
```

## IAMリソースの削除

```sh
CF_STACK_NAME=pipeline-iam-role && CF_STACK_STATUS=`aws cloudformation describe-stacks --stack-name $CF_STACK_NAME --query "Stacks[0].StackStatus" --output json --profile cicd_handson` && echo $CF_STACK_STATUS && aws cloudformation delete-stack --stack-name $CF_STACK_NAME --profile cicd_handson

CF_STACK_NAME=codebuild-iam-role && CF_STACK_STATUS=`aws cloudformation describe-stacks --stack-name $CF_STACK_NAME --query "Stacks[0].StackStatus" --output json --profile cicd_handson` && echo $CF_STACK_STATUS && aws cloudformation delete-stack --stack-name $CF_STACK_NAME --profile cicd_handson

CF_STACK_NAME=event-bridge-iam-role && CF_STACK_STATUS=`aws cloudformation describe-stacks --stack-name $CF_STACK_NAME --query "Stacks[0].StackStatus" --output json --profile cicd_handson` && echo $CF_STACK_STATUS && aws cloudformation delete-stack --stack-name $CF_STACK_NAME --profile cicd_handson
```

```sh
aws cloudformation describe-stacks --query "Stacks[*].StackName" --output json --profile cicd_handson
```

## S3バケットの削除

バケット名を確認します。

```sh
S3_BUCKET_NAME=s3
S3_ARN=$(aws cloudformation describe-stacks --stack-name s3 --query "Stacks[*].Outputs[?OutputKey==\`Name\`].OutputValue | [0] | [0]" --profile cicd_handson) && echo $S3_ARN
```

表示されたバケット名を使って、S3の中身に空にします。

```sh
aws s3 rm s3://cicdhandson-bucket-{アカウントID} --recursive --profile cicd_handson
```

表示されたバケット名を使って、S3バケット上のファイルを全て削除します。

```sh
aws s3 rm s3://sam-build-bucket-{アカウントID} --recursive --profile cicd_handson
```

{S3に関してはここから下はコンソールで作業}

`permanently delete`と入力してS3バケットを空にします。

## SAM用S3バケットの削除

```sh
S3_BUCKET_NAME=s3-build
S3_ARN=$(aws cloudformation describe-stacks --stack-name $S3_BUCKET_NAME --query "Stacks[*].Outputs[?OutputKey==\`Name\`].OutputValue | [0] | [0]" --profile cicd_handson) && echo $S3_ARN
```

表示されたバケット名を使って、S3バケット上のファイルを全て削除します。

```sh
aws s3 rm s3://sam-build-bucket-{アカウントID} --recursive --profile cicd_handson
```

{S3に関してはここから下はコンソールで作業}

`permanently delete`と入力してS3バケットを空にします。

## CodeCommitリソースの削除

```sh
CF_STACK_NAME=codecommit && CF_STACK_STATUS=`aws cloudformation describe-stacks --stack-name $CF_STACK_NAME --query "Stacks[0].StackStatus" --output json --profile cicd_handson` && echo $CF_STACK_STATUS && aws cloudformation delete-stack --stack-name $CF_STACK_NAME --profile cicd_handson
```

```sh
aws cloudformation describe-stacks --query "Stacks[*].StackName" --output json --profile cicd_handson
```
