+++
title = "command_line_interface.h"
toc_hide = "true"
linkTitle = "C++"
description = "このセクションには、C++でプロトコルバッファクラスを使用するためのリファレンスドキュメントが含まれています。"
type = "docs"
+++

<p><code>#include &lt;google/protobuf/compiler/command_line_interface.h&gt;<br>namespace <a href="#google.protobuf.compiler">google::protobuf::compiler</a></code></p><p>このファイルは、プロトコルコンパイラのフロントエンドを実装し、他の言語をサポートするためにカスタムコンパイラで再利用できるようにします。</p><table width="100%"><tr><th colspan="2"><h3 style="margin-top: 4px">このファイル内のクラス</h3></th></tr><tr><td><div><code><a href="#CommandLineInterface">CommandLineInterface</a></code></div><div style="font-style: italic; margin-top: 4px; margin-left: 16px;">このクラスは、プロトコルコンパイラへのコマンドラインインターフェースを実装します。</div></td></tr></table><h2 id="CommandLineInterface">class CommandLineInterface</h2><p><code>#include &lt;<a href="#">google/protobuf/compiler/command_line_interface.h</a>&gt;<br>namespace <a href="#google.protobuf.compiler">google::protobuf::compiler</a></code></p><p>このクラスは、プロトコルコンパイラへのコマンドラインインターフェースを実装します。</p><p>このクラスは、選択した言語をサポートするカスタムプロトコルコンパイラを非常に簡単に作成できるように設計されています。たとえば、通常のC++サポートと独自の出力 "Foo" のサポートを含むカスタムプロトコルコンパイラバイナリを作成したい場合、<a href='google.protobuf.compiler.code_generator#CodeGenerator'>CodeGenerator</a> インターフェースを実装する "FooGenerator" クラスを作成し、次のように main() プロシージャを記述します:</p>

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
  </pre>

```cpp
  return cli.Run(argc, argv);
```

<p>コンパイラは次のような構文で呼び出されます:</p>

```bash
protoc --cpp_out=outdir --foo_out=outdir --proto_path=src src/foo.proto
```

<p>コンパイルする.protoファイルは、コマンドラインで物理ファイルパスまたは&ndash;proto_pathで指定されたディレクトリに対する仮想パスのいずれかを使用して指定できます。たとえば、src/foo.protoの場合、次の2つのprotoc呼び出しは同じように機能します:</p>

```bash
1. protoc --proto_path=src src/foo.proto (物理ファイルパス)
2. protoc --proto_path=src foo.proto (srcに対する仮想パス)
```

<p>ファイルパスが物理ファイルパスと相対仮想パスの両方として解釈できる場合、物理ファイルパスが優先されます。</p>

<p>コマンドライン構文の詳細については、&ndash;helpを使用して呼び出してください。</p>

