+++
title = "field_mask_util.h"
toc_hide = "true"
linkTitle = "C++"
description = "このセクションには、C++でプロトコルバッファクラスを操作するためのリファレンスドキュメントが含まれています。"
type = "docs"
+++

```markdown
<code>#include &lt;google/protobuf/util/field_mask_util.h&gt;<br>namespace <a href="#google.protobuf.util">google::protobuf::util</a></code>

Defines utilities for the FieldMask well known type.

---

### Classes in this file {#examples}

- `<a href="#FieldMaskUtil">FieldMaskUtil</a>`
- `<a href="#FieldMaskUtil.MergeOptions">FieldMaskUtil::MergeOptions</a>`
- `<a href="#FieldMaskUtil.TrimOptions">FieldMaskUtil::TrimOptions</a>`

---

## class FieldMaskUtil {#FieldMaskUtil}

<code>#include &lt;<a href="#">google/protobuf/util/field_mask_util.h</a>&gt;<br>namespace <a href="#google.protobuf.util">google::protobuf::util</a></code>

---

### Members

- `static std::string` **ToString**(const FieldMask & mask)
  - Converts FieldMask to/from string, formatted by separating each path with a comma (e.g., "foo_bar,baz.quz").

- `static void` **FromString**(StringPiece str, FieldMask * out)

- `template static void` **FromFieldNumbers**(const std::vector&lt; int64_t &gt; & field_numbers, FieldMask * out)
  - Populates the FieldMask with the paths corresponding to the fields with the given numbers, after checking that all field numbers are valid.

- `static bool` **ToJsonString**(const FieldMask & mask, std::string * out)
  - Converts FieldMask to/from string, formatted according to proto3 JSON spec for FieldMask (e.g., "fooBar,baz.quz"). [more...](#FieldMaskUtil.ToJsonString.details)

- `static bool` **FromJsonString**(StringPiece str, FieldMask * out)

- `static bool` **GetFieldDescriptors**(const <a href='google.protobuf.descriptor#Descriptor'>Descriptor</a> * descriptor, StringPiece path, std::vector&lt; const <a href='google.protobuf.descriptor#FieldDescriptor'>FieldDescriptor</a> * &gt; * field_descriptors)
  - Get the descriptors of the fields which the given path from the message descriptor traverses, if field_descriptors is not null. [more...](#FieldMaskUtil.GetFieldDescriptors.details)

- `template static bool` **IsValidPath**(StringPiece path)
  - Checks whether the given path is valid for type T.

- `template static bool` **IsValidFieldMask**(const FieldMask & mask)
  - Checks whether the given FieldMask is valid for type T.

- `template static void` **AddPathToFieldMask**(StringPiece path, FieldMask * mask)
  - Adds a path to FieldMask after checking whether the given path is valid. [more...](#FieldMaskUtil.AddPathToFieldMask.details)

- `template static FieldMask` **GetFieldMaskForAllFields**()
  - Creates a FieldMask with all fields of type T. [more...](#FieldMaskUtil.GetFieldMaskForAllFields.details)

- `static void` **GetFieldMaskForAllFields**(const <a href='google.protobuf.descriptor#Descriptor'>Descriptor</a> * descriptor, FieldMask * out)
  - This flavor takes the protobuf type descriptor as an argument. [more...](#FieldMaskUtil.GetFieldMaskForAllFields.details)

- `static void` **ToCanonicalForm**(const FieldMask & mask, FieldMask * out)
  - Converts a FieldMask to the canonical form. [more...](#FieldMaskUtil.ToCanonicalForm.details)

- `static void` **Union**(const FieldMask & mask1, const FieldMask & mask2, FieldMask * out)
  - Creates an union of two FieldMasks.

- `static void` **Intersect**(const FieldMask & mask1, const FieldMask & mask2, FieldMask * out)
  - Creates an intersection of two FieldMasks.

- `template static void` **Subtract**(const FieldMask & mask1, const FieldMask & mask2, FieldMask * out)
  - Subtracts mask2 from mask1 base of type T.

- `static void` **Subtract**(const <a href='google.protobuf.descriptor#Descriptor'>Descriptor</a> * descriptor, const FieldMask & mask1, const FieldMask & mask2, FieldMask * out)
  - This flavor takes the protobuf type descriptor as an argument. [more...](#FieldMaskUtil.Subtract.details)

- `static bool` **IsPathInFieldMask**(StringPiece path, const FieldMask & mask)
  - Returns true if path is covered by the given FieldMask. [more...](#FieldMaskUtil.IsPathInFieldMask.details)

- `static void` **MergeMessageTo**(const <a href='google.protobuf.message#Message'>Message</a> & source, const FieldMask & mask, const <a href='#FieldMaskUtil.MergeOptions'>MergeOptions</a> & options, <a href='google.protobuf.message#Message'>Message</a> * destination)
  - Merges fields specified in a FieldMask into another message.

- `static bool` **TrimMessage**(const FieldMask & mask, <a href='google.protobuf.message#Message'>Message</a> * message)
  - Removes from 'message' any field that is not represented in the given FieldMask. [more...](#FieldMaskUtil.TrimMessage.details)

- `static bool` **TrimMessage**(const FieldMask & mask, <a href='google.protobuf.message#Message'>Message</a> * message, const <a href='#FieldMaskUtil.TrimOptions'>TrimOptions</a> & options)
  - Removes from 'message' any field that is not represented in the given FieldMask with customized <a href='#FieldMaskUtil.TrimOptions'>TrimOptions</a>. [more...](#FieldMaskUtil.TrimMessage.details)

---

### FieldMaskUtil::ToJsonString {#FieldMaskUtil.ToJsonString.details}

- Converts FieldMask to/from string, formatted according to proto3 JSON spec for FieldMask (e.g., "fooBar,baz.quz").
- If the field name is not style conforming (i.e., not snake_case when converted to string, or not camelCase when converted from string), the conversion will fail.

---

### FieldMaskUtil::GetFieldDescriptors {#FieldMaskUtil.GetFieldDescriptors.details}

- Get the descriptors of the fields which the given path from the message descriptor traverses, if field_descriptors is not null.
- Return false if the path is not valid, and the content of field_descriptors is unspecified.

---

### FieldMaskUtil::AddPathToFieldMask {#FieldMaskUtil.AddPathToFieldMask.details}

- Adds a path to FieldMask after checking whether the given path is valid.
- This method check-fails if the path is not a valid path for type T.

---

### FieldMaskUtil::GetFieldMaskForAllFields {#FieldMaskUtil.GetFieldMaskForAllFields.details}

- Creates a FieldMask with all fields of type T.
- This FieldMask only contains fields of T but not any sub-message fields.

---

### FieldMaskUtil::GetFieldMaskForAllFields {#FieldMaskUtil.GetFieldMaskForAllFields.details}

- This flavor takes the protobuf type descriptor as an argument.
- Useful when the type is not known at compile time.

---

### FieldMaskUtil::ToCanonicalForm {#FieldMaskUtil.ToCanonicalForm.details}

- Converts a FieldMask to the canonical form.
- It will:
```

