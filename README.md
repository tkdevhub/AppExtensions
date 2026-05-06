# AppExtensions

AppExtensions は、.NET アプリケーション向けの再利用可能な拡張メソッドおよび共通ヘルパーライブラリです。

業務アプリケーションでよく利用する DataTable、DataRow、DataGridView、CSV、Excel、文字列、日付、Enum、型変換などの共通処理をまとめて管理することを目的としています。

## 目的

このライブラリの目的は、複数のアプリケーションで利用する拡張メソッドや共通ユーティリティを一元管理し、実装の重複を減らすことです。

主な用途は以下のとおりです。

- DataTable 拡張
- DataRow 拡張
- DataGridView 拡張
- String 拡張
- DateTime 拡張
- Enum 拡張
- CSV 読み書き
- Excel インポート、エクスポート
- Nothing / DBNull の共通処理
- 型変換ヘルパー
- 表示用フォーマット処理

## 対象フレームワーク

DataGridView など WinForms 関連の拡張を含める場合：

```xml
<TargetFramework>.NET Framework 4.8.1</TargetFramework>
<UseWindowsForms>true</UseWindowsForms>
```

汎用的な拡張メソッドのみを扱う場合：

```xml
<TargetFramework>.NET Framework 4.8.1</TargetFramework>
```

初期段階では、以下を推奨します。

```xml
<TargetFramework>.NET Framework 4.8.1</TargetFramework>
<UseWindowsForms>true</UseWindowsForms>
```

これにより、汎用拡張と WinForms 関連拡張の両方を含めることができます。

## 推奨プロジェクト構成

```text
AppExtensions
├─ Data
│  ├─ DataTableExtensions.vb
│  ├─ DataRowExtensions.vb
│  └─ DataColumnExtensions.vb
│
├─ WinForms
│  ├─ DataGridViewExtensions.vb
│  ├─ ControlExtensions.vb
│  └─ BindingSourceExtensions.vb
│
├─ Text
│  ├─ StringExtensions.vb
│  └─ StringBuilderExtensions.vb
│
├─ Date
│  └─ DateTimeExtensions.vb
│
├─ Enum
│  └─ EnumExtensions.vb
│
├─ Files
│  ├─ CsvExtensions.vb
│  └─ ExcelExtensions.vb
│
├─ Conversion
│  └─ ConvertExtensions.vb
│
└─ README.md
```

## 命名方針

拡張メソッド名は、短く、分かりやすく、動作が予測しやすい名前にします。

良い例：

```vbnet
ToInt32OrDefault()
ToDecimalOrDefault()
ToDateOrNothing()
IsNullOrEmptyValue()
GetString()
GetInteger()
WriteCsv()
LoadCsv()
```

避けたい例：

```vbnet
ConvertValue()
DoWork()
GetData()
Process()
```

## 設計方針

このライブラリは以下の方針で設計します。

1. 拡張メソッドは小さく、目的を明確にする
2. 重要なエラーを暗黙的に握りつぶさない
3. `Nothing` と `DBNull.Value` を一貫した方針で扱う
4. 汎用拡張メソッドに業務ルールを入れない
5. UI 関連拡張とデータ関連拡張を分けて管理する
6. メソッド名から動作が分かるようにする
7. 業務アプリケーションの一般的なデータ処理で安全に使えること

## 基本的な使い方

開発中はプロジェクト参照で利用します。

```xml
<ProjectReference Include="..\libs\AppExtensions\AppExtensions.vbproj" />
```

利用側で名前空間をインポートします。

```vbnet
Imports AppExtensions
```

使用例：

```vbnet
Dim value As Integer = row.GetInteger("AMOUNT")
```

```vbnet
Dim text As String = row.GetString("USER_NAME")
```

```vbnet
dt.WriteCsv("C:\Temp\output.csv")
```

## DataTable 拡張の例

使用例：

