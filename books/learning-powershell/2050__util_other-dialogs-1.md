---
title: "🐥 その他のダイアログ 1"
---

https://learn.microsoft.com/ja-jp/dotnet/api/system.windows.forms

ここでは前項で扱った `FolderBrowserDialog` 以外の、利用できそうなフォームについてまとめています。ここに記載しているコードは、基本的な形式でダイアログを表示し その中身を `Out-GridView` で表示するようにしております。

実務上でダイアログを使う場合には、（前項含め）下記が主になると考えています。

- `OpenFileDialog`：ファイル選択
- `SaveFileDialog`：保存先の選択
- `FolderBrowserDialog`：フォルダ選択
- `MessageBox`：（何かしらの）確認用
- `Form`：独自 UI 用

:::message

`ShowDialog()` 等でダイアログを開いた際、その返り値として `DialogResult` 型が返ってきます。下記はその一例となります。

```txt
   TypeName: System.Windows.Forms.DialogResult

Name              MemberType Definition
----              ---------- ----------
Equals            Method     static bool Equals(System.Object objA, System.Object objB)
Format            Method     static string Format(type enumType, System.Object value, string format)
GetName           Method     static string GetName(type enumType, System.Object value)
GetNames          Method     static string[] GetNames(type enumType)
GetUnderlyingType Method     static type GetUnderlyingType(type enumType)
GetValues         Method     static array GetValues(type enumType)
IsDefined         Method     static bool IsDefined(type enumType, System.Object value)
Parse             Method     static System.Object Parse(type enumType, string value), static System.Object Parse(type enumType, string value, bool ignoreCase)
ReferenceEquals   Method     static bool ReferenceEquals(System.Object objA, System.Object objB)
ToObject          Method     static System.Object ToObject(type enumType, System.Object value), static System.Object ToObject(type enumType, sbyte value), static System.Object ToObject(type enumTy...
TryParse          Method     static bool TryParse[TEnum](string value, [ref] TEnum result), static bool TryParse[TEnum](string value, bool ignoreCase, [ref] TEnum result)
Abort             Property   static System.Windows.Forms.DialogResult Abort {get;}
Cancel            Property   static System.Windows.Forms.DialogResult Cancel {get;}
Ignore            Property   static System.Windows.Forms.DialogResult Ignore {get;}
No                Property   static System.Windows.Forms.DialogResult No {get;}
None              Property   static System.Windows.Forms.DialogResult None {get;}
OK                Property   static System.Windows.Forms.DialogResult OK {get;}
Retry             Property   static System.Windows.Forms.DialogResult Retry {get;}
Yes               Property   static System.Windows.Forms.DialogResult Yes {get;}
```

:::

## サンプル：`MessageBox`

https://learn.microsoft.com/ja-jp/dotnet/api/system.windows.forms.messagebox

![MessageBox](/images/books/learning-powershell/dialogs-02.png)

VBA 等でもおなじみ メッセージや確認内容を表示するためのダイアログです。

OK、Yes/No などのボタンでユーザーの意思確認を行います。

:::message

他と異なり、インスタンス化せず 静的メソッド `[System.Windows.Forms.MessageBox]::Show(...)` で呼び出します。

:::

```ps1
Add-Type -AssemblyName System.Windows.Forms

# 引数は `テキスト, タイトル, MessageBoxButtons, MessageBoxIcon` となっている
$MessageBox = [System.Windows.Forms.MessageBox]::Show(
  'Message',
  'Title',
  [System.Windows.Forms.MessageBoxButtons]::YesNo,
  [System.Windows.Forms.MessageBoxIcon]::Information
)

Write-Host "DialogResult: $MessageBox"
# -> DialogResult: Yes

([System.Windows.Forms.MessageBox]) | Get-Member -Static -MemberType Method, Property |
  Out-GridView -Title "$(([System.Windows.Forms.MessageBox]).GetType().Name) Methods / Properties"
<#
   TypeName: System.Windows.Forms.MessageBox

Name            MemberType Definition
----            ---------- ----------
Equals          Method     static bool Equals(System.Object objA, System.Object objB)
ReferenceEquals Method     static bool ReferenceEquals(System.Object objA, System.Object objB)
Show            Method     static System.Windows.Forms.DialogResult Show(string text, string caption, System.Windows.Forms.MessageBoxButtons buttons, System.Windows.Forms.MessageBoxIcon icon, Syst...
#>
```

## サンプル：`ColorDialog`

https://learn.microsoft.com/ja-jp/dotnet/api/system.windows.forms.colordialog

![ColorDialog](/images/books/learning-powershell/dialogs-03.png)

ユーザーに色を選択してもらうためのダイアログです。

