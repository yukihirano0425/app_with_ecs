# aws_happy_code

## CodeシリーズでハッピーなCI/CDを構築しよう

## Python環境のアクティベート
python3 -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt

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

## delete-resources

リソースを削除する方法(AWS SDK for Pythonを利用)

### part2

```bash
cd ~/Desktop/aws_happy_code/cleanup
python part2.py
```

### part3

```bash
cd ~/Desktop/aws_happy_code/cleanup
python part3.py
```

### part4

```bash
cd ~/Desktop/aws_happy_code/cleanup
python part4.py
```
