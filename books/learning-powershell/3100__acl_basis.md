---
title: "🐓 ACL：基礎"
---

ここでは、Windows のファイルシステムに設定されたアクセス許可を確認・変更する基本的な方法について説明します。

- ACL と ACE の関係について
- `Get-Acl` を使って、ファイルやフォルダのアクセス許可を確認する
- ACE に含まれる主なプロパティを読み取ってみる
- `FileSystemAccessRule` を使って新しいアクセス規則を作成する
- 新しいアクセス規則を ACL オブジェクトへ追加する
- `Set-Acl` を使って、変更後の内容をフォルダへ反映する

アクセス許可そのものの考え方や、Windows 上での設定方法については、「ファイル アクセスの構成と管理 - Training（Microsoft Learn）」が参考になります。

https://learn.microsoft.com/ja-jp/training/modules/configure-manage-file-access/

:::message alert

`Set-Acl` によるアクセス許可の変更は、誤った対象や設定で実行すると、必要なファイルへアクセスできなくなる可能性があります。

本稿の内容は、削除しても問題ない場所で行うようにしてください。

:::

## ACL について

### セキュリティ記述子とは

https://learn.microsoft.com/ja-jp/windows-hardware/drivers/ifs/security-descriptors

Windows において、ファイルやフォルダのセキュリティ情報は、**セキュリティ記述子**という情報のまとまりで管理されています。

セキュリティ記述子には、主に次の情報が含まれます。

```txt
セキュリティ記述子
├─ 所有者
├─ DACL（誰に、どのアクセスを許可または拒否するか）
└─ SACL（どのアクセスを監査するか）
```

PowerShell の `Get-Acl` は、指定したファイルやフォルダのセキュリティ記述子を表すオブジェクトを取得します。

:::message

本書では説明を簡潔にするため、`Get-Acl` で取得したオブジェクトを **ACL オブジェクト** と呼ぶことがあります。

:::

### ACL とは

**ACL**（Access Control List）とは、**アクセス制御**に関する情報の一覧です。

ACL には、主に次の 2 種類があります。

| 種類 | 用途 |
| :--- | :--- |
| **DACL**（ *Discretionary Access Control List* ） | 誰に、*どのアクセスを許可または拒否するか* を管理する |
| **SACL**（ *System Access Control List* ） | どのアクセスを *監査記録の対象にするか* を管理する |

この資料では、ファイルやフォルダへのアクセス許可を設定するため、**DACL を中心に扱います**。

:::message

SACL については、監査に使用する ACL が存在することを覚えておけば十分です。

:::

### ACE とは

ACL は、複数の **ACE**（ *Access Control Entry* ）によって構成されています。

ACE は、**1 件分のアクセス規則**となります。

- ACL（ *Access Control **List*** ）：アクセス制御情報の **一覧**
- ACE（ *Access Control **Entry*** ）：ACL に含まれる **1 件の**アクセス規則

:::message

例えば、あるフォルダに次のアクセス許可を設定したとします。

- `Administrators` にフルコントロールを許可する
- `Users` に読み取りを許可する
- `Backup Operators` に変更を許可する

これらは、それぞれ個別の ACE として DACL に登録されます。

```txt
DACL
├─ ACE：BUILTIN\Administrators に FullControl を許可
├─ ACE：BUILTIN\Users に ReadAndExecute を許可
└─ ACE：BUILTIN\Backup Operators に Modify を許可
```

:::

## 演習用フォルダを準備する

以降のコードでは、`C:\AclTraining` を演習用フォルダとして使用します。

```ps1
$folderPath = 'C:\AclTraining'

# フォルダが存在するか確認し、存在しない場合だけ `New-Item` で作成
if (-not (Test-Path -LiteralPath $folderPath)) {
  New-Item -Path $folderPath -ItemType Directory | Out-Null
}

# フォルダ構成の確認
Get-Item -LiteralPath $folderPath
<#
    ディレクトリ: C:\

Mode                LastWriteTime         Length Name
----                -------------         ------ ----
d-----       2026/08/25     10:00                AclTraining
#>
```

