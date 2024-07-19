+++
title = "プロトコルバッファバージョン2言語仕様"
weight = 800
linkTitle = "バージョン2言語仕様"
description = "プロトコルバッファ言語（proto2）のバージョン2の言語仕様リファレンス。"
type = "docs"
+++

構文は、[拡張バッカス・ナウア形式（EBNF）](https://en.wikipedia.org/wiki/Extended_Backus%E2%80%93Naur_Form)を使用して指定されています：

```
|   alternation
()  grouping
[]  option (zero or one time)
{}  repetition (any number of times)
```

proto2の使用に関する詳細は、[言語ガイド](/programming-guides/proto2)を参照してください。

## 字句要素 {#lexical_elements}

### 文字と数字 {#letters_and_digits}

```
letter = "A" ... "Z" | "a" ... "z"
capitalLetter =  "A" ... "Z"
decimalDigit = "0" ... "9"
octalDigit   = "0" ... "7"
hexDigit     = "0" ... "9" | "A" ... "F" | "a" ... "f"
```

### 識別子

```
ident = letter { letter | decimalDigit | "_" }
fullIdent = ident { "." ident }
messageName = ident
enumName = ident
fieldName = ident
oneofName = ident
mapName = ident
serviceName = ident
rpcName = ident
streamName = ident
messageType = [ "." ] { ident "." } messageName
enumType = [ "." ] { ident "." } enumName
groupName = capitalLetter { letter | decimalDigit | "_" }
```

### 整数リテラル {#integer_literals}

```
intLit     = decimalLit | octalLit | hexLit
decimalLit = [-] ( "1" ... "9" ) { decimalDigit }
octalLit   = [-] "0" { octalDigit }
hexLit     = [-] "0" ( "x" | "X" ) hexDigit { hexDigit }
```

### 浮動小数点リテラル

```
floatLit = [-] ( decimals "." [ decimals ] [ exponent ] | decimals exponent | "."decimals [ exponent ] ) | "inf" | "nan"
decimals  = [-] decimalDigit { decimalDigit }
exponent  = ( "e" | "E" ) [ "+" | "-" ] decimals
```

### ブール値

```
boolLit = "true" | "false"
```

### 文字列リテラル {#string_literals}

```
strLit = strLitSingle { strLitSingle }
strLitSingle = ( "'" { charValue } "'" ) | ( '"' { charValue } '"' )
charValue = hexEscape | octEscape | charEscape | unicodeEscape | unicodeLongEscape | /[^\0\n\\]/
hexEscape = '\' ( "x" | "X" ) hexDigit [ hexDigit ]
octEscape = '\' octalDigit [ octalDigit [ octalDigit ] ]
charEscape = '\' ( "a" | "b" | "f" | "n" | "r" | "t" | "v" | '\' | "'" | '"' )
unicodeEscape = '\' "u" hexDigit hexDigit hexDigit hexDigit
unicodeLongEscape = '\' "U" ( "000" hexDigit hexDigit hexDigit hexDigit hexDigit |
                              "0010" hexDigit hexDigit hexDigit hexDigit
```

### 空文

```
emptyStatement = ";"
```

### 定数

```
constant = fullIdent | ( [ "-" | "+" ] intLit ) | ( [ "-" | "+" ] floatLit ) |
                strLit | boolLit | MessageValue
```

`MessageValue` は、[テキスト形式言語仕様](/reference/protobuf/textformat-spec#fields)で定義されています。

## 構文

構文文は、protobufバージョンを定義するために使用されます。

```
syntax = "syntax" "=" ("'" "proto2" "'" | '"' "proto2" '"') ";"
```

## インポート文 {#import_statement}

インポート文は、別の .proto の定義をインポートするために使用されます。

```
import = "import" [ "weak" | "public" ] strLit ";"
```

例：

```proto
import public "other.proto";
```

## パッケージ

パッケージ指定子は、プロトコルメッセージタイプ間の名前の衝突を防ぐために使用できます。

```
package = "package" fullIdent ";"
```

例：

```proto
package foo.bar;
```

## オプション

オプションは、protoファイル、メッセージ、列挙型、サービスで使用できます。オプションは、protobufで定義されたオプションまたはカスタムオプションであることができます。詳細については、[オプション](/programming-guides/proto2#options)を言語ガイドで参照してください。

```
option = "option" optionName "=" constant ";"
optionName = ( ident | bracedFullIdent ) { "." ( ident | bracedFullIdent ) }
bracedFullIdent = "(" ["."] fullIdent ")"

For examples:

```proto
option java_package = "com.example.foo";
```

## フィールド

フィールドはプロトコルバッファメッセージの基本要素です。フィールドには通常のフィールド、グループフィールド、oneofフィールド、またはマップフィールドがあります。フィールドにはラベル、タイプ、およびフィールド番号があります。

```
label = "required" | "optional" | "repeated"
type = "double" | "float" | "int32" | "int64" | "uint32" | "uint64"
      | "sint32" | "sint64" | "fixed32" | "fixed64" | "sfixed32" | "sfixed64"
      | "bool" | "string" | "bytes" | messageType | enumType
fieldNumber = intLit;
```

### 通常のフィールド {#normal_field}

各フィールドにはラベル、タイプ、名前、およびフィールド番号があります。フィールドオプションを持つことができます。

```
field = label type fieldName "=" fieldNumber "[" fieldOptions "]" ";"
fieldOptions = fieldOption { "," fieldOption }
fieldOption = optionName "=" constant
```

例:

```proto
optional foo.bar nested_message = 2;
repeated int32 samples = 4 [packed=true];
```

### グループフィールド {#group_field}

**この機能は非推奨であり、新しいメッセージタイプを作成する際には使用しないでください -- 代わりにネストされたメッセージタイプを使用してください。**

グループはメッセージ定義内で情報をネストする方法の1つです。グループ名は大文字で始まる必要があります。

```
group = label "group" groupName "=" fieldNumber messageBody
```

例:

```proto
repeated group Result = 1 {
    required string url = 1;
    optional string title = 2;
    repeated string snippets = 3;
}
```

### Oneofおよびoneofフィールド {#oneof_and_oneof_field}

oneofはoneofフィールドとoneof名から構成されます。oneofフィールドにはラベルがありません。

```
oneof = "oneof" oneofName "{" { option | oneofField } "}"
oneofField = type fieldName "=" fieldNumber "[" fieldOptions "]" ";"
```

例:

```proto
oneof foo {
    string name = 4;
    SubMessage sub_message = 9;
}
```

### マップフィールド {#map_field}

マップフィールドにはキータイプ、値タイプ、名前、およびフィールド番号があります。キータイプは整数型または文字列型である必要があります。キータイプは列挙型であってはいけません。

```
mapField = "map" "<" keyType "," type ">" mapName "=" fieldNumber "[" fieldOptions "]" ";"
keyType = "int32" | "int64" | "uint32" | "uint64" | "sint32" | "sint64" |
          "fixed32" | "fixed64" | "sfixed32" | "sfixed64" | "bool" | "string"
```

例:

```proto
map<string, Project> projects = 3;
```

## 拡張と予約 {#extensions_and_reserved}

拡張と予約は、フィールド番号またはフィールド名の範囲を宣言するメッセージ要素です。

### 拡張

拡張は、メッセージ内のフィールド番号の範囲がサードパーティの拡張用に利用可能であることを宣言します。他の人は、自分自身の .proto ファイルでそれらの数値タグを使用して、メッセージタイプに新しいフィールドを宣言できます。元のファイルを編集する必要はありません。

```
extensions = "extensions" ranges ";"
ranges = range { "," range }
range =  intLit [ "to" ( intLit | "max" ) ]
```

例:

```proto
extensions 100 to 199;
extensions 4, 20 to max;
```

### 予約

予約は、メッセージ内のフィールド番号またはフィールド名の範囲を宣言し、使用できないことを示します。

```
reserved = "reserved" ( ranges | strFieldNames ) ";"
strFieldNames = strFieldName { "," strFieldName }
strFieldName = "'" fieldName "'" | '"' fieldName '"'
```

例:

```proto
reserved 2, 15, 9 to 11;
reserved "foo", "bar";
```

## トップレベルの定義 {#top_level_definitions}

### Enum 定義 {#enum_definition}

Enum 定義は名前と Enum 本体で構成されます。Enum 本体には、オプション、Enum フィールド、および予約ステートメントが含まれることがあります。

```
enum = "enum" enumName enumBody
enumBody = "{" { option | enumField | emptyStatement | reserved } "}"
enumField = ident "=" [ "-" ] intLit [ "[" enumValueOption { ","  enumValueOption } "]" ]";"
enumValueOption = optionName "=" constant
```

例:

```proto
enum EnumAllowingAlias {
  option allow_alias = true;
  EAA_UNSPECIFIED = 0;
  EAA_STARTED = 1;
  EAA_RUNNING = 2 [(custom_option) = "hello world"];
}
```

### Message 定義 {#message_definition}

メッセージはメッセージ名とメッセージ本体で構成されます。メッセージ本体には、フィールド、ネストされた Enum 定義、ネストされたメッセージ定義、拡張ステートメント、拡張、グループ、オプション、OneOf、Map フィールド、および予約ステートメントが含まれることがあります。メッセージスキーマ内の同じメッセージで同じ名前のフィールドを2つ含めることはできません。

```
message = "message" messageName messageBody
messageBody = "{" { field | enum | message | extend | extensions | group |
option | oneof | mapField | reserved | emptyStatement } "}"
```

例:

```proto
message Outer {
  option (my_option).a = true;
  message Inner {   // Level 2
    required int64 ival = 1;
  }
  map<int32, string> my_map = 2;
  extensions 20 to 30;
}
```

メッセージ内で宣言されたエンティティのいずれもが競合する名前を持ってはいけません。以下のすべてが禁止されています:

```
message MyMessage {
  optional string foo = 1;
  message foo {}
}

message MyMessage {
  optional string foo = 1;
  oneof foo {
    string bar = 2;
  }
}

message MyMessage {
  optional string foo = 1;
  extend Extendable {
    optional string foo = 2;
  }
}

message MyMessage {
  optional string foo = 1;
  enum E {
    foo = 0;
  }
}
```

### 拡張

同じまたはインポートされた .proto ファイル内のメッセージが拡張用の範囲を予約している場合、そのメッセージを拡張することができます。

```
extend = "拡張" messageType "{" {field | group} "}"
```

Example:

```proto
extend Foo {
  optional int32 bar = 126;
}
```

### サービス定義 {#service_definition}

```
service = "service" serviceName "{" { option | rpc | emptyStatement } "}"
rpc = "rpc" rpcName "(" [ "stream" ] messageType ")" "returns" "(" [ "stream" ]
messageType ")" (( "{" { option | emptyStatement } "}" ) | ";" )
```

Example:

```proto
service SearchService {
  rpc Search (SearchRequest) returns (SearchResponse);
}
```

## Proto ファイル {#proto_file}

```
proto = [syntax] { import | package | option | topLevelDef | emptyStatement }
topLevelDef = message | enum | extend | service
```

An example .proto file:

```proto
syntax = "proto2";
import public "other.proto";
option java_package = "com.example.foo";
enum EnumAllowingAlias {
  option allow_alias = true;
  EAA_UNSPECIFIED = 0;
  EAA_STARTED = 1;
  EAA_RUNNING = 1;
  EAA_FINISHED = 2 [(custom_option) = "hello world"];
}
message Outer {
  option (my_option).a = true;
  message Inner {   // Level 2
    required int64 ival = 1;
  }
  repeated Inner inner_message = 2;
  optional EnumAllowingAlias enum_field = 3;
  map<int32, string> my_map = 4;
  extensions 20 to 30;
}
message Foo {
  optional group GroupMessage = 1 {
    optional bool a = 1;
  }
}
```
