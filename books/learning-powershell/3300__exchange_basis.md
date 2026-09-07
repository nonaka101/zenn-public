---
title: "🐓 Exchange：基礎"
---

PowerShell では様々なパラメータを扱うことになりますが、その中でも特に ActiveDirectory 及び Microsoft Exchange に関するものが多いです。言い換えれば、その分 様々な項目を PowerShell 上で設定可能ということになります。

ここでは、Exchange Online PowerShell を使用し、メールボックスの設定を確認・変更する基本的な方法について説明します。

## Exchange Online PowerShell モジュールについて

https://learn.microsoft.com/ja-jp/powershell/exchange/exchange-online-powershell-v2

Exchange Online を PowerShell から操作するには、`ExchangeOnlineManagement` モジュールを使用します。

このモジュールを利用すると、多要素認証（MFA）を使用する環境でも Exchange Online に接続し、権限の範囲内でコマンドレットを実行することができます。

:::message

本項では、モジュールがインストール済みであり、メールボックスの参照や変更に必要な権限が割り当てられているものという想定で、話を進めていきます。

:::

## Exchange Online への接続と切断

https://learn.microsoft.com/ja-jp/powershell/exchange/connect-to-exchange-online-powershell

Exchange Online のコマンドレットを実行するためには、`Connect-ExchangeOnline` で組織へ接続しておく必要があります。

### `Connect-ExchangeOnline`

https://learn.microsoft.com/ja-jp/powershell/module/exchangepowershell/connect-exchangeonline

:::message

実行すると、サインイン画面が表示される場合があります。

接続後に利用できるコマンドレットやパラメータは、接続したアカウントへ割り当てられた役割によって異なります。

:::

```ps1
# Exchange Online へ接続
Connect-ExchangeOnline
```

### `Disconnect-ExchangeOnline`

https://learn.microsoft.com/ja-jp/powershell/module/exchangepowershell/disconnect-exchangeonline

作業が終了したら、`Disconnect-ExchangeOnline` で切断します。

:::message

接続したまま PowerShell を閉じるのではなく、不要になった時点で明示的に切断するようにしましょう。

無意味に接続を維持していたりキャッシュが残ったままというのは、望ましくありません。

:::

```ps1
# 確認プロンプトを表示せずに切断
Disconnect-ExchangeOnline -Confirm $false
```

## Identity で操作対象を指定する

Exchange の多くのコマンドレットでは、`Identity` パラメータを使って**操作対象を指定**する必要があります。

本項では メールボックスに関する操作を扱う都合上、対象は 明確でわかりやすい**メールアドレス**を使用します。

```ps1
# 例：メールボックス設定を取得する
Get-Mailbox -Identity 'hoge@fuga.jp'
```

:::message

`Identity` にはメールアドレス以外の識別子を指定できる場合もありますが、利用可能な値はコマンドレットによって異なってきます。

詳細は各コマンドレットの公式リファレンスを確認してください。

:::

## 基本的な流れ：確認・変更・再確認

本項での基本的な流れは、下記のようになっております。

1. `Get-*` コマンドレットで**現在の設定を取得する**
2. `Set-*` コマンドレットで**設定を変更する**
3. `Get-*` コマンドレットで**設定を再取得する**（確認用）

```ps1
# 例：メールボックスへの特定アクセス（POP3, IMAP）を無効化する

# 1. 変更前の値を確認
Get-CASMailbox -Identity 'hoge@fuga.jp' |
  Select-Object Identity, PopEnabled, ImapEnabled

# 2. 設定を変更
Set-CASMailbox -Identity 'hoge@fuga.jp' `
  -PopEnabled $false `
  -ImapEnabled $false

# 3. 変更後の値を再取得
Get-CASMailbox -Identity 'hoge@fuga.jp' |
  Select-Object Identity, PopEnabled, ImapEnabled
```

上記の流れというのは、おおよその場面で役立つ流れです。

ただ、Exchange Online の設定変更においては、この流れが *特に* 重要となってきます。それは、**設定する言語**（英語、日本語 など）によってフォルダパスなど様々な要素が異なってくるからです。

