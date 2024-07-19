+++
title = "Protocol Buffer Basics: C#"
weight = 220
linkTitle = "C#"
description = "プロトコルバッファを使用した基本的なC#プログラマー向けの導入。"
type = "docs"
+++

このチュートリアルでは、プロトコルバッファ言語の[proto3](/programming-guides/proto3)バージョンを使用して、プロトコルバッファを扱う基本的なC#プログラマー向けの導入を提供します。簡単な例のアプリケーションを作成することで、次の方法を示します。

- `.proto`ファイルでメッセージ形式を定義する。
- プロトコルバッファコンパイラを使用する。
- C#プロトコルバッファAPIを使用してメッセージを書き込み、読み取る。

これはC#でプロトコルバッファを使用する包括的なガイドではありません。詳細な参照情報については、[Protocol Buffer Language Guide](/programming-guides/proto3)、[C# API Reference](/reference/csharp/api-docs)、[C# Generated Code Guide](/reference/csharp/csharp-generated)、および[Encoding Reference](/programming-guides/encoding)を参照してください。

## 問題領域 {#problem-domain}

使用する例は非常にシンプルな「アドレス帳」アプリケーションで、人々の連絡先詳細をファイルに読み書きできるものです。アドレス帳の各人物には、名前、ID、メールアドレス、連絡先電話番号があります。

このような構造化データをシリアライズおよび取得する方法はどうすればよいでしょうか？この問題を解決するためのいくつかの方法があります。

- .NETバイナリシリアル化を`System.Runtime.Serialization.Formatters.Binary.BinaryFormatter`および関連クラスと共に使用する。これは変更に対して非常に脆弱であり、場合によってはデータサイズが大きくなります。また、他のプラットフォーム向けに書かれたアプリケーションとデータを共有する必要がある場合にはあまり適していません。
- データ項目を単一の文字列にエンコードするための特別な方法を考案することができます。たとえば、4つの整数を"12:3:-23:67"としてエンコードするなどです。これはシンプルで柔軟なアプローチですが、一時的なエンコーディングおよびパーシングコードの記述が必要であり、パーシングにはわずかなランタイムコストがかかります。これは非常に単純なデータをエンコードする場合に最適です。
- データをXMLにシリアライズする。このアプローチは非常に魅力的であり、XMLは（ある程度）人間が読める形式であり、多くの言語にバインディングライブラリが存在します。他のアプリケーション/プロジェクトとデータを共有したい場合には適しています。ただし、XMLはスペースを多く取り、エンコード/デコードにはアプリケーションに大きなパフォーマンスペナルティを課す可能性があります。また、XML DOMツリーをナビゲートすることは、通常のクラスのフィールドをナビゲートするよりもかなり複雑です。

プロトコルバッファは、この問題を解決する柔軟で効率的な自動化されたソリューションです。プロトコルバッファを使用すると、保存したいデータ構造の`.proto`説明を記述します。その後、プロトコルバッファコンパイラは、効率的なバイナリ形式でプロトコルバッファデータの自動エンコーディングとパースを実装するクラスを作成します。生成されたクラスは、プロトコルバッファを構成するフィールドのゲッターとセッターを提供し、プロトコルバッファを単位として読み書きの詳細を処理します。重要なのは、プロトコルバッファ形式が、コードが古い形式でエンコードされたデータをまだ読み取れるように、時間の経過とともに形式を拡張するアイデアをサポートしていることです。

## 例コードの場所 {#example-code}