<table><tr><th colspan="2"><h3 style="margin-top: 4px">メンバー</h3></th></tr><tr><td style="border-right-width: 0px; text-align: right;"><code>const char *const</code></td><td style="border-left-width: 0px"id="CommandLineInterface.kPathSeparator"><div style="padding-left: 16px; text-indent: -16px"><code><b>kPathSeparator</b></code></div></td></tr><tr><td style="border-right-width: 0px; text-align: right;"><code></code></td><td style="border-left-width: 0px"id="CommandLineInterface.CommandLineInterface"><div style="padding-left: 16px; text-indent: -16px"><code><b>CommandLineInterface</b>()</code></div></td></tr><tr><td style="border-right-width: 0px; text-align: right;"><code></code></td><td style="border-left-width: 0px"id="CommandLineInterface.~CommandLineInterface"><div style="padding-left: 16px; text-indent: -16px"><code><b>~CommandLineInterface</b>()</code></div></td></tr><tr><td style="border-right-width: 0px; text-align: right;"><code>void</code></td><td style="border-left-width: 0px"id="CommandLineInterface.RegisterGenerator"><div style="padding-left: 16px; text-indent: -16px"><code><b>RegisterGenerator</b>(const std::string &amp; flag_name, <a href='google.protobuf.compiler.code_generator#CodeGenerator'>CodeGenerator</a> * generator, const std::string &amp; help_text)</code></div><div style="font-style: italic; margin-top: 4px; margin-left: 16px;">言語のためのコードジェネレータを登録します。  <a href="#CommandLineInterface.RegisterGenerator.details">詳細...</a></div></td></tr><tr><td style="border-right-width: 0px; text-align: right;"><code>void</code></td><td style="border-left-width: 0px"id="CommandLineInterface.RegisterGenerator"><div style="padding-left: 16px; text-indent: -16px"><code><b>RegisterGenerator</b>(const std::string &amp; flag_name, const std::string &amp; option_flag_name, <a href='google.protobuf.compiler.code_generator#CodeGenerator'>CodeGenerator</a> * generator, const std::string &amp; help_text)</code></div><div style="font-style: italic; margin-top: 4px; margin-left: 16px;">言語のためのコードジェネレータを登録します。  <a href="#CommandLineInterface.RegisterGenerator.details">詳細...</a></div></td></tr><tr><td style="border-right-width: 0px; text-align: right;"><code>void</code></td><td style="border-left-width: 0px"id="CommandLineInterface.AllowPlugins"><div style="padding-left: 16px; text-indent: -16px"><code><b>AllowPlugins</b>(const std::string &amp; exe_name_prefix)</code></div><div style="font-style: italic; margin-top: 4px; margin-left: 16px;">"プラグイン"を有効にします。  <a href="#CommandLineInterface.AllowPlugins.details">詳細...</a></div></td></tr><tr><td style="border-right-width: 0px; text-align: right;"><code>int</code></td><td style="border-left-width: 0px"id="CommandLineInterface.Run"><div style="padding-left: 16px; text-indent: -16px"><code><b>Run</b>(int argc, const char *const argv)</code></div><div style="font-style: italic; margin-top: 4px; margin-left: 16px;">指定されたコマンドラインパラメータでプロトコルコンパイラを実行します。  <a href="#CommandLineInterface.Run.details">詳細...</a></div></td></tr><tr><td style="border-right-width: 0px; text-align: right;"><code>void</code></td><td style="border-left-width: 0px"id="CommandLineInterface.SetInputsAreProtoPathRelative"><div style="padding-left: 16px; text-indent: -16px"><code><b>SetInputsAreProtoPathRelative</b>(bool )</code></div><div style="font-style: italic; margin-top: 4px; margin-left: 16px;">非推奨です。  <a href="#CommandLineInterface.SetInputsAreProtoPathRelative.details">詳細...</a></div></td></tr><tr><td style="border-right-width: 0px; text-align: right;"><code>void</code></td><td style="border-left-width: 0px"id="CommandLineInterface.SetVersionInfo"><div style="padding-left: 16px; text-indent: -16px"><code><b>SetVersionInfo</b>(const std::string &amp; text)</code></div><div style="font-style: italic; margin-top: 4px; margin-left: 16px;">&ndash;versionフラグが使用されたときに印刷されるテキストを提供します。  <a href="#CommandLineInterface.SetVersionInfo.details">詳細...</a></div></td></tr></table> <hr><h3 id="CommandLineInterface.RegisterGenerator.details"><code>void CommandLineInterface::RegisterGenerator(<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;const std::string &amp; flag_name,<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href='google.protobuf.compiler.code_generator#CodeGenerator'>CodeGenerator</a> * generator,<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;const std::string &amp; help_text)</code></h3><div style="margin-left: 16px"><p>言語のためのコードジェネレータを登録します。 </p><p>パラメータ:</p>
<ul>
  <li>flag_name: このタイプの出力ファイルを指定するために使用されるコマンドラインフラグ。名前は'-'で始まる必要があります。名前が1文字以上の場合、2つの'-'で始まる必要があります。</li>
  <li>generator: このタイプのファイルを生成するために呼び出される<a href='google.protobuf.compiler.code_generator#CodeGenerator'>CodeGenerator</a>。</li>
  <li>help_text: &ndash;help出力でこのフラグを説明するテキスト。</li>
