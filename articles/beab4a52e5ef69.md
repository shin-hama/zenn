---
title: "Deno 2.0 にしたら VS Code プラグインによる外部モジュールの参照に失敗するようになった"
emoji: "😊"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["deno", "vscode"]
published: true
---

## TL;DR

`Node.js` とのモノレポ運用するときは、ルートディレクトリに `deno.jsonc` を作成して以下のように設定すると解決できる。

```json
{
  "nodeModulesDir": "auto"
}
```

## 背景

遅ればせながら Deno 2.0 にアップデートしました。
VS Code 上でアップデートしたのですが、アップデートした途端に外部モジュールの参照に失敗するようになりました。

![エラーメッセージ](/images/deno-failed-install/image-1.png)

```bash
> Error caching: Resolving npm specifier entrypoints this way is currently not supported with "nodeModules": "manual". In the meantime, try with --node-modules-dir=auto instead
```

## 解決策

エラーメッセージを見ると、`--node-modules-dir=auto` を指定するようにとのことでした。
どこに設定したらいいかドキュメントを探したところ `deno.jsonc` に設定すればよいということがわかりました。

https://docs.deno.com/runtime/fundamentals/configuration/#node-modules-directory

```json
{
  "nodeModulesDir": "auto"
}
```

また、ドキュメントを最後まで読んだところ、エラーが出ている原因も判明しました。

> When using workspaces, this setting can only be used in the workspace root. Specifying it in any of the members will result in warnings. The "manual" setting will only be applied automatically if there's a package.json file in the workspace root.


> #日本語訳
> ワークスペースを使用する場合、この設定はワークスペース・ルートでのみ使用できます。どのメンバーで指定しても、警告が表示されます。manual "設定は、ワークスペース・ルートにpackage.jsonファイルがある場合のみ、自動的に適用されます。

実は今回はフロントエンドを `Node.js` バックエンドを `Deno` で開発しているモノレポで運用していました。
そのときルートディレクトリに `package.json` だけを用意していたため `--node-modules-dir=manual` に設定されてしまったことが原因でした。
ディレクトリ構造は以下のようなイメージです。

```bash
|- apps/
|  |- frontend/
|    |- package.json
|  |- backend/
|    |- deno.jsonc
|- package.json
```

この場合はルートディレクトリに `deno.jsonc` を作成すればよいのですが、ファイルを不用意に増やしたくない人は VS Code の方で設定を変えることでも対応可能です。

1. `Ctrl` + `,` で設定画面を開き Workspace タブを選択
2. `@ext:denoland.vscode-deno config` を検索
3. Workspace 内の `deno.jsonc` への相対パスを入力
   1. 今回の例なら `apps/backend/deno.jsonc`

![alt text](/images/deno-failed-install/image.png)

:::message
この設定が Deno 2.0 になってどのように変更になって、アップデート後にエラーとなったたかまではわかりませんでした。
もし分かる人がいれば教えて下さい。
:::
