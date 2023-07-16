# Part2

## 事前準備

[aws_happy_code - GitHub](https://github.com/ymd65536/aws_happy_code.git)をgitコマンドでDesktop上にcloneします。

## トラブルシューティング

## コマンド一覧

CodeCommitを構築する。

```sh
aws cloudformation deploy --stack-name codecommit --template-file ./codecommit.yml --tags Name=cicdhandson --profile cicd_handson
```

Desktop上にリポジトリをクローンする。

```sh
git clone codecommit::ap-northeast-1://cicd_handson@cicdhandson ~/Desktop/cicdhandson
```

ディレクトリを移動します。

```sh
cd ~/Desktop/cicdhandson
```

mainブランチを作成します。

```sh
git checkout -b main
```

```sh
git add .
git commit -m "part2"
```

```sh
git push --set-upstream origin main
```

```sh
git add .
git commit -m "part2"
```

```sh
git push --set-upstream origin code_build_handson
```

```sh
aws cloudformation deploy --stack-name s3 --template-file ./s3.yml --tags Name=cicdhandson --profile cicd_handson
```

```sh
aws cloudformation deploy --stack-name ecr --template-file ./ecr.yml --tags Name=cicdhandson --profile cicd_handson
```

```sh
aws cloudformation deploy --stack-name codebuild-iam-role --template-file ./codebuild-role.yml --tags Name=cicdhandson --capabilities CAPABILITY_NAMED_IAM --profile cicd_handson && aws cloudformation deploy --stack-name event-bridge-iam-role --template-file ./event-bridge-iam-role.yml --tags Name=cicdhandson --capabilities CAPABILITY_NAMED_IAM --profile cicd_handson && aws cloudformation deploy --stack-name pipeline-iam-role --template-file ./pipeline-iam-role.yml --tags Name=cicdhandson --capabilities CAPABILITY_NAMED_IAM --profile cicd_handson
```

```sh
aws cloudformation deploy --stack-name code-build --template-file ./code-build.yml --tags Name=cicdhandson --profile cicd_handson
```

```sh
aws cloudformation deploy --stack-name pipeline --template-file ./pipeline.yml --tags Name=cicdhandson --profile cicd_handson
```

```sh
aws codecommit create-pull-request --title "part2" --description "part2 image ci/cd" --targets repositoryName=cicdhandson,sourceReference=code_build_handson --profile cicd_handson
```

```sh
PULL_REQUEST_ID=`aws codecommit list-pull-requests --profile cicd_handson --pull-request-status OPEN --repository-name cicdhandson --query 'pullRequestIds' --output text` && echo $PULL_REQUEST_ID
```

```sh
COMMITID=`aws codecommit get-branch --repository-name cicdhandson --branch-name code_build_handson --profile cicd_handson --query 'branch.commitId' --output text` && echo $COMMITID
```

```sh
aws codecommit merge-pull-request-by-fast-forward --pull-request-id $PULL_REQUEST_ID --source-commit-id $COMMITID --repository-name cicdhandson --profile cicd_handson
```

```sh
aws ecr describe-repositories --profile cicd_handson --output json 
```

```sh
aws ecr list-images --profile cicd_handson --repository-name cicdhandson --query "imageIds[*].imageDigest" --output table
```
