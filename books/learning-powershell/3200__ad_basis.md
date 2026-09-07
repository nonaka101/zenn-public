---
title: "🐓 AD：基礎"
---

PowerShell では様々なパラメータを扱うことになりますが、その中でも特に ActiveDirectory 及び Microsoft Exchange に関するものが多いです。言い換えれば、その分 様々な項目を PowerShell 上で設定可能ということになります。

ここでは、Active Directory Domain Services（以下、AD と呼びます）に対して Windows PowerShell の `ActiveDirectory` モジュールを使用し、AD ユーザーを安全に作成・設定・確認する基本的な流れを説明します。

## ActiveDirectory モジュールを準備する

https://learn.microsoft.com/ja-jp/powershell/module/activedirectory/

### モジュールが利用可能か確認する

`Get-Module` を使い、`ActiveDirectory` モジュールの存在を確認します。

```ps1
Get-Module -ListAvailable -Name ActiveDirectory
<#
  ディレクトリ: C:\Windows\system32\WindowsPowerShell\v1.0\Modules


ModuleType Version    Name                                ExportedCommands
---------- -------    ----                                ----------------
Manifest   1.0.1.0    ActiveDirectory                     {Add-ADCentralAccessPolicyMember, Add-ADComputerServiceAcc...
#>
```

:::message

出力された一覧の中に `ActiveDirectory` が表示されない場合は、機能が導入されていない可能性があります。

その場合は、RSAT（リモートサーバー管理ツール）の AD DS 管理ツールなど、必要な機能を導入してください。

:::

### モジュールを読み込む

```ps1
Import-Module ActiveDirectory -ErrorAction Stop

# 読み込まれたモジュールを確認
Get-Module -Name ActiveDirectory
<#
ModuleType Version    Name                                ExportedCommands
---------- -------    ----                                ----------------
Manifest   1.0.1.0    ActiveDirectory                     {Add-ADCentralAccessPolicyMember, Add-ADComputerServiceAcc...
#>
```

### 利用できるコマンドを確認する

