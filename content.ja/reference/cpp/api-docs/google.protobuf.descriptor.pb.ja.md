```markdown
+++
title = "descriptor.pb.h"
toc_hide = "true"
linkTitle = "C++"
description = "このセクションには、C++でプロトコルバッファクラスを操作するためのリファレンスドキュメントが含まれています。"
type = "docs"
+++

<p><code>#include &lt;google/protobuf/descriptor.pb.h&gt;<br>namespace <a href="#google.protobuf">google::protobuf</a></code></p><p>ディスクリプタのプロトコルバッファ表現。</p><p>このファイルは、<a href="google.protobuf.descriptor">descriptor.h</a>で定義されたクラスによって表される情報と同じ情報を表すプロトコルメッセージクラスのセットを定義しています。 <a href='#FileDescriptorProto'>FileDescriptorProto</a>を<a href='google.protobuf.descriptor#FileDescriptor'>FileDescriptor</a>に変換するには、<a href="google.protobuf.descriptor#DescriptorPool">DescriptorPool</a>クラスを使用できます。 したがって、このファイルのクラスは、プロセス間でプロトコル型定義を効率的に伝達することを可能にします。</p>

<p>プロトコルコンパイラは現在、自動生成されたドキュメントをサポートしていないため、このページには説明が含まれていません。 このファイルは、<code>descriptor.proto</code>からプロトコルコンパイラによって生成されました。その内容は以下の通りです:</p>

<pre>// Protocol Buffers - Google's data interchange format
// Copyright 2008 Google Inc.  All rights reserved.
// https://developers.google.com/protocol-buffers/
//
// Redistribution and use in source and binary forms, with or without
// modification, are permitted provided that the following conditions are
// met:
//
//     * Redistributions of source code must retain the above copyright
// notice, this list of conditions and the following disclaimer.
//     * Redistributions in binary form must reproduce the above
// copyright notice, this list of conditions and the following disclaimer
// in the documentation and/or other materials provided with the
// distribution.
//     * Neither the name of Google Inc. nor the names of its
// contributors may be used to endorse or promote products derived from
// this software without specific prior written permission.
//
// THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS
// "AS IS" AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT
// LIMITED TO, THE IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR
// A PARTICULAR PURPOSE ARE DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT
// OWNER OR CONTRIBUTORS BE LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL,
// SPECIAL, EXEMPLARY, OR CONSEQUENTIAL DAMAGES (INCLUDING, BUT NOT
// LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR SERVICES; LOSS OF USE,
// DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER CAUSED AND ON ANY
// THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY, OR TORT
// (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE
// OF THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.
```

```proto
// 作者: kenton@google.com (Kenton Varda)
// オリジナルの Protocol Buffers デザインに基づいて
// Sanjay Ghemawat、Jeff Dean、およびその他の方々。
//
// このファイルのメッセージは .proto ファイルで見つかる定義を記述しています。
// 有効な .proto ファイルは、そのインポートを読み込むことなしに、
// 直接 FileDescriptorProto に変換できます。

syntax = "proto2";

package google.protobuf;

option go_package = "google.golang.org/protobuf/types/descriptorpb";
option java_package = "com.google.protobuf";
option java_outer_classname = "DescriptorProtos";
option csharp_namespace = "Google.Protobuf.Reflection";
option objc_class_prefix = "GPB";
option cc_enable_arenas = true;

// descriptor.proto は、ブートストラップ中にリフレクションベースの
// アルゴリズムが機能しないため、速度最適化されている必要があります。
option optimize_for = SPEED;

// プロトコルコンパイラは、解析した .proto ファイルを含む
// FileDescriptorSet を出力できます。
message FileDescriptorSet {
  repeated FileDescriptorProto file = 1;
}

// 完全な .proto ファイルを記述します。
message FileDescriptorProto {
  optional string name = 1;     // ソースツリーのルートからのファイル名
  optional string package = 2;  // 例: "foo", "foo.bar" など

  // このファイルでインポートされたファイルの名前。
  repeated string dependency = 3;
  // 依存リスト内のパブリックにインポートされたファイルのインデックス。
  repeated int32 public_dependency = 10;
  // 依存リスト内の弱くインポートされたファイルのインデックス。
  // Google 内部の移行専用です。使用しないでください。
  repeated int32 weak_dependency = 11;

  // このファイル内のすべてのトップレベル定義。
  repeated DescriptorProto message_type = 4;
  repeated EnumDescriptorProto enum_type = 5;
  repeated ServiceDescriptorProto service = 6;
  repeated FieldDescriptorProto extension = 7;

  optional FileOptions options = 8;

  // このフィールドには、元のソースコードに関するオプション情報が含まれます。
  // 開発ツールのみが必要とする情報であり、
  // ディスクリプタのランタイム機能に影響を与えることはありません。
  optional SourceCodeInfo source_code_info = 9;
}
```