```vbnet
Dim dt As DataTable = LoadUserData()

If dt.HasRows() Then
    dt.WriteCsv("C:\Temp\users.csv")
End If
```

拡張メソッド例：

```vbnet
<Extension>
Public Function HasRows(table As DataTable) As Boolean
    Return table IsNot Nothing AndAlso table.Rows.Count > 0
End Function
```

## DataRow 拡張の例

使用例：

```vbnet
Dim userName As String = row.GetString("USER_NAME")
Dim age As Integer = row.GetInteger("AGE")
Dim amount As Decimal = row.GetDecimal("AMOUNT")
```

推奨される動作方針：

- `DBNull.Value` を安全に扱う
- 存在しない列は明確な例外にする、または Safe 系メソッドで処理する
- 数値変換は明示的かつ予測しやすくする

命名の分け方：

```vbnet
GetString("USER_NAME")              ' 列が存在しない場合は例外
GetStringOrDefault("USER_NAME")     ' 無効な場合は既定値を返す
```

## DataGridView 拡張の例

使用例：

```vbnet
dgv.ApplyStandardStyle()
dgv.ClearSelectionSafe()
dgv.SetDoubleBuffered(True)
```

DataGridView 関連の拡張は `WinForms` フォルダに配置します。

## CSV 拡張の例

使用例：

```vbnet
Dim dt As DataTable = DataTableExtensions.LoadCsv("C:\Temp\input.csv")
dt.WriteCsv("C:\Temp\output.csv")
```

CSV 関連メソッドでは、以下の仕様を明確にします。

- 文字コード
- ヘッダー行の有無
- 区切り文字
- ダブルクォーテーションの扱い
- 空値の扱い
- 改行の扱い

## エラーハンドリング方針

拡張メソッドは、以下のどちらの方針かを明確にします。

### Strict 系メソッド

処理に失敗した場合は例外を発生させます。

例：

```vbnet
GetInteger("AMOUNT")
```

### Safe 系メソッド

処理に失敗した場合は既定値を返します。

例：

```vbnet
GetIntegerOrDefault("AMOUNT", 0)
```

Strict 系と Safe 系の動作を、同じメソッド名の中で混在させないようにします。

## NuGet パッケージ化

ライブラリが安定した段階で NuGet パッケージとして作成できます。

```bash
dotnet pack -c Release
```

開発中は `ProjectReference` による参照を推奨します。

安定版として利用する場合は、NuGet パッケージ参照に切り替えることもできます。

## バージョン管理方針

このプロジェクトはセマンティックバージョニングを採用します。

```text
Major.Minor.Patch
```

例：

```text
1.0.0
1.1.0
1.1.1
2.0.0
```

バージョン更新の目安：

- Patch：不具合修正
- Minor：新しい拡張メソッド追加、互換性のある改善
- Major：メソッド名、引数、戻り値、動作の互換性がない変更

## 将来的な分割方針

ライブラリが大きくなった場合は、用途別に分割することを検討します。

例：

```text
AppExtensions.Core
AppExtensions.Data
AppExtensions.WinForms
AppExtensions.Excel
```

分割の目安：

- Core：String、DateTime、Enum、型変換
- Data：DataTable、DataRow、データベース関連ヘルパー
- WinForms：DataGridView、Control、BindingSource
- Excel：Excel インポート、エクスポート

初期段階では、`AppExtensions` にまとめて管理する方がシンプルです。

## リポジトリ利用方法

このリポジトリは、以下のいずれかの方法で利用することを想定しています。

1. アプリケーションソリューション内の Git Submodule
2. 開発中のプロジェクト参照
3. 安定後の NuGet パッケージ参照

推奨される開発時の構成：

```text
MainApp
├─ MainApp.sln
└─ src
   ├─ MainApp
   └─ libs
      └─ AppExtensions
```

## ライセンス

このプロジェクトは現在、個人利用または社内利用を想定しています。

公開リポジトリとして運用する場合は、このセクションを更新してください。