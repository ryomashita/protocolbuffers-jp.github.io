+++
title = "Protocol Buffers バージョン 3 言語仕様"
weight = 810
linkTitle = "バージョン 3 言語仕様"
description = "Protocol Buffers 言語（proto3）のバージョン 3 の言語仕様リファレンス。"
type = "docs"
+++

構文は、[拡張バッカス・ナウア形式（EBNF）](https://en.wikipedia.org/wiki/Extended_Backus%E2%80%93Naur_Form) を使用して指定されています：

```
|   alternation
()  grouping
[]  option (zero or one time)
{}  repetition (any number of times)
```

proto3 の使用に関する詳細は、[言語ガイド](/programming-guides/proto3) を参照してください。

## 字句要素 {#lexical_elements}

### 文字と数字 {#letters_and_digits}

```
letter = "A" ... "Z" | "a" ... "z"
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
messageType = [ "." ] { ident "." } messageName
enumType = [ "." ] { ident "." } enumName
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
strLitSingle = ( "'" { charValue } "'" ) |  ( '"' { charValue } '"' )
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

## 構文

構文文は、protobuf バージョンを定義するために使用されます。

```
syntax = "syntax" "=" ("'" "proto3" "'" | '"' "proto3" '"') ";"
```

例：

```proto
syntax = "proto3";
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

## オプション {#examples}

オプションは、protoファイル、メッセージ、列挙型、およびサービスで使用できます。オプションは、protobufで定義されたオプションまたはカスタムオプションであることができます。詳細については、[Options](/programming-guides/proto3#options)を参照してください。

```
option = "option" optionName  "=" constant ";"
optionName = ( ident | bracedFullIdent ) { "." ( ident | bracedFullIdent ) }
bracedFullIdent = "(" ["."] fullIdent ")"
optionNamePart = { ident | "(" ["."] fullIdent ")" }
```

例:

```proto
option java_package = "com.example.foo";
```

## フィールド {#examples}

フィールドは、プロトコルバッファメッセージの基本要素です。フィールドには通常のフィールド、oneofフィールド、またはmapフィールドがあります。フィールドには型とフィールド番号があります。

```
type = "double" | "float" | "int32" | "int64" | "uint32" | "uint64"
      | "sint32" | "sint64" | "fixed32" | "fixed64" | "sfixed32" | "sfixed64"
      | "bool" | "string" | "bytes" | messageType | enumType
fieldNumber = intLit;
```

### 通常のフィールド {#normal_field}

各フィールドには、型、名前、およびフィールド番号があります。フィールドオプションを持つことができます。

```
field = [ "repeated" ] type fieldName "=" fieldNumber [ "[" fieldOptions "]" ] ";"
fieldOptions = fieldOption { ","  fieldOption }
fieldOption = optionName "=" constant
```

例:

```proto
foo.Bar nested_message = 2;
repeated int32 samples = 4 [packed=true];
```

### OneofおよびOneofフィールド {#oneof_and_oneof_field}

oneofは、oneofフィールドとoneof名から構成されます。

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

### Mapフィールド {#map_field}

マップフィールドには、キーの型、値の型、名前、およびフィールド番号があります。キーの型は、整数型または文字列型のいずれかであることができます。

```
mapField = "map" "<" keyType "," type ">" mapName "=" fieldNumber [ "[" fieldOptions "]" ] ";"
keyType = "int32" | "int64" | "uint32" | "uint64" | "sint32" | "sint64" |
          "fixed32" | "fixed64" | "sfixed32" | "sfixed64" | "bool" | "string"
```

例:

```proto
map<string, Project> projects = 3;
```

## 予約 {#examples}

予約ステートメントは、このメッセージで使用できないフィールド番号またはフィールド名の範囲を宣言します。

```
reserved = "reserved" ( ranges | strFieldNames ) ";"
ranges = range { "," range }
range =  intLit [ "to" ( intLit | "max" ) ]
strFieldNames = strFieldName { "," strFieldName }
strFieldName = "'" fieldName "'" | '"' fieldName '"'
```

例:

```proto
reserved 2, 15, 9 to 11;
reserved "foo", "bar";
```

## トップレベルの定義 {#top_level_definitions}

### 列挙型定義 {#enum_definition}

列挙型定義には、名前と列挙体が含まれます。列挙体には、オプション、列挙フィールド、および予約ステートメントが含まれることがあります。

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

### メッセージ定義 {#message_definition}

メッセージはメッセージ名とメッセージ本体で構成されます。メッセージ本体には、フィールド、ネストされた列挙型の定義、ネストされたメッセージの定義、オプション、oneofs、マップフィールド、および予約ステートメントが含まれることができます。メッセージスキーマ内の同じ名前を持つ2つのフィールドを含めることはできません。

```
message = "message" messageName messageBody
messageBody = "{" { field | enum | message | option | oneof | mapField |
reserved | emptyStatement } "}"
```

例:

```proto
message Outer {
  option (my_option).a = true;
  message Inner {   // Level 2
    int64 ival = 1;
  }
  map<int32, string> my_map = 2;
}
```

メッセージ内で宣言されたエンティティのいずれもが競合する名前を持つことはできません。以下のすべてが禁止されています:

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
  enum E {
    foo = 0;
  }
}
```

### サービス定義 {#service_definition}

```
service = "service" serviceName "{" { option | rpc | emptyStatement } "}"
rpc = "rpc" rpcName "(" [ "stream" ] messageType ")" "returns" "(" [ "stream" ]
messageType ")" (( "{" {option | emptyStatement } "}" ) | ";")
```

例:

```proto
service SearchService {
  rpc Search (SearchRequest) returns (SearchResponse);
}
```

## Protoファイル {#proto_file}

```
proto = [syntax] { import | package | option | topLevelDef | emptyStatement }
topLevelDef = message | enum | service
```

例えば.protoファイル:

```proto
syntax = "proto3";
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
    int64 ival = 1;
  }
  repeated Inner inner_message = 2;
  EnumAllowingAlias enum_field = 3;
  map<int32, string> my_map = 4;
}
```
