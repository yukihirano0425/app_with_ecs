# Part2で使うコマンド

## CloudFormationの操作

### アクセスキーの作成

```sh
aws cloudformation deploy --stack-name iam --template-file ./iam_user.yml --tags Name=cicdhandson --capabilities CAPABILITY_NAMED_IAM --profile cicd_handson
```

```sh
aws cloudformation describe-stacks --stack-name iam --profile cicd_handson
```

### CodeCommitでリポジトリを作成する

```sh
aws cloudformation deploy --stack-name codecommit --template-file ./codecommit.yml --tags Name=cicdhandson --profile cicd_handson
```

```sh
aws cloudformation describe-stacks --stack-name codecommit --profile cicd_handson
```

```sh
git clone codecommit::ap-northeast-1://cicd_handson@cicd_handson
```

## dockerfileをリモートリポジトリに追加する

```sh
git add .
git commit -m "dockerfile"
git push -u 
```

### ビルドされたイメージを確認する

CloudFormationで作成されたリポジトリを確認します。

```sh
aws ecr describe-repositories --profile {プロファイル名} --output json 
```

コマンドを実行すると以下のようにリポジトリの詳細が出力されます。

```json
{
    "repositories": [
        {
            "repositoryArn": "arn:aws:ecr:ap-northeast-1:{アカウント}:repository/cicdhandson",
            "registryId": "{アカウントID}",
            "repositoryName": "cicdhandson",
            "repositoryUri": "{アカウントID}.dkr.ecr.ap-northeast-1.amazonaws.com/cicdhandson",
            "createdAt": "2023-05-24T00:14:05+09:00",
            "imageTagMutability": "MUTABLE",
            "imageScanningConfiguration": {
                "scanOnPush": false
            },
            "encryptionConfiguration": {
                "encryptionType": "AES256"
            }
        }
    ]
}
```

イメージダイジェストを確認します。

```sh
aws ecr list-images --profile {プロファイル名} --repository-name cicdhandson --query "imageIds[*].imageTag" --output table
```

```txt
-----------------------------------------------------------------------------
|                                ListImages                                 |
+---------------------------------------------------------------------------+
|  sha256:2326ba7ae9bff1c55a618051b86c7d71401712898afe1a833734076962a231e5  |
+---------------------------------------------------------------------------+
```

イメージのタグを確認します。

```sh
aws ecr list-images --profile {プロファイル名} --repository-name cicdhandson --query "imageIds[*].imageDigest" --output table
```

```txt
------------
|ListImages|
+----------+
|  latest  |
+----------+
```

## 片付け

### CodeCommit

```sh
aws cloudformation delete-stack --stack-name codecommit --profile cicd_handson
```

## IAM

```sh
aws cloudformation delete-stack --stack-name iam --profile cicd_handson
```

### S3バケットを空にする

```sh
aws s3 ls --profile {プロファイル名} | grep cicdhandson | awk '{print $3}'
```

```sh
aws s3 rm s3://cicdhandson-bucket-{アカウント}/ --recursive --profile {プロファイル名}
```

中身を確認します。

```sh
aws s3 ls s3://cicdhandson-bucket-{アカウントID} --profile {プロファイル名}
```

```sh
aws s3 rb s3://cicdhandson-bucket-{アカウントID} --force
```
