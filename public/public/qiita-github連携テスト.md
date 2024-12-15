---
title: Qiita CLI GitHub連携 投稿テスト
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
アウトプットを習慣化したいのですが、中々できていません。
習慣化のためより楽にブログを投稿する方法を探していました。
そこでQiita CLIすると、使い慣れたエディタで記事を作成できることを知りました。
この記事はそのお試し投稿となります。

# プレビュー

`npx qiita preview`を実行すると、プレビュー画面が開かれます。
過去に投稿した他の記事をちょっとみたい時に見やすそうです。
画像をつけたい場合は、左上の`画像をアップロードする`をクリックします。
![スクリーンショット 2024-12-15 11.20.10.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3862159/cc24255b-334e-eb66-d364-dd389c3f0140.png)

## 画像のアップロード


![スクリーンショット 2024-12-15 11.21.51.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3862159/fe131e3a-2cb2-1022-d873-adc039969535.png)

画像アドレスをコピーし、エディタに貼ると画像をつけられます。
![スクリーンショット 2024-12-15 11.24.31.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3862159/29130d65-2e15-8646-d5d7-28b412c5e5ca.png)

記事の投稿は`npx qiita publish <ファイル名>`
GitHubと連携すると、リポジトリにpushすると投稿できる。

# 感想
* 初期設定は思っていたより簡単で始めやすかった
* 使い慣れたエディタが利用可能
* 他の記事を参照しやすいのが便利
* GitHubとも連携すると今後の記事の管理が楽になりそう

# 参考

https://qiita.com/Qiita/items/666e190490d0af90a92b

https://qiita.com/Qiita/items/32c79014509987541130