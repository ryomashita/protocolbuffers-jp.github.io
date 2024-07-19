
+++
title = "plugin.pb.h"
toc_hide = "true"
linkTitle = "C++"
description = "このセクションには、C++でプロトコルバッファクラスを使用するためのリファレンスドキュメントが含まれています。"
type = "docs"
+++

<p><code>#include &lt;google/protobuf/compiler/plugin.pb.h&gt;<br>namespace <a href="#google.protobuf.compiler">google::protobuf::compiler</a></code></p><p>protocプラグイン用のAPI。</p><p>このファイルは、protocコードジェネレータプラグインへのAPIを構成するプロトコルメッセージクラスのセットを定義しています。C++で書かれたプラグインは、おそらくprotobufレベルのAPIではなく、<a href="google.protobuf.compiler.plugin">plugin.h</a>のAPIをベースに構築すべきですが、他の言語で書かれたプラグインは以下で定義された生のメッセージを扱う必要があります。</p><p>プロトコルコンパイラは現在、自動生成されたドキュメントをサポートしていないため、このページには説明が含まれていません。このファイルは、<code>plugin.proto</code>からプロトコルコンパイラによって生成されました。その内容は以下の通りです:</p>

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
// 標準入力からCodeGeneratorRequestを読み取り、標準出力にCodeGeneratorResponseを書き込むだけのプログラムです。
//
// C++を使用して書かれたプラグインは、ここで定義された生のプロトコルを扱う代わりに、google/protobuf/compiler/plugin.hを使用できます。
//
// プラグイン実行可能ファイルはパスのどこかに配置するだけで十分です。プラグインは "protoc-gen-$NAME" という名前にする必要があり、
// その後、protocにフラグ "--${NAME}_out" が渡されたときに使用されます。

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
  // アルファ、ベータ、またはリリース候補のサフィックス、例: "alpha-1", "rc2"。メインラインの安定リリースの場合は空であるべきです。
  optional string suffix = 4;
}

// エンコードされたCodeGeneratorRequestはプラグインの標準入力に書き込まれます。
message CodeGeneratorRequest {
  // コマンドラインで明示的にリストされた.protoファイル。コードジェネレータはこれらのファイルのみにコードを生成する必要があります。
  // 各ファイルのディスクリプタは、以下のproto_fileに含まれます。
  repeated string file_to_generate = 1;

  // コマンドラインで渡されたジェネレータパラメータ。
  optional string parameter = 2;

  // files_to_generateおよびそれらがインポートするすべてのファイルのFileDescriptorProtos。
  // ファイルはトポロジカル順序で表示されるため、各ファイルはそれをインポートするファイルよりも前に表示されます。
  //
  // protocは、proto_filesが上記のフィールドの後に書き込まれることを保証しますが、
  // これはprotobufワイヤーフォーマットでは技術的に保証されていないためです。
  // 理論的には、プラグインがFileDescriptorProtosをストリーミングして一度にすべてをメモリに読み込むのではなく、
  // 1つずつ処理することができる可能性があります。ただし、この執筆時点では、
  // protoc側で同様に最適化されていないため、protocはすべてのフィールドを一度にメモリに保存してからプラグインに送信します。
  //
  // FileDescriptorProto内のフィールドと拡張の型名は常に完全修飾名です。
  repeated FileDescriptorProto proto_file = 15;
}
```

```protobuf
  // Protocol Compilerのバージョン番号。
  optional Version compiler_version = 3;

}

