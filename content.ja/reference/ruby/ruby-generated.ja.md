+++
title = "Ruby Generated Code Guide"
weight = 780
linkTitle = "生成されたコードガイド"
description = "プロトコルバッファコンパイラが任意のプロトコル定義に対して生成するメッセージオブジェクトのAPIを説明します。"
type = "docs"
+++

このドキュメントを読む前に、[proto2](/programming-guides/proto2) や [proto3](/programming-guides/proto3) の言語ガイドを読むことをお勧めします。

Ruby用のプロトコルコンパイラは、メッセージスキーマを定義するDSLを使用してRubyソースファイルを生成します。ただし、このDSLはまだ変更の対象となります。このガイドでは、生成されたメッセージのAPIのみを説明し、DSLについては触れません。

## コンパイラの呼び出し {#invocation}

プロトコルバッファコンパイラは、`--ruby_out=` コマンドラインフラグを使用して呼び出されると、Rubyの出力を生成します。`--ruby_out=` オプションのパラメータは、コンパイラがRubyの出力を書き込むディレクトリです。コンパイラは、各 `.proto` ファイルの入力ごとに `.rb` ファイルを作成します。出力ファイルの名前は、次の2つの変更を加えることで計算されます。

- 拡張子（`.proto`）が `_pb.rb` に置き換えられます。
- protoパス（`--proto_path=` または `-I` コマンドラインフラグで指定された）が出力パス（`--ruby_out=` フラグで指定された）に置き換えられます。

例えば、次のようにコンパイラを呼び出したとします：

```shell
protoc --proto_path=src --ruby_out=build/gen src/foo.proto src/bar/baz.proto
```

コンパイラは、`src/foo.proto` と `src/bar/baz.proto` ファイルを読み込み、2つの出力ファイルを生成します：`build/gen/foo_pb.rb` と `build/gen/bar/baz_pb.rb`。必要に応じてコンパイラはディレクトリ `build/gen/bar` を自動的に作成しますが、`build` または `build/gen` は自動的に作成しません。これらはすでに存在している必要があります。

## パッケージ {#package}

`.proto` ファイルで定義されたパッケージ名は、生成されたメッセージのモジュール構造を生成するために使用されます。次のようなファイルがあるとします：

```proto
package foo_bar.baz;

message MyMessage {}
```

プロトコルコンパイラは、名前が `FooBar::Baz::MyMessage` である出力メッセージを生成します。

ただし、`.proto` ファイルに `ruby_package` オプションが含まれている場合、次のようになります：

```proto
option ruby_package = "Foo::Bar";
```

生成された出力は、`ruby_package` オプションを優先し、`Foo::Bar::MyMessage` を生成します。

## Messages {#message}

単純なメッセージ宣言がある場合:

```proto
message Foo {}
```

プロトコルバッファコンパイラは `Foo` というクラスを生成します。生成されたクラスは Ruby の `Object` クラスから派生します（プロトには共通の基底クラスがありません）。C++ と Java とは異なり、Ruby 生成コードは `.proto` ファイルの `optimize_for` オプションに影響を受けません。実際には、すべての Ruby コードはコードサイズに最適化されています。

独自の `Foo` サブクラスを作成しないでください。生成されたクラスはサブクラス化を前提としておらず、「壊れやすい基底クラス」の問題を引き起こす可能性があります。

Ruby メッセージクラスは、各フィールドに対するアクセサを定義し、以下の標準メソッドも提供します:

-   `Message#dup`, `Message#clone`: このメッセージの浅いコピーを実行し、新しいコピーを返します。
-   `Message#==`: 2 つのメッセージ間での深い等価比較を実行します。
-   `Message#hash`: メッセージの値の浅いハッシュを計算します。
-   `Message#to_hash`, `Message#to_h`: オブジェクトを Ruby の `Hash` オブジェクトに変換します。トップレベルのメッセージのみが変換されます。
-   `Message#inspect`: このメッセージを表す人間が読める文字列を返します。
-   `Message#[]`, `Message#[]=`: 文字列名でフィールドを取得または設定します。将来的には、これは拡張機能の取得/設定にも使用される可能性があります。

メッセージクラスは、以下のメソッドも静的として定義します。（一般的に、フィールド名と競合する通常のメソッドよりも静的メソッドを好みます。）

-   `Message.decode(str)`: このメッセージのバイナリ protobuf をデコードし、新しいインスタンスで返します。
-   `Message.encode(proto)`: このクラスのメッセージオブジェクトをバイナリ文字列にシリアライズします。
-   `Message.decode_json(str)`: このメッセージのための JSON テキスト文字列をデコードし、新しいインスタンスで返します。
-   `Message.encode_json(proto)`: このクラスのメッセージオブジェクトを JSON テキスト文字列にシリアライズします。
-   `Message.descriptor`: このメッセージの `Google::Protobuf::Descriptor` オブジェクトを返します。

