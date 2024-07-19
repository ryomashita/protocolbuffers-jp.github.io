+++
title = "スタイルガイド"
weight = 50
description = "`.proto` 定義を最適に構造化する方法に関する指針を提供します。"
type = "docs"
+++

このドキュメントは `.proto` ファイルのスタイルガイドを提供します。これらの規則に従うことで、Protocol Buffers メッセージ定義とそれに対応するクラスを一貫性があり読みやすくします。

Protocol Buffers のスタイルは時間とともに進化してきたため、異なる規則やスタイルで書かれた `.proto` ファイルを見ることがあるかもしれません。これらのファイルを変更する際には、**既存のスタイルを尊重**してください。**一貫性が重要**です。ただし、新しい `.proto` ファイルを作成する際には、現在のベストスタイルを採用するのが最善です。

## 標準ファイルフォーマット {#standard-file-formatting}

*   行の長さを 80 文字以下に保つこと。
*   インデントにはスペース 2 つを使用すること。
*   文字列にはダブルクォーテーションを使用することを好むこと。

## ファイル構造 {#file-structure}

ファイルは `lower_snake_case.proto` という名前にする必要があります。

すべてのファイルは以下の順序で並べる必要があります:

1.  ライセンスヘッダー（該当する場合）
1.  ファイルの概要
1.  構文
1.  パッケージ
1.  インポート（ソート済み）
1.  ファイルオプション
1.  その他すべて

## パッケージ {#packages}

パッケージ名は小文字である必要があります。パッケージ名はプロジェクト名に基づいて一意の名前を持つべきであり、プロトコルバッファ型定義を含むファイルのパスに基づいている可能性があります。

## メッセージとフィールド名 {#message-field-names}

メッセージ名には PascalCase（最初の文字が大文字）を使用してください: `SongServerRequest`。略語は単語として大文字で表記することを好む: `GetDnsRequest` ではなく `GetDNSRequest`。フィールド名には lower_snake_case を使用し、oneof フィールドや拡張フィールド名も含まれます: `song_name`。

```proto
message SongServerRequest {
  optional string song_name = 1;
}
```

このフィールド名の命名規則を使用すると、次の 2 つのコードサンプルに示されているようなアクセサが得られます。

C++:

```cpp
const string& song_name() { ... }
void set_song_name(const string& x) { ... }
```

Java:

```java
public String getSongName() { ... }
public Builder setSongName(String v) { ... }
```

## 繰り返しフィールド {#repeated-fields}

繰り返しフィールドには複数形の名前を使用してください。

```proto
repeated string keys = 1;
  ...
  repeated MyMessage accounts = 17;
```

## 列挙型 {#enums}

列挙型の型名にはPascalCase（最初の文字が大文字）を使用し、値の名前にはCAPITALS_WITH_UNDERSCORESを使用してください：

```proto
enum FooBar {
  FOO_BAR_UNSPECIFIED = 0;
  FOO_BAR_FIRST_VALUE = 1;
  FOO_BAR_SECOND_VALUE = 2;
}
```

各列挙値はセミコロンで終わるべきであり、カンマではないべきです。列挙値を囲むメッセージを避け、代わりに列挙値に接頭辞を付けることを好むべきです。一部の言語では列挙型が「struct」タイプ内で定義されることをサポートしていないため、これによりバインディング言語全体で一貫したアプローチが確保されます。

ゼロ値の列挙型は、予期しない列挙値を受け取ったサーバーやアプリケーションが、protoインスタンス内のフィールドを未設定としてマークするため、接尾辞`UNSPECIFIED`を持つべきです。その後、フィールドアクセサはデフォルト値を返します。列挙型の場合、デフォルト値は最初の列挙値です。未指定の列挙値に関する詳細は、[Proto Best Practicesページ](/programming-guides/dos-donts#unspecified-enum)を参照してください。

## サービス {#services}

`.proto`がRPCサービスを定義している場合、サービス名とRPCメソッド名の両方にPascalCase（最初の文字が大文字）を使用してください：

```proto
service FooService {
  rpc GetSomething(GetSomethingRequest) returns (GetSomethingResponse);
  rpc ListSomething(ListSomethingRequest) returns (ListSomethingResponse);
}
```

サービスに関する詳細なガイダンスについては、API Best Practicesトピックの[Create Unique Protos per Method](/programming-guides/api#unique-protos)および[Don't Include Primitive Types in a Top-level Request or Response Proto](/programming-guides/api#dont-include-primitive-types)を参照してください。

## 避けるべきこと {#avoid}

*   必須フィールド（proto2のみ）
*   グループ（proto2のみ）
