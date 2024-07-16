```markdown
+++
title = "plugin.pb.h"
toc_hide = "true"
linkTitle = "C++"
description = "このセクションには、C++でプロトコルバッファクラスを使用するためのリファレンスドキュメントが含まれています。"
type = "docs"
+++

<p><code>#include &lt;google/protobuf/compiler/plugin.pb.h&gt;<br>namespace <a href="#google.protobuf.compiler">google::protobuf::compiler</a></code></p><p>protocプラグインのAPI。</p><p>このファイルは、protocコードジェネレータプラグインのAPIを構成するプロトコルメッセージクラスのセットを定義しています。C++で書かれたプラグインは、おそらくprotobufレベルのAPIではなく<a href="google.protobuf.compiler.plugin">plugin.h</a>のAPIをベースに構築すべきですが、他の言語で書かれたプラグインは以下で定義された生のメッセージを扱う必要があります。</p><p>プロトコルコンパイラは現在、自動生成されたドキュメントをサポートしていないため、このページには説明が含まれていません。このファイルは、<code>plugin.proto</code>からプロトコルコンパイラによって生成されました。その内容は以下の通りです:</p>

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
//
// 警告: プラグインインターフェイスは現在実験的であり、変更の対象となります。
//
// protoc（別名プロトコルコンパイラ）はプラグインを介して拡張できます。プラグインは
// 標準入力から CodeGeneratorRequest を読み取り、標準出力に CodeGeneratorResponse を書き込むだけのプログラムです。
//
// C++ を使用して書かれたプラグインは、ここで定義された生のプロトコルを扱う代わりに google/protobuf/compiler/plugin.h を使用できます。
//
// プラグインの実行可能ファイルはパスのどこかに配置するだけで十分です。プラグインは "protoc-gen-$NAME" という名前にする必要があり、
// その後、protoc に "--${NAME}_out" フラグが渡されたときに使用されます。

syntax = "proto2";

package google.protobuf.compiler;
option java_package = "com.google.protobuf.compiler";
option java_outer_classname = "PluginProtos";

option go_package = "google.golang.org/protobuf/types/pluginpb";

import "google/protobuf/descriptor.proto";

// プロトコルコンパイラのバージョン番号。
message Version {
  optional int32 major = 1;
  optional int32 minor = 2;
  optional int32 patch = 3;
  // アルファ、ベータ、またはリリース候補のサフィックス、例: "alpha-1", "rc2"。メインラインの安定リリースの場合は空にする必要があります。
  optional string suffix = 4;
}

// エンコードされた CodeGeneratorRequest がプラグインの標準入力に書き込まれます。
message CodeGeneratorRequest {
  // コマンドラインで明示的にリストされた .proto ファイル。コードジェネレータはこれらのファイルのみにコードを生成する必要があります。各ファイルの
  // 記述子は、以下の proto_file に含まれます。
  repeated string file_to_generate = 1;

  // コマンドラインで渡されたジェネレータパラメータ。
  optional string parameter = 2;

  // files_to_generate およびそれらがインポートするすべてのファイルの FileDescriptorProtos。ファイルはトポロジカル順に表示されるため、各ファイルは
  // それをインポートするファイルの前に表示されます。
  //
  // protoc は、proto_files が上記のフィールドの後に書き込まれることを保証しますが、これは protobuf ワイヤーフォーマットでは技術的に保証されていないことに注意してください。
  // 理論的には、プラグインが FileDescriptorProtos をストリーミングして一度にすべてをメモリに読み込むのではなく、1 つずつ処理することができる可能性があります。
  // ただし、この執筆時点では、protoc 側で同様に最適化されていないため、すべてのフィールドを一度にメモリに格納してからプラグインに送信します。
  //
  // FileDescriptorProto のフィールドおよび拡張の型名は常に完全修飾名です。
  repeated FileDescriptorProto proto_file = 15;
}
```

```protobuf
  // プロトコルコンパイラのバージョン番号。
  optional Version compiler_version = 3;
}
```

