+++
title = "Protocol Buffer Basics: Dart"
weight = 230
linkTitle = "Dart"
description = "プロトコルバッファを使用した基本的なDartプログラマーの紹介。"
type = "docs"
+++

このチュートリアルでは、プロトコルバッファ言語の[proto3](/programming-guides/proto3)バージョンを使用して、プロトコルバッファを扱う基本的なDartプログラマーの紹介を提供します。シンプルな例のアプリケーションを作成することで、次の方法を示します。

-   `.proto`ファイルでメッセージ形式を定義する。
-   プロトコルバッファコンパイラを使用する。
-   DartプロトコルバッファAPIを使用してメッセージを書き込み、読み取る。

これはDartでプロトコルバッファを使用する包括的なガイドではありません。詳細な参照情報については、[Protocol Buffer Language Guide](/programming-guides/proto3)、[Dart Language Tour](https://www.dartlang.org/guides/language/language-tour)、[Dart API Reference](https://pub.dartlang.org/documentation/protobuf)、[Dart Generated Code Guide](/reference/dart/dart-generated)、および[Encoding Reference](/programming-guides/encoding)を参照してください。

## 問題領域 {#problem-domain}

使用する例は、非常にシンプルな「アドレス帳」アプリケーションです。このアプリケーションは、人々の連絡先詳細をファイルに読み書きできます。アドレス帳の各人物には、名前、ID、メールアドレス、連絡先電話番号があります。

このような構造化データをどのようにシリアライズおよび取得しますか？この問題を解決する方法はいくつかあります。

-   データ項目を単一の文字列にエンコードするアドホックな方法を考案することができます。たとえば、4つの整数を「12:3:-23:67」としてエンコードするなどです。これはシンプルで柔軟なアプローチですが、ワンオフのエンコーディングおよびパーシングコードの記述が必要であり、パーシングにはわずかなランタイムコストがかかります。これは非常に単純なデータをエンコードする場合に最適です。
-   データをXMLにシリアライズします。このアプローチは非常に魅力的であり、XMLは（ある程度）人間が読める形式であり、多くの言語にバインディングライブラリが存在するためです。他のアプリケーション/プロジェクトとデータを共有したい場合には適しています。ただし、XMLはスペースを多く必要とし、エンコード/デコードにはアプリケーションに大きなパフォーマンスペナルティを課す可能性があります。また、XML DOMツリーをナビゲートすることは、通常のクラスの単純なフィールドをナビゲートするよりもかなり複雑です。

プロトコルバッファは、この問題を解決するための柔軟で効率的な自動化されたソリューションです。プロトコルバッファを使用すると、保存したいデータ構造の`.proto`記述を作成します。その後、プロトコルバッファコンパイラは、効率的なバイナリ形式でプロトコルバッファデータの自動エンコーディングとパースを実装するクラスを作成します。生成されたクラスは、プロトコルバッファを構成するフィールドのゲッターとセッターを提供し、プロトコルバッファを単位として読み書きの詳細を処理します。重要なのは、プロトコルバッファ形式が、コードが古い形式でエンコードされたデータをまだ読み取れるように、時間の経過とともに形式を拡張するアイデアをサポートしていることです。

## 例コードの場所 {#example-code}

当社の例は、プロトコルバッファを使用してエンコードされたアドレスブックデータファイルを管理するためのコマンドラインアプリケーションのセットです。`dart add_person.dart`コマンドは、データファイルに新しいエントリを追加します。`dart list_people.dart`コマンドは、データファイルを解析してデータをコンソールに出力します。

GitHubリポジトリの[examplesディレクトリ](https://github.com/protocolbuffers/protobuf/tree/master/examples)に完全な例があります。

## プロトコル形式の定義 {#protocol-format}

アドレスブックアプリケーションを作成するには、`.proto`ファイルから始める必要があります。`.proto`ファイルの定義はシンプルです: シリアライズしたい各データ構造に対して*メッセージ*を追加し、メッセージ内の各フィールドに名前とタイプを指定します。この例では、メッセージを定義する`.proto`ファイルは[`addressbook.proto`](https://github.com/protocolbuffers/protobuf/blob/master/examples/addressbook.proto)です。

`.proto`ファイルは、異なるプロジェクト間の名前の競合を防ぐのに役立つパッケージ宣言で始まります。

```proto
syntax = "proto3";
package tutorial;

import "google/protobuf/timestamp.proto";
```

次に、メッセージの定義が続きます。メッセージは、単なる型付きフィールドのセットを含む集約です。`bool`、`int32`、`float`、`double`、`string`など、多くの標準的な単純データ型がフィールドタイプとして利用可能です。また、他のメッセージタイプをフィールドタイプとして使用することで、メッセージにさらなる構造を追加することもできます。

上記の例では、`Person` メッセージに `PhoneNumber` メッセージが含まれており、`AddressBook` メッセージには `Person` メッセージが含まれています。他のメッセージの中にネストされたメッセージ型を定義することもできます。例えば、`PhoneNumber` 型が `Person` の中で定義されていることがわかります。また、事前に定義された値のリストの中からフィールドの1つに値を持たせたい場合は、`enum` 型を定義することもできます。ここでは、電話番号が `PHONE_TYPE_MOBILE`、`PHONE_TYPE_HOME`、または `PHONE_TYPE_WORK` のいずれかであることを指定したいとします。

各要素の " = 1"、" = 2" のマーカーは、そのフィールドがバイナリエンコーディングで使用する一意の "tag" を識別します。タグ番号 1-15 は、高い番号よりも1バイト少なくエンコードする必要があるため、よく使用されるまたは繰り返し使用される要素にこれらのタグを使用することを最適化として選択できます。タグ 16 以上は、あまり一般的に使用されないオプション要素に残されます。繰り返しフィールド内の各要素は、タグ番号を再エンコードする必要があるため、繰り返しフィールドはこの最適化の特に適した候補です。

フィールドの値が設定されていない場合、[デフォルト値](/programming-guides/proto3#default) が使用されます。数値型の場合はゼロ、文字列の場合は空の文字列、bool 型の場合は false です。埋め込まれたメッセージの場合、デフォルト値は常にメッセージの "デフォルトインスタンス" または "プロトタイプ" であり、そのフィールドのいずれも設定されていません。明示的に設定されていないフィールドの値を取得するためのアクセサを呼び出すと、常にそのフィールドのデフォルト値が返されます。

フィールドが `repeated` の場合、そのフィールドは任意の回数（ゼロを含む）繰り返すことができます。プロトコルバッファ内で繰り返し値の順序が保持されます。繰り返しフィールドは、動的サイズの配列と考えてください。

`.proto` ファイルを記述するための完全なガイド -- すべての可能なフィールドタイプを含む -- は、[Protocol Buffer Language Guide](/programming-guides/proto3) にあります。
ただし、クラスの継承に類似した機能を探すことはしないでください -- プロトコルバッファはそのような機能を提供しません。

## Protocol Buffers のコンパイル {#compiling-protocol-buffers}

`.proto` ファイルができたら、次に行うべきことは、`AddressBook`（およびそれに伴う `Person` および `PhoneNumber`）メッセージを読み書きするために必要なクラスを生成することです。これを行うには、`.proto` ファイルに対してプロトコルバッファコンパイラ `protoc` を実行する必要があります。

1.  コンパイラをインストールしていない場合は、[パッケージをダウンロード](/downloads)して、READMEの指示に従ってください。

2.  Dart Protocol Buffer プラグインをインストールします。[そのREADME](https://github.com/google/protobuf.dart/tree/master/protoc_plugin#how-to-build)に記載されている手順に従います。実行可能ファイル `bin/protoc-gen-dart` は、プロトコルバッファ `protoc` がそれを見つけるためにあなたの `PATH` にある必要があります。

3.  今、コンパイラを実行し、ソースディレクトリ（アプリケーションのソースコードが存在する場所 -- 値を提供しない場合は現在のディレクトリが使用されます）、生成されたコードを配置する宛先ディレクトリ（通常は `$SRC_DIR` と同じ場所）、および `.proto` ファイルへのパスを指定します。この場合、次のように呼び出します：

    ```shell
    protoc -I=$SRC_DIR --dart_out=$DST_DIR $SRC_DIR/addressbook.proto
    ```

    Dartコードを生成したいため、`--dart_out` オプションを使用します -- 他のサポートされている言語に対しても同様のオプションが提供されています。

これにより、指定した宛先ディレクトリに `addressbook.pb.dart` が生成されます。

## Protocol Buffer API {#protobuf-api}

`addressbook.pb.dart` を生成すると、以下の便利な型が得られます：

-   `AddressBook` クラスと `List<Person> get people` ゲッターを持つ。
-   `Person` クラスと `name`、`id`、`email`、`phones` に対するアクセサメソッドを持つ。
-   `Person_PhoneNumber` クラスと `number`、`type` に対するアクセサメソッドを持つ。
-   `Person_PhoneType` クラスと、`Person.PhoneType` 列挙型の各値に対する静的フィールドを持つ。

生成される内容の詳細については、[Dart Generated Code ガイド](/reference/dart/dart-generated)を参照してください。

## メッセージの書き込み {#writing-a-message}

さて、プロトコルバッファクラスを使用してみましょう。アドレス帳アプリケーションができるようにしたい最初のことは、個人の詳細をアドレス帳ファイルに書き込むことです。これを行うには、プロトコルバッファクラスのインスタンスを作成し、値を設定してから、それらを出力ストリームに書き込む必要があります。

以下は、ファイルから `AddressBook` を読み取り、ユーザー入力に基づいて新しい `Person` を1つ追加し、新しい `AddressBook` を再度ファイルに書き込むプログラムです。プロトコルコンパイラによって生成されたコードを直接呼び出す部分または参照する部分が強調表示されています。

```dart
import 'dart:io';

import 'dart_tutorial/addressbook.pb.dart';

// This function fills in a Person message based on user input.
Person promptForAddress() {
  Person person = Person();

  print('Enter person ID: ');
  String input = stdin.readLineSync();
  person.id = int.parse(input);

  print('Enter name');
  person.name = stdin.readLineSync();

  print('Enter email address (blank for none) : ');
  String email = stdin.readLineSync();
  if (email.isNotEmpty) {
    person.email = email;
  }

  while (true) {
    print('Enter a phone number (or leave blank to finish): ');
    String number = stdin.readLineSync();
    if (number.isEmpty) break;

    Person_PhoneNumber phoneNumber = Person_PhoneNumber();

    phoneNumber.number = number;
    print('Is this a mobile, home, or work phone? ');

    String type = stdin.readLineSync();
    switch (type) {
      case 'mobile':
        phoneNumber.type = Person_PhoneType.PHONE_TYPE_MOBILE;
        break;
      case 'home':
        phoneNumber.type = Person_PhoneType.PHONE_TYPE_HOME;
        break;
      case 'work':
        phoneNumber.type = Person_PhoneType.PHONE_TYPE_WORK;
        break;
      default:
        print('Unknown phone type.  Using default.');
    }
    person.phones.add(phoneNumber);
  }

  return person;
}

// Reads the entire address book from a file, adds one person based
// on user input, then writes it back out to the same file.
main(List arguments) {
  if (arguments.length != 1) {
    print('Usage: add_person ADDRESS_BOOK_FILE');
    exit(-1);
  }

  File file = File(arguments.first);
  AddressBook addressBook;
  if (!file.existsSync()) {
    print('File not found. Creating new file.');
    addressBook = AddressBook();
  } else {
    addressBook = AddressBook.fromBuffer(file.readAsBytesSync());
  }
  addressBook.people.add(promptForAddress());
  file.writeAsBytes(addressBook.writeToBuffer());
}
```

## メッセージの読み取り {#reading-a-message}

もちろん、アドレス帳は、その中から情報を取得できない場合はあまり役に立ちません！この例では、上記の例で作成されたファイルを読み取り、その中のすべての情報を出力します。

```dart
import 'dart:io';

import 'dart_tutorial/addressbook.pb.dart';
import 'dart_tutorial/addressbook.pbenum.dart';

// Iterates though all people in the AddressBook and prints info about them.
void printAddressBook(AddressBook addressBook) {
  for (Person person in addressBook.people) {
    print('Person ID: ${ person.id}');
    print('  Name: ${ person.name}');
    if (person.hasEmail()) {
      print('  E-mail address:${ person.email}');
    }

    for (Person_PhoneNumber phoneNumber in person.phones) {
      switch (phoneNumber.type) {
        case Person_PhoneType.PHONE_TYPE_MOBILE:
          print('   Mobile phone #: ');
          break;
        case Person_PhoneType.PHONE_TYPE_HOME:
          print('   Home phone #: ');
          break;
        case Person_PhoneType.PHONE_TYPE_WORK:
          print('   Work phone #: ');
          break;
        default:
          print('   Unknown phone #: ');
          break;
      }
      print(phoneNumber.number);
    }
  }
}

// Reads the entire address book from a file and prints all
// the information inside.
main(List arguments) {
  if (arguments.length != 1) {
    print('Usage: list_person ADDRESS_BOOK_FILE');
    exit(-1);
  }

  // Read the existing address book.
  File file = new File(arguments.first);
 AddressBook addressBook = new AddressBook.fromBuffer(file.readAsBytesSync());
  printAddressBook(addressBook);
}
```

## Protocol Bufferの拡張 {#extending-a-protobuf}

プロトコルバッファを使用するコードをリリースした後、プロトコルバッファの定義を「改善」したくなることは避けられません。新しいバッファが後方互換性を持ち、古いバッファが前方互換性を持つようにしたい場合（そしてほぼ間違いなくそうしたいと思うでしょう）、以下のルールに従う必要があります。新しいバージョンのプロトコルバッファでは：

- 既存のフィールドのタグ番号を変更してはいけません。
- フィールドを削除しても構いません。
- 新しいフィールドを追加しても構いませんが、新しいタグ番号を使用する必要があります（つまり、このプロトコルバッファで使用されたことのないタグ番号を使用する必要があります。削除されたフィールドですら使用されていないタグ番号を使用する必要があります）。

（これらのルールには[いくつかの例外](/programming-guides/proto3#updating)がありますが、ほとんど使用されません。）

これらのルールに従うと、古いコードは新しいメッセージを問題なく読み取り、新しいフィールドを単に無視します。古いコードにとって、削除された単一フィールドは単にデフォルト値を持ち、削除された繰り返しフィールドは空になります。新しいコードも古いメッセージを透過的に読み取ります。

ただし、新しいフィールドは古いメッセージに存在しないため、デフォルト値で何かを適切に行う必要があります。タイプ固有の[デフォルト値](/programming-guides/proto3#default)が使用されます：文字列の場合、デフォルト値は空の文字列です。ブール値の場合、デフォルト値はfalseです。数値型の場合、デフォルト値はゼロです。
```
