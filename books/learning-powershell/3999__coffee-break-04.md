---
title: "☕ Coffee Break : 4"
---

## ActiveDirectory 管理センターのすすめ

![ActiveDirectory管理センターの操作画面](/images/books/learning-powershell/adac-08.png)

https://learn.microsoft.com/ja-jp/windows-server/identity/ad-ds/get-started/adac/use-active-directory-administrative-center-powershell-history

https://learn.microsoft.com/ja-jp/windows-server/identity/ad-ds/get-started/adac/advanced-ad-ds-management-using-active-directory-administrative-center--level-200-

GUI を使って ActiveDirectory を操作する場合、**ADUC**（ActiveDirectory ユーザーとコンピューター）を使うケースが多いかもしれません。

ここでは Windows Server 2008 R2（2012年〜）以降に追加された **ActiveDirectory 管理センター**（**ADAC**：*Active Directory Administrative Center* ）を使った方法について説明します。

:::message

以降は 略称として **AD 管理センター**と呼ぶようにします。

:::

### ActiveDirectory 管理センターとは

ActiveDirectory 管理センター（ADAC）は GUI を通じて ActiveDirectory を操作することが可能です。

一見、「ActiveDirectory ユーザーとコンピューター」と同じように見えますが、その実態は **GUI の皮を被った PowerShell** です。

画面上で行った　ユーザー作成　や　グループ変更　などの操作は、すべてバックグラウンドで **Windows PowerShell コマンドレット** に変換されて実行されています。

### ADACとPowerShellの強力な連携機能

ADAC を勧める最たる理由は、**自分がGUIで行った操作の「生のPowerShellコード」をリアルタイムで確認できる機能**が備わっているためです。

![ActiveDirectory管理センターの操作画面](/images/books/learning-powershell/adac-08.png)

上図画面の下にある「Windows PowerShell 履歴」タスクペインが、それにあたります。GUIでボタンをクリックするたびに、裏で走った正確なPowerShellコマンドと引数、値がすべてログとして出力されます。

履歴ペインに表示されたコマンドは、そのままコピーして使い回せます。例えば「1人分のユーザー作成手順」を GUI で行えば、**その正確なコードを使って「100人分の一括作成スクリプト」を簡単に作ることができます**。

### 削除してしまったデータの「ごみ箱機能」

**ADUC** では、ごみ箱を有効化する機能自体がなく、削除されたオブジェクトを復元するには高度なコマンド操作が必要になります

一方で **ADAC** では、画面上に「ごみ箱の有効化」ボタンがあり、間違えて消したユーザーやPCも、**画面上で右クリックして「復元」**を選ぶだけで一発で元に戻すことができます。

### 一連の操作

ここでは、「ADAC の起動からユーザーを作成し、そのコードを確認する」という一連の操作を順を追って見てみます。

#### AD 管理センターの起動

AD 管理センターは、サーバーマネージャーのツール欄から起動できます。

![サーバーマネージャーのツール欄から管理センターを起動できる](/images/books/learning-powershell/adac-01.png)

下図が、AD 管理センターの起動画面です。

![ActiveDirectory管理センター](/images/books/learning-powershell/adac-02.png)

:::message

ドメインの中を見てみると、下図のように AD 構造が階層管理されていることがわかります。

![AD構造が階層管理されている](/images/books/learning-powershell/adac-03.png)

:::

#### ユーザーの作成

ユーザーを作成したい階層まで辿り、そこで「新規」からユーザーを作成することができます。ここでは `OU=AZR,OU=People,DC=fuga,DC=jp` にユーザーを作成してみます。

![特定のOU内で、ユーザーを作成できる](/images/books/learning-powershell/adac-04.png)

下図が、ユーザーの作成画面となります。赤字で `*` の箇所が、作成に必須となる入力欄です。

![ユーザーの作成画面](/images/books/learning-powershell/adac-05.png)

ここでは、ダミーの情報で埋めてみました。

