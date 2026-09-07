---
title: "☕ Coffee Break : 1"
---

## ヘルプファイルの更新

PowerShell を学習していくにあたり、最初にヘルプ情報を更新しておきます。

`Update-Help` で、ヘルプファイルの更新が行なえます。

:::message

下記のように 極一部 エラーでアップデートできないことがあります。

```ps1
Update-Help
<#
Update-Help : UI カルチャ {en-US} を使用してモジュール 'Microsoft.PowerShell.Host' のヘルプを更新できませんでした: ルート要素が複数あります。 行 71、位置 26。
発生場所 行:1 文字:1
+ Update-Help
+ ~~~~~~~~~~~
    + CategoryInfo          : InvalidOperation: (:) [Update-Help]、Exception
    + FullyQualifiedErrorId : UnknownErrorId,Microsoft.PowerShell.Commands.UpdateHelpCommand
#>
```

:::

## コマンドプロンプトと PowerShell での動作の違い

PowerShell は これまでのコマンドプロンプトや VBS の後継として登場し、従来の Windows コマンドも実行することができます。

```ps1
Test-Connection google.com  # PowerShell
ping google.com  # CommandPrompt
```

しかし コマンドプロンプトで通っていた下記のような書き方は、PowerShell 上で行うことはできません。PowerShell で実行した場合、`ipconfig/all` で 1 語として認識されてしまいます。

```ps1
# 下記のような書き方は、PowerShell では不可
ipconfig/all
```

## 環境変数の確認

https://learn.microsoft.com/ja-jp/powershell/module/microsoft.powershell.core/about/about_environment_variables

仮想ドライブ `Env:` から、環境変数を引き出すことができます。

```ps1
Get-ChildItem Env: | Format-Table
<#
Name                           Value
----                           -----
ALLUSERSPROFILE                C:\ProgramData
APPDATA                        C:\Users\hoge\AppData\Roaming
CommonProgramFiles             C:\Program Files\Common Files
CommonProgramFiles(Arm)        C:\Program Files (Arm)\Common Files
CommonProgramFiles(x86)        C:\Program Files (x86)\Common Files
CommonProgramW6432             C:\Program Files\Common Files
COMPUTERNAME                   Hoge
ComSpec                        C:\Windows\system32\cmd.exe
DriverData                     C:\Windows\System32\Drivers\DriverData
HOMEDRIVE                      C:
HOMEPATH                       \Users\hoge
LOCALAPPDATA                   C:\Users\hoge\AppData\Local
LOGONSERVER                    \\HOGE
NUMBER_OF_PROCESSORS           8
OneDrive                       C:\Users\hoge\OneDrive
OneDriveConsumer               C:\Users\hoge\OneDrive
OS                             Windows_NT
Path                           C:\Windows\system32;C:\Windows;C:\Windows\System32\Wbem;C:\Windows\System32\WindowsPo...
PATHEXT                        .COM;.EXE;.BAT;.CMD;.VBS;.VBE;.JS;.JSE;.WSF;.WSH;.MSC;.CPL
PROCESSOR_ARCHITECTURE         ARM64
PROCESSOR_IDENTIFIER           ARMv8 (64-bit) Family 8 Model 1 Revision 201, Qualcomm Technologies Inc
PROCESSOR_LEVEL                1
PROCESSOR_REVISION             0201
ProgramData                    C:\ProgramData
ProgramFiles                   C:\Program Files
ProgramFiles(Arm)              C:\Program Files (Arm)
ProgramFiles(x86)              C:\Program Files (x86)
ProgramW6432                   C:\Program Files
PSModulePath                   C:\Users\hoge\OneDrive\ドキュメント\WindowsPowerShell\Modules;C:\Program Files\Windo...
PUBLIC                         C:\Users\Public
SystemDrive                    C:
SystemRoot                     C:\Windows
TEMP                           C:\Users\hoge\AppData\Local\Temp
TMP                            C:\Users\hoge\AppData\Local\Temp
USERDOMAIN                     HOGE
USERDOMAIN_ROAMINGPROFILE      HOGE
USERNAME                       hoge
USERPROFILE                    C:\Users\hoge
windir                         C:\Windows
#>
```

## スクリプト実行ポリシー

https://learn.microsoft.com/ja-jp/powershell/module/microsoft.powershell.core/about/about_execution_policies

