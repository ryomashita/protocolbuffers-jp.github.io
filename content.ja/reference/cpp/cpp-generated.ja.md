+++
title = "C++ 生成コードガイド"
weight = 510
linkTitle = "生成コードガイド"
description = "任意のプロトコル定義に対してプロトコルバッファコンパイラが生成する C++ コードを正確に説明します。"
type = "docs"
+++

proto2 と proto3 で生成されるコードの違いは強調されています - これらの違いは、このドキュメントで説明されている生成されたコードにありますが、ベースのメッセージクラス/インターフェースは両バージョンで同じです。
このドキュメントを読む前に、
[proto2 言語ガイド](/programming-guides/proto2)
および/または
[proto3 言語ガイド](/programming-guides/proto3)
を読むべきです。

## コンパイラの呼び出し {#invocation}

プロトコルバッファコンパイラは、`--cpp_out=` コマンドラインフラグを使用して呼び出されるときに C++ 出力を生成します。`--cpp_out=` オプションのパラメータは、コンパイラが C++ 出力を書き込むディレクトリです。コンパイラは、各 `.proto` ファイルの入力に対してヘッダーファイルと実装ファイルを作成します。出力ファイルの名前は、`.proto` ファイルの名前を取り、次の 2 つの変更を加えて計算されます:

-   拡張子（`.proto`）は、ヘッダーファイルまたは実装ファイルに対してそれぞれ `.pb.h` または `.pb.cc` に置き換えられます。
-   proto パス（`--proto_path=` または `-I` コマンドラインフラグで指定）は、出力パス（`--cpp_out=` フラグで指定）に置き換えられます。

例えば、次のようにコンパイラを呼び出すとします:

```shell
protoc --proto_path=src --cpp_out=build/gen src/foo.proto src/bar/baz.proto
```

コンパイラはファイル `src/foo.proto` と `src/bar/baz.proto` を読み込み、4 つの出力ファイルを生成します: `build/gen/foo.pb.h`, `build/gen/foo.pb.cc`, `build/gen/bar/baz.pb.h`, `build/gen/bar/baz.pb.cc`。コンパイラは必要に応じてディレクトリ `build/gen/bar` を自動的に作成しますが、`build` または `build/gen` は作成しません。それらはすでに存在している必要があります。

## パッケージ {#package}

`.proto` ファイルに `package` 宣言が含まれている場合、ファイル全体が対応する C++ 名前空間に配置されます。例えば、次の `package` 宣言がある場合:

```proto
package foo.bar;
```

ファイル内のすべての宣言は `foo::bar` ネームスペースに存在します。

## メッセージ {#message}

単純なメッセージ宣言が与えられた場合:

```proto
message Foo {}
```

プロトコルバッファコンパイラは `Foo` というクラスを生成します。このクラスは
[`google::protobuf::Message`](/reference/cpp/api-docs/google.protobuf.message)
を公開的に継承します。このクラスは具象クラスであり、純粋仮想メソッドは未実装のままです。
`Message` で仮想的なメソッドであるが純粋仮想ではないメソッドは、最適化モードに応じて
`Foo` によってオーバーライドされるかどうかが異なります。デフォルトでは、`Foo` は最大の速度を実現するためにすべてのメソッドの専用バージョンを実装します。ただし、`.proto` ファイルに次の行が含まれている場合:

```proto
option optimize_for = CODE_SIZE;
```

その場合、`Foo` は機能するために必要な最小限のメソッドセットのみをオーバーライドし、残りのメソッドにはリフレクションベースの実装を依存します。これにより生成されるコードのサイズが大幅に削減されますが、パフォーマンスも低下します。
また、`.proto` ファイルに次の行が含まれている場合:

```proto
option optimize_for = LITE_RUNTIME;
```

その場合、`Foo` はすべてのメソッドの高速な実装を含みますが、
[`google::protobuf::MessageLite`](/reference/cpp/api-docs/google.protobuf.message_lite)
インターフェースを実装します。このインターフェースには `Message` の一部のメソッドしか含まれておらず、記述子やリフレクションをサポートしません。ただし、このモードでは生成されたコードは `libprotobuf-lite.so`
(`libprotobuf.lib` は Windows 用) にリンクする必要があります。"lite" ライブラリはフルライブラリよりもはるかに小さく、モバイル電話などのリソースに制約のあるシステムに適しています。

独自の `Foo` サブクラスを作成すべきではありません。このクラスをサブクラス化して仮想メソッドをオーバーライドすると、オーバーライドが無視される可能性があります。多くの生成されたメソッド呼び出しはパフォーマンスを向上させるために仮想化解除されるためです。

`Message` インターフェースは、バイナリ文字列からのパースやシリアライズを含む、メッセージ全体のチェック、操作、読み取り、書き込みを行うメソッドを定義します。
```

-   `bool ParseFromString(const string& data)`: 指定されたシリアライズされたバイナリ文字列（ワイヤーフォーマットとも呼ばれる）からメッセージをパースします。{/*examples*/}
-   `bool SerializeToString(string* output) const`: 指定されたメッセージをバイナリ文字列にシリアライズします。{/*examples*/}
-   `string DebugString()`: proto の `text_format` 表現を返す文字列を返します（デバッグ目的でのみ使用するべきです）。{/*examples*/}

これらのメソッドに加えて、`Foo` クラスは以下のメソッドを定義しています:

-   `Foo()`: デフォルトコンストラクタ。{/*examples*/}
-   `~Foo()`: デフォルトデストラクタ。{/*examples*/}
-   `Foo(const Foo& other)`: コピーコンストラクタ。{/*examples*/}
-   `Foo(Foo&& other)`: ムーブコンストラクタ。{/*examples*/}
-   `Foo& operator=(const Foo& other)`: 代入演算子。{/*examples*/}
-   `Foo& operator=(Foo&& other)`: ムーブ代入演算子。{/*examples*/}
-   `void Swap(Foo* other)`: 別のメッセージと内容を交換します。{/*examples*/}
-   `const UnknownFieldSet& unknown_fields() const`: このメッセージをパースする際に遭遇した未知のフィールドのセットを返します。`.proto` ファイルで `option optimize_for = LITE_RUNTIME` が指定されている場合、戻り値の型が `std::string&` に変わります。{/*examples*/}
-   `UnknownFieldSet* mutable_unknown_fields()`: このメッセージをパースする際に遭遇した未知のフィールドのセットへのポインタを返します。`.proto` ファイルで `option optimize_for = LITE_RUNTIME` が指定されている場合、戻り値の型が `std::string*` に変わります。{/*examples*/}

このクラスは以下の静的メソッドも定義しています:

-   `static const Descriptor* descriptor()`: タイプの記述子を返します。これには、フィールドの情報やその型などが含まれています。これは [reflection](/reference/cpp/api-docs/google.protobuf.message#Reflection) と組み合わせてフィールドをプログラムで調査するために使用できます。{/*examples*/}
-   `static const Foo& default_instance()`: `Foo` の const シングルトンインスタンスを返します。これは新しく構築された `Foo` インスタンスと同一であり、すべての単一フィールドが未設定であり、すべての繰り返しフィールドが空です。メッセージのデフォルトインスタンスは、その `New()` メソッドを呼び出すことでファクトリとして使用できます。{/*examples*/}

### 生成されたファイル名 {#generated-filenames}

[予約語](https://github.com/protocolbuffers/protobuf/blob/main/src/google/protobuf/compiler/cpp/helpers.cc#L4)
は生成された出力にアンダースコアが付加されます。

たとえば、次の proto3 定義構文:

```proto
message MyMessage {
  string false = 1;
  string myFalse = 2;
}
```

は次の部分的な出力を生成します:

```cpp
  void clear_false_() ;
  const std::string& false_() const;
  void set_false_(Arg_&& arg, Args_... args);
  std::string* mutable_false_();
  PROTOBUF_NODISCARD std::string* release_false_();
  void set_allocated_false_(std::string* ptr);

  void clear_myfalse() ;
  const std::string& myfalse() const;
  void set_myfalse(Arg_&& arg, Args_... args);
  std::string* mutable_myfalse();
  PROTOBUF_NODISCARD std::string* release_myfalse();
  void set_allocated_myfalse(std::string* ptr);