```protobuf
// protoファイルの構文。
// サポートされる値は "proto2" と "proto3" です。
optional string syntax = 12;
}
{/*examples*/}

// メッセージタイプを記述します。
message DescriptorProto {
  optional string name = 1;

  repeated FieldDescriptorProto field = 2;
  repeated FieldDescriptorProto extension = 6;

  repeated DescriptorProto nested_type = 3;
  repeated EnumDescriptorProto enum_type = 4;

  message ExtensionRange {
    optional int32 start = 1;  // Inclusive.
    optional int32 end = 2;    // Exclusive.

optional ExtensionRangeOptions options = 3;

  }
  repeated ExtensionRange extension_range = 5;

  repeated OneofDescriptorProto oneof_decl = 8;

  optional MessageOptions options = 7;

  // 予約されたタグ番号の範囲。予約されたタグ番号は同じメッセージ内のフィールドまたは拡張範囲で使用できません。予約された範囲は重なってはいけません。
  message ReservedRange {
    optional int32 start = 1;  // Inclusive.
    optional int32 end = 2;    // Exclusive.
  }
  repeated ReservedRange reserved_range = 9;
  // 同じメッセージ内のフィールドで使用できない予約されたフィールド名。
  // 特定の名前は1回だけ予約できます。
  repeated string reserved_name = 10;
}

message ExtensionRangeOptions {
  // パーサーは認識しないオプションをここに保存します。上記を参照してください。
  repeated UninterpretedOption uninterpreted_option = 999;

  // クライアントはこのメッセージの拡張でカスタムオプションを定義できます。上記を参照してください。
  extensions 1000 to max;
}

// メッセージ内のフィールドを記述します。
message FieldDescriptorProto {
  enum Type {
    // 0 はエラー用に予約されています。
    // 歴史的な理由から順序が奇妙です。
    TYPE_DOUBLE = 1;
    TYPE_FLOAT = 2;
    // ZigZagエンコードされていません。負の数は10バイトを取ります。負の値が多い場合はTYPE_SINT64を使用してください。
    TYPE_INT64 = 3;
    TYPE_UINT64 = 4;
    // ZigZagエンコードされていません。負の数は10バイトを取ります。負の値が多い場合はTYPE_SINT32を使用してください。
    TYPE_INT32 = 5;
    TYPE_FIXED64 = 6;
    TYPE_FIXED32 = 7;
    TYPE_BOOL = 8;
    TYPE_STRING = 9;
    // タグ区切りの集合。
    // グループタイプは非推奨であり、proto3ではサポートされていません。ただし、Proto3の実装は引き続きグループワイヤーフォーマットを解析し、グループフィールドを未知のフィールドとして扱う必要があります。
    TYPE_GROUP = 10;
    TYPE_MESSAGE = 11;  // 長さ区切りの集合。
```

```markdown
// バージョン2で新規追加されたもの。
TYPE_BYTES = 12;
TYPE_UINT32 = 13;
TYPE_ENUM = 14;
TYPE_SFIXED32 = 15;
TYPE_SFIXED64 = 16;
TYPE_SINT32 = 17;  // ZigZagエンコーディングを使用します。
TYPE_SINT64 = 18;  // ZigZagエンコーディングを使用します。

  }

  enum Label {
    // 0はエラー用に予約されています
    LABEL_OPTIONAL = 1;
    LABEL_REQUIRED = 2;
    LABEL_REPEATED = 3;
  }

  optional string name = 1;
  optional int32 number = 3;
  optional Label label = 4;

  // type_nameが設定されている場合、これは設定する必要はありません。type_nameと両方設定されている場合、これはTYPE_ENUM、TYPE_MESSAGE、またはTYPE_GROUPのいずれかである必要があります。
  optional Type type = 5;

  // メッセージおよび列挙型の場合、これは型の名前です。名前が'.'で始まる場合、完全修飾名です。それ以外の場合、C++のようなスコープルールが使用されて型が見つかります（つまり、まずこのメッセージ内のネストされた型が検索され、次に親、最終的にはルート名前空間まで検索されます）。
  optional string type_name = 6;

  // 拡張の場合、これは拡張されている型の名前です。type_nameと同じ方法で解決されます。
  optional string extendee = 2;

  // 数値型の場合、値の元のテキスト表現が含まれます。
  // ブール値の場合、"true"または"false"。
  // 文字列の場合、デフォルトのテキスト内容が含まれます（エスケープされていません）。
  // バイトの場合、Cでエスケープされた値が含まれます。すべてのバイトが128以上の場合はエスケープされます。
  optional string default_value = 7;

  // 設定されている場合、含まれる型のoneof_declリスト内のoneofのインデックスを指定します。このフィールドはそのoneofのメンバーです。
  optional int32 oneof_index = 9;

  // このフィールドのJSON名。値はプロトコルコンパイラによって設定されます。ユーザーがこのフィールドに"json_name"オプションを設定している場合、そのオプションの値が使用されます。それ以外の場合、フィールドの名前をcamelCaseに変換して推測されます。
  optional string json_name = 10;

  optional FieldOptions options = 8;

  // trueの場合、これはproto3の"optional"です。proto3フィールドがオプショナルの場合、フィールドタイプに関係なく存在を追跡します。
  //
  // proto3_optionalがtrueの場合、このフィールドは、古いproto3クライアントにこのフィールドの存在が追跡されていることを示すために、oneofに属している必要があります。このoneofは"合成"oneofとして知られ、このフィールドはその唯一のメンバーでなければなりません（各proto3オプショナルフィールドには独自の合成oneofがあります）。合成oneofは記述子にのみ存在し、APIを生成しません。合成oneofはすべての「実際の」oneofの後に配置する必要があります。
  //
  // メッセージフィールドの場合、proto3_optionalは、繰り返しのないメッセージフィールドは常に存在を追跡するため、意味的な変更を作成しません。ただし、ユーザーが「optional」を書いたかどうかの意味的な詳細を示します。これは.protoファイルをラウンドトリップする際に役立ちます。一貫性を保つため、メッセージフィールドには合成oneofも付けますが、存在を追跡する必要はありません。これは、パーサーがフィールドがメッセージか列挙型かを区別できないため、常に合成oneofを作成する必要があるため特に重要です。
  //
  // Proto2オプショナルフィールドはこのフラグを設定しません。なぜなら、それらは既に`LABEL_OPTIONAL`でオプショナルを示しているからです。
  optional bool proto3_optional = 17;
}
```

