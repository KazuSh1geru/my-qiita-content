# my-qiita-content


## はじめに
Qiitaの記事を管理するためのリポジトリです。

## 記事ファイルの見方
```markdown
---
title: newArticle001 # 記事のタイトル
tags:
  - "" # タグ（ブロックスタイルで複数タグを追加できます）
private: false # true: 限定共有記事 / false: 公開記事
updated_at: "" # 記事を投稿した際に自動的に記事の更新日時に変わります
id: null # 記事を投稿した際に自動的に記事のUUIDに変わります
organization_url_name: null # 関連付けるOrganizationのURL名
slide: false # true: スライドモードON / false: スライドモードOFF
ignorePublish: false # true: `publish`コマンドにおいて無視されます（Qiitaに投稿されません） / false: `publish`コマンドで処理されます（Qiitaに投稿されます）
---
```
## 記事の投稿Tips

### 伝わる記事のチェックポイント
```markdown
３つの問い
- Target: 伝える相手のこと、ちゃんとわかっている？
  - その人が気になっていることは何？
  - 理解度は？
- Before/After: 相手をどんな感情にしたい？どんな行動をとってほしい？
  - プレゼン終わった一言目相手がどんなコメントをする？
  - どんな行動をとる？
- Message: Before→Afterにするために相手に発見させたいことは何？
  - 獲得して欲しい認知は何？
  - XXではなく、YYという認知を獲得して欲しい
```

## qiita cli

### よく使うコマンド
```
# プレビュー
npx qiita preview --verbose
# 記事の投稿更新
npx qiita publish 記事のファイルのベース名
# 全記事の投稿更新
npx qiita publish --all
```

### ヘルプ
```
USAGE:
qiita <COMMAND> [<OPTIONS>]

COMMAND:
  init                    記事をGitHubで管理するための初期設定
  login                   Qiita APIの認証認可
  new [<basename>] ...    新しい記事を追加
  preview                 コンテンツをブラウザでプレビュー
  publish <basename> ...  記事を投稿、更新
  publish --all           全ての記事を投稿、更新
  pull                    記事ファイルをQiitaと同期
  version                 Qiita CLIのバージョンを表示
  help                    ヘルプを表示

OPTIONS:
  --credential <credential_dir>
    Qiita CLIの認証情報を配置するディレクトリを指定

  --root <root_dir>
    記事ファイルをダウンロードするディレクトリを指定

  --verbose
    詳細表示オプションを有効

  --config
    設定ファイルを配置するディレクトリを指定

詳細についてはReadme(https://github.com/increments/qiita-cli)をご覧ください
```