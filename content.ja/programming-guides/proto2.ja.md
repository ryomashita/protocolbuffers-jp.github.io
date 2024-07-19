+++
title = "言語ガイド（proto 2）"
weight = 30
description = "プロジェクトで Protocol Buffers のバージョン 2 を使用する方法について説明します。"
aliases = "/programming-guides/proto/"
type = "docs"
+++

このガイドでは、プロトコルバッファデータを構造化するためのプロトコルバッファ言語の使用方法について説明します。`.proto` ファイルの構文や`.proto` ファイルからデータアクセスクラスを生成する方法について説明します。このガイドは、プロトコルバッファ言語の **proto2** バージョンをカバーしています。**proto3** 構文に関する情報は、[Proto3 言語ガイド](/programming-guides/proto3)を参照してください。

これはリファレンスガイドです。このドキュメントで説明されている機能の多くを使用するステップバイステップの例については、選択した言語の [チュートリアル](/getting-started)を参照してください。

## メッセージタイプの定義 {#simple}

まず、非常にシンプルな例を見てみましょう。検索リクエストメッセージ形式を定義したいとします。各検索リクエストには、クエリ文字列、興味のある結果ページ、およびページごとの結果数が含まれます。次に、メッセージタイプを定義するために使用する`.proto` ファイルを示します。

```proto
syntax = "proto2";

message SearchRequest {
  optional string query = 1;
  optional int32 page_number = 2;
  optional int32 results_per_page = 3;
}
```

*   ファイルの最初の行は、`proto2` 構文を使用していることを指定しています。これは、ファイル内で最初の空でないコメント行である必要があります。
*   `SearchRequest` メッセージ定義は、このタイプのメッセージに含めたいデータの各部分（名前/値のペア）を指定します。各フィールドには名前とタイプがあります。

### フィールドタイプの指定 {#specifying-types}

