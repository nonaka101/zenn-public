---
title: "🐥 LDAP を用いたユーザー情報の取得2"
---

前項では ActiveDirectory に検索クエリを出し、`GetDirectoryEntry()` を使ってエントリを取得する一連の流れについて説明しました。

本項では その続きとして、下記の内容について説明していきます。

- エントリから取得した情報を、利用しやすい形に変換する
- これまでの流れを関数として整理する

## ActiveDirectory 上の `objectSid` を利用しやすい形に変換する

前項では `sAMAccountName`, `displayName`, `userPrincipalName`, `distinguishedName` そして `objectSid` 属性の情報を ActiveDirectory にクエリを出して取得しました。

:::message

下記はそこまでの流れを簡略化したコードとなります。  
（一部の処理は説明に不要なので、省略したものとなっています）

```ps1
# DirectorySearcher を作成し、フィルタリングを設定
$Searcher = New-Object System.DirectoryServices.DirectorySearcher
$Searcher.Filter = "(&(objectCategory=person)(objectClass=user)(sAMAccountName=hoge001))"

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

# 検索実行し、エントリを取得
$Results = $Searcher.FindAll()
$Entry = $Results[0].GetDirectoryEntry()
```

:::

ただ、ここで取得した ActiveDirectory の `objectSid` はそのままでは使えず、様々な作業で利用できる `SID`（ *Security Identifer* ）にするために もうひと手間必要となってきます。

```ps1
# 〜（エントリを取得するところまでは省略）〜
$Entry = $Results[0].GetDirectoryEntry()

# バイナリ形式の objectSid 情報から、.NET での SecurityIdentifier 型に変換
$SidBytes = [byte[]]$Entry.Properties['objectSid'][0]
$Sid = New-Object System.Security.Principal.SecurityIdentifier($SidBytes, 0)
```

ここでは、ActiveDirectory から取得した `objectSid` を利用しやすい `SecurityIdentifier` 型（`System.Security.Principal.SecurityIdentifier`）に変換する処理について説明します。

### `SecurityIdentifier` について

https://learn.microsoft.com/ja-jp/dotnet/api/system.security.principal.securityidentifier

`SecurityIdentifier` は、`SID` を `.NET` の型として安全に扱えるようにするためのクラスとなります。

この `SecurityIdentifier` 型は ユーザー等を**一意に識別できる要素**の代表格となるため、例えば下記のようなことが行なえます。

- アクセス権関係（`ACL`）の諸処理
  - `ACL` の解析、設定など
  - ファイルだけでなくレジストリについてのアクセス制御
- アカウント名 ↔ `SID` 間で相互変換
- その他 `SID` を使った諸処理
  - `SID` 同士の比較
  - グループなどの種別判定（例：`Administrator` グループに所属しているか）
  - ウェルノウン `SID`（≒ 定義済みの特殊な `SID`）かの判定

### `objectSid` の実態

まず最初に、`objectSid` は `DirectoryEntry` 内の `Properties['objectSid']` に格納されています。

```ps1
# エントリの取得
$Entry = $Results[0].GetDirectoryEntry()

# エントリ内のプロパティ `objectSid`
$Entry.Properties['objectSid']
```

そして `DirectoryEntry.Properties['objectSid']` は、単一の値に見えても、PowerShell 上ではプロパティ値の**コレクション**として扱われています。

そのため、実際の値を取り出すために `[0]` を指定しています。

```ps1
# `objectSid` コレクションの中から、最初の要素を取り出す
$Entry.Properties['objectSid'][0]
```

そして、ActiveDirectory の `objectSid` データは、画面でよく見るような次の文字列形式でそのまま入っているわけではありません。

```txt
S-1-5-21-1234567890-123456789-123456789-1001
```

ActiveDirectory 上では、属性 `SID` が**バイナリ形式**で格納されています。

そのため、`DirectoryEntry` から取得した直後の値を、そのまま表示用の文字列として扱うのではなく、まず `byte[]` として取り出す必要があります。

```ps1
# エントリ内の `objectSid` を、明示的な型変換を使い バイト配列 として取り出す
$SidBytes = [byte[]]$Entry.Properties['objectSid'][0]
```

これで `SecurityIdentifier` 型のデータを作成するために必要な情報が取得できました。

### `SecurityIdentifier` への変換

https://learn.microsoft.com/ja-jp/dotnet/api/system.security.principal.securityidentifier.-ctor

`SecurityIdentifier` データを生成する際、必要となるのは 2 つの引数です。

- `binaryForm`（`Byte[]`）： SID を表すバイト配列
- `offset`（`Int32`）： `binaryForm` の開始インデックスとして使用するバイトオフセット値

