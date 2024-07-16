+++
title = "C# 生成コードガイド"
weight = 550
linkTitle = "生成コードガイド"
description = "proto3 構文を使用してプロトコル定義を行った場合に、プロトコルバッファコンパイラが生成する C# コードについて正確に説明します。"
type = "docs"
+++

このドキュメントを読む前に、[proto3 言語ガイド](/programming-guides/proto3)を読むことをお勧めします。

{{% alert title="Note" color="note" %}}
プロトコルバッファコンパイラは、リリース 3.10 から `proto2` 構文を使用した定義のための C\# インターフェースを生成できます。`proto2` 定義のセマンティクスの詳細については、[proto2 言語ガイド](/programming-guides/proto2)を参照し、`docs/csharp/proto2.md` ([GitHub で表示](https://github.com/protocolbuffers/protobuf/blob/master/docs/csharp/proto2.md)) を参照してください。
{{% /alert %}}

## コンパイラの呼び出し {#invocation}

プロトコルバッファコンパイラは、`--csharp_out` コマンドラインフラグを使用して C\# 出力を生成します。`--csharp_out` オプションのパラメータは、コンパイラが C\# 出力を書き込むディレクトリですが、[他のオプション](#compiler_options)によっては、指定したディレクトリのサブディレクトリを作成する場合もあります。コンパイラは、入力の各 `.proto` ファイルに対して単一のソースファイルを作成します。デフォルトでは拡張子は `.cs` ですが、コンパイラオプションを介して設定可能です。

C\# コードジェネレータは `proto3` メッセージのみをサポートしています。各 `.proto` ファイルが次の宣言で始まることを確認してください:

```proto
syntax = "proto3";
```

### C\# 固有のオプション {#compiler_options}

プロトコルバッファコンパイラに対して、`--csharp_opt` コマンドラインフラグを使用してさらなる C\# オプションを指定できます。サポートされているオプションは次のとおりです:

-   **file\_extension**: 生成されたコードのファイル拡張子を設定します。デフォルトは `.cs` ですが、一般的な代替として、生成されたコードを含むことを示すために `.g.cs` を使用することもあります。

-   **base\_namespace**: このオプションが指定されている場合、生成されたソースコードのディレクトリ階層を生成されたクラスの名前空間に対応させ、オプションの値を使用して出力ディレクトリの「ベース」と考えるべき名前空間の部分を示します。たとえば、次のコマンドラインを使用すると:

```shell
protoc --proto_path=bar --csharp_out=src --csharp_opt=base_namespace=Example player.proto
```

`player.proto`に`csharp_namespace`オプションが`Example.Game`として指定されている場合、プロトコルバッファコンパイラは`src/Game/Player.cs`というファイルを生成します。このオプションは通常、Visual StudioのC#プロジェクトでの**デフォルトの名前空間**オプションに対応します。オプションが指定されているが空の値である場合、生成されたファイルで使用される完全なC#名前空間がディレクトリ階層に使用されます。オプションが全く指定されていない場合、生成されたファイルは単に`--csharp_out`で指定されたディレクトリに書き込まれ、階層は作成されません。

- **internal_access**: このオプションが指定されている場合、ジェネレータは`public`ではなく`internal`アクセス修飾子を持つ型を作成します。

- **serializable**: このオプションが指定されている場合、ジェネレータは生成されたメッセージクラスに`[Serializable]`属性を追加します。

複数のオプションは、次の例のようにカンマで区切って指定できます。

```shell
protoc --proto_path=src --csharp_out=build/gen --csharp_opt=file_extension=.g.cs,base_namespace=Example,internal_access src/foo.proto
```

## ファイル構造 {#structure}

