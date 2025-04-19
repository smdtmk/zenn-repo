---
title: "【AWS入門】AWS CLIのインストールと設定ガイド - 初心者でもできる3ステップ解説"
emoji: "⚡️"
type: "tech"
topics: ["aws", "awscli", "初心者", "インフラ", "開発環境"]
published: false
---

## 👋 はじめに：AWS CLIで開発効率を劇的に改善

「AWSの操作、毎回コンソール画面ポチポチするの大変だな...」

そう感じたことはありませんか？実は私も最初はそうでした。でも、AWS CLIと出会ってから、その悩みは一気に解消されたんです。

:::message
この記事では、AWS CLIのセットアップから実践的な使い方まで、私自身の経験を交えながら解説していきます。
特に、セキュリティ面でのベストプラクティスは、実際のトラブル事例も含めて詳しく説明しますよ。

📝 **この記事の所要時間: 約15分**
:::

## 🤔 AWS CLIとは？コマンドラインでAWSを操作する魔法のツール

AWS CLI（Command Line Interface）は、AWSのサービスをコマンドラインから操作できる便利なツールです。最初は「CLIって難しそう...」と尻込みしてしまうかもしれませんが、慣れてくると次のようなメリットを実感できます：

- **コマンドラインからの快適操作**: ブラウザを開かなくても、ターミナルからサクッとAWSを操作できます
- **自動化の強い味方**: シェルスクリプトと組み合わせれば、面倒な作業も自動化できちゃいます
- **一貫した操作感**: すべてのAWSサービスを同じような感覚で操作できるんです
- **マルチプラットフォーム対応**: Windows、macOS、Linuxのどの環境でも使えます

:::message alert
ただし、使い始める前に知っておいてほしいことが...
AWS CLIを使うにはAWSアカウントとアクセスキーが必要です。
セキュリティは特に気を付けたいポイントなので、この記事では安全な設定方法もしっかり解説していきますね。
:::

## 🎯 この記事で学べること

この記事を読み終わると、次のことができるようになります：

- [x] AWS CLIを安全にインストールして設定できる
- [x] AWS認証情報を適切に管理できる
- [x] 基本的なAWSコマンドを使いこなせる
- [x] セキュリティのベストプラクティスを実践できる

## 👥 こんな方におすすめ

「この記事、自分に合ってるかな？」と迷っている方へ。
以下のような方にピッタリの内容です：

- AWSをもっと効率的に操作したい方
- コマンドラインでの操作に興味がある方
- 自動化の第一歩を踏み出したい方
- セキュアな開発環境を目指している方

## 📋 準備するもの

まずは必要なものを確認していきましょう。

### 必要な環境

```bash:requirements.txt
# バージョンはこちらを参考に
AWS CLI: 2.x
Python: 3.x（AWS CLIの動作に必要です）
```

### 事前に用意するもの

:::details チェックリスト
- [ ] AWSアカウント
  - IAMユーザーを作れる権限が必要です
  - アクセスキーを発行できる権限も必要です
- [ ] ターミナル
  - macOSなら「ターミナル.app」か「iTerm2」
  - Linuxならお好みのターミナル
  - Windowsなら「PowerShell」か「コマンドプロンプト」
:::

## 🔧 セットアップ手順

それでは実際の設定に入っていきましょう。私も最初は戸惑いましたが、順番に進めれば意外と簡単です。

### 1. AWS CLIのインストール

OSごとにインストール方法が異なります。ご使用の環境に合わせて進めてください。

:::details OS別インストール手順
#### macOSの場合
```bash
# Homebrewを使うのが一番楽ちんです
brew install awscli

# または公式インストーラーを使う場合（こちらでも問題ありません）
curl "https://awscli.amazonaws.com/AWSCLIV2.pkg" -o "AWSCLIV2.pkg"
sudo installer -pkg AWSCLIV2.pkg -target /
```

#### Linuxの場合
```bash
# Amazon Linux 2 / RHEL系の場合
sudo yum install -y aws-cli-v2

# Ubuntu / Debian系の場合
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
```

#### Windowsの場合
```powershell
# MSIインストーラーをダウンロードして実行
Invoke-WebRequest -Uri "https://awscli.amazonaws.com/AWSCLIV2.msi" -OutFile "AWSCLIV2.msi"
Start-Process msiexec.exe -Wait -ArgumentList '/i AWSCLIV2.msi /quiet'
```
:::

### 2. IAMユーザーとアクセスキーの準備

ここが一番重要なポイントです。セキュリティ面で特に気を付けたいところをお伝えします。

:::message alert
🚨 **セキュリティ重要ポイント**
- 必要最小限の権限だけを持つIAMユーザーを作りましょう
- アクセスキーは定期的に更新する習慣をつけましょう
- 本番環境では必ずMFA（多要素認証）を有効にしましょう
:::