</ul>
<p>一部のジェネレータは追加のパラメータを受け入れます。出力ディレクトリの前にコロンで区切ってコマンドラインでこのパラメータを指定できます:</p>
<pre>protoc --foo_out=enable_bar:outdir</pre>
<p>コロンの前のテキストは<a href='google.protobuf.compiler.code_generator#CodeGenerator.Generate'>CodeGenerator::Generate()</a>に"パラメータ"として渡されます。</p>
</div> <hr><h3 id="CommandLineInterface.RegisterGenerator.details"><code>void CommandLineInterface::RegisterGenerator(<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;const std::string &amp; flag_name,<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;const std::string &amp; option_flag_name,<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href='google.protobuf.compiler.code_generator#CodeGenerator'>CodeGenerator</a> * generator,<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;const std::string &amp; help_text)</code></h3><div style="margin-left: 16px"><p>言語のためのコードジェネレータを登録します。 </p><p>flag_nameに加えて、登録されたコードジェネレータに追加のパラメータを渡すために使用できるoption_flag_nameを指定できます。次のようにジェネレータを登録したとします:</p>
<pre>command_line_interface.RegisterGenerator("--foo_out", "--foo_opt", ...)</pre>
<p> その後、次のようなコマンドでコンパイラを呼び出すことができます:</p>
<pre>protoc --foo_out=enable_bar:outdir --foo_opt=enable_baz</pre>
<p> これにより、"enable_bar,enable_baz"がジェネレータにパラメータとして渡されます。</p>
</div> <hr><h3 id="CommandLineInterface.AllowPlugins.details"><code>void CommandLineInterface::AllowPlugins(<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;const std::string &amp; exe_name_prefix)</code></h3><div style="margin-left: 16px"><p>"プラグイン"を有効にします。 </p><p>このモードでは、コマンドラインフラグが"_out"で終わり、登録されたジェネレータと一致しない場合、コンパイラは"プラグイン"を見つけてジェネレータを実装しようとします。プラグインは単なる実行可能ファイルです。PATHのどこかに存在する必要があります。</p>
<p>コンパイラは、exe_name_prefixと認識されないフラグ名を連結し、"_out"を削除して検索する実行可能ファイル名を決定します。たとえば、exe_name_prefixが"protoc-"で、フラグ&ndash;foo_outを渡すと、コンパイラはプログラム"protoc-gen-foo"を実行しようとします。</p>
<p>プラグインプログラムは次のような使用法を実装する必要があります:</p>
<pre>plugin [--out=OUTDIR] [--parameter=PARAMETER] PROTO_FILES &lt; DESCRIPTORS</pre>
<p>&ndash;outは出力ディレクトリを示し（&ndash;foo_outパラメータに渡される）、省略された場合は現在のディレクトリを使用する必要があります。&ndash;parameterは、提供された場合のジェネレータパラメータを指定します（以下を参照）。PROTO_FILESはコンパイラコマンドラインで指定された.protoファイルのリストであり、これらはプラグインが出力コードを生成することが期待されるファイルです。最後に、DESCRIPTORSはエンコードされたFileDescriptorSet（descriptor.protoで定義）です。これはプラグインのstdinにパイプされます。セットには、PROTO_FILESにリストされているすべてのファイルとそれらがインポートするすべてのファイルの記述子が含まれます。プラグインはPROTO_FILESを直接読み取ろうとしてはいけません。FileDescriptorSetを使用する必要があります。</p>
<p>プラグインは通常のビルトインジェネレータと同様に、必要なファイルを生成する必要があります。生成されたすべてのファイルの名前をstdoutに書き込む必要があります。名前は出力ディレクトリを基準として、絶対名または現在のディレクトリを基準としてではなく、相対名である必要があります。エラーが発生した場合は、エラーメッセージをstderrに書き込む必要があります。エラーが致命的な場合、プラグインはゼロ以外の終了コードで終了する必要があります。</p>
<p>プラグインは通常のビルトインジェネレータと同様に、ジェネレータパラメータを持つことができます。追加のジェネレータパラメータは一致する"_opt"パラメータを介して渡すことができます。たとえば:</p>
<pre>protoc --plug_out=enable_bar:outdir --plug_opt=enable_baz</pre>
<p> これにより、"enable_bar,enable_baz"がプラグインにパラメータとして渡されます。</p>
</div> <hr><h3 id="CommandLineInterface.Run.details"><code>int CommandLineInterface::Run(<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;int argc,<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;const char *const argv)</code></h3><div style="margin-left: 16px"><p>指定されたコマンドラインパラメータでプロトコルコンパイラを実行します。 </p><p>main()で返すべきエラーコードを返します。</p>
<p><a href='#CommandLineInterface.Run'>Run()</a>をマルチスレッド環境で呼び出すことは安全ではないかもしれません。なぜそうしたいのかはわかりません。</p>
</div> <hr><h3 id="CommandLineInterface.SetInputsAreProtoPathRelative.details"><code>void CommandLineInterface::SetInputsAreProtoPathRelative(<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;bool )</code></h3><div style="margin-left: 16px"><p>非推奨です。 </p><p>このメソッドを呼び出しても効果はありません。プロトコルコンパイラは現在のディレクトリに対して.protoファイルを最初に検索し、ファイルが見つからない場合は入力パスを仮想パスとして扱います。</p>
</div> <hr><h3 id="CommandLineInterface.SetVersionInfo.details"><code>void CommandLineInterface::SetVersionInfo(<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;const std::string &amp; text)</code></h3><div style="margin-left: 16px"><p>&ndash;versionフラグが使用されたときに印刷されるテキストを提供します。 </p><p>libprotocのバージョンも、このテキストの次の行に印刷されます。</p>
</div>

I am ready to start the translation once you provide me with the Markdown content.
