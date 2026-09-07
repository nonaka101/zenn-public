---
title: "🐥 フォルダ選択ダイアログ"
---

## 本処理の目的

**対話形式で処理を進めるスクリプト**を作る場合、ユーザーにフォルダパスを入力して欲しいといった場面がでてきます。

そのたびに都度ユーザーに「エクスプローラーを開いて、場所を辿って、そこのパスをコピーして・・・」といった工程を踏ませるのは、あまり現実的ではないかもしれません。

:::message

また前項で説明したように、**ユーザーに そのパス入力の全般をさせることは、様々なバリエーションを考慮する必要がでてくる**ということでもあります。

:::

本項では そうした問題を回避することを目的に、**フォルダ選択ダイアログ**を使ったフォルダパス入力を実装します。

- *誰もが同じ方法で* 操作し、フォルダを指定できる
- 作業の際には、*ダイアログ上でツリー管理されたフォルダを辿って指定する* だけ。

## `FolderBrowserDialog` クラス

https://learn.microsoft.com/ja-jp/dotnet/api/system.windows.forms.folderbrowserdialog

PowerShell では `.NET` の機能を使い、GUI のフォームダイアログを呼び出すことができます。今回はその中の 1 つである、`FolderBrowserDialog` を扱います。

![FolderBrowserDialog](/images/books/learning-powershell/dialogs-01.png)

### `Add-Type` コマンドレットについて

https://learn.microsoft.com/ja-jp/powershell/module/microsoft.powershell.utility/add-type

`FolderBrowserDialog` を使用するにはまず、`Add-Type` コマンドレットで Microsoft .NET クラスを追加する必要があります。`Add-Type` は PowerShell セッション中に、Microsoft .NET Core クラスを使用するためのものです。

今回はアセンブリ名 `System.Windows.Forms` を指定し利用します。

```ps1
# AssemblyName からの呼び出し
Add-Type -AssemblyName System.Windows.Forms
```

これで、`System.Windows.Forms` 配下にある機能を呼び出すことができるようになりました。

https://learn.microsoft.com/ja-jp/dotnet/api/system.windows.forms

### プロパティについて

今回作成する `FolderBrowserDialog` は、実際に動かしてみると 下記のようなプロパティを持っていることがわかります。

```txt
ShowNew FolderButton  : False
SelectedPath          : C:\Users\hoge\Desktop\Utils
RootFolder            : Desktop
Description           : Select a Folder
Tag                   :
Site                  :
Container             :
```

ここでは、主要なプロパティを見ていきます。

#### `ShowNewFolderButton` プロパティ

https://learn.microsoft.com/ja-jp/dotnet/api/system.windows.forms.folderbrowserdialog.shownewfolderbutton

フォルダブラウザのダイアログボックスに、「新しいフォルダ」ボタンを表示するかを管理します。（真偽値）

#### `SelectedPath` プロパティ

https://learn.microsoft.com/ja-jp/dotnet/api/system.windows.forms.folderbrowserdialog.selectedpath

ダイアログで *最初に* 選択したフォルダのパスが、ここに格納されます。  
（既定値は空文字）

#### `RootFolder` プロパティ

https://learn.microsoft.com/ja-jp/dotnet/api/system.windows.forms.folderbrowserdialog.rootfolder

ダイアログが開いた際の、ルートとなるフォルダを設定できます。ここでは `Environment.SpecialFolder` 列挙型から指定します。

https://learn.microsoft.com/ja-jp/dotnet/api/system.environment.specialfolder

### `ShowDialog` メソッドについて

https://learn.microsoft.com/ja-jp/dotnet/api/system.windows.forms.commondialog.showdialog

フォームは `$Form.ShowDialog()` の形で呼び出すことができます。

ここで選択したフォルダは、`$Form.SelectedPath` プロパティに格納されることになります。

また このメソッドは返り値を持ちます。この返り値は、ダイアログ上で「OK」や「キャンセル」ボタンを押した状態を、`DialogResult` 列挙型として返したものになります。

https://learn.microsoft.com/ja-jp/dotnet/api/system.windows.forms.dialogresult

```ps1
# 例：フォームを格納した変数 `$Form` を表示させ、OK ボタンを押した場合
$result = $Form.ShowDialog()
$result # -> OK
$result -eq [System.Windows.Forms.DialogResult]::OK # -> True
```

実際のスクリプトでは この返り値を使って、**ユーザーがダイアログで押したボタンに応じて処理を分岐させること**ができます。

## サンプル：フォルダ選択ダイアログ

```ps1
function Select-TargetFolder {
  <#
  .SYNOPSIS
    フォルダ選択ダイアログを表示し、選択されたフォルダのパスを返します。

  .DESCRIPTION
    Windows Forms の FolderBrowserDialog を使用して、フォルダ選択ダイアログを
    表示します。ユーザーがフォルダを選択して［OK］を押した場合は、そのフォルダの
    パスを返します。選択をキャンセルした場合は $null を返します。

    ダイアログでは新しいフォルダの作成を無効にしています。また、処理の完了時には
    FolderBrowserDialog オブジェクトを確実に破棄します。

  .PARAMETER RootFolder
    フォルダ選択ダイアログで起点として使用する特殊フォルダを指定します。
    既定値は Desktop です。

  .OUTPUTS
    System.String
    ユーザーが［OK］を押した場合は、選択されたフォルダのパスを返します。
    ダイアログをキャンセルした場合は $null を返します。

  .EXAMPLE
    $SelectedFolder = Select-TargetFolder

    デスクトップを起点とするフォルダ選択ダイアログを表示し、選択された
    フォルダのパスを $SelectedFolder に格納します。
  #>
  [CmdletBinding()]
  param(
    [Parameter()]
    [System.Environment+SpecialFolder]
    $RootFolder = [System.Environment+SpecialFolder]::Desktop
  )

  # フォルダダイアログの生成
  Add-Type -AssemblyName System.Windows.Forms

  $FormProperty = @{
    RootFolder          = $RootFolder
    Description         = 'フォルダを選択してください'
    ShowNewFolderButton = $false
  }

  $FolderBrowser = New-Object System.Windows.Forms.FolderBrowserDialog -Property $FormProperty

  # 適正な入力（フォルダを選択したうえで OK で決定）を受け、フォルダパスを返す
  try {
    if ($FolderBrowser.ShowDialog() -eq [System.Windows.Forms.DialogResult]::OK) {
      return $FolderBrowser.SelectedPath
    }
    return $null
  }
  finally {
    # ダイアログは確実に破棄しておく
    $FolderBrowser.Dispose()
  }
}
```

:::message

最後の `finally` でリソースの解放を行っています。

`finally` ブロックは、[関数から実際に抜ける前に必ず実行される処理となります](https://learn.microsoft.com/ja-jp/powershell/module/microsoft.powershell.core/about/about_try_catch_finally#freeing-resources-using-finally)。

:::
