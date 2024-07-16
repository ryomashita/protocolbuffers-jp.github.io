+++
title = "プロトコルバッファの基礎: Java"
weight = 250
linkTitle = "Java"
description = "プロトコルバッファを使用するための基本的なJavaプログラマー向けの紹介。"
type = "docs"
+++

このチュートリアルでは、プロトコルバッファを使用するための基本的なJavaプログラマー向けの紹介を提供します。単純な例のアプリケーションを作成することで、次の方法を示します。

- `.proto`ファイルでメッセージ形式を定義する。
- プロトコルバッファコンパイラを使用する。
- JavaプロトコルバッファAPIを使用してメッセージを書き込み、読み取る。

これはJavaでプロトコルバッファを使用する包括的なガイドではありません。詳細なリファレンス情報については、[Protocol Buffer Language Guide (proto2)](/programming-guides/proto2)、[Protocol Buffer Language Guide (proto3)](/programming-guides/proto3)、[Java API Reference](/reference/java/api-docs/overview-summary.html)、[Java Generated Code Guide](/reference/java/java-generated)、および[Encoding Reference](/programming-guides/encoding)を参照してください。

## 問題領域 {#problem-domain}

使用する例は非常にシンプルな「アドレス帳」アプリケーションで、人々の連絡先詳細をファイルに読み書きできるものです。アドレス帳の各人物には、名前、ID、メールアドレス、連絡先電話番号があります。

このような構造化データをシリアライズおよび取得するにはどうすればよいですか？この問題を解決するためのいくつかの方法があります。

- Javaシリアライゼーションを使用します。これはデフォルトのアプローチであり、言語に組み込まれているため、一般的な問題がいくつかあります（Josh Blochによる「Effective Java」p. 213を参照）、また、C++やPythonで書かれたアプリケーションとデータを共有する必要がある場合にはあまりうまく機能しません。
- データ項目を単一の文字列にエンコードするための特別な方法を考案することができます。たとえば、4つの整数を「12:3:-23:67」としてエンコードするなどです。これはシンプルで柔軟なアプローチですが、一時的なエンコーディングおよびパースコードの記述が必要であり、パースにはわずかなランタイムコストがかかります。これは非常に単純なデータをエンコードする場合に最適です。
- データをXMLにシリアライズします。このアプローチは非常に魅力的であり、XMLは（ある程度）人間が読める形式であり、多くの言語にバインディングライブラリが存在するためです。他のアプリケーション/プロジェクトとデータを共有したい場合には適しています。ただし、XMLはスペースを多く使用し、エンコード/デコードにはアプリケーションに大きなパフォーマンスペナルティを課す可能性があります。また、XML DOMツリーをナビゲートすることは、通常のクラスのフィールドをナビゲートするよりもかなり複雑です。

代わりにこれらのオプションを使用することができます。プロトコルバッファは、この問題を解決する柔軟で効率的で自動化されたソリューションです。プロトコルバッファを使用すると、保存したいデータ構造の`.proto`の記述を作成します。その後、プロトコルバッファコンパイラが、プロトコルバッファデータの自動エンコーディングとパーシングを実装するクラスを作成し、効率的なバイナリ形式でデータを処理します。生成されたクラスは、プロトコルバッファを構成するフィールドのためのゲッターとセッターを提供し、プロトコルバッファを単位として読み書きする詳細を処理します。重要なのは、プロトコルバッファ形式が、コードが古い形式でエンコードされたデータをまだ読み取れるように、時間の経過とともに形式を拡張する考え方をサポートしていることです。

## 例コードの場所 {#example-code}

例コードは、ソースコードパッケージに含まれており、"examples"ディレクトリの下にあります。[こちらからダウンロードしてください。](/downloads)

## プロトコル形式の定義 {#protocol-format}

アドレス帳アプリケーションを作成するには、`.proto`ファイルから始める必要があります。`.proto`ファイルの定義はシンプルです。シリアライズしたいデータ構造ごとに*メッセージ*を追加し、メッセージ内の各フィールドに名前とタイプを指定します。以下は、メッセージを定義する`.proto`ファイルである`addressbook.proto`です。

