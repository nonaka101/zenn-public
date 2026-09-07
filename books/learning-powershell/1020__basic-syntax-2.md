---
title: "🐣 基本構文 2"
---

## 演算子

https://learn.microsoft.com/ja-jp/powershell/module/microsoft.powershell.core/about/about_operators

ここでは基本的な演算子に絞って取り上げてみます。

### 算術演算子

https://learn.microsoft.com/ja-jp/powershell/module/microsoft.powershell.core/about/about_arithmetic_operators

一般的な四則演算（`+`, `-`, `*`, `/`）や剰余（`%`）を使用できます。

### 代入演算子

https://learn.microsoft.com/ja-jp/powershell/module/microsoft.powershell.core/about/about_assignment_operators

値の代入（`=`）や、加算代入（`+=`）、減算代入（`-=`）などが使用できます。

### 比較演算子

https://learn.microsoft.com/ja-jp/powershell/module/microsoft.powershell.core/about/about_comparison_operators

PowerShell では、他のプログラミング言語のような比較演算子 `==`, `>=` といったものはなく、`-eq` といった形式で利用します。比較演算子は、If 文での条件式によく利用されます。

基本的な、数値同士の比較は下記のようになります。

```ps1
# Equal : 等しい
1 -eq 1 # -> True
1 -eq 2 # -> False

## Not Equal : 等しくない
1 -ne 2 # -> True
1 -ne 1 # -> False

# Greater Than : より大きい
1 -gt 2 # -> False
1 -gt 0 # -> True

# Greater Equal : 以上
1 -ge 0 # -> True
1 -ge 1 # -> True

# Less Than : より小さい
1 -lt 0 # -> False
1 -lt 2 # -> True

# Less Equal : 以下
1 -le 0 # -> False
1 -le 1 # -> True
```

#### 文字列に対する比較演算子

この大小関係等を表す演算子は、数値だけでなく文字列でも利用することができます。

```ps1
# 文字列同士についても大小比較が可能
"a" -gt "b" # -> False
"a" -le "b" # -> True
```

ただ、これでは大文字小文字の区別は行えません。大文字小文字を区別したい場合は `c`（ *Case sensitive* ）をつけます。

```ps1
"a" -eq "a" # -> True
"A" -eq "a" # -> True

# 大文字小文字を区別する (Case Equal)
"A" -ceq "a" # -> False
```

#### 文字列の一致判定

特定の文字が含まれているかを判定するためには、ワイルドカード（`*` や `?`）を使った `-like` 演算子が便利です。

```ps1
"Apple" -eq "A*"      # -> False

# Like 演算子を使った、ワイルドカード検索
"Apple" -like "A*"    # -> True
"Apple" -like "*p*"   # -> True
"Apple" -like "A??le" # -> True
"Ankle" -like "A??le" # -> True
```

正規表現を用いた一致判定を行う場合は、`-match` を使用します。

```ps1
"My name is Alice" -match "alice" # -> True

# 大文字小文字の区別は -cmatch
"My name is Alice" -cmatch "alice" # -> False
```

### 論理演算子

https://learn.microsoft.com/ja-jp/powershell/module/microsoft.powershell.core/about/about_logical_operators

PowerShell では、論理演算子として（他言語で言う）`&&` や `||` ではなく、`-and` や `-or`, `-not` といった演算子を利用します。

:::message

なお PowerShell 7 以降において `&&` や `||` は、パイプラインチェーン演算子という別の用途で使用されています。

:::

```ps1
$a = 5
$a -gt 0 -and $a -lt 10 # -> True
$a -gt 0 -and $a -gt 10 # -> False

# 括弧を使った計算の優先度については、他言語と同様
($a -gt 0) -and (($a -ge 20) -or ($a -le 10)) # -> True
```

### その他の演算子

#### リダイレクト演算子

https://learn.microsoft.com/ja-jp/powershell/module/microsoft.powershell.core/about/about_redirection

#### `&` を使った演算子

##### 呼び出し演算子 `&`

https://learn.microsoft.com/ja-jp/powershell/module/microsoft.powershell.core/about/about_operators#call-operator-

##### Background 演算子 `&`

https://learn.microsoft.com/ja-jp/powershell/module/microsoft.powershell.core/about/about_operators#background-operator-

## 条件分岐

### If 文

`if` を使った条件分岐は、他言語でもよく見られる `if (条件式) {True の場合の処理}` の形が基礎となります。

:::message

条件式は `if ($hoge -eq $true) { ... }` のように書くこともできますが、真偽値であれば直接 `if ($hoge) { ... }` と書くことも可能です。

```ps1
$pathFile = 'C:\temp\test.txt'
If (Test-Path $pathFile) {
  # ファイルが存在する場合
} else {
  # ファイルが存在しない場合
}
```

数値の場合は `0` が `$false`、それ以外が `$true` として評価されます。

```ps1
if (1) { "This is true" } # -> This is true
if (0) { "This is true" } # -> (何も出力されない)
```

:::

また `elseif`, `else` を使うことで、より複雑な条件分岐が行なえます。

