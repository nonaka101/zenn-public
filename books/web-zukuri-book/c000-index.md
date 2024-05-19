---
title: "🚧🗂️C000：第三章概要"
---

## この章について

ここでは、前章までで作成した内容を使い、「WordPressテーマ化する流れ」について説明します。  

## この節の内容について

### C001：静的ページからのテーマ作成

- 各ページで共通している箇所について
- テーマ化にあたっての段階的構造
  1. テンプレートパーツ関係（土台）
     - `loop-post.php`
     - `parts-categories-and-archives.php`
     - `parts-devutils.php`
     - `parts-menu.php`
     - `parts-pagenation.php`
  2. テンプレートパーツ（土台を使用）
     - `sidebar-left.php`
     - `sidebar-right.php`
     - `searchform.php`
     - `footer.php`
     - `header.php`
  3. ページテンプレート
     - `front-page.php`
     - `404.php`
     - `singular.php`
     - `search.php`
     - `archive.php`
     - `index.php`

### C002：主に使うテンプレートタグと関数集

### C003：構成

- wordpressの設定について（スラッグなど）
- ページテンプレートへの分割
- assets フォルダ
- functions.php について
- index.phpの作成（確認用に各種パーツのみ反映）
- [テストデータ](https://github.com/jawordpressorg/theme-test-data-ja/tree/master)の準備

- 各種ベースデータから共通部を抜き出す
- functions.phpの設定
- アクションフックの挿入
- カスタムロゴ
- カスタムメニュー
- 検索ページへのリンク

## 同章の他の節について

ここでは、他の節においてどのようなものを取り上げているかを簡単に解説します。

### 🚧工事中