```proto
syntax = "proto2";

package tutorial;

option java_multiple_files = true;
option java_package = "com.example.tutorial.protos";
option java_outer_classname = "AddressBookProtos";

message Person {
  optional string name = 1;
  optional int32 id = 2;
  optional string email = 3;

  enum PhoneType {
    PHONE_TYPE_UNSPECIFIED = 0;
    PHONE_TYPE_MOBILE = 1;
    PHONE_TYPE_HOME = 2;
    PHONE_TYPE_WORK = 3;
  }

  message PhoneNumber {
    optional string number = 1;
    optional PhoneType type = 2 [default = PHONE_TYPE_HOME];
  }

  repeated PhoneNumber phones = 4;
}

message AddressBook {
  repeated Person people = 1;
}
```

C++やJavaに似た構文であることがわかります。ファイルの各部分を見て、それぞれが何をしているかを見ていきましょう。

`.proto`ファイルは、パッケージ宣言で始まります。これは異なるプロジェクト間の名前の衝突を防ぐのに役立ちます。Javaでは、パッケージ名はJavaパッケージとして使用されますが、ここで`java_package`を明示的に指定している場合を除きます。`java_package`を指定しない場合、通常は`package`宣言で指定されたパッケージ名と一致しますが、これらの名前は通常、適切なJavaパッケージ名ではありません（通常、ドメイン名で始まらないため）。`java_outer_classname`オプションは、このファイルを表すラッパークラスのクラス名を定義します。明示的に`java_outer_classname`を指定しない場合、ファイル名をアッパーキャメルケースに変換してラッパークラス名として生成されます。例えば、"my_proto.proto"は、デフォルトではラッパークラス名として"MyProto"を使用します。`java_multiple_files = true`オプションは、生成された各クラスに対して別々の`.java`ファイルを生成することを可能にします（ラッパークラスの単一の`.java`ファイルを生成し、ラッパークラスを外部クラスとして使用し、他のすべてのクラスをラッパークラス内にネストする従来の動作とは異なります）。

次に、メッセージの定義があります。メッセージは、型付きフィールドのセットを含む集合体です。`bool`、`int32`、`float`、`double`、`string`など、多くの標準的な単純データ型がフィールドタイプとして利用可能です。他のメッセージタイプをフィールドタイプとして使用することで、メッセージにさらなる構造を追加することもできます。上記の例では、`Person`メッセージが`PhoneNumber`メッセージを含み、`AddressBook`メッセージが`Person`メッセージを含んでいます。さらに、他のメッセージの中にネストされたメッセージタイプを定義することもできます。例えば、`PhoneNumber`タイプが`Person`の内部で定義されています。また、`enum`タイプを定義することもでき、フィールドの1つに事前定義された値のリストから1つを持つことを指定したい場合があります。ここでは、電話番号が次の電話タイプのいずれかであることを指定したいとします：`PHONE_TYPE_MOBILE`、`PHONE_TYPE_HOME`、または`PHONE_TYPE_WORK`。

各要素の " = 1"、" = 2" のマーカーは、バイナリエンコーディングでフィールドが使用する一意の "tag" を識別します。タグ番号 1-15 は、高い番号よりも1バイト少なくエンコードする必要があるため、よく使用されるまたは繰り返し使用される要素にこれらのタグを使用することを最適化として選択できます。一方、タグ 16 以上は、あまり一般的に使用されないオプション要素に残されます。繰り返しフィールド内の各要素には、タグ番号を再エンコードする必要があるため、繰り返しフィールドはこの最適化の特に適した候補です。

各フィールドは、次の修飾子のいずれかで注釈付けする必要があります：

