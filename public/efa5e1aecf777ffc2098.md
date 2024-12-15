---
title: 【MacOS Sequoia】 MultipassインスタンスがUnknownとなる原因と解決策
tags:
  - macOS
  - multipass
private: false
updated_at: '2024-09-29T17:00:07+09:00'
id: efa5e1aecf777ffc2098
organization_url_name: null
slide: false
ignorePublish: false
---
# はじめに

:::note warn
本記事は2024/09/29時点の情報です。今後のリリースで問題が解決される可能性があるため、最新版の情報も確認してください。
:::
MacOSをSequoiaにバージョンアップ後、`multipass launch`や`multipass start`でインスタンスを起動しても、STATEが`Unknown`となる現象が発生しました。

バージョンアップ前まで普通に利用できていたので、OSのバージョンアップが原因だと考えられました。

GitHubのIssue #3661にて解決策が提示されているため、ここでまとめます。

https://github.com/canonical/multipass/issues/3661

# 環境

```:OS Version
$ sw_vers
ProductName:		macOS
ProductVersion:		15.0
BuildVersion:		24A335
```

```:Multipass Version
$ multipass version
multipass 1.14.0+mac
multipassd 1.14.0+mac
```

# 原因

上記のIssueによると、OSのバージョンアップにより`/var/db/dhcpd_leases`のフォーマットが変更され、MultipassがインスタンスのIPアドレスを正しく取得できなくなったため、インスタンスがUnknown状態になると考えられます。

# 解決策

MultipassのCollaboratorであるricabさんが上記Issue内で提供しているパッケージを適用します。手順は以下の通りです。

1. GitHub Issueの[コメント](https://github.com/canonical/multipass/issues/3661#issuecomment-2363403467)のリンクからパッケージをダウンロード
2. `.pkg`ファイルをダブルクリックしてインストール
3. セキュリティ設定で許可が必要な場合は、システム設定 > プライバシーとセキュリティから「許可する」を選択

なお、2024/09/25時点でricabさんがRCを近いうちにリリースすると発言されているため、記事作成から時間が経っている場合この問題は解決している可能性が高いです。
>Hi @HansS1964, I am glad it worked for you. If we have no further surprises, we're aiming to have a release candidate (RC) ready in approximately a couple of weeks. After that, if all is well with the RC it still takes a few days to sign and finally release.

# おわりに

上記手順で問題が解決しない場合は、公式の[トラブルシューティングガイド](https://multipass.run/docs/troubleshoot-networking)を確認するか、リリース候補版（RC）のアップデートが公開されているか確認してください。

記事を閲覧いただきありがとうございました。同じ事象が発生していた方への良い情報提供になれば幸いです。
