+++
title = "Dart Generated Code"
weight = 580
linkTitle = "生成されたコード"
description = "任意のプロトコル定義に対してプロトコルバッファコンパイラが生成するDartコードについて説明します。"
type = "docs"
+++

proto2 と proto3 で生成されるコードの違いは強調されています - これらの違いは、このドキュメントで説明されている生成されたコードにありますが、ベースのAPIは両バージョンで同じです。このドキュメントを読む前に、[proto2 言語ガイド](/programming-guides/proto2) および/または [proto3 言語ガイド](/programming-guides/proto3) を読む必要があります。

## コンパイラの呼び出し {#invocation}

プロトコルバッファコンパイラには、[Dartコードを生成するためのプラグイン](https://github.com/dart-lang/dart-protoc-plugin) が必要です。これをインストールするには、[手順](https://github.com/dart-lang/dart-protoc-plugin#how-to-build-and-use) に従って `protoc-gen-dart` バイナリが提供され、`protoc` が `--dart_out` コマンドラインフラグを使用して呼び出されます。`--dart_out` フラグは、コンパイラにDartソースファイルを書き込む場所を指示します。`.proto` ファイルの入力に対して、コンパイラは `.pb.dart` ファイルを生成します。

`.pb.dart` ファイルの名前は、`.proto` ファイルの名前を取り、2つの変更を加えて計算されます:

- 拡張子（`.proto`）が `.pb.dart` に置き換えられます。たとえば、`foo.proto` というファイルは `foo.pb.dart` という出力ファイルになります。
- protoパス（`--proto_path` または `-I` コマンドラインフラグで指定）は、出力パス（`--dart_out` フラグで指定）に置き換えられます。

たとえば、次のようにコンパイラを呼び出すと:

```shell
protoc --proto_path=src --dart_out=build/gen src/foo.proto src/bar/baz.proto
```

コンパイラは `src/foo.proto` と `src/bar/baz.proto` ファイルを読み込みます。`build/gen/foo.pb.dart` と `build/gen/bar/baz.pb.dart` を生成します。コンパイラは必要に応じてディレクトリ `build/gen/bar` を自動的に作成しますが、`build` または `build/gen` を作成しません。これらはすでに存在している必要があります。

## メッセージ {#message}

単純なメッセージ宣言が与えられた場合:

```proto
message Foo {}
```

プロトコルバッファコンパイラは、`GeneratedMessage` クラスを拡張する `Foo` というクラスを生成します。

`GeneratedMessage` クラスは、メッセージ全体をチェック、操作、読み取り、書き込みするためのメソッドを定義しています。これらのメソッドに加えて、`Foo` クラスは以下のメソッドとコンストラクタを定義しています:

-   `Foo()`: デフォルトコンストラクタ。すべての単一フィールドが未設定で、繰り返しフィールドが空のインスタンスを作成します。
-   `Foo.fromBuffer(...)`: メッセージを表すシリアライズされたプロトコルバッファデータから `Foo` を作成します。
-   `Foo.fromJson(...)`: メッセージをエンコードした JSON 文字列から `Foo` を作成します。
-   `Foo clone()`: メッセージ内のフィールドのディープクローンを作成します。
-   `Foo copyWith(void Function(Foo) updates)`: このメッセージの書き込み可能なコピーを作成し、`updates` を適用してから、コピーを読み取り専用にマークして返します。
-   `static Foo create()`: 単一の `Foo` を作成するためのファクトリ関数です。
-   `static PbList<Foo> createRepeated()`: `Foo` 要素の可変繰り返しフィールドを実装するリストを作成するためのファクトリ関数です。
-   `static Foo getDefault()`: `Foo` のシングルトンインスタンスを返します。これは、新しく構築された `Foo` インスタンスと同一であり、すべての単一フィールドが未設定で、すべての繰り返しフィールドが空です。

### ネストされた型

メッセージは他のメッセージ内で宣言することができます。例:

```proto
message Foo {
  message Bar {
  }
}
```

この場合、コンパイラは `Foo` と `Foo_Bar` の2つのクラスを生成します。

## フィールド

前のセクションで説明したメソッドに加えて、プロトコルバッファコンパイラは、`.proto` ファイル内で定義された各フィールドに対してアクセサメソッドを生成します。

生成される名前は常にキャメルケースの命名規則を使用します。`.proto` ファイル内のフィールド名がアンダースコアを使用している場合でも、この変換が適用されます。変換は以下のように行われます:

1.  名前内の各アンダースコアは削除され、次の文字が大文字になります。
2.  名前に接頭辞が付加される場合（例: \"has\"）、最初の文字は大文字になります。それ以外の場合は小文字になります。

したがって、`foo_bar_baz` フィールドの場合、ゲッターは `get fooBarBaz` となり、`has` で接頭辞が付いたメソッドは `hasFooBarBaz` となります。

### 単数のプリミティブフィールド（proto2）

次のいずれかのフィールド定義がある場合：

```proto
optional int32 foo = 1;
required int32 foo = 1;
```

コンパイラは、メッセージクラス内に次のアクセサメソッドを生成します：

-   `int get foo`：フィールドの現在の値を返します。フィールドが設定されていない場合は、デフォルト値を返します。
-   `bool hasFoo()`：フィールドが設定されている場合は `true` を返します。
-   `set foo(int value)`：フィールドの値を設定します。これを呼び出した後、`hasFoo()` は `true` を返し、`get foo` は `value` を返します。
-   `void clearFoo()`：フィールドの値をクリアします。これを呼び出した後、`hasFoo()` は `false` を返し、`get foo` はデフォルト値を返します。

他の単純なフィールドタイプについては、対応する Dart タイプは、[スカラー値タイプテーブル](/programming-guides/proto2#scalar)に従って選択されます。メッセージおよび列挙型の場合、値のタイプはメッセージまたは列挙型クラスに置き換えられます。

### 単数のプリミティブフィールド（proto3）

このフィールド定義に対して：

```proto
int32 foo = 1;
```

コンパイラは、メッセージクラス内に次のアクセサメソッドを生成します：

-   `int get foo`：フィールドの現在の値を返します。フィールドが設定されていない場合は、デフォルト値を返します。
-   `set foo(int value)`：フィールドの値を設定します。これを呼び出した後、`get foo` は `value` を返します。
-   `void clearFoo()`：フィールドの値をクリアします。これを呼び出した後、`get foo` はデフォルト値を返します。

**注意：** Dart proto3 の実装上の特異点により、`optional` 修飾子が proto 定義にない場合でも、[存在セマンティクスを要求する](/programming-guides/field_presence#presence-in-proto3-apis)ために使用される `hasFoo()` などのメソッドが生成されます。

-   `bool hasFoo()`：フィールドが設定されている場合は `true` を返します。
-   `void clearFoo()`：フィールドの値をクリアします。これを呼び出した後、`hasFoo()` は `false` を返し、`get foo` はデフォルト値を返します。

### 単数のメッセージフィールド {#singular-message}

```proto
message Bar {}
```

メッセージに `Bar` フィールドがある場合：

```proto
// proto2
message Baz {
  optional Bar bar = 1;
  // The generated code is the same result if required instead of optional.
}

// proto3
message Baz {
  Bar bar = 1;
}
```

コンパイラは、メッセージクラス内に以下のアクセサメソッドを生成します：

-   `Bar get bar`: フィールドの現在の値を返します。フィールドが設定されていない場合は、デフォルト値を返します。
-   `set bar(Bar value)`: フィールドの値を設定します。これを呼び出した後、`hasBar()` は `true` を返し、`get bar` は `value` を返します。
-   `bool hasBar()`: フィールドが設定されている場合は `true` を返します。
-   `void clearBar()`: フィールドの値をクリアします。これを呼び出した後、`hasBar()` は `false` を返し、`get bar` はデフォルト値を返します。
-   `Bar ensureBar()`: `hasBar()` が `false` を返す場合、`bar` を空のインスタンスに設定し、その後 `bar` の値を返します。これを呼び出した後、`hasBar()` は `true` を返します。

### 繰り返しフィールド

このフィールド定義に対して：

```proto
repeated int32 foo = 1;
```

コンパイラは以下を生成します：

-   `List<int> get foo`: フィールドをバックアップするリストを返します。フィールドが設定されていない場合は空のリストを返します。リストへの変更はフィールドに反映されます。

### Int64 フィールド

このフィールド定義に対して：

```proto
// proto2
optional int64 bar = 1;

// proto3
int64 bar = 1;
```

コンパイラは以下を生成します：

-   `Int64 get bar`: フィールド値を含む `Int64` オブジェクトを返します。

`Int64` は Dart のコアライブラリに組み込まれていません。これらのオブジェクトを使用するには、Dart の `fixnum` ライブラリをインポートする必要があるかもしれません：

```dart
import 'package:fixnum/fixnum.dart';
```

### Map フィールド

このような [`map`](/programming-guides/proto3#maps) フィールド定義がある場合：

```proto
map<int32, int32> map_field = 1;
```

コンパイラは以下のゲッターを生成します：

-   `Map<int, int> get mapField`: フィールドをバックアップする Dart マップを返します。フィールドが設定されていない場合は空のマップを返します。マップへの変更はフィールドに反映されます。

## Any

このような [`Any`](/programming-guides/proto3#any) フィールドがある場合：

```proto
import "google/protobuf/any.proto";

message ErrorStatus {
  string message = 1;
  google.protobuf.Any details = 2;
}
```

生成されたコードでは、`details` フィールドのゲッターは `com.google.protobuf.Any` のインスタンスを返します。これにより、`Any` の値をパックおよびアンパックするための特別なメソッドが提供されます。

```dart
    /// Unpacks the message in [value] into [instance].
    ///
    /// Throws a [InvalidProtocolBufferException] if [typeUrl] does not correspond
    /// to the type of [instance].
    ///
    /// A typical usage would be `any.unpackInto(new Message())`.
    ///
    /// Returns [instance].
    T unpackInto<T extends GeneratedMessage>(T instance,
        {ExtensionRegistry extensionRegistry = ExtensionRegistry.EMPTY});

    /// Returns `true` if the encoded message matches the type of [instance].
    ///
    /// Can be used with a default instance:
    /// `any.canUnpackInto(Message.getDefault())`
    bool canUnpackInto(GeneratedMessage instance);

    /// Creates a new [Any] encoding [message].
    ///
    /// The [typeUrl] will be [typeUrlPrefix]/`fullName` where `fullName` is
    /// the fully qualified name of the type of [message].
    static Any pack(GeneratedMessage message,
        {String typeUrlPrefix = 'type.googleapis.com'});
```

## Oneof

[`oneof`](/programming-guides/proto3#oneof) という定義が与えられた場合、以下のように Dart の enum 型が生成されます:

```proto
 enum Foo_Test { name, subMessage, notSet }
```

さらに、以下のメソッドが生成されます:

-   `Foo_Test whichTest()`: どのフィールドが設定されているかを示す enum を返します。どれも設定されていない場合は `Foo_Test.notSet` を返します。
-   `void clearTest()`: 現在設定されている oneof フィールドの値をクリアし、oneof ケースを `Foo_Test.notSet` に設定します。

oneof 定義内の各フィールドに対して通常のフィールドアクセサメソッドが生成されます。例えば `name` の場合:

-   `String get name`: oneof ケースが `Foo_Test.name` の場合、フィールドの現在の値を返します。それ以外の場合はデフォルト値を返します。
-   `set name(String value)`: フィールドの値を設定し、oneof ケースを `Foo_Test.name` に設定します。これを呼び出した後、`get name` は `value` を返し、`whichTest()` は `Foo_Test.name` を返します。
-   `void clearName()`: oneof ケースが `Foo_Test.name` でない場合は何も変更されません。そうでない場合はフィールドの値をクリアします。これを呼び出した後、`get name` はデフォルト値を返し、`whichTest()` は `Foo_Test.notSet` を返します。

## Enumerations {#enum}

以下のような enum 定義が与えられた場合、protocol buffer コンパイラは `Color` というクラスを生成します。このクラスは `ProtobufEnum` クラスを拡張し、4 つの値ごとに `static const Color` を含み、値を含む `static const List<Color>` も含まれます。

```dart
static const List<Color> values = <Color> [
  COLOR_UNSPECIFIED,
  COLOR_RED,
  COLOR_GREEN,
  COLOR_BLUE,
];
```

また、以下のメソッドが含まれます:

-   `static Color? valueOf(int value)`: 指定された数値に対応する `Color` を返します。

各値には以下のプロパティがあります:

-   `name`: .proto ファイルで指定された enum の名前。
-   `value`: .proto ファイルで指定された enum の整数値。

`.proto` 言語では、複数の enum シンボルが同じ数値を持つことが許可されています。同じ数値を持つシンボルは同義語です。例:

```proto
enum Foo {
  BAR = 0;
  BAZ = 0;
}
```

この場合、`BAZ`は`BAR`の同義語であり、次のように定義されます：

```dart
static const Foo BAZ = BAR;
```

列挙型はメッセージ型の中にネストして定義することができます。たとえば、次のような列挙型の定義があるとします：

```proto
message Bar {
  enum Color {
    COLOR_UNSPECIFIED = 0;
    COLOR_RED = 1;
    COLOR_GREEN = 2;
    COLOR_BLUE = 3;
  }
}
```

プロトコルバッファコンパイラは、`Bar`というクラスを生成し、`GeneratedMessage`を拡張するクラス`Bar_Color`を生成します。

## 拡張機能（proto2のみ） {#extension}

`foo_test.proto`というファイルがあり、[拡張範囲](/programming-guides/proto2#extensions)を持つメッセージとトップレベルの拡張機能の定義が含まれている場合：

```proto
message Foo {
  extensions 100 to 199;
}

extend Foo {
  optional int32 bar = 101;
}
```

プロトコルバッファコンパイラは、`Foo`クラスに加えて、`Foo_test`というクラスを生成します。このクラスには、ファイル内の各拡張フィールドに対する`static Extension`と、`ExtensionRegistry`にすべての拡張機能を登録するメソッドが含まれます：

-   `static final Extension bar`
-   `static void registerAllExtensions(ExtensionRegistry registry)`：指定されたレジストリにすべての定義済み拡張機能を登録します。

`Foo`の拡張機能アクセサは次のように使用できます：

```dart
Foo foo = Foo();
foo.setExtension(Foo_test.bar, 1);
assert(foo.hasExtension(Foo_test.bar));
assert(foo.getExtension(Foo_test.bar)) == 1);
```

拡張機能は、別のメッセージ内にネストして宣言することもできます：

```proto
message Baz {
  extend Foo {
    optional int32 bar = 124;
  }
}
```

この場合、拡張機能`bar`は`Baz`クラスの静的メンバーとして宣言されます。

拡張機能を持つメッセージを解析する際には、解析したい拡張機能を登録した`ExtensionRegistry`を提供する必要があります。そうしないと、これらの拡張機能は未知のフィールドとして扱われます。たとえば：

```dart
ExtensionRegistry registry = ExtensionRegistry();
registry.add(Baz.bar);
Foo foo = Foo.fromBuffer(input, registry);
```

既に未知のフィールドを持つ解析済みメッセージがある場合、`ExtensionRegistry`の`reparseMessage`を使用してメッセージを再解析できます。未知のフィールドのセットに登録されている拡張機能がレジストリに存在する場合、これらの拡張機能が解析され、未知のフィールドセットから削除されます。メッセージにすでに存在する拡張機能は保持されます。

```dart
Foo foo = Foo.fromBuffer(input);
ExtensionRegistry registry = ExtensionRegistry();
registry.add(Baz.bar);
Foo reparsed = registry.reparseMessage(foo);
```

この拡張を取得する方法は、全体的にコストが高いです。
可能な限り、`GeneratedMessage.fromBuffer` を行う際に、必要なすべての拡張を持つ `ExtensionRegistry` を使用することをお勧めします。

## サービス {#service}

サービス定義が与えられた場合：

```proto
service Foo {
  rpc Bar(FooRequest) returns(FooResponse);
}
```

プロトコルバッファコンパイラは、`grpc` オプション（例：`--dart_out=grpc:output_folder`）を指定して呼び出すことができます。その場合、[gRPC](//www.grpc.io/) をサポートするコードが生成されます。詳細については、[gRPC Dartクイックスタートガイド](https://grpc.io/docs/quickstart/dart.html) を参照してください。
```
