```markdown
+++
title = "type_resolver.h"
toc_hide = "true"
linkTitle = "C++"
description = "このセクションには、C++でプロトコルバッファクラスを使用するためのリファレンスドキュメントが含まれています。"
type = "docs"
+++

<p><code>#include &lt;google/protobuf/util/type_resolver.h&gt;<br>namespace <a href="#google.protobuf.util">google::protobuf::util</a></code></p><p>Any メッセージ用の TypeResolver を定義します。</p><table width="100%"><tr><th colspan="2"><h3 style="margin-top: 4px">このファイル内のクラス</h3></th></tr><tr><td><div><code><a href="#TypeResolver">TypeResolver</a></code></div><div style="font-style: italic; margin-top: 4px; margin-left: 16px;">タイプリゾルバーのための抽象インターフェース。</div></td></tr></table><h2 id="TypeResolver">class TypeResolver</h2><p><code>#include &lt;<a href="#">google/protobuf/util/type_resolver.h</a>&gt;<br>namespace <a href="#google.protobuf.util">google::protobuf::util</a></code></p><p>タイプリゾルバーのための抽象インターフェース。</p><p>このインターフェースの実装はスレッドセーフである必要があります。</p>

<table><tr><th colspan="2"><h3 style="margin-top: 4px">メンバー</h3></th></tr><tr><td style="border-right-width: 0px; text-align: right;"><code></code></td><td style="border-left-width: 0px"id="TypeResolver.TypeResolver"><div style="padding-left: 16px; text-indent: -16px"><code><b>TypeResolver</b>()</code></div></td></tr><tr><td style="border-right-width: 0px; text-align: right;"><code>virtual </code></td><td style="border-left-width: 0px"id="TypeResolver.~TypeResolver"><div style="padding-left: 16px; text-indent: -16px"><code><b>~TypeResolver</b>()</code></div></td></tr><tr><td style="border-right-width: 0px; text-align: right;"><code>virtual util::Status</code></td><td style="border-left-width: 0px"id="TypeResolver.ResolveMessageType"><div style="padding-left: 16px; text-indent: -16px"><code><b>ResolveMessageType</b>(const std::string &amp; type_url, google::protobuf::Type * message_type)  = 0</code></div><div style="font-style: italic; margin-top: 4px; margin-left: 16px;">メッセージタイプのためのタイプURLを解決します。</div></td></tr><tr><td style="border-right-width: 0px; text-align: right;"><code>virtual util::Status</code></td><td style="border-left-width: 0px"id="TypeResolver.ResolveEnumType"><div style="padding-left: 16px; text-indent: -16px"><code><b>ResolveEnumType</b>(const std::string &amp; type_url, google::protobuf::Enum * enum_type)  = 0</code></div><div style="font-style: italic; margin-top: 4px; margin-left: 16px;">列挙型のためのタイプURLを解決します。</div></td></tr></table>
```

Please paste the Markdown content that you would like me to translate into Japanese.