今回取得している `$SidBytes` については 配列の先頭から読み取って大丈夫な形になっていますので、コードに起こすと下記のようになります。

```ps1
$SidBytes = [byte[]]$Entry.Properties['objectSid']

# バイト配列データから、SecurityIdentifier を生成
$Sid = New-Object System.Security.Principal.SecurityIdentifier($SidBytes, 0)
```

これで、ActiveDirectory 上の `objectSid` を `SecurityIdentifier` に変換する処理が完了しました。

#### 活用例：`NTAccount` に変換

生成した `SecurityIdentifier` の活用例として、ここでは `NTAccount` への変換を取り上げてみます。

`NTAccount` は、**Windows のユーザーやグループを表すためのアカウント名形式 を扱うもの**であり、実体は `SID` の値を `DOMAIN\User` の形で**人が読める形に変換したもの**となります。

- 例：`FUGA\hoge`
- 例：`BUILTIN\Administrators`（ローカルコンピューターの管理者グループ）
- 例：`NT AUTHORITY\SYSTEM`（Windows 自身が使用する特別なシステムアカウント）

`NTAccount` は ユーザーオブジェクトが持つ `SID` から変換することが可能です。下記のコードは、`NTAccount` 名を取得するまでの一連の流れとなります。

```ps1
# ここでは SID テキストから SecurityIdentifier を生成している（本項では バイト配列 から生成）
$Sid = New-Object System.Security.Principal.SecurityIdentifier(
    "S-1-5-18"
)

$account = $Sid.Translate( [System.Security.Principal.NTAccount] )
$account.Value  # -> NT AUTHORITY\SYSTEM
```

この `NTAccount` は、例えば ファイルやフォルダのアクセス権（`ACL`）を扱う際に使用します。

:::message

フォルダの ACL 設定は、`FileSystemAccessRule` に `SecurityIdentifier` を渡すことで処理できます。  
（アクセス制御に関する説明は `ACL` の項で別途行うので、ここではコードのみ掲載しています）

```ps1
# 事前に SID を `$Sid` に格納できているものとする

# 指定ユーザーに対する「変更」権限を、アクセスルールとして設ける
$Rule = New-Object System.Security.AccessControl.FileSystemAccessRule(
  $Sid,
  [System.Security.AccessControl.FileSystemRights]::Modify,
  [System.Security.AccessControl.InheritanceFlags]'ContainerInherit, ObjectInherit',
  [System.Security.AccessControl.PropagationFlags]::None,
  [System.Security.AccessControl.AccessControlType]::Allow
)
```

アクセス制御の実体は `SID` を使うのが通常ですが、作業者には `SID` だけでは誰に対するものなのかわからないという事情があります。

そのため **人が理解できる`NTAccount`（`DOMAIN\user`）情報を「表示用」として持っておくことは必要**と私は考えており、本項では両者を保持しておくことを想定しています。

:::

## ここまでのまとめ

ここまでの流れを踏まえて、まずは関数にする云々は気にせず、`sAMAccountName` からユーザーを検索する流れを順番に書いてみます。

1. `sAMAccountName` を受け取る（説明上の都合上 `hoge001` で固定）
2. LDAP フィルター用にエスケープする
3. `DirectorySearcher` を作る
4. `DirectorySearcher` に LDAP フィルターを設定する
5. `DirectorySearcher.PropertiesToLoad` に取得したい属性を追加する
6. `FindAll()` でクエリを発行し検索する
7. 検索結果を精査し、処理を分ける
   - 結果件数が 1 でなければ、処理からはじく
   - 結果件数が 1 であれば、`DirectoryEntry` を取得
8. `objectSid` を `SecurityIdentifier` に変換する
9. 必要に応じて `NTAccount` に変換する
10. 後続処理で使いやすいように `PSCustomObject` にまとめる
11. 使い終わったオブジェクトを `Dispose()` する

:::message

*10* 番については、このタイミングで初登場したものになります。ただ やっていることとしては、`[PSCustomObject]` を使って整理しているだけです。

