+++
title = "Protocol Buffer Basics: Kotlin"
weight = 260
linkTitle = "Kotlin"
description = "プロトコルバッファを扱うための基本的なKotlinプログラマー向けの紹介。"
type = "docs"
+++

このチュートリアルでは、プロトコルバッファを扱うための基本的なKotlinプログラマー向けの紹介を提供します。プロトコルバッファ言語の[proto3](/programming-guides/proto3)バージョンを使用します。単純な例のアプリケーションを作成することで、以下の方法を示します。

- `.proto`ファイルでメッセージ形式を定義する。
- プロトコルバッファコンパイラを使用する。
- KotlinプロトコルバッファAPIを使用してメッセージを書き込み、読み取る。

これはKotlinでプロトコルバッファを使用する包括的なガイドではありません。詳細なリファレンス情報については、[Protocol Buffer Language Guide](/programming-guides/proto3)、[Kotlin API Reference](/reference/kotlin/api-docs)、[Kotlin Generated Code Guide](/reference/kotlin/kotlin-generated)、および[Encoding Reference](/programming-guides/encoding)を参照してください。

## 問題領域 {#problem-domain}

使用する例は非常にシンプルな「アドレス帳」アプリケーションです。このアプリケーションは、人々の連絡先詳細をファイルに読み書きできます。アドレス帳の各人物には、名前、ID、メールアドレス、連絡先電話番号があります。

このような構造化データをシリアライズおよび取得する方法はどうすればよいでしょうか？この問題を解決するためのいくつかの方法があります。