```

### ネストされた型 {#nested-types}

メッセージは別のメッセージの内部で宣言されることができます。たとえば:

```proto
message Foo {
  message Bar {}
}
```

この場合、コンパイラは `Foo` と `Foo_Bar` の2つのクラスを生成します。さらに、コンパイラは `Foo` の内部に次のように typedef を生成します:

```cpp
typedef Foo_Bar Bar;
```

これにより、ネストされた型のクラスを、ネストされたクラス `Foo::Bar` であるかのように使用できます。ただし、C++ ではネストされた型を前方宣言することはできません。別のファイルで `Bar` を前方宣言してその宣言を使用したい場合は、`Foo_Bar` として識別する必要があります。

## フィールド {#fields}

前のセクションで説明されたメソッドに加えて、プロトコルバッファコンパイラは、`.proto` ファイル内で定義された各フィールドに対して一連のアクセサメソッドを生成します。これらのメソッドは、`has_foo()` や `clear_foo()` などの小文字/スネークケースで表記されます。

アクセサメソッドだけでなく、コンパイラは各フィールドに対して整数定数を生成し、そのフィールド番号を含めます。定数名は、`k` という文字に続いてキャメルケースに変換されたフィールド名、そして `FieldNumber` が続きます。たとえば、`optional int32 foo_bar = 5;` というフィールドが与えられた場合、コンパイラは定数 `static const int kFooBarFieldNumber = 5;` を生成します。

`const` 参照を返すフィールドアクセサについては、その参照は、メッセージへの次の変更アクセスが行われると無効になる可能性があります。これには、任意のフィールドの非 `const` アクセサを呼び出すこと、`Message` から継承された任意の非 `const` メソッドを呼び出すこと、または他の方法でメッセージを変更すること（たとえば、`Swap()` の引数としてメッセージを使用すること）が含まれます。同様に、返された参照のアドレスは、メッセージへの次の変更アクセスが行われない限り、異なる呼び出し間で同じであることが保証されます。

フィールドアクセサーがポインタを返す場合、そのポインタは、メッセージへの次の変更または非変更アクセスが行われると無効になる可能性があります。これには、constnessに関係なく、任意のフィールドのアクセサーの呼び出し、`Message`から継承された任意のメソッドの呼び出し、または他の方法でメッセージにアクセスすること（たとえば、コピーコンストラクタを使用してメッセージをコピーすること）が含まれます。同様に、返されたポインタの値は、アクセサーの2つの異なる呼び出し間で常に同じであることは保証されません。

### オプションの数値フィールド（proto2およびproto3） {#numeric}

次のいずれかのフィールド定義の場合：

```proto
optional int32 foo = 1;
required int32 foo = 1;
```

コンパイラは次のアクセサーメソッドを生成します：

-   `bool has_foo() const`：フィールドが設定されている場合は`true`を返します。
-   `int32 foo() const`：フィールドの現在の値を返します。フィールドが設定されていない場合はデフォルト値を返します。
-   `void set_foo(int32 value)`：フィールドの値を設定します。これを呼び出した後、`has_foo()`は`true`を返し、`foo()`は`value`を返します。
-   `void clear_foo()`：フィールドの値をクリアします。これを呼び出した後、`has_foo()`は`false`を返し、`foo()`はデフォルト値を返します。

その他の数値フィールドタイプ（`bool`を含む）については、`int32`は、[スカラー値タイプテーブル](/programming-guides/proto3#scalar)に従って対応するC++タイプに置き換えられます。

### 暗黙の存在数値フィールド（proto3） {#implicit-numeric}

次のフィールド定義の場合：

```proto
optional int32 foo = 1;
int32 foo = 1;  // フィールドラベルが指定されていない場合、暗黙の存在にデフォルトで設定されます。
```

コンパイラは次のアクセサーメソッドを生成します：

-   `int32 foo() const`：フィールドの現在の値を返します。フィールドが設定されていない場合は0を返します。
-   `void set_foo(int32 value)`：フィールドの値を設定します。これを呼び出した後、`foo()`は`value`を返します。
-   `void clear_foo()`：フィールドの値をクリアします。これを呼び出した後、`foo()`は0を返します。

その他の数値フィールドタイプ（`bool`を含む）については、`int32`は、[スカラー値タイプテーブル](/programming-guides/proto3#scalar)に従って対応するC++タイプに置き換えられます。

### オプションの文字列/バイトフィールド（proto2 および proto3） {#string}

次のいずれかのフィールド定義に対して：

```proto
optional string foo = 1;
required string foo = 1;
optional bytes foo = 1;
required bytes foo = 1;
```

コンパイラは次のアクセサメソッドを生成します：

-   `bool has_foo() const`: フィールドが設定されている場合は `true` を返します。
-   `const string& foo() const`: フィールドの現在の値を返します。フィールドが設定されていない場合はデフォルト値を返します。
-   `void set_foo(const string& value)`: フィールドの値を設定します。これを呼び出した後、`has_foo()` は `true` を返し、`foo()` は `value` のコピーを返します。
-   `void set_foo(string&& value)`（C++11 以降）: フィールドの値を設定し、渡された文字列から移動します。これを呼び出した後、`has_foo()` は `true` を返し、`foo()` は `value` のコピーを返します。
-   `void set_foo(const char* value)`: C スタイルのヌル終端文字列を使用してフィールドの値を設定します。これを呼び出した後、`has_foo()` は `true` を返し、`foo()` は `value` のコピーを返します。
-   `void set_foo(const char* value, int size)`: 上記と同様ですが、文字列のサイズが明示的に指定され、ヌル終端バイトを探す代わりに決定されます。
-   `string* mutable_foo()`: フィールドの値を格納する可変 `string` オブジェクトへのポインタを返します。呼び出し前にフィールドが設定されていない場合、返される文字列は空になります（デフォルト値ではありません）。これを呼び出した後、`has_foo()` は `true` を返し、`foo()` は指定された文字列に書き込まれた値を返します。
-   `void clear_foo()`: フィールドの値をクリアします。これを呼び出した後、`has_foo()` は `false` を返し、`foo()` はデフォルト値を返します。
-   `void set_allocated_foo(string* value)`: `string` オブジェクトをフィールドに設定し、以前のフィールド値を解放します。`string` ポインタが `NULL` でない場合、メッセージは割り当てられた `string` オブジェクトの所有権を取得し、`has_foo()` は `true` を返します。メッセージはいつでも割り当てられた `string` オブジェクトを削除することができるため、オブジェクトへの参照は無効になる可能性があります。それ以外の場合、`value` が `NULL` の場合、動作は `clear_foo()` を呼び出すのと同じです。
-   `string* release_foo()`: フィールドの所有権を解放し、`string` オブジェクトのポインタを返します。これを呼び出した後、呼び出し元が割り当てられた `string` オブジェクトの所有権を取得し、`has_foo()` は `false` を返し、`foo()` はデフォルト値を返します。

<a id="proto3_string"></a>

### 暗黙の存在文字列/バイトフィールド（proto3）{#implicit-string}

次のいずれかのフィールド定義に対して：

```proto
optional string foo = 1;
string foo = 1;  // no field label specified, defaults to implicit presence.
optional bytes foo = 1;
bytes foo = 1;
```

コンパイラは次のアクセサメソッドを生成します：

-   `const string& foo() const`: フィールドの現在の値を返します。フィールドが設定されていない場合は、空の文字列/空のバイトを返します。
-   `void set_foo(const string& value)`: フィールドの値を設定します。これを呼び出した後、`foo()` は `value` のコピーを返します。
-   `void set_foo(string&& value)`（C++11以降）: フィールドの値を設定し、渡された文字列から移動します。これを呼び出した後、`foo()` は `value` のコピーを返します。
-   `void set_foo(const char* value)`: Cスタイルのヌル終端文字列を使用してフィールドの値を設定します。これを呼び出した後、`foo()` は `value` のコピーを返します。
-   `void set_foo(const char* value, int size)`: 上記と同様ですが、文字列のサイズが明示的に指定され、ヌル終端バイトを探す代わりに決定されます。
-   `string* mutable_foo()`: フィールドの値を格納する可変 `string` オブジェクトへのポインタを返します。呼び出し前にフィールドが設定されていない場合、返される文字列は空になります。これを呼び出した後、`foo()` は指定された文字列に書き込まれた値を返します。
-   `void clear_foo()`: フィールドの値をクリアします。これを呼び出した後、`foo()` は空の文字列/空のバイトを返します。
-   `void set_allocated_foo(string* value)`: `string` オブジェクトをフィールドに設定し、以前のフィールド値を解放します。`string` ポインタが `NULL` でない場合、メッセージは割り当てられた `string` オブジェクトの所有権を取ります。メッセージはいつでも割り当てられた `string` オブジェクトを削除することができるため、オブジェクトへの参照は無効になる可能性があります。それ以外の場合、`value` が `NULL` の場合、動作は `clear_foo()` を呼び出すのと同じです。
-   `string* release_foo()`: フィールドの所有権を解放し、`string` オブジェクトのポインタを返します。これを呼び出した後、呼び出し元が割り当てられた `string` オブジェクトの所有権を取り、`foo()` は空の文字列/空のバイトを返します。

### Singular Bytes Fields with Cord Support {#cord}

v23.0 では、単一の `bytes` フィールド（[`oneof` フィールド](#oneof-numeric)を含む）に対して[`absl::Cord`](https://github.com/abseil/abseil-cpp/blob/master/absl/strings/cord.h) をサポートする機能が追加されました。`Cord` は、単一の `string`、`repeated string`、`repeated bytes` フィールドでは使用できません。

`absl::Cord` を使用してデータを格納するために単一の `bytes` フィールドを設定するには、次の構文を使用します：

```proto
optional bytes foo = 25 [ctype=CORD];
bytes bar = 26 [ctype=CORD];
```

`repeated bytes` フィールドには `cord` を使用することはできません。これらのフィールドに対して `[ctype=CORD]` 設定を行っても、Protoc は無視します。

コンパイラは、次のアクセサメソッドを生成します：

-   `const ::absl::Cord& foo() const`：フィールドの現在の値を返します。フィールドが設定されていない場合は、空の `Cord`（proto3）またはデフォルト値（proto2）を返します。
-   `void set_foo(const ::absl::Cord& value)`：フィールドの値を設定します。これを呼び出した後、`foo()` は `value` を返します。
-   `void set_foo(::absl::string_view value)`：フィールドの値を設定します。これを呼び出した後、`foo()` は `absl::Cord` として `value` を返します。
-   `void clear_foo()`：フィールドの値をクリアします。これを呼び出した後、`foo()` は空の `Cord`（proto3）またはデフォルト値（proto2）を返します。
-   `bool has_foo()`：フィールドが設定されている場合は `true` を返します。

### Optional Enum Fields (proto2 and proto3) {#enum_field}

次の列挙型が与えられた場合：

```proto
enum Bar {
  BAR_UNSPECIFIED = 0;
  BAR_VALUE = 1;
  BAR_OTHER_VALUE = 2;
}
```

次のいずれかのフィールド定義に対して：

```proto
optional Bar foo = 1;
required Bar foo = 1;
```

コンパイラは、次のアクセサメソッドを生成します：

-   `bool has_foo() const`：フィールドが設定されている場合は `true` を返します。
-   `Bar foo() const`：フィールドの現在の値を返します。フィールドが設定されていない場合はデフォルト値を返します。
-   `void set_foo(Bar value)`：フィールドの値を設定します。これを呼び出した後、`has_foo()` は `true` を返し、`foo()` は `value` を返します。デバッグモード（つまり、`NDEBUG` が定義されていない場合）では、`value` が `Bar` に定義された値と一致しない場合、このメソッドはプロセスを中止します。
-   `void clear_foo()`：フィールドの値をクリアします。これを呼び出した後、`has_foo()` は `false` を返し、`foo()` はデフォルト値を返します。

### 暗黙の存在エンムフィールド（proto3）{#implicit-enum}

Enumタイプが与えられた場合：

```proto
enum Bar {
  BAR_UNSPECIFIED = 0;
  BAR_VALUE = 1;
  BAR_OTHER_VALUE = 2;
}
```

次のフィールド定義に対して：

```proto
optional Bar foo = 1;
Bar foo = 1;  // フィールドラベルが指定されていないため、暗黙の存在にデフォルトします。
```

コンパイラは次のアクセサメソッドを生成します：

-   `Bar foo() const`：フィールドの現在の値を返します。フィールドが設定されていない場合、デフォルト値（0）を返します。
-   `void set_foo(Bar value)`：フィールドの値を設定します。これを呼び出した後、`foo()`は`value`を返します。
-   `void clear_foo()`：フィールドの値をクリアします。これを呼び出した後、`foo()`はデフォルト値を返します。

### オプションの埋め込みメッセージフィールド（proto2およびproto3）{#embeddedmessage}

メッセージタイプが与えられた場合：

```proto
message Bar {}
```

次のフィールド定義のいずれかに対して：

```proto
//proto2
optional Bar foo = 1;
required Bar foo = 1;

