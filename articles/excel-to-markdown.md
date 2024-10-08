---
title: "表形式データを markdown テーブルに変換" # 記事のタイトル
emoji: "📊" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech" # tech: 技術記事 / idea: アイデア記事
topics: ["javascript", "tsv", "markdown", "excel", "googlesheets"] # タグ。["markdown", "rust", "aws"]のように指定する
published: true # 公開設定（falseにすると下書き）
---

## はじめに

本記事では「表計算ソフト（例：**Microsoft Excel** や **Google Sheets** など）のデータを、**Markdown テーブル** に変換する」ことを目的とし、その実装を考えていきます。なお 実際の動き（利用）としては、下記を想定しています。

1. 表データをコピーする
2. **クリップボードを経由**して、テキスト形式（≒ `tsv`）でフォームに貼り付け
3. フォーム内容を基に、`markdown` テーブルを作成

[クリップボードについての話](#クリップボードについて)は、最下段に補足として乗せておきます。

:::message alert

本記事の逆パターン [markdown から表形式データに変換](https://zenn.dev/nonaka101/articles/markdown-to-excel) の関係で、エスケープシーケンスの扱いが少し複雑になっています。あちらの処理では、**文字列リテラル**で処理する場合と**生の文字列**で処理する場合で、結果が異なってくるのが原因です。

本記事では 向こうに合わせる形で、**文字列リテラル**を使った処理を中心に説明が進みます。向こう側で `form` 要素や `String.raw` で**生の文字列**を使える場合は、[実際に使用する際の注意事項](#実際に使用する際の注意事項)の項目に その場合のコード等を置いていますのでご参照ください。  
（※ エスケープ処理を調整するだけなので、設計や作成等の基本的な流れは一緒になります）

:::

### その他紹介

本記事と逆のパターンも用意していますので、興味がある方は下記記事を参照してください。

https://zenn.dev/nonaka101/articles/markdown-to-excel

なお 本記事で扱った内容は、下記場所にて使っています。

https://nonaka101.github.io/jig-a/

## 事前準備

ここでは設計を始める前段階として、目的や条件等の詳細を考えていきます。

### 環境の定義

本記事では JavaScript を使って処理していきます。

なお JavaScript 特有の機能を使うわけではないので、他言語でも [設計](#設計) や[作成](#作成) の内容を流用することは可能です。本記事で主に必要となる機能は、下記のとおりです。

- 「改行コード」や「タブ文字」といった**特殊文字**を扱う
- **配列**を操作する（要素数の取得や、分割・統合・挿入など）

### 条件の定義

ここでは、実装する機能の**入出力データをどうするか**について定義します。

#### 入力値となる表形式データはタブ文字と改行コードによるものとする

表形式のデータと言っても、Excel や GoogleSheets など様々あります。下図は GoogleSheets を使った場合のデータ（表形式＋テキスト）です。

![３✕３の表データ、内容については下記参照](/images/articles/excel-to-markdown/excel2markdown-01.png)

```text:上図の表形式データをテキストにした場合
表	列A	列B
行1	セルA1	セルB1
行2	セルA2	セルB2
```

上記テキストは、**タブが列の区切り**、**改行が行の区切り**で表現されています。そのため タブを `\t`、改行を `\n` に置き換えると、下記のようになります。

```text:上記例の正規表現版
表\t列A\t列B\n行1\tセルA1\tセルB1\n行2\tセルA2\tセルB2
```

よって 入力されるデータについては、下記のように定義します。

- 各行は改行コード（例：`\n`）によって区切られる
- 各列はタブ文字（例：`\t`）によって区切られる
- セル結合は考慮しない

##### 空白セルが含まれる入力を想定する

更に、入力に使われるデータはこうした綺麗なデータのみとは限りません。例えば下図は、中央の**セルが空白**になった状況を表しています。

![これまでの表データで、中央のセルが空白になっている](/images/articles/excel-to-markdown/excel2markdown-03.png)

こうしたデータも入力として想定し、出力の際に行列の関係が崩れることなく処理することが求められます。

#### 出力する文字列はmarkdown形式のテーブルとする

出力される文字列について、`markdown` のテーブル形式となるようにします。下図は、先程の入力を `markdown` に変換した結果（テーブル形式＋テキストデータ）です。

| 表 | 列A | 列B |
| --- | --- | --- |
| 行1 | セルA1 | セルB1 |
| 行2 | セルA2 | セルB2 |

```text:上図テーブルのテキスト
| 表 | 列A | 列B |
| --- | --- | --- |
| 行1 | セルA1 | セルB1 |
| 行2 | セルA2 | セルB2 |
```

このことから 出力されるデータについて、下記のように定義します。

- 表の見出し（ヘッダー）は、1行目を使用する（≒クロス集計表のような、行列組み合わせた表は想定しない）
- 最小限の構成で出力する
  - 左右揃え、中央揃え等を考慮しない
  - スペースやハイフンを使った見栄え調整（例：テキスト上での列幅を揃える）は行わない

#### オプション項目：出力時のエスケープ処理について

上記の[入力](#入力値となる表形式データはタブ文字と改行コードによるものとする)や[出力](#出力する文字列はmarkdown形式のテーブルとする)に関する条件は、機能として必須のものとなっています。ここでは、必須ではない努力項目として**markdown ルールに抵触する文字**についての処理を定義します。

下記は、基本となる入出力の例です。**タブによる区切り**が**パイプ**(`|`)**による区切り**に変わっていることが確認できます。

```text:基本的な入出力処理例
表	列A	列B
行1	セルA1	セルB1
行2	セルA2	セルB2

  ↓ 正常に出力される

| 表 | 列A | 列B |
| --- | --- | --- |
| 行1 | セルA1 | セルB1 |
| 行2 | セルA2 | セルB2 |
```

では **セル内にパイプ記号が含まれた場合**はどうなるでしょうか？

![これまでの表データで、中央のセルに特殊文字のパイプが使われている](/images/articles/excel-to-markdown/excel2markdown-02.png)

```text:セル内にパイプ記号が含まれるとテーブル構成が崩れてしまう
表	列A	列B
行1	セルA1	セルB1
行2	セルA2	セルB2

  ↓ A1セルが分割されてしまう

| 表 | 列A | 列B |
| --- | --- | --- |
| 行1 | セル|A1 | セルB1 |
| 行2 | セルA2 | セルB2 |
```

この場合、`markdown` では 「セル|A1」を「セル」「A1」と分割してしまい、正しい形にはなりません。これは `|` が**セルを区切る役割**と**セル内の単なる文字列**のどちらであるのか、区別できないためです。

これを区別させるには、**特殊な意味を持たない、単なる文字である**ことを伝えるために**エスケープ処理**を挟む必要があります。

| 表 | 列A | 列B |
| --- | --- | --- |
| 行1 | セル\\|A1 | セルB1 |
| 行2 | セルA2 | セルB2 |

```text:エスケープ処理して正しい形にした上図テーブル
| 表 | 列A | 列B |
| --- | --- | --- |
| 行1 | セル\\|A1 | セルB1 |
| 行2 | セルA2 | セルB2 |
```

このように、**markdown出力時に影響を及ぼす文字に対し、エスケープ処理を行う**ことをオプション項目として条件に設定しておきます。

## 設計

ここからは[事前準備](#事前準備)で決めた内容を基に、どのように実装していくかを考えていきます。大きな流れとしては、必須のものとして変換処理があり、その前後にオプションとして無害化と復元処理が入る形となります。具体的には、下記となります。

1. 処理に不都合な文字列を無害化（オプション）
2. 変換処理
   1. 特殊文字の表現を統一
   2. 二次元配列の形で格納
   3. 列数を調べ、セパレート行の文字列を生成
   4. 各行処理（一次配列を `|` で繋げて文字列化し、両端に `|` を付ける）
   5. 2行目にセパレート行を挿入し、改行コードで繋げて 1 つの文字列に
3. 1番で施した処理を、エスケープ処理した形で元に戻して完成（オプション）

2番の「変換処理」を ひとまとめにしているのは、後の[作成](#作成)段階で 一連の処理を関数化するためです。**変換処理を行う関数**をベースとして、それをラップする形で**エスケープ処理付き変換関数**を作成することになります。

### 1：処理に不都合な文字列を無害化（オプション）

まずは、セル内に含まれている `|` などを無害化しておきます。後の処理で `markdown` テーブルのセル区切りとしての `|` が出てきてしまうので、識別できる内に何とかしなければなりません。

ここでは、**別の文字列に置き換える**ことで無害化します。例えば `|` は `@@PIPE@@` といった、**通常使われることのない文字列に置換**しておきます。処理の最後で復元する必要があるので、無害化する文字種が複数ある場合は区別できるようしておかなければなりません。

### 2：変換処理

ここからは、必須条件となる「表形式データを `markdown` に変換」する処理となります。

#### 2-1：特殊文字の表現を統一

無害化処理が終わったら、後の処理に備えて特殊文字の表現を統一しておきます。例えば改行コードは OS によって、LF(`\n`) や CRLF(`\r\n`) のように表記ブレが起きるケースがあります。

これを片方に合わせておくことで、後の処理をスムーズにしておきます。

#### 2-2：文字列を二次元配列に

ここでは表形式データの文字列を、配列形式で格納します。中身は**改行コードとタブ文字**の組み合わせでデータが分けられているので、それを利用し二次元配列の形式に変換します。

```text:タブが列、改行コードが行の区切りとなっている
表	列A	列B
行1	セルA1	セルB1
行2	セルA2	セルB2
```

```javascript:特殊文字から二次元配列に変換
[
  ['表', '列A', '列B'],
  ['行1', 'セルA1', 'セルB1'],
  ['行2', 'セルA2', 'セルB2'],
]
```

:::message

この配列化の際、空欄セルも空要素として存在しておく必要があります。つまり**各行を表すそれぞれの一次配列は、全て同じ要素数でなければなりません**。

![これまでの表データで、中央のセルが空白になっている](/images/articles/excel-to-markdown/excel2markdown-03.png)

```javascript:空白セルも空要素として二次元配列に
[
  ['表', '列A', '列B'],
  ['行1', '', 'セルB1'],
  ['行2', 'セルA2', 'セルB2'],
]
```

:::

#### 2-3：列数からセパレート行文字列を作成

配列に格納したことで、表の列数がわかります。ここでは配列の最初の要素の数を調べ、列数を調べます。

```javascript:最初の要素に含まれる数から3列であることを確認
[
  ['表', '列A', '列B'],
  ['行1', 'セルA1', 'セルB1'],
  ['行2', 'セルA2', 'セルB2'],
]
```

列数がわかれば、`markdown` テーブルのセパレート行を作れます。セパレート行とは、表頭（ヘッダー）と表体（ボディ）の間にある、`|---|---|---|` 形式の文字列です。

```text:2行目がセパレート行
| 表 | 列A | 列B |
| --- | --- | --- |
| 行1 | セルA1 | セルB1 |
| 行2 | セルA2 | セルB2 |
```

これには まず列数分の `---` を用意し、それを `|` で繋げた文字列を作ります。そして その文字列に対し 両端に `|` を加えれば、セパレート行の完成です。

#### 2-4：各行のデータ（配列形式）を文字列化

セパレート行は完成したので、次は残りの表データを `markdown` 文に変えていきます。残りの表データというのは、下記の配列のことです。

```javascript:配列データ
[
  ['表', '列A', '列B'],
  ['行1', 'セルA1', 'セルB1'],
  ['行2', 'セルA2', 'セルB2'],
]
```

この配列の要素は各行単位となっており、各行の中に更に列単位の要素に分けられています。そのため、まずは各行単位で 1 つの文字列としてまとめていきます。

処理の内容は、先程のセパレート行の生成と ほぼ一緒です。各要素を `|` で繋げて 1 つの文字列とし、それの両端に `|` をつければ完成です。

```javascript:セパレート行を除いた、各行単位のmarkdown文
[
  '| 表 | 列A | 列B |',
  '| 行1 | セルA1 | セルB1 |',
  '| 行2 | セルA2 | セルB2 |',
]
```

#### 2-5：表データ（配列形式）を文字列化

先ほど作成した 各行単位の `markdown` 文ですが、完成させるためには セパレート行 が不足しています。そこで、用意したセパレート行文字列を配列の中に挿入します。

```javascript:セパレート行を挿入
[
  '| 表 | 列A | 列B |',
  '| --- | --- | --- |',
  '| 行1 | セルA1 | セルB1 |',
  '| 行2 | セルA2 | セルB2 |',
]
```

後は各要素を改行コードで繋げて 1 つの文字列にしてしまえば、必須処理としては完成です。

```javascript:改行コードで繋げ、1つの文字列に
'| 表 | 列A | 列B |\n| --- | --- | --- |\n| 行1 | セルA1 | セルB1 |\n| 行2 | セルA2 | セルB2 |'
```

### 3：無害化していた文字列をエスケープ処理した形で復元し完成（オプション）

1番で行った無害化処理が含まれている場合には、無害化した文字列を元の意味に戻す処理が必要となります。

ただし、無害化した `|` を そのまま `|` として戻してしまっては、`markdown` に影響が出てしまうので意味がありません。そのため `|` は `\\|` にするなどの、エスケープ処理した形での復元でなければなりません。

::::message

`markdown` でのエスケープ処理のみを考えた場合 `\|` でも問題はありません。しかし本記事と逆のパターン「`markdown` テーブル形式のデータを表形式データに変換する」での事情から、`\|` でなく `\\|` にしています。

これは `markdown` テーブル形式の文字列からエスケープ処理された文字列を抽出する際、`\|` だと上手く処理できなかったためです。

:::details 個人メモ：原因について

`\|` を含むデータを**文字列として解釈した際に、意味のないバックスラッシュとして無視される**ことが原因のようです。

文字列リテラルの中の `\` は、自動的にエスケープシーケンスとして扱われます。その際、`\|` は未定義のエスケープシーケンスと判断され、`\` の部分は無視されてしまうようです。

これを防ぐためには、バックスラッシュそのものを文字として認識させるために、`\\` とエスケープ処理が必要となります。

```javascript:エスケープシーケンス
console.log('|');     // -> '|'
console.log('\|');    // -> '|'  : 未定義のエスケープシーケンスとして無視される
console.log('\\|');   // -> '\|'
console.log('\\\|');  // -> '\|' : 2つはエスケープシーケンス、残る一つは無視される
```

:::

::::

## 作成

ここからは[設計](#設計)で決めた内容を基に、コードに起こしていきます。設計で説明した通り、大枠として 2 つの関数を作成することになります。

- ベースとなる、**表形式データを markdown に変換する**関数
- （上記をラップする形で）**エスケープ処理付き**の変換関数

### ベースとなる変換関数

まずは必須条件を満たす、**表形式データを markdown テーブルに変換する関数**を作成していきます。

ここでは関数名を `excel2markdown()` とします。テスト文を付けると、下記のようになります。

```javascript:関数名とテスト文
function excel2markdown(excelStr) {
  // 表形式データ（文字列）を、markdown テーブル（文字列）に変換して返す
}

// テスト
const excelStr = `表\t列A\t列B\n行1\tセルA1\tセルB1\n行2\tセルA2\tセルB2`;
console.log(excel2markdown(excelStr));
/* ↓ 出力結果
| 表 | 列A | 列B |
| --- | --- | --- |
| 行1 | セルA1 | セルB1 |
| 行2 | セルA2 | セルB2 |
*/
```

:::message

テスト文は 本記事で度々出てきた下図を、正規表現で1行にしたものです。

![テスト文の、３✕３の表データ](/images/articles/excel-to-markdown/excel2markdown-01.png)

:::

#### 特殊文字の表現を統一

最初に行うのは、OS により表記ブレが起きうる特殊文字を統一です。

ここでは**改行コード**の LF(`\n`) と CRLF(`\r\n`) を、`\n` に統一しておきます。

```javascript:特殊文字の統一
function excel2markdown(excelStr) {
  // 改行コードを \n に統一
  const normalizedStr = excelStr.replace(/\r\n/g, '\n');
}
```

#### テーブルデータを二次元配列に

次に、テーブルデータを配列形式に格納します。入力として受け取る文字列は、`\n` で行、`\t` で列を区切っているので、それらを `split()` で分割し、二次元配列にします。

```javascript:行を改行コード単位で分割した後、各行をタブ文字単位で分割
function excel2markdown(excelStr) {
  // 改行コードを \n に統一
  const normalizedStr = excelStr.replace(/\r\n/g, '\n');

  // 行→列の順で、配列に分割
  const rows = normalizedStr.trim().split('\n');
  const table = rows.map(row => row.split('\t'));
}
```

:::message

処理する文字列がテスト文の場合、`table` に格納されている中身は、下記のようになります。

```javascript:テスト文でのtableの中身
[
  ['表', '列A', '列B'],
  ['行1', 'セルA1', 'セルB1'],
  ['行2', 'セルA2', 'セルB2'],
]
```

:::

後々の処理は、この `table` を使っていくことになります。

#### 列数からセパレート行の生成

次に、`markdown` テーブルに必要となるセパレート行（例：`|---|---|---|`）を作成します。ここで必要な作業は下記のとおりです。

- 列数分 `---` を用意し、間を `|` でつなげる
- できた文字列の両端を `|` で囲む

列数は `table[0].length` で取得できます。なので、列数分の枠を設けた空配列を用意し、それに `---` で埋めることからはじめます。

```javascript:列数分のセルを配列で表現
const numColumns = table[0].length;
const separateStr = Array(numColumns).fill('---');
```

最終的に必要なのは `|---|---|---|` という文字列です。なので列数分あるこの配列を、`join(' | ')` で繋げて文字列化します。  
（※ ここでは最低限の見栄え確保のため、`|` の前後にスペースを設け、データ間の間隔を空けています）

```javascript:パイプ記号で繋げて1つの文字列に
const numColumns = table[0].length;
const separateStr = Array(numColumns).fill('---').join(' | ');
```

この段階では、`separateStr` は `--- | --- | ---` のような形です。なので、両端に `|` をつければ完成となります。ここではテンプレートリテラルを使い、これまでの式を埋め込む形で、一気に必要な文字列を作成しています。  
（※ ここでも最低限の見栄え確保のため、両端につける `|` とデータとの間にスペースを設けています）

```javascript:テンプレートリテラルを使って糖衣構文としてまとめる
const numColumns = table[0].length;
const separateStr = `| ${Array(numColumns).fill('---').join(' | ')} |`;
```

関数の中に組み込んだものが、下記となります。

```javascript:表データの配列と、セパレート行が完成
function excel2markdown(excelStr) {
  // 改行コードを \n に統一
  const normalizedStr = excelStr.replace(/\r\n/g, '\n');

  // 行→列の順で、配列に分割
  const rows = normalizedStr.trim().split('\n');
  const table = rows.map(row => row.split('\t'));

  // 列数から、Markdown のセパレート文字列を生成
  const numColumns = table[0].length;
  const separateStr = `| ${Array(numColumns).fill('---').join(' | ')} |`;
}
```

#### 各行単位で配列を文字列化

現在 `table` は二次元配列となっていますが、これはセパレート行を作るために列数が必要だったためです。

```javascript:テスト文でのtableの中身
[
  ['表', '列A', '列B'],
  ['行1', 'セルA1', 'セルB1'],
  ['行2', 'セルA2', 'セルB2'],
]
```

セパレート行を作成した今の段階では、もう各行要素が配列形式である必要はありません。なので、ここで配列を `markdown` 形式の文字列に変えてしまいます。

ここでは、`return` で使う変数 `result` を用意し、二次元配列である `table` を一次配列にした形で代入します。下記では `map` 関数を使い各行単位で、配列を文字列化する処理を行っています。

文字列化処理については、[セパレート行の生成](#列数からセパレート行の生成)で行った処理と ほぼ同じです。  
（向こうでは空配列に `---` を埋めてましたが、こちらはすでにデータはある状態なので そのまま `|` で繋げています）

```javascript:関数
function excel2markdown(excelStr) {
  // 改行コードを \n に統一
  const normalizedStr = excelStr.replace(/\r\n/g, '\n');

  // 行→列の順で、配列に分割
  const rows = normalizedStr.trim().split('\n');
  let table = rows.map(row => row.split('\t'));

  // 列数から、Markdown のセパレート文字列を生成
  const numColumns = table[0].length;
  const separateStr = `| ${Array(numColumns).fill('---').join(' | ')} |`;

  // 各行を `|` で分割した文字列に再統合
  let result;
  result = table.map(row => `| ${row.join(' | ')} |`);
}
```

:::message

処理する文字列がテスト文の場合、`result` に格納されている中身は、下記のようになります。

```javascript:テスト分でのresultの中身
[
  '| 表 | 列A | 列B |',
  '| 行1 | セルA1 | セルB1 |',
  '| 行2 | セルA2 | セルB2 |',
]
```

:::

#### 表全体を一つの文字列にして完成

残る作業は、セパレート行を `result` の中に組み込んで、配列を 1 つの文字列にすることです。それを結果として返すことで、ベースとなる変換関数は完成となります。

セパレート行を表す文字列は `separateStr` で用意しています。これを、`result` のインデックス 0 と 1 の間に挿入するために、ここでは `splice(start, deleteCount, item1)` を使います。

```javascript:セパレート行を2行目として挿入
result.splice(1, 0, separateStr);
```

これで、`result` の中身は下記のようになりました（テスト文のケース）

```javascript:テスト分でのresultの中身
[
  '| 表 | 列A | 列B |',
  '| --- | --- | --- |',
  '| 行1 | セルA1 | セルB1 |',
  '| 行2 | セルA2 | セルB2 |',
]
```

後はこの配列を、改行コード `\n` で繋げる形で文字列化すれば完成です。これまでのコードに組み込み、関数として完成したものが下記になります。

```javascript:セパレート行を挿入、改行コードで繋げた文字列を返して完成
function excel2markdown(excelStr) {
  // 改行コードを \n に統一
  const normalizedStr = excelStr.replace(/\r\n/g, '\n');

  // 行→列の順で、配列に分割
  const rows = normalizedStr.trim().split('\n');
  let table = rows.map(row => row.split('\t'));

  // 列数から、Markdown のセパレート文字列を生成
  const numColumns = table[0].length;
  const separateStr = `| ${Array(numColumns).fill('---').join(' | ')} |`;

  // 各行を `|` で分割した文字列に再統合
  let result;
  result = table.map(row => `| ${row.join(' | ')} |`);

  // セパレート行を2行目に挿入後、配列を改行文字列で繋げて返す
  result.splice(1, 0, separateStr);
  return result.join('\n');
}
```

### エスケープ処理付き変換関数（オプション）

ここからは[ベースとなる変換関数](#ベースとなる変換関数)をラップする形で、エスケープ処理付きの変換関数を作成していきます。

手順としては、下記のようになります。

1. 文字列に含まれる、処理に不都合な文字列を無害化（一時的に別の文字列に置換）
2. [変換処理](#ベースとなる変換関数)
3. 1番で行った処理を、エスケープ処理付きで復元

無害化が必要な文字列としては、`markdown` 化した際に確実に影響が出てしまう `|` を想定します。

#### 置換処理について

JavaScript で文字列を置換する主な方法としては、下記の2つが考えられます。

1. `replace(pattern, replacement)` 関数による置換
2. `{string}.split(separator1).join(separator2)` のように、配列化からの統合

1番については、正規表現を使えるのが特徴です。今回ですと文字列全体を対象とするため `g` フラグによるグローバル検索を使います。また、`|` は正規表現において「論理和」の役割を持つため、`replace(/\\|/g, '@@PIPE@@')` のようにエスケープ処理付きで渡してあげる必要があります。

一方、2番の方法は正規表現を用いないのが特徴です。置換対象となる文字列（今回は `|`）を区切り文字として配列に分割し、置換先文字列（今回は `@@PIPE@@`）で文字列として繋ぎなおす処理を行います。

置換対象が `|` のみの現状において、**本記事では 2 番の処理を採用**します。

::::message

1番を採用しなかった理由は、無害化と復元化で行う置換処理が複雑で分かりづらくなったためです。

:::details replace関数を使った場合のコード

```javascript:replace関数による手法
function excel2markdownWithEscaped(excelStr) {
  const conversionTable = [
    {
      original: '\\|', 
      evacuation: '@@PIPE@@',
      escaped: '\\\\|'
    },
  ];

  // 退避用文字列に置換
  conversionTable.forEach(replacement => {
    excelStr = excelStr.replace(new RegExp(replacement.original, 'g'), replacement.evacuation);
  });

  // ベース処理
  excelStr = excel2markdown(excelStr);

  // 退避させてた文字列をエスケープ処理付きで復元
  conversionTable.forEach(replacement => {
    excelStr = excelStr.replace(new RegExp(replacement.evacuation, 'g'), replacement.escaped);
  });

  return excelStr;
}
```

上記コードの場合、`conversionTable` の `original`, `evacuation` は正規表現に、`evacuation`, `escaped` は通常文字列として利用しています。オブジェクトとして格納しているデータの扱いが複雑で、一見すると何故エスケープ処理のない表形式データに対する `original` で `\\|` となるのか、わかりづらい状態です。

:::

現状においては、簡素な `split()` + `join()` の手法が使えるので、こちらを採用することにしました。ただし状況が変化した場合には、再度こちらの手法を突き詰めていく必要が出るかもしれません。  
（例：`|` 以外に要エスケープ処理の文字列が追加され、それが複雑なパターンで**正規表現でないと解決できない**場合）

::::

#### エスケープ処理付き変換関数の作成

置換する方法が決まったので、ここからはそれに基づきコーディングしていきます。

```javascript:関数名とテスト文
function excel2markdownWithEscaped(excelStr) {
  // （引数に含まれる特定の文字列をエスケープ処理した上で）表形式データを、markdown テーブルに変換して返す
}

// テスト
const testWithPipe = '表\t列A\t列B\n行1\tセル|A1\tセル|B1\n行2\tセル|A2\tセルB2';
console.log(excel2markdownWithEscaped(testWithPipe));
/* ↓ 出力結果
| 表 | 列A | 列B |
| --- | --- | --- |
| 行1 | セル\\|A1 | セル\\|B1 |
| 行2 | セル\\|A2 | セルB2 |
*/
```

##### 変換テーブルの準備

本処理にあたって、必要となる情報は下記の 3 つです。

- エスケープ処理対象となる文字列
- 一時退避用の無害化文字列
- エスケープ処理した文字列

今回は、`|` のみ考えるので、上記内容をまとめると下表になります。

|項目|値|
|:---|---|
|処理対象の文字列 `original`|`\|`|
|退避用の文字列 `evacuation`|`@@PIPE@@`|
|処理済み文字列 `escaped`|`\\\|`|

今後パターンが増えるかもしれないことを考慮して、この 3 つの情報をオブジェクトとして纏め、配列形式で格納しておきます。

```javascript:変換テーブルを作成し、3種のデータを準備
function excel2markdownWithEscaped(excelStr) {
  const conversionTable = [
    {
      original: '|', 
      evacuation: '@@PIPE@@',
      escaped: '\\\\|'  // -> エスケープシーケンスで '\\|'
    },
  ];
}
```

:::message

`@@〜@@` という 通常使われない文字列表現は、置換処理にデータを巻き込まないようするためです。

また `@@〜@@` を扱う位置について「`evacuation: 'PIPE'` で格納しておき、処理中に `@@` をつける方法でもいいのでは？」と思われるかもしれません。こちらについては、下記の考えから直接入力にしています。

1. なにか問題があって `@@` を変える際、他のパターンも影響を受けてしまう。個別につければ、問題のある箇所だけ変えて対処ができる。
2. `@@PIPE@@` と見える形であることにより、退避用文字列であることが ひと目でわかる。

:::

##### 退避用文字列への置換処理

次に変換テーブルをループ（複数パターンを想定）させ、処理に不都合な文字列を退避用文字列に置換します。ここでは `original` にある文字列を**区切り文字として配列に分割**し、`evacuation` にある**文字列で繋げる**ことで置換処理を行っています。

```javascript:退避用文字列へ置換
function excel2markdownWithEscaped(excelStr) {
  const conversionTable = [
    {
      original: '|', 
      evacuation: '@@PIPE@@',
      escaped: '\\\\|'
    },
  ];

  // 退避用文字列に置換
  conversionTable.forEach(replacement => {
    excelStr = excelStr.split(replacement.original).join(replacement.evacuation);
  });
}

const testWithPipe = '表\t列A\t列B\n行1\tセル|A1\tセル|B1\n行2\tセル|A2\tセルB2';
console.log(excel2markdownWithEscaped(testWithPipe));
/* ↓ この段階だと、下記のようになる
表	列A	列B
行1	セル@@PIPE@@A1	セル@@PIPE@@B1
行2	セル@@PIPE@@A2	セルB2
*/
```

##### 変換処理後に復元化

処理に不都合な `|` は `@@PIPE@@` に変わっているため、ベースとなる変換関数 `excel2markdown()` に渡しても問題なくなりました。

```javascript:ベースとなる変換関数に渡す
function excel2markdownWithEscaped(excelStr) {
  const conversionTable = [
    {
      original: '|', 
      evacuation: '@@PIPE@@',
      escaped: '\\\\|'
    },
  ];

  // 退避用文字列に置換
  conversionTable.forEach(replacement => {
    excelStr = excelStr.split(replacement.original).join(replacement.evacuation);
  });

  // ベース処理
  excelStr = excel2markdown(excelStr);
}

const testWithPipe = '表\t列A\t列B\n行1\tセル|A1\tセル|B1\n行2\tセル|A2\tセルB2';
console.log(excel2markdownWithEscaped(testWithPipe));
/* ↓ この段階だと、下記のようになる
| 表 | 列A | 列B |
| --- | --- | --- |
| 行1 | セル@@PIPE@@A1 | セル@@PIPE@@B1 |
| 行2 | セル@@PIPE@@A2 | セルB2 |
*/
```

後は `@@PIPE@@` をエスケープ処理した文字列 `\\|` に置き換えれば、関数の返り値が完成します。ここでの置換処理は `evacuation` 文字列を区切りに配列化し、`escaped` 文字列で繋げることで行います。

```javascript:復元化して完成
function excel2markdownWithEscaped(excelStr) {
  const conversionTable = [
    {
      original: '|', 
      evacuation: '@@PIPE@@',
      escaped: '\\\\|'
    },
  ];

  // 退避用文字列に置換
  conversionTable.forEach(replacement => {
    excelStr = excelStr.split(replacement.original).join(replacement.evacuation);
  });

  // ベース処理
  excelStr = excel2markdown(excelStr);

  // 退避させてた文字列をエスケープ処理付きで復元
  conversionTable.forEach(replacement => {
    excelStr = excelStr.split(replacement.evacuation).join(replacement.escaped);
  });

  return excelStr;
}

const testWithPipe = '表\t列A\t列B\n行1\tセル|A1\tセル|B1\n行2\tセル|A2\tセルB2';
console.log(excel2markdownWithEscaped(testWithPipe));
/* ↓ 返り値は、目標としていたデータと一致
| 表 | 列A | 列B |
| --- | --- | --- |
| 行1 | セル\\|A1 | セル\\|B1 |
| 行2 | セル\\|A2 | セルB2 |
*/
```

## 補足

### コード全文

:::details コード全文（長いので格納しています）

```javascript:関数
function excel2markdown(excelStr) {
  // 改行コードを \n に統一
  const normalizedStr = excelStr.replace(/\r\n/g, '\n');

  // 行→列の順で、配列に分割
  const rows = normalizedStr.trim().split('\n');
  let table = rows.map(row => row.split('\t'));

  // 列数から、Markdown のセパレート文字列を生成
  const numColumns = table[0].length;
  const separateStr = `| ${Array(numColumns).fill('---').join(' | ')} |`;

  // 各行を `|` で分割した文字列に再統合
  let result;
  result = table.map(row => `| ${row.join(' | ')} |`);

  // セパレート行を2行目に挿入後、配列を改行文字列で繋げて返す
  result.splice(1, 0, separateStr);
  return result.join('\n');
}

function excel2markdownWithEscaped(excelStr) {
  const conversionTable = [
    {
      original: '|', 
      evacuation: '@@PIPE@@',
      escaped: '\\\\|'
    },
  ];

  // 退避用文字列に置換
  conversionTable.forEach(replacement => {
    excelStr = excelStr.split(replacement.original).join(replacement.evacuation);
  });

  // ベース処理
  excelStr = excel2markdown(excelStr);

  // 退避させてた文字列をエスケープ処理付きで復元
  conversionTable.forEach(replacement => {
    excelStr = excelStr.split(replacement.evacuation).join(replacement.escaped);
  });

  return excelStr;
}

const test = '表\t列A\t列B\n行1\tセルA1\tセルB1\n行2\tセルA2\tセルB2';
console.log(excel2markdown(test));
/* ↓ 出力結果
| 表 | 列A | 列B |
| --- | --- | --- |
| 行1 | セルA1 | セルB1 |
| 行2 | セルA2 | セルB2 |
*/

const testWithPipe = '表\t列A\t列B\n行1\tセル|A1\tセル|B1\n行2\tセル|A2\tセルB2';
console.log(excel2markdownWithEscaped(testWithPipe));
/* ↓ 出力結果
| 表 | 列A | 列B |
| --- | --- | --- |
| 行1 | セル\\|A1 | セル\\|B1 |
| 行2 | セル\\|A2 | セルB2 |
*/
```

:::

### 実際に使用する際の注意事項

本記事では、逆パターン [markdown から表形式データに変換](https://zenn.dev/nonaka101/articles/markdown-to-excel) が上手く処理できないという理由から `\\|` という形でのエスケープしています。

ですが**実際に使用する際には** `\|` **で処理できるケースがあります**。例として、[私が作成している簡易ツール集](https://nonaka101.github.io/jig-a/) では `textarea` 要素を使って入力を受け付けており、`\|` の形で処理できています。

これはスクリプト上で用意した**文字列リテラルでは自動的にエスケープ処理される**のに対し、`textarea` 等に**入力されたデータは生の文字列**として受け入れているためです。

このような挙動の違いがあるため、実際に利用する際にはエスケープ処理の扱いに注意が必要となってきます。下記は生の文字列が使える場合のパターンで、`\|` でエスケープするようになっています。

:::details 生の文字列で処理する場合の関数

```javascript:生の文字列用の関数
function excel2markdownWithEscaped(excelStr) {
  const conversionTable = [
    {
      original: '|',
      evacuation: '@@PIPE@@',
      escaped: '\\|'
    },
  ];

  // 退避用文字列に置換
  conversionTable.forEach(replacement => {
    excelStr = excelStr.split(replacement.original).join(replacement.evacuation);
  });

  // ベース処理
  excelStr = excel2markdown(excelStr);

  // 退避させてた文字列をエスケープ処理付きで復元
  conversionTable.forEach(replacement => {
    excelStr = excelStr.split(replacement.evacuation).join(replacement.escaped);
  });

  return excelStr;
}

const testWithPipe = '表\t列A\t列B\n行1\tセル|A1\tセル|B1\n行2\tセル|A2\tセルB2';
console.log(excel2markdownWithEscaped(testWithPipe));
// ↓ 出力結果
// | 表 | 列A | 列B |
// | --- | --- | --- |
// | 行1 | セル\|A1 | セル\|B1 |
// | 行2 | セル\|A2 | セルB2 |
```

:::

### クリップボードについて

:::message alert

下記内容は、私が調べて理解できた範囲を わかりやすく噛み砕いて説明したものです。そのため、正確な情報ではないことをご了承ください。

:::

#### クリップボードの概要

![Excel表データをコピーして、GoogleSheets に書式付きでペーストしている](/images/articles/excel-to-markdown/clipboard-01.png)

Excel 表をコピーして GoogleSheets にペーストすると、書式（セルや文字の色、太字など）を含めて転写することができます。同じ表計算ソフトとはいえ、何故このようなことができるのでしょう？

それは、コピー＆ペースト間にある**クリップボード**の仕組みが関係しています。クリップボードは、コピー元となる表データのみを記憶しているのでなく、**アプリケーション間での利用を想定して様々な形式の情報を取得**しています。そしてペースト側のアプリケーションは、**情報を解釈しコピー元を可能な限り再現しよう**と、利用できる情報を拾ってきます。

この一連の流れを図に表すと、下図のようになります。

![Excel表データをクリップボードに格納すると、様々な形式の情報が格納され、ペースト先のアプリケーションに合わせ引き出されている](/images/articles/excel-to-markdown/clipboard-02.png)

- コピー元から、様々な形式の情報を取得する
- コピー先へは、可能な限り再現しようと情報を利用する

例えば上図のように、表データをコピーしたとします。  
テキストエディタにペーストする場合、テキストデータのみ転写され、書式情報や画像は利用できないため抜け落ちることになります。  
一方ペイントにペーストした場合は、画像データを転写し、テキストや書式情報は扱えないため抜け落ちることになります。

そして GoogleSheets にペーストした場合ですが、まずテキストデータが転写されます。標準的な表計算ソフトは `tsv`( *Tab-Separated Values* ) がサポートされているので、コピー元と同じ行列構造で転写することができます。  
更にコピー元を再現する過程で、セルの色やフォント（太字など）といった書式情報も利用可能であるため 転写されます。ただし完全にサポートされるわけではないので、場合によっては一部情報が抜け落ちることはありえます。  
（上図では Numbers でセル色が変になっています。こうした場合、「プレーンテキストとして貼り付ける」「値のみ貼り付け」などで引き出す情報を絞って利用した方が良いかもしれません）

このようにデータの解釈がアプリケーション間で行われていることが、クリップボードの特徴となります。

#### 本記事においてクリップボードはどう関わるのか？

本記事においては、`tsv` 形式のテキストデータのみを利用して処理をしています。Excel, GoogleSheets において表をコピーすると、タブ文字と改行コードで区切られた `tsv` 形式のテキストを取得できるので、これを利用して `markdown` テーブルに変換しています。

なお `tsv` 形式なら良いため、実は HTML の `table` 要素にも本機能が使えたりします。下図は本記事の文字列をカウントし、様々な情報を `table` 要素で出力しています。

![JIG-Aで本記事の文字列カウントしている図](/images/articles/excel-to-markdown/clipboard-03.png)

この `table` 要素をコピーし、本記事での処理に掛けると、問題なく `markdown` 形式に変換されていることがわかります。

![先程の table 要素を使って、markdown を出力している](/images/articles/excel-to-markdown/clipboard-04.png)

これもクリップボードが `html` 要素を格納するだけでなく、`tsv` 形式のテキストデータを取得しているからできるものだと思われます。
