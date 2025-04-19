---
title: "【AWS入門】AWS CLIでスイッチロールを使いこなそう！セキュアな権限管理の完全ガイド"
emoji: "🔄"
type: "tech"
topics: ["aws", "awscli", "iam", "初心者", "スイッチロール"]
published: false
---

## 👋 はじめに：スイッチロールで複数アカウントを楽々管理

「別のAWSアカウントのリソースを操作したいけど、いちいちコンソールでロールを切り替えるのが面倒...」

「CLIでスイッチロールってどうやるんだろう？設定ファイルの書き方がよくわからない...」

そんな悩みを抱えていませんか？実は、AWS CLIを使えば、コマンド一つでスイッチロールができるんです。

:::message
この記事では、AWS CLIでのスイッチロール機能の使い方を、私の経験を交えながら詳しく解説します。
特に、セキュリティ面での注意点は、実際のトラブル事例も含めて丁寧に説明していきますよ。

📝 **この記事の所要時間: 約20分**
:::

## 🎯 この記事で学べること

この記事を読み終わると、次のことができるようになります：

- [x] AWS CLIでのスイッチロールの基本概念を理解できる
- [x] 設定ファイルを適切に管理できる
- [x] セキュアなスイッチロールの設定ができる
- [x] 実践的なユースケースを実装できる

## 👥 こんな方におすすめ

「この記事、自分に合ってるかな？」と迷っている方へ。
以下のような方にピッタリの内容です：

- 複数のAWSアカウントを管理している方
- AWS CLIでの操作を効率化したい方
- セキュアなクロスアカウントアクセスを実現したい方
- IAMロールの基本は理解している方

## ⚠️ 重要な注意点

スイッチロールを使用する際の重要なポイントをまとめました：

:::message alert
1. **アクセス権限の排他性**
   - IAMユーザーの権限とロールの権限は合体しません（これ、意外と知られていないんですよね）
   - 同時に使える権限は1つだけ。ロールを切り替えると、元の権限は一時的に無効になります
   - つまり、「あれもこれも」はできないということです

2. **セッション制限**
   - デフォルトでは1時間で切れちゃいます
   - 設定で15分から最大12時間まで調整できます（`duration-seconds`パラメータを使います）
   - ロールの連鎖（ロールからロールへの切り替え）の場合は、最長1時間までしか設定できません

3. **監査とセキュリティ**
   - 管理者はCloudTrailで全ての操作を追跡できます（やりすぎには注意！）
   - 「誰が何をしたか」を明確にするため、ソースIDやセッション名の指定が求められることも
   - CloudTrailには`AWS-CLI-session-nnnnnnnn`という形式で記録が残ります
:::

## 📋 前提条件

まずは必要なものを確認していきましょう。

