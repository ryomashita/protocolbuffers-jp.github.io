+++
title = "Python 生成コードガイド"
weight = 750
linkTitle = "生成コードガイド"
description = "与えられたプロトコル定義に対してプロトコルバッファコンパイラが生成する Python 定義を正確に説明します。"
type = "docs"
+++

プロト2とプロト3で生成されるコードの違いは強調されています - これらの違いは、このドキュメントで説明されている生成されたコードにあります。ベースのメッセージクラス/インターフェースは両バージョンで同じです。このドキュメントを読む前に、[プロト2言語ガイド](/programming-guides/proto2)および/または[プロト3言語ガイド](/programming-guides/proto3)を読む必要があります。

Python Protocol Buffers の実装は、C++やJavaとは少し異なります。Pythonでは、コンパイラは生成されたクラスの記述子を構築するためのコードのみを出力し、[Python メタクラス](https://docs.python.org/2.7/reference/datamodel#metaclasses)が実際の作業を行います。このドキュメントは、メタクラスが適用された後に得られる内容を説明します。

## コンパイラの呼び出し {#invocation}

プロトコルバッファコンパイラは、`--python_out=` コマンドラインフラグを使用して呼び出されると、Python 出力を生成します。`--python_out=` オプションのパラメータは、コンパイラが Python 出力を書き込むディレクトリです。コンパイラは、各 `.proto` ファイルの入力ごとに `.py` ファイルを作成します。出力ファイルの名前は、次の2つの変更を行うことで計算されます。

-   拡張子（`.proto`）は `_pb2.py` に置き換えられます。
-   proto パス（`--proto_path=` または `-I` コマンドラインフラグで指定）は、出力パス（`--python_out=` フラグで指定）に置き換えられます。

例えば、次のようにコンパイラを呼び出すとします：

```shell
protoc --proto_path=src --python_out=build/gen src/foo.proto src/bar/baz.proto
```

コンパイラは、`src/foo.proto` と `src/bar/baz.proto` ファイルを読み込み、2つの出力ファイル `build/gen/foo_pb2.py` と `build/gen/bar/baz_pb2.py` を生成します。必要に応じてコンパイラはディレクトリ `build/gen/bar` を自動的に作成しますが、`build` または `build/gen` は作成しません。それらはすでに存在している必要があります。

Protocは、`--pyi_out`パラメータを使用してPythonスタブ（`.pyi`）を生成できます。

`.proto`ファイルまたはそのパスにPythonモジュール名として使用できない文字が含まれている場合（例：ハイフンなど）、それらはアンダースコアに置き換えられます。したがって、ファイル`foo-bar.proto`はPythonファイル`foo_bar_pb2.py`になります。

{{% alert title="ヒント" color="note" %}}Pythonコードを出力する際、プロトコルバッファコンパイラがZIPアーカイブに直接出力できる機能は特に便利です。Pythonインタプリタは、これらのアーカイブが`PYTHONPATH`に配置されている場合に直接読み取ることができます。ZIPファイルに出力するには、単に`.zip`で終わる出力場所を指定します。{{% /alert %}}

{{% alert title="注意" color="note" %}}
拡張子`_pb2.py`の中の数字2は、Protocol Buffersのバージョン2を示しています。バージョン1は主にGoogle内部で使用されていましたが、Protocol Buffersより前にリリースされた他のPythonコードの一部に含まれている可能性があります。Python Protocol Buffersのバージョン2は、完全に異なるインターフェースを持っており、Pythonにはコンパイル時の型チェックがないため、間違いをキャッチするためのコンパイル時の型チェックがないため、生成されたPythonファイル名の一部としてバージョン番号を目立たせることを選択しました。現在、proto2とproto3の両方が生成されたファイルに`_pb2.py`を使用しています。{{% /alert %}}

## パッケージ {#package}

プロトコルバッファコンパイラによって生成されたPythonコードは、`.proto`ファイルで定義されたパッケージ名には全く影響を受けません。代わりに、Pythonパッケージはディレクトリ構造によって識別されます。

## メッセージ {#message}

単純なメッセージ宣言がある場合：

```proto
message Foo {}
```

プロトコルバッファコンパイラは、`google.protobuf.Message`をサブクラス化した`Foo`というクラスを生成します。このクラスは具象クラスであり、未実装の抽象メソッドはありません。C++やJavaとは異なり、Python生成コードは`.proto`ファイルの`optimize_for`オプションに影響を受けません。実際には、すべてのPythonコードはコードサイズに最適化されています。

Pythonのキーワードと競合する名前の場合、そのクラスは[*Pythonのキーワードと競合する名前*](#keyword-conflicts)セクションで説明されているように`getattr()`を介してのみアクセス可能です。

独自の`Foo`のサブクラスを作成しないでください。生成されたクラスはサブクラス化のために設計されておらず、「壊れやすい基底クラス」の問題を引き起こす可能性があります。さらに、実装継承は悪い設計です。

Pythonのメッセージクラスには、`Message`インターフェースで定義されたものと、ネストされたフィールド、メッセージ、列挙型で生成されたものを除いて、特に公開メンバーはありません（以下で説明）。`Message`は、バイナリ文字列からのパースやシリアライズを含む、メッセージ全体のチェック、操作、読み取り、書き込みに使用できるメソッドを提供します。これらのメソッドに加えて、`Foo`クラスは以下の静的メソッドを定義しています：

-   `FromString(s)`: 指定された文字列からデシリアライズされた新しいメッセージインスタンスを返します。

プロトコルメッセージをテキスト形式で扱うために[`text_format`](https://googleapis.dev/python/protobuf/latest/google/protobuf/text_format.html)モジュールを使用することもできます。たとえば、`Merge()`メソッドを使用すると、メッセージのASCII表現を既存のメッセージにマージできます。

### ネストされた型 {#nested-types}

メッセージは他のメッセージの内部で宣言できます。例：

```proto
message Foo {
  message Bar {}
}
```

この場合、`Bar`クラスは`Foo`の静的メンバーとして宣言されているため、`Foo.Bar`として参照できます。

## Well Known Types {#wkt}

Protocol Buffersは、独自のメッセージ型と一緒に使用できるいくつかの[well-known types](/reference/protobuf/google.protobuf)を提供しています。一部のWKTメッセージには、通常のプロトコルバッファメッセージメソッドに加えて特別なメソッドがあり、これらは[`google.protobuf.Message`](https://googleapis.dev/python/protobuf/latest/google/protobuf/message.html#google.protobuf.message.Message)とWKTクラスの両方をサブクラス化しています。

### Any {#any}

Anyメッセージの場合、`Pack()`を呼び出して指定されたメッセージを現在のAnyメッセージにパックしたり、`Unpack()`を呼び出して現在のAnyメッセージを指定されたメッセージにアンパックしたりできます。例：

```python
any_message.Pack(message)
any_message.Unpack(message)
```

`Unpack()`は、渡されたメッセージオブジェクトの記述子を格納されたものと照合し、一致しない場合は`False`を返し、アンパッキングを試みません。一致する場合は`True`を返します。

また、`Is()`メソッドを呼び出して、Anyメッセージが指定されたプロトコルバッファタイプを表しているかどうかを確認できます。例：

```python
assert any_message.Is(message.DESCRIPTOR)
```

`TypeName()`メソッドを使用して、内部メッセージのprotobufタイプ名を取得できます。

### タイムスタンプ {#timestamp}

タイムスタンプメッセージは、RFC 3339日付文字列形式（JSON文字列）に変換するために`ToJsonString()`/`FromJsonString()`メソッドを使用できます。例：

```python
timestamp_message.FromJsonString("1970-01-01T00:00:00Z")
assert timestamp_message.ToJsonString() == "1970-01-01T00:00:00Z"
```

また、`GetCurrentTime()`を呼び出して、タイムスタンプメッセージを現在時刻で埋めることもできます。

```python
timestamp_message.GetCurrentTime()
```

エポック以降の他の時間単位間の変換には、`ToNanoseconds()、FromNanoseconds()、ToMicroseconds()、FromMicroseconds()、ToMilliseconds()、FromMilliseconds()、ToSeconds()`、`FromSeconds()`を呼び出すことができます。生成されたコードには、Pythonのdatetimeオブジェクトとタイムスタンプ間の変換を行う`ToDatetime()`および`FromDatetime()`メソッドもあります。例：

```python
timestamp_message.FromMicroseconds(-1)
assert timestamp_message.ToMicroseconds() == -1
dt = datetime(2016, 1, 1)
timestamp_message.FromDatetime(dt)
self.assertEqual(dt, timestamp_message.ToDatetime())
```

### 持続時間 {#duration}

持続時間メッセージは、JSON文字列と他の時間単位間の変換に関するタイムスタンプと同じメソッドを持っています。timedeltaとDurationの間で変換するには、`ToTimedelta()`または`FromTimedelta`を呼び出すことができます。例：

```python
duration_message.FromNanoseconds(1999999999)
td = duration_message.ToTimedelta()
assert td.seconds == 1
assert td.microseconds == 999999
```

### FieldMask {#fieldmask}

FieldMaskメッセージは、`ToJsonString()`/`FromJsonString()`メソッドを使用してJSON文字列に変換および変換できます。さらに、FieldMaskメッセージには次のメソッドがあります：

- `IsValidForDescriptor(message_descriptor)`: FieldMaskがメッセージ記述子に対して有効かどうかをチェックします。
- `AllFieldsFromDescriptor(message_descriptor)`: メッセージ記述子のすべての直接フィールドをFieldMaskに取得します。
- `CanonicalFormFromMask(mask)`: FieldMaskを正規形に変換します。
- `Union(mask1, mask2)`: 2つのFieldMaskをこのFieldMaskにマージします。
- `Intersect(mask1, mask2)`: 2つのFieldMaskをこのFieldMaskにインターセクトします。
- `MergeMessage(source, destination, replace_message_field=False, replace_repeated_field=False)`: FieldMaskで指定されたフィールドをソースからデスティネーションにマージします。

### Struct {#struct}

Structメッセージを使用すると、アイテムを直接取得および設定できます。例：

```python
struct_message["key1"] = 5
struct_message["key2"] = "abc"
struct_message["key3"] = True
```

リスト/構造体を取得または作成するには、`get_or_create_list()`/`get_or_create_struct()` を呼び出すことができます。例：

```python
struct.get_or_create_struct("key4")["subkey"] = 11.0
struct.get_or_create_list("key5")
```

### ListValue {#listvalue}

ListValueメッセージは、Pythonのシーケンスのように機能し、次のような操作ができます：

```python
list_value = struct_message.get_or_create_list("key")
list_value.extend([6, "seven", True, None])
list_value.append(False)
assert len(list_value) == 5
assert list_value[0] == 6
assert list_value[1] == "seven"
assert list_value[2] == True
assert list_value[3] == None
assert list_Value[4] == False
```

ListValue/Structを追加するには、`add_list()`/`add_struct()` を呼び出します。例：

```python
list_value.add_struct()["key"] = 1
list_value.add_list().extend([1, "two", True])
```

## Fields {#fields}

メッセージタイプの各フィールドについて、対応するクラスにはフィールドと同じ名前のプロパティがあります。プロパティを操作する方法は、そのタイプによって異なります。

プロパティと同様に、コンパイラは各フィールドに対して整数定数を生成し、そのフィールド番号を含みます。定数名は、フィールド名を大文字に変換してから `_FIELD_NUMBER` を追加したものです。たとえば、フィールド `optional int32 foo_bar = 5;` が与えられた場合、コンパイラは定数 `FOO_BAR_FIELD_NUMBER = 5` を生成します。

フィールドの名前がPythonのキーワードの場合、そのプロパティは `getattr()` および `setattr()` を介してのみアクセス可能であり、[*Pythonのキーワードと競合する名前*](#keyword-conflicts) セクションで説明されているようになります。

### Singular Fields (proto2) {#singular-fields-proto2}

任意の非メッセージタイプの単数の（オプションまたは必須の）フィールド `foo` がある場合、`foo` フィールドを通常のフィールドとして操作できます。たとえば、`foo` のタイプが `int32` の場合、次のようにできます：

```python
message.foo = 123
print(message.foo)
```

`foo` を間違ったタイプの値に設定すると、`TypeError` が発生します。

`foo` が設定されていない状態で読み取られると、そのフィールドのデフォルト値がその値となります。`foo` が設定されているかどうかを確認したり、`foo` の値をクリアするには、[`Message`](https://googleapis.dev/python/protobuf/latest/google/protobuf/message.html#google.protobuf.message.Message) インターフェースの `HasField()` メソッドまたは `ClearField()` メソッドを呼び出す必要があります。例：

```python
assert not message.HasField("foo")
message.foo = 123
assert message.HasField("foo")
message.ClearField("foo")
assert not message.HasField("foo")
```

### 単数フィールド（proto3）{#singular-fields-proto3}

もし、任意の非メッセージ型の単数フィールド `foo` がある場合、そのフィールド `foo` を通常のフィールドのように操作できます。例えば、もし `foo` の型が `int32` である場合、以下のようにできます：

```python
message.foo = 123
print(message.foo)
```

`foo` を間違った型の値に設定すると `TypeError` が発生することに注意してください。

もし `foo` が設定されていない状態で読み取られた場合、そのフィールドのデフォルト値がその値となります。`foo` の値をクリアして、その型のデフォルト値にリセットするには、[`Message`](https://googleapis.dev/python/protobuf/latest/google/protobuf/message.html#google.protobuf.message.Message) インターフェースの `ClearField()` メソッドを呼び出します。例えば：

```python
message.foo = 123
message.ClearField("foo")
```

### 単数メッセージフィールド {#embedded_message}

メッセージ型は少し異なる動作をします。埋め込まれたメッセージフィールドに値を割り当てることはできません。代わりに、子メッセージ内の任意のフィールドに値を割り当てることは、親のメッセージフィールドを設定することを意味します。また、親メッセージの `HasField()` メソッドを使用して、メッセージ型フィールドの値が設定されているかどうかを確認できます。

例えば、次の `.proto` 定義があるとします：

```proto
message Foo {
  optional Bar bar = 1;
}
message Bar {
  optional int32 i = 1;
}
```

以下のようなことはできません：

```python
foo = Foo()
foo.bar = Bar()  # 間違い！
```

代わりに、`bar` を設定するには、単純に `bar` 内のフィールドに直接値を割り当てるだけで、`foo` に `bar` フィールドができます：

```python
foo = Foo()
assert not foo.HasField("bar")
foo.bar.i = 1
assert foo.HasField("bar")
assert foo.bar.i == 1
foo.ClearField("bar")
assert not foo.HasField("bar")
assert foo.bar.i == 0  # Default value
```

同様に、[`Message`](https://googleapis.dev/python/protobuf/latest/google/protobuf/message.html#google.protobuf.message.Message) インターフェースの `CopyFrom()` メソッドを使用して `bar` を設定できます。これにより、`bar` と同じ型の別のメッセージからすべての値がコピーされます。

```python
foo.bar.CopyFrom(baz)
```

`bar` 内のフィールドを単に読み取るだけでは、そのフィールドは設定されません：

```python
foo = Foo()
assert not foo.HasField("bar")
print(foo.bar.i)  # Print i's default value
assert not foo.HasField("bar")
```

もし、設定する必要のないメッセージに \"has\" ビットが必要な場合は、`SetInParent()` メソッドを使用できます。

```python
foo = Foo()
assert not foo.HasField("bar")
foo.bar.SetInParent()  # Set Foo.bar to a default Bar message
assert foo.HasField("bar")
```

### 繰り返しフィールド {#repeated-fields}

繰り返しフィールドは、Pythonのシーケンスのように振る舞うオブジェクトとして表されます。
埋め込まれたメッセージと同様に、フィールドに直接値を割り当てることはできませんが、操作することはできます。たとえば、次のメッセージ定義が与えられた場合：

```proto
message Foo {
  repeated int32 nums = 1;
}
```

次のようにできます：

```python
foo = Foo()
foo.nums.append(15)        # Appends one value
foo.nums.extend([32, 47])  # Appends an entire list

assert len(foo.nums) == 3
assert foo.nums[0] == 15
assert foo.nums[1] == 32
assert foo.nums == [15, 32, 47]

foo.nums[:] = [33, 48]     # Assigns an entire list
assert foo.nums == [33, 48]

foo.nums[1] = 56    # Reassigns a value
assert foo.nums[1] == 56
for i in foo.nums:  # Loops and print
  print(i)
del foo.nums[:]     # Clears list (works just like in a Python list)
```

[`Message`](https://googleapis.dev/python/protobuf/latest/google/protobuf/message.html#google.protobuf.message.Message) インターフェースの `ClearField()` メソッドを使用することも、Python の `del` を使用することもできます。

値を取得するためにインデックスを使用する場合、負の数を使用して、リスト内の最後の要素を取得することができます。インデックスが範囲外になると、`IndexError: list index out of range` が発生します。

### 繰り返しメッセージフィールド {#repeated-message-fields}

繰り返しメッセージフィールドは、繰り返しスカラーフィールドと同様に機能します。ただし、対応する Python オブジェクトには、新しいメッセージオブジェクトを作成し、リストに追加して呼び出し元が埋め込むための `add()` メソッドもあります。また、オブジェクトの `append()` メソッドは、与えられたメッセージのコピーを作成し、そのコピーをリストに追加します。これは、メッセージが常に親メッセージによって所有されるようにするために行われます。これにより、可変データ構造が複数の所有者を持つ場合に発生する循環参照やその他の混乱を回避できます。同様に、オブジェクトの `extend()` メソッドは、メッセージのリスト全体を追加しますが、リスト内の各メッセージのコピーを作成します。

たとえば、次のメッセージ定義が与えられた場合：

```proto
message Foo {
  repeated Bar bars = 1;
}
message Bar {
  optional int32 i = 1;
  optional int32 j = 2;
}
```

次のようにできます：

```python
foo = Foo()
bar = foo.bars.add()        # Adds a Bar then modify
bar.i = 15
foo.bars.add().i = 32       # Adds and modify at the same time
new_bar = Bar()
new_bar.i = 40
another_bar = Bar()
another_bar.i = 57
foo.bars.append(new_bar)        # Uses append() to copy
foo.bars.extend([another_bar])  # Uses extend() to copy

assert len(foo.bars) == 4
assert foo.bars[0].i == 15
assert foo.bars[1].i == 32
assert foo.bars[2].i == 40
assert foo.bars[2] == new_bar      # The appended message is equal,
assert foo.bars[2] is not new_bar  # but it is a copy!
assert foo.bars[3].i == 57
assert foo.bars[3] == another_bar      # The extended message is equal,
assert foo.bars[3] is not another_bar  # but it is a copy!

foo.bars[1].i = 56    # Modifies a single element
assert foo.bars[1].i == 56
for bar in foo.bars:  # Loops and print
  print(bar.i)
del foo.bars[:]       # Clears list

# add() also forwards keyword arguments to the concrete class.
# For example, you can do:

foo.bars.add(i=12, j=13)

# Initializers forward keyword arguments to a concrete class too.
# For example:

foo = Foo(             # Creates Foo
  bars=[               # with its field bars set to a list
    Bar(i=15, j=17),   # where each list member is also initialized during creation.
    Bar(i=32),
    Bar(i=47, j=77),
  ]
)

assert len(foo.bars) == 3
assert foo.bars[0].i == 15
assert foo.bars[0].j == 17
assert foo.bars[1].i == 32
assert foo.bars[2].i == 47
assert foo.bars[2].j == 77
```

繰り返しスカラーフィールドとは異なり、繰り返しメッセージフィールドはアイテムの割り当てをサポートしません。

### グループ (proto2) {#groups-proto2}

**新しいメッセージタイプを作成する際には、グループは非推奨であり、代わりにネストされたメッセージタイプを使用してください。**

グループは、ネストされたメッセージタイプとフィールドを1つの宣言に組み合わせ、メッセージに対して異なる[ワイヤーフォーマット](/programming-guides/encoding)を使用します。生成されたメッセージはグループと同じ名前になります。生成されたフィールドの名前はグループの**小文字化**された名前です。

たとえば、ワイヤーフォーマットを除いて、次の2つのメッセージ定義は同等です:

```python
// Version 1: Using groups
message SearchResponse {
  repeated group SearchResult = 1 {
    optional string url = 1;
  }
}
// Version 2: Not using groups
message SearchResponse {
  message SearchResult {
    optional string url = 1;
  }
  repeated SearchResult searchresult = 1;
}
```

グループは`required`、`optional`、または`repeated`のいずれかです。`required`または`optional`のグループは、通常の単数メッセージフィールドと同じAPIを使用して操作されます。`repeated`グループは、通常の繰り返しメッセージフィールドと同じAPIを使用して操作されます。

たとえば、上記の`SearchResponse`定義がある場合、次のようにできます:

```python
resp = SearchResponse()
resp.searchresult.add(url="https://blog.google")
assert resp.searchresult[0].url == "https://blog.google"
assert resp.searchresult[0] == SearchResponse.SearchResult(url="https://blog.google")
```

### マップフィールド {#map-fields}

次のメッセージ定義が与えられた場合:

```proto
message MyMessage {
  map<int32, int32> mapfield = 1;
}
```

マップフィールドの生成されたPython APIは、Pythonの`dict`と同様です:

```python
# Assign value to map
m.mapfield[5] = 10

# Read value from map
m.mapfield[5]

# Iterate over map keys
for key in m.mapfield:
  print(key)
  print(m.mapfield[key])

# Test whether key is in map:
if 5 in m.mapfield:
  print(“Found!”)

# Delete key from map.
del m.mapfield[key]
```

[埋め込みメッセージフィールド](#embedded_message)と同様に、メッセージは直接マップ値に割り当てることはできません。代わりに、メッセージをマップ値として追加するには、[未定義のキー](#undefined)を参照し、新しいサブメッセージを構築して返します:

```python
m.message_map[key].submessage_field = 10
```

次のセクションで未定義のキーについて詳しく説明します。

#### 未定義のキーの参照 {#undefined}

Protocol Bufferマップのセマンティクスは、未定義のキーに関してPythonの`dict`とわずかに異なる動作をします。通常のPythonの`dict`では、未定義のキーを参照するとKeyError例外が発生します:

```python
>>> x = {}
>>> x[5]
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
KeyError: 5
```

ただし、Protocol Buffersマップでは、未定義のキーを参照すると、マップ内にキーが作成され、ゼロ/偽/空の値が設定されます。この動作は、Python標準ライブラリの`defaultdict`に似ています。

```python
>>> dict(m.mapfield)
{}
>>> m.mapfield[5]
0
>>> dict(m.mapfield)
{5: 0}
```

この動作は、メッセージ型の値を持つマップに特に便利です。なぜなら、返されたメッセージのフィールドを直接更新できるからです。

```python
>>> m.message_map[5].foo = 3
```

メッセージフィールドに値を割り当てなくても、サブメッセージはマップ内に作成されます：

```python
>>> m.message_map[10]
<test_pb2.M2 object at 0x7fb022af28c0>
>>> dict(m.message_map)
{10: <test_pb2.M2 object at 0x7fb022af28c0>}
```

これは通常の[埋め込みメッセージフィールド](#embedded_message)とは**異なり**、メッセージ自体はフィールドのいずれかに値を割り当てるときにのみ作成されます。

コードを読む人にとって、たとえば`m.message_map[10]`だけでサブメッセージが作成される可能性が明らかでない場合、同じことを行う`get_or_create()`メソッドも提供していますが、その名前が可能なメッセージ作成をより明示的にします：

```python
# Equivalent to:
#   m.message_map[10]
# but more explicit that the statement might be creating a new
# empty message in the map.
m.message_map.get_or_create(10)
```

## 列挙型 {#enum}

Pythonでは、列挙型は単なる整数です。列挙型の定義値に対応する整数定数のセットが定義されます。たとえば、次のように与えられた場合：

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

定数`VALUE_A`、`VALUE_B`、`VALUE_C`はそれぞれ値0、5、1234で定義されています。必要に応じて`SomeEnum`にアクセスできます。列挙型が外部スコープで定義されている場合、値はモジュール定数になります。メッセージ内で定義されている場合、それらはそのメッセージクラスの静的メンバになります。

たとえば、proto内の次の列挙型に対して、次の3つの方法で値にアクセスできます：

```proto
enum SomeEnum {
    VALUE_A = 0;
    VALUE_B = 5;
    VALUE_C = 1234;
}
```

```python
value_a = myproto_pb2.SomeEnum.VALUE_A
# or
myproto_pb2.VALUE_A
# or
myproto_pb2.SomeEnum.Value('VALUE_A')
```

列挙型フィールドはスカラーフィールドと同様に機能します。

```python
foo = Foo()
foo.bar = Foo.VALUE_A
assert foo.bar == 0
assert foo.bar == Foo.VALUE_A
```

列挙型の名前（または列挙値）がPythonのキーワードである場合、そのオブジェクト（または列挙値のプロパティ）は、[*Pythonのキーワードと競合する名前*](#keyword-conflicts)セクションで説明されているように、`getattr()`を介してのみアクセスできます。

列挙型で設定できる値は、Protocol Buffersのバージョンによって異なります：

- **proto2**では、列挙型には、列挙型のタイプで定義されていない数値を含めることはできません。列挙型に含まれていない値を割り当てると、生成されたコードは例外をスローします。
- **proto3**はオープンな列挙型セマンティクスを使用します：列挙型フィールドには任意の`int32`値を含めることができます。

列挙型には、値からフィールド名を取得したり、その逆を行ったりするためのユーティリティメソッドがいくつかあります。これらは[`enum_type_wrapper.EnumTypeWrapper`](https://github.com/protocolbuffers/protobuf/blob/master/python/google/protobuf/internal/enum_type_wrapper.py) で定義されています（生成された列挙型クラスの基本クラス）。たとえば、`myproto.proto` に次の独立した列挙型がある場合：

```proto
enum SomeEnum {
    VALUE_A = 0;
    VALUE_B = 5;
    VALUE_C = 1234;
}
```

...次のようにすることができます：

```python
self.assertEqual('VALUE_A', myproto_pb2.SomeEnum.Name(myproto_pb2.VALUE_A))
self.assertEqual(5, myproto_pb2.SomeEnum.Value('VALUE_B'))
```

Foo内で宣言された列挙型の場合、構文は似ています：

```python
self.assertEqual('VALUE_A', myproto_pb2.Foo.SomeEnum.Name(myproto_pb2.Foo.VALUE_A))
self.assertEqual(5, myproto_pb2.Foo.SomeEnum.Value('VALUE_B'))
```

複数の列挙定数が同じ値を持つ場合（エイリアス）、最初に定義された定数が返されます。

```proto
enum SomeEnum {
    option allow_alias = true;
    VALUE_A = 0;
    VALUE_B = 5;
    VALUE_C = 1234;
    VALUE_B_ALIAS = 5;
}
```

上記の例では、`myproto_pb2.SomeEnum.Name(5)` は `"VALUE_B"` を返します。

## Oneof {#oneof}

oneofを持つメッセージがあるとします：

```proto
message Foo {
  oneof test_oneof {
     string name = 1;
     int32 serial_number = 2;
  }
}
```

`Foo` に対応するPythonクラスには、通常の[field](#fields)と同様に `name` と `serial_number` という名前のプロパティがあります。ただし、通常のフィールドとは異なり、oneof内のフィールドは1つだけ設定されることがあります。これはランタイムによって保証されています。たとえば：

```python
message = Foo()
message.name = "Bender"
assert message.HasField("name")
message.serial_number = 2716057
assert message.HasField("serial_number")
assert not message.HasField("name")
```

メッセージクラスには、`WhichOneof` メソッドもあり、oneof内で設定されたフィールド（あれば）を見つけることができます。このメソッドは、設定されているフィールドの名前を返し、何も設定されていない場合は `None` を返します：

```python
assert message.WhichOneof("test_oneof") is None
message.name = "Bender"
assert message.WhichOneof("test_oneof") == "name"
```

`HasField` と `ClearField` は、フィールド名に加えてoneof名も受け入れます：

```python
assert not message.HasField("test_oneof")
message.name = "Bender"
assert message.HasField("test_oneof")
message.serial_number = 2716057
assert message.HasField("test_oneof")
message.ClearField("test_oneof")
assert not message.HasField("test_oneof")
assert not message.HasField("serial_number")
```

oneofに対して `ClearField` を呼び出すと、現在設定されているフィールドがクリアされることに注意してください。

## Pythonのキーワードと競合する名前 {#keyword-conflicts}

```markdown
Pythonのキーワードであるメッセージ、フィールド、列挙型、または列挙値の名前の場合、対応するクラスやプロパティの名前は同じになりますが、Pythonの通常の属性参照構文（つまり、ドット演算子）ではアクセスできず、Pythonの`getattr()`および`setattr()`組み込み関数を使用してのみアクセスできます。

たとえば、次の`.proto`定義がある場合：

```proto
message Baz {
  optional int32 from = 1
  repeated int32 in = 2;
}
```

これらのフィールドにアクセスするには、次のようにします：

```python
baz = Baz()
setattr(baz, "from", 99)
assert getattr(baz, "from") == 99
getattr(baz, "in").append(42)
assert getattr(baz, "in") == [42]
```

一方、これらのフィールドにアクセスするために`obj.attr`構文を使用しようとすると、Pythonはコードを解析する際に構文エラーを発生させます：

```python
# 間違い！
baz.in  # 構文エラー: 無効な構文
baz.from  # 構文エラー: 無効な構文
```

## 拡張機能（proto2のみ） {#extension}

拡張範囲を持つメッセージがある場合：

```proto
message Foo {
  extensions 100 to 199;
}
```

`Foo`に対応するPythonクラスには、`Extensions`というメンバーがあり、これは拡張識別子をその現在の値にマッピングする辞書です。

拡張定義がある場合：

```proto
extend Foo {
  optional int32 bar = 123;
}
```

プロトコルバッファコンパイラは`bar`という「拡張識別子」を生成します。この識別子は`Extensions`辞書のキーとして機能します。この辞書で値を検索する結果は、同じ型の通常のフィールドにアクセスした場合とまったく同じです。したがって、上記の例では、次のようにできます：

```python
foo = Foo()
foo.Extensions[proto_file_pb2.bar] = 2
assert foo.Extensions[proto_file_pb2.bar] == 2
```

拡張識別子定数を指定する必要があることに注意してください。これは、異なるスコープで同じ名前の複数の拡張機能が指定される可能性があるためです。

通常のフィールドと同様に、`Extensions[...]`は、単数メッセージの場合はメッセージオブジェクトを、繰り返しフィールドの場合はシーケンスを返します。
```

[`Message`](https://googleapis.dev/python/protobuf/latest/google/protobuf/message.html#google.protobuf.message.Message) インターフェースの `HasField()` および `ClearField()` メソッドは拡張機能とは動作しません。代わりに `HasExtension()` および `ClearExtension()` を使用する必要があります。`HasExtension()` および `ClearExtension()` メソッドを使用するには、存在を確認する拡張機能の `field_descriptor` を渡します。

## サービス {#service}

`.proto` ファイルに次の行が含まれている場合:

```proto
option py_generic_services = true;
```

その場合、プロトコルバッファコンパイラは、このセクションで説明されているファイル内のサービス定義に基づいてコードを生成します。ただし、生成されるコードは特定の RPC システムに結びついていないため、1 つのシステムに適合したコードよりもさらに多くの間接レベルが必要となります。このコードを生成したくない場合は、次の行をファイルに追加してください:

```proto
option py_generic_services = false;
```

上記のいずれの行も指定されていない場合、オプションは `false` にデフォルトで設定されます。汎用サービスは非推奨です。 (2.4.0 より前では、オプションは `true` にデフォルトで設定されていました)

`.proto` 言語のサービス定義に基づく RPC システムは、システムに適したコードを生成するための[プラグイン](/reference/cpp/api-docs/google.protobuf.compiler.plugin.pb)を提供する必要があります。これらのプラグインは、おそらく抽象サービスが無効になっていることを要求する可能性があります。これにより、同じ名前のクラスを生成できます。プラグインは 2.3.0 版 (2010 年 1 月) で新しくなりました。

このセクションの残りの部分では、抽象サービスが有効になっている場合にプロトコルバッファコンパイラが生成する内容について説明します。

### インターフェース {#interface}

次のサービス定義が与えられた場合:

```proto
service Foo {
  rpc Bar(FooRequest) returns(FooResponse);
}
```

プロトコルバッファコンパイラは、このサービスを表すクラス `Foo` を生成します。`Foo` には、サービス定義で定義された各メソッドに対応するメソッドがあります。この場合、メソッド `Bar` は次のように定義されています:

```python
def Bar(self, rpc_controller, request, done)
```

パラメータは、[`Service.CallMethod()`](https://googleapis.dev/python/protobuf/latest/google/protobuf/service.html#google.protobuf.service.Service.CallMethod) のパラメータと同等ですが、`method_descriptor` 引数は暗黙的に含まれています。

これらの生成されたメソッドは、サブクラスでオーバーライドすることを意図しています。デフォルトの実装は単に、未実装であることを示すエラーメッセージを含む[`controller.SetFailed()`](https://googleapis.dev/python/protobuf/latest/google/protobuf/service.html#google.protobuf.service.RpcController.SetFailed)を呼び出し、その後`done`コールバックを実行します。独自のサービスを実装する際には、この生成されたサービスをサブクラス化し、適切にメソッドを実装する必要があります。

`Foo`は`Service`インターフェースをサブクラス化します。プロトコルバッファコンパイラは、`Service`のメソッドの実装を自動的に生成します。

-   `GetDescriptor`: サービスの[`ServiceDescriptor`](https://googleapis.dev/python/protobuf/latest/google/protobuf/descriptor.html#google.protobuf.descriptor.ServiceDescriptor)を返します。
-   `CallMethod`: 提供されたメソッド記述子に基づいて呼び出されているメソッドを決定し、直接呼び出します。
-   `GetRequestClass`および`GetResponseClass`: 指定されたメソッドの正しいタイプのリクエストまたはレスポンスのクラスを返します。

### Stub {#stub}

プロトコルバッファコンパイラは、サービスインターフェースごとに「スタブ」実装も生成します。これは、サービスを実装するサーバーにリクエストを送信したいクライアントによって使用されます。`Foo`サービス（上記）の場合、スタブ実装`Foo_Stub`が定義されます。

`Foo_Stub`は`Foo`のサブクラスです。そのコンストラクタは[`RpcChannel`](https://googleapis.dev/python/protobuf/latest/google/protobuf/service.html#google.protobuf.service.RpcChannel)をパラメータとして受け取ります。その後、スタブは、各サービスのメソッドを、チャンネルの`CallMethod()`メソッドを呼び出すことで実装します。

Protocol BufferライブラリにはRPCの実装は含まれていません。ただし、任意のRPC実装に生成されたサービスクラスを接続するために必要なツールはすべて含まれています。`RpcChannel`および[`RpcController`](https://googleapis.dev/python/protobuf/latest/google/protobuf/service.html#google.protobuf.service.RpcController)の実装を提供するだけです。

## プラグイン挿入ポイント {#plugins}

[コードジェネレータプラグイン](/reference/cpp/api-docs/google.protobuf.compiler.plugin)
は、Pythonコードジェネレータの出力を拡張したい場合、次の種類のコードを挿入ポイント名を使用して挿入できます。

-   `imports`: インポート文。
-   `module_scope`: トップレベルの宣言。

{{% alert title="警告" color="warning" %}} 次のようなプライベートクラスメンバに依存するコードを生成しないでください。これらの実装の詳細は、Protocol Buffersの将来のバージョンで変更される可能性があります。 {{% /alert %}}

## PythonとC++間でメッセージを共有する {#sharing-messages}

Protobuf Python APIの4.21.0バージョンより前では、Pythonアプリケーションはネイティブ拡張を使用してC++とメッセージを共有できました。4.21.0 APIバージョンからは、PythonとC++間でメッセージを共有することはデフォルトのインストールではサポートされていません。Protobuf Python APIの4.x以降のバージョンで作業する際に、環境変数`PROTOCOL_BUFFERS_PYTHON_IMPLEMENTATION=cpp`を定義し、Python/C++拡張がインストールされていることを確認して、この機能を有効にしてください。
