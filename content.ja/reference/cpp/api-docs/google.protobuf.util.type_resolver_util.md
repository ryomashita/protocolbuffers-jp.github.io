
+++
title = "type_resolver_util.h"
toc_hide = "true"
linkTitle = "C++"
description = "このセクションには、C++でプロトコルバッファクラスを操作するためのリファレンスドキュメントが含まれています。"
type = "docs"
+++

<p><code>#include &lt;google/protobuf/util/type_resolver_util.h&gt;<br>namespace <a href="#google.protobuf.util">google::protobuf::util</a></code></p><p>TypeResolver用のユーティリティを定義します。</p><table width="100%"><tr><th colspan="2"><h3 style="margin-top: 4px">このファイル内のクラス</h3></th></tr></table><table><tr><th colspan="2"><h3 style="margin-top: 4px">ファイルメンバー</h3><div style="font-style: italic; font-weight: normal;">これらの定義はどのクラスにも属していません。</div></th></tr><tr><td style="border-right-width: 0px; text-align: right;"><code><a href='google.protobuf.util.type_resolver#TypeResolver'>TypeResolver</a> *</code></td><td style="border-left-width: 0px"id="NewTypeResolverForDescriptorPool"><div style="padding-left: 16px; text-indent: -16px"><code><b>NewTypeResolverForDescriptorPool</b>(const std::string &amp; url_prefix, const <a href='google.protobuf.descriptor#DescriptorPool'>DescriptorPool</a> * pool)</code></div><div style="font-style: italic; margin-top: 4px; margin-left: 16px;">指定された記述子プール内のタイプ情報を提供する<a href='google.protobuf.util.type_resolver#TypeResolver'>TypeResolver</a>を作成します。<a href="#NewTypeResolverForDescriptorPool.details">詳細...</a></div></td></tr></table> <hr><h3 id="NewTypeResolverForDescriptorPool.details"><code><a href='google.protobuf.util.type_resolver#TypeResolver'>TypeResolver</a> * util::NewTypeResolverForDescriptorPool(<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;const std::string &amp; url_prefix,<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;const <a href='google.protobuf.descriptor#DescriptorPool'>DescriptorPool</a> * pool)</code></h3><div style="margin-left: 16px"><p>指定された記述子プール内のタイプ情報を提供する<a href='google.protobuf.util.type_resolver#TypeResolver'>TypeResolver</a>を作成します。</p><p>呼び出し元は返された<a href='google.protobuf.util.type_resolver#TypeResolver'>TypeResolver</a>の所有権を取ります。</p>
```

</div>