:::message

`Set-*` が *エラーを表示しなかったことだけで完了* とせず、対象から現在値を再取得し、意図した状態になっているかを確認すべきだと考えます。  
（再確認を作業者の目視で行うべきか、自動化に含めるかは状況によりけりではあります）

:::

## クライアントアクセス設定

メールボックスのクライアントアクセス設定を確認・変更するためには、`Get-CASMailbox` と `Set-CASMailbox` を使用します。

:::message

ここでの **CAS** は、「クライアントからメールボックスへ接続するための設定（ *Calient Access* の *Setting* ）」と捉えると分かりやすくなります。

元は *Client Access Server* の意味だったそうですが、主流の環境が オンプレミスからクラウドへと変わった昨今においてはサーバの意味が薄れてしまい、昔の名残として残っているのだそうです。

:::

### `Get-CASMailbox`：現在の設定を確認する

https://learn.microsoft.com/ja-jp/powershell/module/exchangepowershell/get-casmailbox

```ps1
# 各種接続の有効・無効状態を確認
Get-CASMailbox -Identity 'hoge@fuga.jp' |
  Select-Object `
    Identity,
    PopEnabled,
    ImapEnabled,
    ActiveSyncEnabled,
    UniversalOutlookEnabled |
  Format-List
```

各プロパティが `$true` なら有効、`$false` なら無効を意味します。

| プロパティ | 対象 |
| :--- | :--- |
| `PopEnabled` | POP3 による接続 |
| `ImapEnabled` | IMAP4 による接続 |
| `ActiveSyncEnabled` | Exchange ActiveSync による接続 |
| `UniversalOutlookEnabled` | Universal Outlook（※）による接続 |

:::message

*Universal Outlook* は、Windows 10 のアプリ「メール＆カレンダー」を指します。

:::

### `Set-CASMailbox`：設定を変更する

https://learn.microsoft.com/ja-jp/powershell/module/exchangepowershell/set-casmailbox

`Set-CASMailbox` は、メールボックス単位でクライアント接続に関する設定を変更するコマンドレットです。Outlook on the Web、MAPI、EWS、ActiveSync、POP、IMAP、SMTP AUTH などの有効化・無効化や、利用可能なクライアントの制御に使用します。

:::message

設定値が複数ある場合、（整理のため）以降はスプラッティングを使用していきます。

:::

```ps1
# 使用しない接続を無効に設定する
$paramCasMailbox = @{
  Identity                = 'hoge@fuga.jp'
  PopEnabled              = $false
  ImapEnabled             = $false
  ActiveSyncEnabled       = $false
  UniversalOutlookEnabled = $false
}
Set-CASMailbox @paramCasMailbox
```

#### `Get-CASMailbox`：変更結果を確認する

`Set-CASMailbox` で変更した設定内容を、`Get-CASMailbox` で再取得して確認します。

指定した4項目がすべて `False` であれば、適切に変更がされたことになります。

```ps1
Get-CASMailbox -Identity 'hoge@fuga.jp' |
  Select-Object `
    Identity,
    PopEnabled,
    ImapEnabled,
    ActiveSyncEnabled,
    UniversalOutlookEnabled |
  Format-List
```

## メールボックスの地域設定

メールボックスの言語、タイムゾーン、日付形式、時刻形式などを確認・変更するためには、`Get-MailboxRegionalConfiguration` と `Set-MailboxRegionalConfiguration` を使用します。

### `Get-MailboxRegionalConfiguration`：現在の設定を確認する

https://learn.microsoft.com/ja-jp/powershell/module/exchangepowershell/get-mailboxregionalconfiguration

```ps1
# メールボックスの地域設定を確認する
Get-MailboxRegionalConfiguration -Identity 'hoge@fuga.jp' |
  Select-Object Language, TimeZone, DateFormat, TimeFormat |
  Format-List
```

### `Set-MailboxRegionalConfiguration`：設定を変更する

https://learn.microsoft.com/ja-jp/powershell/module/exchangepowershell/set-mailboxregionalconfiguration