```ruby
メッセージを作成するときは、コンストラクタ内でフィールドを便利に初期化できます。以下は、メッセージの構築と使用例です：

```ruby
message = MyMessage.new(:int_field => 1,
                        :string_field => "String",
                        :repeated_int_field => [1, 2, 3, 4],
                        :submessage_field => SubMessage.new(:foo => 42))
serialized = MyMessage.encode(message)

message2 = MyMessage.decode(serialized)
raise unless message2.int_field == 1
```

### ネストされた型

メッセージは別のメッセージ内で宣言することができます。例えば：

```proto
message Foo {
  message Bar { }
}
```

この場合、`Bar` クラスは `Foo` の内部にクラスとして宣言されているため、`Foo::Bar` として参照できます。

## フィールド

メッセージ型の各フィールドには、フィールドを設定および取得するためのアクセサメソッドがあります。したがって、フィールド `foo` が与えられた場合、次のように書くことができます：

```ruby
message.foo = get_value()
print message.foo
```

フィールドを設定するときは、そのフィールドの宣言された型に対して値が型チェックされます。値が間違った型である場合（または範囲外の場合）、例外が発生します。

### 単一フィールド

単一のプリミティブフィールド（数値、文字列、およびブール値）の場合、フィールドに割り当てる値は正しい型である必要があり、適切な範囲内である必要があります：

- **数値型**：値は `Fixnum`、`Bignum`、または `Float` である必要があります。割り当てる値は、対象の型で正確に表現可能である必要があります。したがって、int32 フィールドに `1.0` を割り当てることはできますが、`1.2` を割り当てることはできません。
- **ブールフィールド**：値は `true` または `false` である必要があります。true/false 以外の値は自動的に true/false に変換されません。
- **バイトフィールド**：割り当てられる値は `String` オブジェクトである必要があります。protobuf ライブラリは文字列を複製し、ASCII-8BIT エンコーディングに変換し、凍結します。
- **文字列フィールド**：割り当てられる値は `String` オブジェクトである必要があります。protobuf ライブラリは文字列を複製し、UTF-8 エンコーディングに変換し、凍結します。

自動的な `#to_s`、`#to_i` などの呼び出しは自動変換を行いません。必要に応じて、最初に値を自分で変換する必要があります。

#### 存在の確認

`optional` フィールドを使用する場合、フィールドの存在は生成された `has_...?` メソッドを呼び出すことで確認されます。任意の値を設定すると、デフォルト値でさえフィールドが存在することを示します。フィールドは、異なる生成された `clear_...` メソッドを呼び出すことでクリアできます。例えば、int32 フィールド `foo` を持つメッセージ `MyMessage` の場合：```

```ruby
m = MyMessage.new
raise unless !m.has_foo?
m.foo = 0
raise unless m.has_foo?
m.clear_foo
raise unless !m.has_foo?
```

### 単数メッセージフィールド {#embedded_message}

サブメッセージの場合、未設定のフィールドは `nil` を返しますので、メッセージが明示的に設定されたかどうかを常に判別できます。サブメッセージフィールドをクリアするには、その値を明示的に `nil` に設定します。

```ruby
if message.submessage_field.nil?
  puts "Submessage field is unset."
else
  message.submessage_field = nil
  puts "Cleared submessage field."
end
```

`nil` の比較や代入に加えて、生成されたメッセージには `has_...` メソッドと `clear_...` メソッドがあり、基本型と同じように動作します。

```ruby
if message.has_submessage_field?
  raise unless message.submessage_field == nil
  puts "Submessage field is unset."
else
  raise unless message.submessage_field != nil
  message.clear_submessage_field
  raise unless message.submessage_field == nil
  puts "Cleared submessage field."
end
```

サブメッセージを割り当てる場合、正しい型の生成されたメッセージオブジェクトである必要があります。

サブメッセージを割り当てると、メッセージサイクルを作成することができます。例えば：

```proto
// foo.proto
message RecursiveMessage {
  RecursiveMessage submessage = 1;
}

# test.rb

require 'foo'

message = RecursiveSubmessage.new
message.submessage = message
```

これをシリアライズしようとすると、ライブラリはサイクルを検出してシリアライズに失敗します。

### 繰り返しフィールド

繰り返しフィールドはカスタムクラス `Google::Protobuf::RepeatedField` を使用して表されます。このクラスは Ruby の `Array` のように動作し、`Enumerable` をミックスインします。通常の Ruby 配列とは異なり、`RepeatedField` は特定の型で構築され、すべての配列メンバーが正しい型であることを期待します。型と範囲はメッセージフィールドと同様にチェックされます。

```ruby
int_repeatedfield = Google::Protobuf::RepeatedField.new(:int32, [1, 2, 3])

raise unless !int_repeatedfield.empty?

