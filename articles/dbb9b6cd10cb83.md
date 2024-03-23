---
title: "Neon CLI で認証できないときはブラウザを変えてみろ"
emoji: "🐕"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Neon", "トラブルシューティング"]
published: true
---

PlanetScale の無料枠終了に伴い個人開発の Database を Neon に移行したこぷらです。
Neon CLI での作業中に認証ができないときの対処法をメモしておきます。

## 環境

- macOS 14.4（23E214）
- Node.js v20.3.0
- Neon CLI 1.27.5
- Chrome 123.0.6312.59

## エラー内容

移行に伴い色々作業している際 Neon CLI を使って認証を行おうとしたら下記のエラーが発生して先に進めなくなりました。

```shell
$ neonctl auth
/usr/local/Cellar/neonctl/1.27.5/libexec/lib/node_modules/neonctl/node_modules/openid-client/lib/client.js:373
          throw new TypeError('invalid IncomingMessage method');
                ^

TypeError: invalid IncomingMessage method
    at Client.callbackParams (/usr/local/Cellar/neonctl/1.27.5/libexec/lib/node_modules/neonctl/node_modules/openid-client/lib/client.js:373:17)
    at Server.<anonymous> (file:///usr/local/Cellar/neonctl/1.27.5/libexec/lib/node_modules/neonctl/auth.js:69:44)
    at Server.emit (node:events:511:28)
    at parserOnIncoming (node:_http_server:1121:12)
    at HTTPParser.parserOnHeadersComplete (node:_http_common:119:17)

Node.js v20.3.0
```

## 対処法

エラー内容をみてもよくわからなかったので、調べてみるとブラウザの問題かもしれないという情報を見つけました。

https://github.com/neondatabase/neonctl/issues/180

> I got the auth to work by temporarily switching from Chrome to Firefox as the default browser.

とのことで、同じようにブラウザを変えてみると認証が通りました。
自分の場合は Chrome から Safari に変えてみました。

## まとめ

タイトルだけで完結するシンプルな内容に最後までお付き合いいただきありがとうございます。
解決方法が個人的にかなり予想外の方法だったので、以外と時間を取られてしまいました。
同じような症状で困っている人の参考になれば幸いです。
