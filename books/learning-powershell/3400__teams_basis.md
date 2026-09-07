---
title: "🐓 Teams：基礎"
---

https://learn.microsoft.com/ja-jp/microsoftteams/teams-powershell-overview

PowerShell では、Microsoft 365 の各種サービスをコマンドレットから操作できます。Microsoft Teams についても、`MicrosoftTeams` モジュールを使用することで、チームの作成、チーム情報の取得、ユーザーの追加、ユーザーの削除、Owner 権限の確認などを自動化できます。

本項では、Teams の「チーム」の裏側にある仕組みを確認した後、基本的なコマンドレットを順に説明します。

## Teams における「チーム」の仕組み

https://learn.microsoft.com/ja-jp/MicrosoftTeams/office-365-groups

Teams でチームを作成すると、メンバーはチャネルでの会話だけでなく、ファイル共有や共同作業などの機能も利用できます。これは、Teams のチームが **Microsoft 365 グループを基盤としている**ためです。

チームを作成した際、その裏側では **対応する Microsoft 365 グループ**も作成されます。

Microsoft 365 グループは、「どのユーザーが共同作業に参加するか」というメンバーシップの情報を管理します。Teams の Owner や Member の管理も、このグループと関係しています。

また、Teams で扱う情報のすべてが *Teams 自体に* 保存されているわけではありません。例えば、標準チャネルで共有したファイルは、チームに関連付けられた SharePoint サイトに保存されます。

:::message alert

このため、Teams のチームを削除するときは、**Teams の画面だけでなく、関連する Microsoft 365 グループや各サービスにも影響が及ぶこと**を理解しておく必要があります。

:::

### Microsoft Teams を PowerShell を使って操作する

Teams は様々なアプリケーションやサービスに波及しており、それは前述したように Microsoft 365 グループに関係しております。