```ps1
Get-Command -Module ActiveDirectory
<#
CommandType     Name                                               Version    Source
-----------     ----                                               -------    ------
Cmdlet          Add-ADCentralAccessPolicyMember                    1.0.1.0    ActiveDirectory
Cmdlet          Add-ADComputerServiceAccount                       1.0.1.0    ActiveDirectory
Cmdlet          Add-ADDomainControllerPasswordReplicationPolicy    1.0.1.0    ActiveDirectory
Cmdlet          Add-ADFineGrainedPasswordPolicySubject             1.0.1.0    ActiveDirectory
Cmdlet          Add-ADGroupMember                                  1.0.1.0    ActiveDirectory
Cmdlet          Add-ADPrincipalGroupMembership                     1.0.1.0    ActiveDirectory
Cmdlet          Add-ADResourcePropertyListMember                   1.0.1.0    ActiveDirectory
Cmdlet          Clear-ADAccountExpiration                          1.0.1.0    ActiveDirectory
Cmdlet          Clear-ADClaimTransformLink                         1.0.1.0    ActiveDirectory
Cmdlet          Disable-ADAccount                                  1.0.1.0    ActiveDirectory
Cmdlet          Disable-ADOptionalFeature                          1.0.1.0    ActiveDirectory
Cmdlet          Enable-ADAccount                                   1.0.1.0    ActiveDirectory
Cmdlet          Enable-ADOptionalFeature                           1.0.1.0    ActiveDirectory
Cmdlet          Get-ADAccountAuthorizationGroup                    1.0.1.0    ActiveDirectory
Cmdlet          Get-ADAccountResultantPasswordReplicationPolicy    1.0.1.0    ActiveDirectory
Cmdlet          Get-ADAuthenticationPolicy                         1.0.1.0    ActiveDirectory
Cmdlet          Get-ADAuthenticationPolicySilo                     1.0.1.0    ActiveDirectory
Cmdlet          Get-ADCentralAccessPolicy                          1.0.1.0    ActiveDirectory
Cmdlet          Get-ADCentralAccessRule                            1.0.1.0    ActiveDirectory
Cmdlet          Get-ADClaimTransformPolicy                         1.0.1.0    ActiveDirectory
Cmdlet          Get-ADClaimType                                    1.0.1.0    ActiveDirectory
Cmdlet          Get-ADComputer                                     1.0.1.0    ActiveDirectory
Cmdlet          Get-ADComputerServiceAccount                       1.0.1.0    ActiveDirectory
Cmdlet          Get-ADDCCloningExcludedApplicationList             1.0.1.0    ActiveDirectory
Cmdlet          Get-ADDefaultDomainPasswordPolicy                  1.0.1.0    ActiveDirectory
Cmdlet          Get-ADDomain                                       1.0.1.0    ActiveDirectory
Cmdlet          Get-ADDomainController                             1.0.1.0    ActiveDirectory
Cmdlet          Get-ADDomainControllerPasswordReplicationPolicy    1.0.1.0    ActiveDirectory
Cmdlet          Get-ADDomainControllerPasswordReplicationPolicy... 1.0.1.0    ActiveDirectory
Cmdlet          Get-ADFineGrainedPasswordPolicy                    1.0.1.0    ActiveDirectory
Cmdlet          Get-ADFineGrainedPasswordPolicySubject             1.0.1.0    ActiveDirectory
Cmdlet          Get-ADForest                                       1.0.1.0    ActiveDirectory
Cmdlet          Get-ADGroup                                        1.0.1.0    ActiveDirectory
Cmdlet          Get-ADGroupMember                                  1.0.1.0    ActiveDirectory
Cmdlet          Get-ADObject                                       1.0.1.0    ActiveDirectory
Cmdlet          Get-ADOptionalFeature                              1.0.1.0    ActiveDirectory
Cmdlet          Get-ADOrganizationalUnit                           1.0.1.0    ActiveDirectory
Cmdlet          Get-ADPrincipalGroupMembership                     1.0.1.0    ActiveDirectory
Cmdlet          Get-ADReplicationAttributeMetadata                 1.0.1.0    ActiveDirectory
Cmdlet          Get-ADReplicationConnection                        1.0.1.0    ActiveDirectory
Cmdlet          Get-ADReplicationFailure                           1.0.1.0    ActiveDirectory
Cmdlet          Get-ADReplicationPartnerMetadata                   1.0.1.0    ActiveDirectory
Cmdlet          Get-ADReplicationQueueOperation                    1.0.1.0    ActiveDirectory
Cmdlet          Get-ADReplicationSite                              1.0.1.0    ActiveDirectory
Cmdlet          Get-ADReplicationSiteLink                          1.0.1.0    ActiveDirectory
Cmdlet          Get-ADReplicationSiteLinkBridge                    1.0.1.0    ActiveDirectory
Cmdlet          Get-ADReplicationSubnet                            1.0.1.0    ActiveDirectory
Cmdlet          Get-ADReplicationUpToDatenessVectorTable           1.0.1.0    ActiveDirectory
Cmdlet          Get-ADResourceProperty                             1.0.1.0    ActiveDirectory
Cmdlet          Get-ADResourcePropertyList                         1.0.1.0    ActiveDirectory
Cmdlet          Get-ADResourcePropertyValueType                    1.0.1.0    ActiveDirectory
Cmdlet          Get-ADRootDSE                                      1.0.1.0    ActiveDirectory
Cmdlet          Get-ADServiceAccount                               1.0.1.0    ActiveDirectory
Cmdlet          Get-ADTrust                                        1.0.1.0    ActiveDirectory
Cmdlet          Get-ADUser                                         1.0.1.0    ActiveDirectory
Cmdlet          Get-ADUserResultantPasswordPolicy                  1.0.1.0    ActiveDirectory
Cmdlet          Grant-ADAuthenticationPolicySiloAccess             1.0.1.0    ActiveDirectory
Cmdlet          Install-ADServiceAccount                           1.0.1.0    ActiveDirectory
Cmdlet          Move-ADDirectoryServer                             1.0.1.0    ActiveDirectory
Cmdlet          Move-ADDirectoryServerOperationMasterRole          1.0.1.0    ActiveDirectory
Cmdlet          Move-ADObject                                      1.0.1.0    ActiveDirectory
Cmdlet          New-ADAuthenticationPolicy                         1.0.1.0    ActiveDirectory
Cmdlet          New-ADAuthenticationPolicySilo                     1.0.1.0    ActiveDirectory
Cmdlet          New-ADCentralAccessPolicy                          1.0.1.0    ActiveDirectory
Cmdlet          New-ADCentralAccessRule                            1.0.1.0    ActiveDirectory
Cmdlet          New-ADClaimTransformPolicy                         1.0.1.0    ActiveDirectory
Cmdlet          New-ADClaimType                                    1.0.1.0    ActiveDirectory
Cmdlet          New-ADComputer                                     1.0.1.0    ActiveDirectory
Cmdlet          New-ADDCCloneConfigFile                            1.0.1.0    ActiveDirectory
Cmdlet          New-ADFineGrainedPasswordPolicy                    1.0.1.0    ActiveDirectory
Cmdlet          New-ADGroup                                        1.0.1.0    ActiveDirectory
Cmdlet          New-ADObject                                       1.0.1.0    ActiveDirectory
Cmdlet          New-ADOrganizationalUnit                           1.0.1.0    ActiveDirectory
Cmdlet          New-ADReplicationSite                              1.0.1.0    ActiveDirectory
Cmdlet          New-ADReplicationSiteLink                          1.0.1.0    ActiveDirectory
Cmdlet          New-ADReplicationSiteLinkBridge                    1.0.1.0    ActiveDirectory
Cmdlet          New-ADReplicationSubnet                            1.0.1.0    ActiveDirectory
Cmdlet          New-ADResourceProperty                             1.0.1.0    ActiveDirectory
Cmdlet          New-ADResourcePropertyList                         1.0.1.0    ActiveDirectory
Cmdlet          New-ADServiceAccount                               1.0.1.0    ActiveDirectory
Cmdlet          New-ADUser                                         1.0.1.0    ActiveDirectory
Cmdlet          Remove-ADAuthenticationPolicy                      1.0.1.0    ActiveDirectory
Cmdlet          Remove-ADAuthenticationPolicySilo                  1.0.1.0    ActiveDirectory
Cmdlet          Remove-ADCentralAccessPolicy                       1.0.1.0    ActiveDirectory
Cmdlet          Remove-ADCentralAccessPolicyMember                 1.0.1.0    ActiveDirectory
Cmdlet          Remove-ADCentralAccessRule                         1.0.1.0    ActiveDirectory
Cmdlet          Remove-ADClaimTransformPolicy                      1.0.1.0    ActiveDirectory
Cmdlet          Remove-ADClaimType                                 1.0.1.0    ActiveDirectory
Cmdlet          Remove-ADComputer                                  1.0.1.0    ActiveDirectory
Cmdlet          Remove-ADComputerServiceAccount                    1.0.1.0    ActiveDirectory
Cmdlet          Remove-ADDomainControllerPasswordReplicationPolicy 1.0.1.0    ActiveDirectory
Cmdlet          Remove-ADFineGrainedPasswordPolicy                 1.0.1.0    ActiveDirectory
Cmdlet          Remove-ADFineGrainedPasswordPolicySubject          1.0.1.0    ActiveDirectory
Cmdlet          Remove-ADGroup                                     1.0.1.0    ActiveDirectory
Cmdlet          Remove-ADGroupMember                               1.0.1.0    ActiveDirectory
Cmdlet          Remove-ADObject                                    1.0.1.0    ActiveDirectory
Cmdlet          Remove-ADOrganizationalUnit                        1.0.1.0    ActiveDirectory
Cmdlet          Remove-ADPrincipalGroupMembership                  1.0.1.0    ActiveDirectory
Cmdlet          Remove-ADReplicationSite                           1.0.1.0    ActiveDirectory
Cmdlet          Remove-ADReplicationSiteLink                       1.0.1.0    ActiveDirectory
Cmdlet          Remove-ADReplicationSiteLinkBridge                 1.0.1.0    ActiveDirectory
Cmdlet          Remove-ADReplicationSubnet                         1.0.1.0    ActiveDirectory
Cmdlet          Remove-ADResourceProperty                          1.0.1.0    ActiveDirectory
Cmdlet          Remove-ADResourcePropertyList                      1.0.1.0    ActiveDirectory
Cmdlet          Remove-ADResourcePropertyListMember                1.0.1.0    ActiveDirectory
Cmdlet          Remove-ADServiceAccount                            1.0.1.0    ActiveDirectory
Cmdlet          Remove-ADUser                                      1.0.1.0    ActiveDirectory
Cmdlet          Rename-ADObject                                    1.0.1.0    ActiveDirectory
Cmdlet          Reset-ADServiceAccountPassword                     1.0.1.0    ActiveDirectory
Cmdlet          Restore-ADObject                                   1.0.1.0    ActiveDirectory
Cmdlet          Revoke-ADAuthenticationPolicySiloAccess            1.0.1.0    ActiveDirectory
Cmdlet          Search-ADAccount                                   1.0.1.0    ActiveDirectory
Cmdlet          Set-ADAccountAuthenticationPolicySilo              1.0.1.0    ActiveDirectory
Cmdlet          Set-ADAccountControl                               1.0.1.0    ActiveDirectory
Cmdlet          Set-ADAccountExpiration                            1.0.1.0    ActiveDirectory
Cmdlet          Set-ADAccountPassword                              1.0.1.0    ActiveDirectory
Cmdlet          Set-ADAuthenticationPolicy                         1.0.1.0    ActiveDirectory
Cmdlet          Set-ADAuthenticationPolicySilo                     1.0.1.0    ActiveDirectory
Cmdlet          Set-ADCentralAccessPolicy                          1.0.1.0    ActiveDirectory
Cmdlet          Set-ADCentralAccessRule                            1.0.1.0    ActiveDirectory
Cmdlet          Set-ADClaimTransformLink                           1.0.1.0    ActiveDirectory
Cmdlet          Set-ADClaimTransformPolicy                         1.0.1.0    ActiveDirectory
Cmdlet          Set-ADClaimType                                    1.0.1.0    ActiveDirectory
Cmdlet          Set-ADComputer                                     1.0.1.0    ActiveDirectory
Cmdlet          Set-ADDefaultDomainPasswordPolicy                  1.0.1.0    ActiveDirectory
Cmdlet          Set-ADDomain                                       1.0.1.0    ActiveDirectory
Cmdlet          Set-ADDomainMode                                   1.0.1.0    ActiveDirectory
Cmdlet          Set-ADFineGrainedPasswordPolicy                    1.0.1.0    ActiveDirectory
Cmdlet          Set-ADForest                                       1.0.1.0    ActiveDirectory
Cmdlet          Set-ADForestMode                                   1.0.1.0    ActiveDirectory
Cmdlet          Set-ADGroup                                        1.0.1.0    ActiveDirectory
Cmdlet          Set-ADObject                                       1.0.1.0    ActiveDirectory
Cmdlet          Set-ADOrganizationalUnit                           1.0.1.0    ActiveDirectory
Cmdlet          Set-ADReplicationConnection                        1.0.1.0    ActiveDirectory
Cmdlet          Set-ADReplicationSite                              1.0.1.0    ActiveDirectory
Cmdlet          Set-ADReplicationSiteLink                          1.0.1.0    ActiveDirectory
Cmdlet          Set-ADReplicationSiteLinkBridge                    1.0.1.0    ActiveDirectory
Cmdlet          Set-ADReplicationSubnet                            1.0.1.0    ActiveDirectory
Cmdlet          Set-ADResourceProperty                             1.0.1.0    ActiveDirectory
Cmdlet          Set-ADResourcePropertyList                         1.0.1.0    ActiveDirectory
Cmdlet          Set-ADServiceAccount                               1.0.1.0    ActiveDirectory
Cmdlet          Set-ADUser                                         1.0.1.0    ActiveDirectory
Cmdlet          Show-ADAuthenticationPolicyExpression              1.0.1.0    ActiveDirectory
Cmdlet          Sync-ADObject                                      1.0.1.0    ActiveDirectory
Cmdlet          Test-ADServiceAccount                              1.0.1.0    ActiveDirectory
Cmdlet          Uninstall-ADServiceAccount                         1.0.1.0    ActiveDirectory
Cmdlet          Unlock-ADAccount                                   1.0.1.0    ActiveDirectory
#>
```