// プラグインは、エンコードされたCodeGeneratorResponseを標準出力に書き込みます。
message CodeGeneratorResponse {
  // エラーメッセージ。空でない場合、コード生成が失敗しました。プラグインプロセス
  // は、この方法でエラーを報告しても、ステータスコードがゼロで終了する必要があります。
  //
  // これは、.protoファイルでのエラーを示すために使用するべきです。コードジェネレータが正しいコードを生成できないエラー。 protoc自体に問題があるエラー -- たとえば、入力CodeGeneratorRequestが
  // 解析不可能である -- stderrにメッセージを書き込んで報告し、
  // 非ゼロのステータスコードで終了する必要があります。
  optional string error = 1;

  // コードジェネレータがサポートするサポートされる機能のビットマスク。
  // これは、Feature enumからの値のビットごとの "or" です。
  optional uint64 supported_features = 2;

  // code_generator.hと同期します。
  enum Feature {
    FEATURE_NONE = 0;
    FEATURE_PROTO3_OPTIONAL = 1;
  }

  // 生成された単一のファイルを表します。
  message File {
    // 出力ディレクトリに対する相対的なファイル名。名前には
    // "."または".."のコンポーネントを含めてはいけません。相対的でなければなりません
    // 絶対ではない（つまり、ファイルは出力ディレクトリの外には存在できません）。パスセパレータとして
    // "/"を使用する必要があります、"\"ではありません。
    //
    // 名前が省略された場合、内容は前の
    // ファイルに追加されます。これにより、ジェネレータは大きなファイルを小さなチャンクに分割できます。
    // 生成されたテキストをストリームバックに戻すことができるため、大きな
    // ファイルは一度に完全にメモリに存在する必要はありません。この執筆時点では
    // protocはこれに最適化されていません -- CodeGeneratorResponse全体を読み込んでから
    // ファイルをディスクに書き込みます。
    optional string name = 1;

// 空でない場合、指定されたファイルがすでに存在し、ここに内容が挿入されるべきであることを示します
// 定義された挿入点。この機能を使用すると、コードジェネレータが出力を拡張できます
// 他のコードジェネレータによって生成された出力。元のジェネレータは提供できます
// 挿入ポイントをファイルに特別な注釈を配置することで
// 次のように見える:
//   @@protoc_insertion_point(NAME)
// 注釈には、それを使用する他のジェネレータが挿入ポイントとして使用する識別子を置き換える
// べきです。このポイントに挿入されたコードは、挿入ポイントを含む行の直前に配置されます
// （したがって、同じポイントに複数の挿入が追加された場合、追加された順に出力されます）。
// 二重@は、生成されたコードが挿入ポイントのように見えることがないようにすることを意図しています。
//
// たとえば、C++コードジェネレータは、次の行を
// 生成する.pb.hファイルに配置します:
//   // @@protoc_insertion_point(namespace_scope)
// この行は、ファイルのパッケージ名前空間のスコープ内に現れますが
// 特定のクラスの外側です。別のプラグインは、次のように指定できます
// 挿入ポイント "namespace_scope" は、このスコープに配置する追加のクラスまたは
// 他の宣言。
//
// 挿入ポイントを含む行が空白で始まる場合、挿入されたテキストのすべての行に同じ空白が追加されます。
// これは、Pythonなどの言語に適しています。挿入ポイントコメント
// 挿入されたコードがそのコンテキストで正しく機能するために必要な量だけインデントされている必要があります。
//
// 初期ファイルを生成するコードジェネレータと、それに挿入するコードジェネレータは、両方とも
// protocの単一の呼び出しの一部として実行する必要があります。
// コードジェネレータは、コマンドラインに表示される順序で実行されます。
//
// |insertion_point|が存在する場合、|name|も存在する必要があります。
optional string insertion_point = 2;
```  

```markdown
// ファイルの内容。
optional string content = 15;

// 挿入されるファイルの内容を説明する情報。挿入ポイントが使用されている場合、この情報は適切にオフセットされ、生成されたファイルのコード生成メタデータに挿入されます。
optional GeneratedCodeInfo generated_code_info = 16;

  }
  repeated File file = 15;
}
</pre><table width="100%"><tr><th colspan="2"><h3 style="margin-top: 4px">このファイル内のクラス</h3></th></tr></table><table><tr><th colspan="2"><h3 style="margin-top: 4px">ファイルメンバー</h3><div style="font-style: italic; font-weight: normal;">これらの定義はどのクラスにも属していません。</div></th></tr><tr><td style="border-right-width: 0px; text-align: right;"><code>const ::protobuf::internal::DescriptorTable</code></td><td style="border-left-width: 0px"id="descriptor_table_google_2fprotobuf_2fcompiler_2fplugin_2eproto"><div style="padding-left: 16px; text-indent: -16px"><code><b>descriptor_table_google_2fprotobuf_2fcompiler_2fplugin_2eproto</b></code></div></td></tr><tr><td style="border-right-width: 0px; text-align: right;"><code>PROTOBUF_NAMESPACE_CLOSE PROTOBUF_NAMESPACE_OPEN protobuf::compiler::CodeGeneratorRequest *</code></td><td style="border-left-width: 0px"id="Arena::CreateMaybeMessage< protobuf::compiler::CodeGeneratorRequest >"><div style="padding-left: 16px; text-indent: -16px"><code><b>Arena::CreateMaybeMessage< protobuf::compiler::CodeGeneratorRequest ></b>(Arena * )</code></div></td></tr><tr><td style="border-right-width: 0px; text-align: right;"><code>protobuf::compiler::CodeGeneratorResponse *</code></td><td style="border-left-width: 0px"id="Arena::CreateMaybeMessage< protobuf::compiler::CodeGeneratorResponse >"><div style="padding-left: 16px; text-indent: -16px"><code><b>Arena::CreateMaybeMessage< protobuf::compiler::CodeGeneratorResponse ></b>(Arena * )</code></div></td></tr><tr><td style="border-right-width: 0px; text-align: right;"><code>protobuf::compiler::CodeGeneratorResponse_File *</code></td><td style="border-left-width: 0px"id="Arena::CreateMaybeMessage< protobuf::compiler::CodeGeneratorResponse_File >"><div style="padding-left: 16px; text-indent: -16px"><code><b>Arena::CreateMaybeMessage< protobuf::compiler::CodeGeneratorResponse_File ></b>(Arena * )</code></div></td></tr><tr><td style="border-right-width: 0px; text-align: right;"><code>protobuf::compiler::Version *</code></td><td style="border-left-width: 0px"id="Arena::CreateMaybeMessage< protobuf::compiler::Version >"><div style="padding-left: 16px; text-indent: -16px"><code><b>Arena::CreateMaybeMessage< protobuf::compiler::Version ></b>(Arena * )</code></div></td></tr><tr><td style="border-right-width: 0px; text-align: right;"><code>const EnumDescriptor *</code></td><td style="border-left-width: 0px"id="GetEnumDescriptor< protobuf::compiler::CodeGeneratorResponse_Feature >"><div style="padding-left: 16px; text-indent: -16px"><code><b>GetEnumDescriptor< protobuf::compiler::CodeGeneratorResponse_Feature ></b>()</code></div></td></tr></table>
```  

Please provide the Markdown content that you would like me to translate into Japanese.