```protobuf
// 一つの選択肢を記述します。
message OneofDescriptorProto {
  optional string name = 1;
  optional OneofOptions options = 2;
}
{/*examples*/}
// 列挙型を記述します。
message EnumDescriptorProto {
  optional string name = 1;

  repeated EnumValueDescriptorProto value = 2;

  optional EnumOptions options = 3;

  // 予約された数値値の範囲。予約された値は同じ列挙型内のエントリによって使用されない可能性があります。予約された範囲は重なってはいけません。
  //
  // DescriptorProto.ReservedRange とは異なり、これは int32 ドメイン全体を適切に表すことができるように含まれることに注意してください。
  message EnumReservedRange {
    optional int32 start = 1;  // 含まれる。
    optional int32 end = 2;    // 含まれる。
  }

  // 予約された数値値の範囲。予約された数値は同じ列挙型宣言内の列挙値によって使用されない可能性があります。予約された範囲は重なってはいけません。
  repeated EnumReservedRange reserved_range = 4;

  // 再利用できない予約された列挙値の名前。特定の名前は一度だけ予約できます。
  repeated string reserved_name = 5;
}
{/*examples*/}
// 列挙型内の値を記述します。
message EnumValueDescriptorProto {
  optional string name = 1;
  optional int32 number = 2;

  optional EnumValueOptions options = 3;
}
{/*examples*/}
// サービスを記述します。
message ServiceDescriptorProto {
  optional string name = 1;
  repeated MethodDescriptorProto method = 2;

  optional ServiceOptions options = 3;
}
{/*examples*/}
// サービスのメソッドを記述します。
message MethodDescriptorProto {
  optional string name = 1;

  // 入力と出力の型名。これらは FieldDescriptorProto.type_name と同じ方法で解決されますが、メッセージ型を参照する必要があります。
  optional string input_type = 2;
  optional string output_type = 3;

  optional MethodOptions options = 4;

  // クライアントが複数のクライアントメッセージをストリーム化するかどうかを識別します
  optional bool client_streaming = 5 [default = false];
  // サーバーが複数のサーバーメッセージをストリーム化するかどうかを識別します
  optional bool server_streaming = 6 [default = false];
}
{/*examples*/}
// ===================================================================
// オプション
```

```protobuf
// 上記の各定義には、"options" が添付されている場合があります。これらは、コードをわずかに異なる方法で生成する可能性がある注釈であり、プロトコルメッセージを操作するコードのヒントを含む場合もあります。

// クライアントは、*Options メッセージの拡張としてカスタムオプションを定義することができます。これらの拡張は、解析時にはまだ知られていないかもしれないため、パーサーはそれらの値を保存することができません。代わりに、*Options メッセージ内の uninterpreted_option というフィールドにそれらを保存します。このフィールドは、すべての *Options メッセージで同じ名前を持たなければなりません。その後、ディスクリプタを構築する際にこのフィールドを使用して拡張をポピュレートします。この時点ですべてのプロトが解析されているため、すべての拡張が既知です。

// カスタムオプションの拡張番号は、次のように選択できます:
// * 単一のアプリケーションや組織内でのみ使用されるオプションや実験的なオプションには、フィールド番号 50000 から 99999 を使用します。同じ番号を複数のオプションで使用しないようにする責任があります。
// * 複数の独立したエンティティによって公開および使用されるオプションには、拡張番号を予約するために protobuf-global-extension-registry@google.com にメールを送信します。プロジェクト名（例: Objective-C プラグイン）とプロジェクトのウェブサイト（利用可能な場合）を提供するだけで十分です。どのように使用するかを説明する必要はありません。通常、1 つの拡張番号だけが必要です。1 つの拡張番号で複数のオプションを宣言することができます。サブメッセージにそれらを配置することで、1 つの拡張番号で複数のオプションを宣言できます。例については、ドキュメントの Custom Options セクションを参照してください:
//   /programming-guides/proto#options
//   これが人気が出れば、オプション番号を自動的に割り当てるための Web サービスが設定されます。

message FileOptions {

  // この .proto から生成されたクラスが配置される Java パッケージを設定します。デフォルトでは proto パッケージが使用されますが、これは通常、逆ドメイン名で始まらないため適切ではありません。
  optional string java_package = 1;
```
{/*examples*/}

