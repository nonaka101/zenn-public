---
title: "🐥 LDAP を用いたユーザー情報の取得 1"
---

ここでは *ActiveDirectory モジュールをインストールしていない端末* からでも、**ドメイン上のユーザー情報を照会し取得する方法**について説明します。

実務では、例えば下記のような場面で「ユーザーを一意に特定できる情報」が必要になります。

- フォルダに対して、指定ユーザーのアクセス権を付与する
- メールアドレスから、対象ユーザーの AD アカウント情報（例：表示名）を確認する

このような処理においては、最終的に `displayName` や `sAMAccountName`、`objectSid` などのユーザーに関する諸属性（情報）を取得する必要があります。

通常であれば `Get-ADUser` を使うのが最も簡素で わかりやすいのですが、端末によっては PowerShell に ActiveDirectory モジュールが入っていない場合があります。

そのような環境でも、.NET の `System.DirectoryServices.DirectorySearcher` を利用することで、**LDAP**（ *Lightweight Directory Access Protocol* ）経由で ActiveDirectory を検索することができます。

## 本処理の目的

本処理の目的は、ActiveDirectory 上のユーザーオブジェクトを検索し、後続処理で必要になる属性を取得することです。

本項では、下記の 2 パターンでユーザーを検索してみます。

- `sAMAccountName` からユーザーを検索する
- `mail` アドレスからユーザーを検索する

検索後は、下記のような属性を取得しています。

| 属性 | 用途 |
| --- | --- |
| `sAMAccountName` | Windows で使われるログオン名 |
| `displayName` | 表示名 |
| `mail` | メールアドレス |
| `userPrincipalName` | UPN 形式のログオン名（※） |
| `distinguishedName` | AD 上での識別名 |
| `objectSid` | Windows のアクセス制御で使われる SID（ *Security Identifer* ） |

:::message

**UPN**（ *User Principal Name* ）とは、ActiveDirectory ユーザーがログオンする際に使用できる識別子の 1 つです。

形式は `ユーザー名@UPNサフィックス` となり、例えば `hoge@fuga.jp` といった形です。  
（※ UPN は**AD におけるログオン名**として利用される属性であり、メールアドレスとは異なる場合があります）

:::

ユーザーオブジェクトが取得できれば、例えば下記のような利用が可能となります。

- **フォルダへアクセス権を付与する**場合、`SID` を使用して Windows のアクセス制御を操作する
- 入力した情報から取得したユーザーが誰なのか、確認の意味を込めてコンソール上に名前（`DisplayName`）を表示する

### 作業例：アクセス権の付与

手作業で フォルダにアクセス許可を設定する場合、下図のように「ユーザーまたはグループの選択」画面から「名前の確認」を使うことがあります。

:::message

ここでは、SAMアカウント名など一意の識別子を入力して「名前の確認」を押します。

ここでは `Administrator` と入力すると、該当するユーザーを調べ `DOMAIN\Administrator` といった形に解決してくれています。

下図のように下線付きのテキストに変化した状態が、オブジェクトとして認識していることを示しています。

:::

![ユーザーまたはグループの選択 画面](/images/books/learning-powershell/namming.png)

この名前の解決について、Windows 側でやっている処理は単なる文字列の形式のチェックではありません。大雑把に言えば、下記のような処理を行っています。

1. 入力された文字列を受け取る
2. 指定された場所（ドメイン、信頼ドメイン、ローカル SAM など）を検索する
3. **一致する *Security Principal* を探す**
4. 見つかったオブジェクトの `SID` を取得する
5. `SID` を正規化された名前に変換する
6. 画面に表示する

:::message

重要なのは 3 番で、**アクセス権の設定といった処理にはユーザーオブジェクトを探す必要がある**というところです。

処理ではそのために該当するセキュリティプリンシパルを調べ、そこから紐づいている `SID` を取得しています。

:::

本項で取り扱う `DirectorySearcher` を使った処理も、考え方としてはこれに近いものとなります。

1. 入力された文字列からクエリを発行
2. ActiveDirectory を検索
3. 該当するユーザーオブジェクトを取得
4. 必要な属性情報を取り出し、後続処理に渡す

#### 補足：セキュリティ プリンシパル について

