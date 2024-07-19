+++
title = "Kotlin 生成コードガイド"
weight = 680
linkTitle = "生成コードガイド"
description = "任意のプロトコル定義に対してプロトコルバッファコンパイラが生成する Kotlin コードを詳細に説明し、Java に生成されるコードに加えています。"
type = "docs"
+++

proto2 と proto3 で生成されるコードの違いについては、このドキュメントで説明されている生成されたコードの違いが強調されています。これは、ベースのメッセージクラス/インターフェースではなく、生成されたコードにある違いです。このドキュメントを読む前に、[proto2 言語ガイド](/programming-guides/proto2) および/または [proto3 言語ガイド](/programming-guides/proto3) をお読みいただく必要があります。

## コンパイラの呼び出し {#invocation}

プロトコルバッファコンパイラは、Java コードをベースに構築された Kotlin コードを生成します。そのため、`--java_out=` と `--kotlin_out=` の 2 つのコマンドラインフラグを使用して呼び出す必要があります。`--java_out=` オプションのパラメータは、コンパイラが Java 出力を書き込むディレクトリであり、`--kotlin_out=` も同様です。各 `.proto` ファイルの入力に対して、コンパイラは `.proto` ファイル自体を表す Java クラスを含むラッパー `.java` ファイルを作成します。

次のような行を含むかどうかに関係なく、`.proto` ファイルに次のような行が含まれているかどうかに関係なく:

```proto
option java_multiple_files = true;
```

コンパイラは、`.proto` ファイルで宣言された各トップレベルメッセージに対して生成される各クラスおよびファクトリメソッドのために、それぞれ別々の `.kt` ファイルを作成します。