```protobuf
  // .proto ファイルに生成されるラッパー Java クラスの名前を制御します。
  // そのクラスには、.proto ファイルの getDescriptor() メソッドと、.proto ファイルで定義されたトップレベルの拡張が常に含まれます。
  // java_multiple_files が無効になっている場合、.proto ファイルの他のすべてのクラスは、単一のラッパー外部クラスの中にネストされます。
  optional string java_outer_classname = 8;

  // 有効になっている場合、Java コード生成ツールは、.proto ファイルで定義された各トップレベルメッセージ、列挙型、サービスに対して別々の .java ファイルを生成します。
  // したがって、これらの型はラッパークラスにネストされません。
  // ただし、java_outer_classname で指定されたラッパークラスは、ファイルの getDescriptor() メソッドと、ファイルで定義されたトップレベルの拡張を含むように生成されます。
  optional bool java_multiple_files = 10 [default = false];

  // このオプションは何もしません。
  optional bool java_generate_equals_and_hash = 20 [deprecated=true];

  // true に設定されている場合、Java2 コード生成ツールは、非 UTF-8 バイトシーケンスを文字列フィールドに割り当てようとする試みが行われるたびに例外をスローするコードを生成します。
  // メッセージリフレクションも同様です。
  // ただし、拡張フィールドは引き続き非 UTF-8 バイトシーケンスを受け入れます。
  // このオプションは、lite ランタイムと併用した場合には効果がありません。
  optional bool java_string_check_utf8 = 27 [default = false];

  // 生成されたクラスは、速度またはコードサイズの最適化が可能です。
  enum OptimizeMode {
    SPEED = 1;         // パース、シリアライズなどのための完全なコードを生成します。
    CODE_SIZE = 2;     // これらのメソッドを実装するために ReflectionOps を使用します。
    LITE_RUNTIME = 3;  // MessageLite と lite ランタイムを使用してコードを生成します。
  }
  optional OptimizeMode optimize_for = 9 [default = SPEED];

  // この .proto から生成される構造体が配置される Go パッケージを設定します。
  // 省略された場合、Go パッケージは次のように派生します:
  //   - パッケージのインポートパスのベース名（提供されている場合）。
  //   - それ以外の場合、.proto ファイル内のパッケージステートメント（存在する場合）。
  //   - それ以外の場合、拡張子を除いた .proto ファイルのベース名。
  optional string go_package = 11;
```  

```protobuf
  // 各言語で汎用サービスを生成するべきですか？「汎用」サービスは特定のRPCシステムに固有ではありません。各言語の主要なコードジェネレータによって生成されます（追加のプラグインなし）。
  // 汎用サービスは、google.protobufの初期バージョンでサポートされていたサービス生成の唯一の種類でした。
  //
  // 現在、汎用サービスは、特定のRPCシステムに固有のコードを生成するプラグインの使用を推奨するために非推奨とされています。したがって、これらはデフォルトでfalseになります。汎用サービスに依存する古いコードは、明示的にtrueに設定する必要があります。
  optional bool cc_generic_services = 16 [default = false];
  optional bool java_generic_services = 17 [default = false];
  optional bool py_generic_services = 18 [default = false];
  optional bool php_generic_services = 42 [default = false];

  // このファイルは非推奨ですか？
  // ターゲットプラットフォームによっては、このファイル内のすべてにDeprecated注釈を出力するか、完全に無視されるかもしれません。少なくとも、これはファイルを非推奨にするための形式化です。
  optional bool deprecated = 23 [default = false];

  // このファイルのprotoメッセージでのアリーナの使用を有効にします。これはC++の生成されたクラスにのみ適用されます。
  optional bool cc_enable_arenas = 31 [default = true];

  // すべてのobjective c生成クラスに先頭に付加されるobjective cクラスプレフィックスを設定します。デフォルトはありません。
  optional string objc_class_prefix = 36;

  // 生成されたクラスの名前空間。デフォルトはパッケージです。
  optional string csharp_namespace = 37;

  // デフォルトでは、Swiftジェネレータはprotoパッケージを取り、CamelCaseに変換し、'.'をアンダースコアに置き換えて、定義された型/シンボルにプレフィックスとして使用します。このオプションが提供されると、代わりにこの値を使用して型/シンボルにプレフィックスを付けます。
  optional string swift_prefix = 39;

  // この.protoから生成されたすべてのphp生成クラスに先頭に付加されるphpクラスプレフィックスを設定します。デフォルトは空です。
  optional string php_class_prefix = 40;
```  