1. IAMユーザーの作成
   - AWSマネジメントコンソールにログイン
   - IAMサービスに移動（画面上部の検索バーで「IAM」と入力するのが早いです）
   - 「ユーザー」→「ユーザーを追加」を選択
   - 適切な権限を付与（最初は`PowerUserAccess`がおすすめです）

2. アクセスキーの作成
   - 作成したユーザーの詳細画面で「セキュリティ認証情報」タブを選択
   - 「アクセスキーを作成」をクリック
   - アクセスキーIDとシークレットアクセスキーは必ず安全な場所に保存してください

### 3. AWS CLIの初期設定

ここからが実際の設定作業です。落ち着いて一つずつ進めていきましょう。

```bash
# まずは設定ウィザードを起動します
aws configure

# 以下の情報を順番に入力していきます
AWS Access Key ID [None]: YOUR_ACCESS_KEY_ID
AWS Secret Access Key [None]: YOUR_SECRET_ACCESS_KEY
Default region name [None]: ap-northeast-1  # 東京リージョンの場合
Default output format [None]: json  # JSONフォーマットがおすすめです
```

:::message
💡 **プロ技: 複数の環境を使い分けたい場合**
開発環境と本番環境を分けて管理したい場合は、プロファイルを使うと便利です。

```bash
# 開発環境用のプロファイルを作成
aws configure --profile dev

# 本番環境用のプロファイルを作成
aws configure --profile prod
```
:::

### 4. 認証情報の安全な管理

セキュリティの要！認証情報の管理方法をしっかり押さえましょう。

#### 認証情報ファイルの保護
```bash
# 認証情報ファイルのパーミッションを適切に設定
chmod 600 ~/.aws/credentials
chmod 600 ~/.aws/config
```

#### 環境変数の活用
```bash
# 一時的な認証情報として環境変数を使うのもアリです
export AWS_ACCESS_KEY_ID="YOUR_ACCESS_KEY_ID"
export AWS_SECRET_ACCESS_KEY="YOUR_SECRET_ACCESS_KEY"
export AWS_DEFAULT_REGION="ap-northeast-1"
```

### 5. 動作確認

さあ、実際に動くか確認してみましょう！

```bash
# バージョン確認（これが一番シンプル）
aws --version

# 認証確認（これで自分のアカウント情報が見えれば成功です）
aws sts get-caller-identity

# リージョン一覧の取得（これも基本的なコマンドです）
aws ec2 describe-regions
```

## 🚨 トラブルシューティング

最初は誰でもつまずくポイントがあります。私も何度も経験したトラブルと解決策を共有しますね。

:::details よくあるエラーと対処法
1. 認証エラーが出た場合
   ```
   An error occurred (UnauthorizedOperation) when calling the DescribeInstances operation: Unauthorized operation
   ```
   - 原因：IAMの権限が足りていないことが多いです
   - 対処：IAMポリシーを見直してみましょう

2. リージョンエラーが出た場合
   ```
   You must specify a region. You can also configure your region by running "aws configure".
   ```
   - 原因：リージョンが設定されていません
   - 対処：`aws configure`で設定しましょう

3. プロファイルのエラーが出た場合
   ```
   The config profile (profile-name) could not be found
   ```
   - 原因：プロファイル名の指定ミスです
   - 対処：`~/.aws/config`の内容を確認してみましょう
:::

## 🔒 セキュリティベストプラクティス

最後に、セキュリティ面でのベストプラクティスをまとめておきます。

1. アクセスキーは定期的に更新しましょう
```bash
# 新しいアクセスキーを作成
aws iam create-access-key

# 古いアクセスキーを無効化
aws iam update-access-key --access-key-id OLD_ACCESS_KEY_ID --status Inactive

# 古いアクセスキーを削除
aws iam delete-access-key --access-key-id OLD_ACCESS_KEY_ID
```

2. MFAは必ず有効にしましょう
```bash
# MFAデバイスの一覧を確認
aws iam list-virtual-mfa-devices

# MFAデバイスを有効化
aws iam enable-mfa-device
```

## 📝 まとめ：AWS CLIで開発効率を上げよう！

:::message
AWS CLIのセットアップ、いかがでしたか？
ポイントをおさらいしておきましょう：

1. 適切なバージョンのAWS CLIをインストール
2. 最小権限の原則に基づいたIAMユーザーの作成
3. 認証情報は厳重に管理
4. 定期的なセキュリティメンテナンス

AWS CLIを使いこなせば、AWSの操作がグッと楽になりますよ！
:::

## 📚 参考文献

より詳しく知りたい方は、以下のドキュメントもチェックしてみてください：

1. [AWS CLI公式ドキュメント](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-welcome.html)
2. [AWS IAMベストプラクティス](https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html)
3. [AWS CLIセキュリティベストプラクティス](https://docs.aws.amazon.com/cli/latest/userguide/cli-security.html)

---

:::details 🔄 更新履歴
- 2024/03/XX: 記事作成
:::

:::message
**🎉 次のステップ**：
- AWS CLIを使った自動化スクリプトの作成方法
- より高度なセキュリティ設定のテクニック
:::