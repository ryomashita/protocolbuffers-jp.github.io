+++
title = "プロトコルバッファの基礎: C++"
weight = 210
linkTitle = "C++"
description = "プロトコルバッファを扱うための基本的なC++プログラマー向けの紹介。"
type = "docs"
+++

このチュートリアルでは、プロトコルバッファを扱うための基本的なC++プログラマー向けの紹介を提供します。単純な例のアプリケーションを作成することで、次の方法を示します。

- `.proto` ファイルでメッセージ形式を定義する。
- プロトコルバッファコンパイラを使用する。
- C++プロトコルバッファAPIを使用してメッセージを書き込み、読み取る。

これはC++でプロトコルバッファを使用する包括的なガイドではありません。詳細な参照情報については、[Protocol Buffer Language Guide (proto2)](/programming-guides/proto2)、[Protocol Buffer Language Guide (proto3)](/programming-guides/proto3)、[C++ API Reference](/reference/cpp/api-docs)、[C++ Generated Code Guide](/reference/cpp/cpp-generated)、および[Encoding Reference](/programming-guides/encoding)を参照してください。

## 問題領域 {#problem-domain}

使用する例は非常にシンプルな「アドレス帳」アプリケーションで、人々の連絡先情報をファイルに読み書きできるものです。アドレス帳の各人物には、名前、ID、メールアドレス、連絡先電話番号があります。

このような構造化データをどのようにシリアライズおよび取得しますか？この問題を解決するためのいくつかの方法があります。

- 生のインメモリデータ構造をバイナリ形式で送信/保存できます。時間の経過とともに、これは受信/読み取りコードがまったく同じメモリレイアウト、エンディアンなどでコンパイルされている必要があるため、脆弱なアプローチです。また、ファイルが生の形式でデータを蓄積し、その形式に適したソフトウェアのコピーが広まると、形式を拡張するのが非常に難しくなります。
- データ項目を単一の文字列にエンコードするための特別な方法を考案することができます。たとえば、4つの整数を「12:3:-23:67」としてエンコードするなどです。これはシンプルで柔軟なアプローチですが、ワンオフのエンコーディングおよびパーシングコードの記述が必要であり、パーシングにはわずかなランタイムコストがかかります。これは非常に単純なデータをエンコードする場合に最適です。
- データをXMLにシリアライズします。このアプローチは非常に魅力的かもしれません。XMLは（ある程度）人間が読める形式であり、多くの言語にバインディングライブラリが存在します。これは他のアプリケーション/プロジェクトとデータを共有したい場合に適しています。ただし、XMLはスペースを多く取り、エンコード/デコードにはアプリケーションに大きなパフォーマンスペナルティを課す可能性があります。また、XML DOMツリーをナビゲートすることは、通常のクラスのフィールドをナビゲートするよりもかなり複雑です。

代わりにこれらのオプションを使用できます。プロトコルバッファは、この問題を正確に解決するための柔軟で効率的な自動化されたソリューションです。プロトコルバッファを使用すると、保存したいデータ構造の`.proto`説明を書きます。その後、プロトコルバッファコンパイラが、効率的なバイナリ形式でプロトコルバッファデータの自動エンコーディングとパースを実装するクラスを作成します。生成されたクラスは、プロトコルバッファを構成するフィールドのためのゲッターとセッターを提供し、プロトコルバッファを単位として読み書きの詳細を処理します。重要なのは、プロトコルバッファ形式が、コードが古い形式でエンコードされたデータをまだ読み取れるように、時間の経過とともに形式を拡張する考え方をサポートしていることです。

## 例コードの場所 {#example-code}