出力ファイルの名前は、`.proto`ファイル名をパスカルケースに変換して、アンダースコアを単語の区切りとして扱います。例えば、`player_record.proto`というファイルは、`PlayerRecord.cs`という出力ファイルになります（ファイル拡張子は`--csharp_opt`を使用して指定できます、[上記の例](#compiler_options)を参照）。

各生成されたファイルは、公開メンバーの形式を取ります。（実装はここには表示されません。）

```csharp
namespace [...]
{
  public static partial class [... descriptor class name ...]
  {
    public static FileDescriptor Descriptor { get; }
  }

  [... Enums ...]
  [... Message classes ...]
}
```

`namespace`は、protoの`package`から推測され、ファイル名と同じ変換ルールを使用しています。例えば、`example.high_score`というprotoパッケージは、`Example.HighScore`という名前空間になります。特定の.protoファイルに対してデフォルトの生成された名前空間をオーバーライドするには、`csharp_namespace`[ファイルオプション](/programming-guides/proto3#options)を使用できます。

各トップレベルの列挙型やメッセージは、名前空間のメンバーとして宣言される列挙型またはクラスを生成します。さらに、ファイル記述子用に常に単一の静的部分クラスが生成されます。これはリフレクションベースの操作に使用されます。記述子クラスは、拡張子を除いたファイルと同じ名前が付けられます。ただし、同じ名前のメッセージがある場合（これは非常に一般的です）、記述子クラスはメッセージと衝突しないように、ネストされた `Proto` 名前空間に配置されます。

これらのルールの例として、Protocol Buffers の一部として提供される `timestamp.proto` ファイルを考えてみましょう。`timestamp.proto` の簡略版は次のようになります：

```proto
syntax = "proto3";
package google.protobuf;
option csharp_namespace = "Google.Protobuf.WellKnownTypes";

message Timestamp { ... }
```

生成された `Timestamp.cs` ファイルの構造は次のようになります：

```csharp
namespace Google.Protobuf.WellKnownTypes
{
  namespace Proto
  {
    public static partial class Timestamp
    {
      public static FileDescriptor Descriptor { get; }
    }
  }

  public sealed partial class Timestamp : IMessage<Timestamp>
  {
    [...]
  }
}
```

## メッセージ {#message}

単純なメッセージ宣言が与えられた場合：

```proto
message Foo {}
```

プロトコルバッファコンパイラは、`Foo` という名前のシールされた部分クラスを生成し、`IMessage<Foo>` インターフェースを実装します。以下にメンバー宣言を示します。詳細についてはインラインコメントを参照してください。

```csharp
public sealed partial class Foo : IMessage<Foo>
{
  // Static properties for parsing and reflection
  public static MessageParser<Foo> Parser { get; }
  public static MessageDescriptor Descriptor { get; }

  // Explicit implementation of IMessage.Descriptor, to avoid conflicting with
  // the static Descriptor property. Typically the static property is used when
  // referring to a type known at compile time, and the instance property is used
  // when referring to an arbitrary message, such as during JSON serialization.
  MessageDescriptor IMessage.Descriptor { get; }

  // Parameterless constructor which calls the OnConstruction partial method if provided.
  public Foo();
  // Deep-cloning constructor
  public Foo(Foo);
  // Partial method which can be implemented in manually-written code for the same class, to provide
  // a hook for code which should be run whenever an instance is constructed.
  partial void OnConstruction();

  // Implementation of IDeepCloneable<T>.Clone(); creates a deep clone of this message.
  public Foo Clone();

  // Standard equality handling; note that IMessage<T> extends IEquatable<T>
  public override bool Equals(object other);
  public bool Equals(Foo other);
  public override int GetHashCode();

  // Converts the message to a JSON representation
  public override string ToString();

  // Serializes the message to the protobuf binary format
  public void WriteTo(CodedOutputStream output);
  // Calculates the size of the message in protobuf binary format
  public int CalculateSize();

  // Merges the contents of the given message into this one. Typically
  // used by generated code and message parsers.
  public void MergeFrom(Foo other);

  // Merges the contents of the given protobuf binary format stream
  // into this message. Typically used by generated code and message parsers.
  public void MergeFrom(CodedInputStream input);
}
```

これらのメンバーは常に存在することに注意してください。`optimize_for` オプションは C\# コードジェネレータの出力に影響しません。

### ネスト型

メッセージは他のメッセージ内で宣言することができます。例えば：

```proto
message Foo {
  message Bar {
  }
}
```

この場合、またはメッセージがネストされた列挙型を含む場合、コンパイラはネストされた `Types` クラスを生成し、その後 `Types` クラス内に `Bar` クラスを生成します。したがって、完全な生成されたコードは次のようになります：

```csharp
namespace [...]
{
  public sealed partial class Foo : IMessage<Foo>
  {
    public static partial class Types
    {
      public sealed partial class Bar : IMessage<Bar> { ... }
    }
  }
}
```

中間の `Types` クラスは不便ですが、メッセージ内の対応するフィールドを持つネスト型の一般的なシナリオを扱うために必要です。そうでないと、同じ名前のプロパティと型が同じクラス内にネストされることになり、それは無効な C\# になります。

## フィールド

プロトコルバッファコンパイラは、メッセージ内で定義された各フィールドに対して C\# プロパティを生成します。プロパティの具体的な性質は、フィールドの性質に依存します：その型、単数、繰り返し、またはマップフィールドであるかどうか。

### 単数のフィールド {#singular}

任意の単数のフィールドは読み書き可能なプロパティを生成します。`string` または `bytes` フィールドは、null 値が指定された場合に `ArgumentNullException` を生成します。明示的に設定されていないフィールドから値を取得すると、空の文字列または `ByteString` が返されます。メッセージフィールドは null 値に設定でき、これによりフィールドがクリアされます。これは、メッセージタイプの「空の」インスタンスに値を設定するのとは等しくありません。

### 繰り返しフィールド {#repeated}

各繰り返しフィールドは、`Google.Protobuf.Collections.RepeatedField<T>` 型の読み取り専用プロパティを生成します。ここで、`T` はフィールドの要素の型です。ほとんどの場合、これは `List<T>` のように機能しますが、一度に複数のアイテムを追加するための `Add` オーバーロードが追加されています。これは、オブジェクト初期化子で繰り返しフィールドを設定する際に便利です。さらに、`RepeatedField<T>` は直列化、逆直列化、クローン作成を直接サポートしていますが、通常は生成されたコードによって使用され、手動で記述されたアプリケーションコードでは使用されません。

繰り返しフィールドには、メッセージ型を含む null 値を含めることはできません。ただし、[以下で説明されている](#wrapper_types) nullable ラッパータイプを除いて、null 値を含めることはできます。

### マップフィールド {#map}

各マップフィールドは、`Google.Protobuf.Collections.MapField<TKey, TValue>` 型の読み取り専用プロパティを生成します。ここで、`TKey` はフィールドのキーの型であり、`TValue` はフィールドの値の型です。ほとんどの場合、これは `Dictionary<TKey, TValue>` のように機能しますが、一度に別の辞書を追加するための `Add` オーバーロードが追加されています。これは、オブジェクト初期化子で繰り返しフィールドを設定する際に便利です。さらに、`MapField<TKey, TValue>` は直列化、逆直列化、クローン作成を直接サポートしていますが、通常は生成されたコードによって使用され、手動で記述されたアプリケーションコードでは使用されません。マップ内のキーは null であってはならず、値は、対応する単数のフィールド型が null 値をサポートする場合には null であっても構いません。

### Oneof フィールド {#oneof}

Oneof 内の各フィールドには、通常の[単数のフィールド](#singular)と同様に、個別のプロパティがあります。ただし、コンパイラは、列挙型で設定されたフィールドを判断するための追加のプロパティ、列挙型、および Oneof をクリアするためのメソッドも生成します。たとえば、この Oneof フィールドの定義では

```proto
oneof avatar {
  string image_url = 1;
  bytes image_data = 2;
}
```

コンパイラはこれらのパブリックメンバを生成します:

```csharp
enum AvatarOneofCase
{
  None = 0,
  ImageUrl = 1,
  ImageData = 2
}

public AvatarOneofCase AvatarCase { get; }
public void ClearAvatar();
public string ImageUrl { get; set; }
public ByteString ImageData { get; set; }
```

プロパティが現在のoneof "case"である場合、そのプロパティを取得すると、そのプロパティに設定された値が返されます。それ以外の場合、プロパティを取得すると、プロパティの型のデフォルト値が返されます。oneofのメンバーは一度に1つだけ設定できます。

oneofの構成プロパティを設定すると、oneofの報告される"case"が変更されます。通常の[単数フィールド](#singular)と同様に、`string`または`bytes`型のoneofフィールドにnull値を設定することはできません。メッセージ型のフィールドをnullに設定することは、oneof固有の`Clear`メソッドを呼び出すことと同等です。

### ラッパータイプフィールド {#wrapper_types}

proto3のほとんどのwell-known typesはコード生成に影響を与えませんが、ラッパータイプ（`StringWrapper`、`Int32Wrapper`など）はプロパティの型と動作を変更します。

C\#の値型に対応するすべてのラッパータイプ（`Int32Wrapper`、`DoubleWrapper`、`BoolWrapper`など）は、対応する非null許容型である`Nullable<T>`にマップされます。たとえば、`DoubleValue`型のフィールドは、`Nullable<double>`型のC\#プロパティになります。

`StringWrapper`型または`BytesWrapper`型のフィールドは、`string`型と`ByteString`型のC\#プロパティが生成されますが、デフォルト値はnullであり、nullをプロパティ値として設定することができます。

すべてのラッパータイプにおいて、null値は繰り返しフィールドでは許可されませんが、マップエントリの値として許可されます。

## 列挙型 {#enum}

次のような列挙型の定義がある場合:

```proto
enum Color {
  COLOR_UNSPECIFIED = 0;
  COLOR_RED = 1;
  COLOR_GREEN = 5;
  COLOR_BLUE = 1234;
}
```

プロトコルバッファコンパイラは、同じ値のセットを持つ`Color`というC\#列挙型を生成します。列挙値の名前は、C\#開発者向けにより習慣的な形に変換されます:

-   もとの名前が列挙型名の大文字形式で始まる場合、それが削除されます
-   結果がパスカルケースに変換されます

したがって、上記の`Color` proto列挙型は、次のC\#コードになります:

```csharp
enum Color
{
  Unspecified = 0,
  Red = 1,
  Green = 5,
  Blue = 1234
}
```

この名前変換は、JSON表現内で使用されるテキストに影響を与えません。

`.proto`言語では、複数の列挙シンボルが同じ数値を持つことができます。同じ数値を持つシンボルは同義語です。これらはC\#ではまったく同じ方法で表現され、同じ数値に対応する複数の名前があります。

ネストされていない列挙型は、新しい名前空間メンバーとして生成されるC\#の列挙型を生成します。ネストされた列挙型は、列挙型がネストされているメッセージに対応するクラス内の`Types`ネストされたクラスに生成されます。

## サービス {#service}

C\#コードジェネレータはサービスを完全に無視します。
