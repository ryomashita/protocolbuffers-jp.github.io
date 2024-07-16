+++
title = "Java生成コードガイド"
weight = 650
linkTitle = "生成コードガイド"
description = "任意のプロトコル定義に対してプロトコルバッファコンパイラが生成するJavaコードを正確に説明します。"
type = "docs"
+++

proto2とproto3で生成されるコードの違いは強調されます。これらの違いは、このドキュメントで説明されている生成されたコードにありますが、基本メッセージクラス/インターフェースは両バージョンで同じです。このドキュメントを読む前に、[proto2言語ガイド](/programming-guides/proto2)および/または[proto3言語ガイド](/programming-guides/proto3)を読む必要があります。

Javaのプロトコルバッファメソッドは、明示的に指定されていない限り、nullを受け入れたり返したりしません。

## コンパイラの呼び出し {#invocation}

プロトコルバッファコンパイラは、`--java_out=`コマンドラインフラグを使用して呼び出されると、Java出力を生成します。`--java_out=`オプションのパラメータは、コンパイラがJava出力を書き込むディレクトリです。各`.proto`ファイルの入力に対して、コンパイラは、`.proto`ファイル自体を表すJavaクラスを含むラッパー`.java`ファイルを作成します。

`.proto`ファイルに次のような行が含まれている場合：

```proto
option java_multiple_files = true;
```

その場合、コンパイラは、`.proto`ファイルで宣言された各トップレベルメッセージ、列挙型、およびサービスに対して生成されるクラス/列挙型ごとに個別の`.java`ファイルも作成します。

それ以外の場合（つまり、`java_multiple_files`オプションが無効になっている場合；これがデフォルトです）、前述のラッパークラスも外部クラスとして使用され、`.proto`ファイルで宣言された各トップレベルメッセージ、列挙型、およびサービスに対して生成されるクラス/列挙型はすべて外部ラッパークラス内にネストされます。したがって、コンパイラは`.proto`ファイル全体に対して単一の`.java`ファイルのみを生成します。

ラッパークラスの名前は次のように選択されます：`.proto`ファイルに次のような行が含まれている場合：

```proto
option java_outer_classname = "Foo";
```

その場合、ラッパークラスの名前は`Foo`になります。それ以外の場合、ラッパークラスの名前は`.proto`ファイルのベース名をキャメルケースに変換して決定されます。例えば、`foo_bar.proto`は`FooBar`というクラス名を生成します。ファイル内に同じ名前のサービス、列挙型、またはメッセージ（ネストされた型を含む）がある場合、ラッパークラスの名前に"OuterClass"が追加されます。例：

-   `foo_bar.proto` に `FooBar` というメッセージが含まれている場合、ラッパークラスは `FooBarOuterClass` というクラス名を生成します。

-   `foo_bar.proto` に `FooService` というサービスが含まれており、`java_outer_classname` が `FooService` という文字列に設定されている場合、ラッパークラスは `FooServiceOuterClass` というクラス名を生成します。

**注意:** プロトコルバッファ API の非推奨バージョン v1 を使用している場合、メッセージ名との衝突に関係なく `OuterClass` が追加されます。

ネストされたクラスに加えて、ラッパークラス自体には以下の API があります（ラッパークラスが `Foo` という名前であり、`foo.proto` から生成されたと仮定します）:

```java
public final class Foo {
  private Foo() {}  // Not instantiable.

  /** Returns a FileDescriptor message describing the contents of {@code foo.proto}. */
  public static com.google.protobuf.Descriptors.FileDescriptor getDescriptor();
  /** Adds all extensions defined in {@code foo.proto} to the given registry. */
  public static void registerAllExtensions(com.google.protobuf.ExtensionRegistry registry);
  public static void registerAllExtensions(com.google.protobuf.ExtensionRegistryLite registry);

  // (Nested classes omitted)
}
```

