+++
title = "objectivec_helpers.h"
toc_hide = "true"
linkTitle = "C++"
description = "このセクションには、C++でプロトコルバッファクラスを操作するためのリファレンスドキュメントが含まれています。"
type = "docs"
+++

```markdown
`#include <google/protobuf/compiler/objectivec/objectivec_helpers.h>`
namespace [google::protobuf::compiler::objectivec](#google.protobuf.compiler)

Helper functions for generating ObjectiveC code.

## Classes in this file {#Options}
- [`Options`](#Options)
  - Generator options (see objectivec_generator.cc for a description of each):

- [`TextFormatDecodeData`](#TextFormatDecodeData)
  - Generate decode data needed for ObjC's GPBDecodeTextFormatName() to transform the input into the expected output.

- [`LineConsumer`](#LineConsumer)
  - Helper for parsing simple files.

- [`ImportWriter`](#ImportWriter)
  - Helper class for parsing framework import mappings and generating import statements.

## File Members
These definitions are not part of any class.

- `enum` {#ObjectiveCType}
  - `ObjectiveCType`
    - [more...](#ObjectiveCType.details)

- `enum` {#FlagType}
  - `FlagType`
    - [more...](#FlagType.details)

- `const char *const` {#ProtobufLibraryFrameworkName}
  - `ProtobufLibraryFrameworkName`
    - The name commonly used by the library when built as a framework.
    - [more...](#ProtobufLibraryFrameworkName.details)

- `std::string` {#EscapeTrigraphs}
  - `EscapeTrigraphs`(const std::string & to_escape)
    - Escape C++ trigraphs by escaping question marks to "\?".

- `void` {#TrimWhitespace}
  - `TrimWhitespace`(StringPiece * input)
    - Remove white space from either end of a StringPiece.

- `bool` {#IsRetainedName}
  - `IsRetainedName`(const std::string & name)
    - Returns true if the name requires a ns_returns_not_retained attribute applied to it.

- `bool` {#IsInitName}
  - `IsInitName`(const std::string & name)
    - Returns true if the name starts with "init" and will need to have special handling under ARC.

- `std::string` {#FileClassPrefix}
  - `FileClassPrefix`(const [FileDescriptor](google.protobuf.descriptor#FileDescriptor) * file)
    - Gets the objc_class_prefix.

- `std::string` {#FilePath}
  - `FilePath`(const [FileDescriptor](google.protobuf.descriptor#FileDescriptor) * file)
    - Gets the path of the file we're going to generate (sans the .pb.h extension).
    - [more...](#FilePath.details)

- `std::string` {#FilePathBasename}
  - `FilePathBasename`(const [FileDescriptor](google.protobuf.descriptor#FileDescriptor) * file)
    - Just like [FilePath()](#FilePath), but without the directory part.

- `std::string` {#FileClassName}
  - `FileClassName`(const [FileDescriptor](google.protobuf.descriptor#FileDescriptor) * file)
    - Gets the name of the root class we'll generate in the file.
    - [more...](#FileClassName.details)

- `std::string` {#ClassName}
  - `ClassName`(const [Descriptor](google.protobuf.descriptor#Descriptor) * descriptor)
    - These return the fully-qualified class name corresponding to the given descriptor.

- `std::string` {#ClassName}
  - `ClassName`(const [Descriptor](google.protobuf.descriptor#Descriptor) * descriptor, std::string * out_suffix_added)

- `std::string` {#EnumName}
  - `EnumName`(const [EnumDescriptor](google.protobuf.descriptor#EnumDescriptor) * descriptor)

- `std::string` {#EnumValueName}
  - `EnumValueName`(const [EnumValueDescriptor](google.protobuf.descriptor#EnumValueDescriptor) * descriptor)
    - Returns the fully-qualified name of the enum value corresponding to the descriptor.

- `std::string` {#EnumValueShortName}
  - `EnumValueShortName`(const [EnumValueDescriptor](google.protobuf.descriptor#EnumValueDescriptor) * descriptor)
    - Returns the name of the enum value corresponding to the descriptor.

- `std::string` {#UnCamelCaseEnumShortName}
  - `UnCamelCaseEnumShortName`(const std::string & name)
    - Reverse what an enum does.

- `std::string` {#ExtensionMethodName}
  - `ExtensionMethodName`(const [FieldDescriptor](google.protobuf.descriptor#FieldDescriptor) * descriptor)
    - Returns the name to use for the extension (used as the method off the file's Root class).

- `std::string` {#FieldName}
  - `FieldName`(const [FieldDescriptor](google.protobuf.descriptor#FieldDescriptor) * field)
    - Returns the transformed field name.

- `std::string` {#FieldNameCapitalized}
  - `FieldNameCapitalized`(const [FieldDescriptor](google.protobuf.descriptor#FieldDescriptor) * field)

- `std::string` {#OneofEnumName}
  - `OneofEnumName`(const [OneofDescriptor](google.protobuf.descriptor#OneofDescriptor) * descriptor)
    - Returns the transformed oneof name.

- `std::string` {#OneofName}
  - `OneofName`(const [OneofDescriptor](google.protobuf.descriptor#OneofDescriptor) * descriptor)

- `std::string` {#OneofNameCapitalized}
  - `OneofNameCapitalized`(const [OneofDescriptor](google.protobuf.descriptor#OneofDescriptor) * descriptor)

- `std::string` {#ObjCClass}
  - `ObjCClass`(const std::string & class_name)
    - Returns a symbol that can be used in C code to refer to an Objective C class without initializing the class.

- `std::string` {#ObjCClassDeclaration}
  - `ObjCClassDeclaration`(const std::string & class_name)
    - Declares an Objective C class without initializing the class so that it can be referred to by ObjCClass.

- `bool` {#HasPreservingUnknownEnumSemantics}
  - `HasPreservingUnknownEnumSemantics`(const [FileDescriptor](google.protobuf.descriptor#FileDescriptor) * file)

- `bool` {#IsMapEntryMessage}
  - `IsMapEntryMessage`(const [Descriptor](google.protobuf.descriptor#Descriptor) * descriptor)

- `std::string` {#UnCamelCaseFieldName}
  - `UnCamelCaseFieldName`(const std::string & name, const [FieldDescriptor](google.protobuf.descriptor#FieldDescriptor) * field)
    - Reverse of the above.

- `template std::string` {#GetOptionalDeprecatedAttribute}
  - `GetOptionalDeprecatedAttribute`(const TDescriptor * descriptor, const [FileDescriptor](google.protobuf.descriptor#FileDescriptor) * file = NULL, bool preSpace = true, bool postNewline = false)

- `std::string` {#GetCapitalizedType}
  - `GetCapitalizedType`(const [FieldDescriptor](google.protobuf.descriptor#FieldDescriptor) * field)

- `ObjectiveCType` {#GetObjectiveCType}
  - `GetObjectiveCType`([FieldDescriptor.Type](google.protobuf.descriptor#FieldDescriptor.Type) field_type)

- `ObjectiveCType` {#GetObjectiveCType}
  - `GetObjectiveCType`(const [FieldDescriptor](google.protobuf.descriptor#FieldDescriptor) * field)

- `bool` {#IsPrimitiveType}
  - `IsPrimitiveType`(const [FieldDescriptor](google.protobuf.descriptor#FieldDescriptor) * field)

- `bool` {#IsReferenceType}
  - `IsReferenceType`(const [FieldDescriptor](google.protobuf.descriptor#FieldDescriptor) * field)

- `std::string` {#GPBGenericValueFieldName}
  - `GPBGenericValueFieldName`(const [FieldDescriptor](google.protobuf.descriptor#FieldDescriptor) * field)

- `std::string` {#DefaultValue}
  - `DefaultValue`(const [FieldDescriptor](google.protobuf.descriptor#FieldDescriptor) * field)

- `bool` {#HasNonZeroDefaultValue}
  - `HasNonZeroDefaultValue`(const [FieldDescriptor](google.protobuf.descriptor#FieldDescriptor) * field)

- `std::string` {#BuildFlagsString}
  - `BuildFlagsString`(const FlagType type, const std::vector< std::string > & strings)

- `std::string` {#BuildCommentsString}
  - `BuildCommentsString`(const [SourceLocation](google.protobuf.descriptor#SourceLocation) & location, bool prefer_single_line)
    - Builds HeaderDoc/appledoc style comments out of the comments in the .proto file.

- `std::string` {#ProtobufFrameworkImportSymbol}
  - `ProtobufFrameworkImportSymbol`(const std::string & framework_name)
    - Returns the CPP symbol name to use as the gate for framework style imports for the given framework name to use.

- `bool` {#IsProtobufLibraryBundledProtoFile}
  - `IsProtobufLibraryBundledProtoFile`(const [FileDescriptor](google.protobuf.descriptor#FileDescriptor) * file)
    - Checks if the file is one of the proto's bundled with the library.

- `bool` {#ValidateObjCClassPrefixes}
  - `ValidateObjCClassPrefixes`(const std::vector< const [FileDescriptor](google.protobuf.descriptor#FileDescriptor) * > & files, const [Options](#Options) & generation_options, std::string * out_error)
    - Checks the prefix for the given files and outputs any warnings as needed.
    - [more...](#ValidateObjCClassPrefixes.details)

- `bool` {#ParseSimpleFile}
  - `ParseSimpleFile`(const std::string & path, [LineConsumer](#LineConsumer) * line_consumer, std::string * out_error)

---

### ObjectiveCType {#ObjectiveCType.details}
```objective-c
enum objectivec::ObjectiveCType {
    OBJECTIVECTYPE_INT32,
    OBJECTIVECTYPE_UINT32,
    OBJECTIVECTYPE_INT64,
    OBJECTIVECTYPE_UINT64,
    OBJECTIVECTYPE_FLOAT,
    OBJECTIVECTYPE_DOUBLE,
    OBJECTIVECTYPE_BOOLEAN,
    OBJECTIVECTYPE_STRING,
    OBJECTIVECTYPE_DATA,
    OBJECTIVECTYPE_ENUM,
    OBJECTIVECTYPE_MESSAGE
}
```

### FlagType {#FlagType.details}
```objective-c
enum objectivec::FlagType {
    FLAGTYPE_DESCRIPTOR_INITIALIZATION,
    FLAGTYPE_EXTENSION,
    FLAGTYPE_FIELD
}
```

### ProtobufLibraryFrameworkName {#ProtobufLibraryFrameworkName.details}
The name commonly used by the library when built as a framework.
This lines up to the name used in the CocoaPod.
```markdown
```

```markdown
</div> <hr><h3 id="FilePath.details"><code>std::string objectivec::FilePath(<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;const <a href='google.protobuf.descriptor#FileDescriptor'>FileDescriptor</a> * file)</code></h3><div style="margin-left: 16px"><p>生成するファイルのパスを取得します（.pb.h 拡張子を除く）。 </p><p>このパスは、proto パッケージで宣言された objectc パッケージに依存します。 </p>
</div> <hr><h3 id="FileClassName.details"><code>std::string objectivec::FileClassName(<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;const <a href='google.protobuf.descriptor#FileDescriptor'>FileDescriptor</a> * file)</code></h3><div style="margin-left: 16px"><p>ファイル内で生成するルートクラスの名前を取得します。 </p><p>このクラスは外部で使用するためではなく、他のクラスが必要とするヘルパーを含んでいます。 </p>
</div> <hr><h3 id="ValidateObjCClassPrefixes.details"><code>bool objectivec::ValidateObjCClassPrefixes(<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;const std::vector&lt; const <a href='google.protobuf.descriptor#FileDescriptor'>FileDescriptor</a> * &gt; &amp; files,<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;const <a href='#Options'>Options</a> &amp; generation_options,<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;std::string * out_error)</code></h3><div style="margin-left: 16px"><p>指定されたファイルの接頭辞をチェックし、必要に応じて警告を出力します。 </p><p>致命的なエラーがある場合、out_error に最初のエラーが記入され、結果は false になります。 </p>
</div><h2 id="Options">struct Options</h2><p><code>#include &lt;<a href="#">google/protobuf/compiler/objectivec/objectivec_helpers.h</a>&gt;<br>namespace <a href="#google.protobuf.compiler">google::protobuf::compiler::objectivec</a></code></p><p>ジェネレータオプション（各オプションの説明については、objectivec_generator.cc を参照してください）: </p><table><tr><th colspan="2"><h3 style="margin-top: 4px">メンバー</h3></th></tr><tr><td style="border-right-width: 0px; text-align: right;"><code>std::string</code></td><td style="border-left-width: 0px"id="Options.expected_prefixes_path"><div style="padding-left: 16px; text-indent: -16px"><code><b>expected_prefixes_path</b></code></div></td></tr><tr><td style="border-right-width: 0px; text-align: right;"><code>std::vector&lt; std::string &gt;</code></td><td style="border-left-width: 0px"id="Options.expected_prefixes_suppressions"><div style="padding-left: 16px; text-indent: -16px"><code><b>expected_prefixes_suppressions</b></code></div></td></tr><tr><td style="border-right-width: 0px; text-align: right;"><code>std::string</code></td><td style="border-left-width: 0px"id="Options.generate_for_named_framework"><div style="padding-left: 16px; text-indent: -16px"><code><b>generate_for_named_framework</b></code></div></td></tr><tr><td style="border-right-width: 0px; text-align: right;"><code>std::string</code></td><td style="border-left-width: 0px"id="Options.named_framework_to_proto_path_mappings_path"><div style="padding-left: 16px; text-indent: -16px"><code><b>named_framework_to_proto_path_mappings_path</b></code></div></td></tr><tr><td style="border-right-width: 0px; text-align: right;"><code>std::string</code></td><td style="border-left-width: 0px"id="Options.runtime_import_prefix"><div style="padding-left: 16px; text-indent: -16px"><code><b>runtime_import_prefix</b></code></div></td></tr><tr><td style="border-right-width: 0px; text-align: right;"><code></code></td><td style="border-left-width: 0px"id="Options.Options"><div style="padding-left: 16px; text-indent: -16px"><code><b>Options</b>()</code></div></td></tr></table><h2 id="TextFormatDecodeData">class TextFormatDecodeData</h2><p><code>#include &lt;<a href="#">google/protobuf/compiler/objectivec/objectivec_helpers.h</a>&gt;<br>namespace <a href="#google.protobuf.compiler">google::protobuf::compiler::objectivec</a></code></p><p>ObjC の GPBDecodeTextFormatName() で必要なデコードデータを生成して、入力を期待される出力に変換します。 </p><table><tr><th colspan="2"><h3 style="margin-top: 4px">メンバー</h3></th></tr><tr><td style="border-right-width: 0px; text-align: right;"><code></code></td><td style="border-left-width: 0px"id="TextFormatDecodeData.TextFormatDecodeData"><div style="padding-left: 16px; text-indent: -16px"><code><b>TextFormatDecodeData</b>()</code></div></td></tr><tr><td style="border-right-width: 0px; text-align: right;"><code></code></td><td style="border-left-width: 0px"id="TextFormatDecodeData.~TextFormatDecodeData"><div style="padding-left: 16px; text-indent: -16px"><code><b>~TextFormatDecodeData</b>()</code></div></td></tr><tr><td style="border-right-width: 0px; text-align: right;"><code></code></td><td style="border-left-width: 0px"id="TextFormatDecodeData.TextFormatDecodeData"><div style="padding-left: 16px; text-indent: -16px"><code><b>TextFormatDecodeData</b>(const <a href='#TextFormatDecodeData'>TextFormatDecodeData</a> &amp; )</code></div></td></tr><tr><td style="border-right-width: 0px; text-align: right;"><code><a href='#TextFormatDecodeData'>TextFormatDecodeData</a> &amp;</code></td><td style="border-left-width: 0px"id="TextFormatDecodeData.operator="><div style="padding-left: 16px; text-indent: -16px"><code><b>operator=</b>(const <a href='#TextFormatDecodeData'>TextFormatDecodeData</a> &amp; )</code></div></td></tr><tr><td style="border-right-width: 0px; text-align: right;"><code>void</code></td><td style="border-left-width: 0px"id="TextFormatDecodeData.AddString"><div style="padding-left: 16px; text-indent: -16px"><code><b>AddString</b>(int32 key, const std::string &amp; input_for_decode, const std::string &amp; desired_output)</code></div></td></tr><tr><td style="border-right-width: 0px; text-align: right;"><code>size_t</code></td><td style="border-left-width: 0px"id="TextFormatDecodeData.num_entries"><div style="padding-left: 16px; text-indent: -16px"><code><b>num_entries</b>() const</code></div></td></tr><tr><td style="border-right-width: 0px; text-align: right;"><code>std::string</code></td><td style="border-left-width: 0px"id="TextFormatDecodeData.Data"><div style="padding-left: 16px; text-indent: -16px"><code><b>Data</b>() const</code></div></td></tr><tr><td style="border-right-width: 0px; text-align: right;"><code>static std::string</code></td><td style="border-left-width: 0px"id="TextFormatDecodeData.DecodeDataForString"><div style="padding-left: 16px; text-indent: -16px"><code><b>DecodeDataForString</b>(const std::string &amp; input_for_decode, const std::string &amp; desired_output)</code></div></td></tr></table><h2 id="LineConsumer">class LineConsumer</h2><p><code>#include &lt;<a href="#">google/protobuf/compiler/objectivec/objectivec_helpers.h</a>&gt;<br>namespace <a href="#google.protobuf.compiler">google::protobuf::compiler::objectivec</a></code></p><p>シンプルなファイルの解析のためのヘルパー。 </p><table><tr><th colspan="2"><h3 style="margin-top: 4px">メンバー</h3></th></tr><tr><td style="border-right-width: 0px; text-align: right;"><code></code></td><td style="border-left-width: 0px"id="LineConsumer.LineConsumer"><div style="padding-left: 16px; text-indent: -16px"><code><b>LineConsumer</b>()</code></div></td></tr><tr><td style="border-right-width: 0px; text-align: right;"><code>virtual </code></td><td style="border-left-width: 0px"id="LineConsumer.~LineConsumer"><div style="padding-left: 16px; text-indent: -16px"><code><b>~LineConsumer</b>()</code></div></td></tr><tr><td style="border-right-width: 0px; text-align: right;"><code>virtual bool</code></td><td style="border-left-width: 0px"id="LineConsumer.ConsumeLine"><div style="padding-left: 16px; text-indent: -16px"><code><b>ConsumeLine</b>(const StringPiece &amp; line, std::string * out_error)  = 0</code></div></td></tr></table><h2 id="ImportWriter">class ImportWriter</h2><p><code>#include &lt;<a href="#">google/protobuf/compiler/objectivec/objectivec_helpers.h</a>&gt;<br>namespace <a href="#google.protobuf.compiler">google::protobuf::compiler::objectivec</a></code></p><p>フレームワークのインポートマッピングを解析し、インポートステートメントを生成するためのヘルパークラス。 </p><table><tr><th colspan="2"><h3 style="margin-top: 4px">メンバー</h3></th></tr><tr><td style="border-right-width: 0px; text-align: right;"><code></code></td><td style="border-left-width: 0px"id="ImportWriter.ImportWriter"><div style="padding-left: 16px; text-indent: -16px"><code><b>ImportWriter</b>(const std::string &amp; generate_for_named_framework, const std::string &amp; named_framework_to_proto_path_mappings_path, const std::string &amp; runtime_import_prefix, bool include_wkt_imports)</code></div></td></tr><tr><td style="border-right-width: 0px; text-align: right;"><code></code></td><td style="border-left-width: 0px"id="ImportWriter.~ImportWriter"><div style="padding-left: 16px; text-indent: -16px"><code><b>~ImportWriter</b>()</code></div></td></tr><tr><td style="border-right-width: 0px; text-align: right;"><code>void</code></td><td style="border-left-width: 0px"id="ImportWriter.AddFile"><div style="padding-left: 16px; text-indent: -16px"><code><b>AddFile</b>(const <a href='google.protobuf.descriptor#FileDescriptor'>FileDescriptor</a> * file, const std::string &amp; header_extension)</code></div></td></tr><tr><td style="border-right-width: 0px; text-align: right;"><code>void</code></td><td style="border-left-width: 0px"id="ImportWriter.Print"><div style="padding-left: 16px; text-indent: -16px"><code><b>Print</b>(<a href='google.protobuf.io.printer#Printer'>io::Printer</a> * printer) const</code></div></td></tr><tr><td style="border-right-width: 0px; text-align: right;"><code>static void</code></td><td style="border-left-width: 0px"id="ImportWriter.PrintRuntimeImports"><div style="padding-left: 16px; text-indent: -16px"><code><b>PrintRuntimeImports</b>(<a href='google.protobuf.io.printer#Printer'>io::Printer</a> * printer, const std::vector&lt; std::string &gt; &amp; header_to_import, const std::string &amp; runtime_import_prefix, bool default_cpp_symbol = false)</code></div></td></tr></table>
```

Please paste the Markdown content you would like me to translate into Japanese.
