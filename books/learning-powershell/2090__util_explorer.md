---
title: "🐥 指定フォルダをエクスプローラーで表示"
---

## 本処理の目的

「（特定の）フォルダを画面上に表示したい」といった際、`Start-Process` でエクスプローラーを起動することで実装ができます。

利用ケースについては、下記のような *ユーザーに直接やってほしい* という状況を想定しています。

- 処理が完了したフォルダを開いて、ユーザーに**目視で**確認してもらう
- 特権関係など PowerShell 上では難しい処理が含まれている場合、対象の親フォルダを開いて**手作業で**やってもらう

### `Start-Process`

https://learn.microsoft.com/ja-jp/powershell/module/microsoft.powershell.management/start-process

環境変数を反映しているので、`$Path` 上に存在する `notepad` や `explorer.exe` については絶対パスで指定することなく利用が可能です。

## サンプル：指定フォルダをエクスプローラーで表示

```ps1
$FolderPath = 'C:\temp'
Start-Process -FilePath 'explorer.exe' -ArgumentList $FolderPath
```

:::message

上記の `$ForlderPath` については、処理に応じて様々なパターンが考えられます。

開きたい場所が固定であればベタ打ちでも問題ないですし、親フォルダを開きたい場合は `$ParentFolderPath = Split-Path -Path $FolderPath -Parent` といった形で加工が必要となるでしょう。

:::
