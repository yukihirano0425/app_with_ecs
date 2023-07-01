# part7

## 事前準備

### Builders IDの発行

[Amazon CodeCatalyst](https://codecatalyst.aws/explore)のホームページからSign upをクリックします。

メールアドレスを入力します。

![img](./img/1.png)
![img](./img/2-0.png)
![img](./img/2-1.png)
![img](./img/3-0.png)
![img](./img/3-1.png)
![img](./img/4-0.png)
![img](./img/4-1.png)
![img](./img/4-2.png)
![img](./img/5.png)

## メンション名を決める

![img](./img/6-1.png)

## スペース名を決める

![img](./img/6-2.png)
![img](./img/6-4.png)

## AWSアカウントとリンクする

![img](./img/6-5.png)
![img](./img/6-6.png)
![img](./img/6-7.png)
![img](./img/6-7.png)
![img](./img/7.png)

## プロジェクトを作成する

![img](./img/8.png)
![img](./img/9.png)
![img](./img/10.png)

## リポジトリを作成する

![img](./img/11.png)

![img](./img/12.png)

![img](./img/13.png)

## リポジトリをクローンする

catalystリポジトリをDesktopにクローンします。
CodeCatalyst の画面からクローン用のURLとパスワードをコピーします。

以下のコマンドでリポジトリをクローンできます。初回クローン時にはパスワード(Personal Access Token)を聞かれます。

```sh
git clone {コピーしたクローンのURL} ~/Desktop/catalyst
```

実行例

```sh
Desktop $ git clone https://{user_name}@git.us-west-2.codecatalyst.aws/v1/cicd-handson/handson/catalyst
Cloning into 'catalyst'...
Password for 'https://{user_name}@git.us-west-2.codecatalyst.aws/v1/cicd-handson/handson/catalyst':{ここにパスワードを貼り付け}
remote: Counting objects: 3, done.
Unpacking objects: 100% (3/3), 648 bytes | 129.00 KiB/s, done.
Desktop $
```

## ブランチを作成

ディレクトリを変更します。

```sh
cd ~/Desktop/catalyst
```

テストブランチを作成します。

```sh
git checkout -b test
```

ブランチの一覧を確認します。

```sh
git branch
```

実行結果

```text
catalyst $ git branch
  main
* test
catalyst $
```

### READM.mdを変更してリモートリポジトリにプッシュする

以下のコマンドを実行してREADMEを書き換えます。

```sh
echo "# Amazon CodeCatalyst Handson" > README.md
```

変更を反映してリポジトリにプッシュします。

```sh
git add .
git commit -am "Amazon CodeCatalyst Handson"
git push --set-upstream origin test
```

### プルリクエストを作成する
