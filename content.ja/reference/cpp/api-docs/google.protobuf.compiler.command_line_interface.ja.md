+++
title = "command_line_interface.h"
toc_hide = "true"
linkTitle = "C++"
description = "このセクションには、C++でプロトコルバッファクラスを操作するためのリファレンスドキュメントが含まれています。"
type = "docs"
+++

<p><code>#include &lt;google/protobuf/compiler/command_line_interface.h&gt;<br>namespace <a href="#google.protobuf.compiler">google::protobuf::compiler</a></code></p><p>プロトコルコンパイラのフロントエンドを実装し、他の言語をサポートするためにカスタムコンパイラで再利用できるようにします。</p><table width="100%"><tr><th colspan="2"><h3 style="margin-top: 4px">このファイル内のクラス</h3></th></tr><tr><td><div><code><a href="#CommandLineInterface">CommandLineInterface</a></code></div><div style="font-style: italic; margin-top: 4px; margin-left: 16px;">このクラスはプロトコルコンパイラへのコマンドラインインターフェースを実装します。</div></td></tr></table><h2 id="CommandLineInterface">class CommandLineInterface</h2><p><code>#include &lt;<a href="#">google/protobuf/compiler/command_line_interface.h</a>&gt;<br>namespace <a href="#google.protobuf.compiler">google::protobuf::compiler</a></code></p><p>このクラスはプロトコルコンパイラへのコマンドラインインターフェースを実装します。</p><p>このクラスは、選択した言語をサポートするカスタムプロトコルコンパイラを非常に簡単に作成できるように設計されています。たとえば、通常のC++サポートと独自の出力 "Foo" のサポートを含むカスタムプロトコルコンパイラバイナリを作成したい場合、<a href='google.protobuf.compiler.code_generator#CodeGenerator'>CodeGenerator</a> インターフェースを実装する "FooGenerator" クラスを作成し、次のように main() プロシージャを記述します:</p>

<pre>int main(int argc, char* argv[]) {
  google::protobuf::compiler::CommandLineInterface cli;

  // C++ソースとヘッダーの生成をサポートします。
  google::protobuf::compiler::cpp::CppGenerator cpp_generator;
  cli.RegisterGenerator("--cpp_out", &amp;cpp_generator,
    "C++ソースとヘッダーを生成します。");

  // Fooコードの生成をサポートします。
  FooGenerator foo_generator;
  cli.RegisterGenerator("--foo_out", &amp;foo_generator,
    "Fooファイルを生成します。");
```

```markdown
<pre>
  return cli.Run(argc, argv);
}</pre>

<p>コンパイラは次のような構文で呼び出されます:</p>

<pre>protoc --cpp_out=outdir --foo_out=outdir --proto_path=src src/foo.proto</pre>

<p>コンパイルする .proto ファイルは、コマンドラインで物理ファイルパスまたは &ndash;proto_path で指定されたディレクトリに対する仮想パスのいずれかで指定できます。たとえば、src/foo.proto の場合、次の2つの protoc 呼び出しは同じように機能します:</p>

<pre>1. protoc --proto_path=src src/foo.proto (物理ファイルパス)
2. protoc --proto_path=src foo.proto (src に対する仮想パス)</pre>

<p>ファイルパスが物理ファイルパスと相対仮想パスの両方として解釈できる場合、物理ファイルパスが優先されます。</p>

<p>コマンドライン構文の詳細については、&ndash;help を使用して呼び出してください。</p>

