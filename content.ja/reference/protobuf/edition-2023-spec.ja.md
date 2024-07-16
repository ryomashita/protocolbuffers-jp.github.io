+++
title = "Protocol Buffers Edition 2023 言語仕様"
weight = 800
linkTitle = "2023 言語仕様"
description = "Protocol Buffers 言語の 2023 年版の言語仕様リファレンス。"
type = "docs"
+++

構文は、[拡張バッカス・ナウア形式 (EBNF)](https://en.wikipedia.org/wiki/Extended_Backus%E2%80%93Naur_Form) を使用して指定されています：

```
|   alternation
()  grouping
[]  option (zero or one time)
{}  repetition (any number of times)
```

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

`MessageValue` は、[テキスト形式言語仕様](/reference/protobuf/textformat-spec#fields) で定義されています。

## 版

`edition` 文は、従来の `syntax` キーワードを置き換え、このファイルが使用している版を定義するために使用されます。

```
edition = "edition" "=" ("'" decimalLit '"') ";"
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

オプションは、proto ファイル、メッセージ、列挙型、およびサービスで使用できます。オプションは、protobuf で定義されたオプションまたはカスタムオプションであることができます。詳細については、[Options](/programming-guides/proto2#options) を参照してください。オプションは、[Feature Settings](/editions/features) を制御するためにも使用されます。

```
option = "option" optionName  "=" constant ";"
optionName = ( ident | "(" ["."] fullIdent ")" )
```

For examples:

```proto
option java_package = "com.example.foo";
option features.enum_type = CLOSED;
```

## フィールド

フィールドはプロトコルバッファメッセージの基本要素です。フィールドには通常のフィールド、グループフィールド、oneofフィールド、またはマップフィールドがあります。フィールドにはラベル、タイプ、フィールド番号があります。

```
label = "required" | "optional" | "repeated"
type = "double" | "float" | "int32" | "int64" | "uint32" | "uint64"
      | "sint32" | "sint64" | "fixed32" | "fixed64" | "sfixed32" | "sfixed64"
      | "bool" | "string" | "bytes" | messageType | enumType
fieldNumber = intLit;
```

### 通常のフィールド {#normal_field}

各フィールドにはラベル、タイプ、名前、フィールド番号があります。フィールドにはフィールドオプションがある場合があります。

```
field = label type fieldName "=" fieldNumber [ "[" fieldOptions "]" ] ";"
fieldOptions = fieldOption { ","  fieldOption }
fieldOption = optionName "=" constant
```

例:

```proto
optional foo.bar nested_message = 2;
repeated int32 samples = 4 [packed=true];
```

### Oneofフィールドとoneof {#oneof_and_oneof_field}

oneofはoneofフィールドとoneof名で構成されます。oneofフィールドにはラベルがありません。

```
oneof = "oneof" oneofName "{" { option | oneofField } "}"
oneofField = type fieldName "=" fieldNumber [ "[" fieldOptions "]" ] ";"
```

例:

```proto
oneof foo {
    string name = 4;
    SubMessage sub_message = 9;
}
```

### マップフィールド {#map_field}

マップフィールドにはキータイプ、値タイプ、名前、フィールド番号があります。キータイプは整数型または文字列型である必要があります。キータイプは列挙型であってはいけません。

```
mapField = "map" "<" keyType "," type ">" mapName "=" fieldNumber [ "[" fieldOptions "]" ] ";"
keyType = "int32" | "int64" | "uint32" | "uint64" | "sint32" | "sint64" |
          "fixed32" | "fixed64" | "sfixed32" | "sfixed64" | "bool" | "string"
```

例:

```proto
map<string, Project> projects = 3;
```

## 拡張と予約 {#extensions_and_reserved}

拡張と予約は、一連のフィールド番号またはフィールド名の範囲を宣言するメッセージ要素です。

### 拡張

拡張は、メッセージ内の一連のフィールド番号がサードパーティの拡張用に利用可能であることを宣言します。他の人は、元のファイルを編集せずに、自分自身の .proto ファイルでそれらの数値タグを使用してメッセージタイプの新しいフィールドを宣言できます。

```
extensions = "拡張" ranges ";"
ranges = range { "," range }
range =  intLit [ "to" ( intLit | "max" ) ]
```

Examples:

```proto
extensions 100 to 199;
extensions 4, 20 to max;
```

### 予約済み {#top_level_definitions}

Reservedは、使用できないメッセージまたは列挙型のフィールド番号または名前の範囲を宣言します。

```
reserved = "予約済み" ( ranges | reservedIdent ) ";"
fieldNames = fieldName { "," fieldName }
```

Examples:

```proto
reserved 2, 15, 9 to 11;
reserved foo, bar;
```

## トップレベル定義 {#top_level_definitions}

### 列挙型定義 {#enum_definition}

列挙型定義は名前と列挙体から構成されます。列挙体にはオプション、列挙フィールド、および予約ステートメントが含まれます。

```
enum = "enum" enumName enumBody
enumBody = "{" { option | enumField | emptyStatement | reserved } "}"
enumField = fieldName "=" [ "-" ] intLit [ "[" enumValueOption { ","  enumValueOption } "]" ]";"
enumValueOption = optionName "=" constant
```

Example:

```proto
enum EnumAllowingAlias {
  option allow_alias = true;
  EAA_UNSPECIFIED = 0;
  EAA_STARTED = 1;
  EAA_RUNNING = 2 [(custom_option) = "hello world"];
}
```

### メッセージ定義 {#message_definition}

メッセージはメッセージ名とメッセージ本体から構成されます。メッセージ本体にはフィールド、ネストされた列挙型定義、ネストされたメッセージ定義、拡張ステートメント、拡張、グループ、オプション、oneofs、マップフィールド、および予約ステートメントが含まれます。メッセージスキーマ内の同じ名前を持つ2つのフィールドを含めることはできません。

```
message = "メッセージ" messageName messageBody
messageBody = "{" { field | enum | message | extend | extensions | group |
option | oneof | mapField | reserved | emptyStatement } "}"
```

Example:

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

メッセージ内で宣言されたエンティティの名前が競合してはいけません。以下のすべてが禁止されています。

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

同じ.protoファイルまたはインポートされたファイル内のメッセージが拡張のための範囲を予約している場合、そのメッセージを拡張できます。

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
service = "サービス" serviceName "{" { option | rpc | emptyStatement } "}"
rpc = "rpc" rpcName "(" [ "stream" ] messageType ")" "returns" "(" [ "stream" ]
messageType ")" (( "{" { option | emptyStatement } "}" ) | ";" )
```

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

例としての .proto ファイル:

```proto
edition = "2023";
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
    int64 ival = 1 [features.field_presence = LEGACY_REQUIRED];
  }
  repeated Inner inner_message = 2;
  EnumAllowingAlias enum_field = 3;
  map<int32, string> my_map = 4;
  extensions 20 to 30;
  reserved reserved_field;
}
message Foo {
  message GroupMessage {
    optional bool a = 1;
  }
  GroupMessage groupmessage = [features.message_encoding = DELIMITED];
}
```