```protobuf
  // php生成クラスの名前空間を変更するオプションです。デフォルトは空です。このオプションが空の場合、パッケージ名が名前空間の決定に使用されます。
  optional string php_namespace = 41;
  
  // php生成メタデータクラスの名前空間を変更するオプションです。デフォルトは空です。このオプションが空の場合、protoファイル名が名前空間の決定に使用されます。
  optional string php_metadata_namespace = 44;
  
  // ruby生成クラスのパッケージを変更するオプションです。デフォルトは空です。このオプションが設定されていない場合、パッケージ名がrubyパッケージの決定に使用されます。
  optional string ruby_package = 45;
  
  // パーサーは認識しないオプションをここに保存します。
  // 上記の「Options」セクションのドキュメントを参照してください。
  repeated UninterpretedOption uninterpreted_option = 999;
  
  // クライアントはこのメッセージの拡張でカスタムオプションを定義できます。
  // 上記の「Options」セクションのドキュメントを参照してください。
  extensions 1000 to max;
  
  reserved 38;
}

message MessageOptions {
  // 拡張機能のために古いproto1 MessageSetワイヤーフォーマットを使用する場合はtrueに設定します。
  // これは、MessageSetワイヤーフォーマットとの後方互換性のために提供されています。他の理由でこれを使用しないでください：効率が悪く、機能が少なく、複雑です。
  //
  // メッセージは次のように厳密に定義する必要があります：
  //   message Foo {
  //     option message_set_wire_format = true;
  //     extensions 4 to max;
  //   }
  // メッセージには定義されたフィールドがない必要があります。MessageSetには拡張機能のみがあります。
  //
  // あなたのタイプのすべての拡張機能は単一のメッセージでなければなりません。たとえば、int32、列挙型、または繰り返しメッセージにはできません。
  //
  // これがオプションであるため、上記の2つの制限はプロトコルコンパイラによって強制されません。
  optional bool message_set_wire_format = 1 [default = false];
  
  // 標準の「descriptor()」アクセサの生成を無効にします。これは、同じ名前のフィールドと競合する可能性があるためです。これはproto1からの移行を容易にするためのものであり、新しいコードでは「descriptor」という名前のフィールドを避けるべきです。
  optional bool no_standard_descriptor_accessor = 2 [default = false];
```

```protobuf
// このメッセージは非推奨ですか？
// ターゲットプラットフォームによっては、このメッセージにDeprecatedアノテーションが出力されることがあります。
// または、完全に無視されることもあります。少なくとも、これはメッセージを非推奨にするための形式化です。
optional bool deprecated = 3 [default = false];

reserved 4, 5, 6;

// メッセージがマップフィールドの自動生成されたマップエントリータイプであるかどうか。
//
// マップフィールドの場合：
//     map&lt;KeyType, ValueType&gt; map_field = 1;
// パースされたディスクリプタは次のようになります：
//     message MapFieldEntry {
//         option map_entry = true;
//         optional KeyType key = 1;
//         optional ValueType value = 2;
//     }
//     repeated MapFieldEntry map_field = 1;
//
// 実装は、map_entry=trueメッセージを生成しないことを選択することができますが、
// キーと値を保持するためにターゲット言語でネイティブマップを使用します。
// このような実装のリフレクションAPIは、まだフィールドが繰り返しメッセージフィールドであるかのように動作する必要があります。
//
// 注：.protoファイルでオプションを設定しないでください。常にマップ構文を使用してください。
// オプションはprotoコンパイラパーサーによって暗黙的にのみ設定されるべきです。
optional bool map_entry = 7;

reserved 8;  // javalite_serializable
reserved 9;  // javanano_as_lite

// パーサーは認識できないオプションをここに保存します。上記を参照してください。
repeated UninterpretedOption uninterpreted_option = 999;

// クライアントはこのメッセージの拡張でカスタムオプションを定義できます。上記を参照してください。
extensions 1000 to max;
}

message FieldOptions {
// ctypeオプションは、C++コードジェネレータに、通常とは異なるフィールドの表現を使用するように指示します。
// 特定のオプションについては以下を参照してください。このオプションはまだオープンソースリリースでは実装されていません。
optional CType ctype = 1 [default = STRING];
enum CType {
// デフォルトモード。
STRING = 0;

CORD = 1;

STRING_PIECE = 2;

}
// packedオプションは、繰り返しプリミティブフィールドに有効にでき、
// より効率的な表現をワイヤー上で可能にします。
// 各要素のたびにタグとタイプを繰り返し書き込む代わりに、
// 全体の配列が単一の長さ区切りのブロブとしてエンコードされます。
// proto3では、packedエンコーディングを使用しないように明示的に設定するだけで、
// packedオプションを無効にすることができます。
optional bool packed = 2;
```