## AD ユーザーを取得する

:::message

ここでは、ランダムに AD 構成を自動生成する `BadBlood` というツールを使った構成上において、作成したユーザーを使って説明します。

:::

### `Get-ADUser`：ADUser オブジェクトを取得する

https://learn.microsoft.com/ja-jp/powershell/module/activedirectory/get-aduser

`Get-ADUser` は `ADUser`（`Microsoft.ActiveDirectory.Management.ADUser`）オブジェクトを返します。

`-Identity` には、ユーザーを識別できる一意の情報を指定します。例えば `DN`, `ObjectGUID`, `SID`, `SAMAccountName` などが挙げられます。

```ps1
# SAM アカウント名で検索する
Get-ADUser -Identity "hoge001"
<#
DistinguishedName : CN=田中 太郎,OU=AZR,OU=People,DC=fuga,DC=jp
Enabled           : True
GivenName         : 太郎
Name              : 田中 太郎
ObjectClass       : user
ObjectGUID        : 465de488-61f4-4f69-9965-4ed3b50a47af
SamAccountName    : hoge001
SID               : S-1-5-21-1671859770-434810290-2222081234-4188
Surname           : 田中
UserPrincipalName : hoge001@fuga.jp
#>
```

#### 補足：DN とは

**DN**（ *Distinguished Name* ）は、AD 内におけるオブジェクトの位置を示します。

```txt
CN=田中 太郎,OU=AZR,OU=People,DC=fuga,DC=jp
```

| 各名称 | 意味 | 上記の場合 |
| --- | --- | --- |
| CN（ *Common Name* ） | オブジェクト名 | `CN=田中 太郎` |
| OU（ *Organizational Unit* ） | ユーザー格納先 | `OU=AZR`, `OU=People` |
| DC（ *Domain Component* ） | AD ドメイン `fuga.jp` | `DC=fuga,DC=jp` |

### 既定以外のプロパティを取得する

`Get-ADUser` で取得した場合、ADUser オブジェクトに格納された情報のみ扱います。

```ps1
# 既に取得されたプロパティを一覧表示
Get-ADUser -Identity "hoge001" | Format-List *
<#
DistinguishedName  : CN=田中 太郎,OU=AZR,OU=People,DC=fuga,DC=jp
Enabled            : True
GivenName          : 太郎
Name               : 田中 太郎
ObjectClass        : user
ObjectGUID         : 465de488-61f4-4f69-9965-4ed3b50a47af
SamAccountName     : hoge001
SID                : S-1-5-21-1671859770-434810290-2222081234-4188
Surname            : 田中
UserPrincipalName  : hogehoge@fuga.jp
PropertyNames      : {DistinguishedName, Enabled, GivenName, Name...}
AddedProperties    : {}
RemovedProperties  : {}
ModifiedProperties : {}
PropertyCount      : 10
#>
```

AD 上では他にも属性情報等を持っていますが、それを追加取得する場合は `-Properties` を使用します。

```ps1
# 既定以外の属性も AD から取得
Get-ADUser `
  -Identity "hoge001" `
  -Properties * |
  Format-List *
<#
AccountExpirationDate                :
accountExpires                       : 9223372036854775807
AccountLockoutTime                   :
AccountNotDelegated                  : False
AllowReversiblePasswordEncryption    : False
AuthenticationPolicy                 : {}
AuthenticationPolicySilo             : {}
BadLogonCount                        : 0
badPasswordTime                      : 0
badPwdCount                          : 0
CannotChangePassword                 : False
CanonicalName                        : fuga.jp/People/AZR/田中 太郎
Certificates                         : {}
City                                 :
CN                                   : 田中 太郎
codePage                             : 0
Company                              : Hoge
CompoundIdentitySupported            : {}
Country                              :
countryCode                          : 0
Created                              : 2026/08/27 11:31:10
createTimeStamp                      : 2026/08/27 11:31:10
Deleted                              :
Department                           : 営業部
Description                          :
DisplayName                          : 田中 太郎
DistinguishedName                    : CN=田中 太郎,OU=AZR,OU=People,DC=fuga,DC=jp
Division                             :
DoesNotRequirePreAuth                : False
dSCorePropagationData                : {2026/08/27 11:31:11, 1601/01/01 9:00:00}
EmailAddress                         : hogehoge@fuga.jp
EmployeeID                           :
EmployeeNumber                       :
Enabled                              : True
Fax                                  :
GivenName                            : 太郎
HomeDirectory                        :
HomedirRequired                      : False
HomeDrive                            :
HomePage                             :
HomePhone                            :
Initials                             :
instanceType                         : 4
isDeleted                            :
KerberosEncryptionType               : {}
LastBadPasswordAttempt               :
LastKnownParent                      :
lastLogoff                           : 0
lastLogon                            : 0
LastLogonDate                        :
LockedOut                            : False
logonCount                           : 0
LogonWorkstations                    :
Manager                              :
MemberOf                             : {CN=RO-mta-distlist1,OU=Test,OU=SEC,OU=Groups,DC=fuga,DC=jp, CN=Administrators,CN
                                       =Builtin,DC=fuga,DC=jp}
MNSLogonAccount                      : False
MobilePhone                          :
Modified                             : 2026/08/27 11:31:11
modifyTimeStamp                      : 2026/08/27 11:31:11
msDS-PhoneticDepartment              : エイギョウブ
msDS-PhoneticDisplayName             : タナカ タロウ
msDS-PhoneticFirstName               : タロウ
msDS-PhoneticLastName                : タナカ
msDS-User-Account-Control-Computed   : 8388608
Name                                 : 田中 太郎
nTSecurityDescriptor                 : System.DirectoryServices.ActiveDirectorySecurity
ObjectCategory                       : CN=Person,CN=Schema,CN=Configuration,DC=fuga,DC=jp
ObjectClass                          : user
ObjectGUID                           : 465de488-61f4-4f69-9965-4ed3b50a47af
objectSid                            : S-1-5-21-1671859770-434810290-2222081234-4188
Office                               : Tokyo
OfficePhone                          : 012-3456-7890
Organization                         :
OtherName                            :
PasswordExpired                      : True
PasswordLastSet                      :
PasswordNeverExpires                 : False
PasswordNotRequired                  : False
physicalDeliveryOfficeName           : Tokyo
POBox                                :
PostalCode                           :
PrimaryGroup                         : CN=Domain Users,CN=Users,DC=fuga,DC=jp
primaryGroupID                       : 513
PrincipalsAllowedToDelegateToAccount : {}
ProfilePath                          :
ProtectedFromAccidentalDeletion      : False
pwdLastSet                           : 0
SamAccountName                       : hoge001
sAMAccountType                       : 805306368
ScriptPath                           :
sDRightsEffective                    : 15
ServicePrincipalNames                : {}
SID                                  : S-1-5-21-1671859770-434810290-2222081234-4188
SIDHistory                           : {}
SmartcardLogonRequired               : False
sn                                   : 田中
State                                :
StreetAddress                        :
Surname                              : 田中
telephoneNumber                      : 012-3456-7890
Title                                :
TrustedForDelegation                 : False
TrustedToAuthForDelegation           : False
UseDESKeyOnly                        : False
userAccountControl                   : 512
userCertificate                      : {}
UserPrincipalName                    : hoge001@fuga.jp
uSNChanged                           : 81966
uSNCreated                           : 81954
whenChanged                          : 2026/08/27 11:31:11
whenCreated                          : 2026/08/27 11:31:10
PropertyNames                        : {AccountExpirationDate, accountExpires, AccountLockoutTime, AccountNotDelegated.
                                       ..}
AddedProperties                      : {}
RemovedProperties                    : {}
ModifiedProperties                   : {}
PropertyCount                        : 111
#>
```