`Set-MailboxRegionalConfiguration` は、メールボックスの地域設定、つまり言語、日付形式、時刻形式、タイム ゾーン、既定フォルダ名のローカライズを設定するコマンドレットです。

```ps1
# 日本向けの設定に変更する
$paramRegional = @{
  Identity                  = 'hoge@fuga.jp'
  Language                  = 'ja-JP'
  TimeZone                  = 'Tokyo Standard Time'
  LocalizeDefaultFolderName = $true
}
Set-MailboxRegionalConfiguration @paramRegional
```

`LocalizeDefaultFolderName` を指定すると、デフォルトでの `Inbox`, `Sent Items`, `Drafts` といった既定フォルダ名が、指定した言語に合わせてローカライズされます。  
（例：`受信トレイ`, `送信済みアイテム`, `下書き`）

:::message

既定フォルダ名のローカライズは、後述する**予定表フォルダの指定**にも関係します。

日本語化されていれば `予定表`、英語のままであれば `Calendar` となる可能性があります。

:::

#### `Get-MailboxRegionalConfiguration`：変更結果を確認する

`Language` が `ja-JP`、`TimeZone` が `Tokyo Standard Time` であることを確認します。

```ps1
Get-MailboxRegionalConfiguration -Identity 'hoge@fuga.jp' |
  Select-Object Language, TimeZone |
  Format-List
```

## 予定表フォルダのアクセス権

メールボックス内のフォルダに設定されたアクセス許可を確認・変更するためには、
`Get-MailboxFolderPermission` と `Set-MailboxFolderPermission` を使用します。

:::message alert

ここで扱う 予定表フォルダ名や `Default` エントリの表示名は、メールボックスの言語環境によって異なる場合があります。

固定値だけを前提にせず、まず `Get-MailboxFolderPermission` で実際の表示を確認してください。

:::

### フォルダの指定形式

フォルダは、`メールボックス:\フォルダ名` の形式で指定します。

ここでは予定表フォルダを取り扱いますが、下記のようにフォルダパスは**言語によって異なります**。

```ps1
# 日本語化された予定表
$calendarIdentity = "hoge@fuga.jp:\予定表"

# 英語名（Default）の場合
$calendarIdentity = "hoge@fuga.jp:\Calendar"
```

:::message

サンプルコードでは、わかりやすいよう `hoge@fuga.jp:\予定表` をそのまま使用するようにしています。

:::

### `Get-MailboxFolderPermission`：現在のアクセス権を確認する

https://learn.microsoft.com/ja-jp/powershell/module/exchangepowershell/get-mailboxfolderpermission

```ps1
Get-MailboxFolderPermission -Identity 'hoge@fuga.jp:\予定表' |
  Format-Table User, AccessRights -AutoSize
```

予定表には、個別のユーザーやグループのほか、`Default` や `Anonymous` などのエントリが存在します。

- `Default`（`既定`）：個別の権限が設定されていない組織内ユーザーに適用されるエントリ
- `Anonymous`（`匿名`）：未認証のアクセスに対するエントリ（≒ 組織外ユーザー）

:::message

表示言語によって、`Default` が `既定`、`Anonymous` が `匿名` と表示される場合があります。

:::

### `Set-MailboxFolderPermission`：設定を変更する

https://learn.microsoft.com/ja-jp/powershell/module/exchangepowershell/set-mailboxfolderpermission

`Set-MailboxFolderPermission` は、**すでに存在するアクセス許可エントリを変更**します。  
（新しいエントリを追加する場合は `Add-MailboxFolderPermission`、削除する場合は `Remove-MailboxFolderPermission` を使用することになります）

```ps1
# Default の権限を Reviewer に変更する
$paramFolderPermission = @{
  Identity     = 'hoge@fuga.jp:\予定表'
  User         = 'Default'
  AccessRights = 'Reviewer'
}
Set-MailboxFolderPermission @paramFolderPermission
```