```protobuf
// jstypeオプションは、フィールドの値に使用されるJavaScriptタイプを決定します。このオプションは、64ビットの整数型および固定型（int64、uint64、sint64、fixed64、sfixed64）にのみ許可されます。jstypeがJS_STRINGのフィールドは、JavaScript文字列として表され、大きな値が浮動小数点JavaScriptに変換される際に発生する精度の損失を回避します。jstypeにJS_NUMBERを指定すると、生成されたJavaScriptコードがJavaScriptの「number」型を使用するようになります。デフォルトオプションJS_NORMALの動作は実装依存です。

// このオプションは、追加のタイプ（例：goog.math.Integer）を追加できるようにするために列挙型です。
optional JSType jstype = 6 [default = JS_NORMAL];
enum JSType {
  // デフォルトタイプを使用します。
  JS_NORMAL = 0;

  // JavaScript文字列を使用します。
  JS_STRING = 1;

  // JavaScript数値を使用します。
  JS_NUMBER = 2;
}

// このフィールドを遅延的に解析する必要がありますか？遅延は、メッセージ型フィールドにのみ適用されます。これは、外部メッセージが最初に解析されるとき、内部メッセージの内容が解析されず、代わりにエンコードされた形式で保存されることを意味します。内部メッセージは、最初にアクセスされたときに実際に解析されます。

// これはヒントに過ぎません。実装は、このオプションの値に関係なく、イーガーまたはレイジーな解析を使用するかどうかを選択する自由があります。ただし、このオプションをtrueに設定すると、プロトコルの作成者は、このフィールドで遅延解析を使用する価値があると考えていることを示唆しています。

// このオプションは、生成されたコードの公開インターフェースに影響を与えません。すべてのメソッドシグネチャは同じままです。さらに、このオプションによってインターフェースのスレッドセーフ性は影響を受けません。constメソッドは引き続き複数のスレッドから同時に呼び出すことが安全であり、非constメソッドは引き続き排他的なアクセスを必要とします。

// 遅延サブメッセージ内の必須フィールドをチェックしないように選択する実装があることに注意してください。つまり、外部メッセージでIsInitialized()を呼び出しても、内部メッセージに必要なフィールドが欠落している場合でもtrueが返される可能性があります。これは、それ以外の場合、内部メッセージを解析してチェックを実行する必要があるため、遅延解析の目的を阻害するためです。必須フィールドをチェックしないように選択する実装は、それについて一貫していなければなりません。つまり、特定のサブメッセージについて、実装はその必須フィールドを*常に*チェックするか、*決して*チェックしないかを選択しなければなりません。メッセージが解析されているかどうかに関係なく。
optional bool lazy = 5 [default = false];
```

```protobuf
// このフィールドは非推奨ですか？
// ターゲットプラットフォームによっては、アクセサーにDeprecatedアノテーションを出力することができるか、
// または完全に無視されるかもしれません。少なくとも、これはフィールドを非推奨にするための形式化です。
optional bool deprecated = 3 [default = false];

// Google内部の移行専用です。使用しないでください。
optional bool weak = 10 [default = false];

// パーサーは認識しないオプションをここに保存します。上記を参照してください。
repeated UninterpretedOption uninterpreted_option = 999;

// クライアントは、このメッセージの拡張でカスタムオプションを定義できます。上記を参照してください。
extensions 1000 to max;

reserved 4;  // jtypeが削除されました
}

message OneofOptions {
// パーサーは認識しないオプションをここに保存します。上記を参照してください。
repeated UninterpretedOption uninterpreted_option = 999;

// クライアントは、このメッセージの拡張でカスタムオプションを定義できます。上記を参照してください。
extensions 1000 to max;
}

message EnumOptions {

// 異なるタグ名を同じ値にマッピングすることを許可するには、このオプションをtrueに設定します。
optional bool allow_alias = 2;

// この列挙型は非推奨ですか？
// ターゲットプラットフォームによっては、列挙型にDeprecatedアノテーションを出力することができるか、
// または完全に無視されるかもしれません。少なくとも、これは列挙型を非推奨にするための形式化です。
optional bool deprecated = 3 [default = false];

reserved 5;  // javanano_as_lite

// パーサーは認識しないオプションをここに保存します。上記を参照してください。
repeated UninterpretedOption uninterpreted_option = 999;

// クライアントは、このメッセージの拡張でカスタムオプションを定義できます。上記を参照してください。
extensions 1000 to max;
}

message EnumValueOptions {
// この列挙値は非推奨ですか？
// ターゲットプラットフォームによっては、列挙値にDeprecatedアノテーションを出力することができるか、
// または完全に無視されるかもしれません。少なくとも、これは列挙値を非推奨にするための形式化です。
optional bool deprecated = 1 [default = false];

// パーサーは認識しないオプションをここに保存します。上記を参照してください。
repeated UninterpretedOption uninterpreted_option = 999;
```

