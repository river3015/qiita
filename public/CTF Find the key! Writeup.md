---
title: CTF 「Find the key!」 Writeup
tags:
  - Security
  - CTF
  - seccon
private: false
updated_at: ''
id: null
organization_url_name: null
slide: false
ignorePublish: false
---
# 問題

https://gihyo.jp/book/2022/978-4-297-13180-7/support

問題ファイル: `ch5/seccon__q1_pcap.pcap`

# 解法

どんなファイルか調べる。

```:bash
$ file seccon_q1_pcap.pcap
seccon_q1_pcap.pcap: pcap capture file, microsecond ts (little-endian) - version 2.4 (Ethernet, capture length 65535)
```

パケットキャプチャしたデータのため、WireSharkで確認してみる。

![スクリーンショット 2025-10-04 14.29.15.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3862159/1378ee18-7f55-4183-8a2d-b5b242202050.png)

pingの通信だった。
色々見ていると、sequence番号が特徴的な通信があったので確認してみる。

![スクリーンショット 2025-10-04 14.40.45.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3862159/51d2d6fa-2218-4e6d-bd73-a8e7c36608a6.png)

`GET /kagi.png HTTP/1.1`という文字列が見つかるので、これがflagと関係してそうだと判断。

`strings seccon_q1_pcap.pcap`の実行結果の中で、下記の文字列を発見。

```
8GET /kagi.png HTTP/1.1
User-Agent: Wget/1.13.4 (linux-gnu)
Accept: */*
Host: localhost:8080
Connection: Keep-Alive
8GET /kagi.png HTTP/1.1
User-Agent: Wget/1.13.4 (linux-gnu)
Accept: */*
Host: localhost:8080
Connection: Keep-Alive
8HTTP/1.1 200 OK
Date: Tue, 26 Nov 2013 17:12:08 GMT
Server: Apache/2.2.15 (CentOS)
Last-Modified: Tue, 26 Nov 2013 02:52:01 GMT
ETag: "500122-aeb-4ec0b909cdaca"
Accept-Ranges: bytes
Content-Length: 2795
Connection: close
Content-Type: image/png
```
レスポンスのボディ部分を抜き出し、`kagi.png`を復元できればflagを取得できそう。
パケットを扱うためのPythonスクリプトを記述する。
参考: [中島 明日香『入門セキュリティコンテスト ― CTFを解きながら学ぶ実戦技術』翔泳社, 2023年](https://gihyo.jp/book/2022/978-4-297-13180-7)

```:python
# coding: utf-8
from scapy.all import rdpcap, Raw

packets = rdpcap('seccon_q1_pcap.pcap')
parts = []

for i in range(43, 48):
    data = packets[i].getlayer(Raw).load
    parts.append(data[28:])

icmpdata = b"".join(parts)

sig = b'\x89PNG\r\n\x1a\n'
index = icmpdata.find(sig)

key = icmpdata[index:]
with open('kagi.png', 'wb') as f:
  f.write(key)
```

`kagi.png`を復元することで、flagを取得できた。
![kagi.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/3862159/ba9fb7a4-5740-451b-b831-20064ffe82af.png)

# まとめ

ICMPペイロードの中にHTTP通信のためのデータが入るというICMP Tunnelに関する問題だった。