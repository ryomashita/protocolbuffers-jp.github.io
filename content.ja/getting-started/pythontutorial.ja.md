+++
title = "プロトコルバッファの基礎: Python"
weight = 270
linkTitle = "Python"
description = "プロトコルバッファを扱うための基本的なPythonプログラマー向けの紹介。"
type = "docs"
+++

このチュートリアルでは、プロトコルバッファを扱うための基本的なPythonプログラマー向けの紹介を提供します。単純な例のアプリケーションを作成することで、次の方法を示します。

- `.proto` ファイルでメッセージ形式を定義する。
- プロトコルバッファコンパイラを使用する。
- PythonプロトコルバッファAPIを使用してメッセージを書き込み、読み取る。

これはPythonでプロトコルバッファを使用する包括的なガイドではありません。詳細なリファレンス情報については、[Protocol Buffer Language Guide (proto2)](/programming-guides/proto2)、[Protocol Buffer Language Guide (proto3)](/programming-guides/proto3)、[Python API Reference](https://googleapis.dev/python/protobuf/latest/)、[Python Generated Code Guide](/reference/python/python-generated)、および[Encoding Reference](/programming-guides/encoding)を参照してください。

## 問題領域 {#problem-domain}

使用する例は非常にシンプルな「アドレス帳」アプリケーションで、人々の連絡先詳細をファイルに読み書きできるものです。アドレス帳の各人物には、名前、ID、メールアドレス、連絡先電話番号があります。

このような構造化データをシリアライズおよび取得する方法はどうすればよいでしょうか？この問題を解決するためのいくつかの方法があります。

- Pythonのピクリングを使用する。これは言語に組み込まれているため、デフォルトのアプローチですが、スキーマの進化に対応できず、また、C++やJavaで書かれたアプリケーションとデータを共有する必要がある場合にはあまり適していません。
- データ項目を単一の文字列にエンコードするための特別な方法を考案することができます。たとえば、4つの整数を「12:3:-23:67」としてエンコードするなどです。これはシンプルで柔軟なアプローチですが、ワンオフのエンコーディングおよびパーシングコードの記述が必要であり、パーシングにはわずかなランタイムコストがかかります。これは非常に単純なデータをエンコードする場合に最適です。
- データをXMLにシリアライズする。このアプローチは非常に魅力的であり、XMLは（ある程度）人間が読める形式であり、多くの言語にバインディングライブラリが存在するためです。他のアプリケーション/プロジェクトとデータを共有したい場合には適しています。ただし、XMLはスペースを多く必要とし、エンコード/デコードにはアプリケーションに大きなパフォーマンスペナルティを課す可能性があります。また、XML DOMツリーをナビゲートすることは、通常のクラスのフィールドをナビゲートするよりもかなり複雑です。

代わりに、これらのオプションの代わりに、プロトコルバッファを使用できます。プロトコルバッファは、この問題を正確に解決する柔軟で効率的な自動化されたソリューションです。プロトコルバッファを使用すると、保存したいデータ構造の`.proto`記述を作成します。その後、プロトコルバッファコンパイラは、プロトコルバッファデータの効率的なバイナリ形式での自動エンコーディングとパーシングを実装するクラスを作成します。生成されたクラスは、プロトコルバッファを構成するフィールドのためのゲッターとセッターを提供し、プロトコルバッファを単位として読み書きする詳細を処理します。重要なことは、プロトコルバッファ形式が、コードが古い形式でエンコードされたデータをまだ読み取れるように、時間の経過とともに形式を拡張する考え方をサポートしていることです。

## 例コードの場所 {#example-code}

例コードは、ソースコードパッケージに含まれており、"examples"ディレクトリの下にあります。[こちらからダウンロードしてください。](/downloads)

## プロトコル形式の定義 {#protocol-format}

アドレス帳アプリケーションを作成するには、`.proto`ファイルから始める必要があります。`.proto`ファイルの定義はシンプルです。シリアライズしたいデータ構造ごとに*メッセージ*を追加し、メッセージ内の各フィールドに名前とタイプを指定します。以下は、メッセージ`addressbook.proto`を定義する`.proto`ファイルです。

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

C++やJavaに似た構文であることがわかります。ファイルの各部分を見て、それぞれが何をしているかを見ていきましょう。

`.proto`ファイルはパッケージ宣言で始まります。これは異なるプロジェクト間の名前の衝突を防ぐのに役立ちます。Pythonでは、パッケージは通常ディレクトリ構造によって決定されるため、`.proto`ファイルで定義する`package`は生成されたコードに影響を与えません。ただし、Python以外の言語やプロトコルバッファの名前空間内での名前の衝突を避けるために、パッケージを宣言する必要があります。

次に、メッセージの定義があります。メッセージは、型付きフィールドのセットを含む集約体です。`bool`、`int32`、`float`、`double`、`string`など、多くの標準的な単純データ型がフィールド型として利用可能です。他のメッセージ型をフィールド型として使用することで、メッセージにさらなる構造を追加することもできます。上記の例では、`Person`メッセージに`PhoneNumber`メッセージが含まれ、`AddressBook`メッセージに`Person`メッセージが含まれています。他のメッセージ内にネストされたメッセージ型を定義することもできます。`PhoneNumber`タイプが`Person`内で定義されていることがわかります。また、フィールドの1つに事前定義された値リストの1つを持つようにしたい場合は、`enum`型を定義することもできます。ここでは、電話番号が次の電話タイプのいずれかであることを指定したいとします：`PHONE_TYPE_MOBILE`、`PHONE_TYPE_HOME`、または`PHONE_TYPE_WORK`。

各要素の「= 1」、「= 2」のマーカーは、フィールドがバイナリエンコーディングで使用する一意の「タグ」を識別します。タグ番号1-15は、高い番号よりも1バイト少なくエンコードできるため、よく使用されるまたは繰り返される要素にこれらのタグを使用することを最適化として選択できます。タグ16以上は、あまり一般的に使用されないオプションの要素に残します。繰り返しフィールド内の各要素には、タグ番号を再エンコードする必要があるため、繰り返しフィールドはこの最適化の特に適した候補です。

各フィールドは、次の修飾子のいずれかで注釈付けする必要があります：

- `optional`：フィールドは設定されていてもされていなくてもかまいません。オプションのフィールド値が設定されていない場合、デフォルト値が使用されます。単純な型の場合、例として電話番号の`type`で行ったように独自のデフォルト値を指定できます。それ以外の場合、システムのデフォルト値が使用されます：数値型の場合はゼロ、文字列の場合は空の文字列、ブール型の場合はfalse。埋め込みメッセージの場合、デフォルト値は常にメッセージの「デフォルトインスタンス」または「プロトタイプ」であり、そのフィールドは設定されていません。明示的に設定されていないオプション（または必須）フィールドの値を取得するためにアクセサを呼び出すと、常にそのフィールドのデフォルト値が返されます。
- `repeated`：フィールドは任意の回数（ゼロを含む）繰り返すことができます。繰り返し値の順序はプロトコルバッファで保持されます。繰り返しフィールドは、動的サイズの配列と考えてください。
- `required`：フィールドに値を提供する必要があります。そうでない場合、メッセージは「未初期化」と見なされます。未初期化のメッセージをシリアライズすると例外が発生します。未初期化のメッセージを解析すると失敗します。それ以外の場合、必須フィールドはオプションフィールドとまったく同じように動作します。

{{% alert title="重要" color="warning" %}} **必須は永遠**
`required`フィールドとしてフィールドをマークすることについては非常に注意してください。ある時点で必須フィールドの書き込みや送信を停止したい場合、そのフィールドをオプションフィールドに変更することは問題が発生します。古いリーダーは、このフィールドがないメッセージを不完全と見なし、それらを意図せず拒否または破棄する可能性があります。バッファに対してアプリケーション固有のカスタムバリデーションルーチンを記述することを検討すべきです。Google内では、`required`フィールドは強く推奨されていません。proto2構文で定義されたほとんどのメッセージは、`optional`および`repeated`のみを使用します。（Proto3は`required`フィールドをサポートしていません。）
{{% /alert %}}

`.proto` ファイルの作成に関する完全なガイドを見つけることができます。-- すべての可能なフィールドタイプを含む -- [Protocol Buffer Language Guide](/programming-guides/proto2) で。ただし、クラスの継承に類似した機能を探すことはしないでください -- プロトコルバッファはそのような機能を提供しません。

## Protocol Buffers のコンパイル {#compiling-protocol-buffers}

`.proto` ファイルを持っている場合、次に行う必要があるのは、`AddressBook` (およびしたがって `Person` および `PhoneNumber`) メッセージを読み書きするために必要なクラスを生成することです。これを行うには、`.proto` ファイルに対してプロトコルバッファコンパイラ `protoc` を実行する必要があります。

1. コンパイラをインストールしていない場合は、[パッケージをダウンロード](/downloads) して README の手順に従ってください。

2. 今、コンパイラを実行し、ソースディレクトリ (アプリケーションのソースコードが存在する場所 -- 指定しない場合は現在のディレクトリが使用されます)、生成されたコードが配置される宛先ディレクトリ (`$SRC_DIR` と同じ場所がよく使用されます)、および `.proto` ファイルへのパスを指定します。この場合、以下のように...:

    ```shell
    protoc -I=$SRC_DIR --python_out=$DST_DIR $SRC_DIR/addressbook.proto
    ```

    Python クラスを生成したいので、`--python_out` オプションを使用します -- 他のサポートされている言語に対しても同様のオプションが提供されています。

    Protoc は `--pyi_out` を使用して Python スタブ (`.pyi`) も生成できます。

これにより、指定した宛先ディレクトリに `addressbook_pb2.py` (または `addressbook_pb2.pyi`) が生成されます。

## Protocol Buffer API {#protobuf-api}

Java および C++ プロトコルバッファコードを生成する際とは異なり、Python プロトコルバッファコンパイラは直接データアクセスコードを生成しません。代わりに (あなたが `addressbook_pb2.py` を見ればわかるように) 、すべてのメッセージ、列挙型、フィールドに対する特別な記述子と、各メッセージタイプごとに1つの謎めいた空のクラスが生成されます:

```python
class Person(message.Message):
  __metaclass__ = reflection.GeneratedProtocolMessageType

  class PhoneNumber(message.Message):
    __metaclass__ = reflection.GeneratedProtocolMessageType
    DESCRIPTOR = _PERSON_PHONENUMBER
  DESCRIPTOR = _PERSON

class AddressBook(message.Message):
  __metaclass__ = reflection.GeneratedProtocolMessageType
  DESCRIPTOR = _ADDRESSBOOK
```

各クラスの重要な行は `__metaclass__ = reflection.GeneratedProtocolMessageType` です。Python メタクラスの動作の詳細はこのチュートリアルの範囲外ですが、それらはクラスを作成するためのテンプレートのようなものと考えることができます。ロード時に、`GeneratedProtocolMessageType` メタクラスは指定された記述子を使用して、各メッセージタイプと対応するクラスに必要なすべての Python メソッドを作成し、それらを追加します。その後、コード内で完全に満たされたクラスを使用できます。

最終的な効果は、`Person` クラスを `Message` ベースクラスの各フィールドを通常のフィールドとして定義したかのように使用できることです。たとえば、次のように書くことができます：

```python
import addressbook_pb2
person = addressbook_pb2.Person()
person.id = 1234
person.name = "John Doe"
person.email = "jdoe@example.com"
phone = person.phones.add()
phone.number = "555-4321"
phone.type = addressbook_pb2.Person.PHONE_TYPE_HOME
```

これらの代入は、汎用の Python オブジェクトに任意の新しいフィールドを追加するだけではありません。`.proto` ファイルで定義されていないフィールドに値を割り当てようとすると、`AttributeError` が発生します。型が違う値をフィールドに割り当てると、`TypeError` が発生します。また、フィールドに設定される前にその値を読み取ると、デフォルト値が返されます。

```python
person.no_such_field = 1  # AttributeError が発生します
person.id = "1234"        # TypeError が発生します
```

特定のフィールド定義に対してプロトコルコンパイラが生成するメンバーについての詳細情報については、[Python 生成コードリファレンス](/reference/python/python-generated)を参照してください。

### 列挙型 {#enums}

列挙型は、メタクラスによって整数値を持つ一連のシンボリック定数に展開されます。たとえば、定数 `addressbook_pb2.Person.PhoneType.PHONE_TYPE_WORK` は値 2 を持ちます。

### 標準メッセージメソッド {#standard-message-methods}

各メッセージクラスには、すべての必須フィールドが設定されているかどうかを確認したり、メッセージ全体を操作したりするための他の多くのメソッドも含まれています：

-   `IsInitialized()`: すべての必須フィールドが設定されているかどうかを確認します。
-   `__str__()`: メッセージの人間が読める表現を返します。デバッグに特に便利です。（通常は `str(message)` または `print message` として呼び出されます。）
-   `CopyFrom(other_msg)`: 指定されたメッセージの値でメッセージを上書きします。
-   `Clear()`: すべての要素を空の状態にクリアします。

これらのメソッドは `Message` インターフェースを実装しています。詳細については、[`Message` の完全な API ドキュメント](https://googleapis.dev/python/protobuf/latest/google/protobuf/message.html#google.protobuf.message.Message)を参照してください。

### パースとシリアライズ {#parsing-serialization}

最後に、各プロトコルバッファクラスには、プロトコルバッファの[バイナリ形式](/programming-guides/encoding)を使用して選択したタイプのメッセージを書き込み、読み取るためのメソッドがあります。これには次のものが含まれます：

-   `SerializeToString()`: メッセージをシリアライズして文字列として返します。
    バイトはバイナリであり、テキストではないことに注意してください。`str` 型は便利なコンテナとしてのみ使用されます。
-   `ParseFromString(data)`: 指定された文字列からメッセージをパースします。

これらは、パースとシリアライズのために提供されるオプションの一部です。
詳細なリストについては、
[`Message` API リファレンス](https://googleapis.dev/python/protobuf/latest/google/protobuf/message.html#google.protobuf.message.Message)
を参照してください。

{{% alert title="重要" color="warning" %}} **Protocol Buffers とオブジェクト指向設計**
プロトコルバッファクラスは基本的にデータホルダー（C 言語の構造体のようなもの）であり、
追加の機能を提供しません。オブジェクトモデルにおいて一級市民として適していません。
生成されたクラスにより豊かな動作を追加したい場合、
最良の方法は生成されたプロトコルバッファクラスをアプリケーション固有のクラスでラップすることです。
プロトコルバッファをラップすることは、`.proto` ファイルの設計を制御できない場合にも良いアイデアです
（たとえば、別のプロジェクトから再利用している場合）。その場合、ラッパークラスを使用して、
アプリケーションの独自の環境に適したインターフェースを作成できます。
データやメソッドを非表示にし、便利な機能を公開するなどです。
**生成されたクラスに動作を追加することは決してありません**。これは内部メカニズムを壊し、
オブジェクト指向の実践としても良くありません。{{% /alert %}}

## メッセージの書き込み {#writing-a-message}

さて、プロトコルバッファクラスを使用してみましょう。アドレス帳アプリケーションができるようにしたい最初のことは、
個人の詳細をアドレス帳ファイルに書き込むことです。これを行うには、
プロトコルバッファクラスのインスタンスを作成し、値を設定してから、それらを出力ストリームに書き込む必要があります。

以下は、ファイルから `AddressBook` を読み込み、ユーザー入力に基づいて新しい `Person` を1つ追加し、
新しい `AddressBook` を再度ファイルに書き出すプログラムです。プロトコルコンパイラによって生成されたコードを直接呼び出す部分が強調表示されています。

```python
#!/usr/bin/env python3

import addressbook_pb2
import sys

# This function fills in a Person message based on user input.
def PromptForAddress(person):
  person.id = int(input("Enter person ID number: "))
  person.name = input("Enter name: ")

  email = input("Enter email address (blank for none): ")
  if email != "":
    person.email = email

  while True:
    number = input("Enter a phone number (or leave blank to finish): ")
    if number == "":
      break

    phone_number = person.phones.add()
    phone_number.number = number

    phone_type = input("Is this a mobile, home, or work phone? ")
    if phone_type == "mobile":
      phone_number.type = addressbook_pb2.Person.PhoneType.PHONE_TYPE_MOBILE
    elif phone_type == "home":
      phone_number.type = addressbook_pb2.Person.PhoneType.PHONE_TYPE_HOME
    elif phone_type == "work":
      phone_number.type = addressbook_pb2.Person.PhoneType.PHONE_TYPE_WORK
    else:
      print("Unknown phone type; leaving as default value.")

# Main procedure:  Reads the entire address book from a file,
#   adds one person based on user input, then writes it back out to the same
#   file.
if len(sys.argv) != 2:
  print("Usage:", sys.argv[0], "ADDRESS_BOOK_FILE")
  sys.exit(-1)

address_book = addressbook_pb2.AddressBook()

# Read the existing address book.
try:
  with open(sys.argv[1], "rb") as f:
    address_book.ParseFromString(f.read())
except IOError:
  print(sys.argv[1] + ": Could not open file.  Creating a new one.")

# Add an address.
PromptForAddress(address_book.people.add())

# Write the new address book back to disk.
with open(sys.argv[1], "wb") as f:
  f.write(address_book.SerializeToString())
```

## メッセージの読み取り {#reading-a-message}

もちろん、アドレス帳は情報を取得できない場合はあまり役に立ちません！この例では、上記の例で作成されたファイルを読み取り、その中のすべての情報を出力します。

```python
#!/usr/bin/env python3

import addressbook_pb2
import sys

# Iterates though all people in the AddressBook and prints info about them.
def ListPeople(address_book):
  for person in address_book.people:
    print("Person ID:", person.id)
    print("  Name:", person.name)
    if person.HasField('email'):
      print("  E-mail address:", person.email)

    for phone_number in person.phones:
      if phone_number.type == addressbook_pb2.Person.PhoneType.PHONE_TYPE_MOBILE:
        print("  Mobile phone #: ", end="")
      elif phone_number.type == addressbook_pb2.Person.PhoneType.PHONE_TYPE_HOME:
        print("  Home phone #: ", end="")
      elif phone_number.type == addressbook_pb2.Person.PhoneType.PHONE_TYPE_WORK:
        print("  Work phone #: ", end="")
      print(phone_number.number)

# Main procedure:  Reads the entire address book from a file and prints all
#   the information inside.
if len(sys.argv) != 2:
  print("Usage:", sys.argv[0], "ADDRESS_BOOK_FILE")
  sys.exit(-1)

address_book = addressbook_pb2.AddressBook()

# Read the existing address book.
with open(sys.argv[1], "rb") as f:
  address_book.ParseFromString(f.read())

ListPeople(address_book)
```

## Protocol Buffer の拡張 {#extending-a-protobuf}

プロトコルバッファを使用するコードをリリースした後、プロトコルバッファの定義を「改善」したくなることは避けられません。新しいバッファが後方互換性を持ち、古いバッファが前方互換性を持つことを望む場合（ほぼ間違いなくそうしたいと思うでしょう）、以下のルールに従う必要があります。新しいバージョンのプロトコルバッファでは：

- 既存のフィールドのタグ番号を変更してはいけません。
- 必須フィールドを追加または削除してはいけません。
- オプションまたは繰り返しフィールドを削除しても構いません。
- 新しいオプションまたは繰り返しフィールドを追加しても構いませんが、新しいタグ番号を使用する必要があります（つまり、このプロトコルバッファで使用されたことのないタグ番号を使用する必要があります）。

（これらのルールには[いくつかの例外](/programming-guides/proto2#updating)がありますが、それらはほとんど使用されません。）

これらのルールに従うと、古いコードは新しいメッセージを問題なく読み取り、新しいフィールドを単に無視します。古いコードにとって、削除されたオプションフィールドは単にデフォルト値を持ち、削除された繰り返しフィールドは空になります。新しいコードも古いメッセージを透過的に読み取ります。ただし、新しいオプションフィールドは古いメッセージに存在しないため、`has_`で明示的に設定されているかどうかを確認するか、タグ番号の後に`[default = value]`を使用して`.proto`ファイルで適切なデフォルト値を提供する必要があります。オプション要素のデフォルト値が指定されていない場合、タイプ固有のデフォルト値が使用されます：文字列の場合、デフォルト値は空の文字列です。ブール値の場合、デフォルト値はfalseです。数値型の場合、デフォルト値はゼロです。また、新しい繰り返しフィールドを追加した場合、新しいコードはそれが空にされたか（新しいコードによって）、またはまったく設定されていないか（古いコードによって）を判断できません。それに対する`has_`フラグは存在しないためです。```

## 高度な使用法 {#advanced-usage}

プロトコルバッファには、アクセサやシリアライゼーションを超える用途があります。
[Python API リファレンス](https://googleapis.dev/python/protobuf/latest/) を
探索して、それらを使用する方法を確認してください。

プロトコルメッセージクラスが提供する主要な機能の1つは *リフレクション* です。
メッセージのフィールドを反復処理し、特定のメッセージタイプに対してコードを書かずに値を操作できます。
リフレクションを使用する非常に便利な方法の1つは、プロトコルメッセージを他のエンコーディング（XML や JSON など）に変換することです。
リフレクションのより高度な使用法としては、同じタイプの2つのメッセージの違いを見つけたり、
特定のメッセージ内容に一致する式を記述できる「プロトコルメッセージ用の正規表現」を開発したりすることが挙げられます。
想像力を働かせれば、プロトコルバッファを最初に期待していたよりも広範囲の問題に適用することが可能です！

リフレクションは、[`Message` インターフェース](https://googleapis.dev/python/protobuf/latest/google/protobuf/message.html#google.protobuf.message.Message) の一部として提供されています。