:::

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

  # objectSid を SecurityIdentifier に変換
  $SidBytes = [byte[]]$Entry.Properties['objectSid'][0]
  $Sid = New-Object System.Security.Principal.SecurityIdentifier($SidBytes, 0)

  # 表示用に NTAccount へ変換
  $NtAccount = $Sid.Translate([System.Security.Principal.NTAccount])

  # ここまでで取得、加工した諸情報を オブジェクトの形に整理
  [pscustomobject]@{
    SamAccountName    = [string]$Entry.Properties['sAMAccountName'].Value
    DisplayName       = [string]$Entry.Properties['displayName'].Value
    UserPrincipalName = [string]$Entry.Properties['userPrincipalName'].Value
    DistinguishedName = [string]$Entry.Properties['distinguishedName'].Value
    Sid               = $Sid
    NtAccount         = $NtAccount.Value
  }
}
finally {
  # リソースが存在している場合は解放
  if ($null -ne $Entry) { $Entry.Dispose() }
  if ($null -ne $Results) { $Results.Dispose() }
  if ($null -ne $Searcher) { $Searcher.Dispose() }
}
```

### ポイント

本項では `System.DirectoryServices.DirectorySearcher` を使って、ActiveDirectory モジュールがない環境でも、LDAP 経由で AD 上のユーザーを検索する方法を取り扱いました。

ここまでで説明したものの内、ポイントとなるのは下記となります。

- `DirectorySearcher` は LDAP フィルターを使って ActiveDirectory を検索するための `.NET` クラス
- `sAMAccountName` や `mail` を条件にして、ユーザーを検索できる
- LDAP フィルターについて
  - ユーザー入力値を埋め込む場合は、エスケープ処理を施したほうが安全
  - `objectCategory=person` と `objectClass=user` を付けることで、対象をユーザーに絞ることができる
- `PropertiesToLoad` で取得する属性を明示すると、無駄が省かれ コードの意図がわかりやすくなる
- 検索には `FindOne()` と `FindAll()` があるが、一意に解決したい場合は `FindAll()` で件数を確認すると安全
- `SearchResult` から `GetDirectoryEntry()` で AD オブジェクトを取得できる
- `DirectorySearcher`、`SearchResultCollection`、`DirectoryEntry` は使い終わったら `Dispose()` する
- 取得した属性情報について
  - `objectSid` は ACL 制御等に必要になってくるが、`SecurityIdentifier` に変換して上げる必要がある
  - `SecurityIdentifier` を変換することで `NTAccount` を取得できる

:::message

ActiveDirectory モジュールが使えるなら、基本的には `Get-ADUser` を使う方が読みやすいです。

一方で、モジュールを追加できない端末や、.NET の機能だけで完結させたい場面では、`DirectorySearcher` を使う方法が実務上の選択肢になります。

:::

## サンプル：識別子からユーザーの諸属性を取得する

ここまでのコードは `sAMAccountName` のケースで説明をしてきました。一方で実務の場では、他にも `Mail` や `EmployeeId` などを使って同じことをしたい場面がでてきます。

そうしたことを踏まえ、ここでは **ある程度汎用的な関数**にまとめてみます。

```ps1
# LDAP 検索で使う入力値を エスケープ処理する関数
function ConvertTo-LdapEscapedFilterValue {
  <#
  .SYNOPSIS
    LDAP 検索フィルターに使用する文字列をエスケープします。

  .DESCRIPTION
    LDAP 検索フィルターの値に含まれる特殊文字を、安全に検索へ使用できる
    エスケープ表現へ変換します。

    バックスラッシュ、アスタリスク、左丸括弧、右丸括弧、および NULL 文字を
    それぞれ対応する 16 進形式のエスケープ表現へ置換します。

  .PARAMETER Value
    エスケープする文字列を指定します。空文字列も指定できます。

  .OUTPUTS
    System.String
    LDAP 検索フィルター用にエスケープされた文字列を返します。

  .EXAMPLE
    ConvertTo-LdapEscapedFilterValue -Value 'user*(test)'

    特殊文字をエスケープし、user\2a\28test\29 を返します。
  #>
  [CmdletBinding()]
  param(
    [Parameter(Mandatory = $true)]
    [AllowEmptyString()]
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


function Get-LdapPropertyValue {
  <#
  .SYNOPSIS
    DirectoryEntry から指定された AD 属性の値を安全に取得します。

  .DESCRIPTION
    指定された DirectoryEntry に対象のプロパティが存在し、1 件以上の値を
    保持している場合、その値を文字列として返します。

    プロパティが存在しない場合、または値が設定されていない場合は $null を
    返します。

  .PARAMETER Entry
    属性値を取得する System.DirectoryServices.DirectoryEntry オブジェクトを
    指定します。

  .PARAMETER PropertyName
    取得する AD 属性の名前を指定します。

  .OUTPUTS
    System.String
    指定された属性が存在する場合は、その値を文字列として返します。
    属性が存在しない場合、または値が未設定の場合は $null を返します。

  .EXAMPLE
    $Mail = Get-LdapPropertyValue -Entry $Entry -PropertyName 'mail'

    DirectoryEntry から mail 属性を取得します。属性が存在しない場合は
    $null を返します。
  #>
  [CmdletBinding()]
  param(
    [Parameter(Mandatory = $true)]
    [System.DirectoryServices.DirectoryEntry]$Entry,

    [Parameter(Mandatory = $true)]
    [string]$PropertyName
  )

  # プロパティが存在する場合はそれを返し、ない場合は $null を返す
  if ($Entry.Properties.Contains($PropertyName) -and $Entry.Properties[$PropertyName].Count -gt 0) {
    return [string]$Entry.Properties[$PropertyName].Value
  }
  return $null
}


function Resolve-AdUserByLdap {
  <#
  .SYNOPSIS
    LDAP 検索を使用して Active Directory ユーザーの属性を取得します。

  .DESCRIPTION
    指定された検索種別と検索文字列を使用して Active Directory のユーザーを
    LDAP で検索します。検索文字列は LDAP フィルター用にエスケープされます。

    検索結果が 1 件の場合は、ユーザーの主要な AD 属性、SID、および SID から
    変換した NTAccount を PSCustomObject として返します。NTAccount への変換に
    失敗した場合、NtAccount と NtAccountValue には $null を設定します。

    検索結果が 0 件または複数件の場合、あるいは取得したユーザーに objectSid が
    存在しない場合は、終了エラーを発生させます。検索に使用したリソースは、処理の
    成否にかかわらず解放されます。

  .PARAMETER SearchType
    ユーザー検索に使用する属性の種別を指定します。指定できる値は、
    SamAccountName、Mail、UserPrincipalName、DistinguishedName、EmployeeId です。

  .PARAMETER SearchValue
    ユーザー検索に使用する値を指定します。この値は、LDAP フィルターへ組み込む
    前に ConvertTo-LdapEscapedFilterValue によってエスケープされます。

  .OUTPUTS
    System.Management.Automation.PSCustomObject
    検索条件、ユーザーの AD 属性、SID、および NTAccount の情報を格納した
    オブジェクトを返します。

  .EXAMPLE
    $User = Resolve-AdUserByLdap -SearchType SamAccountName -SearchValue 'user01'

    sAMAccountName が user01 と一致する AD ユーザーを検索し、その属性情報を
    $User に格納します。

  .EXAMPLE
    $User = Resolve-AdUserByLdap -SearchType Mail -SearchValue 'user@example.com'

    mail 属性が user@example.com と一致する AD ユーザーを検索します。
  #>
  [CmdletBinding()]
  param(
    # 検索タイプ
    [Parameter(Mandatory = $true)]
    [ValidateSet(
      'SamAccountName',
      'Mail',
      'UserPrincipalName',
      'DistinguishedName',
      'EmployeeId'
    )]
    [string]$SearchType,

    # 検索文字列（関数内でエスケープ処理される）
    [Parameter(Mandatory = $true)]
    [ValidateNotNullOrEmpty()]
    [string]$SearchValue
  )

  # 入力 -SearchType を、LDAP 検索に使う要素に変換する
  $SearchAttributeMap = @{
    SamAccountName    = 'sAMAccountName'
    Mail              = 'mail'
    UserPrincipalName = 'userPrincipalName'
    DistinguishedName = 'distinguishedName'
    EmployeeId        = 'employeeID'
  }
  $LdapAttribute = $SearchAttributeMap[$SearchType]

  # ActiveDirectory から取得するユーザー属性のフィルタリング
  $PropertiesToLoad = @(
    'sAMAccountName',
    'displayName',
    'mail',
    'userPrincipalName',
    'distinguishedName',
    'employeeID',
    'objectSid'
  )

  # 各種リソースの初期化
  $Searcher = $null
  $Results = $null
  $Entry = $null

  try {
    # -----------------------------------
    # LDAP を用いた検索クエリの発行
    # -----------------------------------

    $EscapedSearchValue = ConvertTo-LdapEscapedFilterValue -Value $SearchValue

    $Searcher = New-Object System.DirectoryServices.DirectorySearcher
    $Searcher.Filter = "(&(objectCategory=person)(objectClass=user)($LdapAttribute=$EscapedSearchValue))"

    foreach ($PropertyName in $PropertiesToLoad) {
      [void]$Searcher.PropertiesToLoad.Add($PropertyName)
    }

    $Results = $Searcher.FindAll()

    # -----------------------------------
    # 検索結果の確認とエントリの取得
    # -----------------------------------

    # 結果が想定（ヒットが 1 件のみ）でない場合は、ここで はじく
    if ($Results.Count -eq 0) {
      throw "検索条件 '$SearchType = $SearchValue' に一致するユーザーは ActiveDirectory 上で見つかりませんでした。"
    }
    if ($Results.Count -gt 1) {
      throw "検索条件 '$SearchType = $SearchValue' に一致するユーザーが ActiveDirectory 上で複数件見つかりました。"
    }

    $Entry = $Results[0].GetDirectoryEntry()

    # -----------------------------------
    # SecurityIdentifier の取得
    # -----------------------------------

    # 生成に必要な `objectSid` を持っていない場合、ここで はじく
    if (-not $Entry.Properties.Contains('objectSid') -or $Entry.Properties['objectSid'].Count -eq 0) {
      throw "検索条件 '$SearchType = $SearchValue' で取得したユーザーに objectSid が存在しません。"
    }

    $SidBytes = [byte[]]$Entry.Properties['objectSid'][0]
    $Sid = New-Object System.Security.Principal.SecurityIdentifier($SidBytes, 0)

    # -----------------------------------
    # SecurityIdentifier から NTAccount へ変換
    # -----------------------------------

    $NtAccount = $null
    $NtAccountValue = $null

    # NTAccount に変換できればそれを、できなければ $null を設定
    try {
      $NtAccount = $Sid.Translate([System.Security.Principal.NTAccount])
      $NtAccountValue = $NtAccount.Value
    }
    catch {
      $NtAccount = $null
      $NtAccountValue = $null
    }

    # -----------------------------------
    # 取得した属性情報等を オブジェクトとして返す
    # -----------------------------------

    # 一部のプロパティは AD 属性がないケースもありえるので、`Get-LdapPropertyValue` で安全に取り出す
    return [pscustomobject]@{
      SearchType        = $SearchType
      SearchValue       = $SearchValue
      SamAccountName    = Get-LdapPropertyValue -Entry $Entry -PropertyName 'sAMAccountName'
      DisplayName       = Get-LdapPropertyValue -Entry $Entry -PropertyName 'displayName'
      Mail              = Get-LdapPropertyValue -Entry $Entry -PropertyName 'mail'
      UserPrincipalName = Get-LdapPropertyValue -Entry $Entry -PropertyName 'userPrincipalName'
      DistinguishedName = Get-LdapPropertyValue -Entry $Entry -PropertyName 'distinguishedName'
      EmployeeId        = Get-LdapPropertyValue -Entry $Entry -PropertyName 'employeeID'
      Sid               = $Sid
      SidValue          = $Sid.Value
      NtAccount         = $NtAccount
      NtAccountValue    = $NtAccountValue
    }
  }
  finally {
    # リソースの解放
    if ($null -ne $Entry) { $Entry.Dispose() }
    if ($null -ne $Results) { $Results.Dispose() }
    if ($null -ne $Searcher) { $Searcher.Dispose() }
  }
}
```

使用する際は、一意で識別できる情報を関数に渡すようにします。具体例としては、下記のようになります。

```ps1
# SAM アカウント名から
$User = Resolve-AdUserByLdap -SearchType SamAccountName -SearchValue 'hoge001'

# メールアドレスから
$User = Resolve-AdUserByLdap -SearchType Mail -SearchValue 'yamada@fuga.jp'

# UPN から
$User = Resolve-AdUserByLdap -SearchType UserPrincipalName -SearchValue 'hoge001@fuga.jp'

# DistinguishedName から
$User = Resolve-AdUserByLdap -SearchType DistinguishedName -SearchValue 'CN=hoge001,OU=Users,DC=fuga,DC=jp'

# EmployeeID から
$User = Resolve-AdUserByLdap -SearchType EmployeeId -SearchValue 'E00123'
```

問題なく処理できた場合、ActiveDirectory から取得した諸属性の情報がオブジェクトとして格納されています。イメージとしては、下記のような感じです。

```ps1
$User = Resolve-AdUserByLdap -SearchType Mail -SearchValue 'yamada@fuga.jp'
$User | Format-List *
<#
SearchType        : Mail
SearchValue       : yamada@fuga.jp
SamAccountName    : hoge001
DisplayName       : 山田 太郎
Mail              : yamada@fuga.jp
UserPrincipalName : hoge001@fuga.jp
DistinguishedName : CN=山田 太郎,OU=Users,DC=fuga,DC=jp
EmployeeId        : E00123
Sid               : S-1-5-21-...
SidValue          : S-1-5-21-...
NtAccount         : FUGA\hoge001
NtAccountValue    : FUGA\hoge001
#>
```
