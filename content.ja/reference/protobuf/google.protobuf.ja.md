## インデックス {#any}

-   [`Any`](#any) (メッセージ)
-   [`Api`](#api) (メッセージ)
-   [`BoolValue`](#bool-value) (メッセージ)
-   [`BytesValue`](#bytes-value) (メッセージ)
-   [`DoubleValue`](#double-value) (メッセージ)
-   [`Duration`](#duration) (メッセージ)
-   [`Empty`](#empty) (メッセージ)
-   [`Enum`](#enum) (メッセージ)
-   [`EnumValue`](#enum-value) (メッセージ)
-   [`Field`](#field) (メッセージ)
-   [`Field.Cardinality`](#field-cardinality) (列挙型)
-   [`Field.Kind`](#field-kind) (列挙型)
-   [`FieldMask`](#field-mask) (メッセージ)
-   [`FloatValue`](#float-value) (メッセージ)
-   [`Int32Value`](#int32-value) (メッセージ)
-   [`Int64Value`](#int64-value) (メッセージ)
-   [`ListValue`](#list-value) (メッセージ)
-   [`Method`](#method) (メッセージ)
-   [`Mixin`](#mixin) (メッセージ)
-   [`NullValue`](#null-value) (列挙型)
-   [`Option`](#option) (メッセージ)
-   [`SourceContext`](#source-context) (メッセージ)
-   [`StringValue`](#string-value) (メッセージ)
-   [`Struct`](#struct) (メッセージ)
-   [`Syntax`](#syntax) (列挙型)
-   [`Timestamp`](#timestamp) (メッセージ)
-   [`Type`](#type) (メッセージ)
-   [`UInt32Value`](#uint32-value) (メッセージ)
-   [`UInt64Value`](#uint64-value) (メッセージ)
-   [`Value`](#value) (メッセージ)

## Any {#any}

`Any` は、シリアル化されたメッセージと、シリアル化されたメッセージのタイプを説明する URL を含んでいます。

#### JSON {#json}

`Any` 値の JSON 表現は、デシリアライズされた埋め込まれたメッセージの通常の表現を使用し、追加の `@type` フィールドが含まれています。このフィールドにはタイプの URL が含まれます。例:

```proto
package google.profile;
message Person {
  string first_name = 1;
  string last_name = 2;
}
```

```json
{
  "@type": "type.googleapis.com/google.profile.Person",
  "firstName": <string>,
  "lastName": <string>
}
```

埋め込まれたメッセージのタイプが既知の場合でカスタム JSON 表現がある場合、その表現は `value` フィールドを追加して埋め込まれ、カスタム JSON を保持します。例 (メッセージ `google.protobuf.Duration` の場合):

```json
{
  "@type": "type.googleapis.com/google.protobuf.Duration",
  "value": "1.212s"
}
```

<table id="google.protobuf.Any.FIELDS">
  <thead>
    <tr>
      <th>フィールド名</th>
      <th>タイプ</th>
      <th>説明</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><code>type_url</code></td>
      <td><code class="apitype">string</code></td>
      <td><p>シリアル化されたメッセージのタイプを説明する URL/リソース名。</p><p>スキーマ <code>http</code>、<code>https</code>、またはスキーマなしを使用する URL の場合、次の制限と解釈が適用されます:</p>
<ul>
  <li>スキーマが提供されていない場合、<code>https</code> が想定されます。</li>
  <li>URL のパスの最後のセグメントは、タイプの完全修飾名を表す必要があります (例: <code>path/google.protobuf.Duration</code> として)。</li>
  <li>URL への HTTP GET は、バイナリ形式で <code><a href="#type">google.protobuf.Type</a></code> の値を返すか、エラーを発生させる必要があります。</li>
  <li>アプリケーションは、URL に基づいて検索結果をキャッシュしたり、バイナリに事前コンパイルして検索を回避することができます。そのため、タイプの変更時にバイナリ互換性を維持する必要があります (破壊的な変更を管理するためにバージョン付きのタイプ名を使用してください)。</li>
</ul><p><code>http</code>、<code>https</code> (または空のスキーマ)以外のスキーマは、実装固有のセマンティクスで使用される可能性があります。</p></td>
    </tr>
    <tr>
      <td><code>value</code></td>
      <td><code class="apitype">bytes</code></td>
      <td>上記指定されたタイプの有効なシリアル化されたデータである必要があります。</td>
    </tr>
  </tbody>
</table>

## Api {#api}

Apiはプロトコルバッファサービスのための軽量な記述子です。

<table>
  <thead>
    <tr>
      <th>フィールド名</th>
      <th>タイプ</th>
      <th>説明</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><code>name</code></td>
      <td><code>string</code></td>
      <td>
        このAPIの完全修飾名。パッケージ名に続いてAPIの単純な名前が続きます。
      </td>
    </tr>
    <tr>
      <td><code>methods</code></td>
      <td>
        <code><a href="#method">Method</a></code>
      </td>
      <td>このAPIのメソッド。順序は未指定です。</td>
    </tr>
    <tr>
      <td><code>options</code></td>
      <td>
        <code><a href="#option">Option</a></code>
      </td>
      <td>APIに添付されたメタデータ。</td>
    </tr>
    <tr>
      <td><code>version</code></td>
      <td><code>string</code></td>
      <td>
        <p>
          このAPIのバージョン文字列。指定されている場合、形式は
          <code>major-version.minor-version</code>でなければなりません。例：<code>1.10</code>。
          マイナーバージョンが省略された場合、ゼロにデフォルトします。バージョンフィールド全体が空の場合、
          パッケージ名からメジャーバージョンが導かれます。フィールドが空でない場合、パッケージ名のバージョンは
          ここで提供されたものと一致していることが検証されます。
        </p>
        <p>
          バージョニングスキーマは、
          <a href="http://semver.org">セマンティックバージョニング</a>を使用し、
          メジャーバージョン番号は破壊的変更を示し、マイナーバージョンは追加的で非破壊的な変更を示します。
          両方のバージョン番号は、異なるバージョンから何を期待するかをユーザーに示す信号であり、
          製品計画に基づいて慎重に選択する必要があります。
        </p>
        <p>
          メジャーバージョンは、APIのパッケージ名にも反映され、
          <code>v&lt;major-version&gt;</code>で終わる必要があります。例：<code>google.feature.v1</code>。
          メジャーバージョン0および1の場合、接尾辞は省略できます。ゼロのメジャーバージョンは、
          実験的でGAでないAPIにのみ使用する必要があります。
        </p>
      </td>
    </tr>
    <tr>
      <td><code>source_context</code></td>
      <td>
        <code><a href="#source-context">SourceContext</a></code>
      </td>
      <td>
        このメッセージによって表されるプロトコルバッファサービスのソースコンテキスト。
      </td>
    </tr>
    <tr>
      <td><code>mixins</code></td>
      <td>
        <code><a href="#mixin">Mixin</a></code>
      </td>
      <td>
        含まれるAPI。詳細は
        <code><a href="#mixin">Mixin</a></code>を参照してください。
      </td>
    </tr>
    <tr>
      <td><code>syntax</code></td>
      <td>
        <code><a href="#syntax">Syntax</a></code>
      </td>
      <td>サービスのソース構文。</td>
    </tr>
  </tbody>
</table>

## BoolValue {#bool-value}

`bool` のためのラッパーメッセージ。

`BoolValue` の JSON 表現は JSON `true` と `false` です。

<table>
  <thead>
    <tr>
      <th>フィールド名</th>
      <th>タイプ</th>
      <th>説明</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><code>value</code></td>
      <td><code class="apitype">bool</code></td>
      <td>ブール値。</td>
    </tr>
  </tbody>
</table>
{/*examples*/}

## BytesValue {#bytes-value}

`bytes` のためのラッパーメッセージ。

`BytesValue` の JSON 表現は JSON 文字列です。

<table>
  <thead>
    <tr>
      <th>フィールド名</th>
      <th>タイプ</th>
      <th>説明</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><code>value</code></td>
      <td><code>bytes</code></td>
      <td>バイト値。</td>
    </tr>
  </tbody>
</table>
{/*examples*/}

## DoubleValue {#double-value}

`double` のためのラッパーメッセージ。

`DoubleValue` の JSON 表現は JSON 数値です。

<table>
  <thead>
    <tr>
      <th>フィールド名</th>
      <th>タイプ</th>
      <th>説明</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><code>value</code></td>
      <td><code>double</code></td>
      <td>倍精度浮動小数点数値。</td>
    </tr>
  </tbody>
</table>
{/*examples*/}

## Duration {#duration}

Duration は、秒数とナノ秒単位の秒数のカウントで表される、符号付きの固定長時間スパンを表します。これは任意のカレンダーや「日」や「月」といった概念とは独立しています。Timestamp と関連があり、2 つの Timestamp 値の差は Duration であり、Timestamp から加算または減算することができます。範囲は約 +-10,000 年です。

例 1: 疑似コードで 2 つの Timestamps から Duration を計算します。

```c
Timestamp start = ...;
Timestamp end = ...;
Duration duration = ...;

duration.seconds = end.seconds - start.seconds;
duration.nanos = end.nanos - start.nanos;

if (duration.seconds < 0 && duration.nanos > 0) {
  duration.seconds += 1;
  duration.nanos -= 1000000000;
} else if (duration.seconds > 0 && duration.nanos < 0) {
  duration.seconds -= 1;
  duration.nanos += 1000000000;
}
```

例 2: 疑似コードで Timestamp + Duration から Timestamp を計算します。

```c
Timestamp start = ...;
Duration duration = ...;
Timestamp end = ...;

end.seconds = start.seconds + duration.seconds;
end.nanos = start.nanos + duration.nanos;

if (end.nanos < 0) {
  end.seconds -= 1;
  end.nanos += 1000000000;
} else if (end.nanos >= 1000000000) {
  end.seconds += 1;
  end.nanos -= 1000000000;
}
```

`Duration` の JSON 表現は、秒数を示す `String` であり、秒数の前に `s` が付いており、ナノ秒は小数秒として表されます。

<table>
  <thead>
    <tr>
      <th>フィールド名</th>
      <th>タイプ</th>
      <th>説明</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><code>seconds</code></td>
      <td><code>int64</code></td>
      <td>
        時間スパンの符号付き秒数。-315,576,000,000 から +315,576,000,000 までの範囲内である必要があります。
      </td>
    </tr>
    <tr>
      <td><code>nanos</code></td>
      <td><code>int32</code></td>
      <td>
        時間スパンのナノ秒単位の符号付き秒数。1 秒未満の Duration は、0 の <code>seconds</code> フィールドと正または負の <code>nanos</code> フィールドで表されます。1 秒以上の Duration の場合、<code>nanos</code> フィールドの非ゼロ値は、<code>seconds</code> フィールドと同じ符号である必要があります。-999,999,999 から +999,999,999 までの範囲内である必要があります。
      </td>
    </tr>
  </tbody>
</table>
{/*examples*/}

## 空 {#empty}

API で重複した空メッセージを定義するのを避けるために再利用できる汎用空メッセージ。典型的な例は、API メソッドのリクエストまたはレスポンスの型として使用することです。たとえば：

```proto
service Foo {
  rpc Bar(google.protobuf.Empty) returns (google.protobuf.Empty);
}
```

`Empty` の JSON 表現は空の JSON オブジェクト `{}` です。

## 列挙型 {#enum}

列挙型の定義

<table>
  <thead>
    <tr>
      <th>フィールド名</th>
      <th>型</th>
      <th>説明</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><code>name</code></td>
      <td><code>string</code></td>
      <td>列挙型の名前。</td>
    </tr>
    <tr>
      <td><code>enumvalue</code></td>
      <td>
        <code><a href="#enum-value">EnumValue</a></code>
      </td>
      <td>列挙値の定義。</td>
    </tr>
    <tr>
      <td><code>options</code></td>
      <td>
        <code><a href="#option">Option</a></code>
      </td>
      <td>プロトコルバッファのオプション。</td>
    </tr>
    <tr>
      <td><code>source_context</code></td>
      <td>
        <code><a href="#source-context">SourceContext</a></code>
      </td>
      <td>ソースコンテキスト。</td>
    </tr>
    <tr>
      <td><code>syntax</code></td>
      <td>
        <code><a href="#syntax">Syntax</a></code>
      </td>
      <td>ソース構文。</td>
    </tr>
  </tbody>
</table>

## EnumValue {#enum-value}

列挙値の定義。

<table>
  <thead>
    <tr>
      <th>フィールド名</th>
      <th>型</th>
      <th>説明</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><code>name</code></td>
      <td><code>string</code></td>
      <td>列挙値の名前。</td>
    </tr>
    <tr>
      <td><code>number</code></td>
      <td><code>int32</code></td>
      <td>列挙値の数値。</td>
    </tr>
    <tr>
      <td><code>options</code></td>
      <td>
        <code><a href="#option">Option</a></code>
      </td>
      <td>プロトコルバッファのオプション。</td>
    </tr>
  </tbody>
</table>

## フィールド {#field}

メッセージ型の単一フィールド。

<table>
  <thead>
    <tr>
      <th>フィールド名</th>
      <th>型</th>
      <th>説明</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><code>kind</code></td>
      <td>
        <code><a href="#field-kind">Kind</a></code>
      </td>
      <td>フィールドの型。</td>
    </tr>
    <tr>
      <td><code>cardinality</code></td>
      <td>
        <code><a href="#field-cardinality">Cardinality</a></code>
      </td>
      <td>フィールドの基数。</td>
    </tr>
    <tr>
      <td><code>number</code></td>
      <td><code>int32</code></td>
      <td>フィールド番号。</td>
    </tr>
    <tr>
      <td><code>name</code></td>
      <td><code>string</code></td>
      <td>フィールド名。</td>
    </tr>
    <tr>
      <td><code>type_url</code></td>
      <td><code>string</code></td>
      <td>
        メッセージまたは列挙型のためのスキームを除いたフィールドの型 URL。例：
        <code>&quot;type.googleapis.com/google.protobuf.Timestamp&quot;</code>。
      </td>
    </tr>
    <tr>
      <td><code>oneof_index</code></td>
      <td><code>int32</code></td>
      <td>
        メッセージまたは列挙型の <code>Type.oneofs</code> 内のフィールド型のインデックス。最初の型はインデックス 1 で、ゼロはリストにないことを意味します。
      </td>
    </tr>
    <tr>
      <td><code>packed</code></td>
      <td><code>bool</code></td>
      <td>代替パックされたワイヤ表現を使用するかどうか。</td>
    </tr>
    <tr>
      <td><code>options</code></td>
      <td>
        <code><a href="#option">Option</a></code>
      </td>
      <td>プロトコルバッファのオプション。</td>
    </tr>
    <tr>
      <td><code>json_name</code></td>
      <td><code>string</code></td>
      <td>フィールドの JSON 名。</td>
    </tr>
    <tr>
      <td><code>default_value</code></td>
      <td><code>string</code></td>
      <td>
        このフィールドのデフォルト値の文字列値。Proto2 構文のみ。
      </td>
    </tr>
  </tbody>
</table>

## カーディナリティ {#field-cardinality}

フィールドがオプション、必須、または繰り返しのいずれであるか。

<table>
  <thead>
    <tr>
      <th>Enum値</th>
      <th>説明</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><code>CARDINALITY_UNKNOWN</code></td>
      <td>カーディナリティが不明なフィールド用。</td>
    </tr>
    <tr>
      <td><code>CARDINALITY_OPTIONAL</code></td>
      <td>オプションのフィールド用。</td>
    </tr>
    <tr>
      <td><code>CARDINALITY_REQUIRED</code></td>
      <td>必須のフィールド用。Proto2構文のみ。</td>
    </tr>
    <tr>
      <td><code>CARDINALITY_REPEATED</code></td>
      <td>繰り返しのフィールド用。</td>
    </tr>
  </tbody>
</table>

## 種類 {#field-kind}

基本的なフィールドの種類。

<table class="matchpre">
  <thead>
    <tr>
      <th>Enum値</th>
      <th>説明</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><code>TYPE_UNKNOWN</code></td>
      <td>フィールドの種類が不明。</td>
    </tr>
    <tr>
      <td><code>TYPE_DOUBLE</code></td>
      <td>フィールドの種類 double。</td>
    </tr>
    <tr>
      <td><code>TYPE_FLOAT</code></td>
      <td>フィールドの種類 float。</td>
    </tr>
    <tr>
      <td><code>TYPE_INT64</code></td>
      <td>フィールドの種類 int64。</td>
    </tr>
    <tr>
      <td><code>TYPE_UINT64</code></td>
      <td>フィールドの種類 uint64。</td>
    </tr>
    <tr>
      <td><code>TYPE_INT32</code></td>
      <td>フィールドの種類 int32。</td>
    </tr>
    <tr>
      <td><code>TYPE_FIXED64</code></td>
      <td>フィールドの種類 fixed64。</td>
    </tr>
    <tr>
      <td><code>TYPE_FIXED32</code></td>
      <td>フィールドの種類 fixed32。</td>
    </tr>
    <tr>
      <td><code>TYPE_BOOL</code></td>
      <td>フィールドの種類 bool。</td>
    </tr>
    <tr>
      <td><code>TYPE_STRING</code></td>
      <td>フィールドの種類 string。</td>
    </tr>
    <tr>
      <td><code>TYPE_GROUP</code></td>
      <td>フィールドの種類 group。Proto2構文のみで、非推奨。</td>
    </tr>
    <tr>
      <td><code>TYPE_MESSAGE</code></td>
      <td>フィールドの種類 message。</td>
    </tr>
    <tr>
      <td><code>TYPE_BYTES</code></td>
      <td>フィールドの種類 bytes。</td>
    </tr>
    <tr>
      <td><code>TYPE_UINT32</code></td>
      <td>フィールドの種類 uint32。</td>
    </tr>
    <tr>
      <td><code>TYPE_ENUM</code></td>
      <td>フィールドの種類 enum。</td>
    </tr>
    <tr>
      <td><code>TYPE_SFIXED32</code></td>
      <td>フィールドの種類 sfixed32。</td>
    </tr>
    <tr>
      <td><code>TYPE_SFIXED64</code></td>
      <td>フィールドの種類 sfixed64。</td>
    </tr>
    <tr>
      <td><code>TYPE_SINT32</code></td>
      <td>フィールドの種類 sint32。</td>
    </tr>
    <tr>
      <td><code>TYPE_SINT64</code></td>
      <td>フィールドの種類 sint64。</td>
    </tr>
  </tbody>
</table>

## FieldMask {#field-mask}

`FieldMask`は、例えば次のような一連の象徴的なフィールドパスを表します：

```proto
paths: "f.a"
paths: "f.b.d"
```

ここで、`f`はあるルートメッセージ内のフィールドを表し、`a`と`b`は`f`内で見つかるメッセージのフィールドを、`d`は`f.b`内のメッセージで見つかるフィールドを表します。

FieldMaskは、get操作によって返されるフィールドのサブセットを指定したり（*プロジェクション*）、更新操作によって変更されるフィールドを指定するために使用されます。FieldMaskにはカスタムJSONエンコーディングもあります（以下参照）。

#### プロジェクション内のField Masks {#field-masks-projections}

`FieldMask`が*プロジェクション*を指定する場合、APIは応答メッセージ（またはサブメッセージ）を、マスクで指定されたフィールドのみを含むようにフィルタリングします。例えば、この\"事前マスキング\"応答メッセージを考えてみてください：

```proto
f {
  a : 22
  b {
    d : 1
    x : 2
  }
  y : 13
}
z: 8
```

前述の例でマスクを適用した後、APIの応答にはフィールドx、y、またはzの特定の値が含まれず、その値はデフォルトに設定され、protoテキスト出力では省略されます：

```proto
f {
  a : 22
  b {
    d : 1
  }
}
```

繰り返しフィールドは、フィールドマスクの最後の位置を除いて許可されません。

get操作に`FieldMask`オブジェクトが存在しない場合、操作はすべてのフィールドに適用されます（すべてのフィールドのFieldMaskが指定されたかのように）。

フィールドマスクが必ずしもトップレベルの応答メッセージに適用されるわけではありません。REST get操作の場合、フィールドマスクは応答に直接適用されますが、RESTリスト操作の場合、マスクは代わりに返されたリソースリスト内の各個々のメッセージに適用されます。RESTカスタムメソッドの場合、他の定義が使用される場合があります。マスクが適用される場所は、API内での宣言と明確に文書化されます。いずれの場合も、返されるリソース/リソースへの影響はAPIの必須動作です。

#### 更新操作内のField Masks {#field-masks-updates}

更新操作内のフィールドマスクは、対象リソースのどのフィールドが更新されるかを指定します。APIは、マスクで指定されたフィールドの値のみを変更し、他のフィールドは変更されないようにする必要があります。更新された値を記述するためにリソースが渡された場合、APIはマスクでカバーされていないすべてのフィールドの値を無視します。

フィールドの値をデフォルトにリセットするには、そのフィールドがマスク内にある必要があり、提供されたリソースでデフォルト値に設定されている必要があります。したがって、リソースのすべてのフィールドをリセットするには、リソースのデフォルトインスタンスを提供し、マスク内のすべてのフィールドを設定するか、以下に記載されているようにマスクを提供しないでください。

更新時にフィールドマスクが存在しない場合、操作はすべてのフィールドに適用されます（すべてのフィールドのフィールドマスクが指定されたかのように）。スキーマ進化が存在する場合、これはクライアントが知らないフィールドがあり、したがってリクエストに埋め込まれていないフィールドがデフォルトにリセットされることを意味します。これが望ましくない動作である場合、特定のサービスはクライアントに常にフィールドマスクを指定することを要求し、指定されていない場合はエラーを生成するかもしれません。

取得操作と同様に、リクエストメッセージ内で更新された値を記述するリソースの場所は操作の種類に依存します。いずれの場合も、フィールドマスクの効果はAPIによって尊重される必要があります。

##### HTTP REST の考慮事項 {#http-rest}

フィールドマスクを使用する更新操作のHTTP種類は、HTTPセマンティクスを満たすために PUT の代わりに PATCH に設定する必要があります（PUT は完全な更新にのみ使用される必要があります）。

#### フィールドマスクの JSON エンコーディング {#json-encoding-field-masks}

JSON では、フィールドマスクはパスがコンマで区切られた単一の文字列としてエンコードされます。各パス内のフィールド名は、lower-camel 命名規則に変換されます。

例として、次のメッセージ宣言を考えてみます：

```proto
message Profile {
  User user = 1;
  Photo photo = 2;
}
message User {
  string display_name = 1;
  string address = 2;
}
```

Proto では、`Profile` のフィールドマスクは次のように見えるかもしれません：

```proto
mask {
  paths: "user.display_name"
  paths: "photo"
}
```

JSON では、同じマスクは以下のように表されます：

```json
{
  mask: "user.displayName,photo"
}
```

<table>
  <thead>
    <tr>
      <th>フィールド名</th>
      <th>タイプ</th>
      <th>説明</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><code>paths</code></td>
      <td><code>string</code></td>
      <td>フィールドマスクパスのセット。</td>
    </tr>
  </tbody>
</table>

## FloatValue {#float-value}

`float` のためのラッパーメッセージ。

JSON表現における`FloatValue`はJSON数値です。

<table>
  <thead>
    <tr>
      <th>フィールド名</th>
      <th>タイプ</th>
      <th>説明</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><code>value</code></td>
      <td><code>float</code></td>
      <td>浮動小数点値です。</td>
    </tr>
  </tbody>
</table>

## Int32Value {#int32-value}

`int32`のためのラッパーメッセージ。

`Int32Value`のJSON表現はJSON数値です。

<table>
  <thead>
    <tr>
      <th>フィールド名</th>
      <th>タイプ</th>
      <th>説明</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><code>value</code></td>
      <td><code>int32</code></td>
      <td>int32の値です。</td>
    </tr>
  </tbody>
</table>

## Int64Value {#int64-value}

`int64`のためのラッパーメッセージ。

`Int64Value`のJSON表現はJSON文字列です。

<table>
  <thead>
    <tr>
      <th>フィールド名</th>
      <th>タイプ</th>
      <th>説明</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><code>value</code></td>
      <td><code>int64</code></td>
      <td>int64の値です。</td>
    </tr>
  </tbody>
</table>

## ListValue {#list-value}

`ListValue`は値の繰り返しフィールドをラップします。

`ListValue`のJSON表現はJSON配列です。

<table>
  <thead>
    <tr>
      <th>フィールド名</th>
      <th>タイプ</th>
      <th>説明</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><code>values</code></td>
      <td>
        <code><a href="#value">Value</a></code>
      </td>
      <td>動的に型付けされた値の繰り返しフィールドです。</td>
    </tr>
  </tbody>
</table>

## Method {#method}

MethodはAPIのメソッドを表します。

<table>
  <thead>
    <tr>
      <th>フィールド名</th>
      <th>タイプ</th>
      <th>説明</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><code>name</code></td>
      <td><code>string</code></td>
      <td>このメソッドの単純な名前です。</td>
    </tr>
    <tr>
      <td><code>request_type_url</code></td>
      <td><code>string</code></td>
      <td>入力メッセージタイプのURLです。</td>
    </tr>
    <tr>
      <td><code>request_streaming</code></td>
      <td><code>bool</code></td>
      <td>trueの場合、リクエストはストリーミングされます。</td>
    </tr>
    <tr>
      <td><code>response_type_url</code></td>
      <td><code>string</code></td>
      <td>出力メッセージタイプのURLです。</td>
    </tr>
    <tr>
      <td><code>response_streaming</code></td>
      <td><code>bool</code></td>
      <td>trueの場合、レスポンスはストリーミングされます。</td>
    </tr>
    <tr>
      <td><code>options</code></td>
      <td>
        <code><a href="#option">Option</a></code>
      </td>
      <td>メソッドに添付されたメタデータです。</td>
    </tr>
    <tr>
      <td><code>syntax</code></td>
      <td>
        <code><a href="#syntax">Syntax</a></code>
      </td>
      <td>このメソッドのソース構文です。</td>
    </tr>
  </tbody>
</table>

## ミックスイン {#mixin}

このAPIを含めるAPIを宣言します。含めるAPIは、含まれるAPIからすべてのメソッドを再宣言する必要がありますが、ドキュメントとオプションは以下のように継承されます：

- 再宣言されたメソッドのドキュメント文字列が空である場合、元のメソッドから継承されます。

- サービス構成（http、visibility）に属する各注釈は、再宣言されたメソッドで設定されていない場合に継承されます。

- 継承されたhttp注釈がある場合、パスパターンは以下のように変更されます。バージョン接頭辞は、含まれるAPIのバージョンに置き換えられ、`root`パスが指定されている場合はそれも追加されます。

単純なミックスインの例：

```proto
package google.acl.v1;
service AccessControl {
  // Get the underlying ACL object.
  rpc GetAcl(GetAclRequest) returns (Acl) {
    option (google.api.http).get = "/v1/{resource=**}:getAcl";
  }
}

package google.storage.v2;
service Storage {
  //       rpc GetAcl(GetAclRequest) returns (Acl);

  // Get a data record.
  rpc GetData(GetDataRequest) returns (Data) {
    option (google.api.http).get = "/v2/{resource=**}";
  }
}
```

ミックスイン構成の例：

```
apis:
- name: google.storage.v2.Storage
  mixins:
  - name: google.acl.v1.AccessControl
```

ミックスイン構文は、`AccessControl`内のすべてのメソッドが`Storage`内で同じ名前とリクエスト/レスポンスタイプで宣言されることを意味します。ドキュメントジェネレータまたは注釈プロセッサは、以下のようにドキュメントと注釈を継承した後の`Storage.GetAcl`メソッドを見ることになります：

```proto
service Storage {
  // Get the underlying ACL object.
  rpc GetAcl(GetAclRequest) returns (Acl) {
    option (google.api.http).get = "/v2/{resource=**}:getAcl";
  }
  ...
}
```

パスパターンのバージョンが`v1`から`v2`に変更されたことに注目してください。

ミックスインの`root`フィールドが指定されている場合、継承されたHTTPパスが配置される相対パスである必要があります。例：

```
apis:
- name: google.storage.v2.Storage
  mixins:
  - name: google.acl.v1.AccessControl
    root: acls
```

これは、次の継承されたHTTP注釈を意味します：

```proto
service Storage {
  // Get the underlying ACL object.
  rpc GetAcl(GetAclRequest) returns (Acl) {
    option (google.api.http).get = "/v2/acls/{resource=**}:getAcl";
  }
  ...
}
```

<table>
  <thead>
    <tr>
      <th>フィールド名</th>
      <th>タイプ</th>
      <th>説明</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><code>name</code></td>
      <td><code>string</code></td>
      <td>含まれるAPIの完全修飾名。</td>
    </tr>
    <tr>
      <td><code>root</code></td>
      <td><code>string</code></td>
      <td>
        空でない場合、継承されたHTTPパスが配置されるパスを指定します。
      </td>
    </tr>
  </tbody>
</table>

## NullValue {#null-value}

`NullValue`は、`Value`型のユニオンのnull値を表すシングルトン列挙型です。

JSON表現における`NullValue`はJSON `null`です。

<table>
  <thead>
    <tr>
      <th>列挙値</th>
      <th>説明</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><code>NULL_VALUE</code></td>
      <td>Null値。</td>
    </tr>
  </tbody>
</table>

## オプション {#option}

メッセージ、フィールド、列挙型などにアタッチできるプロトコルバッファオプション。

<table>
  <thead>
    <tr>
      <th>フィールド名</th>
      <th>タイプ</th>
      <th>説明</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><code>name</code></td>
      <td><code>string</code></td>
      <td>
        オプションの名前。例えば、<code>&quot;java_package&quot;</code>。
      </td>
    </tr>
    <tr>
      <td><code>value</code></td>
      <td>
        <code><a href="#any">Any</a></code>
      </td>
      <td>
        オプションの値。例えば、<code>&quot;com.google.protobuf&quot;</code>。
      </td>
    </tr>
  </tbody>
</table>

## SourceContext {#source-context}

`SourceContext`は、protobuf要素のソースに関する情報を表します。たとえば、定義されているファイルです。

<table>
  <thead>
    <tr>
      <th>フィールド名</th>
      <th>タイプ</th>
      <th>説明</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><code>file_name</code></td>
      <td><code>string</code></td>
      <td>
        関連するprotobuf要素を含む.protoファイルのパス修飾名。例えば、<code>&quot;google/protobuf/source.proto&quot;</code>。
      </td>
    </tr>
  </tbody>
</table>

## StringValue {#string-value}

`string`のためのラッパーメッセージ。

JSON表現における`StringValue`はJSON文字列です。

<table>
  <thead>
    <tr>
      <th>フィールド名</th>
      <th>タイプ</th>
      <th>説明</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><code>value</code></td>
      <td><code>string</code></td>
      <td>文字列の値。</td>
    </tr>
  </tbody>
</table>

## Struct {#struct}

`Struct`は、動的に型付けされた値にマップされるフィールドから構成される構造化データ値を表します。一部の言語では、`Struct`はネイティブ表現でサポートされる場合があります。たとえば、JSのようなスクリプト言語では、structはオブジェクトとして表されます。その表現の詳細は、言語のprotoサポートと共に記述されています。

JSON表現における`Struct`はJSONオブジェクトです。

<table>
  <thead>
    <tr>
      <th>フィールド名</th>
      <th>タイプ</th>
      <th>説明</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><code>fields</code></td>
      <td>
        <code>map&lt;string, <a href="#value">Value</a>&gt;</code>
      </td>
      <td>動的型の値のマップ。</td>
    </tr>
  </tbody>
</table>

## 構文 {#syntax}

プロトコルバッファ要素が定義される構文。

<table>
  <thead>
    <tr>
      <th>Enum値</th>
      <th>説明</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><code>SYNTAX_PROTO2</code></td>
      <td>Syntax <code>proto2</code>。</td>
    </tr>
    <tr>
      <td><code>SYNTAX_PROTO3</code></td>
      <td>Syntax <code>proto3</code>。</td>
    </tr>
  </tbody>
</table>

## タイムスタンプ {#timestamp}

タイムゾーンやカレンダーに依存しない時点を表すタイムスタンプで、UTCのエポック時刻で秒とナノ秒の解像度で表されます。プロレプティック・グレゴリオ暦を使用しており、グレゴリオ暦を遡って紀元1年まで拡張しています。すべての分が60秒であると仮定してエンコードされており、うるう秒は「スミアリング」されているため、解釈にはうるう秒テーブルが必要ありません。範囲は0001-01-01T00:00:00Zから9999-12-31T23:59:59.999999999Zまでです。この範囲に制限することで、RFC 3339日付文字列に変換できることを保証しています。詳細は<https://www.ietf.org/rfc/rfc3339.txt>を参照してください。

例1: POSIXの`time()`からタイムスタンプを計算する。

```cpp
Timestamp timestamp;
timestamp.set_seconds(time(NULL));
timestamp.set_nanos(0);
```

例2: POSIXの`gettimeofday()`からタイムスタンプを計算する。

```cpp
struct timeval tv;
gettimeofday(&tv, NULL);

Timestamp timestamp;
timestamp.set_seconds(tv.tv_sec);
timestamp.set_nanos(tv.tv_usec * 1000);
```

例3: Win32の`GetSystemTimeAsFileTime()`からタイムスタンプを計算する。

```cpp
FILETIME ft;
GetSystemTimeAsFileTime(&ft);
UINT64 ticks = (((UINT64)ft.dwHighDateTime) << 32) | ft.dwLowDateTime;

// A Windows tick is 100 nanoseconds. Windows epoch 1601-01-01T00:00:00Z
// is 11644473600 seconds before Unix epoch 1970-01-01T00:00:00Z.
Timestamp timestamp;
timestamp.set_seconds((INT64) ((ticks / 10000000) - 11644473600LL));
timestamp.set_nanos((INT32) ((ticks % 10000000) * 100));
```

例4: Javaの`System.currentTimeMillis()`からタイムスタンプを計算する。

```java
long millis = System.currentTimeMillis();

Timestamp timestamp = Timestamp.newBuilder().setSeconds(millis / 1000)
    .setNanos((int) ((millis % 1000) * 1000000)).build();
```

例5: Pythonで現在時刻からタイムスタンプを計算する。

```py
now = time.time()
seconds = int(now)
nanos = int((now - seconds) * 10**9)
timestamp = Timestamp(seconds=seconds, nanos=nanos)
```

<table>
  <thead>
    <tr>
      <th>フィールド名</th>
      <th>タイプ</th>
      <th>説明</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><code>seconds</code></td>
      <td><code>int64</code></td>
      <td>
        1970-01-01T00:00:00ZからのUTC時刻の秒を表します。
        0001-01-01T00:00:00Zから9999-12-31T23:59:59Zまでの範囲内である必要があります。
      </td>
    </tr>
    <tr>
      <td><code>nanos</code></td>
      <td><code>int32</code></td>
      <td>
        ナノ秒の解像度での非負の秒の小数部です。負の秒値には小数部があっても、時間の前進を示す非負のナノ秒値である必要があります。
        0から999,999,999までの範囲内である必要があります。
      </td>
    </tr>
  </tbody>
</table>

## タイプ {#type}

プロトコルバッファメッセージタイプ。

<table>
  <thead>
    <tr>
      <th>フィールド名</th>
      <th>タイプ</th>
      <th>説明</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><code>name</code></td>
      <td><code>string</code></td>
      <td>完全修飾メッセージ名。</td>
    </tr>
    <tr>
      <td><code>fields</code></td>
      <td>
        <code><a href="#field">Field</a></code>
      </td>
      <td>フィールドのリスト。</td>
    </tr>
    <tr>
      <td><code>oneofs</code></td>
      <td><code>string</code></td>
      <td>
        このタイプの<code>oneof</code>定義に現れるタイプのリスト。
      </td>
    </tr>
    <tr>
      <td><code>options</code></td>
      <td>
        <code><a href="#option">Option</a></code>
      </td>
      <td>プロトコルバッファオプション。</td>
    </tr>
    <tr>
      <td><code>source_context</code></td>
      <td>
        <code><a href="#source-context">SourceContext</a></code>
      </td>
      <td>ソースコンテキスト。</td>
    </tr>
    <tr>
      <td><code>syntax</code></td>
      <td>
        <code><a href="#syntax">Syntax</a></code>
      </td>
      <td>ソース構文。</td>
    </tr>
  </tbody>
</table>

## UInt32Value {#uint32-value}

`uint32` のためのラッパーメッセージ。

`UInt32Value` のJSON表現はJSON数値です。

<table>
  <thead>
    <tr>
      <th>フィールド名</th>
      <th>タイプ</th>
      <th>説明</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><code>value</code></td>
      <td><code>uint32</code></td>
      <td>uint32の値。</td>
    </tr>
  </tbody>
</table>

## UInt64Value {#uint64-value}

`uint64` のためのラッパーメッセージ。

`UInt64Value` のJSON表現はJSON文字列です。

<table>
  <thead>
    <tr>
      <th>フィールド名</th>
      <th>タイプ</th>
      <th>説明</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><code>value</code></td>
      <td><code>uint64</code></td>
      <td>uint64の値。</td>
    </tr>
  </tbody>
</table>

## Value {#value}

`Value` は、null、数値、文字列、ブール値、再帰的な構造値、または値のリストのいずれかである動的型付き値を表します。値の生成者は、そのいずれかのバリアントを設定することが期待されており、どのバリアントも存在しない場合はエラーを示します。

```markdown
`Value` の JSON 表現は JSON 値です。

<table>
  <thead>
    <tr>
      <th>Field name</th>
      <th>Type</th>
      <th>Description</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td colspan="3">Union field, only one of the following:</td>
    </tr>
    <tr>
      <td><code>null_value</code></td>
      <td>
        <code><a href="#null-value">NullValue</a></code>
      </td>
      <td>ヌル値を表します。</td>
    </tr>
    <tr>
      <td><code>number_value</code></td>
      <td><code>double</code></td>
      <td>
        ダブル値を表します。NaN や Infinity をシリアライズしようとするとエラーが発生します（これらを通常のフィールドのように文字列 "NaN" や "Infinity" としてシリアライズすることはできません。それは文字列値ではなく数値値として解析されるためです）。
      </td>
    </tr>
    <tr>
      <td><code>string_value</code></td>
      <td><code>string</code></td>
      <td>文字列値を表します。</td>
    </tr>
    <tr>
      <td><code>bool_value</code></td>
      <td><code>bool</code></td>
      <td>真偽値を表します。</td>
    </tr>
    <tr>
      <td><code>struct_value</code></td>
      <td>
        <code><a href="#struct">Struct</a></code>
      </td>
      <td>構造化された値を表します。</td>
    </tr>
    <tr>
      <td><code>list_value</code></td>
      <td>
        <code><a href="#list-value">ListValue</a></code>
      </td>
      <td><code>Value</code> の繰り返しを表します。</td>
    </tr>
  </tbody>
</table>
```