:::message

`-Properties *` で全取得する使い方は、検証段階までに留めておくべきでしょう。

実務においては不要な情報まで取得するのは望ましくありません。下記のように必要最小限で引き出す運用を心がけるのが良いでしょう。

- `-Properties` では必要な属性だけを取得
- `Select-Object` で出力する項目を絞る

```ps1
# 取得する項目を絞り、コードの意図を明確にする
Get-ADUser `
  -Identity "hoge001" `
  -Properties Department, EmailAddress |
  Select-Object Name, Department, EmailAddress
<#
Name      Department EmailAddress
----      ---------- ------------
田中 太郎 営業部     hogehoge@fuga.jp
#>
```

:::

### 条件を指定してユーザーを検索する

基本例では `-Identity` を使った一般的な方法（`DN`, `ObjectGUID`, `SID`, `SAMAccountName` から**一意のユーザーを選択**）を挙げました。

ここでは別の方法を使い もう少し複雑な条件でのユーザーの取得を行ってみます。

#### `-Filter` を使って条件を絞る

`-Filter` を使用することで、例えば下記のような条件を使った絞り込みが行えるようになります。

- 有効化状態のユーザーから
- 部署が「営業部」のユーザーから

```ps1
# 有効なユーザーを検索（例：このケースだと数十人ヒット）
Get-ADUser -Filter { Enabled -eq $true }
<#
DistinguishedName : CN=JAY_BIRD,OU=Tear1,DC=fuga,DC=jp
Enabled           : True
GivenName         :
Name              : JAY_BIRD
ObjectClass       : user
ObjectGUID        : bfc31423-33ff-461d-83d3-5388f59bffb5
SamAccountName    : JAY_BIRD
SID               : S-1-5-21-1671859770-434810290-2222081234-2279
Surname           : JAY_BIRD
UserPrincipalName : JAY_BIRD@fuga.jp

DistinguishedName : CN=SCOT_PARK,OU=test,DC=fuga,DC=jp
Enabled           : True
GivenName         :
Name              : SCOT_PARK
ObjectClass       : user
ObjectGUID        : 205d2549-d8e0-423a-948e-dfa2aac9fc69
SamAccountName    : SCOT_PARK
SID               : S-1-5-21-1671859770-434810290-2222081234-2300
Surname           : SCOT_PARK
UserPrincipalName : SCOT_PARK@fuga.jp

...
#>

# 部署が営業部のユーザーを検索（例：このケースだと数十人ヒット）
Get-ADUser -Filter { Department -eq "営業部" }
<#
DistinguishedName : CN=KELSEY_GARNER,OU=SEC,OU=People,DC=fuga,DC=jp
Enabled           : True
GivenName         :
Name              : KELSEY_GARNER
ObjectClass       : user
ObjectGUID        : c09515b9-04a6-42d4-a8b4-f171e61ba04e
SamAccountName    : KELSEY_GARNER
SID               : S-1-5-21-1671859770-434810290-2222081234-1570
Surname           : KELSEY_GARNER
UserPrincipalName : KELSEY_GARNER@fuga.jp

DistinguishedName : CN=EDMOND_LE,OU=SEC,OU=People,DC=fuga,DC=jp
Enabled           : True
GivenName         :
Name              : EDMOND_LE
ObjectClass       : user
ObjectGUID        : 0d2d945b-3d9c-4498-8704-04bd5da78bc8
SamAccountName    : EDMOND_LE
SID               : S-1-5-21-1671859770-434810290-2222081234-2138
Surname           : EDMOND_LE
UserPrincipalName : EDMOND_LE@fuga.jp

...
#>
```

:::message

前述した `-Poroperties` 同様、このような大量な情報を扱う際も 検索結果を必要な項目に絞るようにするのが良いでしょう。

```ps1
Get-ADUser `
  -Filter { Enabled -eq $true } `
  -Properties Department, EmailAddress |
  Select-Object Name, SamAccountName, Department, EmailAddress
```

:::

#### `-SearchBase` で検索範囲を限定する

複数のドメインを扱っている場合や、ドメイン内の階層が複雑な場合は、`-SearchBase` を使って場所の絞り込みが可能です。

検索範囲を明示すると、処理対象が分かりやすくなり、意図しない OU まで検索することを防げます。

```ps1
# 指定の OU 内に絞り、ユーザーを取得（この例だと、OU内の全員）
Get-ADUser `
  -Filter * `
  -SearchBase "OU=People,DC=fuga,DC=jp"
<#
DistinguishedName : CN=MARLENE_GEORGE,OU=People,DC=fuga,DC=jp
Enabled           : True
GivenName         :
Name              : MARLENE_GEORGE
ObjectClass       : user
ObjectGUID        : 5e66f35c-4972-44a6-add4-a5cec9518d31
SamAccountName    : MARLENE_GEORGE
SID               : S-1-5-21-1671859770-434810290-2222081234-2565
Surname           : MARLENE_GEORGE
UserPrincipalName : MARLENE_GEORGE@fuga.jp

DistinguishedName : CN=LINA_SPENCE,OU=People,DC=fuga,DC=jp
Enabled           : True
GivenName         :
Name              : LINA_SPENCE
ObjectClass       : user
ObjectGUID        : bd25dda3-40bc-45b7-a799-c70f3f29d3a6
SamAccountName    : LINA_SPENCE
SID               : S-1-5-21-1671859770-434810290-2222081234-2316
Surname           : LINA_SPENCE
UserPrincipalName : LINA_SPENCE@fuga.jp

...
#>
```

## ADUser オブジェクトの中身を確認してみる

`Get-ADUser` で取得したオブジェクトのメンバーを確認してみます。