## `Get-Acl` で現在の状態を確認する

### ACL オブジェクトを取得する

次のコードでは、`Get-Acl` を使って演習用フォルダのセキュリティ記述子を取得し、`$acl` 変数へ格納しています。

```ps1
$acl = Get-Acl -LiteralPath $folderPath
$acl
<#
    ディレクトリ: C:\

Path         Owner                  Access
----         -----                  ------
AclTraining  DOMAIN\Administrator   NT AUTHORITY\SYSTEM Allow  FullControl...
#>
```

### DACL に含まれる ACE を確認する

`Get-Acl` の実行結果には、所有者やアクセス規則などが含まれています。

アクセス規則の詳細を確認するには、`Access` プロパティを使用します。

```ps1
# アクセス規則の一覧を出力
$acl.Access | Format-List *
<#
FileSystemRights  : FullControl
AccessControlType : Allow
IdentityReference : NT AUTHORITY\SYSTEM
IsInherited       : True
InheritanceFlags  : ContainerInherit, ObjectInherit
PropagationFlags  : None

FileSystemRights  : FullControl
AccessControlType : Allow
IdentityReference : BUILTIN\Administrators
IsInherited       : True
InheritanceFlags  : ContainerInherit, ObjectInherit
PropagationFlags  : None

FileSystemRights  : ReadAndExecute
AccessControlType : Allow
IdentityReference : BUILTIN\Users
IsInherited       : True
InheritanceFlags  : ContainerInherit, ObjectInherit
PropagationFlags  : None
#>
```

:::message

`Access` プロパティには、このように DACL に登録されているアクセス規則の一覧が格納されています。  
（出力される内容は、フォルダの作成場所や親フォルダの設定によって異なります）

出力した結果の内、1 つのまとまり（`FileSystemRights` ～ `PropagationFlags`）が 1 つの ACE に対応しています。

:::

### オブジェクトの型を確認する

PowerShell では、コマンドの実行結果がオブジェクトとして扱われます。

ここまでで扱ったオブジェクトに対し、`.GetType()` を使い型を確認してみます。すると次のことがわかります。

- フォルダに対する `Get-Acl` の結果は、`DirectorySecurity` オブジェクトである
- `Access` プロパティ内の各アクセス規則は、`FileSystemAccessRule` オブジェクトである

```ps1
$acl.GetType().FullName # -> System.Security.AccessControl.DirectorySecurity
$acl.Access[0].GetType().FullName # -> System.Security.AccessControl.FileSystemAccessRule
```

## ACE に含まれる情報を理解する

ACE には、「誰に」「どの権限を」「許可するのか、拒否するのか」などの情報が含まれています。

```ps1
## ACL から 1 件のみ ACE を取り出して表示
$acl.Access | Select-Object -First 1 | Format-List *
<#
FileSystemRights  : FullControl
AccessControlType : Allow
IdentityReference : NT AUTHORITY\SYSTEM
IsInherited       : True
InheritanceFlags  : ContainerInherit, ObjectInherit
PropagationFlags  : None
#>
```

プロパティの内容は、下表のとおりです。

| プロパティ | 内容 |
| :--- | :--- |
| `FileSystemRights` | 許可または拒否する**アクセス権** |
| `AccessControlType` | アクセスを**許可**するか、**拒否**するか |
| `IdentityReference` | アクセス規則の対象となる**ユーザーまたはグループ** |
| `IsInherited` | ACE が**親フォルダから継承されたものか** |
| `InheritanceFlags` | ACE を**配下のファイルやサブフォルダへ継承させるか** |
| `PropagationFlags` | 継承を**どこまで伝播させるか** |

### `FileSystemRights`

