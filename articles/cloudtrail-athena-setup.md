---
title: "CloudTrailのログをAthenaを使って調査する"
emoji: "💭"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [AWS,CloudTrail,Athena]
published: false
---

# CloudTrailのログをAthenaを使って調査する

CloudTrailからAPIの実行履歴を調査する状況は業務で多々ありますが、データベース操作に苦手意識がある方も多いと思います。
この記事では、**AWS CloudTrailのログをAmazon Athenaで効率的に検索・分析する基盤の構築手順**を、初心者にも分かりやすく解説します。

> **この記事のポイント**
> - CloudTrailコンソールからAthenaテーブルを自動作成する手順
> - Athenaでの簡単なクエリ例

---

## 目次
- [CloudTrailコンソールからAthenaテーブルを自動作成](#cloudtrailコンソールからathenaテーブルを自動作成)
- [サンプルクエリでログを調査](#サンプルクエリでログを調査)
- [まとめ](#まとめ)

---

## 1. CloudTrailコンソールからAthenaテーブルを自動作成

CloudTrailのコンソールには、Athena用のテーブルを自動作成する機能があります。

1. CloudTrailコンソールの**「イベント履歴」**画面を開きます。
2. 画面右上の**「Athenaでクエリ」**をクリックします。
3. **「Athenaテーブルを作成」**ダイアログが表示されるので、指示に従いテーブル作成を進めます。
4. Athenaのクエリエディタが自動的に開き、テーブル作成用のSQLが自動生成されます。内容を確認し、実行します。
5. テーブル作成後、**「パーティションの修復」**も自動で案内されるので、指示に従い実行します。

---

## 2. サンプルクエリでログを調査

### 例: 直近1週間の特定イベントを検索

```sql
SELECT eventTime, eventName, userIdentity.userName, sourceIPAddress
FROM <テーブル名>.cloudtrail -- <テーブル名>は自動作成時に指定したAthenaのデータベース名に置き換えてください
WHERE eventName = 'ConsoleLogin'
  AND from_iso8601_timestamp(eventTime) > current_timestamp - interval '7' day
ORDER BY eventTime DESC
LIMIT 20;
```

> 💡 **補足**
> - `<テーブル名>`はAthenaテーブル作成時に指定したデータベース名に読み替えてください。
> - クエリはAthenaのクエリエディタで実行できます。

---

## 3. まとめ

- CloudTrailからAthenaテーブルを自動作成することで、手間なくログ分析環境を構築できます。
- Athenaでのクエリにより、柔軟な調査や監査が可能です。
- コンソール上だと90日間までしか表示できないため長期間の調査となるとAthenaで調査が必要となります。

> **参考: CloudTrailログのS3バケットパス**
> CloudTrailログは通常、以下のようなS3バケットに保存されます。
> 例: `s3://your-cloudtrail-bucket/AWSLogs/************/CloudTrail/`
> - `your-cloudtrail-bucket`はCloudTrailログ用に設定したS3バケット名です。
> - `************`はアカウントID部分で、実際のIDは環境により異なります。