```ps1
$user = Get-ADUser -Identity "hoge001"

# ADUser オブジェクトは、様々なメンバーを持っている
$user.DistinguishedName   # -> CN=田中 太郎,OU=AZR,OU=People,DC=fuga,DC=jp
$user.SamAccountName      # -> hoge001
$user.GetType().FullName  # -> Microsoft.ActiveDirectory.Management.ADUser
$user.ToString()          # -> CN=田中 太郎,OU=AZR,OU=People,DC=fuga,DC=jp

# 全メンバーの出力
$user | Get-Member
<#
  TypeName: Microsoft.ActiveDirectory.Management.ADUser

Name              MemberType            Definition
----              ----------            ----------
Contains          Method                bool Contains(string propertyName)
Equals            Method                bool Equals(System.Object obj)
GetEnumerator     Method                System.Collections.IDictionaryEnumerator GetEnumerator()
GetHashCode       Method                int GetHashCode()
GetType           Method                type GetType()
ToString          Method                string ToString()
Item              ParameterizedProperty Microsoft.ActiveDirectory.Management.ADPropertyValueCollection Item(string p...
DistinguishedName Property              System.String DistinguishedName {get;set;}
Enabled           Property              System.Boolean Enabled {get;set;}
GivenName         Property              System.String GivenName {get;set;}
Name              Property              System.String Name {get;}
ObjectClass       Property              System.String ObjectClass {get;set;}
ObjectGUID        Property              System.Nullable`1[[System.Guid, mscorlib, Version=4.0.0.0, Culture=neutral, ...
SamAccountName    Property              System.String SamAccountName {get;set;}
SID               Property              System.Security.Principal.SecurityIdentifier SID {get;set;}
Surname           Property              System.String Surname {get;set;}
UserPrincipalName Property              System.String UserPrincipalName {get;set;}
#>
```

**`ADUser` オブジェクトは、後続の AD コマンドレットの `-Identity` へ直接渡せます**。スクリプトを記述する際は、個々の文字列による指定でなく オブジェクトを使った方がミスもなく便利です。

## AD ユーザーを作成する

本項では AD ユーザーを新規に作成する（一連の）処理について、下記の手順で行っていきます。

1. ユーザーを無効状態で作成する
2. パスワードを設定する
3. ユーザー属性を設定する
4. アカウント制御を設定する
5. グループへ追加する
6. ユーザーを有効化する
7. 作成結果を確認する

:::message

ここでは基本的な形として段階を分けています。これには説明のしやすさという部分もありますが、他にも下記のメリットがあります。

- 段階別に分けることで、エラー時の追跡が容易になる
- 仮に途中で処理に失敗しても、未設定のアカウントが利用される危険性が減る（アカウントは最後に有効化状態を切り替えるため）

なお 本項末尾の備考にて、まとまった形でのユーザー作成の方法も提示します。

:::

以降 作成する AD ユーザーについては、下表の環境を想定し操作していくものとします。

| 項目 | 値の例 |
| --- | --- |
| AD DNS ドメイン名 | `fuga.jp` |
| NetBIOS ドメイン名 | `FUGA` |
| ドメイン DN | `DC=fuga,DC=jp` |
| ドメインコントローラー | `dc01.fuga.jp` |
| ユーザー OU | `OU=Users,DC=fuga,DC=jp` |
| グループ OU | `OU=Groups,DC=fuga,DC=jp` |
| SAM アカウント名 | `hoge001` |
| UPN | `hoge001@fuga.jp` |
| 表示名 | `山田 太郎` |
| メールアドレス | `hogehoge@fuga.jp` |
| 社員番号 | `12345` |

### `New-ADUser`：AD ユーザーを作成する

https://learn.microsoft.com/ja-jp/powershell/module/activedirectory/new-aduser

```ps1
# スプラッティングを利用して、パラメータをまとめて作成
$paramNewADUser = @{
  Name              = "hoge001"
  SamAccountName    = "hoge001"
  UserPrincipalName = "hoge001@fuga.jp"
  Server            = "dc01.fuga.jp"
  Path              = "OU=Users,DC=fuga,DC=jp"
  DisplayName       = "山田 太郎"
  Surname           = "山田"
  GivenName         = "太郎"
  EmailAddress      = "hogehoge@fuga.jp"
  Title             = "Member"
  PassThru          = $true
}

$newUser = New-ADUser @paramNewADUser
```

:::message

`Enabled = $true` を指定していないため、ユーザーは無効状態で作成されます。

なおこのケースで `Enable` を指定しても、パスワードが存在しない場合 有効化は失敗します。

:::

#### `-PassThru`

`New-ADUser` は、**既定では作成結果をパイプラインへ出力しません**。

しかし `-PassThru` を指定すると、作成した `ADUser` オブジェクトを出力します。

```ps1
$paramNewADUser = @{
  Name              = "hoge001"
  SamAccountName    = "hoge001"
  UserPrincipalName = "hoge001@fuga.jp"
  Server            = "dc01.fuga.jp"
  Path              = "OU=Users,DC=fuga,DC=jp"
  DisplayName       = "山田 太郎"
  Surname           = "山田"
  GivenName         = "太郎"
  EmailAddress      = "hogehoge@fuga.jp"
  Title             = "Member"
  PassThru          = $true # <- 作成したオブジェクトを返すよう設定
}

$newUser = New-ADUser @paramNewADUser

# 作成したユーザーオブジェクトは、変数 `$newUser` に格納されている
$newUser.GetType().FullName # -> Microsoft.ActiveDirectory.Management.ADUser
$newUser.DistinguishedName  # -> CN=hoge001,OU=Users,DC=fuga,DC=jp
$newUser.Enabled      # -> False
```

**作成したユーザーに対する追加の操作をしていく場合**、DN（例：`"CN=hoge001,OU=Users,DC=fuga,DC=jp"`）を手動で組み立てるより、**`-PassThru` で取得したオブジェクトを使用する方が堅牢です**。

#### 補足：UPN とは

UPN（ *User Principal Name* ）は、AD ユーザーのサインイン名として使用される値です。形式は `ユーザー名@UPNサフィックス` で、例えば `hoge001@fuga.jp` といった形です。

:::message

**UPN とメールアドレスは、役割が異なります**。

| 項目 | 主な目的 | 値の例 |
| --- | --- | --- |
| UPN | AD ログオン名 | `hoge001@fuga.jp` |
| メールアドレス | メールの宛先 | `hogehoge@fuga.jp` |

ActiveDirectory には従来から SAM アカウント名（`DOMAIN\username` 形式、例えば `FUGA\hoge001` ）によるログオン方法があります。

しかし *複数ドメインや複数フォレストを運用している環境* では、ユーザーが所属ドメイン名を意識する必要がありました。

そこで利用されるのが UPN です。UPN を利用することで、ユーザーはメールアドレスに近い形式でログオンできるため、管理や運用が容易になります。

:::

## パスワードを設定する

### `SecureString` でパスワードを生成する

パスワードを設定するためには、`SecureString` 型のデータを準備しておく必要があります。

```ps1
# スクリプト上で作成する場合（平文を SecureString に変換）
$securePassword = ConvertTo-SecureString `
  -String 'Passw0rd' `
  -AsPlainText -Force

# 対話形式で入力する場合
$securePassword = Read-Host `
  -Prompt "設定する初期パスワードを入力してください" `
  -AsSecureString

# 中身は SecureString 型で、基本的には閲覧はできない
$securePassword # -> System.Security.SecureString
```

:::message alert

本項では説明のしやすさから、平文をスクリプトに記載する方法で行っています。

これはあくまで説明のためであり、実務においてパスワードを平文で直接スクリプトに記載するのは、**セキュリティ上 大きな問題**であることは ご承知おきください。

:::

### `Set-ADAccountPassword` を実行する

https://learn.microsoft.com/ja-jp/powershell/module/activedirectory/set-adaccountpassword

`Set-ADAccountPassword` を使うことで、パスワードの変更ができます。

`-NewPassword` には、先程作成した `SecureString` 型の値を渡します。

```ps1
$securePassword = ConvertTo-SecureString `
  -String 'Passw0rd' `
  -AsPlainText -Force

$paramSetADAccountPassword = @{
  Identity  = $newUser
  NewPassword = $securePassword
  Reset     = $true
  Server    = "dc01.fuga.jp"
}
Set-ADAccountPassword @paramSetADAccountPassword
```

:::message

`Reset = $true` は、現在のパスワードを指定せず、管理者としてパスワードを再設定することを表します。

:::

### 次回ログオン時のパスワード変更を要求する

https://learn.microsoft.com/ja-jp/powershell/module/activedirectory/set-aduser

AD ユーザーオブジェクトの編集は `Set-ADUser` を使う場合が多いです。上記ページのパラメータからわかる通り、非常に多くの項目が設定できます。

ここでは、ログオンした際にパスワード変更を促す設定を施します。

```ps1
Set-ADUser `
  -Identity $newUser `
  -ChangePasswordAtLogon $true `
  -Server "dc01.fuga.jp"
```

:::message

パスワードに関して、`Set-ADUser` で操作できる項目は、下記となります。  
（全て真偽値 `Boolean` を使った設定となります）

| パラメータ名 | 説明 |
| --- | --- |
| `-AllowReversiblePasswordEncryption` | アカウントに対して暗号の復元が可能なパスワードの保存（可逆的暗号化）を許可するか |
| `-CannotChangePassword` | ユーザー自身によるパスワードの変更を禁止するか |
| `-ChangePasswordAtLogon` | 次回ログオン時にユーザーへパスワード変更を強制するか |
| `-PasswordNeverExpires` | アカウントのパスワードが無期限（有効期限なし）にするか |
| `-PasswordNotRequired` | アカウントにパスワードを必須とするか |

:::

## `Set-ADUser`：ユーザー属性を変更する

https://learn.microsoft.com/ja-jp/powershell/module/activedirectory/set-aduser

`New-ADUser` は新しいユーザーを作成し、`Set-ADUser` は作成済みのユーザー情報を変更します。

```ps1
# 役職（Job Title）を変更
Set-ADUser `
  -Identity $newUser `
  -Title "Senior Member" `
  -Server "dc01.fuga.jp"
```

属性の変更方法については、**それが一般的なものか**によって操作が変わってきます。

### 専用パラメーターがある属性

**AD でよく使われる属性**については、**専用のパラメータ**が用意されています。例えば下記のようなものが挙げられます。

- 役職（`-Title`）
- 社員番号（`-EmployeeID`）
- メールアドレス（`-EmailAddress`）

```ps1
# 社員番号、役職、メールアドレスを設定する
Set-ADUser `
  -Identity $newUser `
  -EmployeeID "12345" `
  -Title "Senior Member"
  -EmailAddress "hogehoge@fuga.jp"
  -Server "dc01.fuga.jp"
```

### 専用パラメーターがない属性

AD で管理できる属性は非常に多く、その全てが専用パラメータとして用意されているわけではありません。

**あまり使われないようなパラメータ**（例：属性エディタで操作するようなもの）については、**AD 上で管理されている LDAP 表示名を直接指定する**ことで設定します。

```ps1
# アドレス帳の並び順についての値を "置換"
Set-ADUser `
  -Identity $newUser `
  -Replace @{ 'msDS-HABSeniorityIndex' = 10 } `
  -Server "dc01.fuga.jp"
```

:::message alert

`msDS-HABSeniorityIndex` のように LDAP 表示名にハイフンが含まれている場合、**クォート記号で文字列として明示**しておく必要があります。

明示されていない場合、PowerShell のパーサーが `-` を演算子などの構文として解釈し、エラーを引き起こす可能性があります。

:::

属性によっては、複数の値を管理するケースもあります。

```ps1
# プロキシアドレスの値（複数）を "追加"
Set-ADUser `
  -Identity $newUser `
  -Add @{
    proxyAddresses = @(
      "SMTP:hogehoge@fuga.jp"    # SMTP : プライマリ SMTP アドレス
      "smtp:hogehoge.alias@fuga.jp" # smtp : 追加の SMTP アドレス
    )
  } `
  -Server "dc01.fuga.jp"
```

:::message

下表はこうした属性例をまとめたものとなります。

| LDAP属性名 | 用途 |
| --- | --- |
| `proxyAddresses` | SMTPアドレスなど |
| `msDS-HABSeniorityIndex` | 階層型アドレス帳の並び順 |
| `extensionAttribute1`（〜`15`） | Exchangeなどの拡張属性 |
| `thumbnailPhoto` | ユーザー写真 |
| `info` | ADUCの「メモ」欄 |
| `otherTelephone` | その他の電話番号 |
| `otherMailbox` | その他のメールボックス情報 |
| `url` | 関連URL |

:::

### LDAP を用いた属性値の変更

専用パラメータが用意されていない属性に関しては、LDAP 名と更新用パラメータ（`-Add`、`-Replace`、`-Remove`、`-Clear`）を用いて操作することになります。

- `-Add`：値を追加する
- `-Replace`：現在値を置き換える
- `-Remove`：特定の値だけを削除する
- `-Clear`：属性の値をすべて削除する

#### `-Add`

既存の値を残し、新しい値を追加します。

```ps1
Set-ADUser `
  -Identity $newUser `
  # 既存値を残して追加
  -Add @{
    proxyAddresses = "smtp:new@fuga.jp"
  } `
  -Server "dc01.fuga.jp"
```

既に同じ値が存在する場合は、追加に失敗する可能性があります。

#### `-Replace`

属性の現在値を、指定した値へ置き換えます。

```ps1
Set-ADUser `
  -Identity $newUser `
  # 属性全体を指定内容で置き換え
  -Replace @{
    proxyAddresses = "smtp:replaced@fuga.jp"
  }`
  -Server "dc01.fuga.jp"
```

:::message alert

複数値を管理する属性に `-Replace` を使用すると、**既存値をすべて指定内容に置き換える**ため注意が必要です。

例えば上記のコードにおいて、仮に設定時が `@("SMTP:hogehoge@fuga.jp", "smtp:hogehoge.alias@fuga.jp")` だった場合、Replace の時点で `smtp` だけでなく `SMTP` の内容も失われることになります。

:::

#### `-Remove`

複数値属性などから、指定した値だけを削除します。

```ps1
Set-ADUser `
  -Identity $newUser `
  # 指定した値だけ削除
  -Remove @{
    proxyAddresses = "smtp:old@fuga.jp"
  } `
  -Server "dc01.fuga.jp"
```

#### `-Clear`

属性内の値をすべて削除します。

```ps1
Set-ADUser `
  -Identity $newUser `
  # 属性の値をすべて削除
  -Clear "proxyAddresses" `
  -Server "dc01.fuga.jp"
```

:::message

`-Clear` は指定した属性の値をすべて消去するため注意が必要です。

対象ユーザーと属性名を確認し、必要に応じて現在値を記録してから実行するべきです。

:::

## `Set-ADAccountControl`：アカウント制御を設定する

https://learn.microsoft.com/ja-jp/powershell/module/activedirectory/set-adaccountcontrol

AD 上のユーザーアカウント制御（ *UAC : User Account Control* ）を編集するには、`Set-ADAccountControl` を使用します。

```ps1
# パスワード変更を許可し、有効期限を有限に設定
$paramSetADAccountControl = @{
  Identity              = $newUser
  CannotChangePassword  = $false
  PasswordNeverExpires  = $false
  Server                = "dc01.fuga.jp"
}
Set-ADAccountControl @paramSetADAccountControl
```

:::message

`Set-ADAccountControl` で設定できる下表の項目については、慎重に吟味したうえで扱うべきです。これらは認証やセキュリティへ影響する項目となります。

| パラメータ名 | 設定項目 |
| --- | --- |
| `-AllowReversiblePasswordEncryption` | 可逆暗号化を使用したパスワード保存 |
| `-DoesNotRequirePreAuth` | Kerberos 事前認証を不要にする設定 |
| `-UseDESKeyOnly` | DES 暗号化のみを使用する設定 |
| `-PasswordNeverExpires` | パスワードを無期限にする設定 |
| `-AccountNotDelegated` | アカウントが委任されないよう保護 |
| `-TrustedForDelegation` | 委任に対して信頼 |
| `-TrustedToAuthForDelegation` | 委任のための認証を行うアカウントとして信頼 |

:::

## ユーザーをグループへ追加する

### `Get-ADGroup`：グループを取得する

https://learn.microsoft.com/ja-jp/powershell/module/activedirectory/get-adgroup

（１つ以上の）AD グループを取得する場合は `Get-ADGroup` を使用します。

```ps1
# 識別子を使って取得する
Get-ADGroup `
  -Identity "SG_HogeUsers" `
  -Server "dc01.fuga.jp"
<#
DistinguishedName : CN=SG_HogeUsers,OU=Groups,DC=fuga,DC=jp
GroupCategory     : Security
GroupScope        : Global
Name              : SG_HogeUsers
ObjectClass       : group
ObjectGUID        : 5680f342-0ab1-46f7-bf30-4b9e75450512
SamAccountName    : SG_HogeUsers
SID               : S-1-5-21-1671859770-434810290-2222081234-4018
#>

# 特定の OU 上にあるグループを取得する
Get-ADGroup `
  -Filter * `
  -SearchBase "OU=Groups,DC=fuga,DC=jp" `
  -Server "dc01.fuga.jp"
<#
DistinguishedName : CN=SA-2.8-distlist1,OU=Devices,OU=GOO,OU=Groups,DC=fuga,DC=jp
GroupCategory     : Security
GroupScope        : Global
Name              : SA-2.8-distlist1
ObjectClass       : group
ObjectGUID        : 2e63d939-4547-4e8e-8fd9-bdd88d3fd6f4
SamAccountName    : SA-2.8-distlist1
SID               : S-1-5-21-1671859770-434810290-2222081234-3593

DistinguishedName : CN=PO-bal-admingroup1,OU=Test,OU=OGC,OU=Groups,DC=fuga,DC=jp
GroupCategory     : Security
GroupScope        : Global
Name              : PO-bal-admingroup1
ObjectClass       : group
ObjectGUID        : 303e5cb7-85b5-4a5b-93dd-c9b37d3752f8
SamAccountName    : PO-bal-admingroup1
SID               : S-1-5-21-1671859770-434810290-2222081234-3601

...
#>
```

### グループへの追加

特定のユーザーに対し、グループを追加する処理は *視点に応じて* 2 つの方法が可能です。

- `Add-ADPrincipalGroupMembership`：1人のユーザーを識別子として、（1 つ以上の）グループを追加する
- `Add-ADGroupMember`：1つのグループを識別子として、（1 人以上の）ユーザーを追加する

#### `Add-ADPrincipalGroupMembership`

https://learn.microsoft.com/ja-jp/powershell/module/activedirectory/add-adprincipalgroupmembership

`Add-ADPrincipalGroupMembership` は、1人の**ユーザーに対し**（1つ以上の）グループを追加するコマンドレットです。

```ps1
# ユーザーを識別子に、グループを追加
$paramAddADPrincipalGroupMembership = @{
  Identity = $newUser
  MemberOf = @(
    "CN=SG_HogeUsers,OU=Groups,DC=fuga,DC=jp"
    "CN=SG_Microsoft365Users,OU=Groups,DC=fuga,DC=jp"
  )
  Server   = "dc01.fuga.jp"
}
Add-ADPrincipalGroupMembership @paramAddADPrincipalGroupMembership
```

#### `Add-ADGroupMember`

https://learn.microsoft.com/ja-jp/powershell/module/activedirectory/add-adgroupmember

`Add-ADGroupMember` は、1つの**グループに対し**（1人以上の）ユーザーを追加するコマンドレットです。

```ps1
Add-ADGroupMember `
  -Identity "SG_HogeUsers" `
  -Members $newUser `
  -Server "dc01.fuga.jp"
```

### グループ情報を操作

ここまでで、ユーザーをグループへ所属させることを学習しました。

ここでは、グループのメンバー確認やグループからの削除など、もう少し応用的なことを見ていきます。

:::message

ここでも `-Identity` の内容が、グループだったりユーザーだったりと視点が異なるので注意が必要です。

:::

#### 所属グループを確認する

https://learn.microsoft.com/ja-jp/powershell/module/activedirectory/get-adprincipalgroupmembership

`Get-ADPrincipalGroupMembership` は、特定のユーザーを識別子として、ユーザーが所属しているグループ情報を取得します。

```ps1
# ユーザーが所属しているグループ情報（の一部）を取得
Get-ADPrincipalGroupMembership `
  -Identity $newUser `
  -Server "dc01.fuga.jp" |
  Select-Object Name, SamAccountName, GroupScope, GroupCategory
<#
Name                     SamAccountName           GroupScope GroupCategory
----                     --------------           ---------- -------------
Domain Users             Domain Users             Global     Security
SG_HogeUsers             SG_HogeUsers             Global     Security
SG_Microsoft365Users     SG_Microsoft365Users     Global     Security
#>
```

#### グループのメンバーを確認する

https://learn.microsoft.com/ja-jp/powershell/module/activedirectory/get-adgroupmember

`Get-ADGroupMember` は、特定のグループを識別子として、所属しているユーザー情報を取得します。

```ps1
# グループに所属しているユーザー情報（の一部）を取得
Get-ADGroupMember `
  -Identity "SG_HogeUsers" `
  -Server "dc01.fuga.jp" |
  Select-Object Name, SamAccountName, ObjectClass
<#
Name                SamAccountName      ObjectClass
----                --------------      -----------
GERMAN_TRAVIS       GERMAN_TRAVIS       user
LIONEL_CHERRY       LIONEL_CHERRY       user
STEFANIE_PACE       STEFANIE_PACE       user
...
#>
```

#### グループから削除する

https://learn.microsoft.com/ja-jp/powershell/module/activedirectory/remove-adgroupmember

`Remove-ADGroupMember` は、特定のグループを識別子として、所属しているユーザーを削除することができます。

```ps1
Remove-ADGroupMember `
  -Identity "SG_HogeUsers" `
  -Members $newUser `
  -Server "dc01.fuga.jp"
```

:::message

削除の際には、本当に実行していいかの確認が求められます。

確認を省略したい場合、`-Confirm $false` を使う方法があります。

:::

## アカウントを有効化する

https://learn.microsoft.com/ja-jp/powershell/module/activedirectory/enable-adaccount

AD アカウントを有効にするには、`Enable-ADAccount` を使用します。

:::message

本項では パスワード、属性、アカウント制御、所属グループの設定など、ユーザーとして運用できる状態になってから、アカウントを有効化します。

なお、パスワードポリシーを満たす**パスワードが設定されていない場合**などは、有効化に失敗する可能性があります。

:::

```ps1
$paramEnableADAccount = @{
  Identity = $newUser
  Server   = "dc01.fuga.jp"
}
Enable-ADAccount @paramEnableADAccount
```

## 一連の完成コード

ここまでで説明した内容を、1つのまとまった形にしてみます。実際にユーザーを作るスクリプトを作る際は、この手順を *参考に* してみるのも良いでしょう。

:::message alert

架空環境を想定したサンプルですので、そのままコピペしても動作はしません。

また、教育上の理由から平文パスワードを含んでいますが、実務へは `SecureString` を用いてスクリプト上に掲載すべきでないことは あらためて明記しておきます。

:::

```ps1
Import-Module ActiveDirectory -ErrorAction Stop

# 基本情報
$domain = "fuga.jp"
$server = "dc01.fuga.jp"
$userOu = "OU=Users,DC=fuga,DC=jp"
$name = "hoge001"
$samAccountName = "hoge001"
$userPrincipalName = "${samAccountName}@${domain}"
$displayName = "山田 太郎"
$surname = "山田"
$givenName = "太郎"
$emailAddress = "hogehoge@fuga.jp"
$subAddress = 'hogehoge.alias@fuga.jp'
$title = "Member"
$employeeId = "00012345"
$groupDns = @(
    "CN=SG_HogeUsers,OU=Groups,DC=fuga,DC=jp"
    "CN=SG_Microsoft365Users,OU=Groups,DC=fuga,DC=jp"
)
$msDSHABSeniorityIndex = 10

# ユーザーの作成。Enabled を指定せず、無効状態で作成する
$paramNewADUser = @{
    Name              = $name
    SamAccountName    = $samAccountName
    UserPrincipalName = $userPrincipalName
    Server            = $server
    Path              = $userOu
    DisplayName       = $displayName
    Surname           = $surname
    GivenName         = $givenName
    EmailAddress      = $emailAddress
    Title             = $title
    PassThru          = $true
}
$newUser = New-ADUser @paramNewADUser

# パスワードの設定
$plainPassword = 'Pa$$w0rd-Example'
$securePassword = ConvertTo-SecureString `
    -String $plainPassword `
    -AsPlainText `
    -Force

$paramSetADAccountPassword = @{
    Identity    = $newUser
    NewPassword = $securePassword
    Reset       = $true
    Server      = $server
}
Set-ADAccountPassword @paramSetADAccountPassword

# 一般属性の設定
$paramSetADUser = @{
  Identity               = $newUser
  EmployeeID             = $employeeId
  ChangePasswordAtLogon  = $true
  SmartcardLogonRequired = $false
  Server                 = $server
}
Set-ADUser @$paramSetADUser

# LDAP 属性の設定
Set-ADUser `
    -Identity $newUser `
    -Replace @{ 'msDS-HABSeniorityIndex' = $msDSHABSeniorityIndex } `
    -Server $server

Set-ADUser `
    -Identity $newUser `
    -Add @{
        proxyAddresses = @(
            "SMTP:$emailAddress"   # プライマリ SMTP アドレス
            "smtp:$subAddress" # 追加の SMTP アドレス
        )
    } `
    -Server $server

# アカウント制御
$paramSetADAccountControl = @{
    Identity             = $newUser
    CannotChangePassword = $false
    PasswordNeverExpires = $false
    Server               = $server
}
Set-ADAccountControl @paramSetADAccountControl

# グループへの追加
$paramAddADPrincipalGroupMembership = @{
    Identity = $newUser
    MemberOf = $groupDns
    Server   = $server
}
Add-ADPrincipalGroupMembership @paramAddADPrincipalGroupMembership

# アカウントの有効化
Enable-ADAccount `
    -Identity $newUser `
    -Server $server

# 作成結果の確認
Get-ADUser `
    -Identity $newUser `
    -Properties EmailAddress, employeeID, 'msDS-HABSeniorityIndex', proxyAddresses, PasswordLastSet `
    -Server $server |
    Select-Object Name, SamAccountName, UserPrincipalName, Enabled, EmailAddress, employeeID, 'msDS-HABSeniorityIndex', proxyAddresses, PasswordLastSet, DistinguishedName |
    Format-List

Get-ADPrincipalGroupMembership `
    -Identity $newUser `
    -Server $server |
    Sort-Object Name |
    Select-Object Name, GroupScope, GroupCategory
```

## 備考

### エラーを考慮した実務向けの書き方

基本操作を理解した後は、処理失敗時に原因を把握できるよう、`try`、`catch`、`-ErrorAction Stop` を使用します。

```ps1
# 例：既に登録済みの可能性がある中でユーザーを作成
try {
  $newUser = New-ADUser @newUserParams -ErrorAction Stop
  Write-Host "ユーザーを作成しました。" -ForegroundColor Green
  Write-Host "DN: $($newUser.DistinguishedName)"
}
catch [Microsoft.ActiveDirectory.Management.ADIdentityAlreadyExistsException] {
  # ここに来た場合、アカウントが既に存在している
  Write-Error "同じ名前または識別情報を持つ AD オブジェクトが既に存在します。"
  throw
}
catch {
  Write-Error "ユーザーの作成に失敗しました。"
  Write-Error "エラー: $($_.Exception.Message)"
  throw
}
```

後続処理でも、必要に応じて工程ごとに例外を処理します。

#### 事前に重複を確認する

実務では、「無効化状態だけど、既にその SAM アカウント名は用いられていた」といった事象が多く発生したりします。

なので `New-ADUser` で登録する前に、`try` - `catch` ブロックで拾ったり、下記のような重複確認をしておいたほうが良いでしょう。

```ps1
$existingUser = Get-ADUser `
  -Filter { SamAccountName -eq "hoge001" } `
  -Server "dc01.fuga.jp" `
  -ErrorAction Stop

if ($null -ne $existingUser) {
  throw "SAM アカウント名 '$samAccountName' は既に使用されています。"
}
```

### 作成時にパスワード設定と有効化をまとめる方法

`New-ADUser` には `-AccountPassword` と `-Enabled` があります。

なので、本項の手順のように `Set-ADAccountPassword` で分けなくても、`New-ADUser` 単体でパスワードを含め、アカウントとして有効化することは可能です。

```ps1
$securePassword = Read-Host `
  -Prompt "初期パスワードを入力してください" `
  -AsSecureString

$paramNewADUser = @{
  Name                  = "hoge002"
  SamAccountName        = "hoge002"
  UserPrincipalName     = "hoge002@fuga.jp"
  Path                  = "OU=Users,DC=fuga,DC=jp"
  AccountPassword       = $securePassword
  Enabled               = $true # <- アカウントとして有効化
  ChangePasswordAtLogon = $true
  Server                = "dc01.fuga.jp"
}
New-ADUser @paramNewADUser
  
```

この方法だと短く、１回の処理にまとめて書けます。一方で、本項では次の理由から工程を分けています。

- 各操作の役割を学習しやすい
- どの工程で失敗したかを確認しやすい
- 属性やグループの設定が完了するまで無効状態を維持できる
- 未完成のアカウントが利用される危険を抑えられる