![ダミーでユーザー情報を埋めた状態、アカウント基本情報系](/images/books/learning-powershell/adac-06.png)

![ダミーでユーザー情報を埋めた状態、グループなど](/images/books/learning-powershell/adac-07.png)

これで「OK」ボタンを押すと、該当する DN の場所にユーザーが作成されます。

#### コマンド履歴を確認

「ActiveDirectory ユーザーとコンピューター」と大きく異なるのは ここからです。

作成した際の AD 管理センター画面が下図となります。画面下部に、Windows PowerShell 履歴という部分に何か記録されているのが確認できます。

![ユーザーを作成すると、その際のコマンド内容がPowerShell履歴に記録されている](/images/books/learning-powershell/adac-08.png)

これをコピーしたのが下記となります。

```ps1
New-ADUser -Company:"Hoge" -Department:"営業部" -DisplayName:"田中 太郎" -GivenName:"太郎" -Name:"田中 太郎" -Office:"Tokyo" -OfficePhone:"012-3456-7890" -OtherAttributes:@{"msDS-PhoneticDepartment"="エイギョウブ";"msDS-PhoneticDisplayName"="タナカ タロウ";"msDS-PhoneticFirstName"="タロウ";"msDS-PhoneticLastName"="タナカ"} -Path:"OU=AZR,OU=People,DC=fuga,DC=jp" -SamAccountName:"hoge001" -Server:"WIN-VDVBHE5VJ80.fuga.jp" -Surname:"田中" -Type:"user" -UserPrincipalName:"hoge001@fuga.jp"
# Set-ADAccountPassword -Identity:"CN=田中 太郎,OU=AZR,OU=People,DC=fuga,DC=jp" -NewPassword:"System.Security.SecureString" -Reset:$true -Server:"WIN-VDVBHE5VJ80.fuga.jp"
Enable-ADAccount -Identity:"CN=田中 太郎,OU=AZR,OU=People,DC=fuga,DC=jp" -Server:"WIN-VDVBHE5VJ80.fuga.jp"
Add-ADPrincipalGroupMembership -Identity:"CN=田中 太郎,OU=AZR,OU=People,DC=fuga,DC=jp" -MemberOf:"CN=RO-mta-distlist1,OU=Test,OU=SEC,OU=Stage,DC=fuga,DC=jp","CN=Administrators,CN=Builtin,DC=fuga,DC=jp" -Server:"WIN-VDVBHE5VJ80.fuga.jp"
Set-ADAccountControl -AccountNotDelegated:$false -AllowReversiblePasswordEncryption:$false -CannotChangePassword:$false -DoesNotRequirePreAuth:$false -Identity:"CN=田中 太郎,OU=AZR,OU=People,DC=fuga,DC=jp" -PasswordNeverExpires:$false -Server:"WIN-VDVBHE5VJ80.fuga.jp" -UseDESKeyOnly:$false
Set-ADUser -ChangePasswordAtLogon:$true -Identity:"CN=田中 太郎,OU=AZR,OU=People,DC=fuga,DC=jp" -Server:"WIN-VDVBHE5VJ80.fuga.jp" -SmartcardLogonRequired:$false
```

:::message

AD 管理センター上では、**全ての操作が PowerShell コマンドとして出力されるわけではありません**。

例えば 上記コードでは `Set-ADAccountPassword` がコメントアウトになっています。これは、手打ちしたパスワードが `SecureString` として扱われておりコードとして再現することができないためです。  
（再現できるというのは、コード上にパスワード文字列が存在するということになってしまいます）

他にも、例えば 属性エディタの項目なども再現することができなかったりします。以上のことから、AD 管理センター上でコマンド出力できるのは一部に留まるというのは覚えておいたほうが良いでしょう。

:::

ここからわかる通り、AD 管理センターの大きな特徴は「**操作した内容を PowerShell コマンドとして取得できる**」ことです。
