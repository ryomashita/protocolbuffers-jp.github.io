+++
title = "概要"
weight = 10
description = "Protocol Buffers は、言語に依存しない、プラットフォームに依存しない、拡張可能なメカニズムで、構造化されたデータをシリアル化するためのものです。"
type = "docs"
+++

JSONのようなものですが、
より小さく高速であり、ネイティブ言語のバインディングを生成します。データの構造を一度定義し、特別に生成されたソースコードを使用して、構造化されたデータをさまざまなデータストリームや言語を使用して簡単に書き込み、読み取ることができます。

Protocol Buffers は、定義言語（`.proto` ファイルで作成）、proto コンパイラがデータとインターフェースするために生成するコード、言語固有のランタイムライブラリ、ファイルに書き込まれるデータのシリアル化形式（またはネットワーク接続を介して送信されるデータ）、およびシリアル化されたデータの組み合わせです。

## Protocol Buffers が解決する問題 {#solve}

Protocol Buffers は、最大数メガバイトの構造化データのパケットのシリアル化形式を提供します。この形式は、一時的なネットワークトラフィックや長期的なデータ保存の両方に適しています。Protocol Buffers は、既存のデータを無効にすることなく新しい情報を追加したり、コードを更新することなく拡張することができます。

Protocol Buffers は、Google で最も一般的に使用されるデータ形式です。インジニアが作成した `.proto` ファイルで記述された Protocol Buffer *メッセージ* および *サービス* は、Google のインターサーバ���信やディスク上のデータのアーカイブ保存などで広く使用されています。以下は `message` の例を示しています：

```proto
message Person {
  optional string name = 1;
  optional int32 id = 2;
  optional string email = 3;
}
```

