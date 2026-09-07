---
title: "🐥 その他のダイアログ 2"
---

ここでは、前項までのダイアログに比べ利用頻度が ほとんど無いようなものや、そもそも利用ができないものについて解説をします。

:::message alert

本項は参考用として設けられたものであり、よって**本項を閲覧する必要性は ほとんど無いことを最初に明記しておきます**。

:::

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

## 基本的に利用することがないダイアログ集

この段落に収録しているものは、印刷関係のものであり 基本的には利用することがないと考えられるものとなります。

:::message

なお、本段落でたびたび登場する `PrintDocument` クラスについては、[Microsoft Learn](https://learn.microsoft.com/ja-jp/dotnet/api/system.drawing.printing.printdocument)にある情報を参照ください。

:::

### サンプル：`PageSetupDialog`

https://learn.microsoft.com/ja-jp/dotnet/api/system.windows.forms.pagesetupdialog

![PageSetupDialog](/images/books/learning-powershell/dialogs-07.png)

印刷時の用紙サイズ、余白、印刷方向などを設定するためのダイアログです。

印刷前のページ設定に使用します。

:::message

アセンブリ名 `System.Drawing` が追加で必要となります。

:::

```ps1
Add-Type -AssemblyName System.Windows.Forms
Add-Type -AssemblyName System.Drawing


$PrintDocument = New-Object System.Drawing.Printing.PrintDocument

$PageSetupDialog = New-Object System.Windows.Forms.PageSetupDialog
$PageSetupDialog.Document = $PrintDocument
$PageSetupDialog.AllowMargins = $true
$PageSetupDialog.AllowOrientation = $true
$PageSetupDialog.AllowPaper = $true
$PageSetupDialog.AllowPrinter = $true

$result = $PageSetupDialog.ShowDialog()
Write-Host "DialogResult: $result"
# -> DialogResult: OK

$PageSetupDialog | Get-Member -MemberType Method, Property |
    Out-GridView -Title "$($PageSetupDialog.GetType().Name) Methods / Properties"
<#
   TypeName: System.Windows.Forms.PageSetupDialog

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
AllowMargins              Property   bool AllowMargins {get;set;}
AllowOrientation          Property   bool AllowOrientation {get;set;}
AllowPaper                Property   bool AllowPaper {get;set;}
AllowPrinter              Property   bool AllowPrinter {get;set;}
Container                 Property   System.ComponentModel.IContainer Container {get;}
Document                  Property   System.Drawing.Printing.PrintDocument Document {get;set;}
EnableMetric              Property   bool EnableMetric {get;set;}
MinMargins                Property   System.Drawing.Printing.Margins MinMargins {get;set;}
PageSettings              Property   System.Drawing.Printing.PageSettings PageSettings {get;set;}
PrinterSettings           Property   System.Drawing.Printing.PrinterSettings PrinterSettings {get;set;}
ShowHelp                  Property   bool ShowHelp {get;set;}
ShowNetwork               Property   bool ShowNetwork {get;set;}
Site                      Property   System.ComponentModel.ISite Site {get;set;}
Tag                       Property   System.Object Tag {get;set;}
#>

$PageSetupDialog.Dispose()
$PrintDocument.Dispose()
```

### サンプル：`PrintDialog`

https://learn.microsoft.com/ja-jp/dotnet/api/system.windows.forms.printdialog

![PrintDialog](/images/books/learning-powershell/dialogs-08.png)

プリンターや印刷範囲などの印刷設定を選択するためのダイアログです。

実際の印刷実行前に利用します。

:::message

アセンブリ名 `System.Drawing` が追加で必要となります。

:::

```ps1
Add-Type -AssemblyName System.Windows.Forms
Add-Type -AssemblyName System.Drawing


$PrintDocument = New-Object System.Drawing.Printing.PrintDocument

$PrintDialog = New-Object System.Windows.Forms.PrintDialog
$PrintDialog.Document = $PrintDocument
$PrintDialog.AllowSomePages = $true
$PrintDialog.AllowSelection = $false
$PrintDialog.UseEXDialog = $true

$result = $PrintDialog.ShowDialog()
Write-Host "DialogResult: $result"
# -> DialogResult: OK

$PrintDialog | Get-Member -MemberType Method, Property |
    Out-GridView -Title "$($PrintDialog.GetType().Name) Methods / Properties"
<#
   TypeName: System.Windows.Forms.PrintDialog

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
AllowCurrentPage          Property   bool AllowCurrentPage {get;set;}
AllowPrintToFile          Property   bool AllowPrintToFile {get;set;}
AllowSelection            Property   bool AllowSelection {get;set;}
AllowSomePages            Property   bool AllowSomePages {get;set;}
Container                 Property   System.ComponentModel.IContainer Container {get;}
Document                  Property   System.Drawing.Printing.PrintDocument Document {get;set;}
PrinterSettings           Property   System.Drawing.Printing.PrinterSettings PrinterSettings {get;set;}
PrintToFile               Property   bool PrintToFile {get;set;}
ShowHelp                  Property   bool ShowHelp {get;set;}
ShowNetwork               Property   bool ShowNetwork {get;set;}
Site                      Property   System.ComponentModel.ISite Site {get;set;}
Tag                       Property   System.Object Tag {get;set;}
UseEXDialog               Property   bool UseEXDialog {get;set;}
#>

$PrintDialog.Dispose()
$PrintDocument.Dispose()
```

### サンプル：`PrintPreviewDialog`

https://learn.microsoft.com/ja-jp/dotnet/api/system.windows.forms.printpreviewdialog

![PrintPreviewDialog](/images/books/learning-powershell/dialogs-09.png)

印刷結果を事前に確認するためのプレビュー画面です。

紙に出力する前にレイアウトや内容を確認できます。

:::message

アセンブリ名 `System.Drawing` が追加で必要となります。

:::

```ps1
Add-Type -AssemblyName System.Windows.Forms
Add-Type -AssemblyName System.Drawing


$PrintDocument = New-Object System.Drawing.Printing.PrintDocument
$PrintDocument.DocumentName = 'Preview sample'
$PrintDocument.add_PrintPage({
    param($sender, $e)

    $font = New-Object System.Drawing.Font('MS Gothic', 14)
    $brush = [System.Drawing.Brushes]::Black
    $e.Graphics.DrawString('PrintPreviewDialog sample', $font, $brush, 100, 100)
    $font.Dispose()
    $e.HasMorePages = $false
})

$PrintPreviewDialog = New-Object System.Windows.Forms.PrintPreviewDialog
$PrintPreviewDialog.Document = $PrintDocument
$PrintPreviewDialog.Text = 'Print preview sample'
$PrintPreviewDialog.Width = 800
$PrintPreviewDialog.Height = 600

$result = $PrintPreviewDialog.ShowDialog()
Write-Host "DialogResult: $result"
# -> DialogResult: Cancel

$PrintPreviewDialog| Get-Member -MemberType Method, Property |
    Out-GridView -Title "$($PrintPreviewDialog.GetType().Name) Methods / Properties"
<#
   TypeName: System.Windows.Forms.PrintPreviewDialog

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
Document                           Property   System.Drawing.Printing.PrintDocument Document {get;set;}
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
PrintPreviewControl                Property   System.Windows.Forms.PrintPreviewControl PrintPreviewControl {get;}
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
UseAntiAlias                       Property   bool UseAntiAlias {get;set;}
UseWaitCursor                      Property   bool UseWaitCursor {get;set;}
VerticalScroll                     Property   System.Windows.Forms.VScrollProperties VerticalScroll {get;}
Visible                            Property   bool Visible {get;set;}
Width                              Property   int Width {get;set;}
WindowState                        Property   System.Windows.Forms.FormWindowState WindowState {get;set;}
WindowTarget                       Property   System.Windows.Forms.IWindowTarget WindowTarget {get;set;}
#>

$PrintPreviewDialog.Dispose()
$PrintDocument.Dispose()
```

## 利用できないダイアログについて

下表のフォームについては、`System.Windows.Forms` 名前空間上にあるものですが 実際には利用できないもの（または非推奨）となります。

| 対象 | 説明 |
| --- | --- | --- |
| `CommonDialog` | 抽象基底クラスであり、`New-Object System.Windows.Forms.CommonDialog` のように直接インスタンス化して表示する対象ではありません。 |
| `ThreadExceptionDialog` | この API は製品インフラストラクチャ用で、コードから直接使うものではありません。更に 既定コンストラクターがなく、`Exception` を渡すコンストラクターを使う必要があります。 |