:::details チェックリスト
- [ ] AWS IAMロールとスイッチロールの基本を理解していること
  - IAMロールって何？という方は、まずは[AWS公式のIAM入門](https://aws.amazon.com/jp/iam/)をチェック！
  - 信頼ポリシーとアクセス権限ポリシーの違いが分かること
  - クロスアカウントアクセスの仕組みを理解していること
- [ ] AWS CLIがインストールされていること
- [ ] AWS CLIの基本的な設定が完了していること
- [ ] スイッチロール元となるIAMユーザーまたはロールが存在すること
:::

## 🔧 AWS CLIの設定ファイル構造

AWS CLIでスイッチロールを使うには、2つの設定ファイルを理解する必要があります。最初は私も混乱しましたが、それぞれの役割を理解すれば、そんなに難しくありません。

### 1. ~/.aws/credentials

このファイルは認証情報の金庫みたいなものです。アクセスキーなどの大切な情報は、全部ここに集約します。

:::message
🔐 **セキュリティ重要ポイント**
- このファイルのパーミッションは必ず600に設定しましょう
- アクセスキーは定期的にローテーションしましょう
- Git等でうっかり公開しないよう注意しましょう
:::

```ini:~/.aws/credentials
# デフォルトプロファイル（普段使いの認証情報）
[default]
aws_access_key_id = YOUR_ACCESS_KEY
aws_secret_access_key = YOUR_SECRET_KEY

# 開発環境用プロファイル
[dev]
aws_access_key_id = DEV_ACCESS_KEY
aws_secret_access_key = DEV_SECRET_KEY

# プロダクション環境用プロファイル（スイッチロール設定）
[prod]
role_arn = arn:aws:iam::123456789012:role/AdminRole
source_profile = default
```

:::details credentialsファイルの重要な設定項目
各設定項目には、それぞれ重要な役割があります：

- `aws_access_key_id`: AWSアクセスキーID（これがないと始まりません）
- `aws_secret_access_key`: AWSシークレットアクセスキー（絶対に外部に漏らさないでください！）
- `role_arn`: スイッチ先のロールのARN（Amazon Resource Name）
- `source_profile`: スイッチロール時の元となるプロファイル
- `external_id`: クロスアカウントロール用の外部ID（パートナー連携時などに使用）
:::

### 2. ~/.aws/config

こちらは、AWS CLIの動作設定を管理するファイルです。リージョンやデフォルトの出力形式など、一般的な設定はここで行います。

```ini:~/.aws/config
# デフォルトプロファイル
[profile default]
region = ap-northeast-1
output = json

# 開発環境用プロファイル
[profile dev]
region = ap-northeast-1
output = json

# プロダクション環境用プロファイル（詳細設定）
[profile prod]
region = ap-northeast-1
output = json
role_arn = arn:aws:iam::123456789012:role/AdminRole
source_profile = default
duration_seconds = 3600
```

:::details configファイルの重要な設定項目
各設定の意味を詳しく見ていきましょう：

- `region`: デフォルトのAWSリージョン（普段使う地域を設定）
- `output`: 出力形式（json, text, tableから選べます）
- `role_session_name`: スイッチロール時のセッション名（監査時に重要）
- `duration_seconds`: ロールセッションの有効期間（デフォルト1時間）
- `credential_source`: EC2インスタンスメタデータを使用する場合の設定
:::

## 💡 実践的なユースケース

実際の現場でよくあるシナリオを見ていきましょう。

### 1. 開発環境から本番環境へのスイッチロール

開発環境で作業していて、本番環境のリソースを確認したい...でもいちいちコンソールを切り替えるのは面倒ですよね？そんなときはこう設定します：

```ini:~/.aws/config
[profile production]
role_arn = arn:aws:iam::222222222222:role/ProductionRole
source_profile = default
duration_seconds = 3600
```

:::details 使用例
```bash
# 本番環境のS3バケット一覧を表示
aws s3 ls --profile production

# 環境変数で一時的にプロファイルを切り替え
export AWS_PROFILE=production
aws s3 ls
```
:::

### 2. EC2インスタンスプロファイルからのスイッチロール

EC2インスタンス上でスイッチロールを使いたい場合もありますよね。その場合はこんな感じです：

```ini:~/.aws/config
[profile instancerole]
role_arn = arn:aws:iam::222222222222:role/CrossAccountRole
credential_source = Ec2InstanceMetadata
```

:::details 使用例
```bash
# EC2インスタンスプロファイルの認証情報を使ってスイッチロール
aws s3 ls --profile instancerole
```
:::

### 3. クロスアカウントアクセス（外部ID使用）

パートナー企業のAWSアカウントにアクセスする必要がある場合。セキュリティを考慮して外部IDを使用するのがベストプラクティスです：

```ini:~/.aws/config
[profile partner]
role_arn = arn:aws:iam::333333333333:role/PartnerRole
source_profile = default
external_id = unique-external-id-123
```

:::details 使用例
```bash
# パートナーアカウントのリソースにアクセス
aws resourcegroupstaggingapi get-resources --profile partner
```
:::

## 🔒 セキュリティのベストプラクティス

セキュリティは妥協できません。以下のポイントは必ず押さえておきましょう：

:::message alert
### 1. 最小権限の原則
- 必要最小限の権限だけを付与（過剰な権限は危険です！）
- 定期的に権限を見直す（使っていない権限は削除）
- ロールの信頼関係は慎重に設定（誰でも使えるロールは危険）

### 2. セッション管理
- セッション期間は必要な分だけに（長すぎるのは危険）
- 長期的な認証情報は避ける（一時的な認証情報を使おう）
- `duration-seconds`は適切に設定（デフォルトの1時間から調整）

### 3. 監査と追跡
- CloudTrailで操作ログをしっかり監視
- ロールセッション名で誰が何をしたか追跡
:::

## 🚨 トラブルシューティング

よくあるエラーと解決方法を紹介します：

### 1. 認証エラー
```bash
An error occurred (AccessDenied) when calling the AssumeRole operation
```

:::message
あ、このエラー...私も何度も見ました。以下を確認しましょう：
- IAMロールの信頼関係の設定は正しい？
- ソースプロファイルの権限は十分？
:::

### 2. 設定ファイルの問題
```bash
Unable to locate credentials
```

:::message
このエラーが出たら、こちらをチェック：
- 設定ファイルのパスは正しい？（`~/.aws/`配下にある？）
- プロファイル名のタイプミスは？（大文字小文字も要確認）
- `credential_source`の設定は適切？
:::

## 📝 まとめ

AWS CLIでのスイッチロール、最初は複雑に見えますが、基本を押さえれば意外と簡単です。この記事で紹介した設定方法とベストプラクティスを参考に、ぜひ実践してみてください。

:::message
特に重要なのは：
- 設定ファイルの役割を理解すること
- セキュリティを常に意識すること
- 適切なプロファイル設定を心がけること
:::

次は、AWS CLIを使った自動化やスクリプティングについても解説する予定です。お楽しみに！

## 📚 参考資料

- [IAM ロールに切り替える (AWS CLI)](https://docs.aws.amazon.com/ja_jp/IAM/latest/UserGuide/id_roles_use_switch-role-cli.html)
- [AWS CLIの設定と認証情報ファイル](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-files.html)
- [AWS CLIでのスイッチロールの使用](https://dev.classmethod.jp/articles/switch-role-with-awscli/) 