当社の例は、プロトコルバッファを使用してエンコードされたアドレスブックデータファイルを管理するためのコマンドラインアプリケーションです。コマンド`AddressBook`（参照：[Program.cs](//github.com/protocolbuffers/protobuf/blob/master/csharp/src/AddressBook/Program.cs)）は、データファイルに新しいエントリを追加したり、データファイルを解析してデータをコンソールに出力したりできます。

完全な例は、GitHubリポジトリの[examplesディレクトリ](https://github.com/protocolbuffers/protobuf/tree/master/examples)および[`csharp/src/AddressBook`ディレクトリ](https://github.com/protocolbuffers/protobuf/tree/master/csharp/src/AddressBook)にあります。

## プロトコル形式の定義 {#protocol-format}

アドレスブックアプリケーションを作成するには、`.proto`ファイルから始める必要があります。`.proto`ファイルの定義はシンプルです：シリアライズしたい各データ構造に対して*メッセージ*を追加し、メッセージ内の各フィールドに名前とタイプを指定します。当社の例では、メッセージを定義する`.proto`ファイルは[`addressbook.proto`](https://github.com/protocolbuffers/protobuf/blob/master/examples/addressbook.proto)です。

`.proto`ファイルは、異なるプロジェクト間の名前の競合を防ぐのに役立つパッケージ宣言で始まります。

```proto
syntax = "proto3";
package tutorial;

import "google/protobuf/timestamp.proto";
```

C#では、`csharp_namespace`が指定されていない場合、生成されたクラスは`package`名と一致する名前空間に配置されます。当社の例では、デフォルトをオーバーライドするために`csharp_namespace`オプションが指定されているため、生成されたコードは`Google.Protobuf.Examples.AddressBook`の名前空間を使用します。

```csharp
option csharp_namespace = "Google.Protobuf.Examples.AddressBook";
```

次に、メッセージ定義があります。メッセージは、型付きフィールドのセットを含む集合体です。多くの標準的な単純データ型がフィールドタイプとして利用可能で、`bool`、`int32`、`float`、`double`、`string`などが含まれます。また、他のメッセージタイプをフィールドタイプとして使用することで、メッセージにさらなる構造を追加することもできます。

```proto
message Person {
  string name = 1;
  int32 id = 2;  // Unique ID number for this person.
  string email = 3;

  enum PhoneType {
    PHONE_TYPE_UNSPECIFIED = 0;
    PHONE_TYPE_MOBILE = 1;
    PHONE_TYPE_HOME = 2;
    PHONE_TYPE_WORK = 3;
  }

  message PhoneNumber {
    string number = 1;
    PhoneType type = 2;
  }

  repeated PhoneNumber phones = 4;

  google.protobuf.Timestamp last_updated = 5;
}

// Our address book file is just one of these.
message AddressBook {
  repeated Person people = 1;
}
```

上記の例では、`Person`メッセージには`PhoneNumber`メッセージが含まれており、`AddressBook`メッセージには`Person`メッセージが含まれています。さらに、他のメッセージの中にネストされたメッセージタイプを定義することもできます。例えば、`PhoneNumber`タイプが`Person`の内部で定義されていることがわかります。また、フィールドの中で事前に定義された値のリストから1つを持つようにしたい場合は、`enum`タイプを定義することもできます。ここでは、電話番号が`PHONE_TYPE_MOBILE`、`PHONE_TYPE_HOME`、または`PHONE_TYPE_WORK`のいずれかであることを指定したいとします。

各要素の " = 1"、" = 2" マーカーは、バイナリエンコーディングでフィールドが使用する一意の "tag" を識別します。タグ番号 1-15 は、高い番号よりも1バイト少なくエンコードする必要があるため、よく使用されるまたは繰り返し使用される要素にこれらのタグを使用することを最適化として選択できます。繰り返しフィールド内の各要素は、タグ番号を再エンコードする必要があるため、繰り返しフィールドは特にこの最適化の対象となります。

フィールドの値が設定されていない場合、[デフォルト値](/programming-guides/proto3#default) が使用されます：数値型の場合はゼロ、文字列の場合は空の文字列、boolの場合はfalseです。埋め込まれたメッセージの場合、デフォルト値は常にメッセージの "デフォルトインスタンス" または "プロトタイプ" であり、そのフィールドのいずれも設定されていません。明示的に設定されていないフィールドの値を取得するためのアクセサを呼び出すと、常にそのフィールドのデフォルト値が返されます。

フィールドが `repeated` の場合、そのフィールドは任意の回数（0を含む）繰り返すことができます。繰り返し値の順序はプロトコルバッファで保持されます。繰り返しフィールドを動的サイズの配列と考えてください。
```


`.proto` ファイルを書くための完全なガイドがここにあります -- すべての可能なフィールドタイプを含む -- [Protocol Buffer Language Guide](/programming-guides/proto3) で確認できます。
ただし、クラスの継承に類似した機能を探す必要はありません -- プロトコルバッファはそのような機能を提供していません。

## Protocol Buffers のコンパイル {#compiling-protocol-buffers}

`.proto` ファイルを持っている場合、次に行うべきことは、`AddressBook` (そして `Person` および `PhoneNumber`) メッセージを読み書きするために必要なクラスを生成することです。これを行うには、`.proto` ファイルに対してプロトコルバッファコンパイラ `protoc` を実行する必要があります。

1. コンパイラをインストールしていない場合は、[パッケージをダウンロード](/downloads) して README の手順に従ってください。

2. 今、コンパイラを実行し、ソースディレクトリ (アプリケーションのソースコードが存在する場所 -- 指定しない場合は現在のディレクトリが使用されます)、生成されたコードを配置する宛先ディレクトリ (`$SRC_DIR` と同じ場所が一般的)、および `.proto` ファイルへのパスを指定してください。この場合、次のように実行します:

    ```shell
    protoc -I=$SRC_DIR --csharp_out=$DST_DIR $SRC_DIR/addressbook.proto
    ```

    C# コードを生成するため、`--csharp_out` オプションを使用します -- 他のサポートされている言語に対しても同様のオプションが提供されています。

これにより、指定した宛先ディレクトリに `Addressbook.cs` が生成されます。このコードをコンパイルするには、`Google.Protobuf` アセンブリへの参照を持つプロジェクトが必要です。

## Addressbook クラス {#addressbook-classes}

`Addressbook.cs` を生成すると、以下の 5 つの有用な型が得られます:

- プロトコルバッファメッセージに関するメタデータを含む静的な `Addressbook` クラス。
- 読み取り専用の `People` プロパティを持つ `AddressBook` クラス。
- `Name`、`Id`、`Email`、`Phones` のプロパティを持つ `Person` クラス。
- 静的な `Person.Types` クラスにネストされた `PhoneNumber` クラス。
- `Person.Types` にもネストされた `PhoneType` 列挙型。

[C# Generated Code guide](/reference/csharp/csharp-generated) で生成される内容の詳細について詳しく読むことができますが、ほとんどの場合、これらを通常の C# 型として扱うことができます。強調すべき点は、繰り返しフィールドに対応するプロパティは読み取り専用であることです。コレクションにアイテムを追加したり削除したりすることはできますが、完全に別のコレクションで置き換えることはできません。繰り返しフィールドのコレクションタイプは常に `RepeatedField<T>` です。この型は `List<T>` と似ていますが、コレクションイニシャライザで使用するアイテムのコレクションを受け入れる `Add` オーバーロードなど、いくつかの追加の便利なメソッドがあります。

ここには、Person のインスタンスを作成する方法の例があります：

```csharp
Person john = new Person
{
    Id = 1234,
    Name = "John Doe",
    Email = "jdoe@example.com",
    Phones = { new Person.Types.PhoneNumber { Number = "555-4321", Type = Person.Types.PhoneType.Home } }
};
```

C# 6 では、`using static` を使用して `Person.Types` の冗長性を取り除くことができることに注意してください：

```csharp
// Add this to the other using directives
using static Google.Protobuf.Examples.AddressBook.Person.Types;
...
// The earlier Phones assignment can now be simplified to:
Phones = { new PhoneNumber { Number = "555-4321", Type = PhoneType.HOME } }
```

## パースとシリアライズ {#parsing-serialization}

プロトコルバッファを使用する目的は、データをシリアライズして他の場所でパースできるようにすることです。生成されたすべてのクラスには、`CodedOutputStream` がプロトコルバッファランタイムライブラリのクラスである `WriteTo(CodedOutputStream)` メソッドがあります。ただし、通常は、通常の `System.IO.Stream` に書き込むための拡張メソッドのいずれかを使用したり、メッセージをバイト配列または `ByteString` に変換したりします。これらの拡張メッセージは `Google.Protobuf.MessageExtensions` クラスにありますので、シリアライズする際には通常、`Google.Protobuf` 名前空間の `using` ディレクティブが必要です。例：

```csharp
using Google.Protobuf;
...
Person john = ...; // Code as before
using (var output = File.Create("john.dat"))
{
    john.WriteTo(output);
}
```

パースも簡単です。生成された各クラスには、その型に対して `MessageParser<T>` を返す静的な `Parser` プロパティがあります。これには、ストリーム、バイト配列、および `ByteString` をパースするためのメソッドが含まれています。したがって、作成したファイルをパースするには、次のようにします：

```csharp
Person john;
using (var input = File.OpenRead("john.dat"))
{
    john = Person.Parser.ParseFrom(input);
}
```

これらのメッセージを使用してアドレス帳を維持する完全な例プログラム（新しいエントリを追加し、既存のエントリをリストアップする）は、[Github リポジトリ](https://github.com/protocolbuffers/protobuf/tree/master/csharp/src/AddressBook) で入手できます。

## プロトコルバッファの拡張 {#extending-a-protobuf}

プロトコルバッファを使用するコードをリリースした後、プロトコルバッファの定義を「改善」したくなることは避けられません。新しいバッファが後方互換性を持ち、古いバッファが前方互換性を持つようにしたい場合（ほとんどの場合そうしたいと思うでしょう）、従う必要があるいくつかのルールがあります。プロトコルバッファの新しいバージョンでは：

- 既存のフィールドのタグ番号を変更してはいけません。
- フィールドを削除しても構いません。
- 新しいフィールドを追加しても構いませんが、新しいタグ番号を使用する必要があります（つまり、このプロトコルバッファで使用されていないタグ番号を使用する必要があります）。

(これらのルールには[例外](/programming-guides/proto3#updating)がいくつかありますが、ほとんど使用されません。)

これらのルールに従うと、古いコードは新しいメッセージを問題なく読み取り、単純に新しいフィールドを無視します。古いコードにとって、削除された単数フィールドは単にデフォルト値を持ち、削除された繰り返しフィールドは空になります。新しいコードも古いメッセージを透過的に読み取ります。

ただし、新しいフィールドは古いメッセージに存在しないため、デフォルト値を適切に処理する必要があります。型固有の[デフォルト値](/programming-guides/proto3#default)が使用されます。文字列の場合、デフォルト値は空の文字列です。ブール値の場合、デフォルト値はfalseです。数値型の場合、デフォルト値はゼロです。

## Reflection {#reflection}

メッセージ記述子（`.proto`ファイル内の情報）およびメッセージのインスタンスは、リフレクションAPIを使用してプログラムで調べることができます。これは、異なるテキスト形式やスマートな差分ツールなどの汎用コードを記述する際に役立ちます。各生成されたクラスには静的な`Descriptor`プロパティがあり、任意のインスタンスの記述子は`IMessage.Descriptor`プロパティを使用して取得できます。これらを使用する方法の簡単な例として、任意のメッセージのトップレベルフィールドを出力するための短いメソッドが以下に示されています。

```csharp
public void PrintMessage(IMessage message)
{
    var descriptor = message.Descriptor;
    foreach (var field in descriptor.Fields.InDeclarationOrder())
    {
        Console.WriteLine(
            "Field {0} ({1}): {2}",
            field.FieldNumber,
            field.Name,
            field.Accessor.GetValue(message);
    }
}
```
