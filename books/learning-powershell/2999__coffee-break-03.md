---
title: "☕ Coffee Break : 3"
---

## スクリプトの安全な実行

実践的なスクリプトになればなるほど、**副作用**（≒ システムに大きな変更を加える）のリスクが高まります。

ADのユーザー情報を一括更新したり、ACLを一斉に書き換えたりするスクリプトは、一歩間違えれば大事故に直面します。

PowerShell では、こうした**副作用のあるコマンドレット**に対し `-WhatIf` や `-Confirm` といったオプションを提供しています。

### テスト実行する（`-WhatIf`）

PowerShell において、`Set-*` や `Remove-*`、`Add-*` といった **システムを変更するようなコマンドレット**のほとんどに、`-WhatIf` という強力な共通パラメータが用意されています。

これをコマンドの最後につけて実行すると、**「実際には変更を行わず、もし実行したら何が起こるか」をコンソールにテキストで表示してくれます**。

```ps1
# 実際には削除されない。削除されるはずだった対象が表示されるだけ。
Remove-ADUser -Identity 'hoge' -WhatIf
<#
What if: 対象 "CN=hoge,OU=Sales,DC=fuga,DC=jp" に対して操作 "Remove" を実行しています。
#>
```

スクリプトを作る際は、まずは `-WhatIf` をつけてテスト実行し、意図した対象だけに処理が向かっているかを確認する癖をつけると良いでしょう。

### 実行前に確認を求める（`-Confirm`）

`-WhatIf` と似ていますが、こちらは実行する前に「本当に実行しますか？」という**確認プロンプトを出してくれるパラメータ**です。

```ps1
# 実行する前に確認プロンプトが表示される
Remove-ADUser -Identity 'hoge' -Confirm
<#
確認
この操作を実行しますか?
対象 "CN=hoge,OU=Sales,DC=fuga,DC=jp" に対して操作 "Remove" を実行しています。
[Y] はい(Y)  [A] すべて続行(A)  [N] いいえ(N)  [L] すべて無視(L)  [S] 中断(S)  [?] ヘルプ (既定値は "Y"):
#>
```

重要な変更を（一括で）行うスクリプトの中で、特に慎重に処理したい箇所には意図的に `-Confirm` をつけておくことで、暴走を防ぐことができます。

:::message

なお `-Confirm $false` と言った形にすると、本来確認プロンプトが表示されるべき処理でも、それを無視して処理を実行することができます。

:::

## 権限と特権

現在のセッションで持っている特権を確認したい場合は、`whoami /priv` を使います。

```ps1
whoami /priv
<#
PRIVILEGES INFORMATION
----------------------

特権名                                    説明                                                   状態
========================================= ====================================================== ====
SeIncreaseQuotaPrivilege                  プロセスのメモリ クォータの増加                        無効
SeSecurityPrivilege                       監査とセキュリティ ログの管理                          無効
SeTakeOwnershipPrivilege                  ファイルとその他のオブジェクトの所有権の取得           無効
SeLoadDriverPrivilege                     デバイス ドライバーのロードとアンロード                無効
SeSystemProfilePrivilege                  システム パフォーマンスのプロファイル                  無効
SeSystemtimePrivilege                     システム時刻の変更                                     無効
SeProfileSingleProcessPrivilege           単一プロセスのプロファイル                             無効
SeIncreaseBasePriorityPrivilege           スケジューリング優先順位の繰り上げ                     無効
SeCreatePagefilePrivilege                 ページ ファイルの作成                                  無効
SeBackupPrivilege                         ファイルとディレクトリのバックアップ                   無効
SeRestorePrivilege                        ファイルとディレクトリの復元                           無効
SeShutdownPrivilege                       システムのシャットダウン                               無効
SeDebugPrivilege                          プログラムのデバッグ                                   有効
SeSystemEnvironmentPrivilege              ファームウェア環境値の修正                             無効
SeChangeNotifyPrivilege                   走査チェックのバイパス                                 有効
SeRemoteShutdownPrivilege                 リモート コンピューターからの強制シャットダウン        無効
SeUndockPrivilege                         ドッキング ステーションからコンピューターを削除        無効
SeManageVolumePrivilege                   ボリュームの保守タスクを実行                           無効
SeImpersonatePrivilege                    認証後にクライアントを偽装                             有効
SeCreateGlobalPrivilege                   グローバル オブジェクトの作成                          有効
SeIncreaseWorkingSetPrivilege             プロセス ワーキング セットの増加                       無効
SeTimeZonePrivilege                       タイム ゾーンの変更                                    無効
SeCreateSymbolicLinkPrivilege             シンボリック リンクの作成                              無効
SeDelegateSessionUserImpersonatePrivilege 同じセッションで別のユーザーの偽装トークンを取得します 無効
#>
```

:::message

上記で「無効」となっているのは、**権限はトークンに入っているが、現在の PowerShell プロセスでは有効化されていない**といった意味になります。

管理者権限を使っていない場合は、下記のようになります。一部が割り当てされてない状態となり、「有効/無効」でなくそもそも表示されておりません。

```txt
特権名                        説明                                            状態
============================= =============================================== ====
SeShutdownPrivilege           システムのシャットダウン                        無効
SeChangeNotifyPrivilege       走査チェックのバイパス                          有効
SeUndockPrivilege             ドッキング ステーションからコンピューターを削除 無効
SeIncreaseWorkingSetPrivilege プロセス ワーキング セットの増加                無効
SeTimeZonePrivilege           タイム ゾーンの変更                             無効
```

