---
title: "🐣 エラーハンドリング"
---

https://learn.microsoft.com/ja-jp/powershell/scripting/learn/deep-dives/everything-about-exceptions

## エラーハンドリングの重要性

スクリプトを実運用する際、「もし処理の途中でエラーが起きたらどうするか？」を定義しておくことは非常に重要です。

PowerShell には、終了エラーを処理するための **`try` - `catch` - `finally`** 構文が用意されています。これを用いることで **エラー発生時の処理**や、エラーの有無にかかわらずに実行する**後処理**を記述できます。

### `try` - `catch` 構文

終了エラーが発生する可能性のある処理を `try` ブロック内に書き、エラーが発生した場合の対応を `catch` ブロックに記述します。

:::message

原因がわかりづらくなることを防ぐため、`try` ブロック内のコードは最小限に抑えるべきです。

また 終了しない（軽度な）エラーを `catch` で処理したい場合には、`-ErrorAction Stop` などを使って終了エラーへ変換する必要があります。

:::

```ps1
try {
  # エラーが発生する可能性のある処理
  $result = 10 / 0
}
catch {
  # エラーが発生した際に実行される処理
  Write-Host "エラーが発生しました！"
}
# -> エラーが発生しました！
```

### `finally` ブロック

**エラーの有無に関わらず**、最後に必ず実行したい処理がある場合は `finally` ブロックを追加します。

例えば「開いたファイルを閉じる」「接続したセッションを切断する」などが挙げられます。

```ps1
try {
  Write-Host "処理を開始します..."
  $result = 10 / 0
}
catch {
  Write-Host "エラーを検知しました！"
}
finally {
  Write-Host "後片付けの処理を実行します..."
}
<#
処理を開始します...
エラーを検知しました！
後片付けの処理を実行します...
#>
```

## 注意点：2 種類のエラー

PowerShell のエラーには大きく分けて2種類あります。

1. **終了するエラー (Terminating Error):** 文法エラーや存在しないコマンドの実行など、*影響が大きく処理が即座に止まる* エラー。
2. **終了しないエラー (Non-Terminating Error):** ファイルが見つからない、アクセス権がないなど、コマンドが赤字でエラー文章を吐き出すも *次の行の処理へ進んでしまう* エラー。

`try` - `catch` でキャッチできるのは**原則として「終了するエラー」のみ** です。

:::message

終了するエラーに関しては、実際には更に「ステートメント終了エラー」と「スクリプト終了エラー」に分割することもできます。

両方とも `try` - `catch` で処理できるのは同じです。

:::

### `$ErrorActionPreference` の設定

実務のスクリプトでは、「終了しないエラー」であっても、そこで処理を止めて `catch` ブロックに拾わせたいケースが多々あります。

その場合は、エラーアクションの挙動を制御するユーザー設定変数 `$ErrorActionPreference` を変更します。

https://learn.microsoft.com/ja-jp/powershell/module/microsoft.powershell.core/about/about_preference_variables#erroractionpreference

```ps1
# スクリプトの先頭などで設定する
$ErrorActionPreference = "Stop"

try {
  # 存在しないフォルダを削除しようとする（通常は終了しないエラー）
  Remove-Item "C:\DummyFolderThatDoesNotExist"
}
catch {
  Write-Host "フォルダ削除時にエラーが発生しました。"
}
# -> フォルダ削除時にエラーが発生しました。
```

または、コマンドレットごとに `-ErrorAction Stop` パラメータを付与することでも、局所的にエラーをキャッチさせることができます。

```ps1
try {
  Remove-Item "C:\DummyFolderThatDoesNotExist" -ErrorAction Stop
}
catch {
  Write-Warning "フォルダーを削除できませんでした: $($_.Exception.Message)"
}
```

事故を防ぐためにも、重要な処理を行うスクリプトではエラーハンドリングを徹底しておくべきです。
