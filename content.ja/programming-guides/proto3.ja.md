+++
title = "言語ガイド（proto 3）"
weight = 40
description = "プロジェクトでプロトコルバッファのバージョン3を使用する方法について説明します。"
type = "docs"
+++

このガイドでは、プロトコルバッファ言語を使用してプロトコルバッファデータの構造を定義する方法について説明します。`.proto`ファイルの構文や`.proto`ファイルからデータアクセスクラスを生成する方法も含まれます。このガイドはプロトコルバッファ言語の**proto3**バージョンに対応しています。**proto2**構文については、[Proto2 言語ガイド](/programming-guides/proto2)を参照してください。

これはリファレンスガイドです。このドキュメントで説明されている機能の多くを使用するステップバイステップの例については、選択した言語の[tutorial](/getting-started)を参照してください。

## メッセージタイプの定義 {#simple}

まず、非常にシンプルな例を見てみましょう。検索リクエストメッセージ形式を定義したいとします。各検索リクエストには、クエリ文字列、興味のある結果ページ、およびページごとの結果数が含まれます。以下は、この種類のメッセージを定義するために使用する`.proto`ファイルです。

```proto
syntax = "proto3";

message SearchRequest {
  string query = 1;
  int32 page_number = 2;
  int32 results_per_page = 3;
}
```

*   ファイルの最初の行は、`proto3`構文を使用していることを指定しています。これを行わないと、プロトコルバッファコンパイラは[proto2](/programming-guides/proto2)を使用していると仮定します。これは、ファイル内で最初の空でないコメントでない行でなければなりません。
*   `SearchRequest`メッセージ定義は、このタイプのメッセージに含めたい各データのために3つのフィールド（名前/値のペア）を指定しています。各フィールドには名前と型があります。

### フィールドタイプの指定 {#specifying-types}

