---
title: コマンド一つでQiita記事執筆に取り掛かれるようにする
tags:
  - GitHub
  - 初心者
private: false
updated_at: '2024-12-24T07:06:13+09:00'
id: 3513fdcb80375d3272c6
organization_url_name: null
slide: false
ignorePublish: false
---
# やったこと
コマンド一つで記事執筆に取り掛かれるようにしました。
* 作成する記事のプレビュー画面の起動
* エディタの編集画面の起動
```
qiita <作成する記事のタイトル>
```

![test.gif](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3862159/8a53d5b7-06f1-1b99-2f8d-70e6c8a4d283.gif)

# きっかけ

最近アウトプットができておらず、アウトプットの習慣を作りたいと考えています。
そこでQiitaの記事執筆をラクにする方法を考え、Qiita CLIやGitHubとの連携を行いました。

よりラクにしていく方法を模索し、今回の設定をしました。

# 前提

本記事の設定を行う場合、先に下記2記事の設定を実施する必要があります。

* Qiita CLIの設定

https://qiita.com/Qiita/items/666e190490d0af90a92b

* GitHubの設定

https://qiita.com/Qiita/items/32c79014509987541130

# 設定方法

シェル起動時に読み込まれる設定ファイルに下記を追加します。
私はzshを利用しているので、`~/.zshrc`に追記しました。
`~/git/qiita`の部分は、ご自身のQiitaのリポジトリのルートディレクトリに変更してください。
```
qiita(){
        cd ~/git/qiita
        npx qiita new $1
        code ~/git/qiita/public
        code ~/git/qiita/public/$1.md
        npx qiita preview
}
```

試す場合は`source ~/.zshrc`を実行してください。
シェルを再起動する場合は不要です。

# 感想
* Qiita CLIやGitHubの設定は思っていたより簡単
* 使い慣れたエディタが利用可能で執筆が捗りそう
* 他の記事を参照しやすいのが便利
* 記事の管理が楽になりそう