- `optional`：フィールドは設定されていてもされていなくてもかまいません。オプションのフィールド値が設定されていない場合、デフォルト値が使用されます。単純な型の場合、例の電話番号の `type` で行ったように、独自のデフォルト値を指定できます。それ以外の場合、システムのデフォルト値が使用されます：数値型の場合はゼロ、文字列の場合は空の文字列、bool 型の場合は false です。埋め込みメッセージの場合、デフォルト値は常にメッセージの "デフォルトインスタンス" または "プロトタイプ" であり、そのフィールドが設定されていない状態です。明示的に設定されていないオプション（または必須）フィールドの値を取得するためにアクセサを呼び出すと、常にそのフィールドのデフォルト値が返されます。
- `repeated`：フィールドは任意の回数（0を含む）繰り返すことができます。繰り返し値の順序はプロトコルバッファ内で保持されます。繰り返しフィールドは、動的サイズの配列と考えることができます。
- `required`：フィールドに値を指定する必要があります。そうでない場合、メッセージは "未初期化" と見なされます。未初期化のメッセージを構築しようとすると `RuntimeException` がスローされます。未初期化のメッセージを解析しようとすると `IOException` がスローされます。それ以外の点では、必須フィールドはオプションフィールドとまったく同じように動作します。

{{% alert title="重要" color="warning" %}} **必須は永遠です**
`required` フィールドをマークする際には非常に注意する必要があります。ある時点で必須フィールドの書き込みや送信を停止したい場合、そのフィールドをオプションフィールドに変更することは問題を引き起こす可能性があります -- 古いリーダーはこのフィールドがないメッセージを不完全と見なし、意図せずそれらを拒否または破棄するかもしれません。代わりに、バッファー用のアプリケーション固有のカスタムバリデーションルーチンを作成することを検討すべきです。Google内では、`required` フィールドは強く非推奨です。proto2 構文で定義されたほとんどのメッセージは `optional` と `repeated` のみを使用します。（Proto3 では `required` フィールドをサポートしていません。）
{{% /alert %}}

[Protocol Buffer Language Guide](/programming-guides/proto2) には、`.proto` ファイルを記述するための完全なガイドが含まれています -- すべての可能なフィールドタイプも含まれています。
クラスの継承に類似した機能を探すのはやめておいた方が良いです -- プロトコルバッファはそのような機能を提供していません。

## Protocol Buffers のコンパイル {#compiling-protocol-buffers}

`.proto` ファイルを持っている場合、次に行う必要があるのは、`AddressBook`（およびそれに伴う `Person` および `PhoneNumber`）メッセージを読み書きするために必要なクラスを生成することです。これを行うには、`.proto` ファイルに対してプロトコルバッファコンパイラ `protoc` を実行する必要があります：

1. コンパイラをインストールしていない場合は、[パッケージをダウンロード](/downloads) して README の手順に従ってください。

1. 次に、コンパイラを実行し、ソースディレクトリ（アプリケーションのソースコードが存在するディレクトリ -- 値を提供しない場合は現在のディレクトリが使用されます）、生成されたコードを配置する宛先ディレクトリ（通常は `$SRC_DIR` と同じ）、および `.proto` ファイルへのパスを指定します。この場合、以下のように...:

    ```shell
    protoc -I=$SRC_DIR --java_out=$DST_DIR $SRC_DIR/addressbook.proto
    ```

    Java クラスを生成したい場合は、`--java_out` オプションを使用します -- 他のサポートされている言語に対しても同様のオプションが提供されています。

これにより、指定した宛先ディレクトリに `com/example/tutorial/protos/` サブディレクトリが生成され、いくつかの `.java` ファイルが含まれます。

## プロトコルバッファAPI {#protobuf-api}

