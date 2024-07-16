+++
title = "PHP 生成コードガイド"
weight = 730
linkTitle = "生成コードガイド"
description = "任意のプロトコル定義に対してプロトコルバッファコンパイラが生成する PHP コードを説明します。"
type = "docs"
+++

このドキュメントを読む前に、[proto3 言語ガイド](/programming-guides/proto3)を読むことをお勧めします。プロトコルバッファコンパイラは現在、PHP に対して proto3 コード生成のみをサポートしています。

## コンパイラの呼び出し {#invocation}

プロトコルバッファコンパイラは、`--php_out=` コマンドラインフラグを使用して呼び出されると、PHP 出力を生成します。`--php_out=` オプションのパラメータは、コンパイラが PHP 出力を書き込むディレクトリです。PSR-4 に準拠するため、コンパイラは proto ファイルで定義されたパッケージに対応するサブディレクトリを作成します。さらに、proto ファイルの各メッセージについて、コンパイラはパッケージのサブディレクトリに個別のファイルを作成します。メッセージの出力ファイル名は次の 3 つの部分で構成されます:

-   ベースディレクトリ: proto パス（`--proto_path=` または `-I` コマンドラインフラグで指定）は出力パス（`--php_out=` フラグで指定）に置き換えられます。
-   サブディレクトリ: パッケージ名内の `.` はオペレーティングシステムのディレクトリセパレータに置き換えられます。各パッケージ名のコンポーネントは大文字になります。
-   ファイル: メッセージ名に `.php` が追加されます。

例えば、次のようにコンパイラを呼び出すとします:

```shell
protoc --proto_path=src --php_out=build/gen src/example.proto
```

そして、`src/example.proto` が以下のように定義されているとします:

```proto
package foo.bar;
message MyMessage {}
```

コンパイラはファイル `src/foo.proto` を読み込み、出力ファイル `build/gen/Foo/Bar/MyMessage.php` を生成します。コンパイラは必要に応じてディレクトリ `build/gen/Foo/Bar` を自動的に作成しますが、`build` または `build/gen` は作成しません。それらはすでに存在している必要があります。

## パッケージ {#package}

`.proto` ファイルで定義されたパッケージ名は、生成された PHP クラスのモジュール構造をデフォルトで生成するために使用されます。次のようなファイルがあるとします:

```proto
package foo.bar;

message MyMessage {}
```

プロトコルコンパイラは、`Foo\Bar\MyMessage` という名前の出力クラスを生成します。

### 名前空間オプション {#namespace-options}

コンパイラは、PHP およびメタデータの名前空間を定義するための追加オプションをサポートしています。これらが定義されている場合、これらはモジュール構造と名前空間の生成に使用されます。次のようなオプションが与えられた場合:

```proto
package foo.bar;
option php_namespace = "baz\\qux";
option php_metadata_namespace = "Foo";
message MyMessage {}
```

プロトコルコンパイラは、`baz\qux\MyMessage` という名前の出力クラスを生成します。このクラスは名前空間 `namespace baz\qux` を持ちます。

プロトコルコンパイラは、`Foo\Metadata` という名前のメタデータクラスを生成します。このクラスは名前空間 `namespace Foo` を持ちます。

*生成されるオプションは大文字と小文字を区別します。デフォルトでは、パッケージはパスカルケースに変換されます。*

## メッセージ {#message}

単純なメッセージ宣言が与えられた場合:

```proto
message Foo {}
```

プロトコルバッファコンパイラは、`Foo` という名前の PHP クラスを生成します。このクラスは、一般的な基本クラス `Google\Protobuf\Internal\Message` を継承しており、メッセージタイプのエンコードとデコードのためのメソッドを提供します。次の例に示すように:

```php
$from = new Foo();
$from->setInt32(1);
$from->setString('a');
$from->getRepeatedInt32()[] = 1;
$from->getMapInt32Int32()[1] = 1;
$data = $from->serializeToString();
try {
  $to->mergeFromString($data);
} catch (Exception $e) {
  // Handle parsing error from invalid data.
  ...
}
```

あなたは *自分自身で* `Foo` のサブクラスを作成すべきではありません。生成されたクラスはサブクラス化を意図しておらず、\"fragile base class\" の問題を引き起こす可能性があります。

ネストされたメッセージは、PHP ではネストされたクラスをサポートしていないため、それらを含むメッセージにプレフィックスを付けた同じ名前の PHP クラスになります。したがって、`.proto` に次のようなものがある場合:

```proto
message TestMessage {
  optional int32 a = 1;
  message NestedMessage {...}
}
```

コンパイラは次のクラスを生成します:

```php
class TestMessage {
  public a;
}

// PHP doesn’t support nested classes.
class TestMessage_NestedMessage {...}
```

