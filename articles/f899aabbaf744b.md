---
title: "T3 Stack + Supabase でお手軽に Web 開発を始める"
emoji: "🦔"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Nextjs", "Supabase", "NextAuth", "Prisma"]
published: false
---

流行りの技術に乗っかってアプリを作ってみようと思い立ち T3 Stack + Supabase を使ってみました。

とても使いやすい組み合わせで、これから新しいプロジェクトを立ち上げようとしている人にぜひともおすすめです。

セットアップも非常に簡単でしたが、ドキュメントを行ったり来たりするのが意外と手間でしたので、一箇所にまとめたメモの残そうと想います。

これからプロジェクトを立ち上げようという方の参考になれば幸いです。

## 用語解説

今回採用した T3 Stack と Supabase についての概要となります。

それぞれより詳しい解説記事がありますので、もっと知りたい方はそちらをご覧ください。

### T3 Stack

> The *“T3 Stack”* is a web development stack made by [Theo↗](https://twitter.com/t3dotgg) focused on simplicity, modularity, and full-stack typesafety.
> 引用元: https://create.t3.gg/en/introduction

T3 Stack とはシンプルさ、モジュール性、型安全という 3 つの要素を追求した Web 開発の思想です。

この 3 つの要素を実現するために `Next.js`, `TypeScritpt`, `TailWind CSS` を使って開発しましょう。

要件に応じてバックエンドに `tRPC`, `Prisma`, `NextAuth.js`  を採用するともっと良いですよ、という思想です。

実際にこれらを使ってみるとわかりますが、非常に良い DX (開発体験) が得られます。

まだ使ったこと無いよ、という人にぜひともおすすめしたいです。

https://create.t3.gg/en/introduction

https://zenn.dev/mikinovation/articles/20220911-t3-stack

### Supabase

Subabase は OSS の BaaS (Backend as a Service) です。

DataBase に採用された PostgreSQL をベースに、リアルタイムリスナーや認証、サーバーレス関数などの機能を備えています。

感の良い人ならお気づきかもしれませんが、Firebase の代替を目指したサービスとなります。

今回こちらを採用したのは、Firestore のような NoSQL ではなく RDBMS を使いたいと考えたからです。

その上で無料プランも用意されているため、プロジェクトの立ち上げにはうってつけのサービスと言えます。

https://supabase.com/

## 環境構築の手順

それでは実際にプロジェクトを立ち上げてみます。

大きな流れは以下のとおりです。

1. `create-t3-app` を使ってテンプレートを作成
2. Supabase のプロジェクトを作成
3. `Prisma` に Supabase の Database を設定
4. `NextAuth` に任意の Provider を設定

### T3 Stack アプリの雛形作成

`create-t3-app` を使ってテンプレートからプロジェクトを作成します。

https://github.com/t3-oss/create-t3-app

オプションなどの選択可こんな感じでやりました。

```bash
> yarn create t3-app

? What will your project be called? t3-supabase-sample
? Will you be using TypeScript or JavaScript? TypeScript
Good choice! Using TypeScript!
? Which packages would you like to enable? nextAuth, prisma, trpc
? Initialize a new git repository? Yes
Nice one! Initializing repository!
? Would you like us to run 'yarn'? Yes
Alright. We'll install the dependencies for you!
? What import alias would you like configured? ~/
```

完了すると prisma で DB をセットアップするように促されますが、その前に Supabase のセットアップを行いましょう。

### Supabase でプロジェクトを作成

Supabase にて新規のプロジェクトを作成します。

プロジェクト設定の Database Password だけは後で使うのでどこかに控えておいてください。

![Create new project](/images/t3-stack-supabase/NewProject.png)

作成が完了したら、アプリとの接続のセットアップを行います。

### Prisma

https://www.prisma.io/docs/guides/database/supabase

次に Prisma にて Supabase との接続をセットアップします。

1. Supabase の設定からデータベースへの URL を取得
    1. Supabase のプロジェクトページへ移動
    2. サイドバーから設定を開く
    3. Database のページを開き ConnectionString の URI をコピー
2. `.env` へコピー
    1. YOUR-PASSWORD はプロジェクト作成時に控えておいた Database Password

    ```bash
    DATABASE_URL="postgres://postgres:[YOUR-PASSWORD]@db.[YOUR-PROJECT-REF].supabase.co:5432/postgres"
    ```

3. `./prisma/schema.prisma` の Datasource を `sqlite` から `pstgresql` に書き換える

    ```bash
    datasource db {
        provider = "postgresql"
        url      = env("DATABASE_URL")
    }
    ```

4. `yarn prisma db push` コマンドを実行して Supabase にスキーマを反映

上記が完了したら Supabase の Dashbord から Database を確認してください。

`prisna/schema.prisma` で定義されている Accout や User テーブルなどが作られているはずです。

### NextAuth

https://authjs.dev/reference/adapter/supabase

基本的な設定は `create-t3-app` だけで完結しています。

ただデフォルトの認証方法は Discord だけになっていますので、要件に合わせて任意の Provider を追加しましょう。

今回は Google の認証機能を追加します。

Google の認証機能を追加するには Client ID と Secret が必要となるので、まずは GCP で認証機能の設定から行います。

設定方法はこちらがとてもわかり易いです。

https://zenn.dev/hayato94087/articles/91179fbbe1cad4

アプリ側での設定方法が若干違うので、こちらで解説します。

1. Client ID と Secret を取得したら `.env` に追加(Discord は使わないのでついでに削除)

    ```bash
    # Next Auth Provider
    GOOGLE_CLIENT_ID="YOUR-CLIENT-ID"
    GOOGLE_CLIENT_SECRET="YOUR-CLIENT-SECRET"
    ```

2. `src/env.mjs` に `.env` に追加した Google の認証情報を追加

    ```jsx
    export const env = createEnv({
      server: {
        // ...othres
        GOOGLE_CLIENT_ID: z.string(),
        GOOGLE_CLIENT_SECRET: z.string()
      },
    
      runtimeEnv: {
        // ...others
        GOOGLE_CLIENT_ID: process.env.GOOGLE_CLIENT_ID,
        GOOGLE_CLIENT_SECRET: process.env.GOOGLE_CLIENT_SECRET
      },
    
      // ...others
    })
    ```

3. `src/server/auth.ts` の `NextAuthOptions` に `GoogleProvider` を追加

    ```jsx
    import GoogleProvider from 'next-auth/providers/google'
    
    export const authOptions: NextAuthOptions = {
      // ...others
      providers: [
       // ...others
        GoogleProvider({
          clientId: env.GOOGLE_CLIENT_ID,
          clientSecret: env.GOOGLE_CLIENT_SECRET
        })
      ]
    }
    ```

### Supabase Auth は使わないの?

Supabase にはマネージドの Authentication 機能が存在するため、 NextAuth.js を使わなくても認証機能は実装できます。

ただ、NextAuth.js を使うことでインフラの可搬性が高まります。

例えば最近サービス開始した Vercel Storage が安定版となったから変更したいとなれば、そちらにデータ移行すれば OK です。

また汎用性の高い NextAuth.js を勉強しておけば、別プロジェクトでも活用しやすいでしょう。

## 実行デモ

セットアップを行った認証機能を試してみます。

開発サーバーの立ち上げは Next.js と同じく `yarn dev` で行えます。

`http://localhost:3000` にアクセスすると `create-t3-app` のデフォルトページが開きます。

![Top page of create t3 app](/images/t3-stack-supabase/T3AppTopPage.png)

画面下部の Sign in ボタンを押すと設定した Provider の Sign in ボタンが表示されるはずです。

今回は Discord を削除して Google を追加したので、 Google のボタンだけ表示されました。

![Sign in with google](/images/t3-stack-supabase/SignInWithGoogle.png)

Sign in すると最初には表示されていなかったメッセージが表示されます。

メッセージ内にログインしたアカウント名が表示されていれば無事成功です。

![Top page of create t3 app](/images/t3-stack-supabase/T3AppTopWithAuth.png)

Supabase の方も確認しましょう。

Project Dashbord の Database を開くと Account と User テーブルにレコードが追加されているはずです。

![database dashboard](/images/t3-stack-supabase/DatabaseDashboard.png)

Tabel エディターから User テーブルを確認すると、ログインした Google アカウントの情報が追加されているはずです。

![database editor](/images/t3-stack-supabase/DatabaseUserTable.png)

※セキュリティアラートも出ていますが、今回はテスト用に作っただけなので無視しています。

## create-t3-app と Supabase で最高の開発体験を

今回は今流行の T3-Stack を Supabase と連携させたプロジェクトの立ち上げ方を紹介しました。

`create-t3-app` を使うことで面倒な初期設定の大部分を省け、Supabase との連携もドキュメントを見るだけで簡単に行えました。

実際の作業時間はとても短く、このメモを書きながらでも 1 時間ほどで完了しました。

数ステップの簡単作業で高機能なプロジェクトの雛形が作れるのは中々革命的です。

T3-Stack は人気なだけあり、非常に良い開発体験が得られますので、皆さんも是非使ってみてください。

それでは。
