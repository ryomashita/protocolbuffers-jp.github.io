```markdown
+++
title = "ruby_generator.h"
toc_hide = "true"
linkTitle = "C++"
description = "このセクションには、C++でプロトコルバッファクラスを操作するためのリファレンスドキュメントが含まれています。"
type = "docs"
+++

<p><code>#include &lt;google/protobuf/compiler/ruby/ruby_generator.h&gt;<br>namespace <a href="#google.protobuf.compiler">google::protobuf::compiler::ruby</a></code></p><p>.protoファイルに対してRubyコードを生成します。</p><table width="100%"><tr><th colspan="2"><h3 style="margin-top: 4px">このファイル内のクラス</h3></th></tr><tr><td><div><code><a href="#Generator">Generator</a></code></div><div style="font-style: italic; margin-top: 4px; margin-left: 16px;"><a href='google.protobuf.compiler.code_generator#CodeGenerator'>生成されたRubyプロトコルバッファクラスのためのCodeGenerator</a>の実装。</div></td></tr></table><h2 id="Generator">class Generator: public <a href="google.protobuf.compiler.code_generator#CodeGenerator">CodeGenerator</a></h2><p><code>#include &lt;<a href="#">google/protobuf/compiler/ruby/ruby_generator.h</a>&gt;<br>namespace <a href="#google.protobuf.compiler">google::protobuf::compiler::ruby</a></code></p><p><a href='google.protobuf.compiler.code_generator#CodeGenerator'>生成されたRubyプロトコルバッファクラスのためのCodeGenerator</a>の実装。</p><p>独自のプロトコルコンパイラバイナリを作成し、それをRuby出力をサポートするようにしたい場合は、この<a href='google.protobuf.compiler.code_generator#CodeGenerator'>CodeGenerator</a>のインスタンスをmain()関数内の<a href='google.protobuf.compiler.command_line_interface#CommandLineInterface'>CommandLineInterface</a>に登録することで行うことができます。</p>

<table><tr><th colspan="2"><h3 style="margin-top: 4px">メンバー</h3></th></tr></table>
```