PowerShell の実行ポリシーは、スクリプトを実行する条件を制御し、意図しないスクリプトの実行を防ぐための安全機能です。

:::message

ただし昨今、「悪意ある PowerShell コマンドをクリップボード経由で実行させようとする攻撃」（*ClickFix*）といった事例もあります。

実行ポリシーはユーザー操作などによって容易に回避ができてしまうため、完全なセキュリティ対策というわけではないことに注意が必要です。

:::

（Windows クライアントの場合）デフォルトでの実行ポリシーは `Restricted` になっていると思われますが、この状況では下記のようになります。

- コマンドは許可されますが、スクリプトファイル（拡張子 `.ps1` 含む様々なスクリプト）は実行不可です
- ISE 上でスクリプトファイルを読み込み実行ボタン（▶️）を押しても、ポリシー違反としてエラーになります

```ps1
E:\PowerShell\sample-01.ps1
<#
このシステムではスクリプトの実行が無効になっているため、ファイル E:\PowerShell\sample-01.ps1 を読み込むことができません。詳細については、「about_Execution_Policies」(https://go.microsoft.com/fwlink/?LinkID=135170) を参照してください。
    + CategoryInfo          : セキュリティ エラー: (: ) []、ParentContainsErrorRecordException
    + FullyQualifiedErrorId : UnauthorizedAccess
#>
```

### 実行ポリシーの確認

https://learn.microsoft.com/ja-jp/powershell/module/microsoft.powershell.security/get-executionpolicy

現在の実行ポリシーを確認する場合は、`Get-ExecutionPolicy` コマンドレットを使います。

```ps1
# 現 PowerShell セッションでの、有効な実行ポリシー
Get-ExecutionPolicy # -> Restricted

# 現セッションに影響する 全ての実行ポリシー
Get-ExecutionPolicy -List
<#
        Scope ExecutionPolicy
        ----- ---------------
MachinePolicy       Undefined
   UserPolicy       Undefined
      Process       Undefined
  CurrentUser       Undefined
 LocalMachine      Restricted
#>
```

### 実行ポリシーの変更

https://learn.microsoft.com/ja-jp/powershell/module/microsoft.powershell.security/set-executionpolicy

実行ポリシーを変更する場合は、`Set-ExecutionPolicy` コマンドレットを使います。

```ps1
# ある程度許容する RemoteSigned に変更
Set-ExecutionPolicy RemoteSigned

# 変更内容を確認
Get-ExecutionPolicy -List
<#
        Scope ExecutionPolicy
        ----- ---------------
MachinePolicy       Undefined
   UserPolicy       Undefined
      Process       Undefined
  CurrentUser       Undefined
 LocalMachine    RemoteSigned
#>
```

:::message

スコープを付けることで、影響範囲を明示することもできます。

```ps1
Set-ExecutionPolicy RemoteSigned -Scope CurrentUser
```

:::

:::message alert

変更時に警告が出ると思いますが、**この変更は危険性のあるものとなります**。

まず行うべきは、**元の設定値を `Get-ExecutionPolicy -List` で記録しておく**ことです。

その上で影響範囲を確認し、**必要なくなったら元に戻す** などの処理をすることを推奨します。

:::

## 文字化けについて

PowerShell を使う中で、文字化けに悩まされることがあるかもしれません。文字化けの原因は複数ありますが、私が出くわしたものは下記の 2 つです。

- 文字エンコードの違いによるもの
- PowerShell コンソール上のリッチテキストによるもの

ここではこの 2 つについて、簡単に説明します。

### 文字エンコードの違いによるもの

https://learn.microsoft.com/ja-jp/powershell/module/microsoft.powershell.core/about/about_character_encoding

![文字化け](/images/books/learning-powershell/encode-01.png)

「PowerShell でエクスポートした CSV ファイルを開いてみる」といった際に、文字化けを起こしている場合があります。こうした問題は、文字コードの違いによって生じています。

![コピペした内容を表示すると、文字化けを起こしている](/images/books/learning-powershell/encode-02.png)

VSCode や メモ帳アプリ、Webブラウザ全般、一般的な CSV ファイルというのは、大体は `UTF8`（ *CP65001* ）です。一方で Excel などの Office 製品や PowerShell は*環境にもよりますが* ANSI（その地域の標準的な文字コード規格）となっている場合があり、その関係で日本語環境では Shift-JIS（ *CP932* ）となっていることがあります。