上記の `Reviewer` は、フォルダを表示する `FolderVisible` と、フォルダ内のアイテムを読み取る `ReadItems` を組み合わせたアクセス権ロールです。詳細は [MS Learn 上にある AccessRights パラメータ](https://learn.microsoft.com/ja-jp/powershell/module/exchangepowershell/set-mailboxfolderpermission#-accessrights)を参照してください。

予定表でよく使用する代表的なロールは下表となります。

| ロール | 概要 |
| --- | --- |
| `Reviewer` | アイテムを読み取る |
| `Editor` | アイテムの作成、読み取り、編集、削除を行う |
| `AvailabilityOnly` | 空き時間情報のみを表示する |
| `LimitedDetails` | 空き時間に加え、件名や場所などの限定情報を表示する |
| `None` | アクセス権を付与しない |

:::message alert

前述の `Set-MailboxRegionalConfiguration` でのローカライズが反映されているかどうかで、ここの処理が失敗するかもしれません。  
（エラー例：「`既定` というユーザーエントリはありません」）

下表は、*予定表フォルダ* と *既定を意味するユーザー名* を言語別に記したものです。

| 言語＼要素 | 予定表フォルダ | ユーザー |
| --- | --- | --- |
| 英語 | `hoge@fuga.jp:\Calendar` | `Default` |
| 日本語 | `hoge@fuga.jp:\予定表` | `既定` |

:::

#### `Get-MailboxFolderPermission`：変更結果を確認する

対象エントリの `AccessRights` が `{Reviewer}` になっていることを確認します。

```ps1
Get-MailboxFolderPermission -Identity 'hoge@fuga.jp:\予定表' |
  Where-Object { $_.User -in @('Default', '既定') } |
  Format-Table User, AccessRights -AutoSize
```

## メールボックス容量

メールボックスそのものの設定を確認・変更するには、`Get-Mailbox` と `Set-Mailbox` を使います。

設定項目は多数ありますが、ここではメールボックス容量に関する3項目だけを扱います。

| プロパティ | 意味 |
| --- | --- |
| `IssueWarningQuota` | 使用量に関する**警告を開始する**しきい値 |
| `ProhibitSendQuota` | 新しいメッセージの**送信を禁止する**しきい値 |
| `ProhibitSendReceiveQuota` | 新しいメッセージの**送受信を禁止する**しきい値 |

:::message

上表での各種しきい値は、上から順の関係となっています。

例えば `IssueWarningQuota` で指定する容量は、`ProhibitSendQuota` より小さい必要があります。

:::

### `Get-Mailbox`：現在の容量設定を確認する

https://learn.microsoft.com/ja-jp/powershell/module/exchangepowershell/get-mailbox

```ps1
Get-Mailbox -Identity 'hoge@fuga.jp' |
  Select-Object `
    IssueWarningQuota,
    ProhibitSendQuota,
    ProhibitSendReceiveQuota |
  Format-List
```

### `Set-Mailbox`：設定を変更する

https://learn.microsoft.com/ja-jp/powershell/module/exchangepowershell/set-mailbox

`Set-Mailbox` は、メールボックスそのものの属性やメール フロー、容量、保持、監査、アーカイブ、代理送信、アドレス帳表示、リソース メールボックス設定などを変更するコマンドレットです。

```ps1
# メールボックスの各種容量を変更する
$paramMailbox = @{
  Identity                 = 'hoge@fuga.jp'
  IssueWarningQuota        = '18GB'
  ProhibitSendQuota        = '19GB'
  ProhibitSendReceiveQuota = '20GB'
}
Set-Mailbox @paramMailbox
```

:::message

容量は、組織の運用方針や対象ユーザーのライセンスに合わせて決定します。上記の値は、コマンドの関係を示すための例です。

:::

#### `Get-Mailbox`：変更結果を確認する

3つの値が意図した順序と容量になっていることを確認します。

```ps1
Get-Mailbox -Identity 'hoge@fuga.jp' |
  Select-Object `
    IssueWarningQuota,
    ProhibitSendQuota,
    ProhibitSendReceiveQuota |
  Format-List
```

## 一連のコード

ここまでの設定と確認をまとめると、次のようになります。

```ps1
$targetAddress = 'hoge@fuga.jp'
$calendarIdentity = "${targetAddress}:\予定表"

try {
  # Exchange Online へ接続
  Connect-ExchangeOnline

  # 1. クライアントアクセス設定
  $paramCasMailbox = @{
    Identity                = $targetAddress
    PopEnabled              = $false
    ImapEnabled             = $false
    ActiveSyncEnabled       = $false
    UniversalOutlookEnabled = $false
  }
  Set-CASMailbox @paramCasMailbox

  # 2. 地域設定
  $paramRegional = @{
    Identity                  = $targetAddress
    Language                  = 'ja-JP'
    TimeZone                  = 'Tokyo Standard Time'
    LocalizeDefaultFolderName = $true
  }
  Set-MailboxRegionalConfiguration @paramRegional

  # 3. 予定表の既定アクセス権
  $paramFolderPermission = @{
    Identity     = $calendarIdentity
    User         = 'Default'
    AccessRights = 'Reviewer'
  }
  Set-MailboxFolderPermission @paramFolderPermission

  # 4. メールボックス容量
  $paramMailbox = @{
    Identity                 = $targetAddress
    IssueWarningQuota        = '18GB'
    ProhibitSendQuota        = '19GB'
    ProhibitSendReceiveQuota = '20GB'
  }
  Set-Mailbox @paramMailbox

  # 5. 設定結果を再取得
  Get-CASMailbox -Identity $targetAddress |
    Select-Object Identity, PopEnabled, ImapEnabled,
      ActiveSyncEnabled, UniversalOutlookEnabled |
    Format-List

  Get-MailboxRegionalConfiguration -Identity $targetAddress |
    Select-Object Language, TimeZone |
    Format-List

  Get-MailboxFolderPermission -Identity $calendarIdentity |
    Where-Object { $_.User -in @('Default', '既定') } |
    Format-Table User, AccessRights -AutoSize

  Get-Mailbox -Identity $targetAddress |
    Select-Object IssueWarningQuota, ProhibitSendQuota,
      ProhibitSendReceiveQuota |
    Format-List
}
finally {
  # 処理の成否にかかわらず切断
  Disconnect-ExchangeOnline -Confirm $false
}
```

実務用スクリプトでは、予定表のフォルダ名や既定ユーザー名の表示差、ライセンス種別、例外処理なども考慮する必要があります。

まずは、それぞれの `Get-*` と `Set-*` が何を確認・変更しているかを理解していくことから始めるべきです。

## まとめ

Exchange Online の基本的な設定操作では、次のコマンドレットを使用しました。  
（変更のコマンドレット `Set-*` は、`Get-*` 以降と同じものとなります）

| 対象 | 確認 | 変更 |
| --- | --- | --- |
| **クライアントアクセス** | `Get-CASMailbox` | `Set-*` |
| **地域設定** | `Get-MailboxRegionalConfiguration` | `Set-*` |
| **フォルダアクセス権** | `Get-MailboxFolderPermission` | `Set-*` |
| **メールボックス容量** | `Get-Mailbox` | `Set-*` |

最も重要なのは、次の流れを一組として扱うことです。

- `Get-*` で変更前の状態を確認する
- `Set-*` で設定を変更する
- `Get-*` で変更後の状態を再取得する

:::message

重要である理由は、Exchange の操作ではローカライズの設定が関わり、それによってフォルダパスなどが**言語間で異なってくるため**でした。

:::

また、作業の開始時には `Connect-ExchangeOnline` で接続し、終了時には `Disconnect-ExchangeOnline` で切断するようにしましょう。

## 実際の業務で使用するときの注意

### 対象を事前に確認する

`Identity` に指定したメールアドレスが、目的のメールボックスであることを確認します。

変数を使用する場合は、設定処理の前に値を表示する方法も有効です。

### 変更前後の値を記録する

必要に応じて変更の前後を記録しておくことで、障害調査や設定の見直しに役立てることができます。

### エラー時にも切断する

接続後の処理でエラーが発生した場合にも切断できるよう、実務用スクリプトでは `finally` を利用できます。

```ps1
try {
  Connect-ExchangeOnline
  # Exchange Online に対する処理
}
finally {
  Disconnect-ExchangeOnline -Confirm:$false
}
```