<table><tr><th colspan="2"><h3 style="margin-top: 4px">メンバー</h3></th></tr><tr><td style="border-right-width: 0px; text-align: right;"><code>const char *const</code></td><td style="border-left-width: 0px"id="CommandLineInterface.kPathSeparator"><div style="padding-left: 16px; text-indent: -16px"><code><b>kPathSeparator</b></code></div></td></tr><tr><td style="border-right-width: 0px; text-align: right;"><code></code></td><td style="border-left-width: 0px"id="CommandLineInterface.CommandLineInterface"><div style="padding-left: 16px; text-indent: -16px"><code><b>CommandLineInterface</b>()</code></div></td></tr><tr><td style="border-right-width: 0px; text-align: right;"><code></code></td><td style="border-left-width: 0px"id="CommandLineInterface.~CommandLineInterface"><div style="padding-left: 16px; text-indent: -16px"><code><b>~CommandLineInterface</b>()</code></div></td></tr><tr><td style="border-right-width: 0px; text-align: right;"><code>void</code></td><td style="border-left-width: 0px"id="CommandLineInterface.RegisterGenerator"><div style="padding-left: 16px; text-indent: -16px"><code><b>RegisterGenerator</b>(const std::string &amp; flag_name, <a href='google.protobuf.compiler.code_generator#CodeGenerator'>CodeGenerator</a> * generator, const std::string &amp; help_text)</code></div><div style="font-style: italic; margin-top: 4px; margin-left: 16px;">言語のためのコードジェネレータを登録します。  <a href="#CommandLineInterface.RegisterGenerator.details">詳細...</a></div></td></tr><tr><td style="border-right-width: 0px; text-align: right;"><code>void</code></td><td style="border-left-width: 0px"id="CommandLineInterface.RegisterGenerator"><div style="padding-left: 16px; text-indent: -16px"><code><b>RegisterGenerator</b>(const std::string &amp; flag_name, const std::string &amp; option_flag_name, <a href='google.protobuf.compiler.code_generator#CodeGenerator'>CodeGenerator</a> * generator, const std::string &amp; help_text)</code></div><div style="font-style: italic; margin-top: 4px; margin-left: 16px;">言語のためのコードジェネレータを登録します。  <a href="#CommandLineInterface.RegisterGenerator.details">詳細...</a></div></td></tr><tr><td style="border-right-width: 0px; text-align: right;"><code>void</code></td><td style="border-left-width: 0px"id="CommandLineInterface.AllowPlugins"><div style="padding-left: 16px; text-indent: -16px"><code><b>AllowPlugins</b>(const std::string &amp; exe_name_prefix)</code></div><div style="font-style: italic; margin-top: 4px; margin-left: 16px;">"プラグイン"を有効にします。  <a href="#CommandLineInterface.AllowPlugins.details">詳細...</a></div></td></tr><tr><td style="border-right-width: 0px; text-align: right;"><code>int</code></td><td style="border-left-width: 0px"id="CommandLineInterface.Run"><div style="padding-left: 16px; text-indent: -16px"><code><b>Run</b>(int argc, const char *const argv)</code></div><div style="font-style: italic; margin-top: 4px; margin-left: 16px;">指定されたコマンドラインパラメータで Protocol Compiler を実行します。  <a href="#CommandLineInterface.Run.details">詳細...</a></div></td></tr><tr><td style="border-right-width: 0px; text-align: right;"><code>void</code></td><td style="border-left-width: 0px"id="CommandLineInterface.SetInputsAreProtoPathRelative"><div style="padding-left: 16px; text-indent: -16px"><code><b>SetInputsAreProtoPathRelative</b>(bool )</code></div><div style="font-style: italic; margin-top: 4px; margin-left: 16px;">非推奨です。  <a href="#CommandLineInterface.SetInputsAreProtoPathRelative.details">詳細...</a></div></td></tr><tr><td style="border-right-width: 0px; text-align: right;"><code>void</code></td><td style="border-left-width: 0px"id="CommandLineInterface.SetVersionInfo"><div style="padding-left: 16px; text-indent: -16px"><code><b>SetVersionInfo</b>(const std::string &amp; text)</code></div><div style="font-style: italic; margin-top: 4px; margin-left: 16px;">&ndash;version フラグが使用されたときに表示されるテキストを提供します。  <a href="#CommandLineInterface.SetVersionInfo.details">詳細...</a></div></td></tr></table> <hr><h3 id="CommandLineInterface.RegisterGenerator.details"><code>void CommandLineInterface::RegisterGenerator(<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;const std::string &amp; flag_name,<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href='google.protobuf.compiler.code_generator#CodeGenerator'>CodeGenerator</a> * generator,<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;const std::string &amp; help_text)</code></h3><div style="margin-left: 16px"><p>言語のためのコードジェネレータを登録します。 </p><p>パラメータ:</p>
<ul>
  <li>flag_name: このタイプの出力ファイルを指定するために使用されるコマンドラインフラグ。名前は '-' で始まる必要があります。名前が1文字よりも長い場合、2つの '-' で始める必要があります。</li>
  <li>generator: このタイプのファイルを生成するために呼び出される <a href='google.protobuf.compiler.code_generator#CodeGenerator'>CodeGenerator</a>。</li>
  <li>help_text: &ndash;help 出力でこのフラグを説明するテキスト。</li>
