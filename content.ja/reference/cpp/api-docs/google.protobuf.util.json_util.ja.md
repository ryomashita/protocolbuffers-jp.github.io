```markdown
+++
title = "json_util.h"
toc_hide = "true"
linkTitle = "C++"
description = "このセクションには、C++でプロトコルバッファクラスを操作するためのリファレンスドキュメントが含まれています。"
type = "docs"
+++

<p><code>#include &lt;google/protobuf/util/json_util.h&gt;<br>namespace <a href="#google.protobuf.util">google::protobuf::util</a></code></p><p>protobufのバイナリ形式とproto3 JSON形式の間で変換するためのユーティリティ関数。</p><table width="100%"><tr><th colspan="2"><h3 style="margin-top: 4px">このファイルに含まれるクラス</h3></th></tr><tr><td><div><code><a href="#JsonParseOptions">JsonParseOptions</a></code></div><div style="font-style: italic; margin-top: 4px; margin-left: 16px;"></div></td></tr><tr><td><div><code><a href="#JsonPrintOptions">JsonPrintOptions</a></code></div><div style="font-style: italic; margin-top: 4px; margin-left: 16px;"></div></td></tr></table><table><tr><th colspan="2"><h3 style="margin-top: 4px">ファイルメンバー</h3><div style="font-style: italic; font-weight: normal;">これらの定義はどのクラスにも属していません。</div></th></tr><tr><td style="border-right-width: 0px; text-align: right;"><code>typedef</code></td><td style="border-left-width: 0px"id="JsonOptions"><div style="padding-left: 16px; text-indent: -16px"><code><a href='#JsonPrintOptions'>JsonPrintOptions</a> <b>JsonOptions</b></code></div><div style="font-style: italic; margin-top: 4px; margin-left: 16px;">非推奨。代わりに<a href='#JsonPrintOptions'>JsonPrintOptions</a>を使用してください。</div></td></tr><tr><td style="border-right-width: 0px; text-align: right;"><code>util::Status</code></td><td style="border-left-width: 0px"id="MessageToJsonString"><div style="padding-left: 16px; text-indent: -16px"><code><b>MessageToJsonString</b>(const <a href='google.protobuf.message#Message'>Message</a> &amp; message, std::string * output, const <a href='#JsonOptions'>JsonOptions</a> &amp; options)</code></div><div style="font-style: italic; margin-top: 4px; margin-left: 16px;">protobufメッセージからJSONに変換し、それを|output|に追加します。<a href="#MessageToJsonString.details">詳細...</a></div></td></tr><tr><td style="border-right-width: 0px; text-align: right;"><code>util::Status</code></td><td style="border-left-width: 0px"id="MessageToJsonString"><div style="padding-left: 16px; text-indent: -16px"><code><b>MessageToJsonString</b>(const <a href='google.protobuf.message#Message'>Message</a> &amp; message, std::string * output)</code></div></td></tr><tr><td style="border-right-width: 0px; text-align: right;"><code>util::Status</code></td><td style="border-left-width: 0px"id="JsonStringToMessage"><div style="padding-left: 16px; text-indent: -16px"><code><b>JsonStringToMessage</b>(StringPiece input, <a href='google.protobuf.message#Message'>Message</a> * message, const <a href='#JsonParseOptions'>JsonParseOptions</a> &amp; options)</code></div><div style="font-style: italic; margin-top: 4px; margin-left: 16px;">JSONからprotobufメッセージに変換します。<a href="#JsonStringToMessage.details">詳細...</a></div></td></tr><tr><td style="border-right-width: 0px; text-align: right;"><code>util::Status</code></td><td style="border-left-width: 0px"id="JsonStringToMessage"><div style="padding-left: 16px; text-indent: -16px"><code><b>JsonStringToMessage</b>(StringPiece input, <a href='google.protobuf.message#Message'>Message</a> * message)</code></div></td></tr><tr><td style="border-right-width: 0px; text-align: right;"><code>util::Status</code></td><td style="border-left-width: 0px"id="BinaryToJsonStream"><div style="padding-left: 16px; text-indent: -16px"><code><b>BinaryToJsonStream</b>(<a href='google.protobuf.util.type_resolver#TypeResolver'>TypeResolver</a> * resolver, const std::string &amp; type_url, <a href='google.protobuf.io.zero_copy_stream#ZeroCopyInputStream'>io::ZeroCopyInputStream</a> * binary_input, <a href='google.protobuf.io.zero_copy_stream#ZeroCopyOutputStream'>io::ZeroCopyOutputStream</a> * json_output, const <a href='#JsonPrintOptions'>JsonPrintOptions</a> &amp; options)</code></div><div style="font-style: italic; margin-top: 4px; margin-left: 16px;">protobufバイナリデータをJSONに変換します。<a href="#BinaryToJsonStream.details">詳細...</a></div></td></tr><tr><td style="border-right-width: 0px; text-align: right;"><code>util::Status</code></td><td style="border-left-width: 0px"id="BinaryToJsonStream"><div style="padding-left: 16px; text-indent: -16px"><code><b>BinaryToJsonStream</b>(<a href='google.protobuf.util.type_resolver#TypeResolver'>TypeResolver</a> * resolver, const std::string &amp; type_url, <a href='google.protobuf.io.zero_copy_stream#ZeroCopyInputStream'>io::ZeroCopyInputStream</a> * binary_input, <a href='google.protobuf.io.zero_copy_stream#ZeroCopyOutputStream'>io::ZeroCopyOutputStream</a> * json_output)</code></div></td></tr><tr><td style="border-right-width: 0px; text-align: right;"><code>util::Status</code></td><td style="border-left-width: 0px"id="BinaryToJsonString"><div style="padding-left: 16px; text-indent: -16px"><code><b>BinaryToJsonString</b>(<a href='google.protobuf.util.type_resolver#TypeResolver'>TypeResolver</a> * resolver, const std::string &amp; type_url, const std::string &amp; binary_input, std::string * json_output, const <a href='#JsonPrintOptions'>JsonPrintOptions</a> &amp; options)</code></div></td></tr><tr><td style="border-right-width: 0px; text-align: right;"><code>util::Status</code></td><td style="border-left-width: 0px"id="BinaryToJsonString"><div style="padding-left: 16px; text-indent: -16px"><code><b>BinaryToJsonString</b>(<a href='google.protobuf.util.type_resolver#TypeResolver'>TypeResolver</a> * resolver, const std::string &amp; type_url, const std::string &amp; binary_input, std::string * json_output)</code></div></td></tr><tr><td style="border-right-width: 0px; text-align: right;"><code>util::Status</code></td><td style="border-left-width: 0px"id="JsonToBinaryStream"><div style="padding-left: 16px; text-indent: -16px"><code><b>JsonToBinaryStream</b>(<a href='google.protobuf.util.type_resolver#TypeResolver'>TypeResolver</a> * resolver, const std::string &amp; type_url, <a href='google.protobuf.io.zero_copy_stream#ZeroCopyInputStream'>io::ZeroCopyInputStream</a> * json_input, <a href='google.protobuf.io.zero_copy_stream#ZeroCopyOutputStream'>io::ZeroCopyOutputStream</a> * binary_output, const <a href='#JsonParseOptions'>JsonParseOptions</a> &amp; options)</code></div><div style="font-style: italic; margin-top: 4px; margin-left: 16px;">JSONデータをprotobufバイナリ形式に変換します。<a href="#JsonToBinaryStream.details">詳細...</a></div></td></tr><tr><td style="border-right-width: 0px; text-align: right;"><code>util::Status</code></td><td style="border-left-width: 0px"id="JsonToBinaryStream"><div style="padding-left: 16px; text-indent: -16px"><code><b>JsonToBinaryStream</b>(<a href='google.protobuf.util.type_resolver#TypeResolver'>TypeResolver</a> * resolver, const std::string &amp; type_url, <a href='google.protobuf.io.zero_copy_stream#ZeroCopyInputStream'>io::ZeroCopyInputStream</a> * json_input, <a href='google.protobuf.io.zero_copy_stream#ZeroCopyOutputStream'>io::ZeroCopyOutputStream</a> * binary_output)</code></div></td></tr><tr><td style="border-right-width: 0px; text-align: right;"><code>util::Status</code></td><td style="border-left-width: 0px"id="JsonToBinaryString"><div style="padding-left: 16px; text-indent: -16px"><code><b>JsonToBinaryString</b>(<a href='google.protobuf.util.type_resolver#TypeResolver'>TypeResolver</a> * resolver, const std::string &amp; type_url, StringPiece json_input, std::string * binary_output, const <a href='#JsonParseOptions'>JsonParseOptions</a> &amp; options)</code></div></td></tr><tr><td style="border-right-width: 0px; text-align: right;"><code>util::Status</code></td><td style="border-left-width: 0px"id="JsonToBinaryString"><div style="padding-left: 16px; text-indent: -16px"><code><b>JsonToBinaryString</b>(<a href='google.protobuf.util.type_resolver#TypeResolver'>TypeResolver</a> * resolver, const std::string &amp; type_url, StringPiece json_input, std::string * binary_output)</code></div></td></tr></table> <hr><h3 id="MessageToJsonString.details"><code>util::Status util::MessageToJsonString(<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;const <a href='google.protobuf.message#Message'>Message</a> &amp; message,<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;std::string * output,<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;const <a href='#JsonOptions'>JsonOptions</a> &amp; options)</code></h3><div style="margin-left: 16px"><p>protobufメッセージからJSONに変換し、それを|output|に追加します。</p><p>これはBinaryToJsonString()の単純なラッパーです。渡されたメッセージの<a href='google.protobuf.descriptor#DescriptorPool'>DescriptorPool</a>を使用してAnyタイプを解決します。</p>
</div> <hr><h3 id="JsonStringToMessage.details"><code>util::Status util::JsonStringToMessage(<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;StringPiece input,<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href='google.protobuf.message#Message'>Message</a> * message,<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;const <a href='#JsonParseOptions'>JsonParseOptions</a> &amp; options)</code></h3><div style="margin-left: 16px"><p>JSONからprotobufメッセージに変換します。</p><p>これはJsonStringToBinary()の単純なラッパーです。渡されたメッセージの<a href='google.protobuf.descriptor#DescriptorPool'>DescriptorPool</a>を使用してAnyタイプを解決します。</p>
</div> <hr><h3 id="BinaryToJsonStream.details"><code>util::Status util::BinaryToJsonStream(<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href='google.protobuf.util.type_resolver#TypeResolver'>TypeResolver</a> * resolver,<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;const std::string &amp; type_url,<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href='google.protobuf.io.zero_copy_stream#ZeroCopyInputStream'>io::ZeroCopyInputStream</a> * binary_input,<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href='google.protobuf.io.zero_copy_stream#ZeroCopyOutputStream'>io::ZeroCopyOutputStream</a> * json_output,<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;const <a href='#JsonPrintOptions'>JsonPrintOptions</a> &amp; options)</code></h3><div style="margin-left: 16px"><p>protobufバイナリデータをJSONに変換します。</p><p>変換に失敗する場合があります。</p>
```

