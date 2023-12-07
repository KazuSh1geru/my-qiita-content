---
title: gcloud cheetsheet必須チートシート
tags:
  - googlecloud
private: false
updated_at: '2023-12-07T19:24:54+09:00'
id: 87dbde68e0805376bc23
organization_url_name: null
slide: false
ignorePublish: false
---
# これは何か？
GCPのCLI `gcloud` コマンド: 必須チートシートを翻訳したものです。[原文](https://cloud.google.com/sdk/gcloud/reference/cheat-sheet)

Google Cloud Platform（GCP）は、その広範なサービスと機能により、クラウドコンピューティングの世界で欠かせない存在です。
これらのサービスを効率的に利用するためには、GCPのコマンドラインインターフェース（CLI）ツールである `gcloud` の理解が不可欠です。


```bash
gcloud cheat-sheet
```

## Personalization

GCP環境を設定を構成するための基本的なコマンドです。

- `gcloud config set`: 設定にプロパティ（例：compute/zone）を定義します。[詳細](https://cloud.google.com/sdk/gcloud/reference/config/set)
- `gcloud config get`: Google Cloud CLIのプロパティの値を取得します。[詳細](https://cloud.google.com/sdk/gcloud/reference/config/get)
- `gcloud config list`: 設定のすべてのプロパティを表示します。[詳細](https://cloud.google.com/sdk/gcloud/reference/config/list)
- `gcloud config configurations create`: 新しい名前付き設定を作成します。[詳細](https://cloud.google.com/sdk/gcloud/reference/config/configurations/create)
- `gcloud config configurations list`: 利用可能なすべての設定を表示します。[詳細](https://cloud.google.com/sdk/gcloud/reference/config/configurations/list)
- `gcloud config configurations activate`: 既存の名前付き設定に切り替えます。[詳細](https://cloud.google.com/sdk/gcloud/reference/config/configurations/activate)

## Credentials

GCPへの認証情報を管理するコマンドです。

- `gcloud auth login`: ユーザーがGoogle Cloudにログインします。[詳細](https://cloud.google.com/sdk/gcloud/reference/auth/login)
- `gcloud auth activate-service-account`: サービスアカウントを使用してGoogle Cloudにログインします。[詳細](https://cloud.google.com/sdk/gcloud/reference/auth/activate-service-account)
- `gcloud auth list`: 認証済みアカウントの一覧を表示します。[詳細](https://cloud.google.com/sdk/gcloud/reference/auth/list)
- `gcloud auth print-access-token`: 現在のアカウントのアクセストークンを表示します。[詳細](https://cloud.google.com/sdk/gcloud/reference/auth/print-access-token)
- `gcloud auth revoke`: アカウントのアクセス認証を削除します。[詳細](https://cloud.google.com/sdk/gcloud/reference/auth/revoke)

## Projects

プロジェクトのアクセスポリシーを管理します。

- `gcloud projects describe`: プロジェクトのメタデータ（IDを含む）を表示します。[詳細](https://cloud.google.com/sdk/gcloud/reference/projects/describe)
- `gcloud projects add-iam-policy-binding`: 指定されたプロジェクトにIAMポリシーを追加します。[詳細](https://cloud.google.com/sdk/gcloud/reference/projects/add-iam-policy-binding)


## Identity & Access Management (IAM)

IAMに関する操作を行うコマンド群です。

- `gcloud iam list-grantable-roles [RESOURCE]`: リソースに付与可能なIAMロールを一覧表示します。[詳細](https://cloud.google.com/sdk/gcloud/reference/iam/list-grantable-roles)
- `gcloud iam roles create`: プロジェクトまたは組織にカスタムロールを作成します。[詳細](https://cloud.google.com/sdk/gcloud/reference/iam/roles/create)
- `gcloud iam service-accounts create`: プロジェクトにサービスアカウントを作成します。[詳細](https://cloud.google.com/sdk/gcloud/reference/iam/service-accounts/create)
- `gcloud iam service-accounts add-iam-policy-binding`: サービスアカウントにIAMポリシーを追加します。[詳細](https://cloud.google.com/sdk/gcloud/reference/iam/service-accounts/add-iam-policy-binding)
- `gcloud iam service-accounts set-iam-policy`: 既存のIAMポリシーバインディングを置き換えます。[詳細](https://cloud.google.com/sdk/gcloud/reference/iam/service-accounts/set-iam-policy)
- `gcloud iam service-accounts keys list`: サービスアカウントの鍵を一覧表示します。[詳細](https://cloud.google.com/sdk/gcloud/reference/iam/service-accounts/keys/list)

## Docker & Google Kubernetes Engine (GKE)

DockerおよびGKEの管理に役立つコマンドです。

- `gcloud auth configure-docker`: `gcloud` ツールをDockerのクレデンシャルヘルパーとして登録します。[詳細](https://cloud.google.com/sdk/gcloud/reference/auth/configure-docker)
- `gcloud container clusters create`: GKEコンテナを実行するクラスターを作成します。[詳細](https://cloud.google.com/sdk/gcloud/reference/container/clusters/create)
- `gcloud container clusters list`: GKEコンテナを実行するクラスターを一覧表示します。[詳細](https://cloud.google.com/sdk/gcloud/reference/container/clusters/list)
- `gcloud container clusters get-credentials`: `kubectl` がGKEクラスターを使用するための `kubeconfig` を更新します。[詳細](https://cloud.google.com/sdk/gcloud/reference/container/clusters/get-credentials)
- `gcloud container images list-tags`: コンテナイメージのタグとダイジェストのメタデータを一覧表示します。[詳細](https://cloud.google.com/sdk/gcloud/reference/container/images/list-tags)

## Virtual Machines & Compute Engine

仮想マシンとCompute Engineの作成・実行・管理を行うコマンドです。

- `gcloud compute zones list`: Compute Engineのゾーンを一覧表示します。[詳細](https://cloud.google.com/sdk/gcloud/reference/compute/zones/list)
- `gcloud compute instances describe`: VMインスタンスの詳細を表示します。[詳細](https://cloud.google.com/sdk/gcloud/reference/compute/instances/describe)
- `gcloud compute instances list`: プロジェクト内のすべてのVMインスタンスを一覧表示します。[詳細](https://cloud.google.com/sdk/gcloud/reference/compute/instances/list)
- `gcloud compute disks snapshot`: 永続ディスクのスナップショットを作成します。[詳細](https://cloud.google.com/sdk/gcloud/reference/compute/disks/snapshot)
- `gcloud compute snapshots describe`: スナップショットの詳細を表示します。[詳細](https://cloud.google.com/sdk/g

## まとめ

この記事では、GCPのコマンドラインツール `gcloud` の主要な機能とコマンドについて概観しました。Personalization, Credentials, Projects, IAM, Docker & GKE、そしてCompute Engineなど、GCPを効率的に操作するための重要なコマンドを網羅しました。これらのコマンドは、GCPでの日常的なタスクを迅速かつ効率的に実行するための基礎となります。是非、これらのチートシートを参考にしながら、GCPの様々な機能を最大限に活用してください。GCPの使い方や応用例についてさらに学ぶことで、クラウド環境の管理がより容易になるでしょう。

ご質問やコメントがあれば、ぜひコメントセクションでお知らせください。一緒に学び、成長しましょう！
