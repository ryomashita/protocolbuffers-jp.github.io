+++
title = "java_names.h"
toc_hide = "true"
linkTitle = "C++"
description = "このセクションには、C++でプロトコルバッファクラスを操作するためのリファレンスドキュメントが含まれています。"
type = "docs"
+++

<p><code>#include &lt;google/protobuf/compiler/java/java_names.h&gt;<br>namespace <a href="#google.protobuf.compiler">google::protobuf::compiler::java</a></code></p><p>対応するJavaクラスの完全修飾名にディスクリプタをマッピングするメカニズムを提供します。</p><table width="100%"><tr><th colspan="2"><h3 style="margin-top: 4px">このファイル内のクラス</h3></th></tr></table><table><tr><th colspan="2"><h3 style="margin-top: 4px">ファイルメンバー</h3><div style="font-style: italic; font-weight: normal;">これらの定義はどのクラスにも属していません。</div></th></tr><tr><td style="border-right-width: 0px; text-align: right;"><code>std::string</code></td><td style="border-left-width: 0px"id="ClassName"><div style="padding-left: 16px; text-indent: -16px"><code><b>ClassName</b>(const <a href='google.protobuf.descriptor#Descriptor'>Descriptor</a> * descriptor)</code></div><div style="font-style: italic; margin-top: 4px; margin-left: 16px;">要件:  <a href="#ClassName.details">詳細...</a></div></td></tr><tr><td style="border-right-width: 0px; text-align: right;"><code>std::string</code></td><td style="border-left-width: 0px"id="ClassName"><div style="padding-left: 16px; text-indent: -16px"><code><b>ClassName</b>(const <a href='google.protobuf.descriptor#EnumDescriptor'>EnumDescriptor</a> * descriptor)</code></div><div style="font-style: italic; margin-top: 4px; margin-left: 16px;">要件:  <a href="#ClassName.details">詳細...</a></div></td></tr><tr><td style="border-right-width: 0px; text-align: right;"><code>std::string</code></td><td style="border-left-width: 0px"id="ClassName"><div style="padding-left: 16px; text-indent: -16px"><code><b>ClassName</b>(const <a href='google.protobuf.descriptor#FileDescriptor'>FileDescriptor</a> * descriptor)</code></div><div style="font-style: italic; margin-top: 4px; margin-left: 16px;">要件:  <a href="#ClassName.details">詳細...</a></div></td></tr><tr><td style="border-right-width: 0px; text-align: right;"><code>std::string</code></td><td style="border-left-width: 0px"id="ClassName"><div style="padding-left: 16px; text-indent: -16px"><code><b>ClassName</b>(const <a href='google.protobuf.descriptor#ServiceDescriptor'>ServiceDescriptor</a> * descriptor)</code></div><div style="font-style: italic; margin-top: 4px; margin-left: 16px;">要件:  <a href="#ClassName.details">詳細...</a></div></td></tr><tr><td style="border-right-width: 0px; text-align: right;"><code>std::string</code></td><td style="border-left-width: 0px"id="FileJavaPackage"><div style="padding-left: 16px; text-indent: -16px"><code><b>FileJavaPackage</b>(const <a href='google.protobuf.descriptor#FileDescriptor'>FileDescriptor</a> * descriptor)</code></div><div style="font-style: italic; margin-top: 4px; margin-left: 16px;">要件:  <a href="#FileJavaPackage.details">詳細...</a></div></td></tr><tr><td style="border-right-width: 0px; text-align: right;"><code>std::string</code></td><td style="border-left-width: 0px"id="CapitalizedFieldName"><div style="padding-left: 16px; text-indent: -16px"><code><b>CapitalizedFieldName</b>(const <a href='google.protobuf.descriptor#FieldDescriptor'>FieldDescriptor</a> * descriptor)</code></div><div style="font-style: italic; margin-top: 4px; margin-left: 16px;">要件:  <a href="#CapitalizedFieldName.details">詳細...</a></div></td></tr></table> <hr><h3 id="ClassName.details"><code>std::string java::ClassName(<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;const <a href='google.protobuf.descriptor#Descriptor'>Descriptor</a> * descriptor)</code></h3><div style="margin-left: 16px"><p>要件: </p><pre>descriptor != NULL</pre>
<p>戻り値: </p>
```

```markdown
<div> <hr><h3 id="ClassName.details"><code>std::string java::ClassName(<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;const <a href='google.protobuf.descriptor#EnumDescriptor'>EnumDescriptor</a> * descriptor)</code></h3><div style="margin-left: 16px"><p>Requires: </p><pre>descriptor != NULL</pre>

<p>Returns: </p>

<pre>完全修飾されたJavaクラス名。</pre>
</div> <hr><h3 id="ClassName.details"><code>std::string java::ClassName(<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;const <a href='google.protobuf.descriptor#FileDescriptor'>FileDescriptor</a> * descriptor)</code></h3><div style="margin-left: 16px"><p>Requires: </p><pre>descriptor != NULL</pre>

<p>Returns: </p>

<pre>完全修飾されたJavaクラス名。</pre>
</div> <hr><h3 id="ClassName.details"><code>std::string java::ClassName(<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;const <a href='google.protobuf.descriptor#ServiceDescriptor'>ServiceDescriptor</a> * descriptor)</code></h3><div style="margin-left: 16px"><p>Requires: </p><pre>descriptor != NULL</pre>

<p>Returns: </p>

<pre>完全修飾されたJavaクラス名。</pre>
</div> <hr><h3 id="FileJavaPackage.details"><code>std::string java::FileJavaPackage(<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;const <a href='google.protobuf.descriptor#FileDescriptor'>FileDescriptor</a> * descriptor)</code></h3><div style="margin-left: 16px"><p>Requires: </p><pre>descriptor != NULL</pre>

<p>Returns: </p>

<pre>Javaパッケージ名。</pre>
</div> <hr><h3 id="CapitalizedFieldName.details"><code>std::string java::CapitalizedFieldName(<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;const <a href='google.protobuf.descriptor#FieldDescriptor'>FieldDescriptor</a> * descriptor)</code></h3><div style="margin-left: 16px"><p>Requires: </p><pre>descriptor != NULL</pre>

<p> Returns: </p>

<pre>大文字で始まるキャメルケースのフィールド名。</pre>

</div>
```  
