+++
title = "Proto ベストプラクティス"
weight = 90
description = "Protocol Buffers の作成に関する検証済みのベストプラクティスを共有します。"
type = "docs"
+++

クライアントとサーバーは常に正確に同時に更新されるわけではありません - 同時に更新しようとしても。どちらかがロールバックされることがあります。クライアントとサーバーが同期しているからといって、破壊的な変更を行っても大丈夫だと思わないでください。

<a id="dont-re-use-a-tag-number"></a>

## **タグ番号を再利用しないでください** {#reuse-number}

タグ番号を再利用しないでください。これは逆シリアル化を狂わせます。フィールドが使用されていないと思っていても、タグ番号を再利用しないでください。変更がライブであった場合、どこかのログにあなたの proto のシリアル化バージョンがあるかもしれません。また、他のサーバーに古いコードがあって、それが壊れる可能性があります。

<a id="do-reserve-tag-numbers-for-deleted-fields"></a>

## **削除されたフィールドのためにタグ番号を予約する** {#reserve-tag-numbers}

もはや使用されていないフィールドを削除する場合は、将来誰もそれを誤って再利用しないように、そのタグ番号を予約してください。`reserved 2, 3;` だけで十分です。型は必要ありません（依存関係を削減できます！）。また、削除されたフィールド名を再利用しないように名前を予約することもできます：`reserved "foo", "bar";`。

<a id="do-reserve-numbers-for-deleted-enum-values"></a>

## **削除された列挙値のために番号を予約する** {#reserve-deleted-numbers}

もはや使用されていない列挙値を削除する場合は、将来誰もそれを誤って再利用しないように、その番号を予約してください。`reserved 2, 3;` だけで十分です。また、削除された値名を再利用しないように名前を予約することもできます：`reserved "FOO", "BAR";`。

<a id="dont-change-the-type-of-a-field"></a>

## **フィールドの型を変更しないでください** {#change-type}