前述の例では、すべてのフィールドが[スカラータイプ](#scalar)であり、2つの整数（`page_number`と`results_per_page`）と1つの文字列（`query`）です。また、[列挙型](#enum)や他のメッセージタイプのような複合型をフィールドに指定することもできます。

### フィールド番号の割り当て {#assigning}

メッセージ定義内の各フィールドには、以下の制限を持つ`1`から`536,870,911`の間の番号を割り当てる必要があります。

-   指定された番号は、そのメッセージ内のすべてのフィールドの中で**一意である必要があります**。
-   フィールド番号`19,000`から`19,999`はプロトコルバッファの実装用に予約されています。プロトコルバッファコンパイラは、メッセージ内でこれらの予約されたフィールド番号のいずれかを使用すると警告を出します。
-   以前に[予約された](#fieldreserved)フィールド番号や[拡張](#extensions)に割り当てられたフィールド番号を使用することはできません。

この番号は、メッセージタイプが使用中の場合には**変更できません**。これは、[メッセージワイヤ形式](/programming-guides/encoding)でフィールドを識別するためのものです。フィールド番号を「変更する」ということは、そのフィールドを削除し、同じタイプの新しいフィールドを新しい番号で作成することと同等です。これを正しく行う方法については、[フィールドの削除](#deleting)を参照してください。

フィールド番号は**再利用してはいけません**。新しいフィールド定義で再利用するために、[予約済み](#fieldreserved)リストからフィールド番号を取り出してはいけません。フィールド番号を再利用することの結果については、[フィールド番号の再利用の結果](#consequences)を参照してください。

最も頻繁に設定されるフィールドには、フィールド番号1から15を使用する必要があります。低いフィールド番号の値は、ワイヤ形式でのスペースを取らないようになっています。たとえば、1から15の範囲のフィールド番号は1バイトでエンコードされます。16から2047の範囲のフィールド番号は2バイトを取ります。これについて詳しくは、[プロトコルバッファエンコーディング](/programming-guides/encoding#structure)を参照してください。

#### フィールド番号の再利用の結果 {#consequences}

フィールド番号を再利用すると、ワイヤ形式のメッセージのデコードが曖昧になります。

Protobufのワイヤ形式はシンプルであり、エンコードされたフィールドが別の定義でデコードされた場合を検出する方法を提供していません。

1つの定義でフィールドをエンコードし、異なる定義で同じフィールドをデコードすることは、次のような問題を引き起こす可能性があります：

-   デベロッパーがデバッグに費やす時間
-   パース/マージエラー（最善の場合）
-   PII/SPIIの漏洩
-   データの破損

フィールド番号の再利用の一般的な原因：

-   フィールドの番号を変更する（フィールドの番号順序をより見栄えの良いものにするために行われることがある）。フィールドの番号を変更することは、再利用不可能なワイヤ形式の変更をもたらします。
-   フィールドを削除し、将来の再利用を防ぐために番号を[予約](#fieldreserved)しない。

最大フィールドは、通常の32ビットではなく、29ビットです。これは下位3ビットがワイヤ形式で使用されているためです。詳細については、[エンコーディングトピック](/programming-guides/encoding#structure)を参照してください。

<a id="specifying-field-rules"></a>

### フィールドラベルの指定 {#field-labels}

メッセージフィールドは以下のいずれかになります:

*   `optional`: `optional` フィールドは次の2つの状態のいずれかです:

    *   フィールドが設定され、ワイヤから明示的に設定された値または解析された値を含んでいます。ワイヤにシリアル化されます。
    *   フィールドが未設定で、デフォルト値が返されます。ワイヤにシリアル化されません。

    値が明示的に設定されたかどうかを確認できます。

*   `repeated`: このフィールドタイプは、整形されたメッセージ内でゼロ回以上繰り返すことができます。繰り返し値の順序は保持されます。

*   `map`: これはキーと値のペアのフィールドタイプです。このフィールドタイプについては、[Maps](/programming-guides/encoding#maps) を参照してください。

*   明示的なフィールドラベルが適用されていない場合、"暗黙的なフィールドの存在" と呼ばれるデフォルトのフィールドラベルが想定されます。(この状態にフィールドを明示的に設定することはできません。) 整形されたメッセージには、このフィールドのゼロ個または1個が含まれることができます(1個を超えることはできません)。また、このタイプのフィールドがワイヤから解析されたかどうかを判断することはできません。暗黙的な存在フィールドは、デフォルト値でない限り、ワイヤにシリアル化されます。このトピックについて詳しくは、[Field Presence](/programming-guides/field_presence) を参照してください。

proto3 では、スカラー数値型の `repeated` フィールドはデフォルトで `packed` エンコーディングを使用します。`packed` エンコーディングについて詳細は、[Protocol Buffer Encoding](/programming-guides/encoding#packed) を参照してください。

#### 整形されたメッセージ {#well-formed}

protobuf メッセージに適用される "整形された" という用語は、シリアル化/デシリアル化されたバイトを指します。protoc パーサーは、与えられた proto 定義ファイルが解析可能であることを検証します。

複数の値を持つ `optional` フィールドの場合、protoc パーサーは入力を受け入れますが、最後のフィールドのみを使用します。そのため、"バイト" は "整形されていない" かもしれませんが、結果のメッセージは1つだけであり、"整形されている" である可能性があります(ただし、同じものをラウンドトリップさせることはできません)。

#### メッセージタイプのフィールドにはフィールドの存在があります {#field-presence}

proto3では、メッセージタイプのフィールドは既にフィールドの存在を持っています。そのため、`optional`修飾子を追加してもフィールドの存在は変わりません。

次のコードサンプルの`Message2`と`Message3`の定義は、すべての言語で同じコードを生成し、バイナリ、JSON、TextFormatでの表現に違いはありません：

```proto
syntax="proto3";

package foo.bar;

message Message1 {}

message Message2 {
  Message1 foo = 1;
}

message Message3 {
  optional Message1 bar = 1;
}
```

### 追加のメッセージタイプの追加 {#adding-types}

複数のメッセージタイプを単一の`.proto`ファイルで定義することができます。これは、複数の関連するメッセージを定義する場合に便利です。たとえば、`SearchResponse`メッセージタイプに対応する応答メッセージ形式を定義したい場合、同じ`.proto`に追加できます：

```proto
message SearchRequest {
  string query = 1;
  int32 page_number = 2;
  int32 results_per_page = 3;
}

message SearchResponse {
 ...
}
```

**メッセージの結合は膨張を引き起こす** 単一の`.proto`ファイルで複数のメッセージタイプ（メッセージ、列挙型、サービスなど）を定義できますが、大量のメッセージが異なる依存関係で定義されると依存関係の膨張を引き起こす可能性があります。可能な限り1つの`.proto`ファイルあたりのメッセージタイプを少なくすることが推奨されています。

### コメントの追加 {#adding-comments}

`.proto`ファイルにコメントを追加するには、C/C++スタイルの`//`および`/* ... */`構文を使用します。

```proto
/* SearchRequest represents a search query, with pagination options to
 * indicate which results to include in the response. */

message SearchRequest {
  string query = 1;
  int32 page_number = 2;  // Which page number do we want?
  int32 results_per_page = 3;  // Number of results to return per page.
}
```

### フィールドの削除 {#deleting}

フィールドを削除すると、適切に行われない場合に深刻な問題を引き起こす可能性があります。

フィールドが不要になり、すべてのクライアントコードから参照が削除された場合、メッセージからフィールド定義を削除することができます。ただし、削除したフィールド番号を**必ず**[予約](#fieldreserved)する必要があります。フィールド番号を予約しないと、将来開発者がその番号を再利用する可能性があります。

また、メッセージのJSONおよびTextFormatエンコーディングを引き続き解析できるように、フィールド名も予約する必要があります。

<a id="fieldreserved"></a>

#### 予約されたフィールド番号 {#reserved-field-numbers}

メッセージタイプを[更新](#updating)してフィールドを完全に削除したり、コメントアウトしたりすると、将来の開発者がそのタイプを更新する際にフィールド番号を再利用できるようになります。これは、[フィールド番号の再利用の影響](#consequences)で説明されているように深刻な問題を引き起こす可能性があります。これを防ぐために、削除したフィールド番号を`reserved`リストに追加してください。

protocコンパイラは、将来の開発者がこれらの予約されたフィールド番号を使用しようとするとエラーメッセージを生成します。

```proto
message Foo {
  reserved 2, 15, 9 to 11;
}
```

予約されたフィールド番号の範囲は包括的です（`9 to 11`は`9, 10, 11`と同じです）。

#### 予約されたフィールド名 {#reserved-field-names}

古いフィールド名を後で再利用することは一般的に安全ですが、TextProtoやJSONエンコーディングを使用する場合は、フィールド名がシリアライズされるために注意が必要です。このリスクを回避するために、削除されたフィールド名を`reserved`リストに追加できます。

予約された名前は、プロトコンパイラの動作のみに影響を与え、ランタイムの動作には影響しません。ただし、TextProtoの実装では、予約された名前を持つ未知のフィールドを解析時に無視することがあります（他の未知のフィールドと同様にエラーを発生させず、現在はC++およびGoの実装のみがそうしています）。ランタイムJSONパースには予約された名前は影響しません。

```proto
message Foo {
  reserved 2, 15, 9 to 11;
  reserved "foo", "bar";
}
```

同じ`reserved`ステートメント内でフィールド名とフィールド番号を混在させることはできないことに注意してください。

### `.proto`から生成されるものは何ですか？ {#generated}

`.proto`ファイルに[プロトコルバッファコンパイラ](#generating)を実行すると、選択した言語でコンパイラが生成するコードが、ファイルで記述したメッセージタイプを操作するために必要になります。これには、フィールド値の取得と設定、メッセージを出力ストリームにシリアライズすること、および入力ストリームからメッセージを解析することが含まれます。

*   **C++**の場合、コンパイラは各`.proto`から`.h`および`.cc`ファイルを生成し、ファイルで記述した各メッセージタイプに対するクラスを生成します。
*   **Java**の場合、コンパイラは各メッセージタイプに対するクラスと、メッセージクラスインスタンスを作成するための特別な`Builder`クラスを生成する`.java`ファイルを生成します。
*   **Kotlin**の場合、Javaで生成されたコードに加えて、コンパイラは改良されたKotlin APIを持つ各メッセージタイプ用の`.kt`ファイルを生成します。これには、メッセージインスタンスを作成するために簡素化されたDSL、nullableフィールドアクセサ、およびコピー関数が含まれます。
*   **Python**は少し異なります。Pythonコンパイラは、`.proto`内の各メッセージタイプの静的ディスクリプタを持つモジュールを生成し、これはランタイムで必要なPythonデータアクセスクラスを*メタクラス*と共に作成するために使用されます。
*   **Go**の場合、コンパイラは各ファイル内の各メッセージタイプに対する型を持つ`.pb.go`ファイルを生成します。
*   **Ruby**の場合、コンパイラは各メッセージタイプを含むRubyモジュールを持つ`.rb`ファイルを生成します。
*   **Objective-C**の場合、コンパイラは各`.proto`から`pbobjc.h`および`pbobjc.m`ファイルを生成し、ファイルで記述した各メッセージタイプに対するクラスを生成します。
*   **C#**の場合、コンパイラは各`.proto`から`.cs`ファイルを生成し、ファイルで記述した各メッセージタイプに対するクラスを生成します。
*   **PHP**の場合、コンパイラは各メッセージタイプに対する`.php`メッセージファイルと、コンパイルされた各`.proto`ファイルに対する`.php`メタデータファイルを生成します。メタデータファイルは有効なメッセージタイプをディスクリプタプールにロードするために使用されます。
*   **Dart**の場合、コンパイラは各ファイル内の各メッセージタイプに対するクラスを持つ`.pb.dart`ファイルを生成します。

## スカラー値の型 {#scalar}

スカラーメッセージフィールドには、次のいずれかのタイプが指定できます - テーブルには`.proto`ファイルで指定されたタイプと、自動生成されたクラス内の対応するタイプが示されています。

<div style="overflow:auto;width:100%;">
  <table style="width: 110%;">
    <tbody>
      <tr>
        <th>.proto タイプ</th>
        <th>ノート</th>
        <th>C++ タイプ</th>
        <th>Java/Kotlin タイプ<sup>[1]</sup></th>
        <th>Python タイプ<sup>[3]</sup></th>
        <th>Go タイプ</th>
        <th>Ruby タイプ</th>
        <th>C# タイプ</th>
        <th>PHP タイプ</th>
        <th>Dart タイプ</th>
      </tr>
      <tr>
        <td>double</td>
        <td></td>
        <td>double</td>
        <td>double</td>
        <td>float</td>
        <td>float64</td>
        <td>Float</td>
        <td>double</td>
        <td>float</td>
        <td>double</td>
      </tr>
      <tr>
        <td>float</td>
        <td></td>
        <td>float</td>
        <td>float</td>
        <td>float</td>
        <td>float32</td>
        <td>Float</td>
        <td>float</td>
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
        <td>integer</td>
        <td>int</td>
      </tr>
      <tr>
        <td>int64</td>
        <td>可変長エンコーディングを使用します。負の値をエンコードするには非効率です - フィールドが負の値を取る可能性が高い場合は、代わりにsint64を使用してください。</td>
        <td>int64</td>
        <td>long</td>
        <td>int/long<sup>[4]</sup></td>
        <td>int64</td>
        <td>Bignum</td>
        <td>long</td>
        <td>integer/string<sup>[6]</sup></td>
        <td>Int64</td>
      </tr>
      <tr>
        <td>uint32</td>
        <td>可変長エンコーディングを使用します。</td>
        <td>uint32</td>
        <td>int<sup>[2]</sup></td>
        <td>int/long<sup>[4]</sup></td>
        <td>uint32</td>
        <td>FixnumまたはBignum（必要に応じて）</td>
        <td>uint</td>
        <td>integer</td>
        <td>int</td>
      </tr>
      <tr>
        <td>uint64</td>
        <td>可変長エンコーディングを使用します。</td>
        <td>uint64</td>
        <td>long<sup>[2]</sup></td>
        <td>int/long<sup>[4]</sup></td>
        <td>uint64</td>
        <td>Bignum</td>
        <td>ulong</td>
        <td>integer/string<sup>[6]</sup></td>
        <td>Int64</td>
      </tr>
      <tr>
        <td>sint32</td>
        <td>可変長エンコーディングを使用します。符号付き整数値。これらは通常のint32よりも負の数を効率的にエンコードします。</td>
        <td>int32</td>
        <td>int</td>
        <td>int</td>
        <td>int32</td>
        <td>FixnumまたはBignum（必要に応じて）</td>
        <td>int</td>
        <td>integer</td>
        <td>int</td>
      </tr>
      <tr>
        <td>sint64</td>
        <td>可変長エンコーディングを使用します。符号付き整数値。これらは通常のint64よりも負の数を効率的にエンコードします。</td>
        <td>int64</td>
        <td>long</td>
        <td>int/long<sup>[4]</sup></td>
        <td>int64</td>
        <td>Bignum</td>
        <td>long</td>
        <td>integer/string<sup>[6]</sup></td>
        <td>Int64</td>
      </tr>
      <tr>
        <td>fixed32</td>
        <td>常に4バイトです。値が2<sup>28</sup>よりも大きい場合は、uint32よりも効率的です。</td>
        <td>uint32</td>
        <td>int<sup>[2]</sup></td>
        <td>int/long<sup>[4]</sup></td>
        <td>uint32</td>
        <td>FixnumまたはBignum（必要に応じて）</td>
        <td>uint</td>
        <td>integer</td>
        <td>int</td>
      </tr>
      <tr>
        <td>fixed64</td>
        <td>常に8バイトです。値が2<sup>56</sup>よりも大きい場合は、uint64よりも効率的です。</td>
        <td>uint64</td>
        <td>long<sup>[2]</sup></td>
        <td>int/long<sup>[4]</sup></td>
        <td>uint64</td>
        <td>Bignum</td>
        <td>ulong</td>
        <td>integer/string<sup>[6]</sup></td>
        <td>Int64</td>
      </tr>
      <tr>
        <td>sfixed32</td>
        <td>常に4バイトです。</td>
        <td>int32</td>
        <td>int</td>
        <td>int</td>
        <td>int32</td>
        <td>FixnumまたはBignum（必要に応じて）</td>
        <td>int</td>
        <td>integer</td>
        <td>int</td>
      </tr>
      <tr>
        <td>sfixed64</td>
        <td>常に8バイトです。</td>
        <td>int64</td>
        <td>long</td>
        <td>int/long<sup>[4]</sup></td>
        <td>int64</td>
        <td>Bignum</td>
        <td>long</td>
        <td>integer/string<sup>[6]</sup></td>
        <td>Int64</td>
      </tr>
      <tr>
        <td>bool</td>
        <td></td>
        <td>bool</td>
        <td>boolean</td>
        <td>bool</td>
        <td>bool</td>
        <td>TrueClass/FalseClass</td>
        <td>bool</td>
        <td>boolean</td>
        <td>bool</td>
      </tr>
      <tr>
        <td>string</td>
        <td>文字列は常にUTF-8エンコードまたは7ビットASCIIテキストを含んでいる必要があり、2<sup>32</sup>より長くすることはできません。</td>
        <td>string</td>
        <td>String</td>
        <td>str/unicode<sup>[5]</sup></td>
        <td>string</td>
        <td>String（UTF-8）</td>
        <td>string</td>
        <td>string</td>
        <td>String</td>
      </tr>
      <tr>
        <td>bytes</td>
        <td>2<sup>32</sup>より長くない任意のバイトシーケンスを含むことができます。</td>
        <td>string</td>
        <td>ByteString</td>
        <td>str（Python 2）<br/>bytes（Python 3）</td>
        <td>[]byte</td>
        <td>String（ASCII-8BIT）</td>
        <td>ByteString</td>
        <td>string</td>
        <td>List<int></td>
      </tr>
    </tbody>
  </table>
</div>

<sup>[1]</sup> Kotlinは、Javaと対応する型を使用し、Java/Kotlinの混在するコードベースでの互換性を確保します。

<sup>[2]</sup> Javaでは、符号付きの対応する型を使用して、符号なしの32ビットおよび64ビット整数を表現し、最上位ビットは単純に符号ビットに格納されます。

<sup>[3]</sup> すべての場合において、フィールドに値を設定すると、その値が有効であることを確認するために型チェックが実行されます。

<sup>[4]</sup> 64ビットまたは符号なしの32ビット整数は、デコード時に常にlongとして表現されますが、フィールドを設定する際にintが指定された場合はintとして表現されます。すべての場合において、値は設定時に表現される型に収まる必要があります。[2]を参照してください。

<sup>[5]</sup> Pythonの文字列はデコード時にunicodeとして表現されますが、ASCII文字列が与えられた場合はstrとして表現されます（これは変更される可能性があります）。

<sup>[6]</sup> 64ビットマシンではIntegerが使用され、32ビットマシンではStringが使用されます。

これらの型がどのようにエンコードされるかについて詳細を知るには、メッセージをシリアライズする際の[Protocol Buffer Encoding](/programming-guides/encoding)を参照してください。

## デフォルト値 {#default}

メッセージが解析されるとき、エンコードされたメッセージに特定の暗黙の存在要素が含まれていない場合、解析されたオブジェクトの対応するフィールドにアクセスすると、そのフィールドのデフォルト値が返されます。これらのデフォルト値は型固有です。

- 文字列の場合、デフォルト値は空の文字列です。
- バイトの場合、デフォルト値は空のバイトです。
- ブールの場合、デフォルト値はfalseです。
- 数値型の場合、デフォルト値はゼロです。
- 列挙型の場合、デフォルト値は**最初に定義された列挙値**であり、0である必要があります。
- メッセージフィールドの場合、フィールドは設定されません。その正確な値は言語に依存します。詳細については、[生成されたコードガイド](/reference/)を参照してください。

繰り返しフィールドのデフォルト値は空です（通常、適切な言語で空のリストです）。

スカラーメッセージフィールドの場合、メッセージが解析されると、フィールドがデフォルト値に明示的に設定されたかどうか（たとえば、ブール値が`false`に設定されたかどうか）を判断する方法はありません。メッセージタイプを定義する際には、これを考慮してください。たとえば、デフォルトでその動作が発生しないようにしたくない場合は、`false`に設定された場合にその動作が発生するブールを持たないでください。また、スカラーメッセージフィールドがデフォルト値に設定されている場合、その値はワイヤー上にシリアル化されません。floatまたはdouble値が+0に設定されている場合、シリアル化されませんが、-0は異なる値と見なされ、シリアル化されます。

## 列挙型 {#enum}

メッセージタイプを定義する際、事前に定義された値のリストの中から1つだけを持つようにしたい場合があります。たとえば、各 `SearchRequest` に `corpus` フィールドを追加したいとします。ここで、corpus は `UNIVERSAL`、`WEB`、`IMAGES`、`LOCAL`、`NEWS`、`PRODUCTS`、または `VIDEO` のいずれかであるとします。これは、メッセージ定義に `enum` を追加し、各可能な値に対して定数を持つことで非常に簡単に行うことができます。

以下の例では、すべての可能な値を持つ `Corpus` という `enum` を追加し、`Corpus` 型のフィールドを持っています。

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
  string query = 1;
  int32 page_number = 2;
  int32 results_per_page = 3;
  Corpus corpus = 4;
}
```

`Corpus` 列挙型の最初の定数がゼロにマップされていることがわかります。すべての列挙型定義には、最初の要素としてゼロにマップされる定数が必ず含まれている必要があります。これは、次の理由からです。

- 数値の [デフォルト値](#default) として 0 を使用できるようにするためにゼロ値が必要です。
- ゼロ値は、[proto2](/programming-guides/proto2) のセマンティクスとの互換性のため、最初の列挙値がデフォルトであり、異なる値が明示的に指定されていない限り使用される必要があります。

異なる列挙定数に同じ値を割り当てることでエイリアスを定義することができます。これを行うには、`allow_alias` オプションを `true` に設定する必要があります。そうしないと、エイリアスが見つかったときにプロトコルバッファコンパイラが警告メッセージを生成します。エイリアス値はすべて逆シリアル化中に有効ですが、シリアル化中には常に最初の値が使用されます。

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

列挙定数は 32 ビット整数の範囲内にある必要があります。`enum` 値はワイヤー上で [varint エンコーディング](/programming-guides/encoding) を使用するため、負の値は効率が悪く、推奨されません。以前の例のように、メッセージ定義内に `enum` を定義するか、外部に定義することができます。これらの `enum` は、`.proto` ファイル内の任意のメッセージ定義で再利用することができます。また、1つのメッセージで宣言された `enum` 型を、別のメッセージのフィールドの型として使用することもできます。その際の構文は `_MessageType_._EnumType_` を使用します。

プロトコルバッファコンパイラを実行すると、`enum` を使用する `.proto` ファイルに対して、生成されたコードには、Java、Kotlin、または C++ 向けの対応する `enum`、またはPython 向けの特別な `EnumDescriptor` クラスが生成されます。このクラスは、ランタイムで生成されたクラス内で整数値を持つ一連のシンボリック定数を作成するために使用されます。

{{% alert title="重要" color="warning" %}} 生成されたコードは、言語固有の列挙子の数に関する制限（1つの言語につき数千個以下）の影響を受ける可能性があります。使用する言語の制限事項を確認してください。{{% /alert %}}

逆シリアル化中、認識されない enum 値はメッセージ内に保持されますが、メッセージが逆シリアル化される際にこれがどのように表現されるかは言語に依存します。C++ や Go など、指定されたシンボルの範囲外の値を持つオープンな enum タイプをサポートする言語では、未知の enum 値は単にその基礎となる整数表現として格納されます。Java のようなクローズドな enum タイプを持つ言語では、認識されない値を表すために enum 内のケースが使用され、基礎となる整数は特別なアクセサを使用してアクセスできます。いずれの場合も、メッセージがシリアル化される場合、認識されない値はメッセージとともにシリアル化されます。

{{% alert title="重要" color="warning" %}} 現在の動作と比較して、異なる言語での列挙型の動作に関する情報については、[Enum Behavior](/programming-guides/enum) を参照してください。{{% /alert %}}

アプリケーションでメッセージの `enum` を使用する方法の詳細については、選択した言語向けの[生成されたコードガイド](/reference/)を参照してください。

### 予約された値 {#reserved}

列挙型を[更新](#updating)して、列挙エントリを完全に削除したり、コメントアウトしたりすると、将来のユーザーがその型を更新する際に、数値値を再利用することができます。これは、後で同じ `.proto` の古いバージョンを読み込むと、データの破損、プライバシーバグなどの深刻な問題を引き起こす可能性があります。これを防ぐ方法の1つは、削除されたエントリの数値値（および/または名前、これも JSON シリアル化に問題を引き起こす可能性があります）が `reserved` であることを指定することです。プロトコルバッファコンパイラは、将来のユーザーがこれらの識別子を使用しようとすると警告を出します。`max` キーワードを使用して、予約された数値値の範囲が最大可能な値になるように指定することもできます。

```proto
enum Foo {
  reserved 2, 15, 9 to 11, 40 to max;
  reserved "FOO", "BAR";
}
```

注意：同じ `reserved` ステートメント内でフィールド名と数値を混在させることはできません。

## 他のメッセージタイプの使用 {#other}

他のメッセージタイプをフィールドタイプとして使用することができます。たとえば、各 `SearchResponse` メッセージに `Result` メッセージを含めたい場合は、同じ `.proto` ファイル内で `Result` メッセージタイプを定義し、その後 `SearchResponse` 内で `Result` タイプのフィールドを指定します：

```proto
message SearchResponse {
  repeated Result results = 1;
}

message Result {
  string url = 1;
  string title = 2;
  repeated string snippets = 3;
}
```

### 定義のインポート {#importing}

前述の例では、`Result` メッセージタイプは `SearchResponse` と同じファイル内で定義されています。もし、フィールドタイプとして使用したいメッセージタイプが別の `.proto` ファイルで定義されている場合はどうすればよいでしょうか？

他の `.proto` ファイルからの定義を使用するには、それらを *インポート* します。別の `.proto` の定義をインポートするには、ファイルの先頭にインポートステートメントを追加します：

```proto
import "myproject/other_protos.proto";
```

デフォルトでは、直接インポートされた `.proto` ファイルからの定義のみを使用できます。ただし、時には `.proto` ファイルを新しい場所に移動する必要があるかもしれません。単一の変更ですべての呼び出し箇所を更新する代わりに、古い場所にプレースホルダー `.proto` ファイルを配置し、`import public` 機能を使用してすべてのインポートを新しい場所に転送することができます。

**Java では、`import public` 機能は使用できません。**

`import public` 依存関係は、`import public` ステートメントを含む proto をインポートするすべてのコードによって推移的に依存されることができます。たとえば：

```proto
// new.proto
// すべての定義がここに移動されます
```

```proto
// client.proto
import "old.proto";
// old.proto と new.proto の定義を使用しますが、other.proto は使用しません
```

プロトコルコンパイラは、`-I`/`--proto_path` フラグを使用してプロトコルコンパイラコマンドラインで指定されたディレクトリセット内のインポートファイルを検索します。フラグが指定されていない場合、コンパイラが呼び出されたディレクトリを検索します。一般的には、プロジェクトのルートに `--proto_path` フラグを設定し、すべてのインポートに完全修飾名を使用する必要があります。

### proto2 メッセージタイプの使用 {#proto2}

[proto2](/programming-guides/proto2) メッセージタイプをインポートして、proto3 メッセージで使用したり、その逆も可能です。ただし、proto2 列挙型は proto3 構文で直接使用することはできません（インポートされた proto2 メッセージがそれらを使用している場合は問題ありません）。

## ネストされたタイプ {#nested}

他のメッセージタイプ内でメッセージタイプを定義して使用することができます。次の例では、`Result` メッセージが `SearchResponse` メッセージ内で定義されています：

```proto
message SearchResponse {
  message Result {
    string url = 1;
    string title = 2;
    repeated string snippets = 3;
  }
  repeated Result results = 1;
}
```

このメッセージタイプを親メッセージタイプの外部で再利用したい場合は、`_Parent_._Type_` として参照します：

```proto
message SomeOtherMessage {
  SearchResponse.Result result = 1;
}
```

メッセージを好きなだけ深くネストすることができます。以下の例では、2 つの `Inner` という名前のネストされたタイプが完全に独立していることに注意してください。それらは異なるメッセージ内で定義されています：

```proto
message Outer {       // Level 0
  message MiddleAA {  // Level 1
    message Inner {   // Level 2
      int64 ival = 1;
      bool  booly = 2;
    }
  }
  message MiddleBB {  // Level 1
    message Inner {   // Level 2
      int32 ival = 1;
      bool  booly = 2;
    }
  }
}
```

## メッセージタイプの更新 {#updating}

既存のメッセージタイプがすべてのニーズを満たさなくなった場合（たとえば、メッセージ形式に追加のフィールドが欲しい場合）、古い形式で作成されたコードを引き続き使用したい場合でも心配しないでください！バイナリワイヤ形式を使用すると、既存のコードを壊さずにメッセージタイプを簡単に更新できます。

{{% alert title="注意" color="note" %}} プロトコルバッファメッセージを保存する際に JSON や [proto テキスト形式](/reference/protobuf/textformat-spec) を使用する場合、プロト定義で行える変更は異なります。{{% /alert %}}

[Proto ベストプラクティス](/programming-guides/dos-donts) と以下のルールを確認してください：

*   既存のフィールド番号を変更しないでください。フィールド番号を「変更」することは、フィールドを削除して同じ型の新しいフィールドを追加することと同等です。フィールドの番号を変更したい場合は、[フィールドの削除](#deleting)の手順を参照してください。
*   新しいフィールドを追加しても、古いメッセージ形式を使用して生成されたコードでシリアル化されたメッセージは引き続き新しい生成されたコードでパースできます。これらの要素の[デフォルト値](#default)を考慮してください。新しいコードが古いコードで生成されたメッセージと適切にやり取りできるようにするためです。同様に、新しいコードで作成されたメッセージは古いコードでパースできます。古いバイナリは、新しいフィールドをパースする際に単にそのフィールドを無視します。詳細については、[Unknown Fields](#unknowns) セクションを参照してください。
*   フィールドを削除できますが、更新されたメッセージタイプでフィールド番号が再利用されない限り問題ありません。フィールド名を変更したり、フィールド番号を[予約](#fieldreserved)することもできます。これにより、将来の `.proto` のユーザーが番号を誤って再利用することが防止されます。
*   `int32`、`uint32`、`int64`、`uint64`、`bool` はすべて互換性があります。つまり、これらの型のフィールドを他の型に変更しても、前方互換性や後方互換性が壊れることはありません。対応する型に収まらない数値がワイヤからパースされた場合、その数値を C++ でその型にキャストした場合と同じ効果が得られます（たとえば、64 ビット数値が int32 として読み取られると、32 ビットに切り捨てられます）。
*   `sint32` と `sint64` は互換性がありますが、他の整数型とは互換性がありません。
*   `string` と `bytes` は、バイトが有効な UTF-8 である限り互換性があります。
*   エンコードされたメッセージのバイトが含まれている場合、埋め込まれたメッセージは `bytes` と互換性があります。
*   `fixed32` は `sfixed32` と、`fixed64` は `sfixed64` と互換性があります。
*   `string`、`bytes`、およびメッセージフィールドに対して、`optional` は `repeated` と互換性があります。繰り返しフィールドのシリアル化データを入力として受け取るクライアントは、このフィールドが `optional` であることを期待している場合、最後の入力値を取得します（プリミティブ型フィールドの場合）またはすべての入力要素をマージします（メッセージ型フィールドの場合）。これは、数値型を含む一般的には安全ではないことに注意してください。数値型の繰り返しフィールドは、[packed](/programming-guides/encoding#packed) 形式でシリアル化される可能性があり、`optional` フィールドが期待される場合には正しくパースされません。
*   `enum` は `int32`、`uint32`、`int64`、`uint64` とワイヤ形式で互換性があります（値が収まらない場合は切り捨てられます）。ただし、メッセージがデシリアライズされるときにクライアントコードがそれらを異なる方法で処理する可能性があることに注意してください。たとえば、認識されない proto3 `enum` タイプはメッセージ内で保持されますが、メッセージがデシリアライズされるときにこれがどのように表現されるかは言語に依存します。Int フィールドは常にその値を保持します。
*   `optional` フィールドまたは拡張機能を `oneof` の新しいメンバーに変更することはバイナリ互換性がありますが、一部の言語（特に Go）では生成されたコードの API が非互換な方法で変更されます。このため、Google は[公開 API](https://google.aip.dev/180#moving-into-oneofs)でそのような変更を行いません。ソース互換性について同じ警告がありますが、複数のフィールドを新しい `oneof` に移動することは、1 回に 1 つ以上のコードが設定されないことを確認すれば安全かもしれません。既存の `oneof` にフィールドを移動することは安全ではありません。同様に、単一のフィールド `oneof` を `optional` フィールドまたは拡張機能に変更することは安全です。
*   `map<K, V>` と対応する `repeated` メッセージフィールドの間でフィールドを変更することはバイナリ互換性があります（メッセージのレイアウトやその他の制限については、[Maps](#maps) を参照してください）。ただし、変更の安全性はアプリケーションに依存します。メッセージをデシリアライズして再シリアライズするとき、`repeated` フィールド定義を使用するクライアントは意味的に同一の結果を生成します。ただし、`map` フィールド定義を使用するクライアントはエントリを並べ替えたり、重複キーのエントリを削除したりする場合があります。

## 未知のフィールド {#unknowns}

未知のフィールドは、パーサーが認識しないフィールドを表す、適切に形成されたプロトコルバッファシリアライズされたデータです。たとえば、古いバイナリが新しいフィールドを持つ新しいバイナリによって送信されたデータを解析するとき、その新しいフィールドは古いバイナリ内の未知のフィールドとなります。

Proto3 メッセージは未知のフィールドを保持し、解析中およびシリアライズされた出力時にそれらを含めます。これは proto2 の動作と一致します。

## Any {#any}

`Any` メッセージ型を使用すると、`.proto` 定義を持たないメッセージを埋め込み型として使用できます。`Any` は、`bytes` として任意のシリアライズされたメッセージを含み、そのメッセージのタイプを一意に識別し解決するための URL を伴います。`Any` 型を使用するには、`google/protobuf/any.proto` を[import](#other)する必要があります。

```proto
import "google/protobuf/any.proto";

message ErrorStatus {
  string message = 1;
  repeated google.protobuf.Any details = 2;
}
```

特定のメッセージ型のデフォルトのタイプ URL は `type.googleapis.com/_packagename_._messagename_` です。

異なる言語の実装は、`Any` 値を型安全な方法でパックおよびアンパックするためのランタイムライブラリヘルパーをサポートします。たとえば、Java では、`Any` 型には特別な `pack()` および `unpack()` アクセサがあり、C++ では `PackFrom()` および `UnpackTo()` メソッドがあります。

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

**現在、`Any` 型との作業に関するランタイムライブラリは開発中です**。

`Any` メッセージ型は、proto2 メッセージと同様に任意の proto3 メッセージを保持できます。これにより、[拡張](/programming-guides/proto2#extensions)を許可する proto2 メッセージと同様の機能が可能です。

## Oneof {#oneof}

1 つのメッセージに多くのフィールドがあり、同時に最大 1 つのフィールドが設定される場合、この動作を強制し、メモリを節約するために oneof 機能を使用できます。

Oneof フィールドは通常のフィールドと同様ですが、oneof 内のすべてのフィールドがメモリを共有し、同時に最大 1 つのフィールドが設定されます。oneof のメンバーのいずれかを設定すると、自動的に他のすべてのメンバーがクリアされます。選択した言語に応じて、特別な `case()` または `WhichOneof()` メソッドを使用して、oneof 内で設定された値を確認できます。

複数の値が設定されている場合、proto 内の順序によって決定された最後に設定された値がすべての以前の値を上書きすることに注意してください。


oneof フィールドのフィールド番号は、包含メッセージ内で一意である必要があります。

### Oneof の使用 {#using-oneof}

`.proto` で oneof を定義するには、`oneof` キーワードに続いて oneof 名を指定します。この場合は `test_oneof` です。

```proto
message SampleMessage {
  oneof test_oneof {
    string name = 4;
    SubMessage sub_message = 9;
  }
}
```

その後、oneof 定義に oneof フィールドを追加します。`map` フィールドや `repeated` フィールド以外の任意の型のフィールドを追加できます。oneof に繰り返しフィールドを追加する必要がある場合は、繰り返しフィールドを含むメッセージを使用できます。

生成されたコードでは、oneof フィールドは通常のフィールドと同じゲッターとセッターを持ちます。また、oneof に設定された値を確認するための特別なメソッドも取得できます。選択した言語の oneof API について詳細を知るには、関連する [API リファレンス](/reference/) を参照してください。

### Oneof の機能 {#oneof-features}

*   oneof フィールドを設定すると、oneof の他のメンバーは自動的にクリアされます。つまり、複数の oneof フィールドを設定した場合、最後に設定したフィールドのみが値を保持します。

    ```c++
    SampleMessage message;
    message.set_name("name");
    CHECK_EQ(message.name(), "name");
    // Calling mutable_sub_message() will clear the name field and will set
    // sub_message to a new instance of SubMessage with none of its fields set.
    message.mutable_sub_message();
    CHECK(message.name().empty());
    ```

*   パーサーがワイヤー上で同じ oneof の複数のメンバーに遭遇した場合、解析されたメッセージには最後に見たメンバーのみが使用されます。

*   oneof は `repeated` にすることはできません。

*   oneof フィールドに対してリフレクション API が機能します。

*   oneof フィールドをデフォルト値に設定する場合（たとえば、int32 の oneof フィールドを 0 に設定する場合）、その oneof フィールドの "case" が設定され、値がワイヤー上でシリアライズされます。

*   C++ を使用している場合は、コードがメモリクラッシュを引き起こさないように注意してください。次のサンプルコードはクラッシュします。なぜなら、`set_name()` メソッドを呼び出すことで `sub_message` が既に削除されているからです。

    ```c++
    SampleMessage message;
    SubMessage* sub_message = message.mutable_sub_message();
    message.set_name("name");      // Will delete sub_message
    sub_message->set_...            // Crashes here
    ```

*   再び C++ で、oneof を持つメッセージを `Swap()` すると、各メッセージが他方の oneof ケースを持つようになります。以下の例では、`msg1` には `sub_message` が、`msg2` には `name` が含まれます。

    ```c++
    SampleMessage msg1;
    msg1.set_name("name");
    SampleMessage msg2;
    msg2.mutable_sub_message();
    msg1.swap(&msg2);
    CHECK(msg1.has_sub_message());
    CHECK_EQ(msg2.name(), "name");
    ```

### 互換性の問題 {#backward}

oneof フィールドを追加または削除する際は注意してください。oneof の値をチェックして `None`/`NOT_SET` が返される場合、それは oneof が設定されていないか、異なるバージョンの oneof のフィールドに設定されている可能性があることを意味します。ワイヤー上の未知のフィールドが oneof のメンバーであるかどうかを知る方法がないため、違いを判断する方法はありません。

#### タグ再利用の問題 {#reuse}

*   **oneof にフィールドを移動するか、移動させる**: メッセージがシリアライズおよびパースされた後、一部の情報が失われる可能性があります（一部のフィールドがクリアされる）。ただし、単一のフィールドを**新しい** oneof に安全に移動することができ、1 つだけが設定されることがわかっている場合は複数のフィールドを移動することもできるかもしれません。詳細については、[メッセージ型の更新](#updating)を参照してください。
*   **oneof フィールドを削除して追加する**: メッセージがシリアライズおよびパースされた後、現在設定されている oneof フィールドがクリアされる可能性があります。
*   **oneof を分割または結合する**: これは通常のフィールドを移動する場合と同様の問題があります。

## マップ {#maps}

データ定義の一部として連想マップを作成したい場合、プロトコルバッファは便利なショートカット構文を提供します:

```proto
map<key_type, value_type> map_field = N;
```

...ここで `key_type` は任意の整数型または文字列型（つまり、浮動小数点型および `bytes` を除く任意の[スカラー](#scalar)型）であることに注意してください。`key_type` には enum や proto メッセージは有効ではありません。
`value_type` は別のマップ以外の任意の型であることができます。

たとえば、各 `Project` メッセージが文字列キーと関連付けられるプロジェクトのマップを作成したい場合、次のように定義できます:

```proto
map<string, Project> projects = 3;
```

### マップの機能 {#maps-features}

*   マップフィールドは `repeated` にできません。
*   マップ値のワイヤーフォーマットの順序およびマップの反復順序は未定義なので、特定の順序でマップアイテムがあるとは依存できません。
*   `.proto` のテキスト形式を生成するとき、マップはキーでソートされます。数値キーは数値順にソートされます。
*   ワイヤーからパースするかマージするとき、重複するマップキーがある場合、最後に見たキーが使用されます。テキスト形式からマップをパースするとき、重複するキーがあるとパースに失敗する場合があります。
*   マップフィールドにキーを提供して値を提供しない場合、フィールドがシリアライズされる際の動作は言語に依存します。C++、Java、Kotlin、Python では、その型のデフォルト値がシリアライズされますが、他の言語では何もシリアライズされません。
*   同じスコープ内に `foo` マップの実装として `FooEntry` シンボルは存在できません。なぜなら、`FooEntry` はマップの実装で既に使用されているからです。

生成されたマップAPIは現在、すべてのサポートされている言語で利用可能です。
選択した言語に関するマップAPIについて詳細を知ることができます。
[APIリファレンス](/reference/)を参照してください。

### 互換性の維持 {#backwards}

マップの構文は、ワイヤ上で以下と同等です。そのため、マップをサポートしていないプロトコルバッファの実装でもデータを処理できます。

```proto
message MapFieldEntry {
  key_type key = 1;
  value_type value = 2;
}

repeated MapFieldEntry map_field = N;
```

マップをサポートするプロトコルバッファの実装は、以前の定義で受け入れられるデータを生成および受け入れる必要があります。

## パッケージ {#packages}

`.proto`ファイルにオプションの`package`指定子を追加して、プロトコルメッセージタイプ間の名前の衝突を防ぐことができます。

```proto
package foo.bar;
message Open { ... }
```

その後、メッセージタイプのフィールドを定義する際にパッケージ指定子を使用できます。

```proto
message Foo {
  ...
  foo.bar.Open open = 1;
  ...
}
```

パッケージ指定子が生成されたコードに影響を与える方法は、選択した言語によって異なります。

*   **C++**では、生成されたクラスはC++の名前空間内にラップされます。例えば、`Open`は名前空間`foo::bar`内にあります。
*   **Java**および**Kotlin**では、パッケージはJavaパッケージとして使用されますが、`.proto`ファイルで明示的に`option java_package`を指定しない限りです。
*   **Python**では、`package`ディレクティブは無視されます。Pythonモジュールはファイルシステム内の場所に従って構成されます。
*   **Go**では、`package`ディレクティブは無視され、生成された`.pb.go`ファイルは、対応する`go_proto_library` Bazelルールに基づいて名前付けられたパッケージ内にあります。オープンソースプロジェクトでは、`go_package`オプションを提供するか、Bazelの`-M`フラグを設定する必要があります。
*   **Ruby**では、生成されたクラスはネストされたRuby名前空間内にラップされ、必要なRubyの大文字化スタイルに変換されます（最初の文字が大文字になります。最初の文字が文字でない場合は、`PB_`が前に付けられます）。例えば、`Open`は名前空間`Foo::Bar`にあります。
*   **PHP**では、PascalCaseに変換した後にパッケージが名前空間として使用されますが、`.proto`ファイルで明示的に`option php_namespace`を指定しない限りです。例えば、`Open`は名前空間`Foo\Bar`にあります。
*   **C#**では、PascalCaseに変換した後にパッケージが名前空間として使用されますが、`.proto`ファイルで明示的に`option csharp_namespace`を指定しない限りです。例えば、`Open`は名前空間`Foo.Bar`にあります。

注意してください。`package` ディレクティブが生成されたコードに直接影響を与えない場合でも、たとえば Python の場合、`.proto` ファイルのパッケージを指定することを強くお勧めします。そうしないと、記述子での名前の競合が発生し、他の言語にとってプロトが移植可能でなくなる可能性があります。

### パッケージと名前解決 {#name-resolution}

プロトコルバッファ言語における型名の解決は C++ のように動作します。最初に最も内側のスコープが検索され、次にその内側のスコープが検索され、以降、各パッケージがその親パッケージに対して「内側」であると見なされます。先頭に '.'（たとえば、`.foo.bar.Baz`）を付けると、最も外側のスコープから開始することを意味します。

プロトコルバッファコンパイラは、インポートされた `.proto` ファイルを解析してすべての型名を解決します。各言語のコード生成ツールは、異なるスコープルールを持っていても、その言語で各型を参照する方法を知っています。

## サービスの定義 {#services}

RPC（Remote Procedure Call）システムでメッセージ型を使用したい場合、`.proto` ファイルに RPC サービスインターフェースを定義し、プロトコルバッファコンパイラが選択した言語でサービスインターフェースコードとスタブを生成します。たとえば、`SearchRequest` を受け取り `SearchResponse` を返すメソッドを持つ RPC サービスを定義したい場合、次のように `.proto` ファイルに定義できます。

```proto
service SearchService {
  rpc Search(SearchRequest) returns (SearchResponse);
}
```

プロトコルバッファと特によく組み合わせて使用される最も直感的な RPC システムは、Google で開発された言語やプラットフォームに依存しないオープンソースの RPC システムである [gRPC](https://grpc.io) です。gRPC は、プロトコルバッファと非常によく機能し、特別なプロトコルバッファコンパイラプラグインを使用して、`.proto` ファイルから関連する RPC コードを直接生成できます。

gRPC を使用したくない場合、独自の RPC 実装でプロトコルバッファを使用することも可能です。これについて詳しくは、[Proto2 Language Guide](/programming-guides/proto2#services) を参照してください。

Protocol Buffers のための RPC 実装を開発するための第三者プロジェクトもいくつか進行中です。知っているプロジェクトへのリンクのリストについては、[third-party add-ons wiki page](https://github.com/protocolbuffers/protobuf/blob/master/docs/third_party.md) を参照してください。

## JSON マッピング {#json}

Proto3 は JSON での標準エンコーディングをサポートし、システム間でデータを共有することを容易にします。以下の表に、エンコーディングがタイプごとに記述されています。

JSON エンコードされたデータをプロトコルバッファにパースする際、値が欠落しているか、その値が `null` の場合、それは対応する[デフォルト値](#default)として解釈されます。

プロトコルバッファから JSON エンコードされた出力を生成する際、protobuf フィールドがデフォルト値を持ち、かつフィールドがフィールドの存在をサポートしていない場合、デフォルトでは出力から省略されます。実装は、デフォルト値を持つフィールドを出力に含めるオプションを提供することができます。

`optional` キーワードを使用して定義された proto3 フィールドは、フィールドの存在をサポートします。値が設定されており、フィールドの存在をサポートしているフィールドは、その値を JSON エンコードされた出力に常に含めます。それがデフォルト値であってもです。

<table>
  <tbody>
    <tr>
      <th>proto3</th>
      <th>JSON</th>
      <th>JSON の例</th>
      <th>ノート</th>
    </tr>
    <tr>
      <td>message</td>
      <td>object</td>
      <td><code>{"fooBar": v, "g": null, ...}</code></td>
      <td>JSON オブジェクトを生成します。メッセージフィールド名は lowerCamelCase にマッピングされ、JSON オブジェクトのキーになります。もし `json_name` フィールドオプションが指定されている場合、指定された値がキーとして使用されます。パーサーは lowerCamelCase 名（または `json_name` オプションで指定された名前）と元の proto フィールド名の両方を受け入れます。`null` はすべてのフィールドタイプの受け入れられる値であり、対応するフィールドタイプのデフォルト値として扱われます。ただし、`json_name` の値には `null` を使用できません。詳細については、[json_name の厳格な検証](/news/2023-04-28#json-name)を参照してください。
      </td>
    </tr>
    <tr>
      <td>enum</td>
      <td>string</td>
      <td><code>"FOO_BAR"</code></td>
      <td>proto で指定された enum 値の名前が使用されます。パーサーは enum 名と整数値の両方を受け入れます。
      </td>
    </tr>
    <tr>
      <td>map&lt;K,V&gt;</td>
      <td>object</td>
      <td><code>{"k": v, ...}</code></td>
      <td>すべてのキーは文字列に変換されます。</td>
    </tr>
    <tr>
      <td>repeated V</td>
      <td>array</td>
      <td><code>[v, ...]</code></td>
      <td><code>null</code> は空リスト <code>[]</code> として受け入れられます。</td>
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
      <td>JSON 値は、パディングを使用した標準の base64 エンコーディングを使用して文字列としてエンコードされます。標準または URL セーフな base64 エンコーディング（パディングあり/なし）のいずれも受け入れられます。
      </td>
    </tr>
    <tr>
      <td>int32, fixed32, uint32</td>
      <td>number</td>
      <td><code>1, -10, 0</code></td>
      <td>JSON 値は 10 進数です。数値または文字列のいずれも受け入れられます。
      </td>
    </tr>
    <tr>
      <td>int64, fixed64, uint64</td>
      <td>string</td>
      <td><code>"1", "-10"</code></td>
      <td>JSON 値は 10 進数の文字列です。数値または文字列のいずれも受け入れられます。
      </td>
    </tr>
    <tr>
      <td>float, double</td>
      <td>number</td>
      <td><code>1.1, -10.0, 0, "NaN", "Infinity"</code></td>
      <td>JSON 値は数値または "NaN"、"Infinity"、"-Infinity" の特別な文字列値のいずれかです。数値または文字列のいずれも受け入れられます。指数表記も受け入れられます。
      </td>
    </tr>
    <tr>
      <td>Any</td>
      <td><code>object</code></td>
      <td><code>{"@type": "url", "f": v, ... }</code></td>
      <td><code>Any</code> が特別な JSON マッピングを持つ値を含む場合、それは次のように変換されます: <code>{"@type": xxx, "value": yyy}</code>。それ以外の場合、値は JSON オブジェクトに変換され、実際のデータ型を示すために <code>"@type"</code> フィールドが挿入されます。
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
      <td>生成された出力には常に 0、3、6、または 9 桁の小数部が含まれ、必要な精度に応じて "s" 接尾辞が続きます。フィットする限り、任意の小数部（なしも含む）が受け入れられ、"s" 接尾辞が必要です。
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
      <td>ラッパーは、ラップされたプリミティブ型と同じ表現を JSON で使用しますが、`null` は許可され、データの変換と転送中に保持されます。
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
      <td>任意の JSON 値。詳細については、[google.protobuf.Value](/reference/protobuf/google.protobuf#value) を確認してください。
      </td>
    </tr>
    <tr>
      <td>NullValue</td>
      <td>null</td>
      <td></td>
      <td>JSON の null</td>
    </tr>
    <tr>
      <td>Empty</td>
      <td>object</td>
      <td><code>{}</code></td>
      <td>空の JSON オブジェクト</td>
    </tr>
  </tbody>
</table>

### JSON オプション {#json-options}

proto3 JSON の実装では、以下のオプションが提供される可能性があります:

*   **存在しないフィールドを常に出力する**: 存在しないフィールドでかつデフォルト値を持つフィールドは、デフォルトでは JSON 出力において省略されます（たとえば、暗黙の存在を示す整数で値が 0 の場合、空の文字列である暗黙の存在を示す文字列フィールド、空の繰り返しフィールドやマップフィールド）。実装はこの動作をオーバーライドして、デフォルト値を持つフィールドを出力するオプションを提供することができます。
*   **未知のフィールドを無視する**: Proto3 JSON パーサーはデフォルトで未知のフィールドを拒否するべきですが、パーシング時に未知のフィールドを無視するオプションを提供することができます。
*   **lowerCamelCase 名の代わりに proto フィールド名を使用する**: デフォルトでは、proto3 JSON プリンターはフィールド名を lowerCamelCase に変換し、それを JSON 名として使用するべきです。実装は代わりに proto フィールド名を JSON 名として使用するオプションを提供することができます。Proto3 JSON パーサーは、変換された lowerCamelCase 名と proto フィールド名の両方を受け入れる必要があります。
*   **列挙値を文字列ではなく整数として出力する**: 列挙値の名前がデフォルトで JSON 出力に使用されます。オプションを提供して、列挙値の数値値を代わりに使用することができます。

## オプション {#options}

`.proto` ファイル内の個々の宣言には、複数の *オプション* を付与することができます。オプションは宣言の全体的な意味を変更しませんが、特定のコンテキストでの処理方法に影響を与える可能性があります。利用可能なオプションの完全なリストは、[`/google/protobuf/descriptor.proto`](https://github.com/protocolbuffers/protobuf/blob/main/src/google/protobuf/descriptor.proto) で定義されています。

一部のオプションはファイルレベルのオプションであり、メッセージ、列挙型、またはサービス定義の内部ではなく、トップレベルスコープに記述する必要があります。一部のオプションはメッセージレベルのオプションであり、メッセージ定義の内部に記述する必要があります。一部のオプションはフィールドレベルのオプションであり、フィールド定義の内部に記述する必要があります。オプションは列挙型、列挙値、oneof フィールド、サービス型、およびサービスメソッドにも記述することができますが、これらのいずれに対しても有用なオプションは現在存在しません。

ここには、最も一般的に使用されるいくつかのオプションがあります：

*   `java_package`（ファイルオプション）：生成されたJava/Kotlinクラスで使用するパッケージ。`.proto`ファイルで明示的な`java_package`オプションが指定されていない場合、デフォルトでprotoパッケージ（`.proto`ファイルで「package」キーワードを使用して指定されたもの）が使用されます。ただし、protoパッケージは一般的に逆ドメイン名で始まることが期待されていないため、protoパッケージは良いJavaパッケージとは言えません。JavaまたはKotlinコードを生成しない場合、このオプションは効果がありません。

    ```proto
    option java_package = "com.example.foo";
    ```

*   `java_outer_classname`（ファイルオプション）：生成するラッパーJavaクラスのクラス名（およびしたがってファイル名）。`.proto`ファイルで明示的な`java_outer_classname`が指定されていない場合、クラス名は`.proto`ファイル名をキャメルケースに変換して構築されます（つまり、`foo_bar.proto`は`FooBar.java`になります）。`java_multiple_files`オプションが無効になっている場合、`.proto`ファイルに対して生成される他のすべてのクラス/列挙型などは、この外部ラッパーJavaクラス内にネストされたクラス/列挙型などとして生成されます。Javaコードを生成しない場合、このオプションは効果がありません。

    ```proto
    option java_outer_classname = "Ponycopter";
    ```

*   `java_multiple_files`（ファイルオプション）：falseの場合、この`.proto`ファイルに対して単一の`.java`ファイルが生成され、トップレベルのメッセージ、サービス、および列挙型に対して生成されるすべてのJavaクラス/列挙型などが外部クラス（`java_outer_classname`を参照）内にネストされます。trueの場合、トップレベルのメッセージ、サービス、および列挙型に対して生成される各Javaクラス/列挙型などに対して別々の`.java`ファイルが生成され、この`.proto`ファイルに対して生成されるラッパーJavaクラスにはネストされたクラス/列挙型などが含まれません。これはデフォルトで`false`に設定されるブールオプションです。Javaコードを生成しない場合、このオプションは効果がありません。

    ```proto
    option java_multiple_files = true;
    ```

*   `optimize_for`（ファイルオプション）：`SPEED`、`CODE_SIZE`、または`LITE_RUNTIME`に設定できます。これはC++およびJavaのコードジェネレータ（およびサードパーティのジェネレータ）に以下のように影響します：

    *   `SPEED`（デフォルト）：プロトコルバッファコンパイラは、メッセージタイプのシリアル化、パース、およびその他の一般的な操作のためのコードを生成します。このコードは高度に最適化されています。
    *   `CODE_SIZE`：プロトコルバッファコンパイラは最小限のクラスを生成し、共有されたリフレクションベースのコードに依存してシリアル化、パース、およびさまざまな他の操作を実装します。生成されるコードは`SPEED`よりもはるかに小さくなりますが、操作は遅くなります。クラスは引き続き`SPEED`モードとまったく同じパブリックAPIを実装します。このモードは、非常に多くの`.proto`ファイルを含むアプリで、すべてを極めて高速にする必要がない場合に最も有用です。
    *   `LITE_RUNTIME`：プロトコルバッファコンパイラは、「lite」ランタイムライブラリ（`libprotobuf-lite`ではなく`libprotobuf`）にのみ依存するクラスを生成します。ライトランタイムはフルライブラリよりもはるかに小さく（約1桁小さい）ですが、ディスクリプタやリフレクションなどの特定の機能を省略します。これは、モバイル電話などの制約のあるプラットフォームで実行されるアプリに特に有用です。コンパイラは引き続き、`SPEED`モードと同様にすべてのメソッドの高速な実装を生成します。生成されたクラスは、各言語で`MessageLite`インターフェースのみを実装し、完全な`Message`インターフェースのメソッドのサブセットのみを提供します。

    ```proto
    option optimize_for = CODE_SIZE;
    ```

*   `cc_generic_services`、`java_generic_services`、`py_generic_services`（ファイルオプション）：**ジェネリックサービスは非推奨です。**プロトコルバッファコンパイラが、C++、Java、およびPythonの[サービス定義](#services)に基づいて抽象サービスコードを生成するかどうか。遺産上、これらはデフォルトで`true`になっています。ただし、2010年1月のバージョン2.3.0以降では、RPC実装が各システムに特化したコードを生成するために
    [コードジェネレータプラグイン](/reference/cpp/api-docs/google.protobuf.compiler.plugin.pb)
    を提供することが好ましいと考えられています。それによって、「抽象」サービスに依存するのではなく、よりシステム固有のコードを生成できます。

```proto
(// This file relies on plugins to generate service code.
    option cc_generic_services = false;
    option java_generic_services = false;
    option py_generic_services = false;
```

*   `cc_enable_arenas`（ファイルオプション）：C++生成コードのために[アリーナ割り当て](/reference/cpp/arenas)を有効にします。

*   `objc_class_prefix`（ファイルオプション）：この.protoから生成されたすべてのObjective-Cクラスと列挙型に先頭に付加されるObjective-Cクラスプレフィックスを設定します。デフォルトはありません。[Appleによって推奨されている](https://developer.apple.com/library/ios/documentation/Cocoa/Conceptual/ProgrammingWithObjectiveC/Conventions/Conventions.html#//apple_ref/doc/uid/TP40011210-CH10-SW4)3〜5文字の大文字プレフィックスを使用する必要があります。2文字のプレフィックスはすべてAppleによって予約されていることに注意してください。

*   `packed`（フィールドオプション）：基本的な数値型の繰り返しフィールドのデフォルトは`true`で、よりコンパクトな[エンコーディング](/programming-guides/encoding#packed)が使用されます。このオプションを使用するデメリットはありませんが、`false`に設定することもできます。バージョン2.3.0以前では、予期しない状況でパックされたデータを受信したパーサーは無視していました。そのため、既存のフィールドをパック形式に変更することは、ワイヤー互換性を壊さずにはできませんでした。2.3.0以降では、この変更は安全です。パッキング可能なフィールドのパーサーは常に両方の形式を受け入れますが、古いプログラムを使用している場合は注意してください。

    ```proto
    repeated int32 samples = 4 [packed = false];
    ```

*   `deprecated`（フィールドオプション）：`true`に設定すると、フィールドが非推奨であり、新しいコードで使用すべきではないことを示します。ほとんどの言語では、これに実際の効果はありません。Javaでは、これは`@Deprecated`注釈になります。C++では、非推奨のフィールドが使用されるたびにclang-tidyが警告を生成します。将来、他の言語固有のコードジェネレーターは、フィールドのアクセサーに非推奨の注釈を生成し、その結果、フィールドを使用しようとするコードをコンパイルする際に警告が発生します。誰も使用していないフィールドであり、新しいユーザーが使用するのを防ぎたい場合は、フィールド宣言を[予約](#fieldreserved)ステートメントで置き換えることを検討してください。

```proto
int32 old_field = 6 [deprecated = true];
```

### 列挙値オプション {#enum-value-options}

列挙値オプションはサポートされています。`deprecated` オプションを使用して、もはや使用されない値を示すことができます。また、拡張機能を使用してカスタムオプションを作成することもできます。

次の例は、これらのオプションを追加する構文を示しています：

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

列挙値やフィールドにカスタムオプションを適用する方法については、[カスタムオプション](#customoptions) を参照してください。

### カスタムオプション {#customoptions}

Protocol Buffers では、独自のオプションを定義して使用することもできます。これはほとんどの人が必要としない **高度な機能** です。独自のオプションを作成する必要があると考える場合は、詳細については [Proto2 言語ガイド](/programming-guides/proto2#customoptions) を参照してください。カスタムオプションの作成には、proto3 ではカスタムオプション専用に許可されている [拡張機能](/programming-guides/proto2#extensions) を使用します。

### オプションの保持 {#option-retention}

オプションには *保持* の概念があり、オプションが生成されたコードで保持されるかどうかを制御します。オプションはデフォルトで *ランタイム保持* を持ち、つまり生成されたコードで保持され、生成された記述子プールでランタイムで見えるようになります。ただし、`retention = RETENTION_SOURCE` を設定して、オプション（またはオプション内のフィールド）をランタイムで保持しないように指定することもできます。これを *ソース保持* と呼びます。

オプションの保持は、ほとんどのユーザーが気にする必要がない高度な機能ですが、特定のオプションを使用したい場合に、それらをバイナリに保持するコードサイズのコストを支払わずに使用することができます。ソース保持のオプションは、`protoc` や `protoc` プラグインには引き続き見えるため、コード生成ツールは動作をカスタマイズするためにそれらを使用できます。

保持は、次のようにオプションに直接設定することもできます：

```proto
extend google.protobuf.FileOptions {
  optional int32 source_retention_option = 1234
      [retention = RETENTION_SOURCE];
}
```

また、オプション内に現れる場合にのみ効果があるように、プレーンフィールドにも設定できます：

```proto
message OptionsMessage {
  int32 source_retention_field = 1 [retention = RETENTION_SOURCE];
}
```

`retention = RETENTION_RUNTIME` を設定しても構いませんが、これはデフォルトの動作ですので効果はありません。メッセージフィールドが `RETENTION_SOURCE` とマークされている場合、その内容全体が削除されます。その内部のフィールドは `RETENTION_RUNTIME` を設定しようとしてもそれを上書きすることはできません。

{{% alert title="Note" color="note" %}}Protocol Buffers 22.0 では、オプションの保持に関するサポートはまだ進行中であり、C++ と Java のみがサポートされています。Go は 1.29.0 からサポートされています。Python のサポートは完了していますが、リリースにはまだ含まれていません。{{% /alert %}}

### オプションのターゲット {#option-targets}

フィールドには `targets` オプションがあり、そのフィールドがオプションとして使用される際に適用できるエンティティの種類を制御します。たとえば、フィールドに `targets = TARGET_TYPE_MESSAGE` がある場合、そのフィールドは enum（または他の非メッセージエンティティ）のカスタムオプションで設定できません。Protoc はこれを強制し、ターゲット制約が違反された場合にエラーを発生させます。

一見すると、この機能は不要に見えるかもしれません。なぜなら、すべてのカスタムオプションは特定のエンティティ用のオプションメッセージの拡張であり、すでにそのオプションをその1つのエンティティに制約しているからです。ただし、オプションのターゲットは、複数のエンティティタイプに適用される共有オプションメッセージがあり、そのメッセージ内の個々のフィールドの使用を制御したい場合に便利です。たとえば：

```proto
message MyOptions {
  string file_only_option = 1 [targets = TARGET_TYPE_FILE];
  int32 message_and_enum_option = 2 [targets = TARGET_TYPE_MESSAGE,
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

`.proto` ファイルで定義されたメッセージタイプで作業するために必要な Java、Kotlin、Python、C++、Go、Ruby、Objective-C、または C# コードを生成するには、`.proto` ファイルに対してプロトコルバッファコンパイラ `protoc` を実行する必要があります。コンパイラをインストールしていない場合は、[パッケージをダウンロード](/downloads) して README の手順に従ってください。Go の場合、コンパイラ用の特別なコードジェネレータプラグインもインストールする必要があります。これについては、GitHub の [golang/protobuf](https://github.com/golang/protobuf/) リポジトリで見つけることができ、インストール手順も記載されています。

プロトコルコンパイラは次のように呼び出されます：

```sh
protoc --proto_path=IMPORT_PATH --cpp_out=DST_DIR --java_out=DST_DIR --python_out=DST_DIR --go_out=DST_DIR --ruby_out=DST_DIR --objc_out=DST_DIR --csharp_out=DST_DIR path/to/file.proto
```

*   `IMPORT_PATH`は、`import`ディレクティブを解決する際に`.proto`ファイルを探すディレクトリを指定します。省略した場合、現在のディレクトリが使用されます。複数のインポートディレクトリを指定するには、`--proto_path`オプションを複数回渡すことができます。順番に検索されます。`-I=_IMPORT_PATH_`は、`--proto_path`の短縮形として使用できます。

*   1つ以上の*出力ディレクティブ*を指定できます：

    *   `--cpp_out`は、`DST_DIR`にC++コードを生成します。詳細については、[C++生成コードリファレンス](/reference/cpp/cpp-generated)を参照してください。
    *   `--java_out`は、`DST_DIR`にJavaコードを生成します。詳細については、[Java生成コードリファレンス](/reference/java/java-generated)を参照してください。
    *   `--kotlin_out`は、`DST_DIR`に追加のKotlinコードを生成します。詳細については、[Kotlin生成コードリファレンス](/reference/kotlin/kotlin-generated)を参照してください。
    *   `--python_out`は、`DST_DIR`にPythonコードを生成します。詳細については、[Python生成コードリファレンス](/reference/python/python-generated)を参照してください。
    *   `--go_out`は、`DST_DIR`にGoコードを生成します。詳細については、[Go生成コードリファレンス](/reference/go/go-generated)を参照してください。
    *   `--ruby_out`は、`DST_DIR`にRubyコードを生成します。詳細については、[Ruby生成コードリファレンス](/reference/ruby/ruby-generated)を参照してください。
    *   `--objc_out`は、`DST_DIR`にObjective-Cコードを生成します。詳細については、[Objective-C生成コードリファレンス](/reference/objective-c/objective-c-generated)を参照してください。
    *   `--csharp_out`は、`DST_DIR`にC#コードを生成します。詳細については、[C#生成コードリファレンス](/reference/csharp/csharp-generated)を参照してください。
    *   `--php_out`は、`DST_DIR`にPHPコードを生成します。詳細については、[PHP生成コードリファレンス](/reference/php/php-generated)を参照してください。

    さらなる便宜のために、`DST_DIR`が`.zip`または`.jar`で終わる場合、コンパイラは指定された名前の単一のZIP形式アーカイブファイルに出力を書き込みます。`.jar`出力には、Java JAR仕様で必要とされるマニフェストファイルも付属します。出力アーカイブがすでに存在する場合は上書きされることに注意してください。
```

*   入力として1つ以上の`.proto`ファイルを提供する必要があります。複数の`.proto`ファイルを一度に指定できます。ファイルは現在のディレクトリを基準として名前が付けられていますが、各ファイルは`IMPORT_PATH`のいずれかに存在している必要があります。これにより、コンパイラがその正規名を決定できます。

## ファイルの場所 {#location}

`.proto`ファイルを他の言語ソースと同じディレクトリに置かないことをお勧めします。プロジェクトのルートパッケージの下に`.proto`ファイル用のサブパッケージ`proto`を作成することを検討してください。

### 言語に依存しない場所 {#location-language-agnostic}

Javaコードを扱う場合、関連する`.proto`ファイルをJavaソースと同じディレクトリに置くと便利です。ただし、他の言語のコードが同じprotoを使用する場合、パスの接頭辞はもはや意味をなさなくなります。そのため、一般的には、`//myteam/mypackage`などの関連する言語に依存しないディレクトリにprotoを配置してください。

このルールの例外は、protoがテストなどでJavaコンテキストでのみ使用されることが明らかな場合です。

## サポートされるプラットフォーム {#platforms}

以下の情報については:

*   サポートされているオペレーティングシステム、コンパイラ、ビルドシステム、およびC++のバージョンについては、[Foundational C++ Support Policy](https://opensource.google/documentation/policies/cplusplus-support)を参照してください。
*   サポートされているPHPのバージョンについては、[Supported PHP versions](https://cloud.google.com/php/getting-started/supported-php-versions)を参照してください。