proto コンパイラは、ビルド時に `.proto` ファイルで呼び出され、さまざまなプログラミング言語のコードを生成します（このトピックの後半でカバーされる[クロス言語互換性](#cross-lang)）。各生成されたクラスには、各フィールド用の簡単なアクセサと、構造全体を生のバイトにシリアル化および解析するためのメソッドが含まれています。以下は、それらの生成されたメソッドを使用する例を示しています：

```java
Person john = Person.newBuilder()
    .setId(1234)
    .setName("John Doe")
    .setEmail("jdoe@example.com")
    .build();
output = new FileOutputStream(args[0]);
john.writeTo(output);
```

Protocol Buffers は、Google のさまざまなサービス全体で広く使用されており、それらのデータは一定期間保存される可能性があるため、後方互換性の維持が重要です。Protocol Buffers は、既存のサービスを壊すことなく、新しいフィールドの追加や既存のフィールドの削除などの変更をシームレスにサポートします。このトピックについて詳しくは、このトピックの後半で説明される[コードの更新なしで Proto 定義を更新する](#updating-defs)を参照してください。

## Protocol Buffersの利点は何ですか？ {#benefits}

Protocol Buffersは、言語に依存せず、プラットフォームに依存せず、拡張可能な方法で構造化された、レコードのような、型付きデータをシリアライズする必要がある場合に最適です。これらは、通信プロトコルの定義（gRPCと共に）やデータストレージのために最も頻繁に使用されます。

Protocol Buffersを使用する利点の一部は次のとおりです：

* コンパクトなデータストレージ
* 高速なパース処理
* 多くのプログラミング言語で利用可能
* 自動生成されたクラスによる最適化された機能

### 言語間の互換性 {#cross-lang}

同じメッセージは、サポートされている任意のプログラミング言語で書かれたコードによって読み取ることができます。たとえば、あるプラットフォーム上のJavaプログラムが、`.proto`の定義に基づいてデータをシリアライズし、別のプラットフォーム上で実行されている別のPythonアプリケーションからそのシリアライズされたデータから特定の値を抽出することができます。

以下の言語は、Protocol Buffersコンパイラprotocで直接サポートされています：

* [C++](/reference/cpp/cpp-generated#invocation)
* [C#](/reference/csharp/csharp-generated#invocation)
* [Java](/reference/java/java-generated#invocation)
* [Kotlin](/reference/kotlin/kotlin-generated#invocation)
* [Objective-C](/reference/objective-c/objective-c-generated#invocation)
* [PHP](/reference/php/php-generated#invocation)
* [Python](/reference/python/python-generated#invocation)
* [Ruby](/reference/ruby/ruby-generated#invocation)

以下の言語はGoogleによってサポートされていますが、プロジェクトのソースコードはGitHubリポジトリにあります。これらの言語については、protocコンパイラはこれらの言語用のプラグインを使用します：

<!-- mdformat off(mdformat adds a space between the ) and the {) -->
* [Dart](https://github.com/google/protobuf.dart)
* [Go](https://github.com/protocolbuffers/protobuf-go)
<!-- mdformat on -->

追加の言語はGoogleによって直接サポートされておらず、代わりに他のGitHubプロジェクトによってサポートされています。これらの言語については、[Protocol Buffersのサードパーティアドオン](https://github.com/protocolbuffers/protobuf/blob/master/docs/third_party.md)でカバーされています。

### クロスプロジェクトサポート {#cross-proj}

特定のプロジェクトのコードベースの外にある`.proto`ファイルで`message`タイプを定義することで、プロジェクト間でプロトコルバッファを使用できます。直近のチームの外で広く使用されると予想される`message`タイプや列挙型を定義する場合は、依存関係のない独自のファイルに配置できます。

Google内で広く使用されているプロト定義の例として、[`timestamp.proto`](https://github.com/protocolbuffers/protobuf/blob/master/src/google/protobuf/timestamp.proto)や[`status.proto`](https://github.com/googleapis/googleapis/blob/master/google/rpc/status.proto)があります。

### コードを更新せずにプロト定義を更新する {#updating-defs}

ソフトウェア製品が後方互換性を持つのは標準ですが、前方互換性を持つのは一般的ではありません。`.proto`定義を更新する際には、[簡単な手順](/programming-guides/proto3/#updating)に従えば、古いコードでも新しいメッセージを問題なく読み取り、新たに追加されたフィールドを無視します。削除されたフィールドに対しては、古いコードではデフォルト値が使用され、削除された繰り返しフィールドは空になります。"繰り返し"フィールドとは何かについては、このトピックの後の[プロトコルバッファ定義構文](#syntax)を参照してください。

新しいコードも古いメッセージを透過的に読み取ります。新しいフィールドは古いメッセージに存在しません。この場合、プロトコルバッファは合理的なデフォルト値を提供します。

### プロトコルバッファが適さない場合 {#not-good-fit}

プロトコルバッファはすべてのデータに適しているわけではありません。特に以下の点に注意してください：

*   プロトコルバッファは、メッセージ全体を一度にメモリに読み込んでオブジェクトグラフよりも大きくないと仮定しています。数メガバイトを超えるデータには別の解決策を検討してください。大きなデータを扱う場合、シリアル化されたコピーによりデータの複数のコピーが効果的に生成され、予期しないメモリ使用量の急増を引き起こす可能性があります。
*   プロトコルバッファがシリアル化されると、同じデータでも多くの異なるバイナリシリアル化が可能です。完全に解析せずに2つのメッセージを等しく比較することはできません。
*   メッセージは圧縮されません。メッセージは他のファイルと同様にzippedやgzippedできますが、JPEGやPNGで使用されるような専用の圧縮アルゴリズムを使用すると、適切なタイプのデータに対してははるかに小さなファイルが生成されます。
*   プロトコルバッファメッセージは、大規模で多次元の浮動小数点数の配列を含む多くの科学技術用途において、サイズと速度の両方で最適ではありません。これらのアプリケーションでは、[FITS](https://en.wikipedia.org/wiki/FITS)や類似の形式の方がオーバーヘッドが少ないです。
*   プロトコルバッファは、FortranやIDLなどの科学計算で人気のないオブジェクト指向でない言語でのサポートが不十分です。
*   プロトコルバッファメッセージは、データを自己記述することはありませんが、完全に反射的なスキーマを持っており、自己記述を実装するために使用できます。つまり、対応する`.proto`ファイルにアクセスしないと完全に解釈することはできません。
*   プロトコルバッファは、どの組織の形式基準でもありません。これは、標準の上に構築する法的またはその他の要件がある環境での使用には適していません。

## Protocol Buffersを使用するのは誰ですか？ {#who-uses}

多くのプロジェクトがProtocol Buffersを使用しており、以下が含まれます：

<!-- mdformat off(mdformat adds a space between the ) and the {) -->

+   [gRPC](https://grpc.io)
+   [Google Cloud](https://cloud.google.com)
+   [Envoy Proxy](https://www.envoyproxy.io) <!-- mdformat on -->

## Protocol Buffersの動作原理は？ {#work}

次の図は、データを扱うためにProtocol Buffersを使用する方法を示しています。

![](/images/protocol-buffers-concepts.png) \
**図1. Protocol Buffersのワークフロー**

Protocol Buffersによって生成されたコードは、ファイルやストリームからデータを取得したり、データから個々の値を抽出したり、データの存在を確認したり、データをファイルやストリームにシリアライズしたりするためのユーティリティメソッドを提供します。

次のコードサンプルは、このフローの例をJavaで示しています。先に示したように、これは`.proto`の定義です：

```proto
message Person {
  optional string name = 1;
  optional int32 id = 2;
  optional string email = 3;
}
```

この`.proto`ファイルをコンパイルすると、新しいインスタンスを作成するために使用できる`Builder`クラスが作成されます。以下のJavaコードのように使用できます：

```java
Person john = Person.newBuilder()
    .setId(1234)
    .setName("John Doe")
    .setEmail("jdoe@example.com")
    .build();
output = new FileOutputStream(args[0]);
john.writeTo(output);
```

その後、C++などの他の言語でProtocol Buffersが作成するメソッドを使用してデータを逆シリアル化できます：

```cpp
Person john;
fstream input(argv[1], ios::in | ios::binary);
john.ParseFromIstream(&input);
int id = john.id();
std::string name = john.name();
std::string email = john.email();
```

## Protocol Buffersの定義構文 {#syntax}

`.proto`ファイルを定義する際、フィールドを`optional`または`repeated`（proto2およびproto3）に指定するか、proto3ではデフォルトの暗黙的存在に設定したままにするかを指定できます（proto3では`required`フィールドを設定するオプションは存在せず、proto2では強く推奨されません。詳細については、「Required is Forever」を参照してください。[フィールドルールの指定](/programming-guides/proto3#specifying-field-rules)）。

フィールドのオプション性/繰り返し性を設定した後、データ型を指定します。Protocol Buffersは、整数、ブール値、浮動小数点数などの通常のプリミティブデータ型をサポートしています。完全なリストについては、[スカラー値の種類](/programming-guides/proto3#scalar)を参照してください。

フィールドは次のようなものにすることもできます：

*   `message`型：定義の一部をネストして繰り返しデータセットを指定できます。
*   `enum`型：選択できる値のセットを指定できます。
*   `oneof`型：メッセージに多くのオプションフィールドがあり、同時に1つのフィールドが設定される場合に使用できます。
*   `map`型：定義にキーと値のペアを追加できます。

proto2では、メッセージは**拡張**を許可して、メッセージ自体の外側にフィールドを定義することができます。たとえば、protobufライブラリの内部メッセージスキーマでは、カスタムの使用目的に特化したオプションのための拡張が可能です。

利用可能なオプションについての詳細は、[proto2](/programming-guides/proto2)または[proto3](/programming-guides/proto3)の言語ガイドを参照してください。

オプション性とフィールドタイプを設定した後、フィールドに名前を付けます。フィールド名を設定する際には、以下の点に注意する必要があります：

*   プロダクションで使用された後にフィールド名を変更することは、時には困難であるか、不可能であることがあります。
*   フィールド名にはダッシュを含めることはできません。フィールド名の構文について詳しくは、[メッセージとフィールド名](/programming-guides/style#message-field-names)を参照してください。
*   繰り返しフィールドには複数形の名前を使用してください。

フィールドに名前を割り当てた後、フィールド番号を割り当てます。フィールド番号は再利用や再利用できません。フィールドを削除する場合は、誰かが誤って番号を再利用することを防ぐために、そのフィールド番号を予約する必要があります。

## 追加のデータ型サポート {#data-types}

プロトコルバッファは、可変長エンコーディングと固定サイズを使用する整数を含む多くのスカラー値タイプをサポートしています。また、フィールドに割り当てることができるデータ型であるメッセージを定義することで、独自の複合データ型を作成することもできます。単純な値タイプと複合値タイプに加えて、いくつかの[一般的なタイプ](/programming-guides/dos-donts#common)が公開されています。

## 履歴 {#history}

プロトコルバッファプロジェクトの歴史については、[Protocol Buffersの歴史](/history)を参照してください。

## Protocol Buffersオープンソースの哲学 {#philosophy}

プロトコルバッファは、Google外の開発者に、内部で得られる利点と同じ利点を提供する手段として、2008年にオープンソース化されました。私たちは、内部要件をサポートするためにこれらの変更を行う際に、言語への定期的な更新を通じてオープンソースコミュニティを支援しています。外部の開発者からの一部のプルリクエストを受け入れていますが、Googleの特定のニーズに準拠しない機能リクエストやバグ修正を常に優先することはできません。

## 開発者コミュニティ {#community}

Protocol Buffersの今後の変更に関するアラートを受け取り、protobufの開発者やユーザーとつながるには、
[Google グループに参加してください](https://groups.google.com/g/protobuf)。

## 追加リソース {#additional-resources}

*   [Protocol Buffers GitHub](https://github.com/protocolbuffers/protobuf/)
*   [Codelabs](/getting-started/codelabs)
