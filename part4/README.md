# part3

## 事前準備

[aws_happy_code - GitHub](https://github.com/ymd65536/aws_happy_code.git)をgitコマンドでDesktop上にcloneします。

### リポジトリの作成〜リモートリポジトリの更新

```sh
cd ~/Desktop/aws_happy_code/part4 && aws cloudformation deploy --stack-name codecommit --template-file ./codecommit.yml --tags Name=cicdhandson --profile cicd_handson && git clone codecommit::ap-northeast-1://cicd_handson@cicdhandson ~/Desktop/cicdhandson && cd ~/Desktop/cicdhandson && git checkout -b main && echo "Hello SAM" > README.md && git add . && git commit -m "part4" && git push --set-upstream origin main
```

### AWSのリソース作成

```sh
cd ~/Desktop/aws_happy_code/part4 && aws cloudformation deploy --stack-name s3 --template-file ./s3.yml --tags Name=cicdhandson --profile cicd_handson && aws cloudformation deploy --stack-name s3-build --template-file ./sam-s3.yml --tags Name=cicdhandson --profile cicd_handson && aws cloudformation deploy --stack-name codebuild-iam-role --template-file ./codebuild-role.yml --tags Name=cicdhandson --capabilities CAPABILITY_NAMED_IAM --profile cicd_handson && aws cloudformation deploy --stack-name event-bridge-iam-role --template-file ./event-bridge-iam-role.yml --tags Name=cicdhandson --capabilities CAPABILITY_NAMED_IAM --profile cicd_handson && aws cloudformation deploy --stack-name pipeline-iam-role --template-file ./pipeline-iam-role.yml --tags Name=cicdhandson --capabilities CAPABILITY_NAMED_IAM --profile cicd_handson && aws cloudformation deploy --stack-name code-build --template-file ./code-build.yml --tags Name=cicdhandson --profile cicd_handson && aws cloudformation deploy --stack-name pipeline --template-file ./pipeline.yml --tags Name=cicdhandson --profile cicd_handson
```

### SAMの準備

```bash
cd ~/Desktop/cicdhandson && git checkout -b sam_handson && cp ~/Desktop/aws_happy_code/part4/buildspec.yml ~/Desktop/cicdhandson/ && cp ~/Desktop/aws_happy_code/part4/SAM-Tutorial/* ~/Desktop/cicdhandson/ && git add . && git commit -m "part4" && git push --set-upstream origin sam_handson
```

### プルリクエストの作成

```bash
aws codecommit create-pull-request --title "part4" --description "part4 lambda ci/cd" --targets repositoryName=cicdhandson,sourceReference=sam_handson --profile cicd_handson
```

### ブランチをマージする

```bash
PULL_REQUEST_ID=`aws codecommit list-pull-requests --profile cicd_handson --pull-request-status OPEN --repository-name cicdhandson --query 'pullRequestIds' --output text` && echo $PULL_REQUEST_ID
```

```bash
COMMITID=`aws codecommit get-branch --repository-name cicdhandson --branch-name sam_handson --profile cicd_handson --query 'branch.commitId' --output text` && echo $COMMITID
```

```sh
aws codecommit merge-pull-request-by-fast-forward --pull-request-id $PULL_REQUEST_ID --source-commit-id $COMMITID --repository-name cicdhandson --profile cicd_handson
```

### CodeDeployでアプリケーションをデプロイ

```sh
cd ~/Desktop/cicdhandson && git checkout sam_handson && echo "# Hello CodeDeploy" > README.md && git add . && git commit -m "codedeploy" && git push -u
```

### プルリクエストを作成する(2回目)

```sh
aws codecommit create-pull-request --title "CodeDeploy" --description "codedeploy deploy" --targets repositoryName=cicdhandson,sourceReference=sam_handson --profile cicd_handson
```

プルリクエストIDを環境変数PULL_REQUEST_IDに保存する。

```sh
PULL_REQUEST_ID=`aws codecommit list-pull-requests --profile cicd_handson --pull-request-status OPEN --repository-name cicdhandson --query 'pullRequestIds' --output text` && echo $PULL_REQUEST_ID
```

コミットIDを環境変数COMMITIDに保存する。

```sh
COMMITID=`aws codecommit get-branch --repository-name cicdhandson --branch-name sam_handson --profile cicd_handson --query 'branch.commitId' --output text` && echo $COMMITID
```

### ブランチをマージする(2回目)

```sh
aws codecommit merge-pull-request-by-fast-forward --pull-request-id $PULL_REQUEST_ID --source-commit-id $COMMITID --repository-name cicdhandson --profile cicd_handson
```

## 片付け

### リソース削除

part4で作成したリソースを削除します。

```bash
python ~/Desktop/aws_happy_code/cleanup/part4.py
```

## ローカルにクローンしたリポジトリ削除

デスクトップ上にクローンしたCodeCommitのリポジトリを削除します。

```bash
rm -rf ~/Desktop/cicdhandson
```