選択した色は画面表示やデザイン設定などに利用できます。

:::message

アセンブリ名 `System.Drawing` が追加で必要となります。

:::

```ps1
Add-Type -AssemblyName System.Windows.Forms
Add-Type -AssemblyName System.Drawing


$ColorDialog = New-Object System.Windows.Forms.ColorDialog
$ColorDialog.Color = [System.Drawing.Color]::SkyBlue
$result = $ColorDialog.ShowDialog()

Write-Host "DialogResult: $result"
# -> DialogResult: OK
Write-Host "Selected Color: $($ColorDialog.Color)"
# -> Selected Color: Color [A=255, R=0, G=255, B=128]

$ColorDialog | Get-Member -MemberType Method, Property |
    Out-GridView -Title "$($ColorDialog.GetType().Name) Methods / Properties"
<#
   TypeName: System.Windows.Forms.ColorDialog

Name                      MemberType Definition
----                      ---------- ----------
CreateObjRef              Method     System.Runtime.Remoting.ObjRef CreateObjRef(type requestedType)
Dispose                   Method     void Dispose(), void IDisposable.Dispose()
Equals                    Method     bool Equals(System.Object obj)
GetHashCode               Method     int GetHashCode()
GetLifetimeService        Method     System.Object GetLifetimeService()
GetType                   Method     type GetType()
InitializeLifetimeService Method     System.Object InitializeLifetimeService()
Reset                     Method     void Reset()
ShowDialog                Method     System.Windows.Forms.DialogResult ShowDialog(), System.Windows.Forms.DialogResult ShowDialog(System.Windows.Forms.IWin32Window owner)
ToString                  Method     string ToString()
AllowFullOpen             Property   bool AllowFullOpen {get;set;}
AnyColor                  Property   bool AnyColor {get;set;}
Color                     Property   System.Drawing.Color Color {get;set;}
Container                 Property   System.ComponentModel.IContainer Container {get;}
CustomColors              Property   int[] CustomColors {get;set;}
FullOpen                  Property   bool FullOpen {get;set;}
ShowHelp                  Property   bool ShowHelp {get;set;}
Site                      Property   System.ComponentModel.ISite Site {get;set;}
SolidColorOnly            Property   bool SolidColorOnly {get;set;}
Tag                       Property   System.Object Tag {get;set;}
#>

$ColorDialog.Dispose()
```

## サンプル：`OpenFileDialog`

https://learn.microsoft.com/ja-jp/dotnet/api/system.windows.forms.openfiledialog

![OpenFileDialog](/images/books/learning-powershell/dialogs-04.png)

既存のファイルを選択するためのダイアログです。

ユーザーが指定したファイルパスを取得できます。

```ps1
Add-Type -AssemblyName System.Windows.Forms


$OpenFileDialog = New-Object System.Windows.Forms.OpenFileDialog
$OpenFileDialog.Title = 'Select a file'
$OpenFileDialog.InitialDirectory = [Environment]::GetFolderPath('Desktop')
$OpenFileDialog.Filter = 'All files (*.*)|*.*|Text files (*.txt)|*.txt'
$OpenFileDialog.Multiselect = $false
$OpenFileDialog.RestoreDirectory = $true

$result = $OpenFileDialog.ShowDialog()
Write-Host "DialogResult: $result"
# -> DialogResult: OK
Write-Host "Selected File: $($OpenFileDialog.FileName)"
# -> Selected File: C:\audio.log

$OpenFileDialog | Get-Member -MemberType Method, Property |
    Out-GridView -Title "$($OpenFileDialog.GetType().Name) Methods / Properties"
<#
   TypeName: System.Windows.Forms.OpenFileDialog

Name                         MemberType Definition
----                         ---------- ----------
CreateObjRef                 Method     System.Runtime.Remoting.ObjRef CreateObjRef(type requestedType)
Dispose                      Method     void Dispose(), void IDisposable.Dispose()
Equals                       Method     bool Equals(System.Object obj)
GetHashCode                  Method     int GetHashCode()
GetLifetimeService           Method     System.Object GetLifetimeService()
GetType                      Method     type GetType()
InitializeLifetimeService    Method     System.Object InitializeLifetimeService()
OpenFile                     Method     System.IO.Stream OpenFile()
Reset                        Method     void Reset()
ShowDialog                   Method     System.Windows.Forms.DialogResult ShowDialog(), System.Windows.Forms.DialogResult ShowDialog(System.Windows.Forms.IWin32Window owner)
ToString                     Method     string ToString()
AddExtension                 Property   bool AddExtension {get;set;}
AutoUpgradeEnabled           Property   bool AutoUpgradeEnabled {get;set;}
CheckFileExists              Property   bool CheckFileExists {get;set;}
CheckPathExists              Property   bool CheckPathExists {get;set;}
Container                    Property   System.ComponentModel.IContainer Container {get;}
CustomPlaces                 Property   System.Windows.Forms.FileDialogCustomPlacesCollection CustomPlaces {get;}
DefaultExt                   Property   string DefaultExt {get;set;}
DereferenceLinks             Property   bool DereferenceLinks {get;set;}
FileName                     Property   string FileName {get;set;}
FileNames                    Property   string[] FileNames {get;}
Filter                       Property   string Filter {get;set;}
FilterIndex                  Property   int FilterIndex {get;set;}
InitialDirectory             Property   string InitialDirectory {get;set;}
Multiselect                  Property   bool Multiselect {get;set;}
ReadOnlyChecked              Property   bool ReadOnlyChecked {get;set;}
RestoreDirectory             Property   bool RestoreDirectory {get;set;}
SafeFileName                 Property   string SafeFileName {get;}
SafeFileNames                Property   string[] SafeFileNames {get;}
ShowHelp                     Property   bool ShowHelp {get;set;}
ShowReadOnly                 Property   bool ShowReadOnly {get;set;}
Site                         Property   System.ComponentModel.ISite Site {get;set;}
SupportMultiDottedExtensions Property   bool SupportMultiDottedExtensions {get;set;}
Tag                          Property   System.Object Tag {get;set;}
Title                        Property   string Title {get;set;}
ValidateNames                Property   bool ValidateNames {get;set;}
#>

$OpenFileDialog.Dispose()
```

