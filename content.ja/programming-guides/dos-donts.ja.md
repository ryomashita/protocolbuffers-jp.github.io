Clients and servers are never updated at exactly the same time - even when you try to update them at the same time. One or the other may get rolled back. Don’t assume that you can make a breaking change and it'll be okay because the client and server are in sync.

<a id="dont-re-use-a-tag-number"></a>

## **Don't** Re-use a Tag Number {#reuse-number}

Never re-use a tag number. It messes up deserialization. Even if you think no one is using the field, don’t re-use a tag number. If the change was live ever, there could be serialized versions of your proto in a log somewhere. Or there could be old code in another server that will break.

<a id="do-reserve-tag-numbers-for-deleted-fields"></a>

## **Do** Reserve Tag Numbers for Deleted Fields {#reserve-tag-numbers}

When you delete a field that's no longer used, reserve its tag number so that no one accidentally re-uses it in the future. Just `reserved 2, 3;` is enough. No type required (lets you trim dependencies!). You can also reserve names to avoid recycling now-deleted field names: `reserved "foo", "bar";`.

<a id="do-reserve-numbers-for-deleted-enum-values"></a>

## **Do** Reserve Numbers for Deleted Enum Values {#reserve-deleted-numbers}

When you delete an enum value that's no longer used, reserve its number so that no one accidentally re-uses it in the future. Just `reserved 2, 3;` is enough. You can also reserve names to avoid recycling now-deleted value names: `reserved "FOO", "BAR";`.

<a id="dont-change-the-type-of-a-field"></a>

## **Don't** Change the Type of a Field {#change-type}

Almost never change the type of a field; it'll mess up deserialization, same as re-using a tag number. The [protobuf docs](/programming-guides/proto2#updating) outline a small number of cases that are okay (for example, going between `int32`, `uint32`, `int64` and `bool`). However, changing a field’s message type **will break** unless the new message is a superset of the old one.

<a id="dont-add-a-required-field"></a>

## 必須フィールドを追加しない {#add-required}

必須フィールドを追加しないでください。代わりに、APIの契約を文書化するために `// required` を追加してください。必須フィールドは、多くの人々にとって有害と見なされ、proto3から完全に削除されました。すべてのフィールドをオプションまたは繰り返し可能にしてください。メッセージタイプがどれだけ長持ちするか、将来的に必須ではなくなったときに誰かが必須フィールドに空の文字列やゼロを入力することが強制されるかどうかはわかりませんが、protoはそれが必要であるとまだ言っているかもしれません。

proto3では `required` フィールドは存在しないため、このアドバイスは適用されません。

<a id="dont-make-a-message-with-lots-of-fields"></a>

## 多くのフィールドを持つメッセージを作成しない {#lots-of-fields}

“多く”（数百と考えてください）のフィールドを持つメッセージを作成しないでください。C++では、各フィールドがメモリ内オブジェクトサイズに約65ビットを追加します。それが設定されているかどうかに関わらず（ポインタ用の8バイトと、フィールドがオプションとして宣言されている場合は、フィールドが設定されているかどうかを追跡するビットフィールド内の別のビット）。protoが大きくなりすぎると、生成されたコードがコンパイルできなくなるかもしれません（たとえば、Javaではメソッドのサイズには厳しい制限があります）。

<a id="do-include-an-unspecified-value-in-an-enum"></a>

## Enumに未指定の値を含める {#unspecified-enum}

Enumには、宣言の最初の値としてデフォルトの `FOO_UNSPECIFIED` 値を含めるべきです。proto2のenumに新しい値が追加されると、古いクライアントはフィールドを未設定として見て、getterはデフォルト値またはデフォルトが存在しない場合は最初に宣言された値を返します。一貫した動作をするために、[proto enums][proto-enums]と同様に、最初に宣言されたenum値はデフォルトの `FOO_UNSPECIFIED` 値であるべきであり、タグ0を使用するべきです。このデフォルト値を意味のある値として宣言することは誘惑されるかもしれませんが、一般的なルールとして、新しいenum値が時間とともに追加されるプロトコルの進化を助けるために、しないでください。コンテナメッセージの下に宣言されたすべてのenum値は同じC++名前空間にありますので、コンパイルエラーを回避するために、未指定の値をenumの名前で接頭辞付けてください。クロス言語の定数が必要ない場合、`int32` は未知の値を保持し、コードを少なく生成します。[proto enums][proto-enums]は最初の値をゼロにする必要があり、未知のenum値をラウンドトリップ（逆シリアル化、シリアル化）できます。