そのため Teams でのチーム作成などは通常、特殊な権限を持った者のみが行えるようになっています。例えば管理者は、Microsoft 365 管理センター上にある [Microsoft Teams 管理センター](https://admin.teams.microsoft.com/) で GUI を使い、チームを作成したりメンバーの役割を切り替えたりできます。

そしてこうした作業は、GUI に限らず PowerShell を使ったコマンドライン上でも行うことができます。

Teams を操作するには、Microsoft 365 設定全般を管理できる [Microsoft Graph PowerShell](https://learn.microsoft.com/ja-jp/microsoft-365/enterprise/connect-to-microsoft-365-powershell) もありますが、ここでは [Teams 専用のモジュール](https://learn.microsoft.com/ja-jp/microsoftteams/teams-powershell-install)である `MicrosoftTeams` モジュールを使った方法について説明します。

:::message

本書では取り扱いませんが、「Teams モジュールに存在しない細かな機能を扱いたい」や「Teams に限定せず Microsoft 365 の各機能を横断する必要がある」などの場合は **Microsoft Graph PowerShell**（Microsoft Graph API の PowerShell ラッパー）を使用した方が便利です。

ただし、こちらを使用する場合は、できることに反比例して「コマンドが複雑になりやすい」「API 権限など、Microsoft 365 全般に関わる理解が必要となる（≒ 学習コストが重い）」といった問題点が出てきます。

:::

## MicrosoftTeams モジュール

`MicrosoftTeams` は、Microsoft が Teams の管理用に提供している PowerShell モジュールです。

本項では **Teams 操作の基礎**として、次のコマンドレットを扱います。

| コマンドレット | 用途 |
| --- | --- |
| `Connect-MicrosoftTeams` | Teams の管理操作を行うために**接続する** |
| `Disconnect-MicrosoftTeams` | Teams との**接続を終了する** |
| `Get-Team` | チームを**取得する** |
| `Get-TeamUser` | チームに所属する**ユーザーを取得する** |
| `New-Team` | チームを**作成する** |
| `Set-Team` | チームの名前や設定を**変更する** |
| `Add-TeamUser` | チームに**ユーザーを追加する** |
| `Remove-TeamUser` | ユーザーまたは Owner ロールを**削除する** |
| `Remove-Team` | チームを**削除する** |

:::message

チームは Microsoft 365 グループを基盤として作成されます。そのため、Teams のチームを作成すると、裏側では対応する Microsoft 365 グループも作成されます。これは PowerShell での操作であっても同様です。

Teams PowerShell の各コマンドレットで頻出する `GroupId` は、このチーム、つまり**裏側にある Microsot 365 グループを識別するための ID** を意味します。

:::

### モジュールがインストールされているか確認する

`Get-Module` に `-ListAvailable` を指定すると、（現在のセッションに読み込まれているかにかかわらず）利用可能なモジュールを確認できます。

```ps1
Get-Module -ListAvailable -Name MicrosoftTeams |
  Sort-Object Version -Descending |
  Select-Object -First 1 Name, Version, Path
```

何も表示されない場合は、MicrosoftTeams モジュールが見つからないことを意味します。

### モジュールをインストールする

https://learn.microsoft.com/ja-jp/microsoftteams/teams-powershell-install

`MicrosoftTeams` モジュールがない場合には、PowerShell Gallery 経由でインストールします。

:::message

PowerShell Gallery を初めて使用する環境では、信頼されていないリポジトリに関する確認が表示される場合があります。組織の運用ルールを確認した上で続行してください。

:::

```ps1
Install-Module -Name MicrosoftTeams -Force -AllowClobber
```

インストールできたかについては、[モジュールの確認](#モジュールがインストールされているか確認する)に記載のコードで確認することができます。

### モジュールを読み込む

モジュールがインストールされていることを確認した後、次のように読み込みます。

```ps1
Import-Module -Name MicrosoftTeams -ErrorAction Stop
```

`Import-Module` を実行すると、現在の PowerShell セッションで MicrosoftTeams モジュールのコマンドレット（例：`Connect-MicrosoftTeams`, `New-Team`）を利用できるようになります。

## Teams への接続と切断

モジュールに含まれる Teams 関連のコマンドレットを実行するためには、`Connect-MicrosoftTeams` から Azure 資格情報を使用してサインインする必要があります。

### `Connect-MicrosoftTeams`

https://learn.microsoft.com/ja-jp/powershell/module/microsoftteams/connect-microsoftteams

:::message

実行すると、サインイン画面が表示される場合があります。

ここで接続したアカウント（に付与された権限）によって、実行できる処理の範囲は異なってきます。

:::

```ps1
# Teams へ接続
Connect-MicrosoftTeams
```

### `Disconnect-MicrosoftTeams`

https://learn.microsoft.com/ja-jp/powershell/module/microsoftteams/disconnect-microsoftteams

作業が終了したら、`Disconnect-MicrosoftTeams` で切断します。

:::message

接続したまま PowerShell を閉じるのではなく、不要になった時点で明示的に切断するようにしましょう。

無意味に接続を維持していたりキャッシュが残ったままというのは、望ましくありません。

:::

```ps1
# 確認プロンプトを表示せずに切断
Disconnect-MicrosoftTeams -Confirm $false
```

## `Get-Team`：チームを取得する

https://learn.microsoft.com/ja-jp/powershell/module/microsoftteams/get-team

何かしらの識別子に基づきチームを取得したい場合は、`Get-Team` を使用します。

### `Get-Team` の基本的な使い方

`Get-Team` は、既存のチームを取得するコマンドレットです。何も指定しないと、（テナント内にある）全てのチームを取得します。

```ps1
# 全てのチームを取得する
Get-Team
```

実務においては、全チームの取得が必要なケースというのは ほとんどないと思います。テナント内に多数のチームがある場合、全てを取得しようとすると時間がかかってしまいます。

通常は**何かしらの識別子**（例：`GroupID`, `User`, `DisplayName`）を使う、または**識別子に加え 状態**（例：`Archived`, `Visibility`）も組み合わせて絞り込みをかけるのが一般的です。

```ps1
# 例：特定のユーザーが所属しているチームをフィルタリング
Get-Team -User 'hoge@fuga.jp'

# 例：表示名からチームをフィルタリング
Get-Team -DisplayName '営業部'

# 例：表示名＋チームのアクセス許可レベルを指定してフィルタリング
Get-Team -DisplayName '営業部' -Visibility Private
```

### チーム名を完全一致で絞り込む

`Get-Team -DisplayName` は、**指定した文字列に（部分）一致するチーム**が返ってきます。そのため、例えば `営業部` と `営業部_連絡用` の 2 種がある場合、`Get-Team -DisplayName '営業部'` でチームを取得しようとすると、両方が含まれたものが返ってきます。

**完全一致するチームだけ**を取得するには、`Get-Team` で取得した結果に対し、更に `Where-Object` を使って ふるい分けする必要があります。

```ps1
# 例：'営業部', '営業部_連絡用' が存在し、'営業部' のみを取得したい

# 取得したいチーム名
$TeamDisplayName = '営業部'

# Get-Team の段階では、'営業部', '営業部_連絡用' の両方が取得される
$Team = Get-Team -DisplayName $TeamDisplayName |
  # ここから更に、名前の完全一致で ふるい分けを行う
  Where-Object {
    $_.DisplayName -eq $TeamDisplayName
  }
```

:::message

ここでの説明からわかる通り、`Get-Team` だけでは **チーム名が重複する**可能性があります。

結果が 1 件であると決めつけず、必要に応じて件数や `GroupId` も確認するのが良いでしょう。

```ps1
# 取得した情報を使って、表示して目視確認したり、機械的に判別して後の処理に活かす
$Team | Select-Object DisplayName, GroupId, Visibility, Description
```

:::

### チームのプロパティを確認する

取得したチームは、文字列ではなく複数のプロパティを持つオブジェクトです。プロパティ名を指定すると、必要な値を取り出せます。

```ps1
$Team.DisplayName # 表示名
$Team.Visibility  # チームのアクセス許可レベル
$Team.Description # 説明文
```

**後続の処理で特に重要なのが、チームを識別する値となる `GroupId` です**。ユーザーの取得、追加、削除や、チームの設定変更などでは、この値を使用して対象を指定します。

```ps1
# Team に関する諸処理のため、GroupID を取得する
$GroupId = [string]$Team.GroupId
$GroupId  # -> xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
```

## `Get-TeamUser`：チームに所属するユーザーを取得する

https://learn.microsoft.com/ja-jp/powershell/module/microsoftteams/get-teamuser

チームに所属するユーザーを取得するには、`Get-TeamUser` を使います。チームの指定には、チームを一意に識別できる `GroupId` が必要となります。

```ps1
# チームに所属するユーザーを取得
Get-TeamUser -GroupId $GroupId
```

ユーザー情報には下表のプロパティが含まれています。  
（複数ユーザーがいる場合、各ユーザーは**配列形式**で格納されています）

| プロパティ | 内容 |
| --- | --- |
| `User` | ユーザーの UPN（ *User Principal Name* ） |
| `UserId` | ユーザーを識別する ID |
| `Name` | ユーザーの表示名 |
| `Role` | チーム内での役割（`Owner`, `Member`, `Guest` など） |

```ps1
# 例：必要な項目だけを表示する
Get-TeamUser -GroupId $GroupId |
  Select-Object User, Name, Role

# 例：所属ユーザーの内、役割 Owner を持つ者のみ抽出
Get-TeamUser -GroupId $GroupId -Role Owner
```

### ユーザーが持つ役割について

チームに所属するユーザーが持つ役割は、主に `Owner` と `Member` の 2 種に分かれます。

- **Owner**: チームの設定変更やメンバー管理などを行う所有者
- **Member**: チームに参加し、許可された範囲で共同作業を行う一般メンバー

:::message

ユーザーがチームに所属していることと、そのユーザーが Owner であることは**別の状態**です。

役割を確認するときは、`Get-TeamUser` で取得したユーザー情報から `Role` を確認します。

:::

## `New-Team`：チームを作成する

https://learn.microsoft.com/ja-jp/powershell/module/microsoftteams/new-team

新しいチームを作成するためには、`New-Team` を使用します。

:::message

同名チームの作成を避けたい場合は、`New-Team` で作成する前に `Get-Team` と完全一致の絞り込みを使い、既存チームがないか確認しておくべきでしょう。

:::

```ps1
# 例：オーナーを特定のユーザーに割り当てた、非公開チームを作成する

# スプラッティングを使用してパラメータを整理
$paramNewTeam = @{
  DisplayName = '営業部_連絡用'   # チーム名
  Visibility  = Private         # 公開範囲：非公開
  Owner       = 'hoge@fuga.jp'  # UPN（≠ メールアドレス）
}
$CreatedTeam = New-Team @paramNewTeam

# 作成したチームの GroupID を取得
$GroupId = [string]$CreatedTeam.GroupId
```

`New-Team` の返り値には、作成されたチームの情報が含まれます。ここでは後続の処理に備えて、`GroupId` を取得しています。

:::message

なお、実際の `New-Team` コマンドレットでは様々なパラメータが設定可能です。詳細については、[Microsoft Learn のページ](https://learn.microsoft.com/ja-jp/powershell/module/microsoftteams/new-team)を参照ください。

:::

## `Set-Team`：チームの設定を変更する

https://learn.microsoft.com/ja-jp/powershell/module/microsoftteams/set-team

`Set-Team` は、既存チームの表示名、説明、公開範囲やチーム固有の設定などを変更できるコマンドレットです。対象は `GroupId` で指定します。

:::message alert

チーム名や公開範囲の変更は利用者に影響します。変更前に対象の `GroupId` と現在の設定を確認・記録しておくべきだと考えます。

:::

```ps1
# 例：チーム名を変更する
Set-Team `
  -GroupId $GroupId `
  -DisplayName '営業部_情報共有'

# 例：説明を変更する
Set-Team `
  -GroupId $GroupId `
  -Description '営業部内の情報共有に使用するチームです。'

# 例：複数の項目を変更する
Set-Team `
  -GroupId $GroupId `
  -DisplayName '営業部_情報共有' `
  -Description '営業部内の情報共有に使用するチームです。' `
  -Visibility Private `
  -AllowUserEditMessages $true `
  -AllowTeamMentions $true
```

:::message

なお、実際の `Set-Team` コマンドレットでは、`New-Team` 同様 様々なパラメータが設定可能です。詳細については、[Microsoft Learn のページ](https://learn.microsoft.com/ja-jp/powershell/module/microsoftteams/set-team)を参照ください。

:::

## `Add-TeamUser`：チームにユーザーを追加する

https://learn.microsoft.com/ja-jp/powershell/module/microsoftteams/add-teamuser

チームにユーザーを追加するには、`Add-TeamUser` を使用します。

```ps1
# 役割を明示することも可
Add-TeamUser `
  -GroupId $GroupId `
  -User 'hoge1-member@fuga.jp' `
  -Role Member

# -Role を省略した場合、ユーザーは Member として追加
Add-TeamUser `
  -GroupId $GroupId `
  -User 'hoge2-member@fuga.jp'

# Owner として追加する
Add-TeamUser `
  -GroupId $GroupId `
  -User 'hoge3-owner@fuga.jp' `
  -Role Owner
```

:::message

追加後は、下記のように ユーザーと役割を確認しておくのが確実です。

```ps1
Get-TeamUser -GroupId $GroupId |
  Select-Object User, Role
```

しかし Teams に関する諸処理において、コマンドを実行してから Teams クライアントなどの表示へ**変更が反映されるまで、時間がかかる場合がある**点には注意が必要です。

:::

## `Remove-TeamUser`：チームからユーザーまたは役割を削除する

https://learn.microsoft.com/ja-jp/powershell/module/microsoftteams/remove-teamuser

指定したユーザーをチームから削除する、あるいは Owner ロールだけを外す場合には、`Remove-TeamUser` を使用します。

:::message

両者は操作の趣旨が異なりますが、同じ `Remove-TeamUser` コマンドレットを使います。互いを混同しないよう注意してください。

:::

### ユーザーをチームから削除する

`-Role` を**指定せずに実行**すると、**対象ユーザーをチームから削除します**。

```ps1
# ユーザーをチームから削除
Remove-TeamUser `
  -GroupId $GroupId `
  -User 'hoge1-member@fuga.jp'
```

### Owner ロールだけを外す

`-Role Owner` を**指定すると、対象ユーザーから Owner ロールを外し、Member としてチームに残します**。

```ps1
# 役割 Owner を外す（→ Member としてチーム上には残る）
Remove-TeamUser `
  -GroupId $GroupId `
  -User 'hoge3-owner@fuga.jp' `
  -Role Owner
```

### `Remove-TeamUser` を使う際の注意事項

`Remove-TeamUser` を扱う際には、いくつか注意すべきことがあります。

#### チームには Owner が最低1人必要

まず初めに、**最後の Owner は削除できない**ことは覚えておいてください。チームには Owner ロールが最低 1 人必要となります。

誰かの Owner ロールを外す際には、ほかの Owner が存在することを確認したほうが良いでしょう。

#### コマンドレットには、2種の操作が可能なこと

繰り返しになりますが、`Remove-TeamUser` で できる操作は 2 種に分けることができます。ユーザー自体を削除する操作と、Owner ロールだけを外す操作では結果が異なってくるので注意が必要です。

実行前にはコマンドと対象ユーザーを見直しておくべきです。また操作後についても、下記のように「対象ユーザーが残っているか」、また「`Role` が意図した状態か」を確認しておくべきです。

```ps1
# 例：Remove-TeamUser で Owner ロールのみを削除

# 対象となる UPN
$ownerUPN = 'hoge3-owner@fuga.jp'

# Owner ロールを削除
Remove-TeamUser `
  -GroupId $GroupId `
  -User $ownerUPN `
  -Role Owner

# 状態を確認（→ Member として残っているか）
Get-TeamUser -GroupId $GroupId |
  Where-Object {
    $_.User -eq $ownerUPN
  } |
  Select-Object User, Role
```

## `Remove-Team`：チームを削除する

https://learn.microsoft.com/ja-jp/powershell/module/microsoftteams/remove-team

`Remove-Team` は、指定した `GroupId` のチームを削除します。

```ps1
# チームの削除
Remove-Team -GroupId $GroupId
```

### `Remove-Team` を使う際の注意事項

`Remove-Team` は**影響の大きい操作**です。指定したチームだけでなく、**その基盤となる Microsoft 365 グループと関連コンポーネントも削除されます**。

実行前に必ず、対象の `DisplayName` と `GroupId` を確認してください。

:::message

削除前の確認例を、下記に示します。

```ps1
$TargetTeam = Get-Team -GroupId $GroupId

$TargetTeam |
  Select-Object DisplayName, GroupId, Visibility, Description
```

:::

表示された情報が削除対象と一致していることを確認してから、`Remove-Team` を実行するようにしましょう。

## まとめ

本項では、次の内容を確認しました。

- Teams のチームは Microsoft 365 グループを基盤としています
- MicrosoftTeams モジュールを確認し、必要に応じてインストールやインポートを行います
- `Connect-MicrosoftTeams` と `Disconnect-MicrosoftTeams` で接続を管理します
- `Get-Team -DisplayName` の結果を完全一致で絞り込むことができます
- チームのプロパティから `GroupId` を取得でき、様々な処理で利用できます
- `Get-TeamUser` でユーザーと役割を確認できます
  - Owner と Member で、役割が異なります
- `New-Team` でチームを作成できます
- `Set-Team` でチーム名や説明を変更できます
- `Add-TeamUser` で Member または Owner を追加できます
- `Remove-TeamUser` で 2 種の操作を行えます
  - ユーザーをチームから削除
  - Owner ロールを削除
- `Remove-Team` でチームを削除できます
  - ただし、影響が大きいので事前の確認等は行うべきです

### 基本的な操作の一例

Teams の基本的な管理操作として、ここでは「チームを作成し、ユーザーの追加といった諸所の設定を行う」作業を取り上げてみます。

次のような流れで考えると、わかりやすいかと思います。

1. MicrosoftTeams モジュールを確認し、必要ならインストールする
2. モジュールを読み込む
3. `Connect-MicrosoftTeams` で接続する
4. `Get-Team` で対象チームを取得する
5. 取得したチームから `GroupId` を取り出す
6. `GroupId` を使って、設定変更やユーザー管理を行う（`Add-TeamUser`, `Set-Team` など）
7. `Get-Team` または `Get-TeamUser` で操作結果を確認する
8. `Disconnect-MicrosoftTeams` で切断する

特に重要なのは、**チーム名で対象を探し、確認後の操作では `GroupId` で対象を指定する**という考え方です。

#### チーム名でなく `GroupID` による管理

チーム名は人間には分かりやすいですが、一意性の保証という観点では `GroupId` の方が重要です。

作成後の操作では、必ず `GroupId` を使用して対象チームを指定するようにしましょう。

```ps1
Get-TeamUser -GroupId $GroupId
Add-TeamUser -GroupId $GroupId -User 'hoge3-owner@fuga.jp' -Role Owner
Remove-TeamUser -GroupId $GroupId -User 'hoge1-member@fuga.jp'
```

#### 実際の Teams 操作の際には、反映には時間差がある

`New-Team` や `Add-TeamUser` が成功しても、`Get-TeamUser` の結果や Teams クライアント側の表示に**すぐ反映されるとは限りません**。

:::message

そのため実務では「`Start-Sleep` と再試行を使って、反映されるまで待機する」といった処理を挟む場合があります。

:::