生成されたコードのいくつかを見て、コンパイラが作成したクラスやメソッドを確認しましょう。`com/example/tutorial/protos/` にあると見ることができます。`addressbook.proto` で指定した各メッセージに対するクラスを定義する `.java` ファイルが含まれています。各クラスには、そのクラスのインスタンスを作成するために使用する `Builder` クラスがあります。ビルダーについては、以下の [ビルダー vs. メッセージ](#builders-messages) セクションで詳細を確認できます。

メッセージとビルダーの両方には、メッセージの各フィールドに対する自動生成されたアクセサメソッドがあります。メッセージにはゲッターのみがあり、一方、ビルダーにはゲッターとセッターの両方があります。以下は、`Person` クラスのアクセサの一部です（実装は簡潔のため省略）：

```java
// required string name = 1;
public boolean hasName();
public String getName();

// required int32 id = 2;
public boolean hasId();
public int getId();

// optional string email = 3;
public boolean hasEmail();
public String getEmail();

// repeated .tutorial.Person.PhoneNumber phones = 4;
public List<PhoneNumber> getPhonesList();
public int getPhonesCount();
public PhoneNumber getPhones(int index);
```

一方、`Person.Builder` には同じゲッターに加えてセッターもあります：

```java
// required string name = 1;
public boolean hasName();
public java.lang.String getName();
public Builder setName(String value);
public Builder clearName();

// required int32 id = 2;
public boolean hasId();
public int getId();
public Builder setId(int value);
public Builder clearId();

// optional string email = 3;
public boolean hasEmail();
public String getEmail();
public Builder setEmail(String value);
public Builder clearEmail();

// repeated .tutorial.Person.PhoneNumber phones = 4;
public List<PhoneNumber> getPhonesList();
public int getPhonesCount();
public PhoneNumber getPhones(int index);
public Builder setPhones(int index, PhoneNumber value);
public Builder addPhones(PhoneNumber value);
public Builder addAllPhones(Iterable<PhoneNumber> value);
public Builder clearPhones();
```

ご覧の通り、各フィールドに対してシンプルなJavaBeansスタイルのゲッターとセッターがあります。また、各単数フィールドに対して `has` ゲッターもあり、そのフィールドが設定されている場合に true を返します。最後に、各フィールドには、そのフィールドを空の状態に戻す `clear` メソッドもあります。

繰り返しフィールドにはいくつかの追加メソッドがあります -- `Count` メソッド（リストのサイズの省略形）、インデックスでリストの特定の要素を取得または設定するゲッターとセッター、新しい要素をリストに追加する `add` メソッド、およびリストに要素を追加する `addAll` メソッドがあります。

これらのアクセサメソッドがキャメルケースの命名規則を使用していることに注意してください。`.proto` ファイルではアンダースコア付きの小文字が使用されていますが、この変換はプロトコルバッファコンパイラによって自動的に行われ、生成されたクラスが標準のJavaスタイル規則に一致するようになっています。`.proto` ファイルのフィールド名には常にアンダースコア付きの小文字を使用する必要があります。これにより、生成されたすべての言語で良い命名規則が確保されます。良い`.proto` スタイルについては、[スタイルガイド](/programming-guides/style) を参照してください。

特定のフィールド定義に対してプロトコルコンパイラが生成するメンバーについての詳細情報については、[Java生成コードリファレンス](/reference/java/java-generated) を参照してください。

### 列挙型とネストされたクラス {#enums-nested-classes}

生成されたコードには、`Person` 内にネストされた `PhoneType` Java 5 列挙型が含まれています:

```java
public static enum PhoneType {
  PHONE_TYPE_UNSPECIFIED(0, 0),
  PHONE_TYPE_MOBILE(1, 1),
  PHONE_TYPE_HOME(2, 2),
  PHONE_TYPE_WORK(3, 3),
  ;
  ...
}
```

ネストされた型 `Person.PhoneNumber` は、`Person` 内のネストされたクラスとして生成されます。

### ビルダー vs. メッセージ {#builders-messages}

プロトコルバッファコンパイラによって生成されるメッセージクラスはすべて*不変*です。メッセージオブジェクトが構築されると、Java の `String` のように変更できません。メッセージを構築するには、まずビルダーを構築し、設定したいフィールドを選択した値に設定し、その後ビルダーの `build()` メソッドを呼び出す必要があります。

ビルダーの各メソッドがメッセージを変更する場合、戻り値は別のビルダーです。返されるオブジェクトは実際には、メソッドを呼び出したビルダーそのものです。これは、1行のコードで複数のセッターを連結できるように便宜上返されます。

以下は、`Person` のインスタンスを作成する方法の例です:

```java
Person john =
  Person.newBuilder()
    .setId(1234)
    .setName("John Doe")
    .setEmail("jdoe@example.com")
    .addPhones(
      Person.PhoneNumber.newBuilder()
        .setNumber("555-4321")
        .setType(Person.PhoneType.PHONE_TYPE_HOME)
        .build());
    .build();
```

### 標準メッセージメソッド {#standard-message-methods}

各メッセージおよびビルダークラスには、すべての必須フィールドが設定されているかどうかを確認したり、メッセージ全体を操作したりするための他の多くのメソッドが含まれています:

-   `isInitialized()`: すべての必須フィールドが設定されているかどうかを確認します。
-   `toString()`: メッセージの人間が読める表現を返します。デバッグに特に便利です。
-   `mergeFrom(Message other)`: (ビルダーのみ) `other` の内容をこのメッセージにマージし、単一のスカラーフィールドを上書きし、複合フィールドをマージし、繰り返しフィールドを連結します。
-   `clear()`: (ビルダーのみ) すべてのフィールドを空の状態にクリアします。

これらのメソッドは、すべての Java メッセージおよびビルダーで共有される `Message` および `Message.Builder` インターフェースを実装しています。詳細については、[ `Message` の完全な API ドキュメント](/reference/java/api-docs/com/google/protobuf/Message.html)を参照してください。

### パースとシリアライゼーション {#parsing-serialization}

最後に、各プロトコルバッファクラスには、プロトコルバッファの[バイナリ形式](/programming-guides/encoding)を使用して、選択したタイプのメッセージを書き込み、読み取るためのメソッドがあります。これには次のものが含まれます:

-   `byte[] toByteArray();`: メッセージをシリアライズして、その生のバイトを含むバイト配列を返します。
-   `static Person parseFrom(byte[] data);`: 指定されたバイト配列からメッセージを解析します。
-   `void writeTo(OutputStream output);`: メッセージをシリアライズして、`OutputStream` に書き込みます。
-   `static Person parseFrom(InputStream input);`: `InputStream` からメッセージを読み取り、解析します。

これは、解析とシリアル化のために提供されるオプションの一部です。
詳細なリストについては、再度、[`Message` API リファレンス](/reference/java/api-docs/com/google/protobuf/Message.html) を参照してください。

{{% alert title="重要" color="warning" %}} **プロトコルバッファとオブジェクト指向設計**
プロトコルバッファクラスは基本的にデータホルダー（C 言語の構造体のようなもの）であり、追加の機能を提供しません。これらはオブジェクトモデルにおいて優れたファーストクラスの市民ではありません。生成されたクラスにより豊かな動作を追加したい場合、これを行う最良の方法は、生成されたプロトコルバッファクラスをアプリケーション固有のクラスでラップすることです。プロトコルバッファをラップすることは、`.proto` ファイルの設計を制御できない場合にも良い考えです（別のプロジェクトから再利用する場合など）。その場合、ラッパークラスを使用して、アプリケーションの独自の環境に適したインターフェースを作成できます。データやメソッドを非表示にし、便利な機能を公開するなどです。**生成されたクラスに動作を追加することは決してありません**。これは内部メカニズムを壊し、それに加えて、良いオブジェクト指向の実践ではありません。{{% /alert %}}

## メッセージの書き込み {#writing-a-message}

さて、プロトコルバッファクラスを使用してみましょう。最初にアドレス帳アプリケーションができるようにしたいことは、個人の詳細をアドレス帳ファイルに書き込むことです。これを行うには、プロトコルバッファクラスのインスタンスを作成し、値を設定してから、それらを出力ストリームに書き込む必要があります。

以下は、ファイルから `AddressBook` を読み取り、ユーザー入力に基づいて新しい `Person` を1つ追加し、新しい `AddressBook` を再度ファイルに書き出すプログラムです。プロトコルコンパイラによって生成されたコードを直接呼び出す部分または参照する部分は、ハイライトされています。

```java
import com.example.tutorial.protos.AddressBook;
import com.example.tutorial.protos.Person;
import java.io.BufferedReader;
import java.io.FileInputStream;
import java.io.FileNotFoundException;
import java.io.FileOutputStream;
import java.io.InputStreamReader;
import java.io.IOException;
import java.io.PrintStream;

class AddPerson {
  // This function fills in a Person message based on user input.
  static Person PromptForAddress(BufferedReader stdin,
                                 PrintStream stdout) throws IOException {
    Person.Builder person = Person.newBuilder();

    stdout.print("Enter person ID: ");
    person.setId(Integer.valueOf(stdin.readLine()));

    stdout.print("Enter name: ");
    person.setName(stdin.readLine());

    stdout.print("Enter email address (blank for none): ");
    String email = stdin.readLine();
    if (email.length() > 0) {
      person.setEmail(email);
    }

    while (true) {
      stdout.print("Enter a phone number (or leave blank to finish): ");
      String number = stdin.readLine();
      if (number.length() == 0) {
        break;
      }

      Person.PhoneNumber.Builder phoneNumber =
        Person.PhoneNumber.newBuilder().setNumber(number);

      stdout.print("Is this a mobile, home, or work phone? ");
      String type = stdin.readLine();
      if (type.equals("mobile")) {
        phoneNumber.setType(Person.PhoneType.PHONE_TYPE_MOBILE);
      } else if (type.equals("home")) {
        phoneNumber.setType(Person.PhoneType.PHONE_TYPE_HOME);
      } else if (type.equals("work")) {
        phoneNumber.setType(Person.PhoneType.PHONE_TYPE_WORK);
      } else {
        stdout.println("Unknown phone type.  Using default.");
      }

      person.addPhones(phoneNumber);
    }

    return person.build();
  }

  // Main function:  Reads the entire address book from a file,
  //   adds one person based on user input, then writes it back out to the same
  //   file.
  public static void main(String[] args) throws Exception {
    if (args.length != 1) {
      System.err.println("Usage:  AddPerson ADDRESS_BOOK_FILE");
      System.exit(-1);
    }

    AddressBook.Builder addressBook = AddressBook.newBuilder();

    // Read the existing address book.
    try {
      addressBook.mergeFrom(new FileInputStream(args[0]));
    } catch (FileNotFoundException e) {
      System.out.println(args[0] + ": File not found.  Creating a new file.");
    }

    // Add an address.
    addressBook.addPerson(
      PromptForAddress(new BufferedReader(new InputStreamReader(System.in)),
                       System.out));

    // Write the new address book back to disk.
    FileOutputStream output = new FileOutputStream(args[0]);
    addressBook.build().writeTo(output);
    output.close();
  }
}
```

## メッセージの読み取り {#reading-a-message}

もちろん、アドレス帳は情報を取得できなければあまり役に立ちません！この例では、上記の例で作成されたファイルを読み取り、その中のすべての情報を出力します。

```java
import com.example.tutorial.protos.AddressBook;
import com.example.tutorial.protos.Person;
import java.io.FileInputStream;
import java.io.IOException;
import java.io.PrintStream;

class ListPeople {
  // Iterates though all people in the AddressBook and prints info about them.
  static void Print(AddressBook addressBook) {
    for (Person person: addressBook.getPeopleList()) {
      System.out.println("Person ID: " + person.getId());
      System.out.println("  Name: " + person.getName());
      if (person.hasEmail()) {
        System.out.println("  E-mail address: " + person.getEmail());
      }

      for (Person.PhoneNumber phoneNumber : person.getPhonesList()) {
        switch (phoneNumber.getType()) {
          case PHONE_TYPE_MOBILE:
            System.out.print("  Mobile phone #: ");
            break;
          case PHONE_TYPE_HOME:
            System.out.print("  Home phone #: ");
            break;
          case PHONE_TYPE_WORK:
            System.out.print("  Work phone #: ");
            break;
        }
        System.out.println(phoneNumber.getNumber());
      }
    }
  }

  // Main function:  Reads the entire address book from a file and prints all
  //   the information inside.
  public static void main(String[] args) throws Exception {
    if (args.length != 1) {
      System.err.println("Usage:  ListPeople ADDRESS_BOOK_FILE");
      System.exit(-1);
    }

    // Read the existing address book.
    AddressBook addressBook =
      AddressBook.parseFrom(new FileInputStream(args[0]));

    Print(addressBook);
  }
}
```

## Protocol Buffer の拡張 {#extending-a-protobuf}

プロトコルバッファを使用するコードをリリースした後、プロトコルバッファの定義を「改善」したくなることが遅かれ早かれあるでしょう。新しいバッファが後方互換性を持ち、古いバッファが前方互換性を持つことを望む場合（そしてほぼ間違いなくそうしたいと思うでしょう）、以下のルールに従う必要があります。新しいバージョンのプロトコルバッファでは：

- 既存のフィールドのタグ番号を変更してはいけません。
- 必須フィールドを追加または削除してはいけません。
- オプションまたは繰り返しフィールドを削除しても構いません。
- 新しいオプションまたは繰り返しフィールドを追加しても構いませんが、新しいタグ番号を使用する必要があります（つまり、このプロトコルバッファで使用されたことのないタグ番号、削除されたフィールドですら使用されていないタグ番号）。

（これらのルールには[いくつかの例外](/programming-guides/proto2#updating)がありますが、それらはほとんど使用されません。）

これらのルールに従うと、古いコードは新しいメッセージを問題なく読み取り、新しいフィールドを単に無視します。古いコードにとって、削除されたオプションフィールドは単にデフォルト値を持ち、削除された繰り返しフィールドは空になります。新しいコードも古いメッセージを透過的に読み取ります。ただし、新しいオプションフィールドは古いメッセージには存在しないため、`has_`で明示的に設定されているかどうかを確認するか、`.proto`ファイルで適切なデフォルト値を提供する必要があります（タグ番号の後に `[default = value]` を使用）。オプション要素のデフォルト値が指定されていない場合、型固有のデフォルト値が代わりに使用されます：文字列の場合、デフォルト値は空の文字列です。ブール値の場合、デフォルト値は false です。数値型の場合、デフォルト値はゼロです。また、新しい繰り返しフィールドを追加した場合、新しいコードはそれが空にされたか（新しいコードによって）、またはまったく設定されていないか（古いコードによって）を判断できません。それに対する `has_` フラグは存在しないためです。```

## 高度な使用法 {#advanced-usage}

プロトコルバッファには、アクセサーやシリアライゼーションを超える用途があります。
[Java API リファレンス](/reference/java/api-docs/index-all.html)
を探索して、それらを使用する方法を確認してください。

プロトコルメッセージクラスが提供する主要な機能の1つは*リフレクション*です。
メッセージのフィールドを反復処理し、特定のメッセージタイプに対してコードを記述せずに値を操作できます。
リフレクションを使用する非常に便利な方法の1つは、プロトコルメッセージを他のエンコーディング（XMLやJSONなど）に変換することです。
リフレクションのより高度な使用法としては、同じタイプの2つのメッセージの違いを見つけたり、「プロトコルメッセージ用の正規表現」のようなものを開発したりすることができます。
想像力を働かせれば、プロトコルバッファを最初に想定していたよりもはるかに広範囲の問題に適用することが可能です！

リフレクションは、[`Message`](/reference/java/api-docs/com/google/protobuf/Message.html)
および
[`Message.Builder`](/reference/java/api-docs/com/google/protobuf/Message.Builder.html)
インターフェースの一部として提供されています。