```ps1
<#
if(条件式1){
  条件式1 が True である場合の処理
} elseif(条件式2) {
  条件式2 が True である場合の処理
} else {
  これまでの条件式が当てはまらない場合の処理
}
#>

$score = 85

if ($score -ge 90) {
  Write-Host "優秀です！"
} elseif ($score -ge 70) {
  Write-Host "合格です！"
} else {
  Write-Host "不合格です…"
}
# -> 合格です！
```

### Switch 文

https://learn.microsoft.com/ja-jp/powershell/module/microsoft.powershell.core/about/about_switch

複数の条件をシンプルに記載したい場合は `switch` 文が便利です。

```ps1
<#
switch (値) {
  条件1 {
    # 値が条件1を満たす場合に実行する処理
  }
  条件2 {
    # 値が条件2を満たす場合に実行する処理
  }
  default {
    # どの条件も満たさない場合に実行する処理
  }
}
#>

$signal = "yellow"

switch ($signal) {
  "red" {
    Write-Host "止まれ！"
  }
  "yellow" {
    Write-Host "注意！"
  }
  "green" {
    Write-Host "進め！"
  }
  default {
    Write-Host "不明な信号です"
  }
}
# -> 注意！
```

また `switch` 文では、条件部分にスクリプトブロック（`{...}`）を記述して、動的な評価を行うことも可能です。

```ps1
$hoge = 10

switch ($hoge) {
  {$_ -lt 0} {"0 より小さい"}
  {$_ -lt 7} {"7 より小さい"}
  {$_ -lt 14} {"14 より小さい"}
}
# -> 14 より小さい
```

#### Switch構文の式評価について

`switch` 文を使う際、式評価について気をつける必要があります。

例えば下記のように**どれか単一の式のみ当てはまる**場合については、特に問題はありません。

```ps1
#例2: サービスの状態確認
$serviceName = "wuauserv"
switch ((Get-Service -Name $serviceName).Status) {
  "Running" {
    Write-Host "サービスは実行中です"
  }
  "Stopped" {
    Write-Host "サービスは停止しています"
  }
  default {
    Write-Host "サービスの状態が不明です"
  }
}
```

一方で**複数の条件式が当てはまる**場合、条件に合致したものはすべて実行される点に注意してください（上から順に評価されます）。

```ps1
$hoge = 10

switch ($hoge) {
  {$_ -gt 10} { "10 より大きい" }
  {$_ -gt 5}  { "5 より大きい" }
  {$_ -gt 0}  { "0 より大きい" }
}
<#
5 より大きい
0 より大きい
#>
```

最初に合致した条件式で分岐を抜けたい場合は、`break` を使います。

```ps1
$hoge = 10

switch ($hoge) {
  {$_ -gt 10} { "10 より大きい"; break }
  {$_ -gt 5}  { "5 より大きい"; break }
  {$_ -gt 0}  { "0 より大きい"; break }
}
<#
5 より大きい
#>
```

## ループ処理

PowerShell には複数のループ処理（繰り返し）の構文が用意されています。

### for 構文

決まった回数だけ処理を繰り返す場合に使用します。

```ps1
<#
for (初期化; 条件式; 更新) {
  # ループ処理
}
#>

# i を 1 からインクリメントする形で、5 以下である限り 繰り返す
for ($i = 1; $i -le 5; $i++) {
  Write-Host $i
}
<#
1
2
3
4
5
#>
```

### while 構文

指定した条件が `$true` である間、ループ処理を継続します。

```ps1
<#
while (条件式) {
  # ループ処理
}
#>

$i = 1
# i が 5 以下である限り 繰り返す
while ($i -le 5) {
  Write-Host $i
  $i++  # i をインクリメント（ +1 ）
}
<#
1
2
3
4
5
#>
```

### foreach 構文

配列などのコレクションから、要素を1つずつ取り出して処理する場合に非常に便利です。

```ps1
<#
ForEach ($要素 in $配列) {
  # 各要素に対して実行する処理
}
#>

$names = "太郎", "次郎", "花子"
foreach ($name in $names) {
  Write-Host "こんにちは、$name さん！"
}
<#
こんにちは、太郎 さん！
こんにちは、次郎 さん！
こんにちは、花子 さん！
#>
```

### ForEach-Object コマンドレット

パイプライン（後述）から渡されたオブジェクトを処理する場合に使用します。渡された要素は `$_`（自動変数）に格納されます。

```ps1
$array = @(1, 2, 3, 4, 5) # 配列に 1 〜 5 を格納

# 配列の数値を 2 倍にして出力
$array | ForEach-Object {
  $_ * 2
}
<#
2
4
6
8
10
#>


# `1..5` ->「1 〜 5 を格納した配列」と同様の働きをします
1..5 | ForEach-Object {
  if($_ % 2){
    "$_ is odd."
  }
}
# 1 is odd.
# 3 is odd.
# 5 is odd.
```

パイプラインと `ForEach-Object` は PowerShell の中核となる機能ですので、後の章で詳しく解説します。