## サンプル：`FontDialog`

https://learn.microsoft.com/ja-jp/dotnet/api/system.windows.forms.fontdialog

![FontDialog](/images/books/learning-powershell/dialogs-05.png)

フォントの種類やサイズ、色などを選択するためのダイアログです。

文字表示のカスタマイズに利用します。

:::message

アセンブリ名 `System.Drawing` が追加で必要となります。

:::

```ps1
Add-Type -AssemblyName System.Windows.Forms
Add-Type -AssemblyName System.Drawing


$FontDialog = New-Object System.Windows.Forms.FontDialog
$FontDialog.ShowColor = $true
$FontDialog.ShowEffects = $true
$result = $FontDialog.ShowDialog()

Write-Host "DialogResult: $result"
# -> DialogResult: OK
Write-Host "Selected Font: $($FontDialog.Font)"
# -> Selected Font: [Font: Name=MS UI Gothic, Size=9, Units=3, GdiCharSet=128, GdiVerticalFont=False]
Write-Host "Selected Color: $($FontDialog.Color)"
# -> Selected Color: Color [Black]

$FontDialog | Get-Member -MemberType Method, Property |
    Out-GridView -Title "$($FontDialog.GetType().Name) Methods / Properties"
<#
   TypeName: System.Windows.Forms.FontDialog

Name                      MemberType Definition
----                      ---------- ----------
CreateObjRef              Method     System.Runtime.Remoting.ObjRef CreateObjRef(type requestedType)
Dispose                   Method     void Dispose(), void IDisposable.Dispose()
Equals                    Method     bool Equals(System.Object obj)
GetHashCode               Method     int GetHashCode()
GetLifetimeService        Method     System.Object GetLifetimeService()
GetType                   Method     type GetType()
InitializeLifetimeService Method     System.Object InitializeLifetimeService()
Reset                     Method     void Reset()
ShowDialog                Method     System.Windows.Forms.DialogResult ShowDialog(), System.Windows.Forms.DialogResult ShowDialog(System.Windows.Forms.IWin32Window owner)
ToString                  Method     string ToString()
AllowScriptChange         Property   bool AllowScriptChange {get;set;}
AllowSimulations          Property   bool AllowSimulations {get;set;}
AllowVectorFonts          Property   bool AllowVectorFonts {get;set;}
AllowVerticalFonts        Property   bool AllowVerticalFonts {get;set;}
Color                     Property   System.Drawing.Color Color {get;set;}
Container                 Property   System.ComponentModel.IContainer Container {get;}
FixedPitchOnly            Property   bool FixedPitchOnly {get;set;}
Font                      Property   System.Drawing.Font Font {get;set;}
FontMustExist             Property   bool FontMustExist {get;set;}
MaxSize                   Property   int MaxSize {get;set;}
MinSize                   Property   int MinSize {get;set;}
ScriptsOnly               Property   bool ScriptsOnly {get;set;}
ShowApply                 Property   bool ShowApply {get;set;}
ShowColor                 Property   bool ShowColor {get;set;}
ShowEffects               Property   bool ShowEffects {get;set;}
ShowHelp                  Property   bool ShowHelp {get;set;}
Site                      Property   System.ComponentModel.ISite Site {get;set;}
Tag                       Property   System.Object Tag {get;set;}
#>

$FontDialog.Dispose()
```