ほとんどの場合、フィールドの型を変更しないでください。これは逆シリアル化を狂わせます。タグ番号を再利用するのと同じです。ただし、[protobuf ドキュメント](/programming-guides/proto2#updating)では、いくつかの例外が記載されています（たとえば、`int32`、`uint32`、`int64`、`bool` の間を移動する場合など）。ただし、フィールドのメッセージタイプを変更すると、新しいメッセージが古いメッセージのスーパーセットでない限り、**壊れます**。


<a id="dont-add-a-required-field"></a>

## **必須** フィールドを追加しない {#add-required}

必須フィールドを追加せず、代わりにAPIの契約を文書化するために `// required` を追加してください。必須フィールドは、proto3 から完全に削除されたため、多くの人々にとって有害と見なされています。すべてのフィールドをオプションまたは繰り返し可能にしてください。メッセージタイプがどれだけ長持ちするか、将来的に必須ではなくなったときに誰かが必須フィールドに空の文字列やゼロを入力することが強制されるかどうかはわかりませんが、proto はそれが必要であると言っているかもしれません。

proto3 では `required` フィールドは存在しないため、このアドバイスは適用されません。

<a id="dont-make-a-message-with-lots-of-fields"></a>

## **多数の** フィールドを持つメッセージを作成しない {#lots-of-fields}

“多数”（数百と考えてください）のフィールドを持つメッセージを作成しないでください。C++ では、各フィールドがメモリ内オブジェクトサイズに約 65 ビットを追加します。それが設定されているかどうかに関わらず（ポインタ用の 8 バイトと、フィールドがオプションとして宣言されている場合、フィールドが設定されているかどうかを追跡するビットフィールド内の別のビット）。proto が大きくなりすぎると、生成されたコードはコンパイルすらできなくなるかもしれません（たとえば、Java ではメソッドのサイズにはハードリミットがあります）。

<a id="do-include-an-unspecified-value-in-an-enum"></a>

## 列挙型に未指定の値を含める {#unspecified-enum}

列挙型には、宣言の最初の値としてデフォルトの `FOO_UNSPECIFIED` 値を含めるべきです。proto2 列挙型に新しい値が追加されると、古いクライアントはフィールドを未設定として表示し、ゲッターはデフォルト値またはデフォルトが存在しない場合は最初に宣言された値を返します。[proto enums][proto-enums] と一貫した動作をするために、最初に宣言された列挙値はデフォルトの `FOO_UNSPECIFIED` 値であるべきであり、タグ 0 を使用するべきです。このデフォルト値を意味のある値として宣言することは誘惑されるかもしれませんが、一般的なルールとして、新しい列挙値が時間とともに追加されるプロトコルの進化を助けるために、しないでください。コンテナメッセージの下に宣言されたすべての列挙値は同じ C++ 名前空間にありますので、コンパイルエラーを回避するために、未指定の値を列挙型の名前で接頭辞付きで使用してください。クロス言語定数が必要ない場合は、`int32` を使用すると未知の値が保持され、生成されるコードが少なくなります。[proto enums][proto-enums] では、最初の値がゼロである必要があり、未知の列挙値をラウンドトリップ（逆シリアル化、シリアル化）できます。

## **C/C++ マクロ定数を列挙型の値として使用しない** {#macro-constants}

C++ 言語で既に定義されている単語を使用することは避けてください。特に、`math.h` などのヘッダーで定義されている単語を使用すると、`.proto.h` の `#include` ステートメントがそのヘッダーの前に現れるとコンパイルエラーが発生する可能性があります。"`NULL`"、"`NAN`"、"`DOMAIN`" などのマクロ定数を列挙型の値として使用しないでください。

## **Well-Known Types および Common Types を使用する** {#well-known-common}

以下の共通の型を使用することを強くお勧めします。たとえば、既存の共通の型がすでに存在する場合、コード内で `int32 timestamp_seconds_since_epoch` や `int64 timeout_millis` を使用しないでください！

*   [`duration`](https://github.com/protocolbuffers/protobuf/blob/main/src/google/protobuf/duration.proto)
    は、符号付きの固定長時間スパンです（たとえば、42秒）。
*   [`timestamp`](https://github.com/protocolbuffers/protobuf/blob/main/src/google/protobuf/timestamp.proto)
    は、任意のタイムゾーンやカレンダーに依存しない時点を表します（たとえば、2017-01-15T01:30:15.01Z）。
*   [`interval`](https://github.com/googleapis/googleapis/blob/master/google/type/interval.proto)
    は、タイムゾーンやカレンダーに依存しない時間間隔を表します（たとえば、2017-01-15T01:30:15.01Z - 2017-01-16T02:30:15.01Z）。
*   [`date`](https://github.com/googleapis/googleapis/blob/master/google/type/date.proto)
    は、カレンダーの日付全体を表します（たとえば、2005-09-19）。
*   [`month`](https://github.com/googleapis/googleapis/blob/master/google/type/month.proto)
    は、年の月を表します（たとえば、4月）。
*   [`dayofweek`](https://github.com/googleapis/googleapis/blob/master/google/type/dayofweek.proto)
    は、曜日を表します（たとえば、月曜日）。
*   [`timeofday`](https://github.com/googleapis/googleapis/blob/master/google/type/timeofday.proto)
    は、1日の時間を表します（たとえば、10:42:23）。
*   [`field_mask`](https://github.com/protocolbuffers/protobuf/blob/main/src/google/protobuf/field_mask.proto)
    は、象徴的なフィールドパスのセットを表します（たとえば、f.b.d）。
*   [`postal_address`](https://github.com/googleapis/googleapis/blob/master/google/type/postal_address.proto)
    は、郵便住所を表します（たとえば、1600 Amphitheatre Parkway Mountain View, CA 94043 USA）。
*   [`money`](https://github.com/googleapis/googleapis/blob/master/google/type/money.proto)
    は、通貨タイプ付きの金額を表します（たとえば、42 USD）。
*   [`latlng`](https://github.com/googleapis/googleapis/blob/master/google/type/latlng.proto)
    は、緯度/経度のペアを表します（たとえば、緯度 37.386051、経度 -122.083855）。
*   [`color`](https://github.com/googleapis/googleapis/blob/master/google/type/color.proto)
    は、RGBA カラースペースの色を表します。


<a id="do-define-widely-used-message-types-in-separate-files"></a>

## **Do** 別ファイルで広く使用されるメッセージタイプを定義する {#separate-files}

メッセージタイプや列挙型を定義する際に、直近のチーム外でも広く使用される可能性がある場合は、依存関係のない独自のファイルに配置することを検討してください。そのようなタイプを誰でも簡単に使用できるようになり、他の proto ファイルに移行依存関係を導入することなく使用できます。

<a id="dont-change-the-default-value-of-a-field"></a>

## **Don't** フィールドのデフォルト値を変更しない {#change-default-value}

ほとんどの場合、proto フィールドのデフォルト値を変更しないでください。これにより、クライアントとサーバー間でバージョンの不一致が発生します。設定されていない値を読み取るクライアントは、プロト変更時に同じ設定されていない値を読み取るサーバーと異なる結果を見ることになります。Proto3 では、デフォルト値の設定が削除されました。

<a id="dont-go-from-repeated-to-scalar"></a>

## **Don't** Repeated から Scalar に変更しない {#repeated-to-scalar}

クラッシュを引き起こすことはありませんが、データが失われます。JSON の場合、繰り返しの不一致は*メッセージ全体*が失われます。数値の proto3 フィールドと proto2 の `packed` フィールドでは、繰り返しからスカラーに移行すると、その*フィールド*内のすべてのデータが失われます。数値以外の proto3 フィールドや注釈のない proto2 フィールドでは、繰り返しからスカラーに移行すると、最後に逆シリアル化された値が「勝利」します。

proto2 ではスカラーから繰り返しに移行することができ、proto3 では `[packed=false]` を使用することでバイナリシリアル化の際にスカラー値が1要素のリストになります。

<a id="do-follow-the-style-guide-for-generated-code"></a>

## **Do** 生成されたコードのスタイルガイドに従う {#follow-style-guide}

生成された Proto コードは通常のコードで参照されます。`.proto` ファイルのオプションがスタイルガイドに違反するコードの生成につながらないように注意してください。
例:

*   `java_outer_classname` は以下に従うべきです:
    https://google.github.io/styleguide/javaguide.html#s5.2.2-class-names

*   `java_package` と `java_alt_package` は以下に従うべきです:
    https://google.github.io/styleguide/javaguide.html#s5.2.1-package-names

*   `package`は、`java_package`が存在しない場合にJavaで使用されますが、常にC++の名前空間に直接対応し、したがってhttps://google.github.io/styleguide/cppguide.html#Namespace_Names に従う必要があります。これらのスタイルガイドが競合する場合は、Javaでは`java_package`を使用してください。

*   `ruby_package`は`Foo::Bar::Baz`の形式であるべきであり、`Foo.Bar.Baz`ではない。

<a id="never-use-text-format-messages-for-interchange"></a>

## **交換用にテキスト形式メッセージを使用しないでください** {#text-format-interchange}

テキストベースのシリアル化形式（テキスト形式やJSONなど）は、フィールドや列挙型の値を文字列として表現します。その結果、古いコードを使用してこれらの形式でプロトコルバッファを逆シリアル化すると、フィールドや列挙型の値が名前変更された場合や新しいフィールドや列挙型の値や拡張が追加された場合に失敗します。データの交換には可能な限りバイナリシリアル化を使用し、テキスト形式は人間が編集およびデバッグ目的でのみ使用してください。

APIやデータの保存にJSONに変換されたプロトを使用している場合、フィールドや列挙型を安全に名前変更することができないかもしれません。

<a id="never-rely-on-serialization-stability-across-builds"></a>

## **ビルド間でのシリアル化の安定性に依存しないでください** {#serialization-stability}

プロトのシリアル化の安定性は、バイナリ間または同じバイナリのビルド間で保証されていません。たとえば、キャッシュキーを構築する場合など、それに依存しないでください。

<a id="dont-generate-java-protos-in-the-same-java-package-as-other-code"></a>

## **他のコードと同じJavaパッケージ内にJavaプロトを生成しないでください** {#generate-java-protos}

Javaプロトソースを、手書きのJavaソースとは別のパッケージに生成してください。`package`、`java_package`、`java_alt_api_package` オプションは、[生成されたJavaソースが出力される場所](/reference/java/java-generated#package)を制御します。手書きのJavaソースコードが同じパッケージに存在しないようにしてください。一般的な慣習として、プロジェクト内の`proto`サブパッケージにプロトを生成することです（つまり、手書きのソースコードは含まれません）。

## フィールド名に言語キーワードを使用しないでください {#avoid-keywords}


もしメッセージ、フィールド、列挙型、または列挙値の名前が、そのフィールドに読み書きする言語のキーワードである場合、protobuf はフィールド名を変更し、通常のフィールドとは異なるアクセス方法を持つ可能性があります。例えば、[Python に関するこの警告](/reference/python/python-generated#keyword-conflicts)を参照してください。

また、ファイルパスにキーワードを使用しないようにすることも推奨されます。これは問題を引き起こす可能性があります。

## 付録 {#appendix}

### API ベストプラクティス {#api-best-practices}

このドキュメントでは、破損の原因となる可能性が非常に高い変更のみをリストしています。柔軟に成長する proto API を作成する方法に関する高レベルのガイダンスについては、[API ベストプラクティス](/programming-guides/api)を参照してください。