https://learn.microsoft.com/ja-jp/dotnet/api/system.security.accesscontrol.filesystemrights

`FileSystemRights` は、ファイルやフォルダに対して許可または拒否する**操作**を表します。代表的な値は次のとおりです。

| 値 | 概要 |
| :--- | :--- |
| `Read` | データや属性、アクセス許可などを読み取る |
| `ReadAndExecute` | 読み取りに加えて、ファイルの実行やフォルダの走査を行う |
| `Write` | ファイルの作成やデータ、属性の書き込みを行う |
| `Modify` | 読み取り、書き込み、実行、削除などを行う |
| `FullControl` | アクセス許可の変更や所有権の取得を含む、すべての操作を行う |

:::message

`ReadAndExecute`、`Modify`、`FullControl` などは、複数の基本的な権限を組み合わせた値です。その仕組みは、[文書末尾の備考](#filesystemrights-とビットフラグ)で説明します。

:::

### `AccessControlType`

https://learn.microsoft.com/ja-jp/dotnet/api/system.security.accesscontrol.accesscontroltype

`AccessControlType` は、指定したアクセス権を**許可**するか、**拒否**するかを表します。

| 値 | 説明 |
| :--- | :--- |
| `Allow` | 指定したアクセス権を許可する |
| `Deny` | 指定したアクセス権を拒否する |

:::message alert

`Deny` は**扱いが非常に難しい**ので、実務上でも「触れない」といった判断をしているケースが多くあります。

所属しているグループや継承関係、「許可」「拒否」の場合での影響などが組み合わさり、意図せぬアクセス挙動に繋がる原因だったりします。

:::

### `IdentityReference`

https://learn.microsoft.com/ja-jp/dotnet/api/system.security.principal.identityreference

`IdentityReference` は、アクセス規則の対象となる**ユーザーまたはグループ**を表します。例えば次のようなものです。

- `DOMAIN\hoge`
- `BUILTIN\Administrators`
- `NT AUTHORITY\SYSTEM`

:::message

実務においては、個人ユーザーに直接権限を設定するのではなく、用途ごとに作成したグループへ権限を設定する方式が一般的です。

:::

### `IsInherited`

https://learn.microsoft.com/ja-jp/dotnet/api/system.security.accesscontrol.authorizationrule.isinherited

`IsInherited` は、その ACE が **親フォルダから継承されたものか** を表します。

| 値 | 説明 |
| :--- | :--- |
| `True` | 親フォルダから**継承された** ACE |
| `False` | 現在のファイルまたはフォルダで**直接設定された** ACE |

### `InheritanceFlags`

https://learn.microsoft.com/ja-jp/dotnet/api/system.security.accesscontrol.inheritanceflags

`InheritanceFlags` は、その ACE を対象フォルダの **配下にあるファイルやサブフォルダへ継承させるか** を表します。

| 値 | ACE を継承する対象 |
| :--- | :--- |
| `None` | 配下へ継承しない |
| `ObjectInherit` | 配下のファイル |
| `ContainerInherit` | 配下のサブフォルダ |
| `ContainerInherit, ObjectInherit` | 配下のファイルとサブフォルダ |

:::message

`InheritanceFlags` と `IsInherited` は名前が似ていますが、意味が異なります。

- `InheritanceFlags`：この ACE を、配下のファイルやサブフォルダへ継承させるか（列挙体）
- `IsInherited`：この ACE が、親フォルダから継承されてきたものか（真偽値）

:::

### `PropagationFlags`

https://learn.microsoft.com/ja-jp/dotnet/api/system.security.accesscontrol.propagationflags

`PropagationFlags` は、`InheritanceFlags` で **指定した継承を、どこまで伝播させるか** を表します。

| 値 | 説明 |
| :--- | :--- |
| `None` | `InheritanceFlags` で指定した対象へ、通常どおり継承する |
| `NoPropagateInherit` | 直下の子にのみ継承し、それより下の階層へ伝播させない |
| `InheritOnly` | ACE を現在の対象には適用せず、子にのみ継承させる |

:::message

`PropagationFlags` は、`InheritanceFlags` で継承先が指定されている場合に意味を持ちます。

実務上では、`None` を使用することが一般的と思います。

:::

## アクセス規則を追加する流れ

フォルダへ新しいアクセス規則を追加する流れは、次のようなイメージです。

![ACL のイメージ図](/images/books/learning-powershell/acl-01.png)

実務での 具体的な手順を示すと、下記のようになります。

1. `FileSystemAccessRule` を使って新しいアクセス規則を作成する
2. `Get-Acl` で現在のセキュリティ記述子を取得する
3. `AddAccessRule()` で、ACL オブジェクトへアクセス規則を追加する
4. `Set-Acl` で、変更後のセキュリティ記述子を対象フォルダへ反映する
5. `Get-Acl` で対象フォルダを再取得し、反映結果を確認する

## アクセス規則を追加する

### 今回追加するアクセス規則

演習では、PowerShell を実行している現在のユーザーに、次のアクセス規則を追加します。

| 設定項目 | 設定値 |
| :--- | :--- |
| 対象 | 現在 PowerShell を実行しているユーザー |
| アクセス権 | `Modify` |
| ファイルへの継承 | あり |
| サブフォルダへの継承 | あり |
| 伝播の追加制限 | なし |
| 許可または拒否 | `Allow` |

現在のユーザー名は、次のコードで取得できます。

```ps1
$currentUser = [System.Security.Principal.WindowsIdentity]::GetCurrent().Name
$currentUser # -> DOMAIN\hoge
```

### 新しいアクセス規則を作成する

`FileSystemAccessRule` のコンストラクターへ、アクセス規則を構成する 5 つの値を渡します。

```ps1
$accessRule = [System.Security.AccessControl.FileSystemAccessRule]::new(
  $currentUser,                          # アクセス規則の対象
  'Modify',                              # 許可するアクセス権
  'ContainerInherit, ObjectInherit',     # ファイルとサブフォルダの両方へ継承
  'None',                                # 継承の伝播に追加の制限を設けない
  'Allow'                                # アクセスを許可
)

## 作成したアクセス規則を表示
$accessRule | Format-List *
<#
FileSystemRights  : Modify, Synchronize
AccessControlType : Allow
IdentityReference : DOMAIN\hoge
IsInherited       : False
InheritanceFlags  : ContainerInherit, ObjectInherit
PropagationFlags  : None
#>
```

:::message

`FileSystemRights` に `Synchronize` が併記されることがあります。これは、`.NET` が許可 ACE を作成するときに、指定した権限に応じて `Synchronize` を含めることがあるためです。

:::

### ACL オブジェクトへアクセス規則を追加する

対象フォルダの現在の ACL オブジェクトを `Get-Acl` で取得し、先ほど作成したアクセス規則を `AddAccessRule()` メソッドを使い追加します。

```ps1
# 対象フォルダの ACL を取得
$acl = Get-Acl -LiteralPath $folderPath

# 取得した ACL に、アクセス規則を追加する
$acl.AddAccessRule($accessRule)
```

:::message

`AddAccessRule()` を実行した時点では、メモリ上の `$acl` オブジェクトが変更されただけです。

対象フォルダへの反映を確定させるには、`Set-Acl` を実行する必要があります。

:::

### `Set-Acl` で対象フォルダへ反映する

`Set-Acl` の `-AclObject` パラメーターへ、変更後の `$acl` オブジェクトを渡します。

```ps1
# 渡されたセキュリティ記述子を、対象フォルダへ反映
Set-Acl -LiteralPath $folderPath -AclObject $acl
```

:::message

`Set-Acl` は「指定した ACE を 1 件だけ追加するコマンドレット」ではなく、「`$acl` に含まれるセキュリティ記述子の内容を対象へ適用するコマンドレット」です。

なので、仮に別の場所にあるフォルダの `$acl` を持ってきて `Set-Acl` で適用してしまうと、全く異なるアクセス権の設定に書き換わることになります。

:::

## 反映結果を確認する

`Set-Acl` の実行後は、`Get-Acl` をもう一度実行し、対象フォルダへ反映された内容を確認します。

```ps1
$updatedAcl = Get-Acl -LiteralPath $folderPath
$updatedAcl | Format-List *
<#
--- 他の ACE は省略 ---

IdentityReference : DOMAIN\hoge
FileSystemRights  : Modify, Synchronize
AccessControlType : Allow
InheritanceFlags  : ContainerInherit, ObjectInherit
PropagationFlags  : None
IsInherited       : False

--- 他の ACE は省略 ---
#>
```

`IsInherited` が `False` であるため、この ACE は演習用フォルダへ直接設定されたものだと分かります。

:::message

直接目的の情報（`$currentUser` の `Modify` 権限）を取り出したい場面の時には、下記のように表現できます。  
（条件を絞って抽出しているので、**実効アクセス権**とは違うことには注意が必要です。実際のアクセス権は、所属グループ、継承、拒否の ACE が影響してきます）

```ps1
$updatedRules = Get-Acl -LiteralPath $folderPath |
  Select-Object -ExpandProperty Access |
  Where-Object {
    $_.IdentityReference.Value -eq $currentUser -and
    $_.FileSystemRights -band [System.Security.AccessControl.FileSystemRights]::Modify
  }

$updatedRules | Format-List *
<#
IdentityReference : DOMAIN\hoge
FileSystemRights  : Modify, Synchronize
AccessControlType : Allow
InheritanceFlags  : ContainerInherit, ObjectInherit
PropagationFlags  : None
IsInherited       : False
#>
```

:::

## 一連のコード

ここまでの処理をまとめると、次のようになります。

```ps1
## 対象となる演習用フォルダ
$folderPath = 'C:\AclTraining'

## 現在 PowerShell を実行しているユーザーを取得
$currentUser = [System.Security.Principal.WindowsIdentity]::GetCurrent().Name

## 1. 対象フォルダの現在の ACL オブジェクトを取得
$acl = Get-Acl -LiteralPath $folderPath

## 2. 新しいアクセス規則を作成
$accessRule = [System.Security.AccessControl.FileSystemAccessRule]::new(
  $currentUser,
  'Modify',
  'ContainerInherit, ObjectInherit',
  'None',
  'Allow'
)

## 3. メモリ上の ACL オブジェクトへアクセス規則を追加
$acl.AddAccessRule($accessRule)

## 4. 変更後の内容を対象フォルダへ反映
Set-Acl -LiteralPath $folderPath -AclObject $acl

## 5. 対象フォルダを再取得し、追加したアクセス規則を確認
Get-Acl -LiteralPath $folderPath |
  Select-Object -ExpandProperty Access |
  Where-Object {
    $_.IdentityReference.Value -eq $currentUser -and
    $_.FileSystemRights -band [System.Security.AccessControl.FileSystemRights]::Modify
  } |
  Format-List *
```

## まとめ

ファイルまたはフォルダのアクセス規則を追加するときは、次の処理を行います。

1. `Get-Acl` で現在のセキュリティ記述子を取得する
2. `FileSystemAccessRule` で新しいアクセス規則を作成する
3. `AddAccessRule()` で ACL オブジェクトへ追加する
4. `Set-Acl` で対象へ反映する
5. `Get-Acl` で再取得し、結果を確認する

それぞれの役割を整理すると、次のようになります。

| 処理 | 役割 |
| :--- | :--- |
| `Get-Acl` | セキュリティ記述子を取得する |
| `FileSystemAccessRule` | 新しいアクセス規則を作成する |
| `AddAccessRule()` | メモリ上の ACL オブジェクトへアクセス規則を追加する |
| `Set-Acl` | 変更後のセキュリティ記述子を対象へ反映する |
| （再度の）`Get-Acl` | 実際に反映された結果を確認する |

最も重要なのは、`AddAccessRule()` と `Set-Acl` の役割を区別することです。

- `AddAccessRule()` は、メモリ上の ACL オブジェクトを変更する
- `Set-Acl` は、その内容を実際のファイルやフォルダへ反映する

## 備考

### `FileSystemRights` とビットフラグ

https://learn.microsoft.com/ja-jp/dotnet/api/system.security.accesscontrol.filesystemrights#-----

この備考では、`FileSystemRights` の内部的な仕組みを説明します。

:::message

基本的な ACL 操作だけを学ぶ場合は、この備考を読み飛ばしても問題ありません。

:::

#### `FileSystemRights` 列挙型

`FileSystemRights` は、列挙型として定義されています。利用可能な値は、次の通りです。

```ps1
[System.Enum]::GetNames(
  [System.Security.AccessControl.FileSystemRights]
)
<#
ListDirectory, ReadData, WriteData, CreateFiles, AppendData, CreateDirectories, ReadExtendedAttributes,
WriteExtendedAttributes, ExecuteFile, Traverse, DeleteSubdirectoriesAndFiles, ReadAttributes,
WriteAttributes, Write, Delete, ReadPermissions, Read, ReadAndExecute, Modify, ChangePermissions,
TakeOwnership, Synchronize, FullControl
#>
```

#### 整数値と 2 進数を確認する

列挙値を `[int]` へ変換すると、内部で使用される整数値を確認できます。2 進数として確認する場合は、`[Convert]::ToString()` を使用します。

```ps1
$rights = [System.Security.AccessControl.FileSystemRights]::Modify

# 10 進数で確認
[int]$rights  # -> 197055

# 2 進数で確認
[Convert]::ToString([int]$rights, 2).PadLeft(32, '0')
<#
00000000000000110000000110111111
#>
```

各ビットが、特定のアクセス権を表すフラグとして使用されています。

##### `FileSystemRights` の実体

様々ある `FileSystemRights` 列挙体ですが、その実体は 2 bit のフラグ集合体です。

```txt
111110000001111111111 : FullControl
000000000000000000001 : ListDirectory / ReadData
000000000000000000010 : CreateFiles / WriteData
000000000000000000100 : AppendData / CreateDirectories
000000000000000001000 : ReadExtendedAttributes
000000000000000010000 : WriteExtendedAttributes
000000000000000100000 : ExecuteFile / Traverse
000000000000001000000 : DeleteSubdirectoriesAndFiles
000000000000010000000 : ReadAttributes
000000000000100000000 : WriteAttributes
000010000000000000000 : Delete
000100000000000000000 : ReadPermissions
001000000000000000000 : ChangePermissions
010000000000000000000 : TakeOwnership
```

普段使っている下記の権限は、その実は上記の権限の組み合わせであることがわかります。

```txt
000000000000100010110 : Write
000100000000010001001 : Read
000100000000010101001 : ReadAndExecute
000110000000101111111 : Modify
```

### 型を明示してアクセス規則を作成する

本文では分かりやすさを優先し、コンストラクターへ文字列を渡しました。

```ps1
$accessRule = [System.Security.AccessControl.FileSystemAccessRule]::new(
    $currentUser,
    'Modify',
    'ContainerInherit, ObjectInherit',
    'None',
    'Allow'
)
```

.NET 側では、これらの文字列が対応する列挙型へ変換されます。

記述量は増えますが、型を明示して作成することも可能です。次のように記述でき、各引数の型と使用可能な値が明確にできます。

```ps1
$inheritanceFlags = [System.Security.AccessControl.InheritanceFlags]::ContainerInherit -bor
  [System.Security.AccessControl.InheritanceFlags]::ObjectInherit

$accessRule = [System.Security.AccessControl.FileSystemAccessRule]::new(
  $currentUser,
  [System.Security.AccessControl.FileSystemRights]::Modify,
  $inheritanceFlags,
  [System.Security.AccessControl.PropagationFlags]::None,
  [System.Security.AccessControl.AccessControlType]::Allow
)
```

### 実際の業務で使用するときの注意

実際の業務で ACL を変更する場合、場合によっては影響範囲が大きく取り返しのつかない結果になりかねません。

そのため 慣れていない段階では、次の点にも注意する良いかもしれません。

#### 変更前の ACL を記録する

変更前の状態を確認できるように、ACL や SDDL（セキュリティ記述子定義言語）を記録します。

```ps1
$beforeAcl = Get-Acl -LiteralPath $folderPath
$beforeAcl.Sddl | Out-File -LiteralPath 'C:\AclTraining-before-sddl.txt' -Encoding utf8
```

:::message

ただし、SDDL 文字列を保存しただけでは、自動的に復元されるわけではありません。

復元方法も含め、事前に手順を検証しておく必要があります。

:::

#### 対象パスを検証する

変数の値が空文字列、想定外のパス、ドライブのルートなどになっていないことを確認します。

実務においては、処理の実行前に**確認プロンプトで対象パスを明示する**など行うと良いでしょう。

#### 変更後に再取得する

`Set-Acl` でエラーが表示されなかった場合でも、`Get-Acl` で実際の状態を再取得して確認します。

これは ACL の考え方が他と異なり、メモリ上の `$acl` を変更・表示するだけで完結したものと思い込みやすいからです。

#### 実効アクセス権と ACE 一覧を区別する

`$acl.Access` で確認できるのは、DACL に含まれる ACE の一覧です。

ユーザーの実効アクセス権を判断するには、次のような要素も考慮する必要があります。

- ユーザーが所属するグループ
- 親フォルダから継承された ACE
- `Allow` と `Deny` の組み合わせ
- 共有フォルダを使用する場合の共有アクセス許可
- 所有者や特権による影響

特定ユーザー名の ACE だけを検索しても、実効アクセス権を完全には判断できません。

#### グループを使って権限を管理する

実務では、個人ユーザーへ直接 ACE を設定する方法よりも、役割や用途に応じたグループへ ACE を設定する方法を優先します。

例えば、次のようなグループを用意します。こうすることで、利用者の変更時はグループメンバーシップを更新することで、ファイルシステム側の ACL を頻繁に変更せずに済むようになります。

```txt
DOMAIN\FS-Accounting-Read
DOMAIN\FS-Accounting-Modify
```

#### `Deny` を慎重に使用する

`Deny` の ACE は、グループ経由で付与された `Allow` にも影響する可能性があります。

単にアクセスさせたくない場合は、まず不要な `Allow` を付与しない設計を検討すべきです。

#### 継承された ACE を不用意に変更しない

`IsInherited` が `True` の ACE は、親フォルダから継承されています。

子フォルダだけを見て削除や変更を試みるのではなく、どの親フォルダから継承されているかを確認してください。

継承の無効化は、配下のアクセス許可へ広く影響する可能性があります。

### 演習用フォルダの後片付け

演習が終了したら、必要に応じて演習用フォルダを削除します。

:::message alert

次のコードは、指定したフォルダと、その配下のファイルやサブフォルダを削除します。

変数の値が `C:\AclTraining` であることを確認してから実行してください。

:::

```ps1
$folderPath = 'C:\AclTraining'
if ($folderPath -ne 'C:\AclTraining') {
    throw "削除を許可していないパスです: $folderPath"
}
if (Test-Path -LiteralPath $folderPath -PathType Container) {
    Remove-Item -LiteralPath $folderPath -Recurse -Force
}

# 削除されたことを確認
Test-Path -LiteralPath 'C:\AclTraining' # -> False
```