[Security Principal](https://learn.microsoft.com/ja-jp/windows-server/identity/ad-ds/manage/understand-security-principals)とは、ActiveDirectory 内での「認証され、権限を持ちうる主体」という概念です。

セキュリティプリンシパルは、主に下記の種類が存在します。

| オブジェクト | 説明 |
| --- | --- |
| ユーザー（ *User* ） | 人間の利用者 |
| グループ（ *Group* ） | ユーザーの集合体 |
| コンピューター（ *Computer* ） | ドメイン参加したPCやサーバー |
| サービス アカウント | システムやアプリケーション用アカウント |

これらの要素は、例えば `S-1-5-21-1234567890-987654321-1111111111-1001` といった `SID`（ *Security Identifer* ）で管理され、そこに様々な属性情報がぶら下がっているイメージです。

ユーザーを識別する情報についてまとめたものが下表になります。

| 属性 | 例 | 用途 |
| --- | --- | --- |
| `SID` | `S-1-5-21-1234567890-987654321-1111111111-1001` | 内部識別子 |
| `UPN` | `tanaka@fuga.jp` | ログイン識別子 |
| `SAMAccountName` | `tanaka` | ログイン識別子（旧） |
| `Distinguished Name`（`DN`） | `CN=Tanaka,OU=Users,DC=fuga,DC=jp` | AD内の場所 |
| `GUID`（`ObjectGUID`） | `550e8400-e29b-41d4-a716-446655440000` | オブジェクト一意識別 |

## ActiveDirectory モジュールが使える方法

ActiveDirectory 上のユーザー情報を取得する場合、一般的には `Get-ADUser` を使います。

```ps1
# SAMアカウント名が `hoge001` のユーザーに対し、全ての情報を取得
Get-ADUser -Identity hoge001 -Properties *
```

メールアドレスで検索する場合は、次のように書くこともできます。

```ps1
# メールアドレスが `hoge@fuga.jp` と一致するユーザーに対し、アドレスと表示名を取得
Get-ADUser -Filter "mail -eq 'hoge@fuga.jp'" -Properties mail, displayName
```

ただし、この方法は **ActiveDirectory PowerShell モジュールがインストールされていること**が前提となります。

:::message

現場のファイルサーバーや一部の管理端末では、下記のような制約があることがあります。

- ActiveDirectory モジュールが入っていない
- RSAT（ *Remote Server Administration Tools* ）を追加インストールできない
- 管理端末ではなく、利用者に近い端末上でスクリプトを実行したい

そのような場合の選択肢として、本項では `System.DirectoryServices.DirectorySearcher` を使う方法を説明しています。  
（※ ただしこれは特殊な事情への対応であるため、**AD モジュールが使えるなら素直に `Get-ADUser` を優先すべき**ではあります）

:::

## DirectorySearcher とは

https://learn.microsoft.com/ja-jp/dotnet/api/system.directoryservices.directorysearcher

`System.DirectoryServices.DirectorySearcher` は、.NET から **AD DS**（ *ActiveDirectory Domain Services* ）に対して検索を行うためのクラスです。

検索には **LDAP** を使用します。

:::message

**LDAP**（ *Lightweight Directory Access Protocol* ）は、**ユーザーやグループなどの情報を検索・認証するための共通ルール**（プロトコル）となります。

（Windows 環境向けの ディレクトリサービスである）ActiveDirectory は、この LDAPサーバ機能を実装しています。

ActiveDirectory では、組織内の *人, コンピューター, グループ* などの情報を一元的に管理しています。要は、「**組織内の資源情報を扱うデータベース**」といったイメージです。

このデータベースは、様々な観点が混じった管理がされています。

- 組織構造（例：`事業1部`→`営業課`→`田中`）
- 社員ID（例：`ID001`↔`田中`）
- 役職（例：`課長`→`田中`）

他にも 入社日だったり生年月日だったりと、様々な視点があります。こうしたデータベースに対し、「事業1部の営業課にいる人物から〜」や「課長職にある〜」といった方法で**データを引き出す仕組み**が、LDAP の大まかなイメージです。

:::

PowerShell では、下記のように `.NET` のクラスを直接生成して使うことができます。

```ps1
$Searcher = New-Object System.DirectoryServices.DirectorySearcher
```

`DirectorySearcher` を使うと、次のような処理ができます。

- LDAP フィルターを指定して AD オブジェクトを検索する
- 1 件だけ（あるいは複数件）取得する
- 検索結果から `DirectoryEntry` を取得し、属性を参照する
- 取得する属性を指定する

:::message

本来の `DirectorySearcher` はユーザー以外にもグループやコンピュータなども検索可能です。

ただし今回は 用途の都合上、ユーザー検索に絞って説明をしていきます。

:::

### DirectorySearcher を使ったユーザー検索例

ここでは、実際に `sAMAccountName` からユーザーを検索する例を見ていきます。

```ps1
# 検索器（DirectorySearcher）を生成
$Searcher = New-Object System.DirectoryServices.DirectorySearcher

# 検索器に LDAP フィルターを設定（ユーザーで SAMアカウント名が `hoge001`）
$Searcher.Filter = "(&(objectCategory=person)(objectClass=user)(sAMAccountName=hoge001))"

# LDAP クエリを ActiveDirectory に送信し、結果を変数に格納
$Result = $Searcher.FindOne()


# 処理としては上記で完了しているが、中身を確認してみる
$Result | Format-List
<#
Path        : LDAP://{CN=hoge001,OU=Users,DC=fuga,DC=jp}
Properties  : (givenname, codepage, objectcategory, loginshell...)
#>
```

`$Searcher.Filter` で設定している下記のテキストが、LDAP フィルターとなります。

```txt
(&(objectCategory=person)(objectClass=user)(sAMAccountName=hoge001))
```

これは、下記の 3 条件を `AND` でつないだ検索条件です。

| 条件 | 意味 |
| --- | --- |
| `objectCategory=person` | 人に関するオブジェクトに絞る |
| `objectClass=user` | ユーザーオブジェクトに絞る |
| `sAMAccountName=hoge001` | `sAMAccountName` が `hoge001` のものに絞る |

:::message

LDAP フィルターでは、AND 条件を `&` で表します。

```txt
(&(条件1)(条件2)(条件3))
```

SQL のような書き方（例：`WHERE A = B AND C = D`）と違うため、最初は少しとっつきにくいかもしれません。

イメージとしては「条件を括弧で囲んで、先頭に演算子を書く」といった感じです。

:::

### LDAP フィルターについて

#### LDAP フィルターの基本

LDAP フィルターの基本形は、次のような形です。

```txt
(属性名=値)
```

これに沿った例が、下記となります。

- `(sAMAccountName=hoge001)`
- `(mail=tanaka@fuga.jp)`
- `(displayName=田中 太郎)`

`AND` 条件は `&` を使い、次のように書きます。

```txt
(&(objectCategory=person)(objectClass=user)(sAMAccountName=hoge001))
```

`OR` 条件は `|` を使い、次のように書きます。

```txt
(|(sAMAccountName=hoge001)(mail=tanaka@fuga.jp))
```

`NOT` 条件は `!` を使い、次のように書きます。

```txt
(!(mail=*suzuki*))
```

:::message

`*` はワイルドカードとして使われることがあります。

```txt
(displayName=山田*)
```

:::

#### `objectCategory` と `objectClass` について

本項で ユーザーを検索する際、コードには次の条件を付けています。これらは、どちらも AD オブジェクトの種類を絞り込むための条件です。

- `(objectCategory=person)`
- `(objectClass=user)`

##### `objectCategory=person`

`objectCategory` は、オブジェクトのカテゴリを表す属性です。

`person` を指定することで、人に関するオブジェクトに絞り込みます。

##### `objectClass=user`

`objectClass` は、そのオブジェクトが属するクラスを表す属性です。

`user` を指定することで、ユーザーオブジェクトに絞り込みます。

##### なぜ両方指定するのか

本項で行う処理は、条件に合致する**単一の**検索結果を取り出すことを想定しています。

単純に `sAMAccountName` だけで検索することも可能なのですが、（環境によっては）意図しないオブジェクトが返る可能性が出てきます。

そのため、少なくとも「人であること」「ユーザーであること」を条件に含め、**検索対象を狭める**意図を ここに込めています。

```ps1
# 事前に、エスケープ処理済みの入力値を `$EscapedSam` に格納している
$Searcher.Filter = "(&(objectCategory=person)(objectClass=user)(sAMAccountName=$EscapedSam))"
```

実務コードでは、このようにユーザーアカウントに絞ってから、`sAMAccountName` や `mail` で検索しています。

#### ユーザー入力から LDAP フィルターを生成する場合

ワイルドカードが使えて、`OR` 検索などの機能も使える都合上、**SQL インジェクション**と同様の問題を抱えています。

そのため**ユーザーからの入力をそのまま LDAP フィルターに埋め込む場合**は注意が必要です。

:::message

簡単に言えば、「入力値を悪用して不正なクエリを実行し、データの閲覧や改ざんを行う」といった問題です。

例えば 下記のような入力（`*` や `(` などが含まれている）の場合、これをそのままフィルターに入れてしまうと、検索条件の意味が変わる可能性があります。

```txt
hoge*)
```

:::

そのため、本項では SQL インジェクションへの対策と同様に、**LDAP フィルター用のエスケープ処理**を入れるようにしています。

##### LDAP フィルター用のエスケープ処理

下表は、LDAP フィルター上で特別な意味を持つ文字と、LDAP の文字列表現に合わせてエスケープ処理したものの対応関係となります。

| 文字 | エスケープ後 |
| --- | --- |
| `\` | `\5c` |
| `*` | `\2a` |
| `(` | `\28` |
| `)` | `\29` |
| NULL 文字 | `\00` |

検索キーに 人による入力値が想定される場合は、下記のような関数を設けてエスケープ処理を行い、正規化した文字列を `Filter` に組み込む方が安全です。

```ps1
function ConvertTo-LdapEscapedFilterValue {
  [CmdletBinding()]
  param(
    [Parameter(Mandatory = $true)]
    [string]$Value
  )

  $Escaped = $Value

  $Escaped = $Escaped -replace '\\', '\5c'
  $Escaped = $Escaped -replace '\*', '\2a'
  $Escaped = $Escaped -replace '\(', '\28'
  $Escaped = $Escaped -replace '\)', '\29'
  $Escaped = $Escaped -replace "`0", '\00'

  return $Escaped
}
```

### 本項で使用する検索キーについて

本項では、検索キーとして主に次の 2 つを扱っています。

| 属性 | 説明 | 例 |
| --- | --- | --- |
| `sAMAccountName` | Windows の古い形式のログオン名として使われる属性 | `hoge001` |
| `mail` | メールアドレスとして使われる属性 | `hoge@fuga.jp` |

```ps1
# `$Escaped*` の変数については、前述したエスケープ処理を施した文字列とする

# 例：`sAMAccountName` によるフィルタリング
$Searcher.Filter = "(&(objectCategory=person)(objectClass=user)(sAMAccountName=$EscapedSam))"

# 例：`mail` によるフィルタリング
$Searcher.Filter = "(&(objectCategory=person)(objectClass=user)(mail=$EscapedMailAddress))"
```

### `PropertiesToLoad` で取得する属性を絞る

サンプルで取り上げた検索方法の場合、ヒットしたユーザーオブジェクトに関する**すべての情報**を取得することになります。

:::message

下記のコードの場合、`$Result` には全ての情報が格納されることになります。

```ps1
# 検索器（DirectorySearcher）を生成
$Searcher = New-Object System.DirectoryServices.DirectorySearcher

# 検索器に LDAP フィルターを設定（ユーザーで SAMアカウント名が `hoge001`）
$Searcher.Filter = "(&(objectCategory=person)(objectClass=user)(sAMAccountName=hoge001))"

# LDAP クエリを ActiveDirectory に送信し、結果を変数に格納
$Result = $Searcher.FindOne() # -> ユーザーに関する全ての情報
```

そこに入っている属性情報は、例えば下記のようなものです。

```txt
`cn`, `displayName`, `distinguishedName`, `givenName`, `mail`, `name`,
`objectGUID`, `objectSid`, `proxyAddresses`, `sAMAccountName`, `sAMAccountType`,
`sn`, `uidNumber`, `userPrincipalName`, `whenChanged`, `whenCreated` ... 
```

:::

しかし、実際に必要となる情報は *全体に比べると極少数であるケース* がほとんどなので、このやり方には無駄が含まれてしまっています。

`DirectorySearcher` では 検索結果から取得したい属性を `PropertiesToLoad` に追加でき、**クエリを発行する際に取得する属性を あらかじめ指定することができます**。

今回の用途では、下記の属性を取得するようにします。

- `sAMAccountName`
- `displayName`
- `mail`
- `userPrincipalName`
- `distinguishedName`
- `objectSid`

```ps1
# 検索器（DirectorySearcher）を生成
$Searcher = New-Object System.DirectoryServices.DirectorySearcher

# 検索器に LDAP フィルターを設定（ユーザーで SAMアカウント名が `hoge001`）
$Searcher.Filter = "(&(objectCategory=person)(objectClass=user)(sAMAccountName=hoge001))"

# クエリ発行前に、取得する属性を指定しておく
[void]$Searcher.PropertiesToLoad.Add('sAMAccountName')
[void]$Searcher.PropertiesToLoad.Add('displayName')
[void]$Searcher.PropertiesToLoad.Add('mail')
[void]$Searcher.PropertiesToLoad.Add('userPrincipalName')
[void]$Searcher.PropertiesToLoad.Add('distinguishedName')
[void]$Searcher.PropertiesToLoad.Add('objectSid')

# LDAP クエリを ActiveDirectory に送信し、結果を変数に格納
$Result = $Searcher.FindOne() # -> ユーザーに関する、指定した属性のみ所持
```

:::message

多少冗長になっている上記のコードを、配列を使ってスマートにしたのが下記です。  
（処理としては同じものです）

```ps1
@(
  'sAMAccountName'
  'displayName'
  'mail'
  'userPrincipalName'
  'distinguishedName'
  'objectSid'
) | ForEach-Object {
  [void]$Searcher.PropertiesToLoad.Add($_)
}
```

:::

### `FindOne()` と `FindAll()` で検索クエリを発行する

`DirectorySearcher` で検索を実行する代表的なメソッドは、`FindOne()` と `FindAll()` です。

```ps1
# LDAP クエリを ActiveDirectory に送信し、結果を変数に格納
$Result = $Searcher.FindOne()
```

#### `FindOne()`

`FindOne()` は、条件に一致した**最初の 1 件**を返します。

```ps1
$Result = $Searcher.FindOne()
```

該当するものがなければ `$null` が返ります。

ただし、複数件ヒットした場合でも最初の 1 件だけが返るため、「本当に 1 件だけか」を確認したい場合には向いていません。

#### `FindAll()`

`FindAll()` は、条件に一致した結果の**コレクション**を返します。

```ps1
$Results = $Searcher.FindAll()
```

実務において想定外のケースを踏まえるのであれば、`FindAll()` の方を使い、**ヒットした件数に応じて処理を変える**などのテクニックが役に立つかもしれません。

```ps1
$Searcher = New-Object System.DirectoryServices.DirectorySearcher
$Searcher.Filter = "(&(objectCategory=person)(objectClass=user)(sAMAccountName=hoge001))"

# 複数ヒットの可能性を考えて、FindAll でクエリを発行
$Result = $Searcher.FindAll()  # -> 返り値は 件数に関わらずコレクションとなる

# 想定するケースは「1 件だけヒットしている状態」なので、それ以外の場合は処理からはじく
if ($Results.Count -eq 0) { throw "ユーザーが AD に存在しません。" }
if ($Results.Count -gt 1) { throw "ユーザーが AD で複数ヒットしました。" }

# 想定しているケース（条件にヒットする件数が 1 つのみ）なので、処理を継続する
```

本項では**単一の**ユーザーという想定なので、一意であることを確認するために `FindAll()` で件数を確認し、0 件や複数件をエラーとして扱うようにしています。

### `SearchResult` と `DirectoryEntry` の違い

`FindOne()` や `FindAll()` で発行した検索処理に対し、返ってくるのは まず `SearchResult` です。これは 検索結果として返された情報を持っています。

https://learn.microsoft.com/ja-jp/dotnet/api/system.directoryservices.searchresult

```ps1
# SearchResult 型のインスタンスが、`$Result` に格納される
# ※ `FindAll()` の場合は、これのコレクションとなる
$Result = $Searcher.FindOne()
```

そこから *実際のディレクトリエントリ* として扱いたい場合は、`GetDirectoryEntry()` を呼び出します。

https://learn.microsoft.com/ja-jp/dotnet/api/system.directoryservices.directoryentry

```ps1
$Entry = $Result.GetDirectoryEntry()
```

:::message

大まかには、次のように考えるとわかりやすいです。

| 種類 | 役割 |
| --- | --- |
| `SearchResult` | 検索結果として返ってきた 1 件分の情報 |
| `DirectoryEntry` | AD 上の実体を表すオブジェクト |

属性を参照するだけであれば `SearchResult.Properties` から取得できる場合もあります。

ただ確実なのは、`GetDirectoryEntry()` で実体となる `DirectoryEntry` を取得し、そこから諸属性を取得していく方法です。

本項でもこちらの方法を使い、`objectSid` などの属性を取り出しています。

```ps1
# エントリを取得し、SID 情報を取り出す
$Entry = $Results[0].GetDirectoryEntry()
$SidBytes = [byte[]]$Entry.Properties['objectSid'][0]
```

:::

### 不要になった諸要素は、最後に `Dispose()` でリソース解放する

`DirectorySearcher` や `DirectoryEntry`、`FindAll()` で返される `SearchResultCollection` は、不要になったら `Dispose()` でリソースの解放をしておくことを推奨します。

:::message

これは実務において、長時間稼働し続ける場合を想定しているためです。

PowerShell の短いスクリプトでは見落としがちですが、実務では繰り返し実行される処理や長時間動く処理もあります。

そのため、不要になったタイミングで `.Dispose()` を呼び出す形にしておく方が安全だと考えます。

:::

```ps1
$Searcher = New-Object System.DirectoryServices.DirectorySearcher

try {
  $Results = $Searcher.FindAll()
}
finally {
  if ($null -ne $Results) {$Results.Dispose()}
  if ($null -ne $Searcher) {$Searcher.Dispose()}
}
```

:::message

最後の `if ($null -ne $Result) {...}` について、多言語を経験していると比較対象が左辺に来ていないので若干違和感を感じるかもしれません。

これは **PowerShell での 配列フィルタリングの特殊挙動**が関係しており、スカラー値である `$null` を左辺に置いておきたいという事情があります。

:::

## ここまでのまとめ

前述の `GetDirectoryEntry()` までで、ユーザーオブジェクトに含まれている諸属性を取得することができるようになりました。ここまでの一連の流れをまとめたものが、下記となります。

```ps1
# ※ LDAP 用のエスケープ関数 `ConvertTo-LdapEscapedFilterValue` は省略

# 検索する SAM アカウント名（ここでは固定とする）
$SamAccountName = 'hoge001'

# LDAP フィルター用にエスケープ
$EscapedSam = ConvertTo-LdapEscapedFilterValue -Value $SamAccountName

# DirectorySearcher を作成
$Searcher = New-Object System.DirectoryServices.DirectorySearcher

try {
  # ユーザーアカウントに絞り込んだ上で、フィルタリング設定
  $Searcher.Filter = "(&(objectCategory=person)(objectClass=user)(sAMAccountName=$EscapedSam))"

  # クエリ発行前に、取得する属性を絞っておく
  @(
    'sAMAccountName'
    'displayName'
    'userPrincipalName'
    'distinguishedName'
    'objectSid'
  ) | ForEach-Object {
    [void]$Searcher.PropertiesToLoad.Add($_)
  }

  # 検索実行
  $Results = $Searcher.FindAll()

  # 検索結果が 想定通り（1 件のみ）になっているか確認
  if ($Results.Count -eq 0) { throw "ユーザー '$SamAccountName' が AD に存在しません。" }
  if ($Results.Count -gt 1) { throw "ユーザー '$SamAccountName' が AD で複数ヒットしました。" }

  # 1 件だけ見つかったので DirectoryEntry を取得
  $Entry = $Results[0].GetDirectoryEntry()

  # 後続向けの処理がここに入る
}
finally {
  # リソースが存在している場合は解放
  if ($null -ne $Entry) { $Entry.Dispose() }
  if ($null -ne $Results) { $Results.Dispose() }
  if ($null -ne $Searcher) { $Searcher.Dispose() }
}
```