```markdown
1. TypeResolver は型を解決できません。
2. 入力が有効な JSON 形式でないか、TypeResolver によって返される型情報と競合しています。

注意：未知のフィールドは無視されます。
```

```markdown
`util::Status util::JsonToBinaryStream(TypeResolver * resolver, const std::string & type_url, io::ZeroCopyInputStream * json_input, io::ZeroCopyOutputStream * binary_output, const JsonParseOptions & options)`

JSON データを protobuf バイナリ形式に変換します。

変換が失敗する場合：

1. TypeResolver が型を解決できません。
2. 入力が有効な JSON 形式でないか、TypeResolver によって返される型情報と競合しています。
```

```markdown
### struct JsonParseOptions

```cpp
#include <google/protobuf/util/json_util.h>
namespace google::protobuf::util
```

#### Members

- `bool ignore_unknown_fields`
  - 未知の JSON フィールドをパース中に無視するかどうか。

- `bool case_insensitive_enum_parsing`
  - true の場合、小文字の enum 値がパースに失敗したときに、UPPER_CASE に変換して有効な enum と一致するかどうかを確認します。

```markdown
### struct JsonPrintOptions

```cpp
#include <google/protobuf/util/json_util.h>
namespace google::protobuf::util
```

#### Members

- `bool add_whitespace`
  - JSON 出力を読みやすくするためにスペース、改行、インデントを追加するかどうか。

- `bool always_print_primitive_fields`
  - 常にプリミティブフィールドを出力するかどうか。

- `bool always_print_enums_as_ints`
  - 常に enum を整数として出力するかどうか。

- `bool preserve_proto_field_names`
  - proto フィールド名を保持するかどうか。
```

I am ready to start the translation once you provide the Markdown content.
