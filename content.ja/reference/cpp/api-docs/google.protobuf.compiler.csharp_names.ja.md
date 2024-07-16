```markdown
+++
title = "csharp_names.h"
toc_hide = "true"
linkTitle = "C++"
description = "このセクションには、C++でプロトコルバッファクラスを操作するためのリファレンスドキュメントが含まれています。"
type = "docs"
+++

<p><code>#include &lt;google/protobuf/compiler/csharp/csharp_names.h&gt;<br>namespace <a href="#google.protobuf.compiler">google::protobuf::compiler::csharp</a></code></p><p>対応するC#クラスの完全修飾名にディスクリプタをマッピングするメカニズムを提供します。</p><table width="100%"><tr><th colspan="2"><h3 style="margin-top: 4px">このファイル内のクラス</h3></th></tr></table><table><tr><th colspan="2"><h3 style="margin-top: 4px">ファイルメンバー</h3><div style="font-style: italic; font-weight: normal;">これらの定義はどのクラスにも属していません。</div></th></tr><tr><td style="border-right-width: 0px; text-align: right;"><code>std::string</code></td><td style="border-left-width: 0px"id="GetFileNamespace"><div style="padding-left: 16px; text-indent: -16px"><code><b>GetFileNamespace</b>(const <a href='google.protobuf.descriptor#FileDescriptor'>FileDescriptor</a> * descriptor)</code></div><div style="font-style: italic; margin-top: 4px; margin-left: 16px;">要件:  <a href="#GetFileNamespace.details">詳細...</a></div></td></tr><tr><td style="border-right-width: 0px; text-align: right;"><code>std::string</code></td><td style="border-left-width: 0px"id="GetClassName"><div style="padding-left: 16px; text-indent: -16px"><code><b>GetClassName</b>(const <a href='google.protobuf.descriptor#Descriptor'>Descriptor</a> * descriptor)</code></div><div style="font-style: italic; margin-top: 4px; margin-left: 16px;">要件:  <a href="#GetClassName.details">詳細...</a></div></td></tr><tr><td style="border-right-width: 0px; text-align: right;"><code>std::string</code></td><td style="border-left-width: 0px"id="GetReflectionClassName"><div style="padding-left: 16px; text-indent: -16px"><code><b>GetReflectionClassName</b>(const <a href='google.protobuf.descriptor#FileDescriptor'>FileDescriptor</a> * descriptor)</code></div><div style="font-style: italic; margin-top: 4px; margin-left: 16px;">要件:  <a href="#GetReflectionClassName.details">詳細...</a></div></td></tr><tr><td style="border-right-width: 0px; text-align: right;"><code>std::string</code></td><td style="border-left-width: 0px"id="GetOutputFile"><div style="padding-left: 16px; text-indent: -16px"><code><b>GetOutputFile</b>(const <a href='google.protobuf.descriptor#FileDescriptor'>FileDescriptor</a> * descriptor, const std::string file_extension, const bool generate_directories, const std::string base_namespace, std::string * error)</code></div><div style="font-style: italic; margin-top: 4px; margin-left: 16px;">指定されたファイルディスクリプタの出力ファイル名を生成します。  <a href="#GetOutputFile.details">詳細...</a></div></td></tr></table> <hr><h3 id="GetFileNamespace.details"><code>std::string csharp::GetFileNamespace(<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;const <a href='google.protobuf.descriptor#FileDescriptor'>FileDescriptor</a> * descriptor)</code></h3><div style="margin-left: 16px"><p>要件: </p><pre>descriptor != NULL</pre>
<p>戻り値: </p>
```

<pre>指定されたファイル記述子に使用する名前空間。</pre>
</div> <hr><h3 id="GetClassName.details"><code>std::string csharp::GetClassName(<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;const <a href='google.protobuf.descriptor#Descriptor'>Descriptor</a> * descriptor)</code></h3><div style="margin-left: 16px"><p>必要条件: </p><pre>descriptor != NULL</pre>

<p>戻り値: </p>

<pre>完全修飾された C# クラス名。</pre>
</div> <hr><h3 id="GetReflectionClassName.details"><code>std::string csharp::GetReflectionClassName(<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;const <a href='google.protobuf.descriptor#FileDescriptor'>FileDescriptor</a> * descriptor)</code></h3><div style="margin-left: 16px"><p>必要条件: </p><pre>descriptor != NULL</pre>

<p>戻り値: </p>

<pre>ファイル記述子にアクセスするための C# クラスの完全修飾名。Proto コンパイラは、処理される各 .proto ファイルに対してこのようなクラスを生成します。</pre>
</div> <hr><h3 id="GetOutputFile.details"><code>std::string csharp::GetOutputFile(<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;const <a href='google.protobuf.descriptor#FileDescriptor'>FileDescriptor</a> * descriptor,<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;const std::string file_extension,<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;const bool generate_directories,<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;const std::string base_namespace,<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;std::string * error)</code></h3><div style="margin-left: 16px"><p>指定されたファイル記述子の出力ファイル名を生成します。</p><p>generate_directories が true の場合、出力ファイルはファイルの名前空間に対応するディレクトリに配置されます。base_namespace は、トップレベルのディレクトリの一部を削除するために使用できます。たとえば、名前空間が "Bar.Foo" のファイルで base_namespace が "Bar" の場合、結果として得られるファイルはディレクトリ "Foo" に配置されます（"Bar/Foo" ではありません）。</p>
<p>必要条件: </p>
<pre>descriptor != NULL
error != NULL</pre>

<p>戻り値: </p>

<pre>指定されたファイル記述子に対する出力ファイルとして使用するファイル名。失敗した場合、この関数は空の文字列を返し、エラー パラメータにエラーメッセージが含まれます。</pre>

</div>
