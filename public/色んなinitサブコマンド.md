---
title: 色んなinitサブコマンド
tags:
  - '初心者'
private: false
updated_at: ''
id: null
organization_url_name: null
slide: false
ignorePublish: false
---
# きっかけ
最近terraformを勉強しています。
モジュールを分けると、モジュールごとにterraform initをする必要があります。

terraform init → plan → applyの流れに慣れてきた頃に、
そもそもinitってなんだろうと改めて思いました。

他にもgitとかでinitサブコマンドは使われていますね。
このinitサブコマンドについて疑問に思ったので調べてみました。

# initとは？
initialize(初期化)の略
初期化、設定ファイルの生成、またはワークスペース/プロジェクトの準備を行うためのサブコマンドに使われることが一般的。

## git init
* Gitリポジトリを新規に初期化
* 現在のディレクトリに.gitディレクトリを作成
* 初期状態では、ブランチはmainとして設定され、ファイルは何も追跡されない

## terraform init
* 拡張子.tfファイルに記述されたプロバイダーや外部モジュールをダウンロード
* 状態管理に使用するバックエンドのセットアップ
* 使用するプロバイダーごとに必要なプラグイン（バイナリ）のインストール

### 使用例

https://registry.terraform.io/providers/hashicorp/aws/latest

![スクリーンショット 2024-12-18 0.38.39.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3862159/3c8bc764-03f6-0af4-c509-5d932858aeb4.png)

main.tfに最低限のprovider設定(上記画像の赤枠)を記述し、terraform initを実行してみる。

```:main.tf
terraform {
  required_providers {
    aws = {
      source = "hashicorp/aws"
      version = "5.81.0"
    }
  }
}

provider "aws" {
  # Configuration options
}
```

```:terraform init
$ terraform init
Initializing the backend...
Initializing provider plugins...
- Finding hashicorp/aws versions matching "5.81.0"...
- Installing hashicorp/aws v5.81.0...
- Installed hashicorp/aws v5.81.0 (signed by HashiCorp)
Terraform has created a lock file .terraform.lock.hcl to record the provider
selections it made above. Include this file in your version control repository
so that Terraform can guarantee to make the same selections by default when
you run "terraform init" in the future.

Terraform has been successfully initialized!

You may now begin working with Terraform. Try running "terraform plan" to see
any changes that are required for your infrastructure. All Terraform commands
should now work.

If you ever set or change modules or backend configuration for Terraform,
rerun this command to reinitialize your working directory. If you forget, other
commands will detect it and remind you to do so if necessary.
```

実行内容
* Initializing the backend
* Initializing provider plugins
* Installing hashicorp/aws v5.81.0
* Terraform has created a lock file .terraform.lock.hcl

`.terraform.lock.hcl`は、プロバイダやモジュールのバージョンとそのハッシュ を固定
```:.terraform.lock.hcl
# This file is maintained automatically by "terraform init".
# Manual edits may be lost in future updates.

provider "registry.terraform.io/hashicorp/aws" {
  version     = "5.81.0"
  constraints = "5.81.0"
  hashes = [
    "h1:YoOBDt9gdoivbUh1iGoZNqRBUdBO+PBAxpSZFeTLLYE=",
    "zh:05534adf6f02d6ec26dbeb37a4d2b6edb63f12dc9ab5cc05ab89329fcd793194",
    "zh:1d224056866abc4c8f893d55bc6493b73688126fbeaf017ecfbcf5d2f16649c4",
    "zh:486d28a0a4af2ea23964a8e9087d66e8d794e3438976633b8554684a9237499d",
    "zh:4bc17c2e93034099b64eb94eaea31b48888b6abdf170e26cf0f6ea734926084c",
    "zh:5c48c8e82fa8c410499eaa5980c0ebcf6a42360742dfd695393eb9b0bffd4232",
    "zh:60c387caa94d67e0b768f5874abbd103638c4c9b14073b6cd121018efdfc77bc",
    "zh:72ddd5e5e07aac1c1c54659df238e6490aac3abbd2e4f13ccf7a9d877c2e2d0f",
    "zh:8b03d7c4e23a51c9d323f24784d6bfd044f03e6e512df8d458abc97c943a3d3e",
    "zh:93b6a3c3299fc67d349f8ab80a9b6b65e0e9f3a7e7ea3da0cd87e3ca3b48137b",
    "zh:9982fc3885797ee97aa45ac7eba0fe6870220748bfa3091141ff513dd7583809",
    "zh:9b12af85486a96aedd8d7984b0ff811a4b42e3d88dad1a3fb4c0b580d04fa425",
    "zh:b7d60f8527dbffe11c83a05b63459d18fda921616242246a73cf3044b8732bcf",
    "zh:be7a57524298df3c377cdd676e691500277a423ac50f7b33dd02b7d6f4e924fd",
    "zh:c6ae0b1510804c705aab99659f228bdbafa663fa72ace50c811c0b9220c7dafb",
    "zh:cdf524a269b4aeb5b1f081d91f54bae967ad50d9c392073a0db1602166a48dff",
  ]
}
```

# まとめ

ChatGPTに他にinitサブコマンドを持つコマンドがないか聞いてみました。

以下に、さまざまなツールやコマンドで使用される init サブコマンドの種類とその役割を 表形式 にまとめます。

| **ツール/コマンド**        | **コマンド例**           | **役割・概要**                                                               |
|----------------------------|--------------------------|----------------------------------------------------------------------------|
| **Git**                    | `git init`              | 新しい Git リポジトリを初期化し、`.git` ディレクトリを作成します。          |
| **Terraform**              | `terraform init`        | Terraform プロジェクトを初期化し、プロバイダやモジュールをダウンロードします。|
| **npm**                    | `npm init`              | `package.json` ファイルを生成して、Node.js プロジェクトを初期化します。      |
| **Yarn**                   | `yarn init`             | Yarn を使って `package.json` を作成し、プロジェクトを初期化します。         |
| **Pulumi**                 | `pulumi init`           | Pulumi プロジェクトを初期化し、クラウド環境用の設定を準備します。           |
| **Vagrant**                | `vagrant init`          | `Vagrantfile` を生成し、仮想マシン環境の設定を初期化します。                |
| **Docker Swarm**           | `docker swarm init`     | Docker Swarm クラスターを初期化し、マネージャーノードを作成します。         |
| **Ansible Galaxy**         | `ansible-galaxy init`   | 新しい Ansible ロールのディレクトリ構造を作成します。                       |
| **Go (Go Modules)**        | `go mod init`           | Go モジュールを初期化し、`go.mod` ファイルを生成します。                    |
| **Helm (Kubernetes)**      | `helm init`             | Helm クライアントまたはサーバーを初期化します（※ Helm 2 用）。             |
| **Azure CLI**              | `az init`               | Azure CLI 環境や一部ツールの初期設定を行います。                            |
| **kubectl (Kubernetes)**   | `kubectl init`          | Kubernetes の設定を一部プラグイン経由で初期化します。                       |
| **CMake**                  | `cmake --init`          | CMake プロジェクトの構成を初期化します。                                    |

特徴まとめ
* 共通点: init サブコマンドは、ほとんどの場合 「初期化」 を意味し、プロジェクトや環境を準備する役割を果たします。
* 対象: ソフトウェア開発、インフラ管理、コンテナ、パッケージ管理、クラウド環境設定など多岐にわたります。
*	目的: 環境や依存関係の一貫性を確保し、プロジェクトの構成ファイルや設定を初期化すること。