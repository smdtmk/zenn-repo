---
title: "Amazon Q CLIで麻雀何切る問題のゲームを作ってみた"
emoji: "🎮"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [AWS, AmazonQ, ゲーム開発, JavaScript]
published: true
---

# Amazon Q CLIで麻雀何切る問題のゲームを作ってみた

AWSコミュニティで開催されている「Build Games with Amazon Q CLI」キャンペーンに参加し、Amazon Q CLIを活用して麻雀何切る問題ゲームを開発しました。この記事では、キャンペーンの概要と開発したゲームについて解説します。


> **この記事のポイント**
> - AWSの「Build Games with Amazon Q CLI」キャンペーンについて
> - Amazon Q CLIを活用した麻雀何切る問題ゲームの開発過程
> - 開発で得られた知見と学び

---

## 目次
- [AWSの「Build Games with Amazon Q CLI」キャンペーンとは](#awsのbuild-games-with-amazon-q-cliキャンペーンとは)
- [開発した麻雀ゲームの概要](#開発した麻雀ゲームの概要)
- [Amazon Q CLIを活用した開発プロセス](#amazon-q-cliを活用した開発プロセス)
- [実装のポイント](#実装のポイント)
- [まとめ](#まとめ)

---

## AWSの「Build Games with Amazon Q CLI」キャンペーンとは

AWSコミュニティが主催する「Build Games with Amazon Q CLI」キャンペーンは、Amazon Q CLIを活用してゲームを開発し、その過程をブログや動画で共有するという取り組みです。このキャンペーンは2025年5月20日から6月20日まで開催され、アジア太平洋地域、日本、大中華圏の参加者が対象となっています。

キャンペーンの主な目的は：
- Amazon Q CLIの実践的な活用経験を得る
- クリエイティビティを活かしたゲーム開発
- コミュニティとの知識共有

---

## 開発した麻雀ゲームの概要

今回開発したのは、シンプルな麻雀何切る問題ゲームです。主な機能は以下の通りです：

- 基本的な麻雀のルール実装
- 直感的なUI/UX
- レスポンシブデザイン
- デバッグモードの実装

技術スタック：
- フロントエンド: HTML5, CSS3, JavaScript
- 画像生成: Python (PIL)
- バージョン管理: Git

> **ソースコード**
> 本プロジェクトのソースコードは以下のGitHubリポジトリで公開しています：
> [https://github.com/smdtmk/mahjong-game](https://github.com/smdtmk/mahjong-game)

![](https://storage.googleapis.com/zenn-user-upload/463a09462eb0-20250527.png)
---

## Amazon Q CLIを活用した開発プロセス

Amazon Q CLIを活用することで、以下のような効率的な開発が可能になりました：

1. **コード生成のサポート**
   - 基本的なゲームロジックの実装
   - UIコンポーネントの設計
   - エラーハンドリングの実装

2. **デバッグ支援**
   - エラーの原因特定
   - パフォーマンス最適化の提案
   - コードの品質改善

3. **ドキュメント生成**
   - READMEの作成
   - コードコメントの生成
   - API仕様の文書化

---

## まとめ

Amazon Q CLIを活用したゲーム開発を通じて、以下のような学びを得ることができました：

- AIを活用した効率的な開発プロセス
- コードの品質向上と保守性の確保
- コミュニティとの知識共有の重要性

このキャンペーンは、AWSの最新技術を実践的に学びながら、クリエイティブな開発を行う素晴らしい機会となりました。

> **参考リンク**
> - [Build Games with Amazon Q CLI キャンペーン](https://community.aws/content/2xIoduO0xhkhUApQpVUIqBFGmAc/build-games-with-amazon-q-cli-and-score-a-t-shirt)
> - [Amazon Q CLI ドキュメント](https://docs.aws.amazon.com/q/)
> - [GitHubリポジトリ](https://github.com/smdtmk/mahjong-game)