```protobuf
// クライアントはこのメッセージの拡張でカスタムオプションを定義できます。上記を参照してください。
extensions 1000 to max;
}

message ServiceOptions {

// 注：フィールド番号1から32はGoogleの内部RPC用に予約されています。
// これらの番号を自分たちだけで使用していたため、Protocol Buffersをリリースする前から使用していました。

// このサービスは非推奨ですか？
// ターゲットプラットフォームによっては、このサービスに対してDeprecatedアノテーションを出力するか、
// 完全に無視されるかもしれません。少なくとも、これはサービスを非推奨にするための形式化です。
optional bool deprecated = 33 [default = false];

// パーサーは認識できないオプションをここに保存します。上記を参照してください。
repeated UninterpretedOption uninterpreted_option = 999;

// クライアントはこのメッセージの拡張でカスタムオプションを定義できます。上記を参照してください。
extensions 1000 to max;
}

message MethodOptions {

// 注：フィールド番号1から32はGoogleの内部RPC用に予約されています。
// これらの番号を自分たちだけで使用していたため、Protocol Buffersをリリースする前から使用していました。

// このメソッドは非推奨ですか？
// ターゲットプラットフォームによっては、このメソッドに対してDeprecatedアノテーションを出力するか、
// 完全に無視されるかもしれません。少なくとも、これはメソッドを非推奨にするための形式化です。
optional bool deprecated = 33 [default = false];

// このメソッドは副作用がない（またはHTTPの用語で安全）、あるいは冪等性があるか、
// それともどちらでもないですか？ HTTPベースのRPC実装は、安全なメソッドにGET動詞を選択し、
// 冪等なメソッドにデフォルトのPOSTの代わりにPUT動詞を選択することができます。
enum IdempotencyLevel {
IDEMPOTENCY_UNKNOWN = 0;
NO_SIDE_EFFECTS = 1; // 冪等性を意味します
IDEMPOTENT = 2; // 冪等ですが、副作用があるかもしれません
}
optional IdempotencyLevel idempotency_level = 34
[default = IDEMPOTENCY_UNKNOWN];

// パーサーは認識できないオプションをここに保存します。上記を参照してください。
repeated UninterpretedOption uninterpreted_option = 999;
```

```protobuf
// クライアントはこのメッセージの拡張でカスタムオプションを定義できます。上記を参照してください。
extensions 1000 to max;
}

// パーサーが認識しないオプションを表すメッセージ。これはコンパイラ::Parserクラスによって作成されたオプションプロトにのみ表示されます。
// DescriptorPoolは、Descriptorオブジェクトを構築する際にこれらを解決します。したがって、
// Descriptorオブジェクト（たとえば、Descriptor::options()で返されるもの、またはDescriptor::CopyTo()によって生成されるもの）のオプションプロトには、
// UninterpretedOptionsは含まれません。
message UninterpretedOption {
  // 解釈されないオプションの名前。各文字列は、ドットで区切られた名前のセグメントを表します。セグメントが拡張を表す場合（.protoファイルのオプション仕様で括弧で示される）、is_extensionはtrueです。
  // 例：{ ["foo", false], ["bar.baz", true], ["qux", false] } は "foo.(bar.baz).qux" を表します。
  message NamePart {
    required string name_part = 1;
    required bool is_extension = 2;
  }
  repeated NamePart name = 2;

  // 解釈されないオプションの値。解析中にトークナイザがそれを識別した型でのみ1つ設定されるべきです。
  optional string identifier_value = 3;
  optional uint64 positive_int_value = 4;
  optional int64 negative_int_value = 5;
  optional double double_value = 6;
  optional bytes string_value = 7;
  optional string aggregate_value = 8;
}

// ===================================================================
// オプションのソースコード情報

// FileDescriptorProtoから生成された元のソースファイルに関する情報をカプセル化します。
message SourceCodeInfo {
  // Locationは、特定の定義に対応する.protoファイル内のソースコードの一部を識別します。この情報は、IDE、コードインデクサ、ドキュメント生成ツールなどに役立つことを意図しています。
  //
  // たとえば、次のようなファイルがあるとします：
  //   message Foo {
  //     optional string foo = 1;
  //   }
  // フィールド定義だけを見てみましょう：
  //   optional string foo = 1;
  //   ^       ^^     ^^  ^  ^^^
  //   a       bc     de  f  ghi
  // 次の場所があります：
  //   span   path               represents
  //   [a,i)  [ 4, 0, 2, 0 ]     フィールド定義全体。
  //   [a,b)  [ 4, 0, 2, 0, 4 ]  ラベル（optional）。
  //   [c,d)  [ 4, 0, 2, 0, 5 ]  型（string）。
  //   [e,f)  [ 4, 0, 2, 0, 1 ]  名前（foo）。
  //   [g,h)  [ 4, 0, 2, 0, 3 ]  番号（1）。
  //
  // メモ：
  // - 1つの場所は、繰り返しフィールド自体を参照する場合があります（つまり、それが特定のインデックスを持たない場合）。これは、複数の要素が論理的に単一のコードセグメントに囲まれている場合に使用されます。たとえば、複数の拡張定義を含む可能性がある拡張ブロック全体には、インデックスなしで "extensions" 繰り返しフィールドを参照する外部の場所があります。
  // - 複数の場所が同じパスを持つことがあります。これは、単一の論理的な宣言が複数の場所に分散されている場合に発生します。最も明らかな例は、再び "extend" ブロックです。同じスコープ内に複数の拡張ブロックがある場合、それぞれのパスが同じになります。
  // - 場所のスパンが常に親のスパンの部分集合であるわけではありません。たとえば、拡張宣言の "extendee" は "extend" ブロックの先頭に表示され、ブロック内のすべての拡張で共有されます。
  // - 場所のスパンが他の場所のスパンの部分集合であるからといって、それが子孫であるとは限りません。たとえば、 "group" は単一の宣言で型とフィールドの両方を定義します。したがって、型とフィールドおよびそれらのコンポーネントに対応する場所は重なり合います。
  // - 場所を解釈しようとするコードは、将来さらに多くの種類の場所が記録される可能性があるため、理解できない場所を無視するように設計されているべきです。
  repeated Location location = 1;
  message Location {
    // この場所で定義されたFileDescriptorProtoのどの部分を識別します。
    //
    // 各要素はフィールド番号またはインデックスです。これらは、ルートのFileDescriptorProtoから定義の場所までのパスを形成します。たとえば、このパス：
    //   [ 4, 3, 2, 7, 1 ]
    // は次の場所を指します：
    //   file.message_type(3)  // 4, 3
    //       .field(7)         // 2, 7
    //       .name()           // 1
    // これは、FileDescriptorProto.message_typeがフィールド番号4であるためです：
    //   repeated DescriptorProto message_type = 4;
    // およびDescriptorProto.fieldがフィールド番号2であるためです：
    //   repeated FieldDescriptorProto field = 2;
    // そしてFieldDescriptorProto.nameがフィールド番号1であるためです：
    //   optional string name = 1;
    //
    // したがって、上記のパスはフィールド名の場所を示します。最後の要素を削除した場合：
    //   [ 4, 3, 2, 7 ]
    // このパスはフィールド宣言全体を参照します（ラベルの最初からセミコロンまで）。
    repeated int32 path = 1 [packed = true];
```

