# Part2

## 事前準備

[aws_happy_code - GitHub](https://github.com/ymd65536/aws_happy_code.git)をgitコマンドでDesktop上にcloneします。

## ハンズオンで使うコマンド

### リポジトリの作成〜リモートリポジトリの更新

```sh
aws cloudformation deploy --stack-name codecommit --template-file ./codecommit.yml --tags Name=cicdhandson --profile cicd_handson && git clone codecommit::ap-northeast-1://cicd_handson@cicdhandson ~/Desktop/cicdhandson && cd ~/Desktop/cicdhandson && git checkout -b main && echo "Hello CodeBuild" > README.md && git add . && git commit -m "part2" && git push --set-upstream origin main && git checkout -b code_build_handson && cp ~/Desktop/aws_happy_code/part2/buildspec.yml ~/Desktop/cicdhandson/ && cp ~/Desktop/aws_happy_code/part2/dockerfile ~/Desktop/cicdhandson/ && git add . && git commit -m "part2" && git push --set-upstream origin code_build_handson
```

### AWSのリソース作成

```sh
cd ~/Desktop/aws_happy_code/part2 && aws cloudformation deploy --stack-name s3 --template-file ./s3.yml --tags Name=cicdhandson --profile cicd_handson && aws cloudformation deploy --stack-name ecr --template-file ./ecr.yml --tags Name=cicdhandson --profile cicd_handson && aws cloudformation deploy --stack-name codebuild-iam-role --template-file ./codebuild-role.yml --tags Name=cicdhandson --capabilities CAPABILITY_NAMED_IAM --profile cicd_handson && aws cloudformation deploy --stack-name event-bridge-iam-role --template-file ./event-bridge-iam-role.yml --tags Name=cicdhandson --capabilities CAPABILITY_NAMED_IAM --profile cicd_handson && aws cloudformation deploy --stack-name pipeline-iam-role --template-file ./pipeline-iam-role.yml --tags Name=cicdhandson --capabilities CAPABILITY_NAMED_IAM --profile cicd_handson && aws cloudformation deploy --stack-name code-build --template-file ./code-build.yml --tags Name=cicdhandson --profile cicd_handson && aws cloudformation deploy --stack-name pipeline --template-file ./pipeline.yml --tags Name=cicdhandson --profile cicd_handson
```

### CodeCommitでプルリクエストを作成

```bash
aws codecommit create-pull-request --title "part2" --description "part2 image ci/cd" --targets repositoryName=cicdhandson,sourceReference=code_build_handson --profile cicd_handson 
```

### マージリクエスト

```bash
PULL_REQUEST_ID=`aws codecommit list-pull-requests --profile cicd_handson --pull-request-status OPEN --repository-name cicdhandson --query 'pullRequestIds' --output text` && echo $PULL_REQUEST_ID

```bash
COMMITID=`aws codecommit get-branch --repository-name cicdhandson --branch-name code_build_handson --profile cicd_handson --query 'branch.commitId' --output text` && echo $COMMITID
```

```bash
aws codecommit merge-pull-request-by-fast-forward --pull-request-id $PULL_REQUEST_ID --source-commit-id $COMMITID --repository-name cicdhandson --profile cicd_handson
```

### ECRイメージのチェック

```bash
aws ecr describe-repositories --profile cicd_handson --output json
aws ecr list-images --profile cicd_handson --repository-name cicdhandson --query "imageIds[*].imageDigest" --output table
```

## 片付け

### リソース削除

part2で作成したリソースを削除します。

```bash
python ~/Desktop/aws_happy_code/cleanup/part2.py
```

## ローカルにクローンしたリポジトリ削除

デスクトップ上にクローンしたCodeCommitのリポジトリを削除します。

```bash
rm -rf ~/Desktop/cicdhandson
```