例コードは、ソースコードパッケージに含まれており、["examples"ディレクトリ](https://github.com/protocolbuffers/protobuf/tree/main/examples)の下にあります。

## プロトコル形式の定義 {#protocol-format}

アドレス帳アプリケーションを作成するには、`.proto`ファイルから始める必要があります。`.proto`ファイルの定義はシンプルです。シリアル化したいデータ構造ごとに*メッセージ*を追加し、メッセージ内の各フィールドに名前とタイプを指定します。以下は、メッセージ`addressbook.proto`を定義する`.proto`ファイルです。

```proto
syntax = "proto2";

package tutorial;

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

ご覧の通り、構文はC++やJavaに似ています。ファイルの各部分を見て、それぞれが何をするかを見ていきましょう。

`.proto`ファイルはパッケージ宣言で始まります。これは異なるプロジェクト間の名前の競合を防ぐのに役立ちます。C++では、生成されたクラスはパッケージ名に一致する名前空間に配置されます。

次に、メッセージの定義が続きます。メッセージは、型付きフィールドのセットを含む集約体です。`bool`、`int32`、`float`、`double`、`string`など、多くの標準的な単純データ型がフィールドタイプとして利用可能です。他のメッセージタイプをフィールドタイプとして使用することで、メッセージにさらなる構造を追加することもできます。上記の例では、`Person`メッセージに`PhoneNumber`メッセージが含まれ、`AddressBook`メッセージに`Person`メッセージが含まれています。他のメッセージ内にネストされたメッセージタイプを定義することもできます。`PhoneNumber`タイプが`Person`内で定義されていることがわかります。また、フィールドの1つに事前定義された値リストの1つを持たせたい場合は、`enum`タイプを定義できます。ここでは、電話番号が次の電話タイプのいずれかであることを指定したいとします：`PHONE_TYPE_MOBILE`、`PHONE_TYPE_HOME`、または`PHONE_TYPE_WORK`。

各要素の " = 1"、" = 2" マーカーは、バイナリエンコーディングでそのフィールドが使用する一意のフィールド番号を識別します。フィールド番号 1-15 は、より高い番号よりも1バイト少なくエンコードする必要があるため、よく使用されるまたは繰り返される要素にこれらの番号を使用することを最適化として選択できます。それにより、フィールド番号 16 以上は、あまり一般的に使用されないオプションの要素に残されます。繰り返しフィールド内の各要素は、フィールド番号を再エンコードする必要があるため、繰り返しフィールドは特にこの最適化の良い候補です。

各フィールドは、次の修飾子のいずれかで注釈付けする必要があります：

- `optional`：フィールドは設定されていてもされていなくてもかまいません。オプションのフィールド値が設定されていない場合、デフォルト値が使用されます。単純な型の場合、例として電話番号の `type` で行ったように、独自のデフォルト値を指定できます。それ以外の場合、システムのデフォルトが使用されます：数値型の場合はゼロ、文字列の場合は空の文字列、bool 型の場合は false です。埋め込みメッセージの場合、デフォルト値は常にメッセージの "デフォルトインスタンス" または "プロトタイプ" であり、そのフィールドが設定されていない状態です。明示的に設定されていないオプション（または必須）フィールドの値を取得するためにアクセサを呼び出すと、常にそのフィールドのデフォルト値が返されます。
- `repeated`：フィールドは任意の回数（0を含む）繰り返すことができます。繰り返し値の順序はプロトコルバッファ内で保持されます。繰り返しフィールドは、動的サイズの配列と考えてください。
- `required`：フィールドに値を指定する必要があります。そうでない場合、メッセージは "未初期化" と見なされます。`libprotobuf` がデバッグモードでコンパイルされている場合、未初期化のメッセージをシリアル化すると、アサーションの失敗が発生します。最適化されたビルドでは、チェックがスキップされ、メッセージは書き込まれます。ただし、未初期化のメッセージを解析すると常に失敗します（解析メソッドから `false` が返されます）。それ以外の場合、必須フィールドはオプションフィールドとまったく同じように動作します。

{{% alert title="重要" color="warning" %}} **Required Is Forever**
`required` フィールドをマークする際には非常に注意する必要があります。ある時点で必須フィールドの書き込みや送信を停止したい場合、そのフィールドをオプションフィールドに変更することは問題が発生します。古いリーダーは、このフィールドがないメッセージを不完全と見なし、それらを誤って拒否または破棄する可能性があります。代わりに、バッファに対してアプリケーション固有のカスタムバリデーションルーチンを記述することを検討すべきです。Google内では、`required` フィールドは強く非推奨です。proto2 構文で定義されたほとんどのメッセージは、`optional` と `repeated` のみを使用します（Proto3 では `required` フィールドはサポートされていません）。
{{% /alert %}}


`.proto` ファイルの作成に関する完全なガイドがここにあります -- すべての可能なフィールドタイプを含む -- [Protocol Buffer Language Guide](/programming-guides/proto2) で確認できます。
ただし、クラスの継承に類似した機能を探す必要はありません -- プロトコルバッファはそのような機能を提供していません。

## Protocol Buffers のコンパイル {#compiling-protocol-buffers}

`.proto` ファイルを持っている場合、次に行うべきことは、`AddressBook` (そして `Person` および `PhoneNumber`) メッセージを読み書きするために必要なクラスを生成することです。これを行うには、`.proto` ファイルに対してプロトコルバッファコンパイラ `protoc` を実行する必要があります:

1. コンパイラをインストールしていない場合は、[パッケージをダウンロード](/downloads) して README の手順に従ってください。

2. 今、コンパイラを実行し、ソースディレクトリ (アプリケーションのソースコードが存在する場所 -- 指定しない場合は現在のディレクトリが使用されます)、生成されたコードを配置する宛先ディレクトリ (`$SRC_DIR` と同じ場所がよく使用されます)、および `.proto` ファイルへのパスを指定します。この場合、以下のように...:

    ```shell
    protoc -I=$SRC_DIR --cpp_out=$DST_DIR $SRC_DIR/addressbook.proto
    ```

    C++ クラスを生成したい場合は、`--cpp_out` オプションを使用します -- 他のサポートされている言語に対しても同様のオプションが提供されています。

これにより、指定した宛先ディレクトリに次のファイルが生成されます:

-   `addressbook.pb.h`、生成されたクラスを宣言するヘッダー。
-   `addressbook.pb.cc`、クラスの実装を含むファイル。

## Protocol Buffer API {#protobuf-api}

生成されたコードの一部を見て、コンパイラがどのようなクラスや関数を作成したかを確認しましょう。`addressbook.pb.h` を見ると、`addressbook.proto` で指定した各メッセージに対してクラスが生成されていることがわかります。`Person` クラスを詳しく見ると、コンパイラが各フィールドに対してアクセサを生成していることがわかります。たとえば、`name`、`id`、`email`、`phones` フィールドについて、次のようなメソッドがあります:

```cpp
  // name
  inline bool has_name() const;
  inline void clear_name();
  inline const ::std::string& name() const;
  inline void set_name(const ::std::string& value);
  inline void set_name(const char* value);
  inline ::std::string* mutable_name();

  // id
  inline bool has_id() const;
  inline void clear_id();
  inline int32_t id() const;
  inline void set_id(int32_t value);

  // email
  inline bool has_email() const;
  inline void clear_email();
  inline const ::std::string& email() const;
  inline void set_email(const ::std::string& value);
  inline void set_email(const char* value);
  inline ::std::string* mutable_email();

  // phones
  inline int phones_size() const;
  inline void clear_phones();
  inline const ::google::protobuf::RepeatedPtrField< ::tutorial::Person_PhoneNumber >& phones() const;
  inline ::google::protobuf::RepeatedPtrField< ::tutorial::Person_PhoneNumber >* mutable_phones();
  inline const ::tutorial::Person_PhoneNumber& phones(int index) const;
  inline ::tutorial::Person_PhoneNumber* mutable_phones(int index);
  inline ::tutorial::Person_PhoneNumber* add_phones();
```

ゲッターはフィールド名とまったく同じ名前で、小文字で始まり、セッターメソッドは `set_` で始まります。また、各単数形 (必須またはオプション) フィールドに対して `has_` メソッドがあり、そのフィールドが設定されている場合に true を返します。最後に、各フィールドには `clear_` メソッドがあり、フィールドを空の状態に戻します。

数値の `id` フィールドは、上記で説明した基本アクセサセットのみを持っていますが、
`name` と `email` フィールドは文字列なので、いくつかの追加メソッドがあります。これには、文字列への直接ポインタを取得できる `mutable_` ゲッターと追加のセッターが含まれます。`email` がすでに設定されていなくても、`mutable_email()` を呼び出すことができます。自動的に空の文字列に初期化されます。この例で繰り返しメッセージフィールドがある場合、`mutable_` メソッドはあるが `set_` メソッドはないことに注意してください。

繰り返しフィールドにはいくつかの特別なメソッドもあります。繰り返しフィールド `phones` のメソッドを見ると、次のことができることがわかります。

- 繰り返しフィールドの `_size`（つまり、この `Person` に関連付けられた電話番号の数）を確認できます。
- インデックスを使用して指定された電話番号を取得できます。
- 指定されたインデックスの既存の電話番号を更新できます。
- メッセージに別の電話番号を追加し、その後編集できます（繰り返しスカラータイプには、新しい値を渡すだけの `add_` があります）。

特定のフィールド定義に対してプロトコルコンパイラが生成するメンバーについての詳細情報については、[C++ 生成コードリファレンス](/reference/cpp/cpp-generated)を参照してください。

### 列挙型とネストされたクラス {#enums-nested-classes}

生成されたコードには、`.proto` 列挙型に対応する `PhoneType` 列挙型が含まれています。この型は `Person::PhoneType` として参照し、その値は `Person::PHONE_TYPE_MOBILE`、`Person::PHONE_TYPE_HOME`、`Person::PHONE_TYPE_WORK` として参照できます（実装の詳細は少し複雑ですが、列挙型を使用するためにそれらを理解する必要はありません）。

コンパイラは、`Person::PhoneNumber` というネストされたクラスも生成しました。コードを見ると、「実際の」クラスは実際には `Person_PhoneNumber` と呼ばれていますが、`Person` 内で定義された typedef により、それをネストされたクラスのように扱うことができます。これが違いを生じる唯一のケースは、別のファイルでクラスを前方宣言したい場合です。C++ ではネストされた型を前方宣言することはできませんが、`Person_PhoneNumber` を前方宣言することはできます。

### 標準メッセージメソッド {#standard-message-methods}

各メッセージクラスには、他のメソッドも含まれており、次のようなメッセージ全体をチェックしたり操作したりするためのメソッドがあります。

-   `bool IsInitialized() const;`: 必須フィールドがすべて設定されているかどうかをチェックします。
-   `string DebugString() const;`: メッセージの人間が読める表現を返します。デバッグに特に便利です。
-   `void CopyFrom(const Person& from);`: 指定されたメッセージの値でメッセージを上書きします。
-   `void Clear();`: すべての要素を空の状態にクリアします。

これらと次のセクションで説明されているI/Oメソッドは、すべてのC++プロトコルバッファクラスで共有される`Message`インターフェースを実装しています。詳細については、[`Message`の完全なAPIドキュメント](/reference/cpp/api-docs/google.protobuf.message#Message)を参照してください。

### パースとシリアライゼーション {#parsing-serialization}

最後に、各プロトコルバッファクラスには、プロトコルバッファの[バイナリ形式](/programming-guides/encoding)を使用して、選択したタイプのメッセージを書き込み、読み取るためのメソッドがあります。これには次のものが含まれます。

-   `bool SerializeToString(string* output) const;`: メッセージをシリアライズして、バイトを指定された文字列に格納します。バイトはバイナリであり、テキストではありません。`string`クラスは便利なコンテナとしてのみ使用されます。
-   `bool ParseFromString(const string& data);`: 指定された文字列からメッセージを解析します。
-   `bool SerializeToOstream(ostream* output) const;`: メッセージを指定されたC++ `ostream`に書き込みます。
-   `bool ParseFromIstream(istream* input);`: 指定されたC++ `istream`からメッセージを解析します。

これらは、パースとシリアライゼーションのために提供されているオプションの一部です。詳細については、再度、[`Message` APIリファレンス](/reference/cpp/api-docs/google.protobuf.message#Message)を参照してください。

{{% alert title="重要" color="warning" %}} **プロトコルバッファとオブジェクト指向設計**
プロトコルバッファクラスは基本的にデータホルダー（Cの構造体のようなもの）であり、追加の機能を提供しません。生成されたクラスにより豊かな動作を追加したい場合、最良の方法は、生成されたプロトコルバッファクラスをアプリケーション固有のクラスでラップすることです。プロトコルバッファをラップすることは、`.proto`ファイルの設計を制御できない場合（別のプロジェクトから再利用している場合など）にも良いアイデアです。その場合、ラッパークラスを使用して、アプリケーションの独自の環境に適したインターフェースを作成できます。データやメソッドを非表示にし、便利な機能を公開するなどです。**生成されたクラスに動作を追加することは決してありません**。これは内部メカニズムを壊し、オブジェクト指向の実践としてもよくありません。{{% /alert %}}

## メッセージの書き込み {#writing-a-message}

さて、プロトコルバッファクラスを使用してみましょう。アドレス帳アプリケーションができるようにしたい最初のことは、個人の詳細をアドレス帳ファイルに書き込むことです。これを行うには、プロトコルバッファクラスのインスタンスを作成し、値を設定してから、それらを出力ストリームに書き込む必要があります。

以下は、ファイルから `AddressBook` を読み込み、ユーザーの入力に基づいて新しい `Person` を1つ追加し、新しい `AddressBook` を再度ファイルに書き出すプログラムです。プロトコルコンパイラによって生成されたコードを直接呼び出す部分が強調表示されています。

```cpp
#include <iostream>
#include <fstream>
#include <string>
#include "addressbook.pb.h"
using namespace std;

// This function fills in a Person message based on user input.
void PromptForAddress(tutorial::Person* person) {
  cout << "Enter person ID number: ";
  int id;
  cin >> id;
  person->set_id(id);
  cin.ignore(256, '\n');

  cout << "Enter name: ";
  getline(cin, *person->mutable_name());

  cout << "Enter email address (blank for none): ";
  string email;
  getline(cin, email);
  if (!email.empty()) {
    person->set_email(email);
  }

  while (true) {
    cout << "Enter a phone number (or leave blank to finish): ";
    string number;
    getline(cin, number);
    if (number.empty()) {
      break;
    }

    tutorial::Person::PhoneNumber* phone_number = person->add_phones();
    phone_number->set_number(number);

    cout << "Is this a mobile, home, or work phone? ";
    string type;
    getline(cin, type);
    if (type == "mobile") {
      phone_number->set_type(tutorial::Person::PHONE_TYPE_MOBILE);
    } else if (type == "home") {
      phone_number->set_type(tutorial::Person::PHONE_TYPE_HOME);
    } else if (type == "work") {
      phone_number->set_type(tutorial::Person::PHONE_TYPE_WORK);
    } else {
      cout << "Unknown phone type.  Using default." << endl;
    }
  }
}

// Main function:  Reads the entire address book from a file,
//   adds one person based on user input, then writes it back out to the same
//   file.
int main(int argc, char* argv[]) {
  // Verify that the version of the library that we linked against is
  // compatible with the version of the headers we compiled against.
  GOOGLE_PROTOBUF_VERIFY_VERSION;

  if (argc != 2) {
    cerr << "Usage:  " << argv[0] << " ADDRESS_BOOK_FILE" << endl;
    return -1;
  }

  tutorial::AddressBook address_book;

  {
    // Read the existing address book.
    fstream input(argv[1], ios::in | ios::binary);
    if (!input) {
      cout << argv[1] << ": File not found.  Creating a new file." << endl;
    } else if (!address_book.ParseFromIstream(&input)) {
      cerr << "Failed to parse address book." << endl;
      return -1;
    }
  }

  // Add an address.
  PromptForAddress(address_book.add_people());

  {
    // Write the new address book back to disk.
    fstream output(argv[1], ios::out | ios::trunc | ios::binary);
    if (!address_book.SerializeToOstream(&output)) {
      cerr << "Failed to write address book." << endl;
      return -1;
    }
  }

  // Optional:  Delete all global objects allocated by libprotobuf.
  google::protobuf::ShutdownProtobufLibrary();

  return 0;
}
```

`GOOGLE_PROTOBUF_VERIFY_VERSION` マクロに注意してください。C++ Protocol Buffer ライブラリを使用する前にこのマクロを実行するのは良い習慣ですが、厳密に必要というわけではありません。これにより、コンパイル時に使用したヘッダのバージョンと互換性のないライブラリに誤ってリンクしていないかを確認します。バージョンの不一致が検出されると、プログラムは中止します。なお、すべての `.pb.cc` ファイルは起動時にこのマクロを自動的に呼び出します。

また、プログラムの最後に `ShutdownProtobufLibrary()` を呼び出すことにも注意してください。これは、Protocol Buffer ライブラリによって割り当てられたグローバルオブジェクトを削除するだけです。これはほとんどのプログラムには不要です。なぜなら、プロセスは終了するだけであり、OSがそのメモリを回収するからです。ただし、すべてのオブジェクトを解放する必要があるメモリリークチェッカーを使用している場合や、単一のプロセスで複数回ロードおよびアンロードされる可能性があるライブラリを書いている場合は、Protocol Buffers にすべてをクリーンアップさせたいかもしれません。

## メッセージの読み取り {#reading-a-message}

もちろん、アドレス帳から情報を取得できない場合、アドレス帳はあまり役に立ちません！この例では、上記の例で作成されたファイルを読み込み、その中のすべての情報を表示します。

```cpp
#include <iostream>
#include <fstream>
#include <string>
#include "addressbook.pb.h"
using namespace std;

// Iterates though all people in the AddressBook and prints info about them.
void ListPeople(const tutorial::AddressBook& address_book) {
  for (int i = 0; i < address_book.people_size(); i++) {
    const tutorial::Person& person = address_book.people(i);

    cout << "Person ID: " << person.id() << endl;
    cout << "  Name: " << person.name() << endl;
    if (person.has_email()) {
      cout << "  E-mail address: " << person.email() << endl;
    }

    for (int j = 0; j < person.phones_size(); j++) {
      const tutorial::Person::PhoneNumber& phone_number = person.phones(j);

      switch (phone_number.type()) {
        case tutorial::Person::PHONE_TYPE_MOBILE:
          cout << "  Mobile phone #: ";
          break;
        case tutorial::Person::PHONE_TYPE_HOME:
          cout << "  Home phone #: ";
          break;
        case tutorial::Person::PHONE_TYPE_WORK:
          cout << "  Work phone #: ";
          break;
      }
      cout << phone_number.number() << endl;
    }
  }
}

// Main function:  Reads the entire address book from a file and prints all
//   the information inside.
int main(int argc, char* argv[]) {
  // Verify that the version of the library that we linked against is
  // compatible with the version of the headers we compiled against.
  GOOGLE_PROTOBUF_VERIFY_VERSION;

  if (argc != 2) {
    cerr << "Usage:  " << argv[0] << " ADDRESS_BOOK_FILE" << endl;
    return -1;
  }

  tutorial::AddressBook address_book;

  {
    // Read the existing address book.
    fstream input(argv[1], ios::in | ios::binary);
    if (!address_book.ParseFromIstream(&input)) {
      cerr << "Failed to parse address book." << endl;
      return -1;
    }
  }

  ListPeople(address_book);

  // Optional:  Delete all global objects allocated by libprotobuf.
  google::protobuf::ShutdownProtobufLibrary();

  return 0;
}
```

## プロトコルバッファの拡張 {#extending-a-protobuf}

プロトコルバッファを使用するコードをリリースした後、プロトコルバッファの定義を「改善」したくなることが遅かれ早かれあるでしょう。新しいバッファが後方互換性を持ち、古いバッファが前方互換性を持つようにしたい場合 -- そしておそらくほとんどの場合そうしたいでしょう -- そのためにはいくつかのルールに従う必要があります。新しいバージョンのプロトコルバッファでは、

-   既存のフィールドのフィールド番号を変更してはいけません。
-   必須フィールドを追加したり削除してはいけません。
-   オプションまたは繰り返しフィールドを削除しても構いません。
-   新しいオプションまたは繰り返しフィールドを追加しても構いませんが、新しいフィールド番号を使用する必要があります（つまり、このプロトコルバッファで使用されたことのないフィールド番号を使用してください）。

（[いくつかの例外](/programming-guides/proto2#updating)もありますが、それらはほとんど使用されません。）

これらのルールに従うと、古いコードは新しいメッセージを問題なく読み取り、単に新しいフィールドを無視します。古いコードにとって、削除されたオプションフィールドは単にデフォルト値を持ち、削除された繰り返しフィールドは空になります。新しいコードも古いメッセージを透過的に読み取ります。ただし、新しいオプションフィールドは古いメッセージに存在しないため、`has_`で明示的に設定されているかどうかを確認するか、フィールド番号の後に`[default = value]`を使用して`.proto`ファイルに適切なデフォルト値を提供する必要があります。オプション要素のデフォルト値が指定されていない場合、タイプ固有のデフォルト値が代わりに使用されます：文字列の場合、デフォルト値は空の文字列です。ブール値の場合、デフォルト値はfalseです。数値型の場合、デフォルト値はゼロです。また、新しい繰り返しフィールドを追加した場合、新しいコードはそれが空になっている（新しいコードによって）か、まったく設定されていない（古いコードによって）かを判断できません。それに`has_`フラグがないためです。

## 最適化のヒント {#optimization}

C++ Protocol Buffers ライブラリは非常に高度に最適化されています。ただし、適切な使用方法によりさらなるパフォーマンス向上が可能です。ライブラリから最後の一滴の速度を搾り出すためのいくつかのヒントを以下に示します：

-   可能な限りメッセージオブジェクトを再利用してください。メッセージは、クリアされていても再利用のために割り当てたメモリを保持しようとします。したがって、同じタイプで似た構造を持つ多くのメッセージを連続して処理する場合は、毎回同じメッセージオブジェクトを再利用してメモリアロケーターの負荷を軽減するのが良いアイデアです。ただし、オブジェクトは時間の経過とともに肥大化する可能性があります、特にメッセージの「形状」が異なる場合や通常よりもはるかに大きなメッセージを時折構築する場合。メッセージオブジェクトのサイズを`SpaceUsed`メソッドを呼び出して監視し、あまりに大きくなったら削除してください。
-   システムのメモリアロケーターは、複数のスレッドから多くの小さなオブジェクトを割り当てるために最適化されていない場合があります。代わりに[Google's TCMalloc](https://github.com/google/tcmalloc)を使用してみてください。

## 高度な使用法 {#advanced-usage}

プロトコルバッファには、シンプルなアクセサやシリアライゼーションを超える用途があります。[C++ API リファレンス](/reference/cpp)を探索して、それらを使用する方法を確認してください。

プロトコルメッセージクラスが提供する主要な機能の1つは*リフレクション*です。メッセージのフィールドを反復処理し、特定のメッセージタイプに対してコードを記述せずに値を操作できます。リフレクションを使用する非常に便利な方法の1つは、プロトコルメッセージを他のエンコーディング（例：XMLやJSON）に変換することです。リフレクションのより高度な使用法としては、同じタイプの2つのメッセージの違いを見つけたり、「プロトコルメッセージ用の正規表現」のようなものを開発したりすることが考えられます。想像力を働かせれば、Protocol Buffers を最初に想定していたよりもはるかに広範囲の問題に適用することが可能です！

リフレクションは、[`Message::Reflection` インターフェース](/reference/cpp/api-docs/google.protobuf.message#Reflection)によって提供されます。