- kotlinx.serializationを使用します。これは、C++やPythonで書かれたアプリケーションとデータを共有する必要がある場合にはあまり適していません。kotlinx.serializationには[protobufモード](https://github.com/Kotlin/kotlinx.serialization/blob/master/docs/formats.md#protobuf-experimental)がありますが、これはプロトコルバッファのすべての機能を提供しません。
- データ項目を単一の文字列にエンコードするための特別な方法を考案することができます。たとえば、4つの整数を"12:3:-23:67"としてエンコードするなどです。これはシンプルで柔軟なアプローチですが、一時的なエンコードおよびパースコードの記述が必要であり、パースにはわずかなランタイムコストがかかります。これは非常に単純なデータをエンコードする場合に最適です。
- データをXMLにシリアライズします。このアプローチは非常に魅力的かもしれません。なぜなら、XMLは（ある程度）人間が読める形式であり、多くの言語にバインディングライブラリが存在するからです。他のアプリケーション/プロジェクトとデータを共有したい場合には適しています。ただし、XMLはスペースを多く必要とし、エンコード/デコードにはアプリケーションに大きなパフォーマンスペナルティを課す可能性があります。また、XML DOMツリーをナビゲートすることは、通常のクラスのフィールドをナビゲートするよりもかなり複雑です。

プロトコルバッファは、この問題を解決する柔軟で効率的な自動化されたソリューションです。プロトコルバッファを使用すると、保存したいデータ構造の`.proto`記述を作成します。その後、プロトコルバッファコンパイラは、効率的なバイナリ形式でプロトコルバッファデータの自動エンコーディングとパーシングを実装するクラスを作成します。生成されたクラスは、プロトコルバッファを構成するフィールドのゲッターとセッターを提供し、プロトコルバッファを単位として読み書きの詳細を処理します。重要なのは、プロトコルバッファ形式が、コードが古い形式でエンコードされたデータをまだ読み取れるように、時間の経過とともに形式を拡張するアイデアをサポートしていることです。

## 例コードの場所 {#example-code}

当社の例は、プロトコルバッファを使用してエンコードされたアドレス帳データファイルを管理するためのコマンドラインアプリケーションのセットです。`add_person_kotlin`コマンドは、データファイルに新しいエントリを追加します。`list_people_kotlin`コマンドは、データファイルを解析し、データをコンソールに出力します。

GitHubリポジトリの[examplesディレクトリ](https://github.com/protocolbuffers/protobuf/tree/master/examples)で完全な例を見つけることができます。

## プロトコル形式の定義 {#protocol-format}

アドレス帳アプリケーションを作成するには、`.proto`ファイルから始める必要があります。`.proto`ファイルの定義はシンプルです：シリアライズしたい各データ構造に対して*メッセージ*を追加し、メッセージ内の各フィールドに名前とタイプを指定します。この例では、メッセージを定義する`.proto`ファイルは[`addressbook.proto`](https://github.com/protocolbuffers/protobuf/blob/master/examples/addressbook.proto)です。

`.proto`ファイルは、異なるプロジェクト間の名前の競合を防ぐのに役立つパッケージ宣言で始まります。

```proto
syntax = "proto3";
package tutorial;

import "google/protobuf/timestamp.proto";
```

次に、メッセージ定義が続きます。メッセージは、型付きフィールドのセットを含む集約体です。`bool`、`int32`、`float`、`double`、`string`など、多くの標準的な単純データ型がフィールドタイプとして利用可能です。また、他のメッセージタイプをフィールドタイプとして使用することで、メッセージにさらなる構造を追加することもできます。

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

上記の例では、`Person` メッセージに `PhoneNumber` メッセージが含まれており、
`AddressBook` メッセージには `Person` メッセージが含まれています。他のメッセージの中にネストされたメッセージ型を定義することもできます。例えば、`PhoneNumber` 型が `Person` の中で定義されていることがわかります。また、事前に定義された値のリストの中から1つをフィールドの値として持つようにしたい場合は、`enum` 型を定義することもできます。ここでは、電話番号が `PHONE_TYPE_MOBILE`、`PHONE_TYPE_HOME`、または `PHONE_TYPE_WORK` のいずれかであることを指定したいとします。

各要素の " = 1"、" = 2" のマーカーは、そのフィールドがバイナリエンコーディングで使用する一意の "tag" を識別します。タグ番号 1-15 は、高い番号よりも1バイト少なくエンコードする必要があるため、よく使用されるまたは繰り返し使用される要素にこれらのタグを使用することを最適化として選択できます。タグ 16 以上は、あまり一般的に使用されないオプション要素に残されます。繰り返しフィールド内の各要素は、タグ番号を再エンコードする必要があるため、繰り返しフィールドは特にこの最適化の対象となります。

フィールドの値が設定されていない場合、[デフォルト値](/programming-guides/proto3#default) が使用されます。数値型の場合はゼロ、文字列の場合は空の文字列、bool 型の場合は false です。埋め込みメッセージの場合、デフォルト値は常にメッセージの "デフォルトインスタンス" または "プロトタイプ" であり、そのフィールドのいずれも設定されていません。明示的に設定されていないフィールドの値を取得するためのアクセサを呼び出すと、常にそのフィールドのデフォルト値が返されます。

フィールドが `repeated` の場合、そのフィールドは任意の回数（ゼロを含む）繰り返すことができます。プロトコルバッファ内で繰り返し値の順序が保持されます。繰り返しフィールドは、動的サイズの配列と考えてください。

`.proto` ファイルを記述するための完全なガイド -- すべての可能なフィールドタイプを含む -- は、[Protocol Buffer Language Guide](/programming-guides/proto3) にあります。
ただし、クラスの継承に類似した機能を探すことはやめてください -- プロトコルバッファはそのような機能を提供していません。

## Protocol Buffers のコンパイル {#compiling-protocol-buffers}

`.proto` ファイルがあるので、次に行う必要があるのは、`AddressBook`（およびそれに伴う `Person` および `PhoneNumber`）メッセージを読み書きするために必要なクラスを生成することです。これを行うには、`.proto` ファイルに対してプロトコルバッファコンパイラ `protoc` を実行する必要があります。

1.  コンパイラをインストールしていない場合は、[パッケージをダウンロード](/downloads)して、READMEの手順に従ってください。

2.  今、コンパイラを実行し、ソースディレクトリ（アプリケーションのソースコードが存在する場所 - 指定しない場合は現在のディレクトリが使用されます）、生成されたコードを配置する宛先ディレクトリ（通常は`$SRC_DIR`と同じ）、および`.proto`へのパスを指定します。この場合、次のように呼び出します：

    ```shell
    protoc -I=$SRC_DIR --java_out=$DST_DIR --kotlin_out=$DST_DIR $SRC_DIR/addressbook.proto
    ```

    Kotlinコードを生成したい場合は、`--kotlin_out`オプションを使用します。他のサポートされている言語についても同様のオプションが提供されています。

Kotlinコードを生成する場合は、`--java_out`と`--kotlin_out`の両方を使用する必要があります。これにより、指定したJava宛先ディレクトリに`com/example/tutorial/protos/`サブディレクトリが生成され、いくつかの生成された`.java`ファイルと指定したKotlin宛先ディレクトリに`com/example/tutorial/protos/`サブディレクトリが生成され、いくつかの生成された`.kt`ファイルが含まれます。

## Protocol Buffer API {#protobuf-api}

Kotlin向けのプロトコルバッファコンパイラは、Java向けのプロトコルバッファ用に生成された既存のAPIに追加されるKotlin APIを生成します。これにより、JavaとKotlinで書かれたコードベースが、特別な処理や変換なしに同じプロトコルバッファメッセージオブジェクトとやり取りできるようになります。

JavaScriptやネイティブなど、他のKotlinコンパイルターゲット向けのプロトコルバッファは現在サポートされていません。

`addressbook.proto`をコンパイルすると、次のAPIがJavaで生成されます：

-   `AddressBook`クラス
    -   Kotlinからは、`peopleList : List<Person>`プロパティを持っています
-   `Person`クラス
    -   Kotlinからは、`name`、`id`、`email`、`phonesList`プロパティを持っています
    -   `Person.PhoneNumber`ネストクラスは、`number`と`type`プロパティを持っています
    -   `Person.PhoneType`ネストされた列挙型

さらに、次のKotlin APIが生成されます：

-   `addressBook { ... }`および`person { ... }`ファクトリメソッド
-   `PersonKt`オブジェクトは、`phoneNumber { ... }`ファクトリメソッドを持っています


[Kotlin Generated Code guide](/reference/kotlin/kotlin-generated)で生成される詳細について詳しく読むことができます。

## メッセージの書き込み {#writing-a-message}

さて、プロトコルバッファクラスを使用してみましょう。アドレス帳アプリケーションができるようにしたい最初のことは、個人の詳細をアドレス帳ファイルに書き込むことです。これを行うには、プロトコルバッファクラスのインスタンスを作成し、値を設定してから、それらを出力ストリームに書き込む必要があります。

以下は、ファイルから`AddressBook`を読み取り、ユーザーの入力に基づいて1つの新しい`Person`を追加し、新しい`AddressBook`を再度ファイルに書き込むプログラムです。プロトコルコンパイラによって生成されたコードを直接呼び出す部分が強調表示されています。

```kotlin
import com.example.tutorial.Person
import com.example.tutorial.AddressBook
import com.example.tutorial.person
import com.example.tutorial.addressBook
import com.example.tutorial.PersonKt.phoneNumber
import java.util.Scanner

// This function fills in a Person message based on user input.
fun promptPerson(): Person = person {
  print("Enter person ID: ")
  id = readLine().toInt()

  print("Enter name: ")
  name = readLine()

  print("Enter email address (blank for none): ")
  val email = readLine()
  if (email.isNotEmpty()) {
    this.email = email
  }

  while (true) {
    print("Enter a phone number (or leave blank to finish): ")
    val number = readLine()
    if (number.isEmpty()) break

    print("Is this a mobile, home, or work phone? ")
    val type = when (readLine()) {
      "mobile" -> Person.PhoneType.PHONE_TYPE_MOBILE
      "home" -> Person.PhoneType.PHONE_TYPE_HOME
      "work" -> Person.PhoneType.PHONE_TYPE_WORK
      else -> {
        println("Unknown phone type.  Using home.")
        Person.PhoneType.PHONE_TYPE_HOME
      }
    }
    phones += phoneNumber {
      this.number = number
      this.type = type
    }
  }
}

// Reads the entire address book from a file, adds one person based
// on user input, then writes it back out to the same file.
fun main(args: List) {
  if (arguments.size != 1) {
    println("Usage: add_person ADDRESS_BOOK_FILE")
    exitProcess(-1)
  }
  val path = Path(arguments.single())
  val initialAddressBook = if (!path.exists()) {
    println("File not found. Creating new file.")
    addressBook {}
  } else {
    path.inputStream().use {
      AddressBook.newBuilder().mergeFrom(it).build()
    }
  }
  path.outputStream().use {
    initialAddressBook.copy { peopleList += promptPerson() }.writeTo(it)
  }
}
```

## メッセージの読み取り {#reading-a-message}

もちろん、アドレス帳から情報を取得できない場合、アドレス帳はあまり役に立ちません！この例では、上記の例で作成されたファイルを読み取り、その中のすべての情報を出力します。

```kotlin
import com.example.tutorial.Person
import com.example.tutorial.AddressBook

// Iterates though all people in the AddressBook and prints info about them.
fun print(addressBook: AddressBook) {
  for (person in addressBook.peopleList) {
    println("Person ID: ${person.id}")
    println("  Name: ${person.name}")
    if (person.hasEmail()) {
      println("  Email address: ${person.email}")
    }
    for (phoneNumber in person.phonesList) {
      val modifier = when (phoneNumber.type) {
        Person.PhoneType.PHONE_TYPE_MOBILE -> "Mobile"
        Person.PhoneType.PHONE_TYPE_HOME -> "Home"
        Person.PhoneType.PHONE_TYPE_WORK -> "Work"
        else -> "Unknown"
      }
      println("  $modifier phone #: ${phoneNumber.number}")
    }
  }
}

fun main(args: List) {
  if (arguments.size != 1) {
    println("Usage: list_person ADDRESS_BOOK_FILE")
    exitProcess(-1)
  }
  Path(arguments.single()).inputStream().use {
    print(AddressBook.newBuilder().mergeFrom(it).build())
  }
}
```

## プロトコルバッファの拡張 {#extending-a-protobuf}

プロトコルバッファを使用するコードをリリースした後、プロトコルバッファの定義を「改善」したくなることが遅かれ早かれ起こるでしょう。新しいバッファが後方互換性を持ち、古いバッファが前方互換性を持つようにしたい場合 -- そしておそらくほとんどの場合そうしたいでしょう -- その場合、従う必要があるいくつかのルールがあります。プロトコルバッファの新しいバージョンでは：

- 既存のフィールドのタグ番号を変更してはいけません。
- フィールドを削除しても構いません。
- 新しいフィールドを追加しても構いませんが、新しいタグ番号を使用する必要があります（つまり、このプロトコルバッファで使用されたことのないタグ番号を使用する必要があります。削除されたフィールドですら使用されていないタグ番号を使用する必要があります）。

(これらのルールには[いくつかの例外](/programming-guides/proto3#updating)がありますが、それらはほとんど使用されません。)

これらのルールに従うと、古いコードは新しいメッセージを問題なく読み取り、新しいフィールドを単に無視します。古いコードにとって、削除された単一フィールドは単にデフォルト値を持ち、削除された繰り返しフィールドは空になります。新しいコードも古いメッセージを透過的に読み取ります。

ただし、古いメッセージに新しいフィールドが存在しないことを考慮してください。そのため、デフォルト値を適切に扱う必要があります。型固有の[デフォルト値](/programming-guides/proto3#default)が使用されます。文字列の場合、デフォルト値は空の文字列です。ブール値の場合、デフォルト値はfalseです。数値型の場合、デフォルト値はゼロです。
