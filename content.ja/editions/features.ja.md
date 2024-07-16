+++
title = "エディション向けの機能設定"
weight = 43
description = "Protobufエディションの機能とそれらがprotobufの動作にどのように影響するかについて。"
type = "docs"
+++

このトピックでは、エディション2023に含まれる機能の概要を提供します。各後続のエディションの機能はこのトピックに追加されます。新しいエディションは[ニュースセクション](/news)で発表します。

## Prototiller {#prototiller}

Prototillerは、proto2およびproto3定義ファイルをエディション構文に変換するコマンドラインツールです。まだリリースされていませんが、このトピック全体で参照されています。

## 機能 {#features}

以下のセクションには、エディション2023で構成可能なすべての動作が含まれています。[proto2またはproto3の動作を保持](#preserving)する方法については、デフォルトの動作を上書きして、proto定義ファイルがproto2またはproto3ファイルのように動作するようにする方法を示します。エディションと機能がどのように動作を設定するために一緒に機能するかの詳細については、[Protobufエディションの概要](/editions/overview)を参照してください。

以下の各セクションには、機能が適用されるスコープに関するエントリがあります。これには、ファイル、列挙型、メッセージ、またはフィールドが含まれる場合があります。次のサンプルは、各スコープに適用されたモック機能を示しています：

```proto
edition = "2023";

// File-scope definition
option features.bar = BAZ;

enum Foo {
  // Enum-scope definition
  option features.bar = QUX;

  A = 1;
  B = 2;
}

message Corge {
  // Message-scope definition
  option features.bar = QUUX;

  // Field-scope definition
  Foo A = 1 [features.bar = GRAULT];
}
```

この例では、フィールドスコープの機能定義で設定された`GRAULT`設定が、メッセージスコープの`QUUX`設定を上書きします。

### `features.enum_type` {#enum_type}

この機能は、定義されたセット内に含まれない列挙値の処理方法を設定します。詳細については、[Enum動作](/programming-guides/enum)を参照してください。

この機能はproto3ファイルに影響を与えませんので、このセクションにはproto3ファイルの変更前と変更後がありません。

**利用可能な値:**

*   `CLOSED:` 閉じられた列挙型は、範囲外の列挙値を未知のフィールドセットに格納します。
*   `OPEN:` オープンな列挙型は、範囲外の値を直接フィールドに解析します。

**適用されるスコープ:** ファイル、列挙型

**エディション2023のデフォルト動作:** `OPEN`

**proto2での動作:** `CLOSED`

**proto3での動作:** `OPEN`

```proto
syntax = "proto2";

enum Foo {
  A = 2;
  B = 4;
  C = 6;
}
```

After running [Prototiller](#prototiller), the equivalent code might look like
this:

```proto
edition = "2023";

enum Foo {
  option features.enum_type = CLOSED;
  A = 2;
  B = 4;
  C = 6;
}
```

### `features.field_presence` {#field_presence}

この機能は、フィールドの存在を追跡する動作、またはprotobufフィールドに値があるかどうかの概念を設定します。

**利用可能な値:**

*   `LEGACY_REQUIRED`: フィールドは解析とシリアル化に必要です。明示的に設定された値は、ワイヤにシリアル化されます（デフォルト値と同じでも）。
*   `EXPLICIT`: フィールドには明示的な存在の追跡があります。明示的に設定された値は、ワイヤにシリアル化されます（デフォルト値と同じでも）。単一のプリミティブフィールドの場合、`has_*` 関数が `EXPLICIT` に設定されたフィールドに生成されます。
*   `IMPLICIT`: フィールドには存在の追跡がありません。デフォルト値はワイヤにシリアル化されません（明示的に設定されていても）。`IMPLICIT` に設定されたフィールドには `has_*` 関数が生成されません。

**適用可能なスコープ:** ファイル、フィールド

**Edition 2023 のデフォルト値:** `EXPLICIT`

**proto2 での動作:** `EXPLICIT`

**proto3 での動作:** フィールドに `optional` ラベルがない限り、`IMPLICIT` と同様に動作します。詳細については、[Proto3 API における存在](/programming-guides/field_presence#presence-in-proto3-apis) を参照してください。

以下のコードサンプルは、proto2 ファイルを示しています:

```proto
syntax = "proto2";

message Foo {
  required int32 x = 1;
  optional int32 y = 2;
  repeated int32 z = 3;
}
```

Prototiller を実行した後、同等のコードは次のようになります:

```proto
edition = "2023";

message Foo {
  int32 x = 1 [features.field_presence = LEGACY_REQUIRED];
  int32 y = 2;
  repeated int32 z = 3;
}
```

以下は、proto3 ファイルを示しています:

```proto
syntax = "proto3";

message Bar {
  int32 x = 1;
  optional int32 y = 2;
  repeated int32 z = 3;
}
```

Prototiller を実行した後、同等のコードは次のようになります:

```proto
edition = "2023";
option features.field_presence = IMPLICIT;

message Bar {
  int32 x = 1;
  int32 y = 2 [features.field_presence = EXPLICIT];
  repeated int32 z = 3;
}
```

`required` および `optional` ラベルは、Editions にはもはや存在せず、対応する動作は `field_presence` 機能で明示的に設定されます。

### `features.json_format` {#json_format}

この機能は、JSON の解析とシリアル化の動作を設定します。

この機能はproto3ファイルに影響を与えないため、このセクションにはproto3ファイルの前後がありません。エディションの動作はproto3と一致します。

**利用可能な値:**

- `ALLOW`: ランタイムはJSONのパースとシリアライズを許可する必要があります。Protoレベルでチェックが適用され、JSONへの明確なマッピングがあることを確認します。
- `LEGACY_BEST_EFFORT`: ランタイムはJSONの解析とシリアライズを最善の努力で行います。特定のProtoが許可され、ランタイムで未指定の動作が発生する可能性があります（たとえば、多：1または1：多のマッピングなど）。

**以下のスコープに適用可能:** ファイル、メッセージ、列挙型

**エディション2023のデフォルト動作:** `ALLOW`

**proto2の動作:** `LEGACY_BEST_EFFORT`

**proto3の動作:** `ALLOW`

以下のコードサンプルはproto2ファイルを示しています:

```proto
syntax = "proto2";

message Foo {
  // Warning only
  string bar = 1;
  string bar_ = 2;
}
```

Prototillerを実行した後、同等のコードは次のようになるかもしれません:

```proto
edition = "2023";
features.json_format = LEGACY_BEST_EFFORT;

message Foo {
  string bar = 1;
  string bar_ = 2;
}
```

### `features.message_encoding` {#message_encoding}

この機能は、シリアライズ時にフィールドのエンコーディング動作を設定します。

この機能はproto3ファイルに影響を与えないため、このセクションにはproto3ファイルの前後がありません。

言語によっては、「グループのような」フィールドには、生成されたコードやテキスト形式で予期しない大文字が含まれる場合があります。これは、proto2との後方互換性を提供するためです。メッセージフィールドは、「グループのような」ものである場合、次の条件がすべて満たされています:

- `DELIMITED`メッセージエンコーディングが指定されている
- メッセージタイプがフィールドと同じスコープで定義されている
- フィールド名が正確に小文字化されたタイプ名である

**利用可能な値:**

- `LENGTH_PREFIXED`: フィールドは、[メッセージ構造](/programming-guides/encoding#structure)で説明されているLENワイヤータイプを使用してエンコードされます。
- `DELIMITED`: メッセージ型のフィールドは、[グループ](/programming-guides/proto2#groups)としてエンコードされます。

**以下のスコープに適用可能:** ファイル、フィールド

**エディション2023のデフォルト動作:** `LENGTH_PREFIXED`

**proto2の動作:** グループを除き、`LENGTH_PREFIXED`がデフォルトですが、グループは`DELIMITED`がデフォルトです。

**proto3 での動作:** `LENGTH_PREFIXED`。Proto3 では `DELIMITED` はサポートされていません。

次のコードサンプルは proto2 ファイルを示しています：

```proto
syntax = "proto2";

message Foo {
  group Bar = 1 {
    optional int32 x = 1;
    repeated int32 y = 2;
  }
}
```

Prototiller を実行した後、同等のコードは次のようになるかもしれません：

```proto
edition = "2023";

message Foo {
  message Bar {
    int32 x = 1;
    repeated int32 y = 2;
  }
  Bar bar = 1 [features.message_encoding = DELIMITED];
}
```

### `features.repeated_field_encoding` {#repeated_field_encoding}

この機能は、Editions で `repeated` フィールドのための proto2/proto3 の
[`packed` オプション](/programming-guides/encoding#packed)
が移行されたものです。

**利用可能な値:**

*   `PACKED`: 原始型の `Repeated` フィールドは、各要素を連結した単一の LEN
    レコードとしてエンコードされます。
*   `EXPANDED`: `Repeated` フィールドは、各値ごとにフィールド番号でエンコードされます。

**適用可能なスコープ:** ファイル、フィールド

**Edition 2023 でのデフォルト動作:** `PACKED`

**proto2 での動作:** `EXPANDED`

**proto3 での動作:** `PACKED`

次のコードサンプルは proto2 ファイルを示しています：

```proto
syntax = "proto2";

message Foo {
  repeated int32 bar = 6 [packed=true];
  repeated int32 baz = 7;
}
```

Prototiller を実行した後、同等のコードは次のようになるかもしれません：

```proto
edition = "2023";
option features.repeated_field_encoding = EXPANDED;

message Foo {
  repeated int32 bar = 6 [features.repeated_field_encoding=PACKED];
  repeated int32 baz = 7;
}
```

次のコードは proto3 ファイルを示しています：

```proto
syntax = "proto3";

message Foo {
  repeated int32 bar = 6;
  repeated int32 baz = 7 [packed=false];
}
```

Prototiller を実行した後、同等のコードは次のようになるかもしれません：

```proto
edition = "2023";

message Foo {
  repeated int32 bar = 6;
  repeated int32 baz = 7 [features.repeated_field_encoding=EXPANDED];
}
```

### `features.utf8_validation` {#utf8_validation}

この機能は、文字列の検証方法を設定します。言語固有の `utf8_validation` 機能が
これをオーバーライドする場合を除き、すべての言語に適用されます。
Java 言語固有の機能については、[`features.(pb.java).utf8_validation`](#java-utf8_validation) を参照してください。

この機能は proto3 ファイルに影響を与えませんので、このセクションには
proto3 ファイルの before と after がありません。

**利用可能な値:**

*   `VERIFY`: ランタイムは UTF-8 を検証する必要があります。これはデフォルトの
    proto3 の動作です。
*   `NONE`: フィールドはワイヤー上で検証されていない `bytes` フィールドのように
    振る舞います。パーサーはこのタイプのフィールドを予測不能な方法で処理する可能性があり、
    無効な文字を置換するなどの方法です。これはデフォルトの proto2 の動作です。

**適用可能なスコープ:** ファイル、フィールド

**Edition 2023のデフォルト動作:** `VERIFY`

**proto2での動作:** `NONE`

**proto3での動作:** `VERIFY`

以下のコードサンプルは、proto2ファイルを示しています:

```proto
syntax = "proto2";

message MyMessage {
  string foo = 1;
}
```

Prototillerを実行した後、同等のコードは次のようになるかもしれません:

```proto
edition = "2023";

message MyMessage {
  string foo = 1 [features.utf8_validation = NONE];
}
```

### 言語固有の機能 {#lang-specific}

一部の機能は特定の言語に適用され、他の言語の同じprotoには適用されません。これらの機能を使用するには、言語のランタイムから対応する*_features.protoファイルをインポートする必要があります。次のセクションの例では、これらのインポートを示しています。

#### `features.(pb.cpp/pb.java).legacy_closed_enum` {#legacy_closed_enum}

**言語:** C++, Java

この機能は、オープンなenum型のフィールドをクローズドなenumとして振る舞わせるかどうかを決定します。これにより、JavaとC++でproto2およびproto3から[準拠していない動作](/programming-guides/enum)を再現できます。

この機能はproto3ファイルに影響を与えませんので、このセクションにはproto3ファイルのbeforeとafterがありません。

**利用可能な値:**

*   `true`: [`enum_type`](#enum_type)に関係なくenumをクローズドとして扱います。
*   `false`: `enum_type`で設定された内容を尊重します。

**適用可能なスコープ:** ファイル、フィールド

**Edition 2023のデフォルト動作:** `false`

**proto2での動作:** `true`

**proto3での動作:** `false`

以下のコードサンプルは、proto2ファイルを示しています:

```proto
syntax = "proto2";

import "myproject/proto3file.proto";

message Msg {
  myproject.proto3file.Proto3Enum name = 1;
}
```

Prototillerを実行した後、同等のコードは次のようになるかもしれません:

```proto
edition = "2023";

import "myproject/proto3file.proto";

import "google/protobuf/cpp_features.proto";
import "google/protobuf/java_features.proto";

message Msg {
  myproject.proto3file.Proto3Enum name = 1 [
    features.(pb.cpp).legacy_closed_enum = true,
    features.(pb.java).legacy_closed_enum = true
  ];
}
```

#### `features.(pb.cpp).string_type` {#string_type}

**言語:** C++

この機能は、生成されたコードが文字列フィールドをどのように扱うかを決定します。これはproto2およびproto3の`ctype`オプションを置き換え、新しい`string_view`機能を提供します。Edition 2023では、フィールドに`ctype`または`string_view`のいずれかを指定できますが、両方を指定することはできません。

**利用可能な値:**

*   `VIEW`: フィールドのために`string_view`アクセサを生成します。これは将来のエディションでデフォルトになります。
*   `CORD`: フィールドのために`Cord`アクセサを生成します。
*   `STRING`: フィールドのために`string`アクセサを生成します。

**適用可能なスコープ:** ファイル、フィールド

**Edition 2023のデフォルト動作:** `STRING`

**proto2での動作:** `STRING`

**proto3での動作:** `STRING`

以下のコードサンプルは、proto2ファイルを示しています:

```proto
syntax = "proto2";

message Foo {
  optional string bar = 6;
  optional string baz = 7 [ctype = CORD];
}
```

Prototillerを実行した後、同等のコードは次のようになるかもしれません:

```proto
edition = "2023";

import "google/protobuf/cpp_features.proto";

message Foo {
  string bar = 6;
  string baz = 7 [features.(pb.cpp).string_type = CORD];
}
```

以下は、proto3ファイルを示しています:

```proto
syntax = "proto3"

message Foo {
  string bar = 6;
  string baz = 7 [ctype = CORD];
}
```

Prototillerを実行した後、同等のコードは次のようになるかもしれません:

```proto
edition = "2023";

import "google/protobuf/cpp_features.proto";

message Foo {
  string bar = 6;
  string baz = 7 [features.(pb.cpp).string_type = CORD];
}
```

#### `features.(pb.java).utf8_validation` {#java-utf8_validation}

**言語:** Java

この言語固有の機能を使用すると、Javaのフィールドレベルでファイルレベルの設定を上書きできます。

この機能はproto3ファイルに影響を与えないため、このセクションにはproto3ファイルの変更前後がありません。

**利用可能な値:**

*   `DEFAULT`: [`features.utf8_validation`](#utf8_validation) で設定された動作に一致します。
*   `VERIFY`: Javaのみで`VERIFY`に強制するために、ファイルレベルの`features.utf8_validation`設定を上書きします。

**適用可能なスコープ:** フィールド、ファイル

**Edition 2023のデフォルト動作:** `DEFAULT`

**proto2での動作:** `DEFAULT`

**proto3での動作:** `DEFAULT`

以下のコードサンプルは、proto2ファイルを示しています:

```proto
syntax = "proto2";

option java_string_check_utf8=true;

message MyMessage {
  string foo = 1;
  string bar = 2;
}
```

Prototillerを実行した後、同等のコードは次のようになるかもしれません:

```proto
edition = "2023";

import "google/protobuf/java_features.proto";

option features.utf8_validation = NONE;
option features.(pb.java).utf8_validation = VERIFY;
message MyMessage {
  string foo = 1;
}
```

## proto2またはproto3の動作の保持 {#preserving}

Editions形式に移行したいが、生成されたコードの動作にまだ変更を加えたくない場合があります。このセクションでは、Prototillerツールが行う.protoファイルへの変更を示し、Edition 2023のprotoがproto2またはproto3ファイルのように動作するようにします。

これらの変更がファイルレベルで行われると、proto2またはproto3のデフォルトが得られます。追加の動作の違い（例: [required, proto3 optional](#caveats)）を考慮するか、定義がproto2またはproto3に*ほぼ*似ているようにしたい場合は、より低いレベル（メッセージレベル、フィールドレベル）で上書きできます。


おすすめは、特定の理由がない限り Prototiller を使用することです。Prototiller を使用せずにこれらすべてを手動で適用する場合は、次のセクションの内容を.proto ファイルの先頭に追加してください。

### Proto2 の動作 {#proto2-behavior}

```proto
edition = "2023";

import "google/protobuf/cpp_features.proto";
import "google/protobuf/java_features.proto";

option features.field_presence = EXPLICIT;
option features.enum_type = CLOSED;
option features.repeated_field_encoding = EXPANDED;
option features.json_format = LEGACY_BEST_EFFORT;
option features.utf8_validation = NONE;
option features.(pb.cpp).legacy_closed_enum = true;
option features.(pb.java).legacy_closed_enum = true;
```

### Proto3 の動作 {#proto3-behavior}

```proto
// proto3 behaviors
edition = "2023";

import "google/protobuf/cpp_features.proto";
import "google/protobuf/java_features.proto";

option features.field_presence = IMPLICIT;
option features.enum_type = OPEN;
// `packed=false` needs to be transformed to field-level repeated_field_encoding
// features in Editions syntax
option features.json_format = ALLOW;
option features.utf8_validation = VERIFY;
option features.(pb.cpp).legacy_closed_enum = false;
option features.(pb.java).legacy_closed_enum = false;
```

### 注釈と例外 {#caveats}

このセクションでは、Prototiller を使用しない場合に手動で行う必要がある変更が示されています。

前のセクションに表示されているファイルレベルのデフォルトを設定すると、ほとんどの場合にデフォルトの動作が設定されますが、いくつかの例外があります。

*   `optional`: `optional` ラベルのすべてのインスタンスを削除し、ファイルのデフォルトが `IMPLICIT` の場合は [`features.field_presence`](#field_presence) を `EXPLICIT` に変更します。
*   `required`: `required` ラベルのすべてのインスタンスを削除し、フィールドレベルで [`features.field_presence=LEGACY_REQUIRED`](#field_presence) オプションを追加します。
*   `groups`: `groups` を別のメッセージに展開し、フィールドレベルで `features.message_encoding = DELIMITED` オプションを追加します。詳細については、[`features.message_encoding`](#message_encoding) を参照してください。
*   `java_string_check_utf8`: このファイルオプションを削除し、[`features.(pb.java).utf8_validation`](#java-utf8_validation) に置き換えます。Java の機能をインポートする必要があります。詳細は、[言語固有の機能](#lang-specific)を参照してください。
*   `packed`: editions 形式に変換された proto2 ファイルの場合、`packed` フィールドオプションを削除し、`EXPANDED` 動作を設定したくない場合は、フィールドレベルで `[features.repeated_field_encoding=PACKED]` を追加します。proto3 ファイルを editions 形式に変換した場合は、デフォルトの proto3 の動作を設定したくない場合は、フィールドレベルで `[features.repeated_field_encoding=EXPANDED]` を追加します。