メッセージクラス名が予約されている場合 (例: `Empty` のような場合)、クラス名の接頭辞として `PB` が付加されます:

```php
class PBEmpty {...}
```

また、ファイルレベルのオプション `php_class_prefix` も提供されています。これが指定されている場合、すべての生成されたメッセージクラスの前にそれが付加されます。

## フィールド

メッセージタイプの各フィールドには、フィールドを設定および取得するためのアクセサメソッドがあります。したがって、フィールド `x` が与えられた場合、次のように書くことができます:

```php
$m = new MyMessage();
$m->setX(1);
$val = $m->getX();

$a = 1;
$m->setX($a);
```

常にフィールドを設定すると、そのフィールドの宣言された型に対して値が型チェックされます。値が間違った型である場合（または範囲外の場合）、例外が発生します。デフォルトでは、整数、浮動小数点数、および数値文字列への変換（たとえば、フィールドに値を割り当てたり、繰り返しフィールドに要素を追加したりする場合）が許可されています。許可されていない変換には、配列またはオブジェクトへのすべての変換が含まれます。浮動小数点数から整数へのオーバーフロー変換は未定義です。

各スカラープロトコルバッファタイプに対する対応するPHPタイプを次の[スカラー値タイプテーブル](/programming-guides/proto3#scalar)で確認できます。

### 単数メッセージフィールド {#embedded_message}

メッセージタイプのフィールドはデフォルトでnilに設定され、フィールドにアクセスしたときに自動的に作成されません。したがって、次のように明示的にサブメッセージを作成する必要があります。

```php
$m = new MyMessage();
$m->setZ(new SubMessage());
$m->getZ()->setFoo(42);

$m2 = new MyMessage();
$m2->getZ()->setFoo(42);  // FAILS with an exception
```

インスタンスが他の場所にも保持されている場合でも、メッセージフィールドに任意のインスタンスを割り当てることができます（たとえば、別のメッセージのフィールド値として）。

### 繰り返しフィールド

プロトコルバッファコンパイラは、各繰り返しフィールドに対して特別な`RepeatedField`を生成します。したがって、次のフィールドが与えられた場合：

```proto
repeated int32 foo = 1;
```

生成されたコードでは、次のようにできます：

```php
$m->getFoo()[] =1;
$m->setFoo($array);
```

### マップフィールド

プロトコルバッファコンパイラは、各マップフィールドに対して`MapField`を生成します。したがって、次のフィールドが与えられた場合：

```proto
map<int32, int32> weight = 1;
```

生成されたコードでは、次のようにできます：

```php
$m->getWeight()[1] = 1;
```

## 列挙型 {#enum}

PHPにはネイティブの列挙型がないため、代わりにプロトコルバッファコンパイラは、`.proto`ファイル内の各列挙型に対してPHPクラスを生成します。[メッセージ](#message)と同様に、各値に対して定数が定義されます。したがって、次の列挙型が与えられた場合：

```proto
enum TestEnum {
  Default = 0;
  A = 1;
}
```

コンパイラは次のクラスを生成します：

```php
class TestEnum {
  const DEFAULT = 0;
  const A = 1;
}
```

メッセージと同様に、ネストされた列挙型は、PHPがネストされたクラスをサポートしていないため、その名前に含まれるメッセージとアンダースコアで区切られた同じ名前のPHPクラスになります。

```php
class TestMessage_NestedEnum {...}
```

列挙型のクラスや値の名前が予約語の場合（たとえば、`Empty`）、クラスや値の名前の前に`PB`が付加されます：

```php
class PBEmpty {
  const PBECHO = 0;
}
```

また、ファイルレベルのオプション`php_class_prefix`も提供されています。これが指定されている場合、生成されたすべての列挙型クラスの前にそれが付加されます。

## Oneof

[oneof](/programming-guides/proto3#oneof)に対して、
プロトコルバッファコンパイラは通常の単数フィールドのコードと同じコードを生成しますが、
設定されているoneofフィールドを特定できる特別なアクセサメソッドも追加されます。
したがって、次のメッセージが与えられた場合：

```proto
message TestMessage {
  oneof test_oneof {
    int32 oneof_int32 = 1;
    int64 oneof_int64 = 2;
  }
}
```

コンパイラは次のフィールドと特別なメソッドを生成します：

```php
class TestMessage {
  private oneof_int32;
  private oneof_int64;
  public function getOneofInt32();
  public function setOneofInt32($var);
  public function getOneofInt64();
  public function setOneofInt64($var);
  public function getTestOneof();  // Return field name
}
```

アクセサメソッドの名前はoneofの名前に基づいており、現在設定されているoneof内のフィールドを表す列挙値を返します。
```