Java パッケージ名は、以下の [Packages](#package) で説明されている方法で選択されます。

出力ファイルは、`--java_out=` に連結されたパラメータ、パッケージ名（`.` が `/` に置き換えられたもの）、および `.java` ファイル名によって選択されます。

たとえば、次のようにコンパイラを呼び出したとします:

```shell
protoc --proto_path=src --java_out=build/gen src/foo.proto
```

`foo.proto` の Java パッケージが `com.example` であり、`java_multiple_files` を有効にせず、外部クラス名が `FooProtos` の場合、プロトコルバッファコンパイラはファイル `build/gen/com/example/FooProtos.java` を生成します。プロトコルバッファコンパイラは、必要に応じて `build/gen/com` および `build/gen/com/example` ディレクトリを自動的に作成します。ただし、`build/gen` または `build` は作成されません。複数の `.proto` ファイルを単一の呼び出しで指定できます。すべての出力ファイルが一度に生成されます。

Java コードを出力する際、プロトコルバッファコンパイラが JAR アーカイブに直接出力できる能力は特に便利です。多くの Java ツールがソースコードを JAR ファイルから直接読み取ることができます。JAR ファイルに出力するには、単に `.jar` で終わる出力場所を指定します。注意点として、アーカイブには Java ソースコードのみが配置されます。それを Java クラスファイルに変換するためには、引き続き別途コンパイルする必要があります。

たとえば、`.proto` ファイルに次のような内容が含まれている場合:

```proto
package foo.bar;
```

その後、生成されるJavaクラスはJavaパッケージ`foo.bar`に配置されます。ただし、`.proto`ファイルに`java_package`オプションも含まれている場合、次のようになります：

```proto
package foo.bar;
option java_package = "com.example.foo.bar";
```

その場合、クラスは代わりに`com.example.foo.bar`パッケージに配置されます。`java_package`オプションは、通常の`.proto` `package`宣言が逆ドメイン名で始まることは想定されていないため提供されています。

## メッセージ {#message}

単純なメッセージ宣言が与えられた場合：

```proto
message Foo {}
```

プロトコルバッファコンパイラは`Foo`というクラスを生成し、`Message`インターフェースを実装します。クラスは`final`で宣言されており、さらなるサブクラス化は許可されません。`Foo`は`GeneratedMessage`を拡張していますが、これは実装の詳細と見なすべきです。デフォルトでは、`Foo`は最大の速度向上のために多くの`GeneratedMessage`のメソッドを特殊なバージョンでオーバーライドします。ただし、`.proto`ファイルに次の行が含まれている場合：

```proto
option optimize_for = CODE_SIZE;
```

その場合、`Foo`は機能するために必要な最小限のメソッドのみをオーバーライドし、残りのメソッドは`GeneratedMessage`のリフレクションベースの実装に依存します。これにより生成されるコードのサイズが大幅に削減されますが、パフォーマンスも低下します。また、`.proto`ファイルに次の行が含まれている場合：

```proto
option optimize_for = LITE_RUNTIME;
```

その場合、`Foo`にはすべてのメソッドの高速な実装が含まれますが、`MessageLite`インターフェースを実装し、`Message`のメソッドのサブセットを含みます。特に、記述子、ネストしたビルダー、またはリフレクションをサポートしません。ただし、このモードでは、生成されたコードは`libprotobuf.jar`の代わりに`libprotobuf-lite.jar`にリンクする必要があります。"lite"ライブラリはフルライブラリよりもはるかに小さく、モバイル電話などのリソースに制約のあるシステムに適しています。

`Message`インターフェースは、メッセージ全体をチェック、操作、読み取り、書き込みするためのメソッドを定義します。これらのメソッドに加えて、`Foo`クラスは次の静的メソッドを定義します：```

-   `static Foo getDefaultInstance()`: `Foo`の*シングルトン*インスタンスを返します。このインスタンスの内容は、`Foo.newBuilder().build()`を呼び出した場合と同じです（つまり、すべての単数フィールドが未設定で、すべての繰り返しフィールドが空です）。メッセージのデフォルトインスタンスは、その`newBuilderForType()`メソッドを呼び出すことでファクトリとして使用できます。

-   `static Descriptor getDescriptor()`: タイプの記述子を返します。これには、フィールドの情報やそのタイプなど、タイプに関する情報が含まれています。これは、`Message`のリフレクションメソッドと一緒に使用できます。例えば、`getField()`です。

-   `static Foo parseFrom(...)`: 指定されたソースから`Foo`タイプのメッセージを解析して返します。`Message.Builder`インターフェースの`mergeFrom()`の各バリアントに対応する1つの`parseFrom`メソッドがあります。`parseFrom()`は決して`UninitializedMessageException`をスローしません。必要なフィールドが欠落している場合は、`InvalidProtocolBufferException`をスローします。これにより、`Foo.newBuilder().mergeFrom(...).build()`を呼び出すのと微妙に異なることに注意してください。

-   `static Parser parser()`: `Parser`のインスタンスを返します。さまざまな`parseFrom()`メソッドを実装しています。

-   `Foo.Builder newBuilder()`: 新しいビルダーを作成します（後述）。

-   `Foo.Builder newBuilder(Foo prototype)`: すべてのフィールドが`prototype`で持つ値と同じ値で初期化された新しいビルダーを作成します。埋め込みメッセージや文字列オブジェクトは不変なため、オリジナルとコピーの間で共有されます。

### ビルダー {#builders}

メッセージオブジェクト&mdash;上記で説明した`Foo`クラスのインスタンスなど&mdash;は、Javaの`String`と同様に不変です。メッセージオブジェクトを構築するには、*ビルダー*を使用する必要があります。各メッセージクラスには、独自のビルダークラスがあります。したがって、`Foo`の例では、プロトコルバッファコンパイラは`Foo`を構築するために使用できるネストされたクラス`Foo.Builder`を生成します。`Foo.Builder`は`Message.Builder`インターフェースを実装しています。`GeneratedMessage.Builder`クラスを拡張していますが、これも実装の詳細と見なすべきです。`Foo`と同様に、`Foo.Builder`は、`GeneratedMessage.Builder`内のジェネリックメソッドの実装に依存するか、`optimize_for`オプションが使用されている場合は、はるかに高速な生成されたカスタムコードに依存する場合があります。

`Foo.Builder`は静的メソッドを定義していません。そのインターフェースは、`Message.Builder`インターフェースで定義されているものとまったく同じですが、異なる点は戻り値の型がより具体的であることです。つまり、ビルダーを変更する`Foo.Builder`のメソッドは`Foo.Builder`を返し、`build()`は`Foo`型を返します。

ビルダーの内容を変更するメソッド（フィールドセッターを含む）は常にビルダーへの参照を返します（つまり、`return this;`を行います）。これにより、複数のメソッド呼び出しを1行で連鎖させることができます。例えば：`builder.mergeFrom(obj).setFoo(1).setBar("abc").clearBaz();`

ビルダーはスレッドセーフではないため、1つのビルダーの内容を複数の異なるスレッドが変更する必要がある場合は、Javaの同期化を使用する必要があります。

### サブビルダー {#sub-builders}

サブメッセージを含むメッセージの場合、コンパイラはサブビルダーも生成します。これにより、深くネストされたサブメッセージを繰り返し変更することができます。例：

```proto
message Foo {
  optional int32 val = 1;
  // some other fields.
}

message Bar {
  optional Foo foo = 1;
  // some other fields.
}

message Baz {
  optional Bar bar = 1;
  // some other fields.
}
```

既に`Baz`メッセージがある場合、`Foo`の深くネストされた`val`を変更したい場合。次のように書くことができます：

```java
baz = baz.toBuilder().setBar(
    baz.getBar().toBuilder().setFoo(
        baz.getBar().getFoo().toBuilder().setVal(10).build()
    ).build()).build();
```

次のように書くことができます：

```java
Baz.Builder builder = baz.toBuilder();
builder.getBarBuilder().getFooBuilder().setVal(10);
baz = builder.build();
```

### ネストされた型 {#nested}

メッセージは別のメッセージの内部で宣言することができます。例：

```proto
message Foo {
  message Bar { }
}
```

この場合、コンパイラは`Bar`を`Foo`の内部クラスとして生成します。

## フィールド {#fields}

前のセクションで説明したメソッドに加えて、プロトコルバッファコンパイラは、`.proto`ファイルで定義された各フィールドに対して、アクセサメソッドのセットを生成します。フィールド値を読み取るメソッドは、メッセージクラスとそれに対応するビルダーの両方で定義されています。値を変更するメソッドはビルダーのみで定義されています。

メソッド名は常にキャメルケースの命名規則を使用します。たとえ`.proto`ファイルのフィールド名がアンダースコアを使用した小文字であっても、ケース変換は次のように機能します：

*   各アンダースコアについて、アンダースコアは削除され、次の文字が大文字になります。
*   名前に接頭辞が付いている場合（例： "get"）、最初の文字は大文字になります。それ以外の場合は小文字になります。
*   メソッド名の各数字の最後の数字の後に続く文字は大文字になります。

したがって、フィールド `foo_bar_baz` は `fooBarBaz` になります。`get` と接頭辞が付いている場合は、`getFooBarBaz` になります。そして、`foo_ba23r_baz` は `fooBa23RBaz` になります。

アクセサメソッドだけでなく、コンパイラは各フィールドに対して整数定数を生成し、そのフィールド番号を含みます。定数名はフィールド名を大文字に変換して `_FIELD_NUMBER` が続きます。たとえば、フィールド `optional int32 foo_bar = 5;` が与えられた場合、コンパイラは定数 `public static final int FOO_BAR_FIELD_NUMBER = 5;` を生成します。

### 単数フィールド（proto2） {#singular-proto2}

次のいずれかのフィールド定義がある場合：

```proto
optional int32 foo = 1;
required int32 foo = 1;
```

コンパイラは、メッセージクラスとそのビルダーの両方に次のアクセサメソッドを生成します：

-   `boolean hasFoo()`: フィールドが設定されている場合は `true` を返します。
-   `int getFoo()`: フィールドの現在の値を返します。フィールドが設定されていない場合は、デフォルト値を返します。

コンパイラは、メッセージのビルダーにのみ次のメソッドを生成します：

-   `Builder setFoo(int value)`: フィールドの値を設定します。これを呼び出した後、`hasFoo()` は `true` を返し、`getFoo()` は `value` を返します。
-   `Builder clearFoo()`: フィールドの値をクリアします。これを呼び出した後、`hasFoo()` は `false` を返し、`getFoo()` はデフォルト値を返します。

他の単純なフィールドタイプについては、対応する Java タイプは、
[スカラー値タイプテーブル](/programming-guides/proto2#scalar)
に従って選択されます。メッセージおよび列挙型の場合、値のタイプはメッセージまたは列挙型クラスに置き換えられます。

#### 埋め込みメッセージフィールド {#embedded-message-proto2}

メッセージ型の場合、`setFoo()` はメッセージのビルダータイプのインスタンスもパラメータとして受け入れます。これは、ビルダーで `.build()` を呼び出して結果をメソッドに渡すのと同等のショートカットです。

もしフィールドが設定されていない場合、`getFoo()` はそのフィールドのどれも設定されていない Foo インスタンスを返します（おそらく `Foo.getDefaultInstance()` によって返されるインスタンスです）。

さらに、コンパイラは、メッセージ型の関連するサブビルダーにアクセスできる 2 つのアクセサメソッドを生成します。次のメソッドは、メッセージクラスとそのビルダーの両方に生成されます。

- `FooOrBuilder getFooOrBuilder()`: フィールドのビルダーを返します。既に存在する場合はそのビルダーを、存在しない場合はメッセージを返します。このメソッドをビルダーで呼び出しても、フィールドのサブビルダーは作成されません。

コンパイラは、次のメソッドをメッセージのビルダーだけに生成します。

- `Builder getFooBuilder()`: フィールドのビルダーを返します。

### 単数フィールド（proto3） {#singular-proto3}

このフィールド定義に対して:

```proto
int32 foo = 1;
```

コンパイラは、メッセージクラスとそのビルダーの両方に次のアクセサメソッドを生成します。

- `int getFoo()`: フィールドの現在の値を返します。フィールドが設定されていない場合は、フィールドの型のデフォルト値を返します。

コンパイラは、次のメソッドをメッセージのビルダーだけに生成します。

- `Builder setFoo(int value)`: フィールドの値を設定します。これを呼び出した後、`getFoo()` は `value` を返します。
- `Builder clearFoo()`: フィールドの値をクリアします。これを呼び出した後、`getFoo()` はフィールドの型のデフォルト値を返します。

他の単純なフィールド型については、対応する Java 型が
[スカラー値の種類テーブル](/programming-guides/proto2#scalar)
に従って選択されます。メッセージおよび列挙型の場合、値の型はメッセージまたは列挙型のクラスに置き換えられます。

#### 埋め込みメッセージフィールド {#embedded-message-proto3}

メッセージフィールド型の場合、メッセージクラスとそのビルダーの両方に追加のアクセサメソッドが生成されます。

- `boolean hasFoo()`: フィールドが設定されている場合は `true` を返します。

`setFoo()` は、メッセージのビルダータイプのインスタンスもパラメータとして受け入れます。これは、ビルダーで `.build()` を呼び出してその結果をメソッドに渡すのと同等のショートカットです。

もしフィールドが設定されていない場合、`getFoo()` はそのフィールドが設定されていない Foo インスタンスを返します（おそらく `Foo.getDefaultInstance()` によって返されるインスタンスです）。

さらに、コンパイラは、メッセージ型の関連するサブビルダーにアクセスするための2つのアクセサメソッドを生成します。次のメソッドは、メッセージクラスとそのビルダーの両方に生成されます。

- `FooOrBuilder getFooOrBuilder()`: フィールドのビルダーを返します。既に存在する場合はそのビルダーを、存在しない場合はメッセージを返します。このメソッドをビルダーで呼び出しても、フィールドのサブビルダーは作成されません。

コンパイラは、次のメソッドをメッセージのビルダーのみで生成します。

- `Builder getFooBuilder()`: フィールドのビルダーを返します。

#### 列挙型フィールド {#enum-proto3}

列挙型フィールドの場合、メッセージクラスとそのビルダーの両方に追加のアクセサメソッドが生成されます。

- `int getFooValue()`: 列挙型の整数値を返します。

コンパイラは、次の追加のメソッドをメッセージのビルダーのみで生成します。

- `Builder setFooValue(int value)`: 列挙型の整数値を設定します。

さらに、もし列挙型の値が不明な場合、`getFoo()` は `UNRECOGNIZED` を返します。これは、proto3 コンパイラによって生成される[列挙型](#enum)に追加された特別な値です。

### 繰り返しフィールド {#repeated}

このフィールド定義に対して:

```proto
repeated string foos = 1;
```

コンパイラは、メッセージクラスとそのビルダーの両方に次のアクセサメソッドを生成します。

- `int getFoosCount()`: 現在のフィールド内の要素数を返します。
- `String getFoos(int index)`: 指定されたゼロベースのインデックスの要素を返します。
- `ProtocolStringList getFoosList()`: フィールド全体を `ProtocolStringList` として返します。フィールドが設定されていない場合は空のリストを返します。

コンパイラは、次のメソッドをメッセージのビルダーのみで生成します。

- `Builder setFoos(int index, String value)`: 指定されたゼロベースのインデックスの要素の値を設定します。
- `Builder addFoos(String value)`: 指定された値で新しい要素をフィールドに追加します。
- `Builder addAllFoos(Iterable<? extends String> value)`: 指定された `Iterable` のすべての要素をフィールドに追加します。
- `Builder clearFoos()`: フィールドからすべての要素を削除します。これを呼び出した後、`getFoosCount()` はゼロを返します。

#### 繰り返し埋め込みメッセージフィールド {#repeated-embedded}
メッセージ型の場合、`setFoos()` および `addFoos()` は、メッセージのビルダータイプのインスタンスもパラメータとして受け入れます。これは、ビルダーで `.build()` を呼び出してその結果をメソッドに渡すのと同等のショートカットです。さらに、追加の生成されたメソッドがあります:

- `Builder addFoos(int index, Field value)`: ゼロベースの指定されたインデックスに新しい要素を挿入します。その位置に現在存在する要素（あれば）およびその後続の要素を右にシフトします（インデックスに1を追加します）。

さらに、コンパイラは、メッセージクラスとそのビルダーの両方に、メッセージ型の関連するサブビルダーにアクセスできるようにするための次の追加のアクセサメソッドを生成します:

- `FooOrBuilder getFoosOrBuilder(int index)`: 指定された要素のビルダーを返します。要素が既に存在する場合は、`IndexOutOfBoundsException` をスローします。これがメッセージクラスから呼び出された場合、ビルダーを作成しません。
- `List<FooOrBuilder> getFoosOrBuilderList()`: フィールド全体を、利用可能なビルダーの変更不可能なリストとして返します。利用できない場合はメッセージを返します。これがメッセージクラスから呼び出された場合、変更不可能なビルダーのリストではなく、メッセージの変更不可能なリストを常に返します。

コンパイラは、次のメソッドをメッセージのビルダーだけで生成します:

- `Builder getFoosBuilder(int index)`: 指定されたインデックスの要素のビルダーを返します。インデックスが範囲外の場合は、`IndexOutOfBoundsException` をスローします。
- `Builder addFoosBuilder(int index)`: 指定されたインデックスの繰り返しメッセージのデフォルトメッセージインスタンスのビルダーを挿入して返します。既存のエントリは、挿入されたビルダーのためのスペースを作るためにより高いインデックスにシフトされます。
- `Builder addFoosBuilder()`: 繰り返しメッセージのデフォルトメッセージインスタンスのビルダーを追加して返します。
- `Builder removeFoos(int index)`: 指定されたゼロベースのインデックスの要素を削除します。
- `List<Builder> getFoosBuilderList()`: フィールド全体を、ビルダーの変更不可能なリストとして返します。

#### 繰り返し列挙フィールド（proto3 のみ） {#repeated-enum-proto3}

コンパイラは、メッセージクラスとそのビルダーの両方に、以下の追加メソッドを生成します：

-   `int getFoosValue(int index)`: 指定されたインデックスの列挙型の整数値を返します。
-   `List<java.lang.Integer> getFoosValueList()`: フィールド全体を Integer のリストとして返します。

コンパイラは、メッセージのビルダーにのみ、以下の追加メソッドを生成します：

-   `Builder setFoosValue(int index, int value)`: 指定されたインデックスの列挙型の整数値を設定します。

#### 名前の競合 {#conflicts}

もう1つの繰り返しでないフィールドが、繰り返しフィールドの生成されたメソッドのいずれかと競合する名前を持っている場合、両方のフィールド名には protobuf フィールド番号が末尾に追加されます。

次のフィールド定義の場合：

```proto
int32 foos_count = 1;
repeated string foos = 2;
```

コンパイラは、まずそれらを次のように名前変更します：

```proto
int32 foos_count_1 = 1;
repeated string foos_2 = 2;
```

その後、アクセサメソッドが上記のように生成されます。

<a id="oneof"></a>

### Oneof フィールド {#oneof-fields}

この oneof フィールドの定義の場合：

```proto
oneof choice {
    int32 foo_int = 4;
    string foo_string = 9;
    ...
}
```

`choice` oneof 内のすべてのフィールドは、値に対して単一のプライベートフィールドを使用します。さらに、プロトコルバッファコンパイラは、次のように oneof ケース用の Java 列挙型を生成します：

```java
public enum ChoiceCase
        implements com.google.protobuf.Internal.EnumLite {
      FOO_INT(4),
      FOO_STRING(9),
      ...
      CHOICE_NOT_SET(0);
      ...
    };
```

この列挙型の値には、次の特別なメソッドがあります：

-   `int getNumber()`: .proto ファイルで定義されたオブジェクトの数値を返します。
-   `static ChoiceCase forNumber(int value)`: 指定された数値に対応する列挙オブジェクトを返します（その他の数値の場合は `null` を返します）。

コンパイラは、メッセージクラスとそのビルダーの両方に、以下のアクセサメソッドを生成します：

-   `boolean hasFooInt()`: oneof ケースが `FOO` の場合は `true` を返します。
-   `int getFooInt()`: oneof ケースが `FOO` の場合は `foo` の現在の値を返します。それ以外の場合は、このフィールドのデフォルト値を返します。
-   `ChoiceCase getChoiceCase()`: どのフィールドが設定されているかを示す列挙型を返します。どれも設定されていない場合は `CHOICE_NOT_SET` を返します。

コンパイラは、メッセージのビルダー内で以下のメソッドのみを生成します：

-   `Builder setFooInt(int value)`: `Foo` をこの値に設定し、oneof ケースを `FOO` に設定します。これを呼び出した後、`hasFooInt()` は `true` を返し、`getFooInt()` は `value` を返し、`getChoiceCase()` は `FOO` を返します。
-   `Builder clearFooInt()`:
    -   oneof ケースが `FOO` でない場合、何も変更されません。
    -   oneof ケースが `FOO` の場合、`Foo` を null に設定し、oneof ケースを `FOO_NOT_SET` に設定します。これを呼び出した後、`hasFooInt()` は `false` を返し、`getFooInt()` はデフォルト値を返し、`getChoiceCase()` は `FOO_NOT_SET` を返します。
-   `Builder.clearChoice()`: `choice` の値をリセットし、ビルダーを返します。

他の単純なフィールドタイプについては、対応する Java タイプは、[スカラー値タイプテーブル](/programming-guides/proto2#scalar)に従って選択されます。メッセージおよび列挙型の場合、値のタイプはメッセージまたは列挙型クラスに置き換えられます。

### マップフィールド {#map-fields}

次のマップフィールド定義に対して：

```proto
map<int32, int32> weight = 1;
```

コンパイラは、メッセージクラスとそのビルダーの両方に、以下のアクセサメソッドを生成します：

-   `Map<Integer, Integer> getWeightMap();`: 変更できない `Map` を返します。
-   `int getWeightOrDefault(int key, int default);`: キーの値を返すか、存在しない場合はデフォルト値を返します。
-   `int getWeightOrThrow(int key);`: キーの値を返し、存在しない場合は IllegalArgumentException をスローします。
-   `boolean containsWeight(int key);`: このフィールドにキーが存在するかを示します。
-   `int getWeightCount();`: マップ内の要素数を返します。

コンパイラは、メッセージのビルダー内で以下のメソッドのみを生成します：

-   `Builder putWeight(int key, int value);`: このフィールドに重みを追加します。
-   `Builder putAllWeight(Map<Integer, Integer> value);`: 指定されたマップ内のすべてのエントリをこのフィールドに追加します。
-   `Builder removeWeight(int key);`: このフィールドから重みを削除します。
-   `Builder clearWeight();`: このフィールドからすべての重みを削除します。
-   `@Deprecated Map<Integer, Integer> getMutableWeight();`: 変更可能な `Map` を返します。このメソッドへの複数回の呼び出しは異なるマップインスタンスを返す可能性があります。返されたマップ参照は、ビルダーへの後続のメソッド呼び出しによって無効化される可能性があります。

#### メッセージ値マップフィールド {#message-value-map-fields}

値としてメッセージタイプを持つマップの場合、コンパイラはメッセージのビルダーに追加のメソッドを生成します:

-   `Foo.Builder putFooBuilderIfAbsent(int key);`: `key` がマッピング内に存在することを保証し、存在しない場合は新しい `Foo.Builder` を挿入します。返された `Foo.Builder` への変更は最終的なメッセージに反映されます。

## Any {#any-fields}

次のような [`Any`](/programming-guides/proto3#any) フィールドがある場合:

```proto
import "google/protobuf/any.proto";

message ErrorStatus {
  string message = 1;
  google.protobuf.Any details = 2;
}
```

生成されたコードでは、`details` フィールドのゲッターは `com.google.protobuf.Any` のインスタンスを返します。これには `Any` の値をパックおよびアンパックするための次の特別なメソッドが提供されます:

```java
class Any {
  // Packs the given message into an Any using the default type URL
  // prefix “type.googleapis.com”.
  public static Any pack(Message message);
  // Packs the given message into an Any using the given type URL
  // prefix.
  public static Any pack(Message message,
                         String typeUrlPrefix);

  // Checks whether this Any message’s payload is the given type.
  public <T extends Message> boolean is(class<T> clazz);

  // Unpacks Any into the given message type. Throws exception if
  // the type doesn’t match or parsing the payload has failed.
  public <T extends Message> T unpack(class<T> clazz)
      throws InvalidProtocolBufferException;
}
```

## 列挙型 {#enum}

次のような列挙型の定義がある場合:

```proto
enum Foo {
  VALUE_A = 0;
  VALUE_B = 5;
  VALUE_C = 1234;
}
```

プロトコルバッファコンパイラは、同じ値のセットを持つ Java 列挙型 `Foo` を生成します。proto3 を使用している場合、列挙型に特別な値 `UNRECOGNIZED` も追加されます。生成された列挙型の値には次の特別なメソッドがあります:

-   `int getNumber()`: オブジェクトの数値値を `.proto` ファイルで定義されたものとして返します。
-   `EnumValueDescriptor getValueDescriptor()`: 値の記述子を返し、値の名前、番号、タイプに関する情報を含みます。
-   `EnumDescriptor getDescriptorForType()`: 列挙型の記述子を返し、各定義された値に関する情報などを含みます。

さらに、`Foo` 列挙型には次の静的メソッドが含まれています:

-   `static Foo forNumber(int value)`: 指定された数値値に対応する列挙オブジェクトを返します。対応する列挙オブジェクトがない場合は null を返します。
-   `static Foo valueOf(int value)`: 指定された数値値に対応する列挙オブジェクトを返します。このメソッドは `forNumber(int value)` の代わりに非推奨となり、今後のリリースで削除されます。
-   `static Foo valueOf(EnumValueDescriptor descriptor)`: 指定された値記述子に対応する列挙オブジェクトを返します。`valueOf(int)` よりも高速かもしれません。proto3 では、不明な値記述子が渡された場合は `UNRECOGNIZED` を返します。
-   `EnumDescriptor getDescriptor()`: 列挙型の記述子を返し、各定義された値に関する情報などを含みます。(これは `getDescriptorForType()` と異なり、静的メソッドである点が異なります。)

整数定数は、各enum値に対してサフィックス `_VALUE` が付いて生成されます。

`.proto` 言語では、複数のenumシンボルが同じ数値を持つことができます。同じ数値を持つシンボルは同義語です。例えば：

```proto
enum Foo {
  BAR = 0;
  BAZ = 0;
}
```

この場合、`BAZ` は `BAR` の同義語です。Javaでは、`BAZ` は次のように静的な最終フィールドとして定義されます：

```java
static final Foo BAZ = BAR;
```

したがって、`BAR` と `BAZ` は等しく、`BAZ` は switch 文には決して現れるべきではありません。コンパイラは、特定の数値で定義された最初のシンボルをそのシンボルの「正準」バージョンとして常に選択し、同じ数値を持つすべての後続のシンボルは単なるエイリアスです。

enumはメッセージ型の中にネストして定義することができます。コンパイラは、そのメッセージ型のクラス内にネストされたJava enum定義を生成します。

**注意: Javaコードを生成する際に、protobufのenum内の値の最大数は驚くほど低い場合があります**&mdash;最悪の場合、最大で約1,700以上の値です。この制限は、Javaバイトコードのメソッドごとのサイズ制限に起因し、Javaの実装、protobufスイートの異なるバージョン、および`.proto`ファイルで設定されたオプションによって異なります。

## 拡張機能 (proto2のみ) {#extension}

拡張範囲を持つメッセージがある場合：

```proto
message Foo {
  extensions 100 to 199;
}
```

プロトコルバッファコンパイラは、通常の `GeneratedMessage` の代わりに `Foo` を `GeneratedMessage.ExtendableMessage` に拡張します。同様に、`Foo` のビルダーは `GeneratedMessage.ExtendableBuilder` を拡張します。これらの基本型を名前で参照してはいけません（`GeneratedMessage` は実装の詳細と見なされます）。ただし、これらのスーパークラスは、拡張を操作するために使用できる追加のメソッドをいくつか定義しています。

特に `Foo` と `Foo.Builder` は、`hasExtension()`、`getExtension()`、`getExtensionCount()` メソッドを継承します。さらに、`Foo.Builder` は `setExtension()` と `clearExtension()` メソッドを継承します。これらのメソッドの各々は、第1パラメータとして拡張識別子（後述）を取り、残りのパラメータと戻り値は、同じ型の通常の（拡張でない）フィールドのために生成される対応するアクセサメソッドと全く同じです。

与えられた拡張定義：

```proto
extend Foo {
  optional int32 bar = 123;
}
```

プロトコルバッファコンパイラは、「拡張識別子」と呼ばれる `bar` を生成し、
この拡張にアクセスするために `Foo` の拡張アクセサを使用できます。以下のように：

```java
Foo foo =
  Foo.newBuilder()
     .setExtension(bar, 1)
     .build();
assert foo.hasExtension(bar);
assert foo.getExtension(bar) == 1;
```

（拡張識別子の正確な実装は複雑で、ジェネリックの魔法的な使用を含みますが、
拡張識別子がどのように機能するかを気にする必要はありません。）

`bar` は、上記のように、`.proto` ファイルのラッパークラスの静的フィールドとして宣言されます。
この例では、ラッパークラス名を省略しています。

拡張は、他の型のスコープ内で宣言されることがあり、生成されたシンボル名に接頭辞を付けるために使用されます。
例えば、一般的なパターンは、メッセージをフィールドの宣言内部で拡張することです：

```proto
message Baz {
  extend Foo {
    optional Baz foo_ext = 124;
  }
}
```

この場合、識別子 `foo_ext` と型 `Baz` を持つ拡張が `Baz` の宣言内で宣言され、
`foo_ext` を参照するには `Baz.` 接頭辞を追加する必要があります：

```java
Baz baz = createMyBaz();
Foo foo =
  Foo.newBuilder()
     .setExtension(Baz.fooExt, baz)
     .build();
assert foo.hasExtension(Baz.fooExt);
assert foo.getExtension(Baz.fooExt) == baz;
```

拡張を持つ可能性のあるメッセージを解析する際には、
解析できるようにしたい任意の拡張を登録した [`ExtensionRegistry`](/reference/java/api-docs/com/google/protobuf/ExtensionRegistry.html)
を提供する必要があります。
そうでない場合、これらの拡張は未知のフィールドのように扱われ、
拡張を観察するメソッドは存在しないかのように振る舞います。

```java
ExtensionRegistry registry = ExtensionRegistry.newInstance();
registry.add(Baz.fooExt);
Foo foo = Foo.parseFrom(input, registry);
assert foo.hasExtension(Baz.fooExt);
```

```java
ExtensionRegistry registry = ExtensionRegistry.newInstance();
Foo foo = Foo.parseFrom(input, registry);
assert foo.hasExtension(Baz.fooExt) == false;
```

## サービス {#service}

`.proto` ファイルに次の行が含まれている場合：

```proto
option java_generic_services = true;
```

その後、プロトコルバッファコンパイラは、このセクションで説明されているように、
ファイル内で見つかったサービス定義に基づいてコードを生成します。
ただし、生成されたコードは特定の RPC システムに結びついていないため、
1 つのシステムに合わせたコードよりも間接的なレベルが必要となります。
このコードを生成したくない場合は、ファイルにこの行を追加してください：

```proto
option java_generic_services = false;
```

上記のいずれも指定されていない場合、ジェネリックサービスは非推奨となるため、オプションのデフォルトは `false` になります（2.4.0より前では、オプションのデフォルトは `true` でした）。

`.proto` 言語のサービス定義に基づく RPC システムは、システムに適したコードを生成するための[プラグイン](/reference/cpp/api-docs/google.protobuf.compiler.plugin.pb)を提供する必要があります。これらのプラグインは、抽象サービスが無効になっていることを要求する可能性が高く、そのために彼ら自身の同じ名前のクラスを生成できます。プラグインはバージョン2.3.0（2010年1月）で新しく追加されました。

このセクションの残りの部分では、抽象サービスが有効になっている場合にプロトコルバッファコンパイラが生成する内容について説明します。

### インターフェース {#interface}

以下のサービス定義が与えられた場合：

```proto
service Foo {
  rpc Bar(FooRequest) returns(FooResponse);
}
```

プロトコルバッファコンパイラは、このサービスを表す抽象クラス `Foo` を生成します。`Foo` には、サービス定義で定義された各メソッドに対応する抽象メソッドがあります。この場合、メソッド `Bar` は次のように定義されています：

```java
abstract void bar(RpcController controller, FooRequest request,
                  RpcCallback<FooResponse> done);
```

これらのパラメータは、`Service.CallMethod()` のパラメータと同等ですが、`method` 引数は暗黙的であり、`request` と `done` は正確な型を指定しています。

`Foo` は `Service` インターフェースを継承しています。プロトコルバッファコンパイラは、`Service` のメソッドの実装を自動的に生成します：

-   `getDescriptorForType`: サービスの `ServiceDescriptor` を返します。
-   `callMethod`: 提供されたメソッド記述子に基づいて呼び出されるメソッドを直接呼び出し、リクエストメッセージとコールバックを正しい型にダウンキャストします。
-   `getRequestPrototype` および `getResponsePrototype`: 指定されたメソッドの正しい型のデフォルトインスタンスを返します。

以下の静的メソッドも生成されます：

-   `static ServiceDescriptor getServiceDescriptor()`: タイプの記述子を返し、このサービスにどのメソッドがあるか、それらの入力および出力の型が何かについての情報を含んでいます。

`Foo`にはネストされたインターフェース`Foo.Interface`も含まれます。これは、サービス定義の各メソッドに対応するメソッドを含む純粋なインターフェースです。ただし、このインターフェースは`Service`インターフェースを拡張しません。これは問題です。なぜなら、RPCサーバーの実装は通常、特定のサービスではなく抽象的な`Service`オブジェクトを使用して書かれているからです。この問題を解決するために、`Foo.Interface`を実装するオブジェクト`impl`がある場合、`Foo.newReflectiveService(impl)`を呼び出して、単に`impl`に委譲し、`Service`を実装する`Foo`のインスタンスを構築できます。

要約すると、独自のサービスを実装する際には、2つのオプションがあります。

-   `Foo`をサブクラス化し、適切にそのメソッドを実装し、そのサブクラスのインスタンスを直接RPCサーバーの実装に渡す。これが通常最も簡単ですが、一部の人はそれをより「純粋」とは考えません。
-   `Foo.Interface`を実装し、`Foo.newReflectiveService(Foo.Interface)`を使用してそれをラップする`Service`を構築し、そのラッパーをRPC実装に渡す。

### スタブ {#stub}

プロトコルバッファコンパイラは、サービスインターフェースごとに「スタブ」実装も生成します。これは、サービスを実装するサーバーにリクエストを送信したいクライアントが使用するものです。`Foo`サービス（上記）のスタブ実装`Foo.Stub`はネストされたクラスとして定義されます。

`Foo.Stub`は、次のメソッドも実装する`Foo`のサブクラスです。

-   `Foo.Stub(RpcChannel channel)`: 指定されたチャンネルにリクエストを送信する新しいスタブを構築します。
-   `RpcChannel getChannel()`: このスタブのチャンネルを返します。これはコンストラクタに渡されたものです。

スタブは、各サービスのメソッドをチャンネルをラップする方法で追加で実装します。メソッドの1つを呼び出すと、単に`channel.callMethod()`が呼び出されます。

プロトコルバッファライブラリにはRPCの実装は含まれていません。ただし、生成されたサービスクラスを任意のRPC実装に接続するために必要なすべてのツールが含まれています。`RpcChannel`と`RpcController`の実装を提供するだけです。

### ブロッキングインターフェース {#blocking}

上記で説明したRPCクラスはすべてノンブロッキングセマンティクスを持っています。メソッドを呼び出すときには、メソッドが完了するとコールバックオブジェクトが呼び出されます。メソッドが完了するまで戻らないブロッキングセマンティクスを使用すると、コードを書くのが簡単になりますが、スケーラブル性が低くなる可能性があります。このため、プロトコルバッファコンパイラは、サービスクラスのブロッキングバージョンも生成します。`Foo.BlockingInterface`は、`Foo.Interface`と同等ですが、各メソッドが単に結果を返すだけでコールバックを呼び出さない点が異なります。例えば、`bar`は次のように定義されています：

```java
abstract FooResponse bar(RpcController controller, FooRequest request)
                         throws ServiceException;
```

ノンブロッキングサービスに対応するように、`Foo.newReflectiveBlockingService(Foo.BlockingInterface)`は、いくつかの`Foo.BlockingInterface`をラップした`BlockingService`を返します。最後に、`Foo.BlockingStub`は、リクエストを特定の`BlockingRpcChannel`に送信する`Foo.BlockingInterface`のスタブ実装を返します。

## プラグイン挿入ポイント {#plugins}

Javaコードジェネレータの出力を拡張したい[コードジェネレータプラグイン](/reference/cpp/api-docs/google.protobuf.compiler.plugin.pb)は、次の挿入ポイント名を使用して次のタイプのコードを挿入できます。

-   `outer_class_scope`：ファイルのラッパークラスに属するメンバー宣言。
-   `class_scope:TYPENAME`：メッセージクラスに属するメンバー宣言。`TYPENAME`は完全なプロト名であり、例えば`package.MessageType`です。
-   `builder_scope:TYPENAME`：メッセージのビルダークラスに属するメンバー宣言。`TYPENAME`は完全なプロト名であり、例えば`package.MessageType`です。
-   `enum_scope:TYPENAME`：列挙型クラスに属するメンバー宣言。`TYPENAME`は完全なプロト列挙型名であり、例えば`package.EnumType`です。
-   `message_implements:TYPENAME`：メッセージクラスのクラス実装宣言。`TYPENAME`は完全なプロト名であり、例えば`package.MessageType`です。
-   `builder_implements:TYPENAME`：ビルダークラスのクラス実装宣言。`TYPENAME`は完全なプロト名であり、例えば`package.MessageType`です。

生成されたコードには、定義された型名と競合する可能性があるため、importステートメントを含めるべきではありません。代わりに、外部クラスを参照する際には常に完全修飾名を使用する必要があります。

Javaコードジェネレーターで出力ファイル名を決定するロジックはかなり複雑です。すべてのケースをカバーしていることを確認するには、`protoc`のソースコード、特に`java_headers.cc`を確認することをお勧めします。

標準のコードジェネレーターによって宣言されたプライベートクラスメンバーに依存するコードを生成しないでください。これらの実装の詳細はProtocol Buffersの将来のバージョンで変更される可能性があるためです。

## ユーティリティクラス {#utility-classes}

Protocol Buffersは、メッセージ比較、JSON変換、および[well-known types（一般的なユースケース向けの事前定義されたプロトコルバッファメッセージ）](/reference/protobuf/google.protobuf)の操作に使用する[ユーティリティクラス](/reference/java/api-docs/com/google/protobuf/util/package-summary.html)を提供します。