# Raises TypeError.
int_repeatedfield[2] = "not an int32"

# Raises RangeError
int_repeatedfield[2] = 2**33

message.int32_repeated_field = int_repeatedfield

# This isn't allowed; the regular Ruby array doesn't enforce types like we need.
message.int32_repeated_field = [1, 2, 3, 4]

# This is fine, since the elements are copied into the type-safe array.
message.int32_repeated_field += [1, 2, 3, 4]

# The elements can be cleared without reassigning.
int_repeatedfield.clear
raise unless int_repeatedfield.empty?
```

`RepeatedField` 型は通常の Ruby `Array` と同じメソッドをサポートします。`repeated_field.to_a` で通常の Ruby 配列に変換できます。

単数フィールドとは異なり、繰り返しフィールドには決して `has_...?` メソッドは生成されません。

### マップフィールド

マップフィールドは、Ruby の `Hash` のように動作する特別なクラス `Google::Protobuf::Map` を使用して表されます。通常の Ruby ハッシュとは異なり、`Map` はキーと値の特定の型で構築され、マップのすべてのキーと値が正しい型であることを期待します。型と範囲はメッセージフィールドと `RepeatedField` 要素と同様にチェックされます。

```ruby
int_string_map = Google::Protobuf::Map.new(:int32, :string)

# Returns nil; items is not in the map.
print int_string_map[5]

# Raises TypeError, value should be a string
int_string_map[11] = 200

# Ok.
int_string_map[123] = "abc"

message.int32_string_map_field = int_string_map
```

## 列挙型 {#enum}

Ruby にはネイティブの列挙型がないため、各列挙型に値を定義する定数を持つモジュールを作成します。`.proto` ファイルが与えられた場合：

```proto
message Foo {
  enum SomeEnum {
    VALUE_A = 0;
    VALUE_B = 5;
    VALUE_C = 1234;
  }
  optional SomeEnum bar = 1;
}
```

```ruby
enum値は次のように参照できます：

```ruby
print Foo::SomeEnum::VALUE_A  # => 0
message.bar = Foo::SomeEnum::VALUE_A
```

enumフィールドには、数値またはシンボルのいずれかを割り当てることができます。値を読み取るとき、enum値が既知の場合はシンボルになり、未知の場合は数値になります。**proto3**はオープンなenumセマンティクスを使用しているため、enumフィールドには、enumで定義されていなくても任意の数値を割り当てることができます。

```ruby
message.bar = 0
puts message.bar.inspect  # => :VALUE_A
message.bar = :VALUE_B
puts message.bar.inspect  # => :VALUE_B
message.bar = 999
puts message.bar.inspect  # => 999

# Raises: RangeError: Unknown symbol value for enum field.
message.bar = :UNDEFINED_VALUE

# Switching on an enum value is convenient.
case message.bar
when :VALUE_A
  # ...
when :VALUE_B
  # ...
when :VALUE_C
  # ...
else
  # ...
end
```

enumモジュールは次のユーティリティメソッドも定義しています：

-   `Enum#lookup(number)`: 指定された数値を検索し、その名前を返します。見つからない場合は`nil`を返します。この数値に複数の名前がある場合は、最初に定義された名前を返します。
-   `Enum#resolve(symbol)`: このenum名の数値を返します。見つからない場合は`nil`を返します。
-   `Enum#descriptor`: このenumの記述子を返します。

## Oneof

oneofを持つメッセージがあるとします：

```proto
message Foo {
  oneof test_oneof {
     string name = 1;
     int32 serial_number = 2;
  }
}
```

`Foo`に対応するRubyクラスには、`name`と`serial_number`という名前のメンバーがあり、通常の[field](#fields)と同様のアクセサメソッドがあります。ただし、通常のフィールドとは異なり、oneof内のフィールドは1つだけ設定できるため、1つのフィールドを設定すると他のフィールドはクリアされます。

```ruby
message = Foo.new

# Fields have their defaults.
raise unless message.name == ""
raise unless message.serial_number == 0
raise unless message.test_oneof == nil

message.name = "Bender"
raise unless message.name == "Bender"
raise unless message.serial_number == 0
raise unless message.test_oneof == :name

# Setting serial_number clears name.
message.serial_number = 2716057
raise unless message.name == ""
raise unless message.test_oneof == :serial_number

# Setting serial_number to nil clears the oneof.
message.serial_number = nil
raise unless message.test_oneof == nil
```

proto2メッセージの場合、oneofメンバーには個々の`has_...?`メソッドもあります：

```ruby
message = Foo.new

raise unless !message.has_test_oneof?
raise unless !message.has_name?
raise unless !message.has_serial_number?
raise unless !message.has_test_oneof?

message.name = "Bender"
raise unless message.has_test_oneof?
raise unless message.has_name?
raise unless !message.has_serial_number?
raise unless !message.has_test_oneof?
```