//proto3
Bar foo = 1;
```

コンパイラは次のアクセサメソッドを生成します：

-   `bool has_foo() const`：フィールドが設定されている場合は`true`を返します。
-   `const Bar& foo() const`：フィールドの現在の値を返します。フィールドが設定されていない場合、フィールドの値が設定されていない`Bar`を返します（おそらく`Bar::default_instance()`）。
-   `Bar* mutable_foo()`：フィールドの値を格納する可変`Bar`オブジェクトへのポインタを返します。呼び出し前にフィールドが設定されていない場合、返される`Bar`にはフィールドの値が設定されていない（つまり、新しく割り当てられた`Bar`と同じになります）。これを呼び出した後、`has_foo()`は`true`を返し、`foo()`は`Bar`の同じインスタンスへの参照を返します。
-   `void clear_foo()`：フィールドの値をクリアします。これを呼び出した後、`has_foo()`は`false`を返し、`foo()`はデフォルト値を返します。
-   `void set_allocated_foo(Bar* bar)`：`Bar`オブジェクトをフィールドに設定し、以前のフィールド値を解放します。`Bar`ポインタが`NULL`でない場合、メッセージは割り当てられた`Bar`オブジェクトの所有権を取得し、`has_foo()`は`true`を返します。それ以外の場合、`Bar`が`NULL`である場合、動作は`clear_foo()`を呼び出すのと同じです。
-   `Bar* release_foo()`：フィールドの所有権を解放し、`Bar`オブジェクトのポインタを返します。これを呼び出した後、呼び出し元が割り当てられた`Bar`オブジェクトの所有権を取得し、`has_foo()`は`false`を返し、`foo()`はデフォルト値を返します。

### 繰り返し数値フィールド {#repeatednumeric}

このフィールド定義に対して:

```proto
repeated int32 foo = 1;
```

コンパイラは次のアクセサメソッドを生成します:

-   `int foo_size() const`: 現在のフィールド内の要素数を返します。空のセットをチェックするには、このメソッドではなく、基礎となる`RepeatedField`内の[`empty()`](/reference/cpp/api-docs/google.protobuf.repeated_field#RepeatedPtrField)メソッドを使用してください。
-   `int32 foo(int index) const`: 指定されたゼロベースのインデックスの要素を返します。インデックスが[0, foo_size())の範囲外の場合、未定義の動作が発生します。
-   `void set_foo(int index, int32 value)`: 指定されたゼロベースのインデックスの要素の値を設定します。
-   `void add_foo(int32 value)`: フィールドの末尾に指定された値の新しい要素を追加します。
-   `void clear_foo()`: フィールドからすべての要素を削除します。これを呼び出した後、`foo_size()`はゼロを返します。
-   `const RepeatedField<int32>& foo() const`: フィールドの要素を格納する基礎となる[`RepeatedField`](/reference/cpp/api-docs/google.protobuf.repeated_field#RepeatedField)を返します。このコンテナクラスはSTL風のイテレータやその他のメソッドを提供します。
-   `RepeatedField<int32>* mutable_foo()`: フィールドの要素を格納する基礎となる変更可能な`RepeatedField`へのポインタを返します。このコンテナクラスはSTL風のイテレータやその他のメソッドを提供します。

他の数値フィールドタイプ（`bool`を含む）については、`int32`は[スカラー値の種類の表](/programming-guides/proto2#scalar)に従って対応するC++タイプに置き換えられます。

### 繰り返し文字列フィールド {#repeatedstring}

次のいずれかのフィールド定義に対して:

```proto
repeated string foo = 1;
repeated bytes foo = 1;
```

コンパイラは次のアクセサメソッドを生成します:

-   `int foo_size() const`: 現在のフィールド内の要素数を返します。空のセットをチェックするには、このメソッドではなく、基礎となる`RepeatedField`内の[`empty()`](/reference/cpp/api-docs/google.protobuf.repeated_field#RepeatedPtrField)メソッドを使用してください。
-   `const string& foo(int index) const`: 指定されたゼロベースのインデックスの要素を返します。インデックスが[0, foo_size()-1]の範囲外の場合、未定義の動作が発生します。
-   `void set_foo(int index, const string& value)`: 指定されたゼロベースのインデックスの要素の値を設定します。
-   `void set_foo(int index, const char* value)`: Cスタイルのヌル終端文字列を使用して、指定されたゼロベースのインデックスの要素の値を設定します。
-   `void set_foo(int index, const char* value, int size)`: 上記と同様ですが、文字列のサイズがヌル終端バイトを探すことなく明示的に指定されます。
-   `string* mutable_foo(int index)`: 指定されたゼロベースのインデックスの要素の値を格納する変更可能な`string`オブジェクトへのポインタを返します。インデックスが[0, foo_size())の範囲外の場合、未定義の動作が発生します。
-   `void add_foo(const string& value)`: フィールドの末尾に指定された値の新しい要素を追加します。
-   `void add_foo(const char* value)`: Cスタイルのヌル終端文字列を使用して、フィールドの末尾に新しい要素を追加します。
-   `void add_foo(const char* value, int size)`: 上記と同様ですが、文字列のサイズがヌル終端バイトを探すことなく明示的に指定されます。
-   `string* add_foo()`: フィールドの末尾に新しい空の文字列要素を追加し、そのポインタを返します。
-   `void clear_foo()`: フィールドからすべての要素を削除します。これを呼び出した後、`foo_size()`はゼロを返します。
-   `const RepeatedPtrField<string>& foo() const`: フィールドの要素を格納する基礎となる[`RepeatedPtrField`](/reference/cpp/api-docs/google.protobuf.repeated_field#RepeatedPtrField)を返します。このコンテナクラスはSTL風のイテレータやその他のメソッドを提供します。
-   `RepeatedPtrField<string>* mutable_foo()`: フィールドの要素を格納する基礎となる変更可能な`RepeatedPtrField`へのポインタを返します。このコンテナクラスはSTL風のイテレータやその他のメソッドを提供します。

### 繰り返し列挙フィールド {#repeated_enum}

列挙型が与えられた場合：

```proto
enum Bar {
  BAR_UNSPECIFIED = 0;
  BAR_VALUE = 1;
  BAR_OTHER_VALUE = 2;
}
```

このフィールド定義に対して：

```proto
repeated Bar foo = 1;
```

コンパイラは次のアクセサメソッドを生成します：

-   `int foo_size() const`: 現在のフィールド内の要素数を返します。空のセットをチェックするには、このメソッドではなく、基礎となる `RepeatedField` の [`empty()`](/reference/cpp/api-docs/google.protobuf.repeated_field#RepeatedPtrField) メソッドを使用してください。
-   `Bar foo(int index) const`: 指定されたゼロベースのインデックスの要素を返します。インデックスが [0, foo_size()) の範囲外の場合、未定義の動作が発生します。
-   `void set_foo(int index, Bar value)`: 指定されたゼロベースのインデックスの要素の値を設定します。デバッグモード（つまり、`NDEBUG` が定義されていない場合）では、`value` が `Bar` に定義されている値と一致しない場合、このメソッドはプロセスを中止します。
-   `void add_foo(Bar value)`: フィールドの末尾に新しい要素を追加し、指定された値を設定します。デバッグモード（つまり、`NDEBUG` が定義されていない場合）では、`value` が `Bar` に定義されている値と一致しない場合、このメソッドはプロセスを中止します。
-   `void clear_foo()`: フィールドからすべての要素を削除します。これを呼び出した後、`foo_size()` はゼロを返します。
-   `const RepeatedField<int>& foo() const`: フィールドの要素を格納する基礎となる [`RepeatedField`](/reference/cpp/api-docs/google.protobuf.repeated_field#RepeatedField) を返します。このコンテナクラスは、STL風のイテレータやその他のメソッドを提供します。
-   `RepeatedField<int>* mutable_foo()`: フィールドの要素を格納する変更可能な基礎となる `RepeatedField` へのポインタを返します。このコンテナクラスは、STL風のイテレータやその他のメソッドを提供します。

### 繰り返し埋め込みメッセージフィールド {#repeatedmessage}

メッセージ型が与えられた場合：

```proto
message Bar {}
```

このフィールド定義に対して：

```proto
repeated Bar foo = 1;
```

コンパイラは次のアクセサメソッドを生成します：

-   `int foo_size() const`: 現在のフィールド内の要素数を返します。空のセットをチェックするには、このメソッドではなく、基礎となる `RepeatedField` の [`empty()`](/reference/cpp/api-docs/google.protobuf.repeated_field#RepeatedPtrField) メソッドを使用してください。
-   `const Bar& foo(int index) const`: 指定されたゼロベースのインデックスの要素を返します。インデックスが [0, foo_size()) の範囲外の場合、未定義の動作が発生します。
-   `Bar* mutable_foo(int index)`: 指定されたゼロベースのインデックスの要素の値を格納する変更可能な `Bar` オブジェクトへのポインタを返します。インデックスが [0, foo_size()) の範囲外の場合、未定義の動作が発生します。
-   `Bar* add_foo()`: フィールドの末尾に新しい要素を追加し、そのポインタを返します。返される `Bar` は変更可能であり、そのフィールドは設定されていません（つまり、新しく割り当てられた `Bar` と同じ状態です）。
-   `void clear_foo()`: フィールドからすべての要素を削除します。これを呼び出した後、`foo_size()` はゼロを返します。
-   `const RepeatedPtrField<Bar>& foo() const`: フィールドの要素を格納する基礎となる [`RepeatedPtrField`](/reference/cpp/api-docs/google.protobuf.repeated_field#RepeatedPtrField) を返します。このコンテナクラスは、STL風のイテレータやその他のメソッドを提供します。
-   `RepeatedPtrField<Bar>* mutable_foo()`: フィールドの要素を格納する変更可能な基礎となる `RepeatedPtrField` へのポインタを返します。このコンテナクラスは、STL風のイテレータやその他のメソッドを提供します。

### 数値フィールドの Oneof {#oneof-numeric}

この [oneof](#oneof) フィールドの定義に対して:

```proto
oneof example_name {
    int32 foo = 1;
    ...
}
```

コンパイラは次のアクセサメソッドを生成します:

-   `bool has_foo() const`: `kFoo` の oneof ケースがある場合は `true` を返します。
-   `int32 foo() const`: oneof ケースが `kFoo` の場合はフィールドの現在の値を返し、そうでない場合はデフォルト値を返します。
-   `void set_foo(int32 value)`:
    -   同じ oneof 内の他の oneof フィールドが設定されている場合は、`clear_example_name()` を呼び出します。
    -   このフィールドの値を設定し、oneof ケースを `kFoo` に設定します。
    -   `has_foo()` は `true` を返し、`foo()` は `value` を返し、`example_name_case()` は `kFoo` を返します。
-   `void clear_foo()`:
    -   oneof ケースが `kFoo` でない場合は何も変更されません。
    -   oneof ケースが `kFoo` の場合、フィールドの値と oneof ケースをクリアします。
        `has_foo()` は `false` を返し、`foo()` はデフォルト値を返し、`example_name_case()` は `EXAMPLE_NAME_NOT_SET` を返します。

その他の数値フィールドタイプ（`bool` を含む）については、`int32` は
[scalar value types table](/programming-guides/proto3#scalar) に従って対応する C++ タイプに置き換えられます。

### 文字列フィールドの Oneof {#oneof-string}

これらの [oneof](#oneof) フィールドの定義のいずれかに対して:

```proto
oneof example_name {
    string foo = 1;
    ...
}
oneof example_name {
    bytes foo = 1;
    ...
}
```

コンパイラは次のアクセサメソッドを生成します:

-   `bool has_foo() const`: oneof ケースが `kFoo` の場合は `true` を返します。
-   `const string& foo() const`: oneof ケースが `kFoo` の場合はフィールドの現在の値を返し、そうでない場合はデフォルト値を返します。
-   `void set_foo(const string& value)`:
    -   同じ oneof 内の他の oneof フィールドが設定されている場合は、`clear_example_name()` を呼び出します。
    -   このフィールドの値を設定し、oneof ケースを `kFoo` に設定します。
    -   `has_foo()` は `true` を返し、`foo()` は `value` のコピーを返し、`example_name_case()` は `kFoo` を返します。
-   `void set_foo(const char* value)`:
    -   同じ oneof 内の他の oneof フィールドが設定されている場合は、`clear_example_name()` を呼び出します。
    -   C スタイルのヌル終端文字列を使用してフィールドの値を設定し、oneof ケースを `kFoo` に設定します。
    -   `has_foo()` は `true` を返し、`foo()` は `value` のコピーを返し、`example_name_case()` は `kFoo` を返します。
-   `void set_foo(const char* value, int size)`: 上記と同様ですが、文字列のサイズが明示的に与えられ、ヌル終端バイトを探す代わりに決定されます。
-   `string* mutable_foo()`:
    -   同じ oneof 内の他の oneof フィールドが設定されている場合は、`clear_example_name()` を呼び出します。
    -   oneof ケースを `kFoo` に設定し、フィールドの値を格納する可変文字列オブジェクトへのポインタを返します。呼び出し前に oneof ケースが `kFoo` でない場合、返される文字列は空になります（デフォルト値ではありません）。
    -   `has_foo()` は `true` を返し、`foo()` は指定された文字列に書き込まれた値を返し、`example_name_case()` は `kFoo` を返します。
-   `void clear_foo()`:
    -   oneof ケースが `kFoo` でない場合は何も変更されません。
    -   oneof ケースが `kFoo` の場合、フィールドを解放し、oneof ケースをクリアします。
        `has_foo()` は `false` を返し、`foo()` はデフォルト値を返し、`example_name_case()` は `EXAMPLE_NAME_NOT_SET` を返します。
-   `void set_allocated_foo(string* value)`:
    -   `clear_example_name()` を呼び出します。
    -   文字列ポインタが `NULL` でない場合: 文字列オブジェクトをフィールドに設定し、oneof ケースを `kFoo` に設定します。メッセージは割り当てられた文字列オブジェクトの所有権を取得し、`has_foo()` は `true` を返し、`example_name_case()` は `kFoo` を返します。
    -   文字列ポインタが `NULL` の場合、`has_foo()` は `false` を返し、`example_name_case()` は `EXAMPLE_NAME_NOT_SET` を返します。
-   `string* release_foo()`:
    -   oneof ケースが `kFoo` でない場合は `NULL` を返します。
    -   oneof ケースをクリアし、フィールドの所有権を解放し、文字列オブジェクトのポインタを返します。これを呼び出した後、呼び出し元が割り当てられた文字列オブジェクトの所有権を取得し、`has_foo()` は `false` を返し、`foo()` はデフォルト値を返し、`example_name_case()` は `EXAMPLE_NAME_NOT_SET` を返します。

### Oneof Enum Fields {#oneof-enum}

与えられた列挙型：

```proto
enum Bar {
  BAR_UNSPECIFIED = 0;
  BAR_VALUE = 1;
  BAR_OTHER_VALUE = 2;
}
```

[oneof](#oneof) フィールドの定義：

```proto
oneof example_name {
    Bar foo = 1;
    ...
}
```

コンパイラは以下のアクセサメソッドを生成します：

-   `bool has_foo() const`: `kFoo` の oneof ケースがある場合は `true` を返します。
-   `Bar foo() const`: oneof ケースが `kFoo` の場合はフィールドの現在の値を返し、それ以外の場合はデフォルト値を返します。
-   `void set_foo(Bar value)`:
    -   同じ oneof 内の他のフィールドが設定されている場合、`clear_example_name()` を呼び出します。
    -   このフィールドの値を設定し、oneof ケースを `kFoo` に設定します。
    -   `has_foo()` は `true` を返し、`foo()` は `value` を返し、`example_name_case()` は `kFoo` を返します。
    -   デバッグモード（つまり、NDEBUG が定義されていない場合）では、`value` が `Bar` に定義されている値と一致しない場合、このメソッドはプロセスを中止します。
-   `void clear_foo()`:
    -   oneof ケースが `kFoo` でない場合は何も変更されません。
    -   oneof ケースが `kFoo` の場合、フィールドの値と oneof ケースをクリアします。`has_foo()` は `false` を返し、`foo()` はデフォルト値を返し、`example_name_case()` は `EXAMPLE_NAME_NOT_SET` を返します。

### Oneof Embedded Message Fields {#oneof-embedded}

与えられたメッセージ型：

```proto
message Bar {}
```

[oneof](#oneof) フィールドの定義：

```proto
oneof example_name {
    Bar foo = 1;
    ...
}
```

コンパイラは以下のアクセサメソッドを生成します：

-   `bool has_foo() const`: oneof ケースが `kFoo` の場合は `true` を返します。
-   `const Bar& foo() const`: oneof ケースが `kFoo` の場合はフィールドの現在の値を返し、それ以外の場合はフィールドが設定されていない Bar を返します（おそらく `Bar::default_instance()`）。
-   `Bar* mutable_foo()`:
    -   同じ oneof 内の他のフィールドが設定されている場合、`clear_example_name()` を呼び出します。
    -   oneof ケースを `kFoo` に設定し、フィールドの値を格納する mutable Bar オブジェクトへのポインタを返します。呼び出し前の oneof ケースが `kFoo` でない場合、返された Bar はフィールドが設定されていない状態になります（つまり、新しく割り当てられた Bar と同じ状態になります）。
    -   これを呼び出した後、`has_foo()` は `true` を返し、`foo()` は `Bar` の同じインスタンスへの参照を返し、`example_name_case()` は `kFoo` を返します。
-   `void clear_foo()`:
    -   oneof ケースが `kFoo` でない場合は何も変更されません。
    -   oneof ケースが `kFoo` の場合、フィールドを解放し、oneof ケースをクリアします。`has_foo()` は `false` を返し、`foo()` はデフォルト値を返し、`example_name_case()` は `EXAMPLE_NAME_NOT_SET` を返します。
-   `void set_allocated_foo(Bar* bar)`:
    -   `clear_example_name()` を呼び出します。
    -   `Bar` ポインタが `NULL` でない場合：`Bar` オブジェクトをフィールドに設定し、oneof ケースを `kFoo` に設定します。メッセージは割り当てられた `Bar` オブジェクトの所有権を取得し、`has_foo()` は `true` を返し、`example_name_case()` は `kFoo` を返します。
    -   ポインタが `NULL` の場合、`has_foo()` は `false` を返し、`example_name_case()` は `EXAMPLE_NAME_NOT_SET` を返します（動作は `clear_example_name()` を呼び出すのと同様です）。
-   `Bar* release_foo()`:
    -   oneof ケースが `kFoo` の場合は `NULL` を返します。
    -   oneof ケースが `kFoo` の場合、oneof ケースをクリアし、フィールドの所有権を解放し、`Bar` オブジェクトのポインタを返します。これを呼び出した後、呼び出し元が割り当てられた `Bar` オブジェクトの所有権を取得し、`has_foo()` は `false` を返し、`foo()` はデフォルト値を返し、`example_name_case()` は `EXAMPLE_NAME_NOT_SET` を返します。

### フィールドのマッピング {#map}

このマップフィールドの定義について:

```proto
map<int32, int32> weight = 1;
```

コンパイラは以下のアクセサメソッドを生成します:

-   `const google::protobuf::Map<int32, int32>& weight();`: 変更不可な`Map`を返します。
-   `google::protobuf::Map<int32, int32>* mutable_weight();`: 変更可能な`Map`を返します。

[`google::protobuf::Map`](/reference/cpp/api-docs/google.protobuf.map)は、プロトコルバッファで使用される特別なコンテナ型で、マップフィールドを格納するために使用されます。以下のインターフェースからわかるように、`std::map`と`std::unordered_map`の一般的に使用されるサブセットのメソッドを使用しています。

**注意:** これらのマップは順序付けられていません。

```cpp
template<typename Key, typename T> {
class Map {
  // Member types
  typedef Key key_type;
  typedef T mapped_type;
  typedef MapPair< Key, T > value_type;

  // Iterators
  iterator begin();
  const_iterator begin() const;
  const_iterator cbegin() const;
  iterator end();
  const_iterator end() const;
  const_iterator cend() const;
  // Capacity
  int size() const;
  bool empty() const;

  // Element access
  T& operator[](const Key& key);
  const T& at(const Key& key) const;
  T& at(const Key& key);

  // Lookup
  bool contains(const Key& key) const;
  int count(const Key& key) const;
  const_iterator find(const Key& key) const;
  iterator find(const Key& key);

  // Modifiers
  pair<iterator, bool> insert(const value_type& value);
  template<class InputIt>
  void insert(InputIt first, InputIt last);
  size_type erase(const Key& Key);
  iterator erase(const_iterator pos);
  iterator erase(const_iterator first, const_iterator last);
  void clear();

  // Copy
  Map(const Map& other);
  Map& operator=(const Map& other);
}
```

データを追加する最も簡単な方法は、通常のマップ構文を使用することです。例:

```cpp
std::unique_ptr<ProtoName> my_enclosing_proto(new ProtoName);
(*my_enclosing_proto->mutable_weight())[my_key] = my_value;
```

`pair<iterator, bool> insert(const value_type& value)`は、`value_type`インスタンスのディープコピーを暗黙的に引き起こします。`google::protobuf::Map`に新しい値を挿入する最も効率的な方法は次のとおりです:

```cpp
T& operator[](const Key& key): map[new_key] = new_mapped;
```

#### 標準マップと`google::protobuf::Map`の使用 {#protobuf-map}

`google::protobuf::Map`は`std::map`および`std::unordered_map`と同じイテレータAPIをサポートしています。直接`google::protobuf::Map`を使用したくない場合は、次のようにして`google::protobuf::Map`を標準マップに変換できます:

```cpp
std::map<int32, int32> standard_map(message.weight().begin(),
                                    message.weight().end());
```

これにより、マップ全体のディープコピーが作成されます。

また、次のようにして標準マップから`google::protobuf::Map`を構築することもできます:

```cpp
google::protobuf::Map<int32, int32> weight(standard_map.begin(), standard_map.end());
```

#### 未知の値の解析 {#parsing-unknown}

ワイヤ上では、.protoマップは各キー/値ペアに対してマップエントリメッセージと等価であり、マップ自体はマップエントリの繰り返しフィールドです。通常のメッセージタイプと同様に、解析されたマップエントリメッセージには未知のフィールドが存在する可能性があります: たとえば、`map<int32, string>`と定義されたマップ内の`int64`型のフィールド。


マップエントリメッセージのワイヤーフォーマットに不明なフィールドがある場合、それらは破棄されます。

マップエントリメッセージのワイヤーフォーマットに不明な列挙値がある場合、proto2とproto3では異なる方法で処理されます。proto2では、マップエントリメッセージ全体が含まれるメッセージの不明なフィールドセットに配置されます。proto3では、既知の列挙値であるかのようにマップフィールドに配置されます。

## Any {#any}

次のような[`Any`](/programming-guides/proto3#any)フィールドが与えられた場合：

```proto
import "google/protobuf/any.proto";

message ErrorStatus {
  string message = 1;
  google.protobuf.Any details = 2;
}
```

生成されたコードでは、`details`フィールドのゲッターは`google::protobuf::Any`のインスタンスを返します。これには、`Any`の値をパックおよびアンパックするための次の特別なメソッドが提供されます：

```cpp
class Any {
 public:
  // Packs the given message into this Any using the default type URL
  // prefix “type.googleapis.com”. Returns false if serializing the message failed.
  bool PackFrom(const google::protobuf::Message& message);

  // Packs the given message into this Any using the given type URL
  // prefix. Returns false if serializing the message failed.
  bool PackFrom(const google::protobuf::Message& message,
                const string& type_url_prefix);

  // Unpacks this Any to a Message. Returns false if this Any
  // represents a different protobuf type or parsing fails.
  bool UnpackTo(google::protobuf::Message* message) const;

  // Returns true if this Any represents the given protobuf type.
  template<typename T> bool Is() const;
}
```

## Oneof {#oneof}

次のようなoneof定義が与えられた場合：

```proto
oneof example_name {
    int32 foo_int = 4;
    string foo_string = 9;
    ...
}
```

コンパイラは次のC++列挙型を生成します：

```cpp
enum ExampleNameCase {
  kFooInt = 4,
  kFooString = 9,
  EXAMPLE_NAME_NOT_SET = 0
}
```

さらに、次のメソッドが生成されます：

-   `ExampleNameCase example_name_case() const`：設定されているフィールドを示す列挙型を返します。どれも設定されていない場合は`EXAMPLE_NAME_NOT_SET`を返します。
-   `void clear_example_name()`：oneofフィールドセットがポインタ（MessageまたはString）を使用している場合、オブジェクトを解放し、oneofケースを`EXAMPLE_NAME_NOT_SET`に設定します。

## Enumerations {#enum}

次のような列挙型定義が与えられた場合：

```proto
enum Foo {
  VALUE_A = 0;
  VALUE_B = 5;
  VALUE_C = 1234;
}
```

プロトコルバッファコンパイラは、同じ値のセットを持つC++列挙型`Foo`を生成します。さらに、コンパイラは次の関数を生成します：

-   `const EnumDescriptor* Foo_descriptor()`：この列挙型が定義する値に関する情報を含む型の記述子を返します。
-   `bool Foo_IsValid(int value)`：与えられた数値が`Foo`の定義された値のいずれかと一致する場合は`true`を返します。上記の例では、入力が0、5、または1234の場合に`true`を返します。
-   `const string& Foo_Name(int value)`：指定された数値に対する名前を返します。そのような値が存在しない場合は空の文字列を返します。複数の値がこの数値を持つ場合、最初に定義された値が返されます。上記の例では、`Foo_Name(5)`は`"VALUE_B"`を返します。
-   `bool Foo_Parse(const string& name, Foo* value)`：`name`がこの列挙型の有効な値名である場合、その値を`value`に割り当てて`true`を返します。それ以外の場合は`false`を返します。上記の例では、`Foo_Parse("VALUE_C", &some_foo)`は`true`を返し、`some_foo`を1234に設定します。
-   `const Foo Foo_MIN`：列挙型の最小有効値（例ではVALUE_A）。
-   `const Foo Foo_MAX`：列挙型の最大有効値（例ではVALUE_C）。
-   `const int Foo_ARRAYSIZE`：常に`Foo_MAX + 1`として定義されます。

**proto2の列挙型へ整数をキャストする際には注意してください。** 整数がproto2の列挙型値にキャストされる場合、その整数はその列挙型の有効な値の1つでなければなりません。そうでない場合、結果は未定義になる可能性があります。疑わしい場合は、生成された`Foo_IsValid()`関数を使用してキャストが有効かどうかをテストしてください。proto2メッセージの列挙型フィールドを無効な値に設定すると、アサーションの失敗が発生する可能性があります。proto2メッセージを解析する際に無効な列挙値が読み取られた場合、それは[unknown field](/reference/cpp/api-docs/google.protobuf.unknown_field_set)として扱われます。これらのセマンティクスはproto3で変更されています。int32に収まる限り、任意の整数をproto3の列挙型値に安全にキャストできます。無効な列挙値も、proto3メッセージを解析する際に保持され、列挙型フィールドアクセサによって返されます。

**switch文でproto3の列挙型を使用する際には注意してください。** Proto3の列挙型は、指定されたシンボルの範囲外の可能な値を持つオープンな列挙型です。認識されない列挙値は、proto3メッセージを解析する際に保持され、列挙型フィールドアクセサによって返されます。デフォルトケースのないproto3列挙型のswitch文は、既知のすべてのフィールドがリストされていても、すべてのケースをキャッチすることができません。これは、データの破損やランタイムクラッシュを含む予期しない動作を引き起こす可能性があります。**常にデフォルトケースを追加するか、スイッチの外で`Foo_IsValid(int)`を明示的に呼び出して未知の列挙値を処理してください。**

メッセージ型内で列挙型を定義することができます。この場合、プロトコルバッファコンパイラは、列挙型自体がメッセージクラスのネストされた内部で宣言されたかのように見えるコードを生成します。`Foo_descriptor()`および`Foo_IsValid()`関数は静的メソッドとして宣言されます。実際には、列挙型自体とその値はグローバルスコープでマングルされた名前で宣言され、typedefと一連の定数定義を使用してクラスのスコープにインポートされます。これは、宣言の順序付けの問題を回避するためにのみ行われます。マングルされたトップレベルの名前に依存しないでください。列挙型は本当にメッセージクラスにネストされていると仮定してください。

## 拡張機能（proto2のみ） {#extension}

与えられた拡張範囲を持つメッセージがある場合：

```proto
message Foo {
  extensions 100 to 199;
}
```

プロトコルバッファコンパイラは、`Foo`に対していくつかの追加メソッドを生成します：
`HasExtension()`、`ExtensionSize()`、`ClearExtension()`、`GetExtension()`、
`SetExtension()`、`MutableExtension()`、`AddExtension()`、
`SetAllocatedExtension()`、`ReleaseExtension()`。これらのメソッドの各々は、
拡張フィールドを識別する拡張識別子（後述）を最初のパラメータとして取ります。
残りのパラメータと戻り値は、拡張識別子の型と同じ型の通常の（非拡張）フィールドに
対して生成される対応するアクセサメソッドのものとまったく同じです。
（`GetExtension()`は特別な接頭辞のないアクセサに対応します。）

拡張定義が与えられた場合：

```proto
extend Foo {
  optional int32 bar = 123;
  repeated int32 repeated_bar = 124;
  optional Bar message_bar = 125;
}
```

単数の拡張フィールド `bar` の場合、プロトコルバッファコンパイラは
`bar` と呼ばれる "拡張識別子" を生成します。これを使用して、この拡張に
アクセスするために `Foo` の拡張アクセサを使用できます。以下のように：

```cpp
Foo foo;
assert(!foo.HasExtension(bar));
foo.SetExtension(bar, 1);
assert(foo.HasExtension(bar));
assert(foo.GetExtension(bar) == 1);
foo.ClearExtension(bar);
assert(!foo.HasExtension(bar));
```

メッセージ拡張フィールド `message_bar` の場合、フィールドが設定されていない場合、
`foo.GetExtension(message_bar)` は、そのフィールドのいずれも設定されていない
`Bar` を返します（おそらく `Bar::default_instance()`）。

同様に、繰り返しの拡張フィールド `repeated_bar` の場合、コンパイラは
`repeated_bar` と呼ばれる拡張識別子を生成し、これも `Foo` の拡張アクセサで
使用できます：

```cpp
Foo foo;
for (int i = 0; i < kSize; ++i) {
  foo.AddExtension(repeated_bar, i)
}
assert(foo.ExtensionSize(repeated_bar) == kSize)
for (int i = 0; i < kSize; ++i) {
  assert(foo.GetExtension(repeated_bar, i) == i)
}
```

（拡張識別子の正確な実装は複雑で、テンプレートの魔法的な使用を含みますが、
拡張識別子がどのように機能するかを気にする必要はありません。）

拡張は別の型の中にネストして宣言することができます。例えば、次のような一般的な
パターンがあります：

```proto
message Baz {
  extend Foo {
    optional Baz foo_ext = 124;
  }
}
```

この場合、拡張識別子 `foo_ext` は `Baz` の中にネストして宣言されています。
以下のように使用できます：

```cpp
Foo foo;
Baz* baz = foo.MutableExtension(Baz::foo_ext);
FillInMyBaz(baz);
```

## アリーナ割り当て {#arena}

アリーナ割り当ては、プロトコルバッファを使用する際にメモリ使用量を最適化し、パフォーマンスを向上させるためのC++専用機能です。`.proto` でアリーナ割り当てを有効にすると、生成されたC++コードにアリーナを使用するための追加コードが追加されます。アリーナ割り当て API について詳しくは、[アリーナ割り当てガイド](/reference/cpp/arenas) を参照してください。

## サービス {#service}

`.proto` ファイルに以下の行が含まれている場合：

```proto
option cc_generic_services = true;
```

その場合、プロトコルバッファコンパイラは、このセクションで説明されているサービス定義に基づいてコードを生成します。ただし、生成されたコードは、特定の RPC システムに結びついていないため、1 つのシステムに適したコードよりも間接参照が多く必要となる可能性があります。このコードを生成したくない場合は、ファイルに次の行を追加してください：

```proto
option cc_generic_services = false;
```

上記のいずれの行も指定されていない場合、オプションは `false` にデフォルトで設定されます。汎用サービスは非推奨です。（2.4.0 より前では、オプションは `true` にデフォルトで設定されます）

`.proto` 言語のサービス定義に基づく RPC システムは、システムに適したコードを生成するための[プラグイン](/reference/cpp/api-docs/google.protobuf.compiler.plugin.pb)を提供する必要があります。これらのプラグインは、抽象サービスが無効になっていることを要求する可能性が高く、そのために自分自身の同じ名前のクラスを生成できます。

このセクションの残りの部分では、抽象サービスが有効になっている場合にプロトコルバッファコンパイラが生成する内容について説明します。

### インターフェース

サービス定義が与えられた場合：

```proto
service Foo {
  rpc Bar(FooRequest) returns(FooResponse);
}
```

プロトコルバッファコンパイラは、このサービスを表すクラス `Foo` を生成します。`Foo` には、サービス定義で定義された各メソッドに対応する仮想メソッドがあります。この場合、メソッド `Bar` は次のように定義されています：

```cpp
virtual void Bar(RpcController* controller, const FooRequest* request,
                 FooResponse* response, Closure* done);
```

これらのパラメータは、`Service::CallMethod()` のパラメータと同等ですが、`method` 引数は暗黙的であり、`request` と `response` は正確な型を指定しています。

これらの生成されたメソッドは仮想的ですが、純粋仮想ではありません。デフォルトの実装は単に、そのメソッドが未実装であることを示すエラーメッセージを使用して`controller->SetFailed()`を呼び出し、その後`done`コールバックを実行します。独自のサービスを実装する際には、この生成されたサービスをサブクラス化し、適切にメソッドを実装する必要があります。

`Foo`は`Service`インターフェースをサブクラス化します。プロトコルバッファコンパイラは、`Service`のメソッドの実装を自動的に生成します。

- `GetDescriptor`: サービスの[`ServiceDescriptor`](/reference/cpp/api-docs/google.protobuf.descriptor#ServiceDescriptor)を返します。
- `CallMethod`: 提供されたメソッド記述子に基づいて呼び出されているメソッドを決定し、リクエストとレスポンスメッセージオブジェクトを正しい型にダウンキャストして直接呼び出します。
- `GetRequestPrototype`および`GetResponsePrototype`: 指定されたメソッドに対して適切な型のリクエストまたはレスポンスのデフォルトインスタンスを返します。

以下の静的メソッドも生成されます。

- `static ServiceDescriptor descriptor()`: この型の記述子を返します。この記述子には、このサービスがどのメソッドを持ち、それらの入力および出力の型が何であるかに関する情報が含まれています。

### Stub {#stub}

プロトコルバッファコンパイラは、サービスインターフェースごとに「スタブ」実装も生成します。これは、サービスを実装するサーバーにリクエストを送信したいクライアントが使用します。`Foo`サービス（上記）の場合、スタブ実装`Foo_Stub`が定義されます。ネストされたメッセージタイプと同様に、`Foo_Stub`を`Foo::Stub`として参照できるようにtypedefが使用されます。

`Foo_Stub`は、以下のメソッドも実装する`Foo`のサブクラスです。

- `Foo_Stub(RpcChannel* channel)`: 指定されたチャンネルにリクエストを送信する新しいスタブを構築します。
- `Foo_Stub(RpcChannel* channel, ChannelOwnership ownership)`: 指定されたチャンネルにリクエストを送信し、可能であればそのチャンネルを所有する新しいスタブを構築します。`ownership`が`Service::STUB_OWNS_CHANNEL`の場合、スタブオブジェクトが削除されると、チャンネルも削除されます。
- `RpcChannel* channel()`: このスタブのチャンネルを返します。これはコンストラクタに渡されたものです。

スタブは、チャンネルをラッパーとして使用して、サービスの各メソッドを実装します。メソッドの1つを呼び出すと、単純に`channel->CallMethod()`が呼び出されます。

Protocol BufferライブラリにはRPCの実装が含まれていません。ただし、生成されたサービスクラスを任意のRPC実装に接続するために必要なツールはすべて含まれています。[`RpcChannel`](/reference/cpp/api-docs/google.protobuf.service#RpcChannel)と[`RpcController`](/reference/cpp/api-docs/google.protobuf.service#RpcController)の実装を提供するだけです。詳細については、[`service.h`](/reference/cpp/api-docs/google.protobuf.service)のドキュメントを参照してください。

## プラグイン挿入ポイント {#plugins}

[C++コードジェネレータの出力を拡張したいコードジェネレータプラグイン](/reference/cpp/api-docs/google.protobuf.compiler.plugin.pb)は、指定された挿入ポイント名を使用して次のタイプのコードを挿入できます。各挿入ポイントは、それ以外の場合を除いて`.pb.cc`ファイルと`.pb.h`ファイルの両方に表示されます。

-   `includes`: インクルードディレクティブ。
-   `namespace_scope`: ファイルのパッケージ/名前空間に属する宣言ですが、特定のクラス内にはありません。他のすべての名前空間スコープのコードの後に表示されます。
-   `global_scope`: ファイルの名前空間の外側にあるトップレベルに属する宣言です。ファイルの最後に表示されます。
-   `class_scope:TYPENAME`: メッセージクラスに属するメンバー宣言です。`TYPENAME`は完全なproto名です。例：`package.MessageType`。この挿入ポイントは、クラス内の他のすべての公開宣言の後に表示されます。この挿入ポイントは`.pb.h`ファイルにのみ表示されます。

Protocol Buffersの将来のバージョンでこれらの実装の詳細が変更される可能性があるため、標準コードジェネレータによって宣言されたプライベートクラスメンバーに依存するコードを生成しないでください。