## サンプル：`SaveFileDialog`

https://learn.microsoft.com/ja-jp/dotnet/api/system.windows.forms.savefiledialog

![SaveFileDialog](/images/books/learning-powershell/dialogs-06.png)

ファイルの保存先やファイル名を指定するためのダイアログです。

上書き確認や拡張子の自動付与も行えます。

```ps1
Add-Type -AssemblyName System.Windows.Forms


$SaveFileDialog = New-Object System.Windows.Forms.SaveFileDialog
$SaveFileDialog.Title = 'Save a file'
$SaveFileDialog.InitialDirectory = [Environment]::GetFolderPath('Desktop')
$SaveFileDialog.Filter = 'Text files (*.txt)|*.txt|All files (*.*)|*.*'
$SaveFileDialog.DefaultExt = 'txt'
$SaveFileDialog.AddExtension = $true
$SaveFileDialog.OverwritePrompt = $true

$result = $SaveFileDialog.ShowDialog()
Write-Host "DialogResult: $result"
# -> DialogResult: OK
Write-Host "Selected File: $($SaveFileDialog.FileName)"
# -> Selected File: C:\hoge.txt

$SaveFileDialog | Get-Member -MemberType Method, Property |
    Out-GridView -Title "$($SaveFileDialog.GetType().Name) Methods / Properties"
<#
   TypeName: System.Windows.Forms.SaveFileDialog

Name                         MemberType Definition
----                         ---------- ----------
CreateObjRef                 Method     System.Runtime.Remoting.ObjRef CreateObjRef(type requestedType)
Dispose                      Method     void Dispose(), void IDisposable.Dispose()
Equals                       Method     bool Equals(System.Object obj)
GetHashCode                  Method     int GetHashCode()
GetLifetimeService           Method     System.Object GetLifetimeService()
GetType                      Method     type GetType()
InitializeLifetimeService    Method     System.Object InitializeLifetimeService()
OpenFile                     Method     System.IO.Stream OpenFile()
Reset                        Method     void Reset()
ShowDialog                   Method     System.Windows.Forms.DialogResult ShowDialog(), System.Windows.Forms.DialogResult ShowDialog(System.Windows.Forms.IWin32Window owner)
ToString                     Method     string ToString()
AddExtension                 Property   bool AddExtension {get;set;}
AutoUpgradeEnabled           Property   bool AutoUpgradeEnabled {get;set;}
CheckFileExists              Property   bool CheckFileExists {get;set;}
CheckPathExists              Property   bool CheckPathExists {get;set;}
Container                    Property   System.ComponentModel.IContainer Container {get;}
CreatePrompt                 Property   bool CreatePrompt {get;set;}
CustomPlaces                 Property   System.Windows.Forms.FileDialogCustomPlacesCollection CustomPlaces {get;}
DefaultExt                   Property   string DefaultExt {get;set;}
DereferenceLinks             Property   bool DereferenceLinks {get;set;}
FileName                     Property   string FileName {get;set;}
FileNames                    Property   string[] FileNames {get;}
Filter                       Property   string Filter {get;set;}
FilterIndex                  Property   int FilterIndex {get;set;}
InitialDirectory             Property   string InitialDirectory {get;set;}
OverwritePrompt              Property   bool OverwritePrompt {get;set;}
RestoreDirectory             Property   bool RestoreDirectory {get;set;}
ShowHelp                     Property   bool ShowHelp {get;set;}
Site                         Property   System.ComponentModel.ISite Site {get;set;}
SupportMultiDottedExtensions Property   bool SupportMultiDottedExtensions {get;set;}
Tag                          Property   System.Object Tag {get;set;}
Title                        Property   string Title {get;set;}
ValidateNames                Property   bool ValidateNames {get;set;}
#>

$SaveFileDialog.Dispose()
```

## サンプル：`Form`

https://learn.microsoft.com/ja-jp/dotnet/api/system.windows.forms.form

![Form](/images/books/learning-powershell/dialogs-10.png)

独自の入力画面や確認画面を作成するための基本ウィンドウです。

ラベルやボタンなどのコントロールを自由に配置できます。

:::message alert

**自由度が高い反面、コードを作成する難度は他と比べて頭 2 つ分くらい跳ね上がります**。

- フォーム自体の諸要素（サイズなど）を設定
- ラベルやテキストボックス、ボタンといった諸要素を設定
  - フォーム上のどの位置に、どれだけのサイズで描画するか
  - 要素別に必要となる中身の情報（テキスト情報、文字サイズ など）

