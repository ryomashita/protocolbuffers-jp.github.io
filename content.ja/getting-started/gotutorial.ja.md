+++
title = "プロトコルバッファの基本: Go"
weight = 240
linkTitle = "Go"
description = "プロトコルバッファを使用するための基本的なGoプログラマー向けの紹介。"
type = "docs"
+++

このチュートリアルでは、プロトコルバッファ言語の[proto3](/programming-guides/proto3)バージョンを使用して、プロトコルバッファを扱うための基本的なGoプログラマー向けの紹介を提供します。シンプルな例のアプリケーションを作成することで、次の方法を示します。

-   `.proto`ファイルでメッセージ形式を定義する。
-   プロトコルバッファコンパイラを使用する。
-   GoプロトコルバッファAPIを使用してメッセージを書き込み、読み取る。

これはGoでプロトコルバッファを使用する包括的なガイドではありません。詳細なリファレンス情報については、[プロトコルバッファ言語ガイド](/programming-guides/proto3)、[Go APIリファレンス](https://pkg.go.dev/google.golang.org/protobuf/proto)、[Go生成コードガイド](/reference/go/go-generated)、および[エンコーディングリファレンス](/programming-guides/encoding)を参照してください。

## 問題領域 {#problem-domain}

使用する例は非常にシンプルな「アドレス帳」アプリケーションで、人々の連絡先詳細をファイルに読み書きできるものです。アドレス帳の各人物には、名前、ID、メールアドレス、連絡先電話番号があります。

このような構造化されたデータをシリアライズおよび取得する方法はどうすればよいでしょうか？この問題を解決するためのいくつかの方法があります。

-   [gobs](//golang.org/pkg/encoding/gob/)を使用してGoデータ構造をシリアライズする。これはGo固有の環境では良い解決策ですが、他のプラットフォーム向けに書かれたアプリケーションとデータを共有する必要がある場合には適していません。
-   データ項目を単一の文字列にエンコードするための特別な方法を考案することができます。例えば、4つの整数を"12:3:-23:67"としてエンコードするなどです。これはシンプルで柔軟なアプローチですが、一時的なエンコーディングおよびパーシングコードの記述が必要であり、パーシングにはわずかなランタイムコストがかかります。これは非常に単純なデータをエンコードする場合に最適です。
-   データをXMLにシリアライズする。XMLは（ある程度）人間が読める形式であり、多くの言語にバインディングライブラリが存在するため、このアプローチは非常に魅力的かもしれません。他のアプリケーション/プロジェクトとデータを共有したい場合には適しています。ただし、XMLはスペースを多く必要とし、エンコード/デコードにはアプリケーションに大きなパフォーマンスペナルティを課す可能性があります。また、XML DOMツリーをナビゲートすることは、通常のクラスの単純なフィールドをナビゲートするよりもかなり複雑です。

プロトコルバッファは、この問題を解決する柔軟で効率的な自動化されたソリューションです。プロトコルバッファを使用すると、保存したいデータ構造の`.proto`の記述を行います。その後、プロトコルバッファコンパイラが、効率的なバイナリ形式でプロトコルバッファデータの自動エンコーディングとパースを実装したクラスを作成します。生成されたクラスは、プロトコルバッファを構成するフィールドのためのゲッターとセッターを提供し、プロトコルバッファを単位として読み書きの詳細を処理します。重要なことに、プロトコルバッファ形式は、コードが古い形式でエンコードされたデータをまだ読み取れるように、時間の経過とともに形式を拡張するアイデアをサポートしています。

## 例コードの場所 {#example-code}

当社の例は、プロトコルバッファを使用してエンコードされたアドレス帳データファイルを管理するためのコマンドラインアプリケーションのセットです。`add_person_go`コマンドは、データファイルに新しいエントリを追加します。`list_people_go`コマンドは、データファイルを解析し、データをコンソールに出力します。

GitHubリポジトリの[examples directory](https://github.com/protocolbuffers/protobuf/tree/master/examples)で完全な例を見つけることができます。

## プロトコル形式の定義 {#protocol-format}

アドレス帳アプリケーションを作成するには、`.proto`ファイルから始める必要があります。`.proto`ファイルの定義はシンプルです。シリアライズしたい各データ構造に対して*メッセージ*を追加し、メッセージ内の各フィールドに名前とタイプを指定します。この例では、メッセージを定義する`.proto`ファイルは[`addressbook.proto`](https://github.com/protocolbuffers/protobuf/blob/master/examples/addressbook.proto)です。

`.proto`ファイルは、異なるプロジェクト間の名前の競合を防ぐのに役立つパッケージ宣言で始まります。

```proto
syntax = "proto3";
package tutorial;

import "google/protobuf/timestamp.proto";
```

`go_package`オプションは、このファイルの生成されたすべてのコードを含むパッケージのインポートパスを定義します。Goパッケージ名は、インポートパスの最後のパスコンポーネントになります。例えば、当社の例ではパッケージ名を"tutorialpb"とします。

```proto
option go_package = "github.com/protocolbuffers/protobuf/examples/go/tutorialpb";
```

次に、メッセージ定義があります。メッセージは、型付きフィールドのセットを含む集約体です。`bool`、`int32`、`float`、`double`、`string`など、多くの標準的な単純データ型がフィールド型として利用可能です。また、他のメッセージ型をフィールド型として使用することで、メッセージにさらなる構造を追加することもできます。

```proto
message Person {
  string name = 1;
  int32 id = 2;  // Unique ID number for this person.
  string email = 3;

  message PhoneNumber {
    string number = 1;
    PhoneType type = 2;
  }

  repeated PhoneNumber phones = 4;

  google.protobuf.Timestamp last_updated = 5;
}

enum PhoneType {
  PHONE_TYPE_UNSPECIFIED = 0;
  PHONE_TYPE_MOBILE = 1;
  PHONE_TYPE_HOME = 2;
  PHONE_TYPE_WORK = 3;
}

// Our address book file is just one of these.
message AddressBook {
  repeated Person people = 1;
}
```

上記の例では、`Person`メッセージが`PhoneNumber`メッセージを含み、`AddressBook`メッセージが`Person`メッセージを含んでいます。さらに、他のメッセージの中にネストされたメッセージ型を定義することもできます。例えば、`PhoneNumber`型が`Person`の内部で定義されていることがわかります。また、フィールドの1つに事前定義された値のリストの1つを持つようにしたい場合は、`enum`型を定義することもできます。ここでは、電話番号が`PHONE_TYPE_MOBILE`、`PHONE_TYPE_HOME`、または`PHONE_TYPE_WORK`のいずれかであることを指定したいとします。

各要素の " = 1"、" = 2" マーカーは、バイナリエンコーディングでフィールドが使用する一意の "tag" を識別します。タグ番号 1-15 は、高い番号よりも1バイト少なくエンコードする必要があるため、一般的に使用されるまたは繰り返し使用される要素にこれらのタグを使用することを最適化として選択できます。繰り返しフィールド内の各要素は、タグ番号を再エンコードする必要があるため、繰り返しフィールドは特にこの最適化の対象となります。

フィールドの値が設定されていない場合、[デフォルト値](/programming-guides/proto3#default) が使用されます。数値型の場合はゼロ、文字列の場合は空の文字列、bool型の場合はfalseです。埋め込まれたメッセージの場合、デフォルト値は常にメッセージの "デフォルトインスタンス" または "プロトタイプ" であり、そのフィールドのいずれも設定されていません。明示的に設定されていないフィールドの値を取得するためのアクセサを呼び出すと、常にそのフィールドのデフォルト値が返されます。

フィールドが `repeated` の場合、そのフィールドは任意の回数（ゼロを含む）繰り返すことができます。繰り返し値の順序はプロトコルバッファで保持されます。繰り返しフィールドは、動的サイズの配列と考えてください。
```

`.proto` ファイルを作成するための完全なガイドが見つかります。すべての可能なフィールドタイプを含む
[Protocol Buffer Language Guide](/programming-guides/proto3) には、クラスの継承に類似した機能はありません。

## Protocol Buffers のコンパイル {#compiling-protocol-buffers}

`.proto` ファイルを持っている場合、次に行う必要があるのは、`AddressBook`（および `Person`、`PhoneNumber`）メッセージを読み書きするために必要なクラスを生成することです。これを行うには、`.proto` ファイルに対して protocol buffer コンパイラ `protoc` を実行する必要があります。

1. コンパイラをインストールしていない場合は、[パッケージをダウンロード](/downloads) して README の手順に従ってください。

2. 次のコマンドを実行して Go protocol buffers プラグインをインストールします:

    ```shell
    go install google.golang.org/protobuf/cmd/protoc-gen-go@latest
    ```

    コンパイラプラグイン `protoc-gen-go` は、`$GOBIN` にインストールされます。デフォルトでは `$GOPATH/bin` になります。protocol コンパイラ `protoc` がそれを見つけるためには、`$PATH` に含まれている必要があります。

3. 今、アプリケーションのソースコードが存在するソースディレクトリ（提供しない場合は現在のディレクトリが使用されます）、生成されたコードを配置する宛先ディレクトリ（通常は `$SRC_DIR` と同じ）、および `.proto` ファイルへのパスを指定してコンパイラを実行します。この場合、次のように呼び出します:

    ```shell
    protoc -I=$SRC_DIR --go_out=$DST_DIR $SRC_DIR/addressbook.proto
    ```

    Go コードを生成するため、`--go_out` オプションを使用します。他のサポートされている言語に対しても同様のオプションが提供されています。

これにより、指定した宛先ディレクトリに
`github.com/protocolbuffers/protobuf/examples/go/tutorialpb/addressbook.pb.go`
が生成されます。

## Protocol Buffer API {#protobuf-api}

`addressbook.pb.go` を生成すると、次の便利な型が得られます:

- `People` フィールドを持つ `AddressBook` 構造体。
- `Name`、`Id`、`Email`、`Phones` のフィールドを持つ `Person` 構造体。
- `Number` と `Type` のフィールドを持つ `Person_PhoneNumber` 構造体。
- `Person.PhoneType` 列挙型の各値に対して定義された `Person_PhoneType` 型と値。

詳細については、[Go Generated Code guide](/reference/go/go-generated)を参照してくださいが、ほとんどの場合、これらを完全に通常のGoの型として扱うことができます。

以下は、Personのインスタンスを作成する方法の例です：

```go
p := pb.Person{
    Id:    1234,
    Name:  "John Doe",
    Email: "jdoe@example.com",
    Phones: []*pb.Person_PhoneNumber{
        {Number: "555-4321", Type: pb.PhoneType_PHONE_TYPE_HOME},
    },
}
```

## メッセージの書き込み {#writing-a-message}

プロトコルバッファを使用する目的は、データをシリアライズして他の場所で解析できるようにすることです。Goでは、`proto`ライブラリの[Marshal](https://pkg.go.dev/google.golang.org/protobuf/proto?tab=doc#Marshal)関数を使用して、プロトコルバッファデータをシリアライズします。プロトコルバッファメッセージの`struct`へのポインタは`proto.Message`インターフェースを実装します。`proto.Marshal`を呼び出すと、プロトコルバッファがそのワイヤ形式でエンコードされて返されます。たとえば、この関数を[`add_person`コマンド](https://github.com/protocolbuffers/protobuf/blob/master/examples/go/cmd/add_person/add_person.go)で使用します：

```go
book := &pb.AddressBook{}
// ...

// Write the new address book back to disk.
out, err := proto.Marshal(book)
if err != nil {
    log.Fatalln("Failed to encode address book:", err)
}
if err := ioutil.WriteFile(fname, out, 0644); err != nil {
    log.Fatalln("Failed to write address book:", err)
}
```

## メッセージの読み取り {#reading-a-message}

エンコードされたメッセージを解析するには、`proto`ライブラリの[Unmarshal](https://pkg.go.dev/google.golang.org/protobuf/proto?tab=doc#Unmarshal)関数を使用します。これを呼び出すと、`in`のデータがプロトコルバッファとして解析され、結果が`book`に配置されます。したがって、[`list_people`コマンド](https://github.com/protocolbuffers/protobuf/blob/master/examples/go/cmd/list_people/list_people.go)のファイルを解析するには、次のようにします：

```go
// Read the existing address book.
in, err := ioutil.ReadFile(fname)
if err != nil {
    log.Fatalln("Error reading file:", err)
}
book := &pb.AddressBook{}
if err := proto.Unmarshal(in, book); err != nil {
    log.Fatalln("Failed to parse address book:", err)
}
```

## プロトコルバッファの拡張 {#extending-a-protobuf}

プロトコルバッファを使用するコードをリリースした後、プロトコルバッファの定義を「改善」したくなることが遅かれ早かれあります。新しいバッファが後方互換性があること、古いバッファが前方互換性があることを望む場合 -- そしておそらくほとんどの場合そうしたいでしょう -- その場合には従う必要があるいくつかのルールがあります。新しいバージョンのプロトコルバッファでは：

-  既存のフィールドのタグ番号を変更してはいけません。
-  フィールドを削除しても構いません。
-  新しいフィールドを追加しても構いませんが、新しいタグ番号を使用する必要があります（つまり、このプロトコルバッファで使用されたことのないタグ番号を使用します。削除されたフィールドですら使用されていないタグ番号を使用する必要があります）。

（これらのルールには[いくつかの例外](/programming-guides/proto3#updating)がありますが、ほとんど使用されません。）

これらのルールに従うと、古いコードは新しいメッセージを問題なく読み取り、新しいフィールドを単に無視します。古いコードにとって、削除された単一フィールドは単にデフォルト値を持ち、削除された繰り返しフィールドは空になります。新しいコードも古いメッセージを透過的に読み取ります。

ただし、新しいフィールドは古いメッセージに存在しないため、デフォルト値を適切に処理する必要があります。タイプ固有の[デフォルト値](/programming-guides/proto3#default)が使用されます：文字列の場合、デフォルト値は空の文字列です。ブール値の場合、デフォルト値はfalseです。数値型の場合、デフォルト値はゼロです。