:::

### `SeSecurityPrivilege` とは

この特権を持つと、Securityに関するログを操作することができます。

- Windows の Security イベントログ を閲覧する
- Security イベントログを削除する
- オブジェクトの SACL (System Access Control List) を変更する
  - 「誰がアクセスしたら監査ログを残すか」を設定できる

Security ログには、下記のような重要な記録が含まれています。

- ログオン成功／失敗
- 管理者権限の使用
- ファイルアクセス監査
- 権限変更

そのため改ざんを防ぐために、特別な権限を持っていないと触れないよう保護がかけられています。

## 自身の `NTAccount` を取得する

現在操作中の Windows アカウントを、PowerShell から `NTAccount` として取得する方法を紹介します。

`NTAccount` は、一般的に `DOMAIN\UserName` や `COMPUTERNAME\UserName` のような、Windows で読みやすいアカウント名を表す型です。アクセス権、イベントログ、SID 変換などを扱うときに、この形式が必要になることがあります。

本項では、.NET の `[System.Security.Principal.WindowsIdentity]` を使って、現在のユーザー情報を取得します。

https://learn.microsoft.com/ja-jp/dotnet/api/system.security.principal.windowsidentity

### `GetCurrent()` で自身の `WindowsIdentity` オブジェクトを取得する

https://learn.microsoft.com/ja-jp/dotnet/api/system.security.principal.windowsidentity.getcurrent

`WindowsIdentity.GetCurrent()` は、現在の Windows ユーザーを表す `WindowsIdentity` オブジェクトを返します。

```ps1
[System.Security.Principal.WindowsIdentity]::GetCurrent()
<#
AuthenticationType : CloudAP
ImpersonationLevel : None
IsAuthenticated    : True
IsGuest            : False
IsSystem           : False
IsAnonymous        : False
Name               : FUGA\hoge
Owner              : S-0-0-00-0000000000-0000000000-00000000-0000
Owner              : S-0-0-00-0000000000-0000000000-00000000-0000
Groups             : {S-1-1-0, S-1-5-32-545, S-1-5-4, S-1-2-1...}
Token              : 2208
AccessToken        : Microsoft.Win32.SafeHandles.SafeAccessTokenHandle
UserClaims         : {http://schemas.xmlsoap.org/ws/2005/05/identity/claims/name: ...}
DeviceClaims       : {}
Claims             : {http://schemas.xmlsoap.org/ws/2005/05/identity/claims/name: ...}
Actor              :
BootstrapContext   :
Label              :
NameClaimType      : http://schemas.xmlsoap.org/ws/2005/05/identity/claims/name
RoleClaimType      : http://schemas.microsoft.com/ws/2008/06/identity/claims/groupsid
#>
```

### 方法 1: `Name` プロパティから `NTAccount` を生成する

`Name` プロパティには、現在のユーザー名が `DOMAIN\UserName` 形式で入っています。この値をそのまま `NTAccount` のコンストラクターに渡すと、`NTAccount` オブジェクトを作成できます。

```ps1
$CurrentIdentity = [System.Security.Principal.WindowsIdentity]::GetCurrent()

$MyName = $CurrentIdentity.Name # -> FUGA\hoge

# 文字列のアカウント名から NTAccount オブジェクトを作成する
$MyNTAccount = New-Object System.Security.Principal.NTAccount($MyName)
$MyNTAccount.Value  # -> FUGA\hoge
```

:::message

この手法のポイントは、下記のとおりです。

- **すでに `DOMAIN\UserName` 形式の名前が取れている場合**は、もっともシンプルです。
- 「現在のユーザー名を `NTAccount` として扱いたい」だけなら、この方法で十分です。
- `$CurrentIdentity.Name` は文字列ですが、`$MyNTAccount` は `NTAccount` 型のオブジェクトになります。

型も確認したい場合は、次のようにします。

```ps1
$MyNTAccount.GetType().FullName # ->  System.Security.Principal.NTAccount
```

:::

### 方法 2: `User` プロパティの SID から `NTAccount` に変換する

`User` プロパティには、現在のユーザーを表す SID が入っています。SID は `S-1-5-...` のような形式で、Windows 内部でアカウントを識別するための値です。

`SecurityIdentifier` から `Translate()` メソッドを使うと、SID を `NTAccount` に変換できます。

```ps1
$CurrentIdentity = [System.Security.Principal.WindowsIdentity]::GetCurrent()

$MySid = $CurrentIdentity.User
$MySid
<#
BinaryLength AccountDomainSid                        Value
------------ ----------------                        -----
          28 S-0-0-00-0000000000-0000000000-00000000 S-0-0-00-0000000000-0000000000-00000000-0000
#>

# SID から SecurityIdentifier オブジェクトを作成する
$MySecurityIdentifier = New-Object System.Security.Principal.SecurityIdentifier($MySid)

# SecurityIdentifier を NTAccount に変換する
$MyNTAccount = $MySecurityIdentifier.Translate([System.Security.Principal.NTAccount])
$MyNTAccount.Value  # -> FUGA\hoge
```

:::message

この手法のポイントは、下記のとおりです。

- SID を起点にして、読みやすいアカウント名へ変換する方法です。
- イベントログやアクセス制御リストなど、SID だけが手元にある場面でも応用できます。
- `Translate([System.Security.Principal.NTAccount])` により、`SecurityIdentifier` から `NTAccount` へ変換しています。

:::
