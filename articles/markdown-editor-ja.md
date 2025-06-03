---
title: "Vite で React アプリを作り GitHub Pages で公開する" # 記事のタイトル
emoji: "📝" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech" # tech: 技術記事 / idea: アイデア記事
topics: ["react", "vite", "markdown", "githubpages"]
published: true # 公開設定（falseにすると下書き）
---

## はじめに

本記事は、開発ツール **Vite** を導入し、サンプルアプリケーションを **GitHub Pages** でウェブ上に公開するまでの一連の手順を解説します。

また本記事の内容を基に、React の学習の一環で作成した 日本語用の Markdown エディタ（下記リンク参照）を紹介できたらと思います。

https://github.com/nonaka101/markdown-editor-ja

https://nonaka101.github.io/markdown-editor-ja/

## GitHub Pages 連携までの流れ

ここでは Vite の導入から、その際 作成されるサンプルアプリを GitHub Pages で公開するまでの一連の流れを説明します。

なお Vite 導入より手前となる、下記の基礎的な部分は省略させていただきます。

- Visual Studio Code 関係（インストールや日本語化など）
- GitHub 関係（アカウント作成や VSCode との連携など）
- `npm` 関係（私は [volta](https://volta.sh/) を利用しています）

### Vite の導入

まずはプロジェクトを作成したい場所をカレントディレクトリにし、ターミナルを使って `vite` を導入します。

コマンド `npm create vite@latest` を実行すると、プロジェクト名やフレームワークなどを聞かれるので、指示に沿って入力すれば OK です。

:::message

プロジェクト名（下記だと `vite-project`）は、変更不可なものと考え、**この段階でしっかり決めておく**ことをオススメします。

ここでの名前は、後々 **GitHub のリポジトリ名 及び GitHub Pages** で使うことになります。

:::

```bash:新規プロジェクトの作成
user@penguin:~$ npm create vite@latest

> npx
> create-vite

│
◇  Project name:
│  vite-project
│
◇  Select a framework:
│  React
│
◇  Select a variant:
│  JavaScript
│
◇  Scaffolding project in /home/user/vite-project...
│
└  Done. Now run:

  cd vite-project
  npm install
  npm run dev
```

最後に、プロジェクトに移動しインストールを実行するよう指示が出ますので、それに従ってコマンドを打てば、導入が完了します。

```bash:プロジェクトに移動し必要なモジュール等をインストール
user@penguin:~$ cd vite-project
user@penguin:~/vite-project$ npm install
```

この段階でサンプルアプリが動くようになっているので、問題がないかを確認するために起動してみます。

ローカルサーバーを立ち上げプレビューするために、指示にあった `npm run dev` を実行します。

```bash:ローカルサーバーを立ち上げプレビュー
user@penguin:~/vite-project$ npm run dev

> vite-project@0.0.0 dev
> vite


  VITE v6.3.5  ready in 219 ms

  ➜  Local:   http://localhost:5173/
  ➜  Network: use --host to expose
  ➜  press h + enter to show help
```

これでローカル上（`http://localhost:5173/`）で、現在の内容で生成されたものをリアルタイムで確認できるようになりました。

「ヘルプが欲しい場合は `h` + `Enter`」とあるので、それに従ってみたのが下記です。各種ショートカットを教えてくれます。

```bash:ヘルプを表示
h

  Shortcuts
  press r + enter to restart the server
  press u + enter to show server url
  press o + enter to open in browser
  press c + enter to clear console
  press q + enter to quit
```

下記の 2 つはよく使うので、覚えておくとよいでしょう。

- `o` + `Enter` でブラウザを開く
- `q` + `Enter` で終了

ブラウザで `http://localhost:5173/` を開くと、下図のサンプルアプリ（カウンター）が表示されます。

![サンプルアプリ](/images/articles/markdown-editor-ja/vite-01.png)

ここまでできていれば、Vite の導入は完了です。

#### 導入時点のフォルダ構成

この時点でのフォルダ構成は以下の通りです。

```txt:導入時点でのフォルダ構成
vite-project/
├── node_modules/
├── public/
│   └── vite.svg
├── src/
│   ├── assets/
│   │   └── react.svg
│   ├── App.css
│   ├── App.jsx
│   ├── index.css
│   └── main.jsx
├── .gitignore
├── eslint.config.js
├── index.html
├── package-lock.json
├── package.json
├── README.md
└── vite.config.js
```

これをそのまま GitHub に上げても、GitHub Pages でサンプルアプリを表示させることはできません。

必要となる処置は、下記の 3 つになります。

- `package.json` に `"homepage"` を追加、値を GitHub Pages の URL に設定
- `vite.config.js` の `defaultConfig` に `base` を追加、値を `/リポジトリ名/` に設定
- `.github/workflows/deploy.yml` を作成し、GitHub Actions によるデプロイを設定

ここからは、これらの処置を行っていくことになります。

### GitHub での新規リポジトリ作成

GitHub にて、新規リポジトリを作成します。

- リポジトリ名は、[Vite導入](#vite-の導入)にて作成した**プロジェクト名**を使用
- 公開範囲は、*Public* を選択

![GitHub にて新規リポジトリを作成する画面](/images/articles/markdown-editor-ja/github-01.png)

:::message

公開範囲を Public に設定する理由は、GitHub Pages を利用するためです。

Private の状態で利用することもできますが、GitHub Enterprise Cloud の利用者など制限があります。詳しくは [公式ドキュメント](https://docs.github.com/ja/enterprise-cloud@latest/pages/getting-started-with-github-pages/changing-the-visibility-of-your-github-pages-site) を参照ください。

:::

新規作成した状態が、下図となります。

![リポジトリ名生成直後の画面](/images/articles/markdown-editor-ja/github-02.png)

### Pages用に修正し、GitHubにアップロード

GitHub で作成したリポジトリをクローンし、プロジェクトと結びつけます。

その後 [導入時点のフォルダ構成](#導入時点のフォルダ構成) で説明したように、GitHub Pages で公開できるよう一部のファイルを編集していきます。

#### `package.json`

まずは、`package.json` です。ここでは新規に `"homepage"` を追加し、値を GitHub Pages の URL に設定する必要があります。

GitHub Pages の URL は、サブドメインを弄らない場合、`https://{ユーザー名}.github.io/{リポジトリ名}/` となります。

下記は ユーザー名 *nonaka101*、 リポジトリ名 *vite-project* とした場合です。

```json:package.json
{
  "name": "vite-project",
  "private": true,
  "version": "0.0.0",
  "type": "module",
  "homepage": "https://nonaka101.github.io/vite-project/",  // <- GitHub Pages で生成されるサイトURL
  "scripts": {
    "dev": "vite",
    "build": "vite build",
    "lint": "eslint .",
    "preview": "vite preview"
  },
  "dependencies": {
    // 省略
  },
  "devDependencies": {
    // 省略
  }
}
```

#### `vite.config.js`

次に、`vite.config.js` です。ここでは `defaultConfig` に `base` を追加し、値を `/リポジトリ名/` に設定します。

```javascript:vite.config.js
import { defineConfig } from 'vite'
import react from '@vitejs/plugin-react'

// https://vite.dev/config/
export default defineConfig({
  plugins: [react()],
  base: '/vite-project/', // <- リポジトリ名
})
```

#### GitHub へアップロード

ここまでできたら、一旦 GitHub へプッシュします。

:::message

`.github/workflows/deploy.yml` に関しては、この後の GitHub 側で作業をすることにします。

この段階で直接 `yml` を作ってまとめてアップロードする方法もありますが、今回は説明の都合から GitHub 側の設定の中で行うようにしています。

:::

### ワークフローの作成

ここからは GitHub 側での作業となります。まずは GitHub リポジトリの「設定」タブにある **Pages** 画面に移動します。

![設定画面：GitHub Pages](/images/articles/markdown-editor-ja/github-03.png)

最初にある項目、**Build and deployment** の方式を **GitHub Actions** に変更します。

![source を GitHub Actions に変更した状態](/images/articles/markdown-editor-ja/github-04.png)

ワークフローの作成を提案されますが、ここで「 *create your own* （自身で作成）」をクリックします。

すると、`.github/workflows/` のフォルダが作られ、その中に格納するワークフローの編集画面になります。

ここで、[Vite公式にある、GitHub Pagesにデプロイするためのワークフロー](https://ja.vite.dev/guide/static-deploy.html#github-pages)を、そのまま流用します。ファイル名は `deploy.yml` としておきます。

![ワークフロー編集画面で、内容を埋めた状態](/images/articles/markdown-editor-ja/github-05.png)

:::message

`brunches` を `main` から変更する場合は、ひと手間必要になります。詳細については、[先駆者様の記事](https://zenn.dev/otaki0413/articles/react-deploy-github-pages)を参照ください。

:::

:::details ワークフロー文

```yml:.github/workflows/deploy.yml
# Simple workflow for deploying static content to GitHub Pages
name: Deploy static content to Pages

on:
  # Runs on pushes targeting the default branch
  push:
    branches: ['main']

  # Allows you to run this workflow manually from the Actions tab
  workflow_dispatch:

# Sets the GITHUB_TOKEN permissions to allow deployment to GitHub Pages
permissions:
  contents: read
  pages: write
  id-token: write

# Allow one concurrent deployment
concurrency:
  group: 'pages'
  cancel-in-progress: true

jobs:
  # Single deploy job since we're just deploying
  deploy:
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4
      - name: Set up Node
        uses: actions/setup-node@v4
        with:
          node-version: lts/*
          cache: 'npm'
      - name: Install dependencies
        run: npm ci
      - name: Build
        run: npm run build
      - name: Setup Pages
        uses: actions/configure-pages@v5
      - name: Upload artifact
        uses: actions/upload-pages-artifact@v3
        with:
          # Upload dist folder
          path: './dist'
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4
```

:::

入力が完了したら、「 *Commit changes* 」からコミットします。

![コミット確認画面](/images/articles/markdown-editor-ja/github-06.png)

#### デプロイ開始

ワークフローができたので、それに沿ってデプロイが始まります。リポジトリのトップに戻ると、「Deployment」の *github-pages* が実行中になっていることが確認できます。

![リポジトリトップの画面、デプロイが開始されたことが記されている](/images/articles/markdown-editor-ja/github-07.png)

しばらく待って再更新をかけると「Deployment」の内容が更新されます。下図のようにチェック状態になっていれば、正常に終了したことを示しています。

![1分前にデプロイが正常終了したことを示している](/images/articles/markdown-editor-ja/github-08.png)

「設定」タブにある「Pages」画面に戻ると、生成できたページの URL が表示されます。

![「Your site is live at "https://nonaka101.github.io/vite-project/"」と表示され、サイトにアクセスできます](/images/articles/markdown-editor-ja/github-09.png)

サイトに移動してみると、`npm run dev` で確認したサンプル（下図）と同じものが表示されるはずです。

![サンプルアプリ](/images/articles/markdown-editor-ja/vite-01.png)

## 作品紹介（リポジトリ `README.md` より）

ここでは、React 学習用に作成した [Markdown エディタ](https://github.com/nonaka101/markdown-editor-ja) について紹介いたします。

---

![アプリのスクリーンショット](/images/articles/markdown-editor-ja/screenshot.png =250x)

本アプリはモバイル端末での利用を想定し、（全角文字を使用する）日本語環境下での、Markdown 文書作成を補助する目的で作成しました。

現在、下記の特徴を持っています。

- 各種 Markdown 要素は、ブロック単位で管理
- 作成した内容は `.md` ファイル またはクリップボードに出力可能
- 作成した内容は Web ページとして出力でき、ブラウザの機能を使い PDF として保存可能
- 編集内容自体を `.json` ファイルとして保存可能

また 本アプリは、デスクトップ及びモバイル端末にインストールすることが可能です。インストールすると、ホーム画面からアプリとして呼び出すことができます。

![PWA としてインストールする](/images/articles/markdown-editor-ja/install-01.png)

### 基本画面

基本的な画面は、下図のようになります。

![新規作成時のアプリの初期画面](/images/articles/markdown-editor-ja/base-01.png)

- 画面上部のヘッダーには、文書タイトルとメニューボタンが配置されています
- メインとなる部分では、文書内容をブロック単位で管理しています

### ブロックについて

本アプリでは、文書内容をブロック単位で管理しています。出力する際には、各ブロックに対応した markdown 要素として変換することになります。

基本的なブロックは、下記のような構造です。「↑」「↓」でブロックを上下に移動、「削除」でブロックを削除することができます。

![基本的なブロック、テキスト入力と「↑」「↓」「削除」「挿入」ボタンがある](/images/articles/markdown-editor-ja/paragraph-01.png)

ブロック追加は、ブロック末尾の「挿入」ボタンから行います。ボタンを押すと、直下に挿入するブロックを指定するダイアログが表示されます。

![「追加するブロックを選択」ダイアログ](/images/articles/markdown-editor-ja/add-block-01.png)

### 各種ブロックについて

ここでは、文書に挿入可能な各種ブロックを説明します。

#### 見出し

![見出しブロック、初期状態はレベル2](/images/articles/markdown-editor-ja/heading-01.png)

文章のタイトルや章立てを示すものです。

ブロック内にある「レベル」から、見出し構造を **2 〜 5** の範囲で調整することができます。

![見出しブロックのレベルを5にした状態、見出しの文字がレベル2より小さくなる](/images/articles/markdown-editor-ja/heading-02.png)

対応する markdown 文は、下記になります。

```md
## 見出しレベル２

### 見出しレベル３

#### 見出しレベル４

##### 見出しレベル５
```

:::message

見出しレベル１は、**文書内に 1 つのみ設定できる要素**としており、別の場所（[ドキュメント設定](#ドキュメント設定)）で設定します。

:::

#### 文章関係

![文章ブロック、中に大量のテキストが含まれた状態](/images/articles/markdown-editor-ja/extend-01.png)

ここでは文章など、**複数行に渡って**作成できる要素を説明します。

なお、ここでの各種入力ボックスは、下記の方法で拡張できます。

- ボックス端に表示されているツマミから調整（デスクトップなど）
- ボックスをダブルクリックまたはダブルタップ（モバイル端末など）

![文章ブロック、入力ボックスを拡張し全体が見えるようになった状態](/images/articles/markdown-editor-ja/extend-02.png)

##### 文章

![文章ブロック](/images/articles/markdown-editor-ja/paragraph-01.png)

普通の文字で書かれる、一般的な文章のことです。なお、連続した改行 2 つで別段落として取り扱います。

対応する markdown 文は、下記になります。

```md
段落 1 の文章

段落 2 の文章
```

##### 引用

![引用ブロック](/images/articles/markdown-editor-ja/blockquote-01.png)

他者の言葉や文章を借りてきて表示するものです。なお、連続した改行 2 つで別段落として取り扱います。

対応する markdown 文は、下記になります。

```md
> 段落 1 の引用文
>
> 段落 2 の引用文
```

##### コード

![コードブロック](/images/articles/markdown-editor-ja/code-01.png)

プログラムのソースコードやコマンドなどを そのまま表示するものです。

ブロック内にある「言語」から、プログラミング言語の名前を記述することもできます。ここでの設定は、下記の対応する markdown 文 `language名` の箇所で反映されます。

````md
```language名
コード本文
```
````

#### リスト関係

![リストブロック](/images/articles/markdown-editor-ja/list-01.png)

ここではリスト関係の要素について説明します。

:::message

内部の入力アイテムにフォーカスがある場合、下記のキー入力で特殊な操作が行えます。

- `Enter`：新規アイテムを挿入します
- `Backspace`：アイテムを削除します（入力が空の状態の時）

:::

##### 順序付きリスト

![順序付きリストブロック](/images/articles/markdown-editor-ja/list-02.png)

数字を使って、*順番に意味のある* 項目を並べるものです。

対応する markdown 文は、下記になります。

```md
1. アイテム
2. アイテム
3. アイテム
```

##### 順序なしリスト

![順序なしリストブロック](/images/articles/markdown-editor-ja/list-03.png)

点や記号を使って、*順番に関係なく* 項目を並べるものです。

対応する markdown 文は、下記になります。

```md
- アイテム
- アイテム
- アイテム
```

#### 区切り線

![区切り線ブロック](/images/articles/markdown-editor-ja/hr-01.png)

話題を変えるときなどに、水平な線で文章を区切るものです。

対応する markdown 文は、下記になります。

```md
---
```

### メニュー

ここでは、「メニュー」ダイアログの機能を説明します。このダイアログは、基本画面のヘッダーにある「メニュー」ボタンから呼び出すことができます。

![メニューダイアログ](/images/articles/markdown-editor-ja/menu-01.png)

#### ドキュメント設定

![メニューダイアログのドキュメント設定部分](/images/articles/markdown-editor-ja/menu-02.png)

エディタ全体にかかる内容を取り扱います。ここでは 2 つの設定項目があります。

1. 文書タイトル
2. カラーモード

##### 文書タイトル

文書のタイトルとなる文を設定します。

ここで設定された内容は、*エディタ画面上部のタイトル要素* や *Markdown 文での見出しレベル１* に利用されます。

##### カラーモード

「ライト」「ダーク」間でカラーモードを切り替えることができます。

:::message

「デフォルト」では、デバイス内で設定されているカラーモードを参照し、自動的に切り替えます。

:::

#### 操作ツール

![メニューダイアログの操作ツール設定部分](/images/articles/markdown-editor-ja/menu-03.png)

文書に対して各種操作をすることができます。ここでは 6 つの操作ボタンがあります。

- 新規作成
- プレビュー
- クリップボードにコピー
- ファイルとして保存
- 編集データを保存
- 編集データを読込

##### 新規作成

既存のデータを消去し、新規に文章を作成します。

:::message

新規作成で削除されるまで、文書はデバイス内に保存[^1]されます。

:::

[^1]: データとしては JSON 形式で、`localStorage` 内に `markdownEditorContent` の名前で保存しています

##### プレビュー

![プレビュー画面。ヘッダーに加え、見出しスキップ用の目次が表示されている](/images/articles/markdown-editor-ja/preview-01.png)

![プレビュー画面。コンテンツ本体となる部分が表示されている](/images/articles/markdown-editor-ja/preview-02.png)

現在の編集内容を確認するために、内容を Web ページとして生成し表示します。

ブラウザに搭載されている印刷機能を使うことで、**PDF として出力**することも可能です。

![Web ページの印刷設定画面](/images/articles/markdown-editor-ja/print-01.png)

印刷時、一部の背景色が閲覧時と異なる場合があります。環境によっては、詳細設定「背景のグラフィックス」を有効化することで解決します。

![Web ページの印刷設定画面、背景のグラフィックス設定を有効化した状態で、背景色が適切に反映されている](/images/articles/markdown-editor-ja/print-02.png)

##### クリップボードにコピー

Markdown 形式に変換した文書を、クリップボードに格納します。

##### ファイルとして保存

Markdown 形式に変換した文書を、ファイルとして保存します（拡張子 `.md` ）

![md形式でのファイル保存画面](/images/articles/markdown-editor-ja/menu-03-01.png)

##### 編集データを保存

編集データ自体を、ファイルとして保存します。（拡張子 `.json` ）

Markdown 形式へ出力せず、編集内容自体を管理したい場合に使用します。

![json形式でのファイル保存画面](/images/articles/markdown-editor-ja/menu-03-02.png)

##### 編集データを読込

編集データを読み込み、内容を復元します。（拡張子 `.json` ）

![ファイル読み込み画面](/images/articles/markdown-editor-ja/menu-03-03.png)

#### 見出しジャンプ

![見出しブロック一覧が、リンク要素として羅列されている](/images/articles/markdown-editor-ja/menu-04.png)

文書に「 [見出し](#見出し)構造」がある場合、一覧化された見出し要素が表示されます。

ここでの要素とブロックは紐づけされており、選択すると該当要素に移動することができます。

#### 本アプリについて

![本アプリについての説明文、及び GitHub リポジトリへのリンク](/images/articles/markdown-editor-ja/menu-05.png)

アプリについての簡単な説明が記されています。

### 注意事項

あくまで簡易的なエディタなので、例えば下記のような操作は行えません。

- リスト要素のネスト（リスト要素内に、サブリストを作成する）
- 下記の要素の作成補助（※自身で直接打ち込む分は可能です）
  - 強調（`*`, `**`）
  - リンク（`[リンク文](リンク先)`）
  - 画像（`![alt文](参照パス)`）

### ライセンス

本リポジトリは [MIT ライセンス](./LICENSE) となっています。

また本リポジトリでは、各ブラウザ間のスタイル差をなくすため `normalize.css`（MIT ライセンス）を使用しています。

- [Normalize.css: Make browsers render all elements more consistently.](https://necolas.github.io/normalize.css/)
- [normalize.css/LICENSE.md at master · necolas/normalize.css · GitHub](https://github.com/necolas/normalize.css/blob/master/LICENSE.md)

## 雑記

### 記事作成のきっかけ

新しい分野を学習する際、私はまず手応えを確認するために、図書館などで書籍に触れてみるようにしています。React の学習時にも同じようにしていたのですが、触れた書籍では `create-react-app` を使ったコースになっていました。

しかし 2025 年 2 月に、Create React App は廃止を発表し、フレームワークまたはビルドツールへの移行を推奨しているのが現状です。

https://react.dev/blog/2025/02/14/sunsetting-create-react-app

ビルドツールの移行先として **Vite** が推奨されていますが、GitHub Pages で公開するまでに試行錯誤することになりました。

本記事は、その過程を備忘録として残すものです。

### 何故エディタを作成することにしたか

学習内容を何かしら作ってアウトプットしてみることにした時、何を作ろうかという壁に当たりました。考えを markdown で書き出していた際の入力の手間（スマホやタブレットで、全角半角を都度切り替え）が、本アプリ作成のきっかけとなりました。

Gemini を利用しながらの作成でしたが、React の基本的な部分についての理解ができてきたと感じています。

ただ 今になって振り返ると、「インライン要素（例：\`code\` や \*\*強調\*\*）をどう入力するか」が課題として残ったので、WYSIWYGエディタの方向性で考えたほうがよかったのかもしれません。

### `create-react-app` との差

`create-react-app` と `vite` では、基本的にはだいたい一緒と考えて差し支えありません。多少異なる部分（`.jsx` 形式でないとだめだったり ビルドやプレビューのコマンドなど）はありますが、`create-react-app` で作成したものを `vite` へと移行するのに そこまでの手間はありません。

ただし `index.html` の扱いについては、一度引っかかったので、メモとして残しておきます。

#### `index.html` について

`create-react-app` の場合、`index.html` は `public` フォルダ内にあります。

ファビコン等も `public` 内で管理することになるのですが、呼び出しは `./favicon.ico` ではなく、特殊な形式（`%PUBLIC_URL%`）を使います。

`public` 内には他にもアイコンやマニフェストを格納しているため、`index.html` は下記のようになります。

```html:index.html(create-react-app)
<link rel="icon" href="%PUBLIC_URL%/favicon.ico" />
<link rel="apple-touch-icon" href="%PUBLIC_URL%/logo192.png" />
<!--
  manifest.json provides metadata used when your web app is installed on a
  user's mobile device or desktop. See https://developers.google.com/web/fundamentals/web-app-manifest/
-->
<link rel="manifest" href="%PUBLIC_URL%/manifest.json" />
<!--
  Notice the use of %PUBLIC_URL% in the tags above.
  It will be replaced with the URL of the `public` folder during the build.
  Only files inside the `public` folder can be referenced from the HTML.

  Unlike "/favicon.ico" or "favicon.ico", "%PUBLIC_URL%/favicon.ico" will
  work correctly both with client-side routing and a non-root public URL.
  Learn how to configure a non-root public URL by running `npm run build`.
-->
```

一方 `vite` では、`index.html` はプロジェクト直下に配置されています。

また、`public` フォルダにあるデータ（例：`manifest.json`）を指定する場合、`public/manifest.json` ではなく、`/manifest.json` にする必要があります。

```html:index.html(vite)
<link rel="icon" type="image/svg+xml" href="/favicon.ico" />
<link rel="apple-touch-icon" href="/logo192.png" />
<link rel="manifest" href="/manifest.json" />
```

### その他

#### PWA 化に関して

アイコン用の画像は、『[Image Generator | PWA builder](https://www.pwabuilder.com/imageGenerator)』を利用しました。各プラットフォーム用の画像を生成し、`manifest.json` の `icon` 用の JSON データも出力してくれます。

## まとめ

この文書では、Vite で作成したサンプルアプリケーションを GitHub Pages で公開するための具体的な手順を、プロジェクトの初期設定からデプロイまで順を追って説明しました。`package.json` や `vite.config.js` の編集、GitHub リポジトリの作成と設定、そして GitHub Actions を用いた自動デプロイワークフローの構築方法について解説しました。

さらに、このプロセスを通じて作成・公開した Markdown エディタの機能や特徴、開発の背景にある `create-react-app` から `vite` への移行の経緯、`index.html` の扱いの違いといった技術的な知見についても触れました。

この記録が、同様の環境で開発・公開を行う方の一助となれば幸いです。

### 参考文献

#### Vite

https://vite.dev/

- [静的サイトのデプロイ](https://ja.vite.dev/guide/static-deploy.html)
- [静的アセットの取り扱い](https://ja.vite.dev/guide/assets.html)

#### その他参考文献

https://zenn.dev/otaki0413/articles/react-deploy-github-pages

https://qiita.com/otohusan/items/39160086ceec63c5b04a