<pre>1. 別のパスによってカバーされているパスを削除します。例えば、
   "foo.bar" は "foo" によってカバーされており、"foo" が FieldMask に含まれている場合は削除されます。
2. すべてのパスをアルファベット順に並べ替えます。</pre>

</div> <hr><h3 id="FieldMaskUtil.Subtract.details"><code>static void FieldMaskUtil::Subtract(<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;const <a href='google.protobuf.descriptor#Descriptor'>Descriptor</a> * descriptor,<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;const FieldMask &amp; mask1,<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;const FieldMask &amp; mask2,<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;FieldMask * out)</code></h3><div style="margin-left: 16px"><p>このフレーバーは、protobuf タイプのディスクリプタを引数として取ります。</p><p>コンパイル時にタイプがわからない場合に便利です。</p>
</div> <hr><h3 id="FieldMaskUtil.IsPathInFieldMask.details"><code>static bool FieldMaskUtil::IsPathInFieldMask(<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;StringPiece path,<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;const FieldMask &amp; mask)</code></h3><div style="margin-left: 16px"><p>指定された FieldMask によってパスがカバーされている場合は true を返します。</p><p>パス "foo.bar" は "foo.bar.baz"、"foo.bar.quz.x" などのすべてのパスをカバーし、親パスは明示的な子パスによってカバーされないことに注意してください。つまり、"foo.bar" は "foo" をカバーしません。たとえ "bar" が唯一の子であってもです。</p>
</div> <hr><h3 id="FieldMaskUtil.TrimMessage.details"><code>static bool FieldMaskUtil::TrimMessage(<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;const FieldMask &amp; mask,<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href='google.protobuf.message#Message'>Message</a> * message)</code></h3><div style="margin-left: 16px"><p>指定された FieldMask に表されていないフィールドを 'message' から削除します。</p><p>FieldMask が空の場合、何もしません。メッセージが変更された場合は true を返します。</p>
</div> <hr><h3 id="FieldMaskUtil.TrimMessage.details"><code>static bool FieldMaskUtil::TrimMessage(<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;const FieldMask &amp; mask,<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href='google.protobuf.message#Message'>Message</a> * message,<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;const <a href='#FieldMaskUtil.TrimOptions'>TrimOptions</a> &amp; options)</code></h3><div style="margin-left: 16px"><p>指定された FieldMask によって表されていないフィールドをカスタマイズされた <a href='#FieldMaskUtil.TrimOptions'>TrimOptions</a> で 'message' から削除します。</p><p>FieldMask が空の場合、何もしません。メッセージが変更された場合は true を返します。</p>
</div><h2 id="FieldMaskUtil.MergeOptions">class FieldMaskUtil::MergeOptions</h2><p><code>#include &lt;<a href="#">google/protobuf/util/field_mask_util.h</a>&gt;<br>namespace <a href="#google.protobuf.util">google::protobuf::util</a></code></p><p></p><table><tr><th colspan="2"><h3 style="margin-top: 4px">Members</h3></th></tr><tr><td style="border-right-width: 0px; text-align: right;"><code></code></td><td style="border-left-width: 0px"id="FieldMaskUtil.MergeOptions.MergeOptions"><div style="padding-left: 16px; text-indent: -16px"><code><b>MergeOptions</b>()</code></div></td></tr><tr><td style="border-right-width: 0px; text-align: right;"><code>void</code></td><td style="border-left-width: 0px"id="FieldMaskUtil.MergeOptions.set_replace_message_fields"><div style="padding-left: 16px; text-indent: -16px"><code><b>set_replace_message_fields</b>(bool value)</code></div><div style="font-style: italic; margin-top: 4px; margin-left: 16px;">メッセージフィールドをマージする際のデフォルトの動作は、2 つのメッセージフィールドの内容を一緒にマージすることです。<a href="#FieldMaskUtil.MergeOptions.set_replace_message_fields.details">詳細...</a></div></td></tr><tr><td style="border-right-width: 0px; text-align: right;"><code>bool</code></td><td style="border-left-width: 0px"id="FieldMaskUtil.MergeOptions.replace_message_fields"><div style="padding-left: 16px; text-indent: -16px"><code><b>replace_message_fields</b>() const</code></div></td></tr><tr><td style="border-right-width: 0px; text-align: right;"><code>void</code></td><td style="border-left-width: 0px"id="FieldMaskUtil.MergeOptions.set_replace_repeated_fields"><div style="padding-left: 16px; text-indent: -16px"><code><b>set_replace_repeated_fields</b>(bool value)</code></div><div style="font-style: italic; margin-top: 4px; margin-left: 16px;">デフォルトのマージ動作は、ソースの繰り返しフィールドのエントリを宛先の繰り返しフィールドに追加することです。<a href="#FieldMaskUtil.MergeOptions.set_replace_repeated_fields.details">詳細...</a></div></td></tr><tr><td style="border-right-width: 0px; text-align: right;"><code>bool</code></td><td style="border-left-width: 0px"id="FieldMaskUtil.MergeOptions.replace_repeated_fields"><div style="padding-left: 16px; text-indent: -16px"><code><b>replace_repeated_fields</b>() const</code></div></td></tr></table> <hr><h3 id="FieldMaskUtil.MergeOptions.set_replace_message_fields.details"><code>void MergeOptions::set_replace_message_fields(<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;bool value)</code></h3><div style="margin-left: 16px"><p>メッセージフィールドをマージする際のデフォルトの動作は、2 つのメッセージフィールドの内容を一緒にマージすることです。</p><p>代わりに、ソースメッセージからフィールドを使用して、宛先メッセージの対応するフィールドを置換する場合は、このフラグを true に設定します。このフラグが設定されている場合、ソースに存在しない指定されたサブメッセージフィールドは宛先でクリアされます。</p>
</div> <hr><h3 id="FieldMaskUtil.MergeOptions.set_replace_repeated_fields.details"><code>void MergeOptions::set_replace_repeated_fields(<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;bool value)</code></h3><div style="margin-left: 16px"><p>デフォルトのマージ動作は、ソースの繰り返しフィールドのエントリを宛先の繰り返しフィールドに追加することです。</p><p>ソースの繰り返しフィールドからエントリのみを保持したい場合は、このフラグを true に設定します。</p>
</div><h2 id="FieldMaskUtil.TrimOptions">class FieldMaskUtil::TrimOptions</h2><p><code>#include &lt;<a href="#">google/protobuf/util/field_mask_util.h</a>&gt;<br>namespace <a href="#google.protobuf.util">google::protobuf::util</a></code></p><p></p><table><tr><th colspan="2"><h3 style="margin-top: 4px">Members</h3></th></tr><tr><td style="border-right-width: 0px; text-align: right;"><code></code></td><td style="border-left-width: 0px"id="FieldMaskUtil.TrimOptions.TrimOptions"><div style="padding-left: 16px; text-indent: -16px"><code><b>TrimOptions</b>()</code></div></td></tr><tr><td style="border-right-width: 0px; text-align: right;"><code>void</code></td><td style="border-left-width: 0px"id="FieldMaskUtil.TrimOptions.set_keep_required_fields"><div style="padding-left: 16px; text-indent: -16px"><code><b>set_keep_required_fields</b>(bool value)</code></div><div style="font-style: italic; margin-top: 4px; margin-left: 16px;">メッセージフィールドをトリミングする際のデフォルトの動作は、フィールドマスクで指定されていない場合に、現在のメッセージの必須フィールドをトリミングすることです。<a href="#FieldMaskUtil.TrimOptions.set_keep_required_fields.details">詳細...</a></div></td></tr><tr><td style="border-right-width: 0px; text-align: right;"><code>bool</code></td><td style="border-left-width: 0px"id="FieldMaskUtil.TrimOptions.keep_required_fields"><div style="padding-left: 16px; text-indent: -16px"><code><b>keep_required_fields</b>() const</code></div></td></tr></table> <hr><h3 id="FieldMaskUtil.TrimOptions.set_keep_required_fields.details"><code>void TrimOptions::set_keep_required_fields(<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;bool value)</code></h3><div style="margin-left: 16px"><p>メッセージフィールドをトリミングする際のデフォルトの動作は、フィールドマスクで指定されていない場合に、現在のメッセージの必須フィールドをトリミングすることです。</p><p>代わりに、フィールドマスクで指定されていない場合でも、現在のメッセージの必須フィールドを保持したい場合は、このフラグを true に設定します。</p>
</div>

Please paste the Markdown content that you would like me to translate into Japanese.
