+++
title = "テキスト形式言語仕様"
weight = 820
description = "プロトコルバッファのテキスト形式言語は、protobufデータのテキスト形式での表現の構文を指定します。これは、構成やテストによく使用されます。"
type = "docs"
+++

この形式は、たとえば`.proto`スキーマ内のテキストの形式とは異なります。このドキュメントには、[ISO/IEC 14977 EBNF](https://en.wikipedia.org/wiki/Extended_Backus%E2%80%93Naur_Form)で指定された構文を使用したリファレンスドキュメントが含まれています。

{{% alert title="Note" color="note" %}}
これは、C++テキスト形式の[実装](https://github.com/protocolbuffers/protobuf/blob/master/src/google/protobuf/text_format.cc)から逆エンジニアリングされた下書き仕様であり、さらなる議論とレビューに基づいて変更される可能性があります。サポートされている言語間でテキスト形式を一貫させる努力がなされていますが、非互換性が存在する可能性があります。{{% /alert %}}

## 例 {#example}

```textproto
convolution_benchmark {
  label: "NHWC_128x20x20x56x160"
  input {
    dimension: [128, 56, 20, 20]
    data_type: DATA_HALF
    format: TENSOR_NHWC
  }
}
```

## パースの概要 {#parsing}

この仕様の言語要素は、字句的および構文的カテゴリに分かれます。字句要素は、入力テキストが説明通りに完全に一致する必要がありますが、構文要素はオプションの`WHITESPACE`および`COMMENT`トークンで分割されることができます。

たとえば、符号付き浮動小数点値は2つの構文要素で構成されます: 符号(`-`)と`FLOAT`リテラルです。符号と数値の間にはオプションの空白やコメントが存在するかもしれませんが、数値内には存在しません。例:

```textproto
value: -2.0   # Valid: no additional whitespace.
value: - 2.0  # Valid: whitespace between '-' and '2.0'.
value: -
  # comment
  2.0         # Valid: whitespace and comments between '-' and '2.0'.
value: 2 . 0  # Invalid: the floating point period is part of the lexical
              # element, so no additional whitespace is allowed.
```

特別な注意が必要なエッジケースが1つあります: 数値トークン(`FLOAT`、`DEC_INT`、`OCT_INT`、または`HEX_INT`)の直後に`IDENT`トークンが続くことはできません。例:

```textproto
foo: 10 bar: 20           # Valid: whitespace separates '10' and 'bar'
foo: 10,bar: 20           # Valid: ',' separates '10' and 'bar'
foo: 10[com.foo.ext]: 20  # Valid: '10' is followed immediately by '[', which is
                          # not an identifier.
foo: 10bar: 20            # Invalid: no space between '10' and identifier 'bar'.
```

## 字句要素 {#lexical}

以下に記載されている字句要素は、大文字の主要要素と小文字のフラグメントの2つのカテゴリに分類されます。出力トークンストリームには主要要素のみが含まれ、構文解析中に使用されます。フラグメントは主要要素の構築を簡素化するために存在します。

```textproto
value: 10   # '10' is parsed as a DEC_INT token.
value: 10f  # '10f' is parsed as a FLOAT token, despite containing '10' which
            # would also match DEC_INT. In this case, FLOAT matches a longer
            # subsequence of the input.
```

### 文字 {#characters}

```
char    = ? Any non-NUL unicode character ? ;
newline = ? ASCII #10 (line feed) ? ;

letter = "A" | "B" | "C" | "D" | "E" | "F" | "G" | "H" | "I" | "J" | "K" | "L" | "M"
       | "N" | "O" | "P" | "Q" | "R" | "S" | "T" | "U" | "V" | "W" | "X" | "Y" | "Z"
       | "a" | "b" | "c" | "d" | "e" | "f" | "g" | "h" | "i" | "j" | "k" | "l" | "m"
       | "n" | "o" | "p" | "q" | "r" | "s" | "t" | "u" | "v" | "w" | "x" | "y" | "z"
       | "_" ;

oct = "0" | "1" | "2" | "3" | "4" | "5" | "6" | "7" ;
dec = "0" | "1" | "2" | "3" | "4" | "5" | "6" | "7" | "8" | "9" ;
hex = "0" | "1" | "2" | "3" | "4" | "5" | "6" | "7" | "8" | "9"
    | "A" | "B" | "C" | "D" | "E" | "F"
    | "a" | "b" | "c" | "d" | "e" | "f" ;
```

### 空白とコメント {#whitespace}

```
COMMENT    = "#", { char - newline }, [ newline ] ;
WHITESPACE = " "
           | newline
           | ? ASCII #9  (horizontal tab) ?
           | ? ASCII #11 (vertical tab) ?
           | ? ASCII #12 (form feed) ?
           | ? ASCII #13 (carriage return) ? ;
```

### 識別子 {#identifiers}

```
IDENT = letter, { letter | dec } ;
```

### 数値リテラル {#numeric}

```
dec_lit   = "0"
          | ( dec - "0" ), { dec } ;
float_lit = ".", dec, { dec }, [ exp ]
          | dec_lit, ".", { dec }, [ exp ]
          | dec_lit, exp ;
exp       = ( "E" | "e" ), [ "+" | "-" ], dec, { dec } ;

DEC_INT   = dec_lit
OCT_INT   = "0", oct, { oct } ;
HEX_INT   = "0", ( "X" | "x" ), hex, { hex } ;
FLOAT     = float_lit, [ "F" | "f" ]
          | dec_lit,   ( "F" | "f" ) ;
```

10進整数は、`F` および `f` 接尾辞を使用して浮動小数点値にキャストできます。例：

```textproto
foo: 10    # これは整数値です。
foo: 10f   # これは浮動小数点値です。
foo: 1.0f  # 浮動小数点リテラルにもオプションです。
```

### 文字列リテラル {#string}

```
STRING = single_string | double_string ;
single_string = "'", { escape | char - "'" - newline - "\" }, "'" ;
double_string = '"', { escape | char - '"' - newline - "\" }, '"' ;

escape = "\a"                        (* ASCII #7  (bell)                 *)
       | "\b"                        (* ASCII #8  (backspace)            *)
       | "\f"                        (* ASCII #12 (form feed)            *)
       | "\n"                        (* ASCII #10 (line feed)            *)
       | "\r"                        (* ASCII #13 (carriage return)      *)
       | "\t"                        (* ASCII #9  (horizontal tab)       *)
       | "\v"                        (* ASCII #11 (vertical tab)         *)
       | "\?"                        (* ASCII #63 (question mark)        *)
       | "\\"                        (* ASCII #92 (backslash)            *)
       | "\'"                        (* ASCII #39 (apostrophe)           *)
       | '\"'                        (* ASCII #34 (quote)                *)
       | "\", oct, [ oct, [ oct ] ]  (* octal escaped byte value         *)
       | "\x", hex, [ hex ]          (* hexadecimal escaped byte value   *)
       | "\u", hex, hex, hex, hex    (* Unicode code point up to 0xffff  *)
       | "\U000",
         hex, hex, hex, hex, hex     (* Unicode code point up to 0xfffff *)
       | "\U0010",
         hex, hex, hex, hex ;        (* Unicode code point between 0x100000 and 0x10ffff *)
```

8進エスケープシーケンスは最大3桁の8進数を消費します。追加の桁はエスケープせずに通過します。たとえば、入力 `\1234` をエスケープ解除すると、パーサーは3桁の8進数 (123) を消費してバイト値 0x83 (ASCII 'S') をエスケープし、続く '4' はバイト値 0x34 (ASCII '4') として通過します。正しい解析を確実にするために、3桁の8進数を先頭にゼロを使用して表現し、`\000`、`\001`、`\063`、`\377` のように表現してください。数値文字の後に非数値文字が続く場合、`\5Hello` のように、3桁未満の数字が消費されます。

16進エスケープシーケンスは最大2桁の16進数を消費します。たとえば、`\x213` をエスケープ解除すると、パーサーは最初の2桁 (21) のみを消費してバイト値 0x21 (ASCII '!') をエスケープします。正しい解析を確実にするために、先頭に必要に応じてゼロを使用して、2桁の16進エスケープシーケンスを表現してください。`\x00`、`\x01`、`\xFF` のように。数値文字の後に非16進文字が続く場合、`\xFHello` や `\x3world` のように、2桁未満が消費されます。

バイト単位のエスケープは `bytes` 型のフィールドにのみ使用してください。`string` 型のフィールドでもバイト単位のエスケープを使用することは可能ですが、これらのエスケープシーケンスは有効な UTF-8 シーケンスを形成する必要があります。バイト単位のエスケープを使用して UTF-8 シーケンスを表現することは誤りを生じやすいです。`string` 型のフィールドのリテラルで印刷できない文字や改行文字を表現する場合は、UTF-8 シーケンスを優先し、`string` 型のフィールドのリテラルで使用してください。```

長い文字列は、連続する行で複数の引用符付き文字列に分割できます。
例：

```proto
  quote:
      "When we got into office, the thing that surprised me most was to find "
      "that things were just as bad as we'd been saying they were.\n\n"
      "  -- John F. Kennedy"
```

Unicodeコードポイントは、[Unicode 13 Table A-1 Extended BNF](https://www.unicode.org/versions/Unicode13.0.0/appA.pdf#page=5)に従って解釈され、UTF-8でエンコードされます。

{{% alert title="警告" color="warning" %}} C++の実装は現在、エスケープされた高サロゲートコードポイントをUTF-16コードユニットとして解釈し、`\uHHHH`の低サロゲートコードポイントが直ちに続くことを期待しています。別々の引用符付き文字列をまたがる分割はありません。さらに、対にならないサロゲートは、無効なUTF-8に直接レンダリングされます。これらはどちらも非準拠の動作です[^surrogates]。{{% /alert %}}

[^surrogates]: Unicode 13を参照してください
    [§3.8 サロゲート](https://www.unicode.org/versions/Unicode13.0.0/ch03.pdf#page=48)、
    [§3.2 準拠要件、C1](https://www.unicode.org/versions/Unicode13.0.0/ch03.pdf#page=8)、および
    [§3.9 Unicodeエンコーディング形式、D92](https://www.unicode.org/versions/Unicode13.0.0/ch03.pdf#page=54)。

## 構文要素 {#syntax}

### メッセージ {#message}

メッセージはフィールドのコレクションです。テキスト形式のファイルは単一のメッセージです。

```
Message = { Field } ;
```

### リテラル {#literals}

フィールドのリテラル値は、数値、文字列、または `true` や enum 値などの識別子であることができます。

```
String             = STRING, { STRING } ;
Float              = [ "-" ], FLOAT ;
Identifier         = IDENT ;
SignedIdentifier   = "-", IDENT ;   (* For example, "-inf" *)
DecSignedInteger   = "-", DEC_INT ;
OctSignedInteger   = "-", OCT_INT ;
HexSignedInteger   = "-", HEX_INT ;
DecUnsignedInteger = DEC_INT ;
OctUnsignedInteger = OCT_INT ;
HexUnsignedInteger = HEX_INT ;
```

単一の文字列値は、オプションの空白で区切られた複数の引用部分で構成されることがあります。例：

```textproto
a_string: "first part" 'second part'
          "third part"
no_whitespace: "first""second"'third''fourth'
```

### フィールド名 {#field-names}

含まれるメッセージの一部であるフィールドは、名前として単純な `Identifiers` を使用します。
[`Extension`](/programming-guides/proto2#extensions) および
[`Any`](/programming-guides/proto3#any) フィールド名は、角かっこで囲まれ、完全修飾されます。`Any` フィールド名は、`type.googleapis.com/`などの修飾ドメイン名で接頭辞が付けられます。

```
FieldName     = ExtensionName | AnyName | IDENT ;
ExtensionName = "[", TypeName, "]" ;
AnyName       = "[", Domain, "/", TypeName, "]" ;
TypeName      = IDENT, { ".", IDENT } ;
Domain        = IDENT, { ".", IDENT } ;
```

通常のフィールドと拡張フィールドには、スカラー値またはメッセージ値が含まれることがあります。`Any` フィールドは常にメッセージです。例：

```textproto
reg_scalar: 10
reg_message { foo: "bar" }

[com.foo.ext.scalar]​: 10
[com.foo.ext.message] { foo: "bar" }

any_value {
  [type.googleapis.com/com.foo.any] { foo: "bar" }
}
```

#### 未知のフィールド {#unknown-fields}

テキスト形式のパーサーは、フィールド名の代わりに生のフィールド番号で表される未知のフィールドをサポートできません。なぜなら、6つのワイヤータイプのうち3つがテキスト形式で同じ方法で表されるからです。一部のテキスト形式のシリアライザの実装は、未知のフィールドを、フィールド番号と値の数値表現を使用する形式でエンコードしますが、これはワイヤータイプ情報が無視されるため、本質的に損失が発生します。比較のため、ワイヤーフォーマットはワイヤータイプを`(field_number << 3) | wire_type`として各フィールドタグに含むため、非損失です。エンコードに関する詳細は、[エンコーディング](/programming-guides/encoding.md)トピックを参照してください。

メッセージスキーマからフィールドタイプに関する情報がない場合、値は正しくワイヤーフォーマットの proto メッセージにエンコードできません。

### フィールド {#fields}

フィールドの値はリテラル（文字列、数値、または識別子）またはネストされたメッセージであることができます。

```
Field        = ScalarField | MessageField ;
MessageField = FieldName, [ ":" ], ( MessageValue | MessageList ) [ ";" | "," ];
ScalarField  = FieldName, ":",     ( ScalarValue  | ScalarList  ) [ ";" | "," ];
MessageList  = "[", [ MessageValue, { ",", MessageValue } ], "]" ;
ScalarList   = "[", [ ScalarValue,  { ",", ScalarValue  } ], "]" ;
MessageValue = "{", Message, "}" | "<", Message, ">" ;
ScalarValue  = String
             | Float
             | Identifier
             | SignedIdentifier
             | DecSignedInteger
             | OctSignedInteger
             | HexSignedInteger
             | DecUnsignedInteger
             | OctUnsignedInteger
             | HexUnsignedInteger ;
```

スカラーフィールドの場合、フィールド名と値の間の`:`区切り記号が必要ですが、メッセージフィールド（リストを含む）の場合はオプションです。例：

```textproto
scalar: 10          # Valid
scalar  10          # Invalid
scalars: [1, 2, 3]  # Valid
scalars  [1, 2, 3]  # Invalid
message: {}         # Valid
message  {}         # Valid
messages: [{}, {}]  # Valid
messages  [{}, {}]  # Valid
```

メッセージフィールドの値は波括弧または角括弧で囲むことができます：

```textproto
message: { foo: "bar" }
message: < foo: "bar" >
```

`repeated` とマークされたフィールドは、フィールドを繰り返すことで複数の値を指定できます。特別な`[]`リスト構文を使用するか、その両方の組み合わせを使用できます。値の順序は維持されます。例：

```textproto
repeated_field: 1
repeated_field: 2
repeated_field: [3, 4, 5]
repeated_field: 6
repeated_field: [7, 8, 9]
```

`repeated` でないフィールドはリスト構文を使用できません。たとえば、`optional` または `required` フィールドに対して `[0]` は有効ではありません。`optional` とマークされたフィールドは省略するか、1回指定できます。`required` とマークされたフィールドは正確に1回指定する必要があります。

関連する *.proto* メッセージで指定されていないフィールドは、メッセージの `reserved` フィールドリストにフィールド名が存在しない限り許可されません。`reserved` フィールドは、任意の形式（スカラー、リスト、メッセージ）で存在する場合は、テキスト形式で単に無視されます。

## 値の種類 {#value}

フィールドの関連する *.proto* 値の種類がわかっている場合、以下の値の説明と制約が適用されます。このセクションの目的のために、以下のコンテナ要素を宣言します。

```
signedInteger   = DecSignedInteger | OctSignedInteger | HexSignedInteger ;
unsignedInteger = DecUnsignedInteger | OctUnsignedInteger | HexUnsignedInteger ;
integer         = signedInteger | unsignedInteger ;
```

<table>
  <thead>
    <tr>
      <th style="text-align:center"><strong>.proto タイプ</strong></th>
      <th style="text-align:center"><strong>値</strong></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:center">
        <code>float</code>, <code>double</code>
      </td>
      <td style="text-align:left">
        <code>Float</code>、<code>DecSignedInteger</code>、または
        <code>DecUnsignedInteger</code> 要素、または <code>Identifier</code>
        または <code>SignedIdentifier</code> 要素で、<code>IDENT</code>
        部分が <em>"inf"</em>、<em>"infinity"</em>、または
        <em>"nan"</em>（大文字小文字を区別しない）と等しいもの。オーバーフローは無限大または
        -無限大として扱われます。8進数および16進数の値は有効ではありません。
        <p>
        注: <em>"nan"</em> は <a href="https://en.wikipedia.org/wiki/NaN#Quiet_NaN">Quiet NaN</a> として解釈されるべきです。
        </p>
      </td>
    </tr>
    <tr>
      <td style="text-align:center">
        <code>int32</code>, <code>sint32</code>, <code>sfixed32</code>
      </td>
      <td style="text-align:left">
        範囲内のいずれかの <code>integer</code> 要素：<em>-0x80000000</em> から <em>0x7FFFFFFF</em>。
      </td>
    </tr>
    <tr>
      <td style="text-align:center">
        <code>int64</code>, <code>sint64</code>, <code>sfixed64</code>
      </td>
      <td style="text-align:left">
        範囲内のいずれかの <code>integer</code> 要素：<em>-0x8000000000000000</em> から <em>0x7FFFFFFFFFFFFFFF</em>。
      </td>
    </tr>
    <tr>
      <td style="text-align:center">
        <code>uint32</code>, <code>fixed32</code>
      </td>
      <td style="text-align:left">
        範囲内のいずれかの <code>unsignedInteger</code> 要素：<em>0</em> から <em>0xFFFFFFFF</em>。負の値
        （<em>-0</em>）は有効ではありません。
      </td>
    </tr>
    <tr>
      <td style="text-align:center">
        <code>uint64</code>, <code>fixed64</code>
      </td>
      <td style="text-align:left">
        範囲内のいずれかの <code>unsignedInteger</code> 要素：<em>0</em> から <em>0xFFFFFFFFFFFFFFFF</em>。負の値
        （<em>-0</em>）は有効ではありません。
      </td>
    </tr>
    <tr>
      <td style="text-align:center">
        <code>string</code>
      </td>
      <td style="text-align:left">
        有効な UTF-8 データを含む <code>String</code> 要素。エスケープシーケンスは、アンエスケープ時に有効な
        UTF-8 バイトシーケンスを形成しなければなりません。
      </td>
    </tr>
    <tr>
      <td style="text-align:center">
        <code>bytes</code>
      </td>
      <td style="text-align:left">
        無効な UTF-8 エスケープシーケンスを含む可能性のある <code>String</code> 要素。
      </td>
    </tr>
    <tr>
      <td style="text-align:center">
        <code>bool</code>
      </td>
      <td style="text-align:left">
        <code>Identifier</code> 要素または次のいずれかの <code>unsignedInteger</code> 要素のいずれかに一致する
        値。<br/>
        <strong>True 値:</strong> <em>"True"</em>、<em>"true"</em>、
        <em>"t"</em>、<em>1</em><br/>
        <strong>False 値:</strong> <em>"False"</em>、<em>"false"</em>、
        <em>"f"</em>、<em>0</em><br/>
        <em>0</em> または <em>1</em> のいずれかの符号なし整数表現が許可されます：<em>00</em>、<em>0x0</em>、
        <em>01</em>、<em>0x1</em> など。
      </td>
    </tr>
    <tr>
      <td style="text-align:center">
        <em>enum 値</em>
      </td>
      <td style="text-align:left">
        列挙値名を含む <code>Identifier</code> 要素、または範囲内のいずれかの <code>integer</code> 要素：
        <em>-0x80000000</em> から <em>0x7FFFFFFF</em> で列挙値番号を含むもの。フィールドの <code>enum</code> タイプ定義の
        メンバーでない名前を指定することは無効です。特定の protobuf ランタイム実装によっては、
        フィールドの <code>enum</code> タイプ定義のメンバーでない数値を指定することが有効であるかどうかが異なる場合があります。
        特定のランタイム実装に結びついていないテキスト形式のプロセッサ（IDE サポートなど）は、提供された数値値が有効なメンバーでない場合に警告を発行することがあります。
        他のコンテキストで有効なキーワードである特定の名前、例えば <em>"true"</em> や <em>"infinity"</em> など、も有効な列挙値名です。
      </td>
    </tr>
    <tr>
      <td style="text-align:center">
        <em>メッセージ値</em>
      </td>
      <td style="text-align:left">
        <code>MessageValue</code> 要素。
      </td>
    </tr>
  </tbody>
</table>

## 拡張フィールド {#extension}

拡張フィールドは、修飾名を使用して指定されます。例：

```textproto
local_field: 10
[com.example.ext_field]​: 20
```

拡張フィールドは一般的に他の *.proto* ファイルで定義されます。テキスト形式の言語には、拡張フィールドを定義するファイルの場所を指定するメカニズムが提供されていません。代わりに、パーサーはそれらの場所について事前に知識を持っている必要があります。

## `Any` フィールド {#any}

テキスト形式は、拡張フィールドに似た特別な構文を使用して、[`google.protobuf.Any`](/programming-guides/proto3#any) の拡張形式をサポートしています。例：

```textproto
local_field: 10

# An Any value using regular fields.
any_value {
  type_url: "type.googleapis.com/com.example.SomeType"
  value: "\x0a\x05hello"  # serialized bytes of com.example.SomeType
}

# The same value using Any expansion
any_value {
  [type.googleapis.com/com.example.SomeType] {
    field1: "hello"
  }
}
```

この例では、`any_value` は `google.protobuf.Any` 型のフィールドであり、`field1: hello` を含む `com.example.SomeType` メッセージをシリアル化して格納しています。

## `group` フィールド {#group}

テキスト形式では、`group` フィールドは通常の `MessageValue` 要素をその値として使用しますが、暗黙的な小文字のフィールド名ではなく、大文字のグループ名を使用して指定されます。例：

```proto
message MessageWithGroup {
  optional group MyGroup = 1 {
    optional int32 my_value = 1;
  }
}
```

上記の *.proto* 定義を使用すると、次のテキスト形式は有効な `MessageWithGroup` です：

```textproto
MyGroup {
  my_value: 1
}
```

メッセージフィールドと同様に、グループ名と値の間の `:` デリミタはオプションです。

## `map` フィールド {#map}

テキスト形式では、マップフィールドエントリを指定するためのカスタム構文は提供されていません。[`map`](/programming-guides/proto2#maps) フィールドが *.proto* ファイルで定義されると、`key` と `value` フィールドを含む暗黙的な `Entry` メッセージが定義されます。マップフィールドは常に繰り返され、複数のキー/値エントリを受け入れます。例：

```proto
message MessageWithMap {
  map<string, int32> my_map = 1;
}
```

上記の *.proto* 定義を使用すると、次のテキスト形式は有効な `MessageWithMap` です：

```textproto
my_map { key: "entry1" value: 1 }
my_map { key: "entry2" value: 2 }

# You can also use the list syntax
my_map: [
  { key: "entry3" value: 3 },
  { key: "entry4" value: 4 }
]
```

`key` フィールドと `value` フィールドの両方はオプションであり、指定されていない場合はそれぞれの型のゼロ値にデフォルトで設定されます。キーが重複している場合、解析されたマップには最後に指定された値のみが保持されます。

マップの順序は、textprotosでは維持されません。

## `oneof` フィールド {#oneof}

`oneof` フィールドに関連する特別な構文はありませんが、テキスト形式では一度に1つの `oneof` メンバーのみを指定できます。複数のメンバーを同時に指定することはできません。例:

```proto
message OneofExample {
  message MessageWithOneof {
    optional string not_part_of_oneof = 1;
    oneof Example {
      string first_oneof_field = 2;
      string second_oneof_field = 3;
    }
  }
  repeated MessageWithOneof message = 1;
}
```

上記の *.proto* 定義は、次のテキスト形式の動作を示します:

```textproto
# Valid: only one field from the Example oneof is set.
message {
  not_part_of_oneof: "always valid"
  first_oneof_field: "valid by itself"
}

# Valid: the other oneof field is set.
message {
  not_part_of_oneof: "always valid"
  second_oneof_field: "valid by itself"
}

# Invalid: multiple fields from the Example oneof are set.
message {
  not_part_of_oneof: "always valid"
  first_oneof_field: "not valid"
  second_oneof_field: "not valid"
}
```

## テキスト形式ファイル {#text-format-files}

テキスト形式ファイルは `.txtpb` 拡張子を使用し、単一の `Message` を含みます。テキスト形式ファイルは UTF-8 エンコードされています。以下にテキストプロトファイルの例を示します。

{{% alert title="重要" color="warning" %}}
`.txtpb` は標準的なテキスト形式ファイルの拡張子であり、他の選択肢よりも好ましいです。この拡張子は簡潔さと公式のワイヤーフォーマットファイル拡張子 `.binpb` との一貫性のために推奨されています。古い標準の拡張子 `.textproto` は広く使用されており、ツールサポートもあります。一部のツールは古い拡張子 `.textpb` や `.pbtxt` もサポートしています。上記以外のすべての拡張子は **強く** 推奨されません。特に、`.protoascii` のような拡張子は、テキスト形式が ASCII のみであることを誤って示し、`.pb.txt` のような他の拡張子は一般的なツールで認識されません。
{{% /alert %}}

```textproto
# This is an example of Protocol Buffer's text format.
# Unlike .proto files, only shell-style line comments are supported.

name: "John Smith"

pet {
  kind: DOG
  name: "Fluffy"
  tail_wagginess: 0.65f
}

pet <
  kind: LIZARD
  name: "Lizzy"
  legs: 4
>

string_value_with_escape: "valid \n escape"
repeated_values: [ "one", "two", "three" ]
```

### ヘッダー {#header}

ヘッダーコメント `proto-file` と `proto-message` は、スキーマに関する情報を開発者ツールに提供し、さまざまな機能を提供します。

```textproto
# proto-file: some/proto/my_file.proto
# proto-message: MyMessage
```

## プログラムでのフォーマット操作

個々の Protocol Buffer 実装が一貫した標準的なテキスト形式を出力しないため、TextProto ファイルを変更したり TextProto 出力を生成するツールやライブラリは、出力をフォーマットするために明示的に
https://github.com/protocolbuffers/txtpbfmt
を使用する必要があります。