## **C/C++ マクロ定数を列挙型の値として使用しない** {#macro-constants}

C++ 言語で既に定義されている単語を使用することは避けてください。特に、`math.h` などのヘッダーで定義されている単語を使用すると、`.proto.h` の `#include` ステートメントがそのヘッダーの前に現れる場合、コンパイルエラーが発生する可能性があります。`NULL`、`NAN`、`DOMAIN` などのマクロ定数を列挙型の値として使用しないようにしてください。

{/*examples*/}

## **Well-Known Types および Common Types を使用する** {#well-known-common}

以下の共通の型を使用することを強くお勧めします。たとえば、既存の共通の型がすでに存在する場合、`int32 timestamp_seconds_since_epoch` や `int64 timeout_millis` をコード内で使用しないでください。

{/*examples*/}

*   [`duration`](https://github.com/protocolbuffers/protobuf/blob/main/src/google/protobuf/duration.proto)
    は、符号付きの固定長時間スパンです（たとえば、42秒）。
*   [`timestamp`](https://github.com/protocolbuffers/protobuf/blob/main/src/google/protobuf/timestamp.proto)
    は、任意のタイムゾーンやカレンダーに依存しない時点を表します（たとえば、2017-01-15T01:30:15.01Z）。
*   [`interval`](https://github.com/googleapis/googleapis/blob/master/google/type/interval.proto)
    は、タイムゾーンやカレンダーに依存しない時間間隔を表します（たとえば、2017-01-15T01:30:15.01Z - 2017-01-16T02:30:15.01Z）。
*   [`date`](https://github.com/googleapis/googleapis/blob/master/google/type/date.proto)
    は、完全なカレンダー日付を表します（たとえば、2005-09-19）。
*   [`month`](https://github.com/googleapis/googleapis/blob/master/google/type/month.proto)
    は、年の月を表します（たとえば、April）。
*   [`dayofweek`](https://github.com/googleapis/googleapis/blob/master/google/type/dayofweek.proto)
    は、曜日を表します（たとえば、Monday）。
*   [`timeofday`](https://github.com/googleapis/googleapis/blob/master/google/type/timeofday.proto)
    は、1日の中の時刻を表します（たとえば、10:42:23）。
*   [`field_mask`](https://github.com/protocolbuffers/protobuf/blob/main/src/google/protobuf/field_mask.proto)
    は、象徴的なフィールドパスのセットを表します（たとえば、f.b.d）。
*   [`postal_address`](https://github.com/googleapis/googleapis/blob/master/google/type/postal_address.proto)
    は、郵便住所を表します（たとえば、1600 Amphitheatre Parkway Mountain View, CA 94043 USA）。
*   [`money`](https://github.com/googleapis/googleapis/blob/master/google/type/money.proto)
    は、通貨タイプ付きの金額を表します（たとえば、42 USD）。
*   [`latlng`](https://github.com/googleapis/googleapis/blob/master/google/type/latlng.proto)
    は、緯度/経度のペアを表します（たとえば、緯度 37.386051、経度 -122.083855）。
*   [`color`](https://github.com/googleapis/googleapis/blob/master/google/type/color.proto)
    は、RGBA 色空間の色を表します。


## **Do** 別ファイルに広く使用されるメッセージタイプを定義する {#separate-files}

自分のチーム外で広く使用されることを期待/恐れ/予想しているメッセージタイプや列挙型を定義している場合は、依存関係のない独自のファイルに配置することを検討してください。そのようなタイプを使用することが誰にとっても簡単になり、他の proto ファイルに移行依存関係を導入することなく使用できます。

## **Don't** フィールドのデフォルト値を変更しない {#change-default-value}

ほとんどの場合、proto フィールドのデフォルト値を変更しないでください。これにより、クライアントとサーバー間でバージョンの不一致が発生します。設定されていない値を読み取るクライアントは、ビルドが proto の変更をまたいでいる場合、サーバーが同じ設定されていない値を読み取るときとは異なる結果を見ることになります。Proto3 では、デフォルト値の設定が削除されました。

## **Don't** Repeated から Scalar に変更しない {#repeated-to-scalar}

クラッシュを引き起こすことはありませんが、データが失われます。JSON の場合、繰り返しの不一致は*メッセージ全体*を失います。数値の proto3 フィールドと proto2 の `packed` フィールドでは、繰り返しからスカラーに移行すると、その*フィールド*内のすべてのデータが失われます。数値以外の proto3 フィールドと注釈のない proto2 フィールドでは、繰り返しからスカラーに移行すると、最後に逆シリアル化された値が「勝利」します。

proto2 では、スカラーから繰り返しに移行することは可能であり、proto3 では `[packed=false]` を使用すると、バイナリシリアル化の場合、スカラー値が1要素のリストになります。

## **Do** 生成されたコードのスタイルガイドに従う {#follow-style-guide}

Proto 生成されたコードは通常のコードで参照されます。`.proto` ファイルのオプションがスタイルガイドに違反するコードの生成につながらないようにしてください。
例:

*   `java_outer_classname` は以下に従うべきです:
    https://google.github.io/styleguide/javaguide.html#s5.2.2-class-names

*   `java_package` と `java_alt_package` は以下に従うべきです:
    https://google.github.io/styleguide/javaguide.html#s5.2.1-package-names

*   `package` は、`java_package` が存在しない場合に Java で使用されますが、常に C++ の名前空間に直接対応し、したがって https://google.github.io/styleguide/cppguide.html#Namespace_Names に従う必要があります。これらのスタイルガイドが競合する場合は、Java では `java_package` を使用してください。

*   `ruby_package` は `Foo::Bar::Baz` の形式であるべきであり、`Foo.Bar.Baz` ではないべきです。

<a id="never-use-text-format-messages-for-interchange"></a>

## **交換用に** テキスト形式メッセージを使用しない {#text-format-interchange}

テキスト形式や JSON のようなテキストベースのシリアライゼーション形式は、フィールドや列挙型の値を文字列として表現します。その結果、古いコードを使用してこれらの形式でプロトコルバッファを逆シリアライズすると、フィールドや列挙型の値が名前変更された場合や新しいフィールドや列挙型の値や拡張が追加された場合に失敗します。データの交換には可能な限りバイナリシリアライゼーションを使用し、テキスト形式は人間による編集とデバッグにのみ使用してください。

API やデータの保存に JSON に変換された proto を使用している場合、フィールドや列挙型の名前を安全に変更できないかもしれません。

<a id="never-rely-on-serialization-stability-across-builds"></a>

## **決して** ビルド間でのシリアライゼーションの安定性に依存しない {#serialization-stability}

プロトシリアライゼーションの安定性は、バイナリ間または同じバイナリのビルド間で保証されていません。たとえば、キャッシュキーを構築する場合などにそれに依存しないでください。

<a id="dont-generate-java-protos-in-the-same-java-package-as-other-code"></a>

## 同じ Java パッケージ内に Java Protos を生成しないでください {#generate-java-protos}

手書きの Java ソースとは別のパッケージに Java プロトソースを生成してください。`package`、`java_package`、`java_alt_api_package` オプションは、[生成された Java ソースが出力される場所](/reference/java/java-generated#package)を制御します。手書きの Java ソースコードが同じパッケージに存在しないようにしてください。一般的な慣習として、プロジェクト内の `proto` サブパッケージにのみそれらの proto を生成することです（つまり、手書きのソースコードは含まれません）。

## フィールド名に言語キーワードを使用しないでください {#avoid-keywords}

もしメッセージ、フィールド、列挙型、または列挙型の値の名前が、そのフィールドから読み取り/書き込みを行う言語のキーワードである場合、protobufはフィールド名を変更し、通常のフィールドとは異なるアクセス方法を持つ可能性があります。例えば、[Pythonに関するこの警告](/reference/python/python-generated#keyword-conflicts)を参照してください。

また、ファイルパスにキーワードを使用することも避けるべきです。これも問題を引き起こす可能性があります。

## 付録 {#appendix}

### API ベストプラクティス {#api-best-practices}

この文書では、壊れやすい変更のみをリストしています。柔軟に成長する proto API を作成する方法に関する高レベルのガイダンスについては、[API ベストプラクティス](/programming-guides/api)を参照してください。