```protobuf
// プラグインは、エンコードされたCodeGeneratorResponseを標準出力に書き込みます。
message CodeGeneratorResponse {
  // エラーメッセージ。空でない場合、コード生成が失敗しました。プラグインプロセスは、この方法でエラーを報告しても、ステータスコードがゼロであるべきです。
  //
  // これは、.protoファイルのエラーを示すために使用するべきです。これにより、コードジェネレータが正しいコードを生成できないエラーが示されます。protoc自体に問題を示すエラーは、入力CodeGeneratorRequestが解析不可能であるなどの問題を示すものであるべきです。これらは、stderrにメッセージを書き込んで、ゼロでないステータスコードで終了することで報告されるべきです。
  optional string error = 1;

  // コードジェネレータがサポートするサポートされる機能のビットマスク。
  // これは、Feature列挙型の値のビットごとの「または」です。
  optional uint64 supported_features = 2;

  // code_generator.hと同期します。
  enum Feature {
    FEATURE_NONE = 0;
    FEATURE_PROTO3_OPTIONAL = 1;
  }

  // 生成された単一のファイルを表します。
  message File {
    // 出力ディレクトリに対する相対的なファイル名。名前には "." または ".." のコンポーネントを含めてはいけません。絶対ではなく相対である必要があります（つまり、ファイルは出力ディレクトリの外にあってはなりません）。パスセパレータとして "/" を使用する必要があります。"\"ではなく。
    //
    // 名前が省略された場合、内容は前のファイルに追加されます。これにより、ジェネレータは大きなファイルを小さなチャンクに分割し、生成されたテキストをストリームバックしてprotocに返すことができます。これにより、大きなファイルが一度に完全にメモリに存在する必要がなくなります。この執筆時点では、protocはこれに最適化されていません -- ファイルをディスクに書き込む前に、CodeGeneratorResponse全体を読み込みます。
    optional string name = 1;

// 空でない場合、指定されたファイルがすでに存在し、ここにある内容が定義された挿入ポイントに挿入されるべきであることを示します。この機能により、コードジェネレータが別のコードジェネレータによって生成された出力を拡張することができます。元のジェネレータは、ファイルに特別な注釈を配置することで挿入ポイントを提供できます。
//   @@protoc_insertion_point(NAME)
// 注釈には、行の前後に任意のテキストを配置できます。これにより、コメントに配置できます。NAMEは、他のジェネレータが挿入ポイントとして使用する識別子に置き換える必要があります。このポイントに挿入されたコードは、挿入ポイントを含む行の直前に配置されます（したがって、同じポイントに複数の挿入が追加された場合、追加された順に出力されます）。
// ダブル@は、生成されたコードに挿入ポイントのように見えるものが偶然含まれる可能性を低くすることを意図しています。
//
// たとえば、C++コードジェネレータは、生成された.pb.hファイルに次の行を配置します。
//   // @@protoc_insertion_point(namespace_scope)
// この行は、ファイルのパッケージ名前空間のスコープ内に現れますが、特定のクラスの外側にあります。別のプラグインは、挿入ポイント「namespace_scope」を指定して、このスコープに配置する追加のクラスや他の宣言を生成できます。
//
// 挿入ポイントを含む行が先頭に空白で始まる場合、挿入されるテキストの各行に同じ空白が追加されます。これは、Pythonのような言語にとって便利です。こういった言語では、挿入ポイントコメントは、挿入されるコードがそのコンテキストで正しく機能するために必要な量だけインデントされるべきです。
//
// 初期ファイルを生成するコードジェネレータと、それに挿入するコードジェネレータは、protocの単一の呼び出しの一部として実行されなければなりません。
// コードジェネレータは、コマンドラインに表示される順序で実行されます。
//
// |insertion_point|が存在する場合、|name|も存在する必要があります。
optional string insertion_point = 2;
```

```markdown
// The file contents.
optional string content = 15;

// ファイルに挿入されるファイルコンテンツを説明する情報。挿入ポイントが使用されている場合、この情報は適切にオフセットされ、生成されたファイルのコード生成メタデータに挿入されます。
optional GeneratedCodeInfo generated_code_info = 16;

  }
  repeated File file = 15;
}
```
<table width="100%"><tr><th colspan="2"><h3 style="margin-top: 4px">このファイル内のクラス</h3></th></tr></table><table><tr><th colspan="2"><h3 style="margin-top: 4px">ファイルメンバー</h3><div style="font-style: italic; font-weight: normal;">これらの定義はどのクラスにも属していません。</div></th></tr><tr><td style="border-right-width: 0px; text-align: right;"><code>const ::protobuf::internal::DescriptorTable</code></td><td style="border-left-width: 0px"id="descriptor_table_google_2fprotobuf_2fcompiler_2fplugin_2eproto"><div style="padding-left: 16px; text-indent: -16px"><code><b>descriptor_table_google_2fprotobuf_2fcompiler_2fplugin_2eproto</b></code></div></td></tr><tr><td style="border-right-width: 0px; text-align: right;"><code>PROTOBUF_NAMESPACE_CLOSE PROTOBUF_NAMESPACE_OPEN protobuf::compiler::CodeGeneratorRequest *</code></td><td style="border-left-width: 0px"id="Arena::CreateMaybeMessage< protobuf::compiler::CodeGeneratorRequest >"><div style="padding-left: 16px; text-indent: -16px"><code><b>Arena::CreateMaybeMessage< protobuf::compiler::CodeGeneratorRequest ></b>(Arena * )</code></div></td></tr><tr><td style="border-right-width: 0px; text-align: right;"><code>protobuf::compiler::CodeGeneratorResponse *</code></td><td style="border-left-width: 0px"id="Arena::CreateMaybeMessage< protobuf::compiler::CodeGeneratorResponse >"><div style="padding-left: 16px; text-indent: -16px"><code><b>Arena::CreateMaybeMessage< protobuf::compiler::CodeGeneratorResponse ></b>(Arena * )</code></div></td></tr><tr><td style="border-right-width: 0px; text-align: right;"><code>protobuf::compiler::CodeGeneratorResponse_File *</code></td><td style="border-left-width: 0px"id="Arena::CreateMaybeMessage< protobuf::compiler::CodeGeneratorResponse_File >"><div style="padding-left: 16px; text-indent: -16px"><code><b>Arena::CreateMaybeMessage< protobuf::compiler::CodeGeneratorResponse_File ></b>(Arena * )</code></div></td></tr><tr><td style="border-right-width: 0px; text-align: right;"><code>protobuf::compiler::Version *</code></td><td style="border-left-width: 0px"id="Arena::CreateMaybeMessage< protobuf::compiler::Version >"><div style="padding-left: 16px; text-indent: -16px"><code><b>Arena::CreateMaybeMessage< protobuf::compiler::Version ></b>(Arena * )</code></div></td></tr><tr><td style="border-right-width: 0px; text-align: right;"><code>const EnumDescriptor *</code></td><td style="border-left-width: 0px"id="GetEnumDescriptor< protobuf::compiler::CodeGeneratorResponse_Feature >"><div style="padding-left: 16px; text-indent: -16px"><code><b>GetEnumDescriptor< protobuf::compiler::CodeGeneratorResponse_Feature ></b>()</code></div></td></tr></table>
```

I am ready to start the translation once you provide the Markdown content.