#### `chcp` による対応

**コンソール上で文字化けを起こしている場合**の対応の 1 例として、`chcp`（*Changes (the active console) code page*）を使う方法があります。`chcp` を使うことで、コンソールのアクティプコードページを編集することができます。

```ps1
# 現在のエンコードを確認
chcp    # -> 現在のコード ページ: 932
```

https://learn.microsoft.com/ja-jp/windows-server/administration/windows-commands/chcp

コード ページ識別子 で関係あるのを絞り込むと、下表になります。

| 識別子 | `.NET` 名 | 追加情報 |
| :---: | :--- | :--- |
| `932` | `shift_jis` | ANSI/OEM 日本語;日本語 (Shift-JIS) |
| `20932` | `EUC-JP` | 日本語 (JIS 0208-1990 および 0212-1990) |
| `65001` | `utf-8` | Unicode (UTF-8) |

:::message

他の識別子等は、公式の[Microsoft Learn](https://learn.microsoft.com/ja-jp/windows/win32/intl/code-page-identifiers)にて確認できます。

:::

`chcp` にコードページ識別子をつけることで、文字コードを変えることができます。

```ps1
# UTF-8 に変更
chcp 65001
```

#### ユーザー設定変数での対応

https://learn.microsoft.com/ja-jp/dotnet/api/system.console.outputencoding

https://learn.microsoft.com/ja-jp/powershell/module/microsoft.powershell.core/about/about_parameters_default_values

他にもユーザー設定変数などから、文字コードを確認・変更することができます。

- 基本設定変数 : `$PSDefaultParameterValues`
- 出力書き込みのエンコード : `$OutputEncoding`

```ps1
# コンソールの入出力を UTF-8 に設定
[Console]::OutputEncoding = [System.Text.Encoding]::UTF8
[Console]::InputEncoding = [System.Text.Encoding]::UTF8

# PowerShell から外部のプログラムに文字列を渡す時のエンコーディングを UTF-8 に設定
$OutputEncoding = [System.Text.Encoding]::UTF8

# Out-File コマンドレットを使った際のファイル出力時の既定値を UTF-8 に設定
$PSDefaultParameterValues['Out-File:Encoding'] = 'utf8'
```

:::message

（PowerShell に慣れ、影響範囲を考慮した上ではありますが）これらの設定を都度コンソール上で叩くより、プロファイル（例：`profile.ps1`）に記述するというテクニックもあります。  
（プロファイルは、`bash` で言う `.bashrc` や `.bash_profile` に相当するものです）

また、個別にエンコード設定するのでなく、ワイルドカードを使った一律の設定も可能です。ただし、既存のスクリプトや他システムに影響を及ぼす可能性があることには注意が必要です。

```ps1
# Encoding パラメーターを持つすべてのコマンドレットの既定のエンコードを変更
$PSDefaultParameterValues['*:Encoding'] = 'utf8'
```

:::

#### コマンドレットのパラメータでの対応

https://learn.microsoft.com/ja-jp/powershell/module/microsoft.powershell.core/about/about_character_encoding

テキストファイルや CSV ファイルを入出力できるコマンドレットの場合、大抵は `-Encoding` といったパラメータを指定可能です。

https://learn.microsoft.com/ja-jp/powershell/module/microsoft.powershell.utility/import-csv

https://learn.microsoft.com/ja-jp/powershell/module/microsoft.powershell.utility/export-csv

https://learn.microsoft.com/ja-jp/powershell/module/microsoft.powershell.utility/out-file

### コンソール上のリッチテキストによる（と考えられる）もの

:::message alert

これは **特定環境でコピー時に文字列が崩れた事例** であり、**原因を裏付けることはできておりません**。

公開すべきかも考えましたが、自身の対応を記録しておく意味を込め ここに載せております。

:::

こちらの方は、文字コードの場合と違って起こる状況は限定されますが その分 厄介です。

起きうる場面としては**PowerShell コンソール上にある文字列をコピーして、テキストアプリなどに貼り付けた際**です。

:::message

考えられる原因としては下記があり、また組み合わさった結果かもしれません。いずれにせよ、原因は断定できていないものにはなります。

- リモート接続ソフト
- クリップボード転送
- ターミナル
- IME など

:::

どの様になるかと言うと、例えば コンソール上で `予定表` の文字列をコピーしてみます。これをテキストアプリに貼り付けてみると、`予\uinput2定e表\per` のような文字列がペーストされます。

これは一見 文字化けのように見えますが、実態は違うと考えられます。

これはエンコード云々といった問題ではなく、PowerShell コンソールのリッチテキストが上手くクリップボードに格納できていないことが原因ではないかというのが私の考えです。

PowerShell のコンソールは、文字色や文字背景色など フォント情報を弄ることが可能です。つまり PowerShell のコンソールは、単なる文字列ではなく リッチテキスト情報を持っているのです。

本来はクリップボードを経由する流れの中で、リッチテキストであることの解釈が行われるはずなのですが、それが上手くいかずプレーンテキストとして扱われてしまうと、`予\uinput2定e表\per` のような形で出力されてしまうのでは、と考えています。

#### 外部出力することでの対応

残念ながらこのケースは、コンソール上の設定を弄って解決することができませんでした。

ただ、外部に出力することでこの問題は回避することができました。回避できた理由としては、外部出力はリッチテキスト関係なく、オブジェクトのテキスト情報を（コンソール用に変換することなく）出力するからだと考えています。

例えば出力したい内容を、下記のコマンドレットに渡すことで正確なテキスト情報を抜き出すことができます。

- `Set-Clipboard` でクリップボードに直接渡す
- `Out-File` やリダイレクト演算子を使って、外部ファイルに出力する
- `Start-Transcript` でコンソール上に表示される内容を外部ファイルに出力する

ここでは最後の `Start-Transcript` について、別途 説明を加えます。この方法は実務や練習時のログ取りなど、様々な場面で役立つものだからです。

## `Start-Transcript` を使ったログ取り

https://learn.microsoft.com/ja-jp/powershell/module/microsoft.powershell.host/start-transcript

`Start-Transcript` コマンドレットは、PowerShellセッションの入出力をテキストファイルに記録するためのコマンドレットです。入力したコマンドとコンソールに表示される結果をリアルタイムで保存できるため、作業ログやスクリプト実行時のエラー解析に役立ちます。

```ps1
# そのまま実行すると、`$HOME\Documents` に重複しないファイル名で出力する
Start-Transcript

# パスを指定することも可能
Start-Transcript -Path "$HOME\Desktop\log.txt"
```

出力を止めたい時は、`Stop-Transcript` で止めることができます。

:::message alert

基本的にはセッション上の入出力はログとして記録されますが、その際**記録すべきでない情報が含まれていないか**は注意が必要です。

扱うコードによっては下記のような内容がログとして残る場合があります。

- 機密情報（例：ユーザーの ID や パスワード）
- 個人情報（人物が特定できる情報、例えば氏名や住所、電話番号など）

PowerShell には `SecureString` など見えなくする仕組みはありますが十分に注意し、パスワードやトークンなどをコマンドラインへ直接記述しないようすべきです。

:::

### 参考：プロファイルを使った記録方法について

「セッションの開始から終了までをログに取り、どこか指定の場所に一意の名前で記録しておきたい」といった場合に、参考になるかもしれません。

:::message alert

ここでの話は、私自身で まだ検証できておりません、ご了承ください。

:::

セッションの開始、及び終了には、プロファイルに設定を施すという方法があります。プロファイルとは、Bash でいう `.bashrc` や `bash_profile` に相当するようなものです。

https://learn.microsoft.com/ja-jp/powershell/module/microsoft.powershell.core/about/about_profiles

このプロファイルに、セッション開始及びセッション終了時の挙動を書いておくことで、自動的にログが取れるようにできるかもしれません。

セッション終了時は Bash で言う `.bash_logout` に相当するものがないので、プロファイルに終了時の挙動をイベント登録することで対応することになります。

https://learn.microsoft.com/ja-jp/powershell/module/microsoft.powershell.utility/register-engineevent#2-exiting

一意の形でファイル名を作る際には、下記のサンプルコードが役立ちそうです。

https://learn.microsoft.com/ja-jp/powershell/module/microsoft.powershell.host/start-transcript#3