そのため、使用は「**他のダイアログでは どうしても目的の動作が達成できない**」場合などに制限をかけておくべきです。

:::

:::message

アセンブリ名 `System.Drawing` が追加で必要となります。

:::

```ps1
Add-Type -AssemblyName System.Windows.Forms
Add-Type -AssemblyName System.Drawing


$Form = New-Object System.Windows.Forms.Form
$Form.Text = 'Custom Form sample'
$Form.StartPosition = [System.Windows.Forms.FormStartPosition]::CenterScreen
$Form.FormBorderStyle = [System.Windows.Forms.FormBorderStyle]::FixedDialog
$Form.MaximizeBox = $false
$Form.MinimizeBox = $false
$Form.Width = 360
$Form.Height = 160

$Label = New-Object System.Windows.Forms.Label
$Label.Text = 'This is a custom Form.'
$Label.AutoSize = $true
$Label.Left = 20
$Label.Top = 20

$Button = New-Object System.Windows.Forms.Button
$Button.Text = 'OK'
$Button.Width = 80
$Button.Left = 130
$Button.Top = 70
$Button.DialogResult = [System.Windows.Forms.DialogResult]::OK

$Form.Controls.Add($Label)
$Form.Controls.Add($Button)
$Form.AcceptButton = $Button

$result = $Form.ShowDialog()
Write-Host "DialogResult: $result"
# -> DialogResult: OK

$Form | Get-Member -MemberType Method, Property |
    Out-GridView -Title "$($Form.GetType().Name) Methods / Properties"
<#
   TypeName: System.Windows.Forms.Form

Name                               MemberType Definition
----                               ---------- ----------
Activate                           Method     void Activate()
ActivateControl                    Method     bool IContainerControl.ActivateControl(System.Windows.Forms.Control active)
AddOwnedForm                       Method     void AddOwnedForm(System.Windows.Forms.Form ownedForm)
BeginInvoke                        Method     System.IAsyncResult BeginInvoke(System.Delegate method, Params System.Object[] args), System.IAsyncResult BeginInvoke(System.Delegate method), System.IAsyncResult ISynchronizeInvoke.BeginInvoke(System.Delegate method, System.Object[] args)
BringToFront                       Method     void BringToFront()
Close                              Method     void Close()
Contains                           Method     bool Contains(System.Windows.Forms.Control ctl)
CreateControl                      Method     void CreateControl()
CreateGraphics                     Method     System.Drawing.Graphics CreateGraphics()
CreateObjRef                       Method     System.Runtime.Remoting.ObjRef CreateObjRef(type requestedType)
Dispose                            Method     void Dispose(), void IDisposable.Dispose()
DoDragDrop                         Method     System.Windows.Forms.DragDropEffects DoDragDrop(System.Object data, System.Windows.Forms.DragDropEffects allowedEffects)
DrawToBitmap                       Method     void DrawToBitmap(System.Drawing.Bitmap bitmap, System.Drawing.Rectangle targetBounds)
EndInvoke                          Method     System.Object EndInvoke(System.IAsyncResult asyncResult), System.Object ISynchronizeInvoke.EndInvoke(System.IAsyncResult result)
Equals                             Method     bool Equals(System.Object obj)
FindForm                           Method     System.Windows.Forms.Form FindForm()
Focus                              Method     bool Focus()
GetChildAtPoint                    Method     System.Windows.Forms.Control GetChildAtPoint(System.Drawing.Point pt, System.Windows.Forms.GetChildAtPointSkip skipValue), System.Windows.Forms.Control GetChildAtPoint(System.Drawing.Point pt)
GetContainerControl                Method     System.Windows.Forms.IContainerControl GetContainerControl()
GetHashCode                        Method     int GetHashCode()
GetLifetimeService                 Method     System.Object GetLifetimeService()
GetNextControl                     Method     System.Windows.Forms.Control GetNextControl(System.Windows.Forms.Control ctl, bool forward)
GetPreferredSize                   Method     System.Drawing.Size GetPreferredSize(System.Drawing.Size proposedSize)
GetType                            Method     type GetType()
Hide                               Method     void Hide()
InitializeLifetimeService          Method     System.Object InitializeLifetimeService()
Invalidate                         Method     void Invalidate(System.Drawing.Region region), void Invalidate(System.Drawing.Region region, bool invalidateChildren), void Invalidate(), void Invalidate(bool invalidateChildren), void Invalidate(System.Drawing.Rectangle rc), void Invalidate(System.Drawing.Recta...
Invoke                             Method     System.Object Invoke(System.Delegate method, Params System.Object[] args), System.Object Invoke(System.Delegate method), System.Object ISynchronizeInvoke.Invoke(System.Delegate method, System.Object[] args)
LayoutMdi                          Method     void LayoutMdi(System.Windows.Forms.MdiLayout value)
LogicalToDeviceUnits               Method     int LogicalToDeviceUnits(int value), System.Drawing.Size LogicalToDeviceUnits(System.Drawing.Size value)
OnDragDrop                         Method     void IDropTarget.OnDragDrop(System.Windows.Forms.DragEventArgs e)
OnDragEnter                        Method     void IDropTarget.OnDragEnter(System.Windows.Forms.DragEventArgs e)
OnDragLeave                        Method     void IDropTarget.OnDragLeave(System.EventArgs e)
OnDragOver                         Method     void IDropTarget.OnDragOver(System.Windows.Forms.DragEventArgs e)
PerformAutoScale                   Method     void PerformAutoScale()
PerformLayout                      Method     void PerformLayout(), void PerformLayout(System.Windows.Forms.Control affectedControl, string affectedProperty)
PointToClient                      Method     System.Drawing.Point PointToClient(System.Drawing.Point p)
PointToScreen                      Method     System.Drawing.Point PointToScreen(System.Drawing.Point p)
PreProcessControlMessage           Method     System.Windows.Forms.PreProcessControlState PreProcessControlMessage([ref] System.Windows.Forms.Message msg)
PreProcessMessage                  Method     bool PreProcessMessage([ref] System.Windows.Forms.Message msg)
RectangleToClient                  Method     System.Drawing.Rectangle RectangleToClient(System.Drawing.Rectangle r)
RectangleToScreen                  Method     System.Drawing.Rectangle RectangleToScreen(System.Drawing.Rectangle r)
Refresh                            Method     void Refresh()
RemoveOwnedForm                    Method     void RemoveOwnedForm(System.Windows.Forms.Form ownedForm)
ResetBackColor                     Method     void ResetBackColor()
ResetBindings                      Method     void ResetBindings()
ResetCursor                        Method     void ResetCursor()
ResetFont                          Method     void ResetFont()
ResetForeColor                     Method     void ResetForeColor()
ResetImeMode                       Method     void ResetImeMode()
ResetRightToLeft                   Method     void ResetRightToLeft()
ResetText                          Method     void ResetText()
ResumeLayout                       Method     void ResumeLayout(), void ResumeLayout(bool performLayout)
Scale                              Method     void Scale(float ratio), void Scale(float dx, float dy), void Scale(System.Drawing.SizeF factor)
ScaleBitmapLogicalToDevice         Method     void ScaleBitmapLogicalToDevice([ref] System.Drawing.Bitmap logicalBitmap)
ScrollControlIntoView              Method     void ScrollControlIntoView(System.Windows.Forms.Control activeControl)
Select                             Method     void Select()
SelectNextControl                  Method     bool SelectNextControl(System.Windows.Forms.Control ctl, bool forward, bool tabStopOnly, bool nested, bool wrap)
SendToBack                         Method     void SendToBack()
SetAutoScrollMargin                Method     void SetAutoScrollMargin(int x, int y)
SetBounds                          Method     void SetBounds(int x, int y, int width, int height), void SetBounds(int x, int y, int width, int height, System.Windows.Forms.BoundsSpecified specified)
SetDesktopBounds                   Method     void SetDesktopBounds(int x, int y, int width, int height)
SetDesktopLocation                 Method     void SetDesktopLocation(int x, int y)
Show                               Method     void Show(System.Windows.Forms.IWin32Window owner), void Show()
ShowDialog                         Method     System.Windows.Forms.DialogResult ShowDialog(), System.Windows.Forms.DialogResult ShowDialog(System.Windows.Forms.IWin32Window owner)
SuspendLayout                      Method     void SuspendLayout()
ToString                           Method     string ToString()
Update                             Method     void Update()
Validate                           Method     bool Validate(), bool Validate(bool checkAutoValidate)
ValidateChildren                   Method     bool ValidateChildren(), bool ValidateChildren(System.Windows.Forms.ValidationConstraints validationConstraints)
AcceptButton                       Property   System.Windows.Forms.IButtonControl AcceptButton {get;set;}
AccessibilityObject                Property   System.Windows.Forms.AccessibleObject AccessibilityObject {get;}
AccessibleDefaultActionDescription Property   string AccessibleDefaultActionDescription {get;set;}
AccessibleDescription              Property   string AccessibleDescription {get;set;}
AccessibleName                     Property   string AccessibleName {get;set;}
AccessibleRole                     Property   System.Windows.Forms.AccessibleRole AccessibleRole {get;set;}
ActiveControl                      Property   System.Windows.Forms.Control ActiveControl {get;set;}
ActiveMdiChild                     Property   System.Windows.Forms.Form ActiveMdiChild {get;}
AllowDrop                          Property   bool AllowDrop {get;set;}
AllowTransparency                  Property   bool AllowTransparency {get;set;}
Anchor                             Property   System.Windows.Forms.AnchorStyles Anchor {get;set;}
AutoScale                          Property   bool AutoScale {get;set;}
AutoScaleBaseSize                  Property   System.Drawing.Size AutoScaleBaseSize {get;set;}
AutoScaleDimensions                Property   System.Drawing.SizeF AutoScaleDimensions {get;set;}
AutoScaleMode                      Property   System.Windows.Forms.AutoScaleMode AutoScaleMode {get;set;}
AutoScroll                         Property   bool AutoScroll {get;set;}
AutoScrollMargin                   Property   System.Drawing.Size AutoScrollMargin {get;set;}
AutoScrollMinSize                  Property   System.Drawing.Size AutoScrollMinSize {get;set;}
AutoScrollOffset                   Property   System.Drawing.Point AutoScrollOffset {get;set;}
AutoScrollPosition                 Property   System.Drawing.Point AutoScrollPosition {get;set;}
AutoSize                           Property   bool AutoSize {get;set;}
AutoSizeMode                       Property   System.Windows.Forms.AutoSizeMode AutoSizeMode {get;set;}
AutoValidate                       Property   System.Windows.Forms.AutoValidate AutoValidate {get;set;}
BackColor                          Property   System.Drawing.Color BackColor {get;set;}
BackgroundImage                    Property   System.Drawing.Image BackgroundImage {get;set;}
BackgroundImageLayout              Property   System.Windows.Forms.ImageLayout BackgroundImageLayout {get;set;}
BindingContext                     Property   System.Windows.Forms.BindingContext BindingContext {get;set;}
Bottom                             Property   int Bottom {get;}
Bounds                             Property   System.Drawing.Rectangle Bounds {get;set;}
CancelButton                       Property   System.Windows.Forms.IButtonControl CancelButton {get;set;}
CanFocus                           Property   bool CanFocus {get;}
CanSelect                          Property   bool CanSelect {get;}
Capture                            Property   bool Capture {get;set;}
CausesValidation                   Property   bool CausesValidation {get;set;}
ClientRectangle                    Property   System.Drawing.Rectangle ClientRectangle {get;}
ClientSize                         Property   System.Drawing.Size ClientSize {get;set;}
CompanyName                        Property   string CompanyName {get;}
Container                          Property   System.ComponentModel.IContainer Container {get;}
ContainsFocus                      Property   bool ContainsFocus {get;}
ContextMenu                        Property   System.Windows.Forms.ContextMenu ContextMenu {get;set;}
ContextMenuStrip                   Property   System.Windows.Forms.ContextMenuStrip ContextMenuStrip {get;set;}
ControlBox                         Property   bool ControlBox {get;set;}
Controls                           Property   System.Windows.Forms.Control+ControlCollection Controls {get;}
Created                            Property   bool Created {get;}
CurrentAutoScaleDimensions         Property   System.Drawing.SizeF CurrentAutoScaleDimensions {get;}
Cursor                             Property   System.Windows.Forms.Cursor Cursor {get;set;}
DataBindings                       Property   System.Windows.Forms.ControlBindingsCollection DataBindings {get;}
DesktopBounds                      Property   System.Drawing.Rectangle DesktopBounds {get;set;}
DesktopLocation                    Property   System.Drawing.Point DesktopLocation {get;set;}
DeviceDpi                          Property   int DeviceDpi {get;}
DialogResult                       Property   System.Windows.Forms.DialogResult DialogResult {get;set;}
DisplayRectangle                   Property   System.Drawing.Rectangle DisplayRectangle {get;}
Disposing                          Property   bool Disposing {get;}
Dock                               Property   System.Windows.Forms.DockStyle Dock {get;set;}
DockPadding                        Property   System.Windows.Forms.ScrollableControl+DockPaddingEdges DockPadding {get;}
Enabled                            Property   bool Enabled {get;set;}
Focused                            Property   bool Focused {get;}
Font                               Property   System.Drawing.Font Font {get;set;}
ForeColor                          Property   System.Drawing.Color ForeColor {get;set;}
FormBorderStyle                    Property   System.Windows.Forms.FormBorderStyle FormBorderStyle {get;set;}
Handle                             Property   System.IntPtr Handle {get;}
HasChildren                        Property   bool HasChildren {get;}
Height                             Property   int Height {get;set;}
HelpButton                         Property   bool HelpButton {get;set;}
HorizontalScroll                   Property   System.Windows.Forms.HScrollProperties HorizontalScroll {get;}
Icon                               Property   System.Drawing.Icon Icon {get;set;}
ImeMode                            Property   System.Windows.Forms.ImeMode ImeMode {get;set;}
InvokeRequired                     Property   bool InvokeRequired {get;}
IsAccessible                       Property   bool IsAccessible {get;set;}
IsDisposed                         Property   bool IsDisposed {get;}
IsHandleCreated                    Property   bool IsHandleCreated {get;}
IsMdiChild                         Property   bool IsMdiChild {get;}
IsMdiContainer                     Property   bool IsMdiContainer {get;set;}
IsMirrored                         Property   bool IsMirrored {get;}
IsRestrictedWindow                 Property   bool IsRestrictedWindow {get;}
KeyPreview                         Property   bool KeyPreview {get;set;}
LayoutEngine                       Property   System.Windows.Forms.Layout.LayoutEngine LayoutEngine {get;}
Left                               Property   int Left {get;set;}
Location                           Property   System.Drawing.Point Location {get;set;}
MainMenuStrip                      Property   System.Windows.Forms.MenuStrip MainMenuStrip {get;set;}
Margin                             Property   System.Windows.Forms.Padding Margin {get;set;}
MaximizeBox                        Property   bool MaximizeBox {get;set;}
MaximumSize                        Property   System.Drawing.Size MaximumSize {get;set;}
MdiChildren                        Property   System.Windows.Forms.Form[] MdiChildren {get;}
MdiParent                          Property   System.Windows.Forms.Form MdiParent {get;set;}
Menu                               Property   System.Windows.Forms.MainMenu Menu {get;set;}
MergedMenu                         Property   System.Windows.Forms.MainMenu MergedMenu {get;}
MinimizeBox                        Property   bool MinimizeBox {get;set;}
MinimumSize                        Property   System.Drawing.Size MinimumSize {get;set;}
Modal                              Property   bool Modal {get;}
Name                               Property   string Name {get;set;}
Opacity                            Property   double Opacity {get;set;}
OwnedForms                         Property   System.Windows.Forms.Form[] OwnedForms {get;}
Owner                              Property   System.Windows.Forms.Form Owner {get;set;}
Padding                            Property   System.Windows.Forms.Padding Padding {get;set;}
Parent                             Property   System.Windows.Forms.Control Parent {get;set;}
ParentForm                         Property   System.Windows.Forms.Form ParentForm {get;}
PreferredSize                      Property   System.Drawing.Size PreferredSize {get;}
ProductName                        Property   string ProductName {get;}
ProductVersion                     Property   string ProductVersion {get;}
RecreatingHandle                   Property   bool RecreatingHandle {get;}
Region                             Property   System.Drawing.Region Region {get;set;}
RestoreBounds                      Property   System.Drawing.Rectangle RestoreBounds {get;}
Right                              Property   int Right {get;}
RightToLeft                        Property   System.Windows.Forms.RightToLeft RightToLeft {get;set;}
RightToLeftLayout                  Property   bool RightToLeftLayout {get;set;}
ShowIcon                           Property   bool ShowIcon {get;set;}
ShowInTaskbar                      Property   bool ShowInTaskbar {get;set;}
Site                               Property   System.ComponentModel.ISite Site {get;set;}
Size                               Property   System.Drawing.Size Size {get;set;}
SizeGripStyle                      Property   System.Windows.Forms.SizeGripStyle SizeGripStyle {get;set;}
StartPosition                      Property   System.Windows.Forms.FormStartPosition StartPosition {get;set;}
TabIndex                           Property   int TabIndex {get;set;}
TabStop                            Property   bool TabStop {get;set;}
Tag                                Property   System.Object Tag {get;set;}
Text                               Property   string Text {get;set;}
Top                                Property   int Top {get;set;}
TopLevel                           Property   bool TopLevel {get;set;}
TopLevelControl                    Property   System.Windows.Forms.Control TopLevelControl {get;}
TopMost                            Property   bool TopMost {get;set;}
TransparencyKey                    Property   System.Drawing.Color TransparencyKey {get;set;}
UseWaitCursor                      Property   bool UseWaitCursor {get;set;}
VerticalScroll                     Property   System.Windows.Forms.VScrollProperties VerticalScroll {get;}
Visible                            Property   bool Visible {get;set;}
Width                              Property   int Width {get;set;}
WindowState                        Property   System.Windows.Forms.FormWindowState WindowState {get;set;}
WindowTarget                       Property   System.Windows.Forms.IWindowTarget WindowTarget {get;set;}
#>

$Form.Dispose()
```