```protobuf
// 常に3つまたは4つの要素があります：開始行、開始列、終了行（オプション、それ以外の場合は開始行と同じと見なされます）、終了列。
// これらは効率的に1つのフィールドにパックされます。行と列の番号は0から始まることに注意してください。通常、ユーザーに表示する前にそれぞれに1を追加する必要があります。
repeated int32 span = 2 [packed = true];

// この SourceCodeInfo が完全な宣言を表す場合、宣言の前後に表示されるコメントが含まれます。
//
// 連続する行コメントが連続する行に表示され、その行に他のトークンが表示されていない場合、1つのコメントとして扱われます。
//
// leading_detached_comments は、現在の要素の前に（しかし接続されていない）表示されるコメントの段落を保持します。空行で区切られた各段落は、繰り返しフィールド内の1つのコメント要素になります。
//
// コメントの内容のみが提供されます。コメントマーカー（例：//）は削除されます。ブロックコメントの場合、最初の行以外の各行の先頭から先頭の空白とアスタリスクが削除されます。
// 改行が出力に含まれます。
//
// 例：
//
//   optional int32 foo = 1;  // foo に関連付けられたコメント。
//   // bar に関連付けられたコメント。
//   optional int32 bar = 2;
//
//   optional string baz = 3;
//   // baz に関連付けられたコメント。
//   // baz に関連付けられた別の行。
//
//   // qux に関連付けられたコメント。
//   //
//   // qux に関連付けられた別の行。
//   optional double qux = 4;
//
//   // corge に対する切り離されたコメント。これは qux または corge への先行または後続コメントではないため、
//   // それぞれから空行で区切られています。
//
//   // corge のための切り離されたコメント段落 2。
//
//   optional string corge = 5;
//   /* corge に関連付けられたブロックコメント
//    *。先頭のアスタリスク
//    * は削除されます。 */ // /* grault に関連付けられたブロックコメント。
//   * grault. */
//   optional int32 grault = 6;
//
//   // 無視された切り離されたコメント。
optional string leading_comments = 3;
optional string trailing_comments = 4;
repeated string leading_detached_comments = 6;
```

```protobuf
  }
}

// 生成されたコードと元のソースファイルとの関係を記述します。
// GeneratedCodeInfo メッセージは、1 つの生成元のソースファイルに関連付けられていますが、異なるソース .proto ファイルへの参照を含むことがあります。
message GeneratedCodeInfo {
  // アノテーションは、生成されたコード内のテキストの一部を、その生成元の .proto ファイルの要素に接続します。
  repeated Annotation annotation = 1;
  message Annotation {
    // 元のソース .proto ファイル内の要素を識別します。このフィールドは SourceCodeInfo.Location.path と同じ形式でフォーマットされています。
    repeated int32 path = 1 [packed = true];

    // 元のソース .proto へのファイルシステムパスを識別します。
    optional string source_file = 2;

    // 生成されたコード内の識別されたオブジェクトに関連するバイト単位の開始オフセットを識別します。
    optional int32 begin = 3;

    // 生成されたコード内の識別されたオフセットに関連するバイト単位の終了オフセットを識別します。終了オフセットは、関連するバイトの最後の 1 バイトを超える必要があります（つまり、テキストの長さ = 終了 - 開始）。
    optional int32 end = 4;
  }
}
```

Please paste the Markdown content you would like me to translate into Japanese.