前述の例では、すべてのフィールドが [スカラータイプ](#scalar) です。つまり、2 つの整数（`page_number` と `results_per_page`）と 1 つの文字列（`query`）です。また、フィールドには [列挙型](#enum) や他のメッセージタイプのような複合型を指定することもできます。

### フィールド番号の割り当て {#assigning}

メッセージ定義内の各フィールドには、以下の制限を持つ `1` から `536,870,911` の間の番号を割り当てる必要があります。

-   指定された番号は、そのメッセージ内のすべてのフィールドの中で **一意** である必要があります。
-   フィールド番号 `19,000` から `19,999` は、Protocol Buffers の実装用に予約されています。メッセージ内でこれらの予約されたフィールド番号を使用すると、プロトコルバッファコンパイラがエラーを出力します。
-   以前に [予約された](#fieldreserved) フィールド番号や [拡張](#extensions) に割り当てられたフィールド番号を使用することはできません。

この番号は、メッセージのワイヤ形式でのフィールドを識別するために使用されるため、**メッセージタイプが使用中である場合には変更できません**。フィールド番号を「変更する」とは、そのフィールドを削除し、同じ型で新しい番号を持つ新しいフィールドを作成することと同等です。これを適切に行う方法については、[フィールドの削除](#deleting)を参照してください。

フィールド番号は**決して再利用してはいけません**。新しいフィールド定義で再利用するために、[予約済み](#fieldreserved)リストからフィールド番号を取り出してはいけません。フィールド番号を再利用することの結果については、[フィールド番号の再利用の影響](#consequences)を参照してください。

最も頻繁に設定されるフィールドには、フィールド番号1から15を使用する必要があります。低いフィールド番号値は、ワイヤ形式でのスペースを取らないためです。たとえば、1から15の範囲のフィールド番号は、1バイトでエンコードされます。16から2047の範囲のフィールド番号は2バイトを取ります。これについて詳しくは、[Protocol Buffer Encoding](/programming-guides/encoding#structure)を参照してください。

#### フィールド番号の再利用の影響 {#consequences}

フィールド番号を再利用すると、ワイヤ形式のメッセージのデコードが曖昧になります。

Protobufのワイヤ形式はシンプルであり、エンコードされたフィールドが別の定義でデコードされる方法を提供しません。

1つの定義でフィールドをエンコードし、異なる定義でその同じフィールドをデコードすることは、次のような問題を引き起こす可能性があります：

-   デバッグに費やされる開発者の時間
-   パース/マージエラー（最善の場合）
-   PII/SPIIの漏洩
-   データの破損

フィールド番号の再利用の一般的な原因：

-   フィールドの番号を変更する（フィールドの番号順序をより見栄えの良いものにするために行われることがあります）。番号を変更することは、再利用不可能なワイヤ形式の変更をもたらす、関連するすべてのフィールドを削除して再追加する効果があります。
-   フィールドを削除し、将来の再利用を防ぐために番号を[予約](#fieldreserved)しない。これは、[拡張フィールド](#extensions)に関して非常に簡単なミスであることがあります。[拡張宣言](/programming-guides/extension_declarations)は、拡張フィールドを予約するメカニズムを提供します。

最大フィールドは、3つの下位ビットがワイヤーフォーマットに使用されるため、通常の32ビットではなく29ビットです。詳細は、[エンコーディングトピック](/programming-guides/encoding#structure)を参照してください。

<a id="specifying-rules"></a>

### フィールドラベルの指定 {#field-labels}

メッセージフィールドは次のいずれかになります：

*   `optional`: `optional`フィールドは、次の2つの状態のいずれかにあります：

    *   フィールドが設定され、ワイヤーから明示的に設定された値を含んでいます。ワイヤーにシリアル化されます。
    *   フィールドが未設定で、デフォルト値が返されます。ワイヤーにシリアル化されません。

    値が明示的に設定されたかどうかを確認できます。

*   `repeated`: このフィールドタイプは、整形されたメッセージ内でゼロ回以上繰り返すことができます。繰り返し値の順序が保持されます。

*   `map`: これはキーと値のペアフィールドタイプです。このフィールドタイプについては、[Maps](/programming-guides/encoding#maps)を参照してください。

*   `required`: **使用しないでください。** 必須フィールドは問題が多いため、proto3から削除されました。必須フィールドのセマンティクスはアプリケーションレイヤーで実装する必要があります。使用される場合、整形されたメッセージにはこのフィールドが正確に1つだけ含まれている必要があります。

歴史的な理由から、スカラー数値型（たとえば、`int32`、`int64`、`enum`）の`repeated`フィールドは、効率的にエンコードされていません。新しいコードでは、より効率的なエンコーディングを取得するために特別なオプション`[packed = true]`を使用する必要があります。例：

```proto
repeated int32 samples = 4 [packed = true];
repeated ProtoEnum results = 5 [packed = true];
```

`packed`エンコーディングについて詳細は、[Protocol Buffer Encoding](/programming-guides/encoding#packed)を参照してください。

{{% alert title="重要" color="warning" %}} **必須は永遠に**
前述のように、**新しいフィールドには`required`を使用しないでください**。必須フィールドのセマンティクスは代わりにアプリケーションレイヤーで実装する必要があります。
既存の`required`フィールドは、メッセージ定義の永続的で変更不可能な要素として扱われる必要があります。`required`から`optional`に安全に変更することはほぼ不可能です。古いリーダーが存在する可能性がある場合、このフィールドがないメッセージを不完全と見なし、拒否または破棄する可能性があります。{{% /alert %}}

第二の問題は、必須フィールドに値を追加するときに発生します。この場合、認識されない列挙値は欠落したものとして扱われ、これにより必須値のチェックも失敗します。

#### Well-formed Messages {#well-formed}

「well-formed」という用語は、protobuf メッセージに適用されると、シリアル化/デシリアル化されたバイトを指します。protoc パーサーは、指定された proto 定義ファイルが解析可能であることを検証します。

1つ以上の値を持つ `optional` フィールドの場合、protoc パーサーは入力を受け入れますが、最後のフィールドのみを使用します。そのため、「バイト」は「well-formed」でないかもしれませんが、生成されたメッセージは 1 つだけであり、「well-formed」である可能性があります（ただし、同じものをラウンドトリップできないかもしれません）。

### Adding More Message Types {#adding-types}

1 つの `.proto` ファイルに複数のメッセージタイプを定義することができます。これは、複数の関連するメッセージを定義する場合に便利です。たとえば、`SearchResponse` メッセージタイプに対応する応答メッセージ形式を定義したい場合、同じ `.proto` に追加できます：

```proto
message SearchRequest {
  optional string query = 1;
  optional int32 page_number = 2;
  optional int32 results_per_page = 3;
}

message SearchResponse {
 ...
}
```

**メッセージの結合は膨張をもたらす** 複数のメッセージタイプ（メッセージ、列挙型、サービスなど）を 1 つの `.proto` ファイルで定義できますが、1 つのファイルで多数のメッセージが異なる依存関係で定義されると、依存関係の膨張が発生する可能性があります。可能な限り 1 つの `.proto` ファイルあたりのメッセージタイプを少なくすることが推奨されています。

### Adding Comments {#adding-comments}

`.proto` ファイルにコメントを追加するには、C/C++ スタイルの `//` および `/* ... */` 構文を使用します。

```proto
/* SearchRequest represents a search query, with pagination options to
 * indicate which results to include in the response. */

message SearchRequest {
  optional string query = 1;
  optional int32 page_number = 2;  // Which page number do we want?
  optional int32 results_per_page = 3;  // Number of results to return per page.
}
```

### フィールドの削除 {#deleting}

フィールドを削除すると、適切に行われない場合に深刻な問題が発生する可能性があります。

**削除しないでください** `required` フィールド。これを安全に行うのはほとんど不可能です。

非必須フィールドが不要になり、すべてのクライアントコードから参照が削除された場合、メッセージからフィールド定義を削除することができます。ただし、削除したフィールド番号を[予約する必要があります](#fieldreserved)。フィールド番号を予約しないと、将来開発者がその番号を再利用する可能性があります。


<a id="fieldreserved"></a>

### 予約されたフィールド番号 {#reserved-field-numbers}

メッセージタイプを[更新](#updating)してフィールドを完全に削除したり、コメントアウトしたりすると、将来の開発者がそのタイプを更新する際にフィールド番号を再利用する可能性があります。これは[フィールド番号の再利用の影響](#consequences)で説明されているように深刻な問題を引き起こす可能性があります。これを防ぐために、削除されたフィールド番号を`reserved`リストに追加してください。

protocコンパイラは、将来の開発者がこれらの予約されたフィールド番号を使用しようとするとエラーメッセージを生成します。

```proto
message Foo {
  reserved 2, 15, 9 to 11;
}
```

予約されたフィールド番号の範囲は包括的です（`9 to 11`は`9, 10, 11`と同じです）。

#### 予約されたフィールド名 {#reserved-field-names}

後で古いフィールド名を再利用することは一般的に安全ですが、TextProtoやJSONエンコーディングを使用する場合、フィールド名がシリアライズされるため、問題が発生する可能性があります。このリスクを回避するために、削除されたフィールド名を`reserved`リストに追加できます。

予約された名前はprotocコンパイラの動作のみに影響を与え、ランタイムの動作には影響しません。ただし、TextProtoの実装では、予約された名前を持つ未知のフィールドを解析時に破棄する場合があります（今日、C++およびGoの実装のみがそうしています）。ランタイムのJSONパースは予約された名前に影響を受けません。

```proto
message Foo {
  reserved 2, 15, 9 to 11;
  reserved "foo", "bar";
}
```

フィールド名とフィールド番号を同じ`reserved`ステートメントで混在させることはできないことに注意してください。

### `.proto`から生成されるものは何ですか？ {#generated}

`.proto`ファイルに[protocol bufferコンパイラ](#generating)を実行すると、選択した言語でコードが生成され、ファイルで記述したメッセージタイプを扱うために必要なコードが生成されます。これには、フィールド値の取得と設定、メッセージを出力ストリームにシリアライズすること、およびメッセージを入力ストリームからパースすることが含まれます。

*   **C++**の場合、コンパイラは各`.proto`から`.h`および`.cc`ファイルを生成し、ファイルで記述した各メッセージタイプに対するクラスを生成します。
*   **Java**の場合、コンパイラは各メッセージタイプに対してクラスを生成する`.java`ファイルを生成し、メッセージクラスインスタンスを作成するための特別な`Builder`クラスも生成します。
*   **Kotlin**の場合、Javaで生成されたコードに加えて、各メッセージタイプに対して`.kt`ファイルを生成し、メッセージインスタンスの作成を簡素化するために使用できるDSLを含みます。
*   **Python**は少し異なります。Pythonコンパイラは、`.proto`内の各メッセージタイプの静的ディスクリプタを持つモジュールを生成し、これはランタイムで必要なPythonデータアクセスクラスを作成するために*メタクラス*と共に使用されます。
*   **Go**の場合、コンパイラは`.pb.go`ファイルを生成し、ファイルで記述した各メッセージタイプに対する型を生成します。
*   **Ruby**の場合、コンパイラは、メッセージタイプを含むRubyモジュールを生成する`.rb`ファイルを生成します。
*   **Objective-C**の場合、コンパイラは、各`.proto`から`pbobjc.h`および`pbobjc.m`ファイルを生成し、ファイルで記述した各メッセージタイプに対するクラスを生成します。
*   **C#**の場合、コンパイラは、各`.proto`から`.cs`ファイルを生成し、ファイルで記述した各メッセージタイプに対するクラスを生成します。
*   **Dart**の場合、コンパイラは、ファイルで記述した各メッセージタイプに対するクラスを生成する`.pb.dart`ファイルを生成します。

## スカラー値の型 {#scalar}

スカラーメッセージフィールドは、次のいずれかの型を持つことができます - テーブルには`.proto`ファイルで指定された型と、自動生成されたクラスでの対応する型が示されています。

<div style="overflow:auto;width:100%;">
  <table style="width: 110%;">
    <tbody>
      <tr>
        <th>.protoの型</th>
        <th>注釈</th>
        <th>C++の型</th>
        <th>Java/Kotlinの型<sup>[1]</sup></th>
        <th>Pythonの型<sup>[3]</sup></th>
        <th>Goの型</th>
        <th>Rubyの型</th>
        <th>C#の型</th>
        <th>Dartの型</th>
      </tr>
      <tr>
        <td>double</td>
        <td></td>
        <td>double</td>
        <td>double</td>
        <td>float</td>
        <td>*float64</td>
        <td>Float</td>
        <td>double</td>
        <td>double</td>
      </tr>
      <tr>
        <td>float</td>
        <td></td>
        <td>float</td>
        <td>float</td>
        <td>float</td>
        <td>*float32</td>
        <td>Float</td>
        <td>float</td>
        <td>double</td>
      </tr>
      <tr>
        <td>int32</td>
        <td>可変長エンコーディングを使用します。負の値をエンコードするには非効率です - フィールドが負の値を取る可能性が高い場合は、代わりにsint32を使用してください。</td>
        <td>int32</td>
        <td>int</td>
        <td>int</td>
        <td>int32</td>
        <td>FixnumまたはBignum（必要に応じて）</td>
        <td>int</td>
        <td>*int32</td>
      </tr>
      <tr>
        <td>int64</td>
        <td>可変長エンコーディングを使用します。負の値をエンコードするには非効率です - フィールドが負の値を取る可能性が高い場合は、代わりにsint64を使用してください。</td>
        <td>int64</td>
        <td>long</td>
        <td>int/long<sup>[4]</sup></td>
        <td>*int64</td>
        <td>Bignum</td>
        <td>long</td>
        <td>Int64</td>
      </tr>
      <tr>
        <td>uint32</td>
        <td>可変長エンコーディングを使用します。</td>
        <td>uint32</td>
        <td>int<sup>[2]</sup></td>
        <td>int/long<sup>[4]</sup></td>
        <td>*uint32</td>
        <td>FixnumまたはBignum（必要に応じて）</td>
        <td>uint</td>
        <td>int</td>
      </tr>
      <tr>
        <td>uint64</td>
        <td>可変長エンコーディングを使用します。</td>
        <td>uint64</td>
        <td>long<sup>[2]</sup></td>
        <td>int/long<sup>[4]</sup></td>
        <td>*uint64</td>
        <td>Bignum</td>
        <td>ulong</td>
        <td>Int64</td>
      </tr>
      <tr>
        <td>sint32</td>
        <td>可変長エンコーディングを使用します。符号付き整数値です。これらは通常のint32よりも負の数を効率的にエンコードします。</td>
        <td>int32</td>
        <td>int</td>
        <td>int</td>
        <td>int32</td>
        <td>FixnumまたはBignum（必要に応じて）</td>
        <td>int</td>
        <td>*int32</td>
      </tr>
      <tr>
        <td>sint64</td>
        <td>可変長エンコーディングを使用します。符号付き整数値です。これらは通常のint64よりも負の数を効率的にエンコードします。</td>
        <td>int64</td>
        <td>long</td>
        <td>int/long<sup>[4]</sup></td>
        <td>*int64</td>
        <td>Bignum</td>
        <td>long</td>
        <td>Int64</td>
      </tr>
      <tr>
        <td>fixed32</td>
        <td>常に4バイトです。値が2<sup>28</sup>よりも大きい場合は、uint32よりも効率的です。</td>
        <td>uint32</td>
        <td>int<sup>[2]</sup></td>
        <td>int/long<sup>[4]</sup></td>
        <td>*uint32</td>
        <td>FixnumまたはBignum（必要に応じて）</td>
        <td>uint</td>
        <td>int</td>
      </tr>
      <tr>
        <td>fixed64</td>
        <td>常に8バイトです。値が2<sup>56</sup>よりも大きい場合は、uint64よりも効率的です。</td>
        <td>uint64</td>
        <td>long<sup>[2]</sup></td>
        <td>int/long<sup>[4]</sup></td>
        <td>*uint64</td>
        <td>Bignum</td>
        <td>ulong</td>
        <td>Int64</td>
      </tr>
      <tr>
        <td>sfixed32</td>
        <td>常に4バイトです。</td>
        <td>int32</td>
        <td>int</td>
        <td>int</td>
        <td>*int32</td>
        <td>FixnumまたはBignum（必要に応じて）</td>
        <td>int</td>
        <td>int</td>
      </tr>
      <tr>
        <td>sfixed64</td>
        <td>常に8バイトです。</td>
        <td>int64</td>
        <td>long</td>
        <td>int/long<sup>[4]</sup></td>
        <td>*int64</td>
        <td>Bignum</td>
        <td>long</td>
        <td>Int64</td>
      </tr>
      <tr>
        <td>bool</td>
        <td></td>
        <td>bool</td>
        <td>boolean</td>
        <td>bool</td>
        <td>*bool</td>
        <td>TrueClass/FalseClass</td>
        <td>bool</td>
        <td>bool</td>
      </tr>
      <tr>
        <td>string</td>
        <td>文字列は常にUTF-8エンコード<sup>[5]</sup>または7ビットASCIIテキストを含む必要があり、2<sup>32</sup>より長くすることはできません。</td>
        <td>string</td>
        <td>String</td>
        <td>unicode（Python 2）またはstr（Python 3）</td>
        <td>*string</td>
        <td>String（UTF-8）</td>
        <td>string</td>
        <td>String</td>
      </tr>
      <tr>
        <td>bytes</td>
        <td>任意のバイトシーケンスを含むことができ、2<sup>32</sup>より長くすることはできません。</td>
        <td>string</td>
        <td>ByteString</td>
        <td>bytes</td>
        <td>[]byte</td>
        <td>String（ASCII-8BIT）</td>
        <td>ByteString</td>
        <td>List<int></td>
      </tr>
    </tbody>
  </table>
</div>

<sup>[1]</sup> Kotlinは、Javaと対応する型を使用し、Java/Kotlinの混在するコードベースでの互換性を確保します。

<sup>[2]</sup> Javaでは、符号付きの対応する型を使用して、符号なし32ビットおよび64ビット整数を表現し、最上位ビットは単純に符号ビットに格納されます。

<sup>[3]</sup> すべての場合において、フィールドに値を設定すると、その値が有効であることを確認するために型チェックが実行されます。

<sup>[4]</sup> 64ビットまたは符号なし32ビット整数は、デコード時に常にlongとして表現されますが、フィールドを設定する際にintが指定された場合はintになります。すべての場合において、値は設定時に表現される型に収まる必要があります。[2]を参照してください。

<sup>[5]</sup> Proto2では通常、文字列フィールドのUTF-8の有効性をチェックしません。ただし、言語によって動作が異なり、無効なUTF-8データは文字列フィールドに保存すべきではありません。

これらの型がどのようにエンコードされるかについては、メッセージをシリアライズする際の[Protocol Buffer Encoding](/programming-guides/encoding)を参照してください。

## オプションフィールドとデフォルト値 {#optional}

前述のように、メッセージの説明の要素は`optional`とラベル付けされることがあります。整形されたメッセージには、オプションの要素が含まれていても含まれていなくても構いません。メッセージが解析されると、エンコードされたメッセージにオプションの要素が含まれていない場合、解析されたオブジェクトの対応するフィールドにアクセスすると、そのフィールドのデフォルト値が返されます。デフォルト値はメッセージの説明の一部として指定できます。たとえば、`SearchRequest`の`result_per_page`の値にデフォルト値10を指定したい場合は次のようにします。

```proto
optional int32 result_per_page = 3 [default = 10];
```

オプションの要素にデフォルト値が指定されていない場合、型固有のデフォルト値が代わりに使用されます。

- 文字列の場合、デフォルト値は空の文字列です。
- バイトの場合、デフォルト値は空のバイトです。
- ブールの場合、デフォルト値はfalseです。
- 数値型の場合、デフォルト値はゼロです。
- 列挙型の場合、デフォルト値は**最初に定義された列挙値**です。

列挙型のデフォルト値は最初に定義された列挙値であるため、列挙値リストの先頭に値を追加する際には注意が必要です。定義を安全に変更する方法については、[メッセージタイプの更新](#updating)セクションを参照してください。

## 列挙型 {#enum}

メッセージタイプを定義する際、事前に定義された値のリストの中から1つだけを持つようにしたい場合があります。たとえば、各`SearchRequest`に`corpus`フィールドを追加したいとします。ここで、corpusは`UNIVERSAL`、`WEB`、`IMAGES`、`LOCAL`、`NEWS`、`PRODUCTS`、または`VIDEO`のいずれかであるとします。これは、メッセージ定義に`enum`を追加することで非常に簡単に行うことができます。`enum`型のフィールドは、指定された一連の定数のいずれかを値として持つことができます（異なる値を提供しようとすると、パーサーは未知のフィールドとして扱います）。

以下の例では、すべての可能な値を持つ`Corpus`という`enum`を追加し、`Corpus`型のフィールドを追加しています。

```proto
enum Corpus {
  CORPUS_UNSPECIFIED = 0;
  CORPUS_UNIVERSAL = 1;
  CORPUS_WEB = 2;
  CORPUS_IMAGES = 3;
  CORPUS_LOCAL = 4;
  CORPUS_NEWS = 5;
  CORPUS_PRODUCTS = 6;
  CORPUS_VIDEO = 7;
}

message SearchRequest {
  optional string query = 1;
  optional int32 page_number = 2;
  optional int32 results_per_page = 3 [default = 10];
  optional Corpus corpus = 4 [default = CORPUS_UNIVERSAL];
}
```

`SearchRequest`は`corpus`フィールドの値のデフォルトを設定しているため、`CORPUS_UNSPECIFIED`値はデフォルトとして使用されません。ワイヤー上で値0がエンカウントされた場合には引き続き使用されます。デフォルトを設定していない`Corpus`型の他のインスタンスは、デフォルトとして`CORPUS_UNSPECIFIED`値が使用されます。

異なる列挙定数に同じ値を割り当てることでエイリアスを定義することができます。これを行うには、`allow_alias`オプションを`true`に設定する必要があります。そうでない場合、エイリアスが見つかったときにプロトコルバッファコンパイラが警告メッセージを生成します。エイリアス値はすべて逆シリアル化中に有効ですが、シリアル化時には常に最初の値が使用されます。

```proto
enum EnumAllowingAlias {
  option allow_alias = true;
  EAA_UNSPECIFIED = 0;
  EAA_STARTED = 1;
  EAA_RUNNING = 1;
  EAA_FINISHED = 2;
}
enum EnumNotAllowingAlias {
  ENAA_UNSPECIFIED = 0;
  ENAA_STARTED = 1;
  // ENAA_RUNNING = 1;  // Uncommenting this line will cause a warning message.
  ENAA_FINISHED = 2;
}
```

列挙定数は32ビット整数の範囲内にある必要があります。`enum`値はワイヤー上で[varintエンコーディング](/programming-guides/encoding)を使用するため、負の値は効率が悪く、推奨されません。メッセージ定義内で`enum`を定義することもできます（前述の例のように）、または外部で定義することもできます。これらの`enum`は`.proto`ファイル内の任意のメッセージ定義で再利用することができます。また、1つのメッセージで宣言された`enum`型を、別のメッセージのフィールドの型として使用することもできます。その際の構文は`_MessageType_._EnumType_`です。

`.proto`ファイルで`enum`を使用する場合、プロトコルバッファコンパイラを実行すると、生成されたコードにはJava、Kotlin、またはC++用の対応する`enum`、またはPython用の特別な`EnumDescriptor`クラスが含まれます。これは、ランタイムで生成されるクラス内の整数値を持つ一連のシンボリック定数を作成するために使用されます。

{{% alert title="重要" color="warning" %}} 生成されたコードは、言語固有の制限（1つの言語につき数千の列挙子が制限されることがある）の影響を受ける可能性があります。使用する言語の制限事項を確認してください。{{% /alert %}}

{{% alert title="重要" color="warning" %}} 異なる言語での列挙型の動作と現在の動作の対比に関する情報については、[Enum Behavior](/programming-guides/enum)を参照してください。{{% /alert %}}

列挙値を削除することは、永続的なプロトに対する破壊的な変更です。値を削除する代わりに、値に `reserved` キーワードを付けて列挙値がコード生成されないようにしたり、値を保持したまま `deprecated` フィールドオプションを使用して後で削除されることを示すことができます：

```proto
enum PhoneType {
  PHONE_TYPE_UNSPECIFIED = 0;
  PHONE_TYPE_MOBILE = 1;
  PHONE_TYPE_HOME = 2;
  PHONE_TYPE_WORK = 3 [deprecated=true];
  reserved 4,5;
}
```

アプリケーションでメッセージ `enum` を使用する方法の詳細については、選択した言語の[生成されたコードガイド](/reference/)を参照してください。

### 予約済みの値 {#reserved}

列挙型を[更新](#updating)して列挙エントリを完全に削除したり、コメントアウトしたりすると、将来のユーザーがその型を更新する際に、数値値を再利用できるようになります。これは、同じ `.proto` の古いバージョンを後で読み込むと、データの破損、プライバシーバグなどの深刻な問題を引き起こす可能性があります。これを防ぐ方法の1つは、削除されたエントリの数値値（および/または名前、これもJSONシリアル化に問題を引き起こす可能性があります）が `reserved` であることを指定することです。プロトコルバッファコンパイラは、将来のユーザーがこれらの識別子を使用しようとした場合に警告を出します。`max` キーワードを使用して、予約済みの数値値範囲が最大可能な値までであることを指定できます。

```proto
enum Foo {
  reserved 2, 15, 9 to 11, 40 to max;
  reserved "FOO", "BAR";
}
```

同じ `reserved` ステートメント内でフィールド名と数値値を混在させることはできないことに注意してください。

## 他のメッセージ型の使用 {#other}

他のメッセージ型をフィールド型として使用することができます。たとえば、各 `SearchResponse` メッセージに `Result` メッセージを含めたい場合は、同じ `.proto` で `Result` メッセージ型を定義し、`SearchResponse` で `Result` 型のフィールドを指定します。

```proto
message SearchResponse {
  repeated Result results = 1;
}

message Result {
  optional string url = 1;
  optional string title = 2;
  repeated string snippets = 3;
}
```

### 定義のインポート {#importing}

前述の例では、`SearchResponse` と同じファイルに `Result` メッセージタイプが定義されていますが、もし使用したいメッセージタイプが別の `.proto` ファイルで定義されている場合はどうなるでしょうか？

他の `.proto` ファイルから定義を使用するには、それらを *インポート* します。別の `.proto` の定義をインポートするには、ファイルの先頭にインポートステートメントを追加します。

```proto
import "myproject/other_protos.proto";
```

デフォルトでは、直接インポートされた `.proto` ファイルからのみ定義を使用できます。しかし、時には `.proto` ファイルを新しい場所に移動する必要があるかもしれません。1つの変更ですべての呼び出し箇所を更新する代わりに、古い場所にプレースホルダーの `.proto` ファイルを配置し、`import public` 機能を使用してすべてのインポートを新しい場所に転送することができます。

**Java では `import public` 機能は使用できません。**

`import public` 依存関係は、`import public` ステートメントを含む proto をインポートするすべてのコードによって推移的に依存されることができます。例:

```proto
// new.proto
// すべての定義がここに移動されます
```

```proto
// old.proto
// This is the proto that all clients are importing.
import public "new.proto";
import "other.proto";
```

```proto
// client.proto
import "old.proto";
// old.proto と new.proto の定義を使用しますが、other.proto は使用しません
```

プロトコルコンパイラは、`-I`/`--proto_path` フラグを使用してプロトコルコンパイラのコマンドラインで指定されたディレクトリセット内でインポートされたファイルを検索します。フラグが指定されていない場合、コンパイラが呼び出されたディレクトリを検索します。一般的には、`--proto_path` フラグをプロジェクトのルートに設定し、すべてのインポートに完全修飾名を使用するべきです。

### proto3 メッセージタイプの使用 {#proto3}

[proto3](/programming-guides/proto3) メッセージタイプをインポートして、proto2 メッセージで使用したり、その逆も可能です。ただし、proto2 の列挙型は proto3 構文で使用することはできません。

## ネストしたタイプ {#nested}

他のメッセージタイプ内でメッセージタイプを定義して使用することができます。次の例では、`SearchResponse` メッセージ内に `Result` メッセージが定義されています:

```proto
message SearchResponse {
  message Result {
    optional string url = 1;
    optional string title = 2;
    repeated string snippets = 3;
  }
  repeated Result results = 1;
}
```

このメッセージタイプを親メッセージタイプの外で再利用する場合は、`_Parent_._Type_` として参照します：

```proto
message SomeOtherMessage {
  optional SearchResponse.Result result = 1;
}
```

メッセージを好きなだけ深くネストすることができます。以下の例では、2つの `Inner` という名前のネストされた型は、異なるメッセージ内で定義されているため、完全に独立していることに注意してください：

```proto
message Outer {       // Level 0
  message MiddleAA {  // Level 1
    message Inner {   // Level 2
      optional int64 ival = 1;
      optional bool  booly = 2;
    }
  }
  message MiddleBB {  // Level 1
    message Inner {   // Level 2
      optional int32  ival = 1;
      optional bool   booly = 2;
    }
  }
}
```

### グループ {#groups}

**グループ機能は非推奨であり、新しいメッセージタイプを作成する際に使用すべきではありません。代わりにネストされたメッセージタイプを使用してください。**

グループはメッセージ定義内で情報をネストする別の方法です。たとえば、複数の `Result` を含む `SearchResponse` を指定する別の方法は次のとおりです：

```proto
message SearchResponse {
  repeated group Result = 1 {
    optional string url = 1;
    optional string title = 2;
    repeated string snippets = 3;
  }
}
```

グループは単にネストされたメッセージ型とフィールドを1つの宣言に組み合わせるものです。コード内では、このメッセージを、`result` という名前の `Result` 型フィールドを持つかのように扱うことができます（後者の名前は小文字に変換されるため、前者と競合しないようになります）。したがって、この例は以前の `SearchResponse` とまったく同等であることに注意してください。

## メッセージタイプの更新 {#updating}

既存のメッセージタイプがすべてのニーズを満たさなくなった場合（たとえば、メッセージ形式に追加のフィールドが必要になった場合）でも、古い形式で作成されたコードを引き続き使用したい場合は心配しないでください！バイナリワイヤ形式を使用すると、既存のコードを壊すことなくメッセージタイプを簡単に更新できます。

{{% alert title="注意" color="note" %}} JSON や [proto テキスト形式](/reference/protobuf/textformat-spec) を使用してプロトコルバッファメッセージを保存する場合、プロト定義で行うことができる変更は異なります。{{% /alert %}}

[Proto ベストプラクティス](/programming-guides/dos-donts) と以下のルールを確認してください：

*   既存のフィールドのフィールド番号を変更しないでください。フィールド番号を「変更」することは、フィールドを削除して同じ型の新しいフィールドを追加することと同等です。フィールド番号を変更したい場合は、[フィールドの削除](#deleting)の手順を参照してください。
*   追加する新しいフィールドは `optional` または `repeated` である必要があります。これにより、古いメッセージ形式を使用してシリアル化されたメッセージが、新しい生成されたコードによってパースされる際に `required` 要素が欠落していないため、新しいコードが適切に古いコードによって生成されたメッセージとやり取りできるようになります。これらの要素の [デフォルト値](#optional) を考慮して、新しいコードが古いコードによって生成されたメッセージと適切にやり取りできるようにしてください。同様に、新しいコードによって作成されたメッセージは、古いコードによってパースされることができ、古いバイナリはパース時に新しいフィールドを単に無視します。ただし、未知のフィールドは破棄されず、メッセージが後でシリアル化されると、未知のフィールドはそれと一緒にシリアル化されます。したがって、メッセージが新しいコードに渡される場合、新しいフィールドは引き続き利用可能です。詳細については、[未知のフィールド](#unknowns)セクションを参照してください。
*   必須でないフィールドは、更新されたメッセージタイプでフィールド番号が再利用されない限り削除できます。フィールド名を変更したり、フィールド番号を [予約](#fieldreserved) して、将来の `.proto` のユーザーが誤って番号を再利用できないようにすることができます。
*   必須でないフィールドを [拡張](#extensions) に変換したり、その逆も可能ですが、型と番号が同じである限りです。
*   `int32`、`uint32`、`int64`、`uint64`、`bool` はすべて互換性があります。つまり、これらの型のフィールドを他の型に変更しても、前方互換性や後方互換性が壊れることはありません。対応する型に収まらない数値がワイヤからパースされた場合、C++ でその型に数値をキャストした場合と同じ効果が得られます（たとえば、64ビット数値が int32 として読み取られると、32ビットに切り捨てられます）。
*   `sint32` と `sint64` は互換性がありますが、他の整数型とは互換性がありません。
*   `string` と `bytes` は、バイトが有効な UTF-8 である限り互換性があります。
*   埋め込まれたメッセージは、バイトがメッセージのエンコードされたバージョンを含んでいる場合、`bytes` と互換性があります。
*   `fixed32` は `sfixed32` と、`fixed64` は `sfixed64` と互換性があります。
*   `string`、`bytes`、およびメッセージフィールドに対して、`optional` は `repeated` と互換性があります。繰り返しフィールドのシリアル化データを入力として受け取るクライアントは、このフィールドを `optional` として期待する場合、スカラータイプフィールドの場合は最後の入力値を取り、メッセージタイプフィールドの場合はすべての入力要素をマージします。これは、数値型を含む場合（ブール値や列挙型を含む）には **一般的に安全ではない** ことに注意してください。数値型の繰り返しフィールドは、[packed](/programming-guides/encoding#packed) 形式でシリアル化される可能性があり、`optional` フィールドが期待される場合には正しくパースされません。
*   デフォルト値を変更することは一般的に問題ありませんが、デフォルト値は決してワイヤを介して送信されません。したがって、プログラムが特定のフィールドが設定されていないメッセージを受信した場合、そのプログラムはそのプロトコルのバージョンで定義されたデフォルト値を見ることになります。送信元のコードで定義されたデフォルト値は見えません。 
*   `enum` は `int32`、`uint32`、`int64`、`uint64` とワイヤ形式で互換性があります（値が収まらない場合は切り捨てられます）。ただし、メッセージがデシリアライズされるときにクライアントコードが異なる方法で扱う可能性があることに注意してください。特に、認識されない `enum` 値はメッセージがデシリアライズされるときに破棄されるため、フィールドの `has..` アクセサは false を返し、そのゲッターは `enum` 定義で最初にリストされた値、またはデフォルト値が指定されている場合はデフォルト値を返します。繰り返しの列挙型フィールドの場合、認識されない値はリストから削除されます。ただし、整数フィールドは常に値を保持します。そのため、整数を `enum` にアップグレードする際に、ワイヤ上で範囲外の `enum` 値を受け取る可能性があることに注意する必要があります。
*   現在の Java および C++ の実装では、認識されない `enum` 値が削除されると、他の未知のフィールドと一緒に格納されます。これは、このデータがシリアル化され、クライアントがこれらの値を認識する場合に奇妙な動作を引き起こす可能性があることに注意してください。オプションフィールドの場合、元のメッセージがデシリアライズされた後に新しい値が書き込まれた場合でも、古い値は引き続き認識されることになります。繰り返しフィールドの場合、認識された値と新たに追加された値の後に古い値が表示されるため、順序が保持されません。
*   `optional` フィールドまたは拡張を `oneof` のメンバーに変更することはバイナリ互換性がありますが、一部の言語（特に Go）では生成されたコードの API が非互換な方法で変更される可能性があります。このため、Google は [AIP-180](https://google.aip.dev/180#moving-into-oneofs) で文書化されているように、公開 API でこのような変更を行いません。ソース互換性については同じ警告がありますが、複数のフィールドを新しい `oneof` に移動することは、1回に1つ以上のフィールドを設定するコードがないことを確認すれば安全かもしれません。既存の `oneof` にフィールドを移動することは安全ではありません。同様に、単一のフィールド `oneof` を `optional` フィールドまたは拡張に変更することは安全です。
*   `map<K, V>` と対応する `repeated` メッセージフィールドの間でフィールドを変更することはバイナリ互換性があります（メッセージのレイアウトやその他の制限については、以下の [Maps](#maps) を参照）。ただし、変更の安全性はアプリケーションに依存します。メッセージをデシリアライズして再シリアライズする際、`repeated` フィールド定義を使用するクライアントは意味的に同一の結果を生成します。ただし、`map` フィールド定義を使用するクライアントはエントリを並べ替えたり、重複キーを持つエントリを削除したりする可能性があります。

## 未知のフィールド {#unknowns}

未知のフィールドは、パーサーが認識しないフィールドを表す、適切に形成されたプロトコルバッファシリアライズされたデータです。たとえば、新しいフィールドを持つ新しいバイナリが送信したデータを古いバイナリが解析するとき、その新しいフィールドは古いバイナリ内の未知のフィールドとなります。

元々、proto3 メッセージは、解析中に未知のフィールドを常に破棄していましたが、バージョン 3.5 では、proto2 の動作に合わせて未知のフィールドの保存を再導入しました。バージョン 3.5 以降では、解析中に未知のフィールドが保持され、シリアライズされた出力に含まれます。

## 拡張 {#extensions}

拡張とは、通常、コンテナメッセージの `.proto` ファイルとは別の `.proto` ファイルで定義されたフィールドです。

### 拡張を使用する理由 {#why-ext}

拡張を使用する主な理由は次の2つです：

*   コンテナメッセージの `.proto` ファイルには、より少ないインポート/依存関係があります。これにより、ビルド時間が短縮され、循環依存関係が解消され、疎結合が促進されます。拡張はこれに非常に適しています。
*   システムが最小限の依存関係と調整でコンテナメッセージにデータを添付できるようにします。拡張はこれには適していません。フィールド番号のスペースが限られているため、[フィールド番号の再利用の影響](#consequences)があります。非常に多くの拡張に対して非常に低い調整が必要なユースケースの場合は、代わりに[`Any` メッセージタイプ](/reference/protobuf/google.protobuf#any)を使用することを検討してください。

### 拡張の例 {#ext-example}

例として、次のような拡張を見てみましょう：

```proto
// file kittens/video_ext.proto

import "kittens/video.proto";
import "media/user_content.proto";

package kittens;

// This extension allows kitten videos in a media.UserContent message.
extend media.UserContent {
  // Video is a message imported from kittens/video.proto
  repeated Video kitten_videos = 126;
}
```

拡張を定義するファイル（`kittens/video_ext.proto`）は、コンテナメッセージのファイル（`media/user_content.proto`）をインポートしています。

コンテナメッセージは、拡張のために一部のフィールド番号を予約する必要があります。

```proto
// file media/user_content.proto

package media;

// A container message to hold stuff that a user has created.
message UserContent {
  // Set verification to `DECLARATION` to enforce extension declarations for all
  // extensions in this range.
  extensions 100 to 199 [verification = DECLARATION];
}
```

コンテナメッセージのファイル（`media/user_content.proto`）は、メッセージ `UserContent` を定義し、拡張のためにフィールド番号 [100 から 199] を予約しています。範囲に対して `verification = DECLARATION` を設定して、すべての拡張に宣言が必要になるようにすることが推奨されています。

新しい拡張子 (`kittens/video_ext.proto`) が追加された場合、対応する宣言を `UserContent` に追加し、`verification` を削除する必要があります。

```
// A container message to hold stuff that a user has created.
message UserContent {
  extensions 100 to 199 [
    declaration = {
      number: 126,
      full_name: ".kittens.kitten_videos",
      type: ".kittens.Video",
      repeated: true
    },
    // Ensures all field numbers in this extension range are declarations.
    verification = DECLARATION
  ];
}
```

`UserContent` は、フィールド番号 `126` が、完全修飾名 `.kittens.kitten_videos` および完全修飾型 `.kittens.Video` を持つ `repeated` 拡張フィールドによって使用されることを宣言しています。拡張の宣言について詳しくは、[拡張の宣言](/programming-guides/extension_declarations) を参照してください。

コンテナメッセージのファイル (`media/user_content.proto`) は、子猫ビデオ拡張の定義 (`kittens/video_ext.proto`) を **インポートしていません**。

拡張フィールドのワイヤーフォーマットエンコーディングは、同じフィールド番号、型、基数を持つ標準フィールドと比較しても違いはありません。そのため、標準フィールドをコンテナから拡張に移動させるか、拡張フィールドを標準フィールドとしてコンテナメッセージに移動させることは安全です。ただし、フィールド番号、型、基数が一定である限りです。

ただし、拡張はコンテナメッセージの外部で定義されているため、特殊なアクセサは特定の拡張フィールドを取得および設定するために生成されません。この例では、protobufコンパイラは `AddKittenVideos()` や `GetKittenVideos()` のアクセサを **生成しません**。代わりに、拡張は `HasExtension()`、`ClearExtension()`、`GetExtension()`、`MutableExtension()`、`AddExtension()` のようなパラメータ化された関数を介してアクセスされます。

C++ では、次のようになります：

```cpp
UserContent user_content;
user_content.AddExtension(kittens::kitten_videos, new kittens::Video());
assert(1 == user_content.GetExtensionCount(kittens::kitten_videos));
user_content.GetExtension(kittens::kitten_videos, 0);
```

### 拡張範囲の定義 {#defining-ranges}

コンテナメッセージの所有者である場合、メッセージへの拡張のために拡張範囲を定義する必要があります。

拡張フィールドに割り当てられたフィールド番号は、標準フィールドに再利用することはできません。

定義された後で拡張範囲を拡張することは安全です。比較的小さな数値を 1000 割り当て、そのスペースを拡張宣言を使用して密に埋めるのが良いデフォルトです：

```proto
message ModernExtendableMessage {
  // All extensions in this range should use extension declarations.
  extensions 1000 to 2000 [verification = DECLARATION];
}
```

拡張宣言の前に実際の拡張のための範囲を追加する場合は、`verification = DECLARATION` を追加して、この新しい範囲に宣言が使用されるように強制する必要があります。実際の宣言が追加されると、このプレースホルダを削除できます。

既存の拡張範囲を同じ合計範囲をカバーする複数の範囲に分割することは安全です。これは、レガシー メッセージ型を[拡張宣言](/programming-guides/extension_declarations)に移行する際に必要になる場合があります。たとえば、移行前に範囲が次のように定義されているかもしれません：

```proto
message LegacyMessage {
  extensions 1000 to max;
}
```

そして、移行後（範囲を分割）は次のようになります：

```proto
message LegacyMessage {
  // Legacy range that was using an unverified allocation scheme.
  extensions 1000 to 524999999 [verification = UNVERIFIED];
  // Current range that uses extension declarations.
  extensions 525000000 to max  [verification = DECLARATION];
}
```

拡張範囲を移動または縮小するために開始フィールド番号を増やすことや終了フィールド番号を減らすことは安全ではありません。これらの変更は既存の拡張を無効にする可能性があります。

protoのほとんどのインスタンスで設定される標準フィールドには、フィールド番号1から15を使用することをお勧めします。これらの番号を拡張に使用することはお勧めしません。

番号付けの規則が非常に大きなフィールド番号を持つ拡張を含む可能性がある場合は、`max` キーワードを使用して拡張範囲が可能な限り最大のフィールド番号までであることを指定できます：

```proto
message Foo {
  extensions 1000 to max;
}
```

`max` は 2<sup>29</sup> - 1、つまり 536,870,911 です。

### 拡張番号の選択 {#choosing}

拡張は、コンテナメッセージの外で指定できるフィールドです。拡張フィールド番号には、[フィールド番号の割り当て](#assigning)に関するすべてのルールが適用されます。同じ[フィールド番号の再利用の結果](#consequences)も拡張フィールド番号の再利用に適用されます。

一意の拡張フィールド番号を選択するのは簡単です。コンテナメッセージが[拡張宣言](/programming-guides/extension_declarations)を使用している場合、新しい拡張を定義する際には、コンテナメッセージで定義された最も高い拡張範囲の他のすべての宣言よりも高い最低のフィールド番号を選択します。たとえば、コンテナメッセージが次のように定義されている場合：

```proto
message Container {
  // Legacy range that was using an unverified allocation scheme
  extensions 1000 to 524999999;
  // Current range that uses extension declarations. (highest extension range)
  extensions 525000000 to max  [
    declaration = {
      number: 525000001,
      full_name: ".bar.baz_ext",
      type: ".bar.Baz"
    }
    // 525,000,002 is the lowest field number above all other declarations
  ];
}
```

`Container`の次の拡張機能には、新しい宣言として番号`525000002`を追加する必要があります。

#### 未検証の拡張番号割り当て（推奨されない）{#unverified}

コンテナメッセージの所有者は、自身の未検証の拡張番号割り当て戦略を優先して、拡張宣言を省略することができます。

未検証の割り当てスキームは、選択した拡張範囲内で拡張フィールド番号を割り当てるために、protobufエコシステム外部のメカニズムを使用します。1つの例として、モノレポのコミット番号を使用することができます。このシステムは、protobufコンパイラの観点からは「未検証」であり、拡張フィールド番号が適切に取得されたものであるかどうかを確認する方法がないためです。

拡張宣言のような検証済みシステムに比べて未検証システムの利点は、コンテナメッセージの所有者との調整なしに拡張を定義できることです。

未検証システムのデメリットは、protobufコンパイラが参加者を拡張フィールド番号の再利用から保護できないことです。

**未検証の拡張フィールド番号割り当て戦略は推奨されません**。なぜなら、[フィールド番号の再利用の結果](#consequences)がメッセージのすべての拡張者に影響するためです（推奨事項に従わなかった開発者だけでなく）。非常に低い調整が必要なユースケースの場合は、代わりに[`Any`メッセージ](/reference/protobuf/google.protobuf#any)を使用することを検討してください。

未検証の拡張フィールド番号割り当て戦略は、1から524,999,999の範囲に限定されます。フィールド番号525,000,000以上は拡張宣言でのみ使用できます。

### 拡張タイプの指定 {#specifying-extension-types}

拡張は、`oneof`や`map`を除く任意のフィールドタイプであることができます。

### ネストされた拡張（推奨されない）{#nested-exts}

別のメッセージのスコープ内で拡張を宣言することができます：

```proto
import "common/user_profile.proto";

package puppies;

message Photo {
  extend common.UserProfile {
    optional int32 likes_count = 111;
  }
  ...
}
```

この場合、この拡張にアクセスするためのC++コードは次のようになります：

```cpp
UserProfile user_profile;
user_profile.SetExtension(puppies::Photo::likes_count, 42);
```

言い換えると、`likes_count` は `puppies.Photo` のスコープ内で定義されています。

これはよくある混乱の原因です: メッセージ型の内部に `extend` ブロックをネストして宣言することは、外部型と拡張型の間に関係があることを意味しません。特に、前述の例では `Photo` が `UserProfile` のサブクラスであることを意味しません。単に、`likes_count` シンボルが `Photo` のスコープ内で宣言されていることを意味します。それは単に静的メンバーです。

一般的なパターンは、拡張を拡張のフィールド型のスコープ内に定義することです - 例えば、ここでは、`media.UserContent` の型 `puppies.Photo` に対する拡張が `Photo` の一部として定義されています:

```proto
import "media/user_content.proto";

package puppies;

message Photo {
  extend media.UserContent {
    optional Photo puppy_photo = 127;
  }
  ...
}
```

ただし、メッセージ型を持つ拡張をその型内で定義する必要はありません。標準の定義パターンも使用できます:

```proto
import "media/user_content.proto";

package puppies;

message Photo {
  ...
}

// This can even be in a different file.
extend media.UserContent {
  optional Photo puppy_photo = 127;
}
```

**標準（ファイルレベル）構文が推奨**されており、混乱を避けるために使用されます。ネストされた構文は、拡張について既に馴染みのないユーザーによってしばしばサブクラス化と誤解されます。

## Any {#any}

`Any` メッセージ型を使用すると、`.proto` 定義を持たないメッセージを埋め込み型として使用できます。`Any` は、`bytes` として任意のシリアライズされたメッセージを含み、そのメッセージのタイプを一意に識別し、解決するためのグローバルに一意な識別子として機能する URL を含みます。`Any` 型を使用するには、`google/protobuf/any.proto` を[import](#other)する必要があります。

```proto
import "google/protobuf/any.proto";

message ErrorStatus {
  string message = 1;
  repeated google.protobuf.Any details = 2;
}
```

特定のメッセージ型に対するデフォルトのタイプ URL は `type.googleapis.com/_packagename_._messagename_` です。

異なる言語の実装では、`Any` 値を型安全な方法でパックおよびアンパックするためのランタイムライブラリヘルパーをサポートします - たとえば、Java では、`Any` 型には特別な `pack()` および `unpack()` アクセサがあり、C++ では `PackFrom()` および `UnpackTo()` メソッドがあります:

```c++
// Storing an arbitrary message type in Any.
NetworkErrorDetails details = ...;
ErrorStatus status;
status.add_details()->PackFrom(details);

// Reading an arbitrary message from Any.
ErrorStatus status = ...;
for (const google::protobuf::Any& detail : status.details()) {
  if (detail.Is<NetworkErrorDetails>()) {
    NetworkErrorDetails network_error;
    detail.UnpackTo(&network_error);
    ... processing network_error ...
  }
}
```

**現在、`Any` 型を扱うためのランタイムライブラリは開発中**です。

含まれるメッセージを少数のタイプに制限し、リストに新しいタイプを追加する前に許可を要求する場合は、`Any` メッセージ型の代わりに[extensions](#extensions)と[extension declarations](/programming-guides/extension_declarations)を使用することを検討してください。

## Oneof {#oneof}

多くのオプションフィールドを持ち、同時に設定されるフィールドが最大で1つであるメッセージがある場合、oneof機能を使用してこの動作を強制し、メモリを節約できます。

Oneofフィールドは、すべてのフィールドが1つのメモリを共有し、同時に最大で1つのフィールドが設定できるオプションフィールドのようです。oneofのメンバーのいずれかを設定すると、自動的に他のメンバーがすべてクリアされます。どの値がoneofに設定されているか（あれば）は、選択した言語に応じて特別な `case()` または `WhichOneof()` メソッドを使用して確認できます。

*複数の値が設定されている場合、proto内の順序によって決定された最後に設定された値がすべて以前の値を上書きします*。

oneofフィールドのフィールド番号は、包含メッセージ内で一意である必要があります。

### Oneofの使用 {#using-oneof}

`.proto` でoneofを定義するには、`oneof` キーワードに続いてoneof名を指定します。この場合は `test_oneof` です。

```proto
message SampleMessage {
  oneof test_oneof {
     string name = 4;
     SubMessage sub_message = 9;
  }
}
```

次に、oneof定義にoneofフィールドを追加します。`map` フィールド以外の任意のタイプのフィールドを追加できますが、`required`、`optional`、または `repeated` キーワードは使用できません。oneofに繰り返しフィールドを追加する必要がある場合は、繰り返しフィールドを含むメッセージを使用できます。

生成されたコードでは、oneofフィールドは通常の `optional` フィールドと同じゲッターとセッターを持ちます。また、oneof内で設定された値（あれば）を確認するための特別なメソッドも取得できます。選択した言語に関するoneof APIについては、関連する[APIリファレンス](/reference/)で詳細を確認できます。

### Oneofの機能 {#oneof-features}

*   oneofフィールドを設定すると、oneofの他のメンバーが自動的にクリアされます。したがって、複数のoneofフィールドを設定した場合、最後に設定したフィールドのみが値を持ち続けます。

    ```cpp
    SampleMessage message;
    message.set_name("name");
    CHECK(message.has_name());
    // Calling mutable_sub_message() will clear the name field and will set
    // sub_message to a new instance of SubMessage with none of its fields set.
    message.mutable_sub_message();
    CHECK(!message.has_name());
    ```

*   パーサーがワイヤー上で同じoneofの複数のメンバーに遭遇した場合、解析されたメッセージで使用されるのは最後に見たメンバーのみです。

*   拡張はoneofにはサポートされていません。

*   oneofは `repeated` にすることはできません。

*   oneofフィールドに対してはリフレクションAPIが機能します。

*   ワンオフフィールドをデフォルト値に設定する場合（たとえば、int32のワンオフフィールドを0に設定する場合）、そのワンオフフィールドの「case」が設定され、値がワイヤー上でシリアル化されます。

*   C++を使用している場合は、コードがメモリクラッシュを引き起こさないようにしてください。次のサンプルコードはクラッシュします。なぜなら、`sub_message` が `set_name()` メソッドを呼び出すことで既に削除されていたからです。

    ```c++
    SampleMessage message;
    SubMessage* sub_message = message.mutable_sub_message();
    message.set_name("name");      // Will delete sub_message
    sub_message->set_...            // Crashes here
    ```

*   再びC++で、ワンオフを持つ2つのメッセージを`Swap()`すると、各メッセージがもう一方のワンオフケースを持つようになります。以下の例では、`msg1` には `sub_message` が、`msg2` には `name` が含まれます。

    ```c++
    SampleMessage msg1;
    msg1.set_name("name");
    SampleMessage msg2;
    msg2.mutable_sub_message();
    msg1.swap(&msg2);
    CHECK(msg1.has_sub_message());
    CHECK(msg2.has_name());
    ```

### 逆互換性の問題 {#backward}

ワンオフフィールドを追加または削除する際には注意してください。ワンオフの値をチェックして`None`/`NOT_SET`が返される場合、それはワンオフが設定されていないか、異なるバージョンのワンオフのフィールドに設定されている可能性があることを意味します。ワイヤー上の未知のフィールドがワンオフのメンバーであるかどうかを知る方法がないため、違いを判断する方法はありません。

#### タグ再利用の問題 {#reuse}

*   **オプションフィールドをワンオフに移動または移動させる**: メッセージがシリアル化および解析された後、一部の情報が失われる可能性があります（一部のフィールドがクリアされます）。ただし、1つのフィールドを**新しい**ワンオフに安全に移動でき、1つだけが設定されることがわかっている場合は複数のフィールドを移動できるかもしれません。詳細については、[メッセージタイプの更新](#updating)を参照してください。
*   **ワンオフフィールドを削除して追加する**: メッセージがシリアル化および解析された後、現在設定されているワンオフフィールドがクリアされる可能性があります。
*   **ワンオフの分割または統合**: これは通常の`optional`フィールドを移動する場合と同様の問題があります。

## マップ {#maps}

データ定義の一部として連想マップを作成したい場合、プロトコルバッファは便利なショートカット構文を提供します：

```proto
map<key_type, value_type> map_field = N;
```

...ここで、`key_type` は任意の整数型または文字列型（つまり、浮動小数点型や`bytes`を除く[スカラー](#scalar)型）であることに注意してください。列挙型は有効な`key_type`ではありません。`value_type` は別のマップ以外の任意の型であることができます。

したがって、例えば、各 `Project` メッセージが文字列キーに関連付けられるプロジェクトのマップを作成したい場合は、次のように定義できます。

```proto
map<string, Project> projects = 3;
```

### マップの特徴 {#maps-features}

*   マップでは拡張はサポートされていません。
*   マップは `repeated`、`optional`、または `required` にすることはできません。
*   マップのワイヤーフォーマットの順序とマップの値の反復順序は未定義であり、特定の順序でマップアイテムがあるとは依存できません。
*   `.proto` のテキストフォーマットを生成する際、マップはキーでソートされます。数値キーは数値順にソートされます。
*   ワイヤーからのパースやマージ時に、重複するマップキーがある場合、最後に見たキーが使用されます。テキストフォーマットからマップをパースする際、重複するキーがあるとパースに失敗する可能性があります。

生成されたマップAPIは現在、すべてのサポートされている言語で利用可能です。選択した言語に関するマップAPIの詳細については、関連する[APIリファレンス](/reference/)で確認できます。

### 逆互換性 {#backwards}

マップ構文は、以下のワイヤー上での同等の表現と同等です。したがって、マップをサポートしていないプロトコルバッファの実装でもデータを処理できます。

```proto
message MapFieldEntry {
  optional key_type key = 1;
  optional value_type value = 2;
}

repeated MapFieldEntry map_field = N;
```

マップをサポートするプロトコルバッファの実装は、以前の定義で受け入れられるデータを生成および受け入れる必要があります。

## パッケージ {#packages}

プロトコルメッセージタイプ間の名前の衝突を防ぐために、`.proto` ファイルにオプションの `package` 指定子を追加できます。

```proto
package foo.bar;
message Open { ... }
```

その後、メッセージタイプのフィールドを定義する際にパッケージ指定子を使用できます。

```proto
message Foo {
  ...
  optional foo.bar.Open open = 1;
  ...
}
```

パッケージ指定子が生成されたコードに与える影響は、選択した言語に依存します。

*   **C++** では、生成されたクラスはC++の名前空間内にラップされます。例えば、`Open` は `foo::bar` の名前空間にあります。
*   **Java** および **Kotlin** では、パッケージはJavaパッケージとして使用されますが、`.proto` ファイルで明示的に `option java_package` を指定しない限りです。
*   **Python** では、`package` ディレクティブは無視されます。Pythonモジュールはファイルシステム内の場所に応じて構成されます。
*   **Go** では、`package` ディレクティブは無視され、生成された `.pb.go` ファイルは、対応する `go_proto_library` Bazelルールの名前で指定されたパッケージになります。オープンソースプロジェクトでは、`go_package` オプションを提供するか、Bazelの `-M` フラグを設定する必要があります。
*   **Ruby** では、生成されたクラスはネストされたRuby名前空間内にラップされ、必要なRubyの大文字化スタイルに変換されます（最初の文字が大文字になります。最初の文字が文字でない場合は、`PB_` が先頭に追加されます）。例えば、`Open` は `Foo::Bar` の名前空間にあります。
*   **C#** では、パッケージは、PascalCaseに変換した後の名前空間として使用されますが、`.proto` ファイルで明示的に `option csharp_namespace` を指定しない限りです。例えば、`Open` は `Foo.Bar` の名前空間にあります。

注意してください。`package` ディレクティブが生成されるコードに直接影響を与えない場合でも、例えば Python の場合、`.proto` ファイルのパッケージを指定することを強くお勧めします。そうしないと、記述子での名前の競合が発生し、他の言語にとってプロトが移植可能でなくなる可能性があります。

### パッケージと名前解決 {#name-resolution}

プロトコルバッファ言語における型名の解決は C++ のように動作します。最初に最も内側のスコープが検索され、次にその内側のスコープが検索され、そのように続きます。各パッケージは親パッケージに対して「内側」であると見なされます。先頭に '.'（例: `.foo.bar.Baz`）がある場合、最も外側のスコープから開始することを意味します。

プロトコルバッファコンパイラは、インポートされた `.proto` ファイルを解析してすべての型名を解決します。各言語のコード生成ツールは、異なるスコープルールを持っていても、その言語で各型を参照する方法を知っています。

## サービスの定義 {#services}

メッセージ型を RPC（Remote Procedure Call）システムで使用したい場合、`.proto` ファイルに RPC サービスインターフェースを定義し、プロトコルバッファコンパイラは選択した言語でサービスインターフェースコードとスタブを生成します。例えば、`SearchRequest` を受け取り `SearchResponse` を返すメソッドを持つ RPC サービスを定義したい場合、`.proto` ファイルに次のように定義します。

```proto
service SearchService {
  rpc Search(SearchRequest) returns (SearchResponse);
}
```

デフォルトでは、プロトコルコンパイラは `SearchService` という抽象インターフェースと対応する "スタブ" 実装を生成します。スタブはすべての呼び出しを `RpcChannel` に転送します。`RpcChannel` はあなた自身の RPC システムに関して自分で定義する必要がある抽象インターフェースです。例えば、メッセージをシリアライズして HTTP 経由でサーバーに送信する `RpcChannel` を実装するかもしれません。言い換えれば、生成されたスタブは、特定の RPC 実装に固定されることなく、プロトコルバッファベースの RPC 呼び出しを行うための型安全なインターフェースを提供します。そのため、C++ では次のようなコードが生成されるかもしれません。

```c++
using google::protobuf;

protobuf::RpcChannel* channel;
protobuf::RpcController* controller;
SearchService* service;
SearchRequest request;
SearchResponse response;

void DoSearch() {
  // You provide classes MyRpcChannel and MyRpcController, which implement
  // the abstract interfaces protobuf::RpcChannel and protobuf::RpcController.
  channel = new MyRpcChannel("somehost.example.com:1234");
  controller = new MyRpcController;

  // The protocol compiler generates the SearchService class based on the
  // definition given earlier.
  service = new SearchService::Stub(channel);

  // Set up the request.
  request.set_query("protocol buffers");

  // Execute the RPC.
  service->Search(controller, &request, &response,
                  protobuf::NewCallback(&Done));
}

void Done() {
  delete service;
  delete channel;
  delete controller;
}
```

すべてのサービスクラスは、特定のメソッド名やその入出力タイプをコンパイル時に知ることなく、特定のメソッドを呼び出す方法を提供する `Service` インターフェースも実装します。サーバーサイドでは、これを使用して RPC サーバーを実装し、サービスを登録できます。

```c++
using google::protobuf;

class ExampleSearchService : public SearchService {
 public:
  void Search(protobuf::RpcController* controller,
              const SearchRequest* request,
              SearchResponse* response,
              protobuf::Closure* done) {
    if (request->query() == "google") {
      response->add_result()->set_url("http://www.google.com");
    } else if (request->query() == "protocol buffers") {
      response->add_result()->set_url("http://protobuf.googlecode.com");
    }
    done->Run();
  }
};

int main() {
  // You provide class MyRpcServer.  It does not have to implement any
  // particular interface; this is just an example.
  MyRpcServer server;

  protobuf::Service* service = new ExampleSearchService;
  server.ExportOnPort(1234, service);
  server.Run();

  delete service;
  return 0;
}
```

既存の RPC システムをプラグインしない場合は、Google で開発された言語やプラットフォームに依存しないオープンソースの RPC システムである [gRPC](https://github.com/grpc/grpc-common) を使用できます。gRPC は、特に Protocol Buffers と非常にうまく機能し、`.proto` ファイルから関連する RPC コードを特別なプロトコルバッファコンパイラプラグインを使用して直接生成できます。ただし、proto2 と proto3 で生成されたクライアントとサーバーの間に互換性の問題があるため、gRPC を定義するために proto3 を使用することをお勧めします。proto3 構文について詳しくは、[Proto3 Language Guide](/programming-guides/proto3) を参照してください。proto2 を gRPC で使用したい場合は、プロトコルバッファコンパイラとライブラリのバージョン 3.0.0 以上を使用する必要があります。

gRPC に加えて、Protocol Buffers のための RPC 実装を開発するためのいくつかの第三者プロジェクトも進行中です。知っているプロジェクトへのリンクのリストについては、[third-party add-ons wiki page](https://github.com/protocolbuffers/protobuf/blob/master/docs/third_party.md) を参照してください。

## JSON マッピング {#json}

Proto2 は、JSON での標準エンコーディングをサポートしており、システム間でデータを共有しやすくしています。エンコーディングは以下の表に示されているタイプごとに記述されています。

JSON エンコードされたデータをプロトコルバッファにパースする際、値が欠落しているか、その値が `null` の場合、それは対応する[デフォルト値](#optional)として解釈されます。

`optional` キーワードで定義された proto2 フィールドは、フィールドの存在をサポートします。値が設定されていてフィールドの存在をサポートするフィールドは、デフォルト値であっても、JSON エンコードされた出力に常にフィールド値を含めます。

<table>
  <tbody>
    <tr>
      <th>proto2</th>
      <th>JSON</th>
      <th>JSON の例</th>
      <th>ノート</th>
    </tr>
    <tr>
      <td>message</td>
      <td>object</td>
      <td><code>{"fooBar": v, "g": null, ...}</code></td>
      <td>JSON オブジェクトを生成します。メッセージフィールド名は lowerCamelCase にマッピングされ、JSON オブジェクトのキーになります。`json_name` フィールドオプションが指定されている場合、指定された値がキーとして使用されます。パーサーは、lowerCamelCase 名（または `json_name` オプションで指定された名前）と元の proto フィールド名の両方を受け入れます。`null` はすべてのフィールドタイプの受け入れられる値であり、対応するフィールドタイプのデフォルト値として扱われます。ただし、`json_name` の値には `null` を使用できません。詳細については、[Stricter validation for json_name](/news/2023-04-28#json-name) を参照してください。
      </td>
    </tr>
    <tr>
      <td>enum</td>
      <td>string</td>
      <td><code>"FOO_BAR"</code></td>
      <td>proto で指定された enum 値の名前が使用されます。パーサーは、enum 名と整数値の両方を受け入れます。
      </td>
    </tr>
    <tr>
      <td>map&lt;K,V&gt;</td>
      <td>object</td>
      <td><code>{"k": v, ...}</code></td>
      <td>すべてのキーは文字列に変換されます。
      </td>
    </tr>
    <tr>
      <td>repeated V</td>
      <td>array</td>
      <td><code>[v, ...]</code></td>
      <td><code>null</code> は空リスト <code>[]</code> として受け入れられます。
      </td>
    </tr>
    <tr>
      <td>bool</td>
      <td>true, false</td>
      <td><code>true, false</code></td>
      <td></td>
    </tr>
    <tr>
      <td>string</td>
      <td>string</td>
      <td><code>"Hello World!"</code></td>
      <td></td>
    </tr>
    <tr>
      <td>bytes</td>
      <td>base64 string</td>
      <td><code>"YWJjMTIzIT8kKiYoKSctPUB+"</code></td>
      <td>JSON 値は、パディングを使用した標準ベース64エンコーディングを使用して文字列としてエンコードされます。標準または URL セーフなベース64 エンコーディング（パディングあり/なし）のいずれも受け入れられます。
      </td>
    </tr>
    <tr>
      <td>int32, fixed32, uint32</td>
      <td>number</td>
      <td><code>1, -10, 0</code></td>
      <td>JSON 値は 10 進数になります。数値または文字列のいずれも受け入れられます。
      </td>
    </tr>
    <tr>
      <td>int64, fixed64, uint64</td>
      <td>string</td>
      <td><code>"1", "-10"</code></td>
      <td>JSON 値は 10 進数の文字列になります。数値または文字列のいずれも受け入れられます。
      </td>
    </tr>
    <tr>
      <td>float, double</td>
      <td>number</td>
      <td><code>1.1, -10.0, 0, "NaN", "Infinity"</code></td>
      <td>JSON 値は数値または特別な文字列値 "NaN"、"Infinity"、"-Infinity" のいずれかになります。数値または文字列のいずれも受け入れられます。指数表記も受け入れられます。-0 は 0 と見なされます。
      </td>
    </tr>
    <tr>
      <td>Any</td>
      <td><code>object</code></td>
      <td><code>{"@type": "url", "f": v, ... }</code></td>
      <td><code>Any</code> に特別な JSON マッピングを持つ値が含まれている場合、次のように変換されます: <code>{"@type": xxx, "value": yyy}</code>。それ以外の場合、値は JSON オブジェクトに変換され、実際のデータ型を示すために <code>"@type"</code> フィールドが挿入されます。
      </td>
    </tr>
    <tr>
      <td>Timestamp</td>
      <td>string</td>
      <td><code>"1972-01-01T10:00:20.021Z"</code></td>
      <td>生成された出力は常に Z に正規化され、0、3、6、または 9 桁の小数部を使用します。"Z" 以外のオフセットも受け入れられます。
      </td>
    </tr>
    <tr>
      <td>Duration</td>
      <td>string</td>
      <td><code>"1.000340012s", "1s"</code></td>
      <td>生成された出力には常に 0、3、6、または 9 桁の小数部が含まれ、必要な精度に応じて、"s" 接尾辞が続きます。受け入れられるのは、任意の小数部（なしも含む）で、ナノ秒の精度に収まり、"s" 接尾辞が必要です。
      </td>
    </tr>
    <tr>
      <td>Struct</td>
      <td><code>object</code></td>
      <td><code>{ ... }</code></td>
      <td>任意の JSON オブジェクト。詳細は <code>struct.proto</code> を参照してください。
      </td>
    </tr>
    <tr>
      <td>Wrapper types</td>
      <td>various types</td>
      <td><code>2, "2", "foo", true, "true", null, 0, ...</code></td>
      <td>ラッパーは、ラップされたスカラー型と同じ表現を JSON で使用しますが、`null` が許可され、データの変換と転送中に保持されます。
      </td>
    </tr>
    <tr>
      <td>FieldMask</td>
      <td>string</td>
      <td><code>"f.fooBar,h"</code></td>
      <td><code>field_mask.proto</code> を参照してください。
      </td>
    </tr>
    <tr>
      <td>ListValue</td>
      <td>array</td>
      <td><code>[foo, bar, ...]</code></td>
      <td></td>
    </tr>
    <tr>
      <td>Value</td>
      <td>value</td>
      <td></td>
      <td>任意の JSON 値。詳細については、<code><a href="/reference/protobuf/google.protobuf#value">google.protobuf.Value</a></code> を確認してください。
      </td>
    </tr>
    <tr>
      <td>NullValue</td>
      <td>null</td>
      <td></td>
      <td>JSON の null
      </td>
    </tr>
    <tr>
      <td>Empty</td>
      <td>object</td>
      <td><code>{}</code></td>
      <td>空の JSON オブジェクト
      </td>
    </tr>
  </tbody>
</table>

### JSON オプション {#json-options}

proto2 の JSON 実装では、以下のオプションが提供される場合があります:

- **常に存在しないフィールドを出力**: 存在しないフィールドで、デフォルト値を持つフィールドは、JSON 出力ではデフォルトで省略されます（たとえば、0 の値を持つ暗黙の存在整数、空の文字列である暗黙の存在文字列フィールド、空の繰り返しフィールドやマップフィールド）。実装によっては、この動作をオーバーライドしてデフォルト値を持つフィールドを出力するオプションを提供する場合があります。

    v25.x では、C++、Java、および Python の実装は、このフラグが proto2 の `optional` フィールドに影響を与えるが、proto3 の `optional` フィールドには影響を与えないため、準拠していません。将来のリリースで修正が予定されています。

- **未知のフィールドを無視**: Proto2 の JSON パーサは、デフォルトで未知のフィールドを拒否するべきですが、パーシング時に未知のフィールドを無視するオプションを提供する場合があります。

- **lowerCamelCase 名の代わりに proto フィールド名を使用**: デフォルトでは、proto2 の JSON プリンタはフィールド名を lowerCamelCase に変換してそれを JSON 名として使用するべきです。実装によっては、代わりに proto フィールド名を JSON 名として使用するオプションを提供する場合があります。Proto2 の JSON パーサは、変換された lowerCamelCase 名と proto フィールド名の両方を受け入れる必要があります。

- **列挙値を文字列ではなく整数として出力**: 列挙値の名前がデフォルトで JSON 出力に使用されます。代わりに列挙値の数値を使用するオプションが提供される場合があります。

## オプション {#options}

`.proto` ファイル内の個々の宣言には、複数の *オプション* を付与することができます。オプションは宣言の全体的な意味を変更しませんが、特定のコンテキストでの扱い方に影響を与える場合があります。利用可能なオプションの完全なリストは [`/google/protobuf/descriptor.proto`](https://github.com/protocolbuffers/protobuf/blob/main/src/google/protobuf/descriptor.proto) で定義されています。

一部のオプションはファイルレベルのオプションであり、メッセージ、列挙型、またはサービス定義の内部ではなく、トップレベルスコープに記述する必要があります。一部のオプションはメッセージレベルのオプションであり、メッセージ定義の内部に記述する必要があります。一部のオプションはフィールドレベルのオプションであり、フィールド定義の内部に記述する必要があります。オプションは列挙型、列挙値、oneof フィールド、サービス型、およびサービスメソッドにも記述することができますが、これらのいずれに対しても有用なオプションは現在存在しません。

ここでは、最も一般的に使用されるいくつかのオプションがあります：

- `java_package`（ファイルオプション）：生成されたJava/Kotlinクラスで使用するパッケージです。`.proto`ファイルで明示的な`java_package`オプションが指定されていない場合、デフォルトでprotoパッケージ（`.proto`ファイルで "package" キーワードを使用して指定されたもの）が使用されます。ただし、protoパッケージは一般的に逆ドメイン名で始まることが期待されていないため、protoパッケージは良いJavaパッケージとは言えません。JavaまたはKotlinコードを生成しない場合、このオプションは影響しません。

    ```proto
    option java_package = "com.example.foo";
    ```

- `java_outer_classname`（ファイルオプション）：生成するラッパーJavaクラスのクラス名（およびしたがってファイル名）。`.proto`ファイルで明示的な`java_outer_classname`が指定されていない場合、クラス名は`.proto`ファイル名をキャメルケースに変換して構築されます（つまり、`foo_bar.proto`は`FooBar.java`になります）。`java_multiple_files`オプションが無効になっている場合、`.proto`ファイルに生成される他のすべてのクラス/列挙型などは、この外部のラッパーJavaクラス内にネストされたクラス/列挙型などとして生成されます。Javaコードを生成しない場合、このオプションは影響しません。

    ```proto
    option java_outer_classname = "Ponycopter";
    ```

- `java_multiple_files`（ファイルオプション）：falseの場合、この`.proto`ファイルに対して単一の`.java`ファイルが生成され、トップレベルのメッセージ、サービス、および列挙型に生成されたすべてのJavaクラス/列挙型などが外部クラス（`java_outer_classname`を参照）内にネストされます。trueの場合、トップレベルのメッセージ、サービス、および列挙型に生成された各Javaクラス/列挙型などに対して別々の`.java`ファイルが生成され、この`.proto`ファイルに対して生成されるラッパーJavaクラスにはネストされたクラス/列挙型などが含まれません。これはデフォルトで`false`に設定されるブールオプションです。Javaコードを生成しない場合、このオプションは影響しません。

    ```proto
    option java_multiple_files = true;
    ```

*   `optimize_for`（ファイルオプション）：`SPEED`、`CODE_SIZE`、または`LITE_RUNTIME`に設定できます。これは、C++およびJavaのコードジェネレータ（およびサードパーティのジェネレータ）に以下のように影響します：

    *   `SPEED`（デフォルト）：プロトコルバッファコンパイラは、メッセージタイプに対するシリアル化、パース、およびその他の一般的な操作のためのコードを生成します。このコードは高度に最適化されています。
    *   `CODE_SIZE`：プロトコルバッファコンパイラは最小限のクラスを生成し、共有されたリフレクションベースのコードに依存してシリアル化、パース、およびさまざまな他の操作を実装します。生成されるコードは`SPEED`よりもはるかに小さくなりますが、操作は遅くなります。クラスは引き続き`SPEED`モードと同じパブリックAPIを正確に実装します。このモードは、非常に多くの`.proto`ファイルを含むアプリで、すべてを極めて高速にする必要がない場合に最も有用です。
    *   `LITE_RUNTIME`：プロトコルバッファコンパイラは、"lite"ランタイムライブラリ（`libprotobuf-lite`ではなく`libprotobuf`）にのみ依存するクラスを生成します。ライトランタイムはフルライブラリよりもはるかに小さく（約1桁小さい）ですが、ディスクリプタやリフレクションなどの一部の機能が省略されています。これは、モバイル電話などの制約のあるプラットフォームで実行されるアプリに特に有用です。コンパイラは引き続き、`SPEED`モードと同様にすべてのメソッドの高速な実装を生成します。生成されたクラスは、各言語で`MessageLite`インターフェースのみを実装し、完全な`Message`インターフェースのメソッドのサブセットのみを提供します。

    ```proto
    option optimize_for = CODE_SIZE;
    ```

*   `cc_generic_services`、`java_generic_services`、`py_generic_services`（ファイルオプション）：**ジェネリックサービスは非推奨です。**プロトコルバッファコンパイラが、C++、Java、およびPythonのそれぞれで[サービス定義](#services)に基づいた抽象サービスコードを生成するかどうか。遺産的な理由から、これらはデフォルトで`true`になっています。ただし、2010年1月のバージョン2.3.0以降では、RPC実装が各システムに特化したコードを生成するために、"抽象"サービスに依存するのではなく、
    [コードジェネレータプラグイン](/reference/cpp/api-docs/google.protobuf.compiler.plugin.pb)
    を提供することが好ましいと考えられています。

```proto
    // This file relies on plugins to generate service code.
    option cc_generic_services = false;
    option java_generic_services = false;
    option py_generic_services = false;
    ```

*   `cc_enable_arenas`（ファイルオプション）: C++ 生成コードのために
    [アリーナ割り当て](/reference/cpp/arenas)を有効にします。

*   `objc_class_prefix`（ファイルオプション）: この.proto から生成されたすべての Objective-C クラスおよび列挙型に先頭に付けられる Objective-C クラスプレフィックスを設定します。デフォルトはありません。[Apple によって推奨されている](https://developer.apple.com/library/ios/documentation/Cocoa/Conceptual/ProgrammingWithObjectiveC/Conventions/Conventions.html#//apple_ref/doc/uid/TP40011210-CH10-SW4) 3-5 文字の大文字プレフィックスを使用する必要があります。2 文字のプレフィックスはすべて Apple によって予約されていることに注意してください。

*   `message_set_wire_format`（メッセージオプション）: `true` に設定されている場合、メッセージは、Google 内部で使用されている古い形式である `MessageSet` と互換性のある異なるバイナリ形式を使用します。Google 外部のユーザーはおそらくこのオプションを使用する必要はありません。メッセージは、次のように厳密に宣言されている必要があります：

    ```proto
    message Foo {
      option message_set_wire_format = true;
      extensions 4 to max;
    }
    ```

*   `packed`（フィールドオプション）: 基本的な数値型の繰り返しフィールドに `true` が設定されている場合、よりコンパクトな
    [エンコーディング](/programming-guides/encoding#packed)が使用されます。このオプションを使用するデメリットはありません。ただし、バージョン 2.3.0 より前では、予期しない状況でパックされたデータを受信したパーサーは無視していました。したがって、既存のフィールドをパック形式に変更することは、ワイヤー互換性を壊さずにはできませんでした。2.3.0 以降では、パッキング可能なフィールドのパーサーは常に両方の形式を受け入れるため、この変更は安全ですが、古いプログラムを使用している場合は注意してください。

    ```proto
    repeated int32 samples = 4 [packed = true];
    ```

*   `deprecated`（フィールドオプション）: `true` に設定されている場合、フィールドが非推奨であり、新しいコードで使用すべきではないことを示します。ほとんどの言語では、これに実際の効果はありません。Java では、これは `@Deprecated` 注釈になります。C++ では、clang-tidy は非推奨のフィールドが使用されるたびに警告を生成します。将来、他の言語固有のコード生成ツールは、フィールドのアクセサーに非推奨の注釈を生成し、その結果、フィールドを使用しようとするコードをコンパイルする際に警告が発生します。誰も使用していないフィールドであり、新しいユーザーが使用するのを防ぎたい場合は、フィールド宣言を [予約](#fieldreserved) ステートメントで置き換えることを検討してください。

```proto
optional int32 old_field = 6 [deprecated=true];
```

### 列挙値オプション {#enum-value-options}

列挙値オプションはサポートされています。`deprecated` オプションを使用して、もはや使用されない値を示すことができます。また、拡張を使用してカスタムオプションを作成することもできます。

次の例は、これらのオプションを追加する構文を示しています:

```proto
import "google/protobuf/descriptor.proto";

extend google.protobuf.EnumValueOptions {
  optional string string_name = 123456789;
}

enum Data {
  DATA_UNSPECIFIED = 0;
  DATA_SEARCH = 1 [deprecated = true];
  DATA_DISPLAY = 2 [
    (string_name) = "display_value"
  ];
}
```

`string_name` オプションを読み取るための C++ コードは次のようになります:

```cpp
const absl::string_view foo = proto2::GetEnumDescriptor<Data>()
    ->FindValueByName("DATA_DISPLAY")->options().GetExtension(string_name);
```

列挙値やフィールドにカスタムオプションを適用する方法については、[カスタムオプション](#customoptions) を参照してください。

### カスタムオプション {#customoptions}

Protocol Buffers では、独自のオプションを定義して使用することもできます。これはほとんどの人が必要としない **高度な機能** です。オプションは `google/protobuf/descriptor.proto` で定義されたメッセージによって定義されるため、独自のオプションを定義することは、単にそれらのメッセージを[拡張](#extensions)することです。例:

```proto
import "google/protobuf/descriptor.proto";

extend google.protobuf.MessageOptions {
  optional string my_option = 51234;
}

message MyMessage {
  option (my_option) = "Hello world!";
}
```

ここでは、`MessageOptions` を拡張して新しいメッセージレベルのオプションを定義しました。その後、オプションを使用するときは、オプション名を括弧で囲んで拡張であることを示す必要があります。C++ で `my_option` の値を読み取ることができます:

```proto
string value = MyMessage::descriptor()->options().GetExtension(my_option);
```

ここで、`MyMessage::descriptor()->options()` は `MyMessage` のための `MessageOptions` プロトコルメッセージを返します。それからそれからカスタムオプションを読み取ることは、他の[拡張](#extensions)を読み取るのと同じです。

同様に、Javaでは次のように書きます:

```java
String value = MyProtoFile.MyMessage.getDescriptor().getOptions()
  .getExtension(MyProtoFile.myOption);
```

Pythonでは次のようになります:

```python
value = my_proto_file_pb2.MyMessage.DESCRIPTOR.GetOptions()
  .Extensions[my_proto_file_pb2.my_option]
```

カスタムオプションは、Protocol Buffers 言語のあらゆる構成要素に対して定義することができます。以下は、すべての種類のオプションを使用する例です:

```proto
import "google/protobuf/descriptor.proto";

extend google.protobuf.FileOptions {
  optional string my_file_option = 50000;
}
extend google.protobuf.MessageOptions {
  optional int32 my_message_option = 50001;
}
extend google.protobuf.FieldOptions {
  optional float my_field_option = 50002;
}
extend google.protobuf.OneofOptions {
  optional int64 my_oneof_option = 50003;
}
extend google.protobuf.EnumOptions {
  optional bool my_enum_option = 50004;
}
extend google.protobuf.EnumValueOptions {
  optional uint32 my_enum_value_option = 50005;
}
extend google.protobuf.ServiceOptions {
  optional MyEnum my_service_option = 50006;
}
extend google.protobuf.MethodOptions {
  optional MyMessage my_method_option = 50007;
}

option (my_file_option) = "Hello world!";

message MyMessage {
  option (my_message_option) = 1234;

  optional int32 foo = 1 [(my_field_option) = 4.5];
  optional string bar = 2;
  oneof qux {
    option (my_oneof_option) = 42;

    string quux = 3;
  }
}

enum MyEnum {
  option (my_enum_option) = true;

  FOO = 1 [(my_enum_value_option) = 321];
  BAR = 2;
}

message RequestType {}
message ResponseType {}

service MyService {
  option (my_service_option) = FOO;

  rpc MyMethod(RequestType) returns(ResponseType) {
    // Note:  my_method_option has type MyMessage.  We can set each field
    //   within it using a separate "option" line.
    option (my_method_option).foo = 567;
    option (my_method_option).bar = "Some string";
  }
}
```

パッケージ内で定義されたもの以外のパッケージでカスタムオプションを使用する場合は、
タイプ名と同様にオプション名にパッケージ名を付ける必要があります。例えば：

```proto
// foo.proto
import "google/protobuf/descriptor.proto";
package foo;
extend google.protobuf.MessageOptions {
  optional string my_option = 51234;
}
```

```proto
// bar.proto
import "foo.proto";
package bar;
message MyMessage {
  option (foo.my_option) = "Hello world!";
}
```

最後に：カスタムオプションは拡張機能であるため、他のフィールドや拡張機能と同様にフィールド番号を割り当てる必要があります。前述の例では、50000〜99999の範囲のフィールド番号を使用しています。この範囲は個々の組織内での内部利用のために予約されているため、社内アプリケーションで自由にこの範囲の番号を使用できます。ただし、公開アプリケーションでカスタムオプションを使用する場合は、フィールド番号がグローバルに一意であることを確認する必要があります。グローバルに一意なフィールド番号を取得するには、[protobuf global extension registry](https://github.com/protocolbuffers/protobuf/blob/master/docs/options.md) にエントリを追加するためのリクエストを送信してください。通常、1つの拡張番号だけが必要です。サブメッセージにそれらを配置することで、1つの拡張番号で複数のオプションを宣言できます：

```proto
message FooOptions {
  optional int32 opt1 = 1;
  optional string opt2 = 2;
}

extend google.protobuf.FieldOptions {
  optional FooOptions foo_options = 1234;
}

// usage:
message Bar {
  optional int32 a = 1 [(foo_options).opt1 = 123, (foo_options).opt2 = "baz"];
  // alternative aggregate syntax (uses TextFormat):
  optional int32 b = 2 [(foo_options) = { opt1: 123 opt2: "baz" }];
}
```

また、各オプションタイプ（ファイルレベル、メッセージレベル、フィールドレベルなど）には独自の番号空間がありますので、例えば、FieldOptions と MessageOptions の拡張を同じ番号で宣言することができます。

### オプションの保持 {#option-retention}

オプションには、生成されたコードにオプションが保持されるかどうかを制御する *保持* の概念があります。オプションはデフォルトで *ランタイム保持* を持ち、つまり生成されたコードに保持され、生成されたディスクリプタプールでランタイムで表示されます。ただし、`retention = RETENTION_SOURCE` を設定して、オプション（またはオプション内のフィールド）がランタイムで保持されないように指定することもできます。これを *ソース保持* と呼びます。

オプションの保持は、ほとんどのユーザーが気にする必要がない高度な機能ですが、生成されたバイナリに保持するコードサイズのコストを支払わずに特定のオプションを使用したい場合に役立ちます。ソース保持のオプションはまだ `protoc` と `protoc` プラグインに表示されるため、コードジェネレータは動作をカスタマイズするためにそれらを使用できます。

保持は、次のようにオプションに直接設定できます：

```proto
extend google.protobuf.FileOptions {
  optional int32 source_retention_option = 1234
      [retention = RETENTION_SOURCE];
}
```

また、プレーンフィールドに設定することもできます。その場合、そのフィールドがオプション内に現れるときのみ効果があります：

```proto
message OptionsMessage {
  optional int32 source_retention_field = 1 [retention = RETENTION_SOURCE];
}
```

`retention = RETENTION_RUNTIME` を設定することもできますが、これはデフォルトの動作なので効果はありません。メッセージフィールドが `RETENTION_SOURCE` とマークされている場合、そのフィールド全体が削除されます。その内部のフィールドは `RETENTION_RUNTIME` を設定してもオーバーライドできません。

{{% alert title="Note" color="note" %}} Protocol Buffers 22.0 では、オプションの保持に関するサポートはまだ進行中であり、C++ と Java のみがサポートされています。Go は 1.29.0 からサポートされています。Python のサポートは完了していますが、リリースにはまだ含まれていません。{{% /alert %}}

### オプションのターゲット {#option-targets}

フィールドには `targets` オプションがあり、そのフィールドがオプションとして使用されるときに適用できるエンティティの種類を制御します。たとえば、フィールドに `targets = TARGET_TYPE_MESSAGE` がある場合、そのフィールドは enum（または他の非メッセージエンティティ）のカスタムオプションで設定できません。Protoc はこれを強制し、ターゲット制約が違反された場合にエラーを発生させます。

一見すると、この機能は不要に見えるかもしれません。なぜなら、すべてのカスタムオプションは特定のエンティティ用のオプションメッセージの拡張であり、すでにその1つのエンティティにオプションを制限しているからです。ただし、オプションのターゲットは、複数のエンティティタイプに適用される共有オプションメッセージがある場合や、そのメッセージ内の個々のフィールドの使用を制御したい場合に便利です。たとえば：

```proto
message MyOptions {
  optional string file_only_option = 1 [targets = TARGET_TYPE_FILE];
  optional int32 message_and_enum_option = 2 [targets = TARGET_TYPE_MESSAGE,
                                              targets = TARGET_TYPE_ENUM];
}

extend google.protobuf.FileOptions {
  optional MyOptions file_options = 50000;
}

extend google.protobuf.MessageOptions {
  optional MyOptions message_options = 50000;
}

extend google.protobuf.EnumOptions {
  optional MyOptions enum_options = 50000;
}

// OK: this field is allowed on file options
option (file_options).file_only_option = "abc";

message MyMessage {
  // OK: this field is allowed on both message and enum options
  option (message_options).message_and_enum_option = 42;
}

enum MyEnum {
  MY_ENUM_UNSPECIFIED = 0;
  // Error: file_only_option cannot be set on an enum.
  option (enum_options).file_only_option = "xyz";
}
```

## クラスの生成 {#generating}

`.proto` ファイルで定義されたメッセージタイプで作業するために必要な Java、Kotlin、Python、C++、Go、Ruby、Objective-C、または C# コードを生成するには、`.proto` ファイルに対してプロトコルバッファコンパイラ `protoc` を実行する必要があります。コンパイラをインストールしていない場合は、[パッケージをダウンロード](/downloads)して README の手順に従ってください。Go の場合、コンパイラ用の特別なコードジェネレータプラグインもインストールする必要があります。これについては、GitHub の [golang/protobuf](https://github.com/golang/protobuf/) リポジトリで見つけることができ、インストール手順も記載されています。

プロトコルコンパイラは次のように呼び出されます：

```sh
protoc --proto_path=IMPORT_PATH --cpp_out=DST_DIR --java_out=DST_DIR --python_out=DST_DIR --go_out=DST_DIR --ruby_out=DST_DIR --objc_out=DST_DIR --csharp_out=DST_DIR path/to/file.proto
```

- `IMPORT_PATH` は、`import` ディレクティブを解決する際に `.proto` ファイルを探すディレクトリを指定します。省略した場合、現在のディレクトリが使用されます。複数のインポートディレクトリを指定するには、`--proto_path` オプションを複数回渡すことができます。順番に検索されます。`-I=_IMPORT_PATH_` は `--proto_path` の短縮形として使用できます。

- 1つ以上の *出力ディレクティブ* を指定できます：

    - `--cpp_out` は `DST_DIR` に C++ コードを生成します。詳細については、[C++ 生成コードリファレンス](/reference/cpp/cpp-generated) を参照してください。
    - `--java_out` は `DST_DIR` に Java コードを生成します。詳細については、[Java 生成コードリファレンス](/reference/java/java-generated) を参照してください。
    - `--kotlin_out` は `DST_DIR` に追加の Kotlin コードを生成します。詳細については、[Kotlin 生成コードリファレンス](/reference/kotlin/kotlin-generated) を参照してください。
    - `--python_out` は `DST_DIR` に Python コードを生成します。詳細については、[Python 生成コードリファレンス](/reference/python/python-generated) を参照してください。
    - `--go_out` は `DST_DIR` に Go コードを生成します。詳細については、[Go 生成コードリファレンス](/reference/go/go-generated) を参照してください。
    - `--ruby_out` は `DST_DIR` に Ruby コードを生成します。詳細については、[Ruby 生成コードリファレンス](/reference/ruby/ruby-generated) を参照してください。
    - `--objc_out` は `DST_DIR` に Objective-C コードを生成します。詳細については、[Objective-C 生成コードリファレンス](/reference/objective-c/objective-c-generated) を参照してください。
    - `--csharp_out` は `DST_DIR` に C# コードを生成します。詳細については、[C# 生成コードリファレンス](/reference/csharp/csharp-generated) を参照してください。

    さらなる便宜のために、`DST_DIR` が `.zip` または `.jar` で終わる場合、コンパイラは指定された名前の単一の ZIP 形式アーカイブファイルに出力を書き込みます。`.jar` 出力には、Java JAR 仕様で必要とされるマニフェストファイルも付属します。出力アーカイブがすでに存在する場合は上書きされます。

*   入力として1つ以上の`.proto`ファイルを提供する必要があります。複数の`.proto`ファイルを一度に指定できます。ファイルは現在のディレクトリを基準として名前が付けられますが、各ファイルは`IMPORT_PATH`のいずれかに存在する必要があります。これにより、コンパイラがその正規名を決定できます。

## ファイルの場所 {#location}

`.proto`ファイルを他の言語ソースと同じディレクトリに置かないことをお勧めします。プロジェクトのルートパッケージの下に`.proto`ファイル用のサブパッケージ`proto`を作成することを検討してください。

### 言語に依存しない場所であるべき {#location-language-agnostic}

Javaコードを扱う場合、関連する`.proto`ファイルをJavaソースと同じディレクトリに置くと便利です。ただし、他の言語のコードが同じprotoを使用する場合、パスの接頭辞はもはや意味をなさなくなります。そのため、一般的には、`//myteam/mypackage`などの関連する言語に依存しないディレクトリにprotoを配置してください。

このルールの例外は、protoがテスト用など、Javaコンテキストでのみ使用されることが明確な場合です。

## サポートされるプラットフォーム {#platforms}

以下の情報については:

*   サポートされているオペレーティングシステム、コンパイラ、ビルドシステム、およびC++のバージョンについては、[Foundational C++ Support Policy](https://opensource.google/documentation/policies/cplusplus-support)を参照してください。
*   サポートされているPHPのバージョンについては、[Supported PHP versions](https://cloud.google.com/php/getting-started/supported-php-versions)を参照してください。
