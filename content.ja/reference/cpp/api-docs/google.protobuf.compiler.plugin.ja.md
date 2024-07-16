```markdown
+++
title = "plugin.h"
toc_hide = "true"
linkTitle = "C++"
description = "このセクションには、C++でプロトコルバッファクラスを使用するためのリファレンスドキュメントが含まれています。"
type = "docs"
+++

<p><code>#include &lt;google/protobuf/compiler/plugin.h&gt;<br>namespace <a href="#google.protobuf.compiler">google::protobuf::compiler</a></code></p><p>C++で書かれたprotocコードジェネレータプラグインのフロントエンド。</p><p>C++でprotocプラグインを実装するには、単純に<a href='google.protobuf.compiler.code_generator#CodeGenerator'>CodeGenerator</a>の実装を書き、次に次のようなmain()関数を作成します：</p>

<pre>int main(int argc, char* argv[]) {
  MyCodeGenerator generator;
  return google::protobuf::compiler::PluginMain(argc, argv, &amp;generator);
}</pre>

<p>プラグインをlibprotobufとlibprotocにリンクする必要があります。</p>

<p>PluginMainの中核部分は、与えられた<a href='google.protobuf.compiler.code_generator#CodeGenerator'>CodeGenerator</a>をCodeGeneratorRequestに対して呼び出し、CodeGeneratorResponseを生成することです。この部分はGenerateCodeとして抽象化され、再利用できるようになっており、たとえば、入力のCodeGeneratorRequestに前処理を行い、そのリクエストを指定されたコードジェネレータに渡すPluginMainの変種を実装するために使用できます。</p>

<p>プラグインをprotocで使用するには、次のいずれかを行います：</p>

<ul>
  <li>プラグインバイナリをPATHのどこかに配置し、「protoc-gen-NAME」という名前を付けます（「NAME」をプラグインの名前に置き換えます）。その後、パラメータ「--NAME_out=OUT_DIR」を指定してprotocを起動すると、protocはプラグインを呼び出して出力を生成し、OUT_DIRに配置されます。</li>
  <li>
    <p>プラグインバイナリを任意の場所に配置し、任意の名前を付け、protocに「--plugin」パラメータを渡してプラグインに指示します：</p>
<pre>protoc --plugin=protoc-gen-NAME=path/to/mybinary --NAME_out=OUT_DIR</pre>
    <p>Windowsでは、.exe拡張子を含める必要があります：</p>
<pre>protoc --plugin=protoc-gen-NAME=path/to/mybinary.exe --NAME_out=OUT_DIR</pre>
  </li>
</ul>
```

<table width="100%"><tr><th colspan="2"><h3 style="margin-top: 4px">このファイル内のクラス</h3></th></tr></table><table><tr><th colspan="2"><h3 style="margin-top: 4px">ファイルメンバー</h3><div style="font-style: italic; font-weight: normal;">これらの定義はどのクラスの一部でもありません。</div></th></tr><tr><td style="border-right-width: 0px; text-align: right;"><code>int</code></td><td style="border-left-width: 0px"id="PluginMain"><div style="padding-left: 16px; text-indent: -16px"><code><b>PluginMain</b>(int argc, char * argv, const <a href='google.protobuf.compiler.code_generator#CodeGenerator'>CodeGenerator</a> * generator)</code></div><div style="font-style: italic; margin-top: 4px; margin-left: 16px;">指定されたコードジェネレータを公開するprotocプラグインのmain()を実装します。</div></td></tr><tr><td style="border-right-width: 0px; text-align: right;"><code>bool</code></td><td style="border-left-width: 0px"id="GenerateCode"><div style="padding-left: 16px; text-indent: -16px"><code><b>GenerateCode</b>(const CodeGeneratorRequest &amp; request, const <a href='google.protobuf.compiler.code_generator#CodeGenerator'>CodeGenerator</a> &amp; generator, CodeGeneratorResponse * response, std::string * error_msg)</code></div><div style="font-style: italic; margin-top: 4px; margin-left: 16px;">指定されたコードジェネレータを使用してコードを生成します。<a href="#GenerateCode.details">詳細...</a></div></td></tr></table> <hr><h3 id="GenerateCode.details"><code>bool compiler::GenerateCode(<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;const CodeGeneratorRequest &amp; request,<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;const <a href='google.protobuf.compiler.code_generator#CodeGenerator'>CodeGenerator</a> &amp; generator,<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;CodeGeneratorResponse * response,<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;std::string * error_msg)</code></h3><div style="margin-left: 16px"><p>指定されたコードジェネレータを使用してコードを生成します。</p><p>コード生成が成功した場合はtrueを返します。コード生成に失敗した場合、失敗原因を説明するためにerror_msgが記入される場合があります。</p>
</div>

I am ready to translate the Markdown content into Japanese. Please go ahead and paste the content you would like me to translate.