各ファイルの Java パッケージ名は、[Java 生成コードリファレンス](/reference/java/java-generated#package) で説明されている生成された Java コードで使用されるものと同じです。

出力ファイルは、`--kotlin_out=` のパラメータ、パッケージ名（ピリオド [.] をスラッシュ [/] に置き換えたもの）、および接尾辞 `Kt.kt` ファイル名を連結して選択されます。

たとえば、次のようにコンパイラを呼び出すとします:

```shell
protoc --proto_path=src --java_out=build/gen/java --kotlin_out=build/gen/kotlin src/foo.proto
```

`foo.proto` の Java パッケージが `com.example` であり、`Bar` というメッセージが含まれている場合、プロトコルバッファコンパイラはファイル `build/gen/kotlin/com/example/BarKt.kt` を生成します。プロトコルバッファコンパイラは、必要に応じて `build/gen/kotlin/com` および `build/gen/kotlin/com/example` ディレクトリを自動的に作成します。ただし、`build/gen/kotlin`、`build/gen`、または `build` は作成されません。これらはすでに存在している必要があります。1 回の呼び出しで複数の `.proto` ファイルを指定できます。すべての出力ファイルが一度に生成されます。

## メッセージ {#message}

単純なメッセージ宣言が与えられた場合：

```proto
message FooBar {}
```

プロトコルバッファコンパイラは、生成されたJavaコードに加えて、`FooBarKt`というオブジェクトを生成し、次の構造を持つ2つのトップレベル関数を生成します：

```kotlin
object FooBarKt {
  class Dsl private constructor { ... }
}
inline fun fooBar(block: FooBarKt.Dsl.() -> Unit): FooBar
inline fun FooBar.copy(block: FooBarKt.Dsl.() -> Unit): FooBar
```

### ネストされたタイプ

メッセージは他のメッセージの内部で宣言することができます。例えば：

```proto
message Foo {
  message Bar { }
}
```

この場合、コンパイラは`FooKt`の内部に`BarKt`オブジェクトと`bar`ファクトリメソッドをネストしますが、`copy`メソッドはトップレベルのままです：

```kotlin
object FooKt {
  class Dsl { ... }
  object BarKt {
    class Dsl private constructor { ... }
  }
  inline fun bar(block: FooKt.BarKt.Dsl.() -> Unit): Foo.Bar
}
inline fun foo(block: FooKt.Dsl.() -> Unit): Foo
inline fun Foo.copy(block: FooKt.Dsl.() -> Unit): Foo
inline fun Foo.Bar.copy(block: FooKt.BarKt.Dsl.() -> Unit): Foo.Bar
```

## フィールド

前のセクションで説明したメソッドに加えて、プロトコルバッファコンパイラは、`.proto`ファイル内で定義された各フィールドに対してDSL内の可変プロパティを生成します（Kotlinは既にJavaによって生成されたゲッターからメッセージオブジェクトの読み取り専用プロパティを推論します）。

プロパティは常にキャメルケースの命名規則を使用しますが、`.proto`ファイル内のフィールド名がアンダースコアを使用している場合でも（[使用すべきです](/programming-guides/style)）、ケース変換は次のように機能します：

1. 名前内の各アンダースコアは削除され、次の文字が大文字になります。
2. 名前に接頭辞が付加される場合（例： "clear"）、最初の文字が大文字になります。それ以外の場合は小文字になります。

したがって、フィールド`foo_bar_baz`は`fooBarBaz`になります。

Kotlinの予約語やプロトコルバッファライブラリで既に定義されているメソッドとフィールド名が競合する特別な場合には、追加のアンダースコアが付加されます。例えば、`in`という名前のフィールドのクリアメソッドは`clearIn_()`です。

### 単数フィールド（proto2）

次のいずれかのフィールド定義がある場合：

```proto
optional int32 foo = 1;
required int32 foo = 1;
```

コンパイラはDSL内で次のアクセサを生成します：

- `fun hasFoo(): Boolean`：フィールドが設定されている場合は`true`を返します。
- `var foo: Int`：フィールドの現在の値。フィールドが設定されていない場合はデフォルト値を返します。
- `fun clearFoo()`：フィールドの値をクリアします。これを呼び出した後、`hasFoo()`は`false`を返し、`getFoo()`はデフォルト値を返します。

他の単純なフィールドタイプについては、[スカラー値タイプテーブル](/programming-guides/proto2#scalar)に従って対応するJavaタイプが選択されます。メッセージおよび列挙型の場合、値のタイプはメッセージまたは列挙型クラスに置き換えられます。メッセージタイプがまだJavaで定義されているため、メッセージ内の符号なしタイプは、Javaおよび古いバージョンのKotlinとの互換性のためにDSL内で標準の対応する符号付きタイプを使用して表されます。

#### 埋め込みメッセージフィールド

サブメッセージの特別な処理はありません。たとえば、次のようなフィールドがある場合、

```proto
optional Foo my_foo = 1;
```

次のように記述する必要があります

```kotlin
myFoo = foo {
  ...
}
```

一般的に、これはコンパイラが`Foo`がKotlin DSLを持っているかどうか、または例えばJava APIのみが生成されているかどうかを知らないためです。これは、依存しているメッセージにKotlinコード生成を追加するのを待つ必要がないことを意味します。

### 単数フィールド（proto3）

このフィールド定義に対して：

```proto
int32 foo = 1;
```

コンパイラはDSL内で次のプロパティを生成します：

-   `var foo: Int`：フィールドの現在の値を返します。フィールドが設定されていない場合、フィールドのタイプのデフォルト値を返します。
-   `fun clearFoo()`：フィールドの値をクリアします。これを呼び出した後、`getFoo()`はフィールドのタイプのデフォルト値を返します。

他の単純なフィールドタイプについては、[スカラー値タイプテーブル](/programming-guides/proto2#scalar)に従って対応するJavaタイプが選択されます。メッセージおよび列挙型の場合、値のタイプはメッセージまたは列挙型クラスに置き換えられます。メッセージタイプがまだJavaで定義されているため、メッセージ内の符号なしタイプは、Javaおよび古いバージョンのKotlinとの互換性のためにDSL内で標準の対応する符号付きタイプを使用して表されます。

#### 埋め込みメッセージフィールド

メッセージフィールドタイプの場合、DSL内に追加のアクセサメソッドが生成されます：

-   `boolean hasFoo()`：フィールドが設定されている場合は`true`を返します。

サブメッセージをDSLに基づいて設定するためのショートカットはありません。たとえば、次のようなフィールドがある場合、

```proto
Foo my_foo = 1;
```

```kotlin
myFoo = foo {
  ...
}
```

一般的に、これはコンパイラが `Foo` が Kotlin DSL を持っているかどうかを知らないためです。例えば、Java の API のみが生成されているかもしれません。これは、依存しているメッセージが Kotlin コード生成を追加するのを待つ必要がないことを意味します。

### 繰り返しフィールド {#repeated}

このフィールド定義に対して:

```proto
repeated string foo = 1;
```

コンパイラは、DSL に次のメンバーを生成します:

-   `class FooProxy: DslProxy`、ジェネリックでのみ使用される構築不可能な型
-   `val fooList: DslList<String, FooProxy>`、繰り返しフィールド内の現在の要素の読み取り専用ビュー
-   `fun DslList<String, FooProxy>.add(value: String)`、要素を繰り返しフィールドに追加する拡張関数
-   `operator fun DslList<String, FooProxy>.plusAssign(value: String)`、`add` のエイリアス
-   `fun DslList<String, FooProxy>.addAll(values: Iterable<String>)`、`Iterable` の要素を繰り返しフィールドに追加する拡張関数
-   `operator fun DslList<String, FooProxy>.plusAssign(values: Iterable<String>)`、`addAll` のエイリアス
-   `operator fun DslList<String, FooProxy>.set(index: Int, value: String)`、指定されたゼロベースのインデックスの要素の値を設定する拡張関数
-   `fun DslList<String, FooProxy>.clear()`、繰り返しフィールドの内容をクリアする拡張関数

この異例の構築により、`fooList` は DSL のスコープ内で変更可能なリストのように振る舞い、基礎となるビルダーがサポートするメソッドのみをサポートし、DSL からの変更可能性が「逃げ出す」のを防ぎ、混乱を招く副作用を防止します。

他の単純なフィールドタイプについては、対応する Java 型は、[スカラー値の種類の表](/programming-guides/proto2#scalar)に従って選択されます。メッセージおよび列挙型の場合、型はメッセージまたは列挙型のクラスです。

### Oneof フィールド

この oneof フィールド定義に対して:

```proto
oneof oneof_name {
    int32 foo = 1;
    ...
}
```

コンパイラは、DSL に次のアクセサメソッドを生成します:

-   `val oneofNameCase: OneofNameCase`: どの `oneof_name` フィールドが設定されているかを取得します。戻り値については[Javaコードリファレンス](/reference/java/java-generated#oneof)を参照してください
-   `fun hasFoo(): Boolean` (proto2のみ): oneofケースが `FOO` の場合は `true` を返します
-   `val foo: Int`: oneofケースが `FOO` の場合は `oneof_name` の現在の値を返します。それ以外の場合はこのフィールドのデフォルト値を返します

他の単純なフィールドタイプについては、対応するJavaタイプは[スカラー値タイプテーブル](/programming-guides/proto2#scalar)に従って選択されます。メッセージおよび列挙型の場合、値のタイプはメッセージまたは列挙型クラスに置き換えられます。

### マップフィールド

このマップフィールドの定義に対して:

```proto
map<int32, int32> weight = 1;
```

コンパイラはDSLクラス内に次のメンバーを生成します:

-   `class WeightProxy private constructor(): DslProxy()`: ジェネリックスでのみ使用される構築不可能なタイプ
-   `val weight: DslMap<Int, Int, WeightProxy>`: マップフィールド内の現在のエントリの読み取り専用ビュー
-   `fun DslMap<Int, Int, WeightProxy>.put(key: Int, value: Int)`: このマップフィールドにエントリを追加します
-   `operator fun DslMap<Int, Int, WeightProxy>.put(key: Int, value: Int)`: 演算子構文を使用した `put` のエイリアス
-   `fun DslMap<Int, Int, WeightProxy>.remove(key: Int)`: `key` に関連付けられたエントリを削除します
-   `fun DslMap<Int, Int, WeightProxy>.putAll(map: Map<Int, Int)`: 指定されたマップからすべてのエントリをこのマップフィールドに追加し、既存のキーに対して以前の値を上書きします
-   `fun DslMap<Int, Int, WeightProxy>.clear()`: このマップフィールドからすべてのエントリをクリアします

## 拡張機能 (proto2のみ) {#extension}

拡張範囲を持つメッセージがある場合:

```proto
message Foo {
  extensions 100 to 199;
}
```

プロトコルバッファコンパイラは `FooKt.Dsl` に次のメソッドを追加します:

-   `operator fun <T> get(extension: ExtensionLite<Foo, T>): T`: DSL内の拡張フィールドの現在の値を取得します
-   `operator fun <T> get(extension: ExtensionLite<Foo, List<T>>): ExtensionList<T, Foo>`: DSL内の繰り返し拡張フィールドの現在の値を読み取り専用の `List` として取得します
-   `operator fun <T : Comparable<T>> set(extension: ExtensionLite<Foo, T>)`: DSL内の拡張フィールドの現在の値を設定します (`Comparable` フィールドタイプ用)
-   `operator fun <T : MessageLite> set(extension: ExtensionLite<Foo, T>)`: DSL内の拡張フィールドの現在の値を設定します (メッセージフィールドタイプ用)
-   `operator fun set(extension: ExtensionLite<Foo, ByteString>)`: DSL内の拡張フィールドの現在の値を設定します (`bytes` フィールド用)
-   `operator fun contains(extension: ExtensionLite<Foo, *>): Boolean`: 拡張フィールドに値がある場合は `true` を返します
-   `fun clear(extension: ExtensionLite<Foo, *>)`: 拡張フィールドをクリアします
-   `fun <E> ExtensionList<Foo, E>.add(value: E)`: 繰り返し拡張フィールドに値を追加します
-   `operator fun <E> ExtensionList<Foo, E>.plusAssign(value: E)`: 演算子構文を使用した `add` のエイリアス
-   `operator fun <E> ExtensionList<Foo, E>.addAll(values: Iterable<E>)`: 複数の値を繰り返し拡張フィールドに追加します
-   `operator fun <E> ExtensionList<Foo, E>.plusAssign(values: Iterable<E>)`: 演算子構文を使用した `addAll` のエイリアス
-   `operator fun <E> ExtensionList<Foo, E>.set(index: Int, value: E)`: 指定されたインデックスの繰り返し拡張フィールドの要素を設定します
-   `inline fun ExtensionList<Foo, *>.clear()`: 繰り返し拡張フィールドの要素をクリアします

ジェネリックスはここでは複雑ですが、効果は `this[extension] = value` が繰り返し拡張を除くすべての拡張タイプで機能するということです。そして、繰り返し拡張は [非拡張繰り返しフィールド](#repeated) と同様に機能する「自然な」リスト構文を持っています。

拡張の定義が与えられた場合:

```proto
extend Foo {
  optional int32 bar = 123;
}
```

Java は「拡張識別子」 `bar` を生成し、これが上記の拡張操作の「キー」として使用されます。