</ul>
<p>一部のジェネレータは追加のパラメータを受け入れます。出力ディレクトリの前にコロンで区切ってコマンドラインでこのパラメータを指定できます: </p>
<pre>protoc --foo_out=enable_bar:outdir</pre>
<p>コロンの前のテキストは <a href='google.protobuf.compiler.code_generator#CodeGenerator.Generate'>CodeGenerator::Generate()</a> に "parameter" として渡されます。</p>
</div> <hr><h3 id="CommandLineInterface.RegisterGenerator.details"><code>void CommandLineInterface::RegisterGenerator(<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;const std::string &amp; flag_name,<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;const std::string &amp; option_flag_name,<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href='google.protobuf.compiler.code_generator#CodeGenerator'>CodeGenerator</a> * generator,<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;const std::string &amp; help_text)</code></h3><div style="margin-left: 16px"><p>言語のためのコードジェネレータを登録します。 </p><p>flag_name の他に、登録されたコードジェネレータに追加のパラメータを渡すために使用できる option_flag_name を指定できます。次のようにジェネレータを登録したとします:</p>
<pre>command_line_interface.RegisterGenerator("--foo_out", "--foo_opt", ...)</pre>
<p> その後、次のようなコマンドでコンパイラを呼び出すことができます:</p>
<pre>protoc --foo_out=enable_bar:outdir --foo_opt=enable_baz</pre>
<p> これにより、"enable_bar,enable_baz" がジェネレータにパラメータとして渡されます。</p>
</div> <hr><h3 id="CommandLineInterface.AllowPlugins.details"><code>void CommandLineInterface::AllowPlugins(<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;const std::string &amp; exe_name_prefix)</code></h3><div style="margin-left: 16px"><p>"プラグイン"を有効にします。 </p><p>このモードでは、コマンドラインフラグが "_out" で終わり、登録されたジェネレータと一致しない場合、コンパイラは "プラグイン" を見つけてジェネレータを実装しようとします。プラグインは単なる実行可能ファイルです。PATH のどこかに存在する必要があります。</p>
<p>コンパイラは、未認識のフラグ名に "_out" を削除したものを exe_name_prefix と連結して検索する実行可能ファイル名を決定します。たとえば、exe_name_prefix が "protoc-" で、フラグ &ndash;foo_out を渡した場合、コンパイラはプログラム "protoc-gen-foo" を実行しようとします。</p>
<p>プラグインプログラムは次の使用法を実装する必要があります:</p>
<pre>plugin [--out=OUTDIR] [--parameter=PARAMETER] PROTO_FILES &lt; DESCRIPTORS</pre>
<p>&ndash;out は出力ディレクトリを示し（&ndash;foo_out パラメータに渡される）、省略された場合は現在のディレクトリを使用する必要があります。&ndash;parameter は、提供された場合のジェネレータパラメータを示します（以下を参照）。PROTO_FILES はコンパイラコマンドラインで指定された .proto ファイルのリストであり、これらはプラグインが出力コードを生成することが期待されるファイルです。最後に、DESCRIPTORS はエンコードされた FileDescriptorSet（descriptor.proto で定義されている）です。これはプラグインの標準入力にパイプされます。セットには、PROTO_FILES にリストされているすべてのファイルとそれらがインポートするすべてのファイルの記述子が含まれます。プラグインは PROTO_FILES を直接読み取ろうとしてはいけません - FileDescriptorSet を使用する必要があります。</p>
<p>プラグインは通常のコードジェネレータと同様に必要なファイルを生成する必要があります。生成されたすべてのファイルの名前を stdout に書き込む必要があります。名前は出力ディレクトリを基準とした相対名である必要があり、絶対名や現在のディレクトリを基準とした名前ではありません。エラーが発生した場合は、エラーメッセージを stderr に書き込む必要があります。エラーが致命的な場合、プラグインはゼロ以外の終了コードで終了する必要があります。</p>
<p>プラグインは通常の組み込みジェネレータと同様にジェネレータパラメータを持つことができます。追加のジェネレータパラメータは一致する "_opt" パラメータを介して渡すことができます。たとえば:</p>
<pre>protoc --plug_out=enable_bar:outdir --plug_opt=enable_baz</pre>
<p> これにより、"enable_bar,enable_baz" がプラグインにパラメータとして渡されます。</p>
</div> <hr><h3 id="CommandLineInterface.Run.details"><code>int CommandLineInterface::Run(<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;int argc,<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;const char *const argv)</code></h3><div style="margin-left: 16px"><p>指定されたコマンドラインパラメータで Protocol Compiler を実行します。 </p><p>main() で返すべきエラーコードを返します。</p>
<p><a href='#CommandLineInterface.Run'>Run()</a> をマルチスレッド環境で呼び出すことは安全ではないかもしれません。なぜそうしたいのかはわかりませんが。</p>
</div> <hr><h3 id="CommandLineInterface.SetInputsAreProtoPathRelative.details"><code>void CommandLineInterface::SetInputsAreProtoPathRelative(<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;bool )</code></h3><div style="margin-left: 16px"><p>非推奨です。 </p><p>このメソッドを呼び出しても効果はありません。プロトコルコンパイラは現在のディレクトリに対して .proto ファイルを最初に検索し、ファイルが見つからない場合は入力パスを仮想パスとして扱います。</p>
</div> <hr><h3 id="CommandLineInterface.SetVersionInfo.details"><code>void CommandLineInterface::SetVersionInfo(<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;const std::string &amp; text)</code></h3><div style="margin-left: 16px"><p>&ndash;version フラグが使用されたときに表示されるテキストを提供します。 </p><p>libprotoc のバージョンも、このテキストの次の行に表示されます。</p>
</div>
```

Please provide the Markdown content you would like me to translate into Japanese.
