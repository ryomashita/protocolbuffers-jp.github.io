+++
title = "Protocol Buffers"
weight = 5
toc_hide = "true"
description = "Protocol Buffers are language-neutral, platform-neutral extensible mechanisms for serializing structured data."
type = "docs"
no_list = "true"
+++

## プロトコルバッファとは何ですか？

プロトコルバッファは、Googleの言語に依存しない、プラットフォームに依存しない、拡張可能なメカニズムで、構造化されたデータをシリアライズするためのものです。XMLのようなものですが、より小さく、高速で、シンプルです。データの構造を一度定義し、特別に生成されたソースコードを使用して、構造化されたデータをさまざまなデータストリームやさまざまな言語を使用して簡単に書き込んだり読み取ったりできます。

## お好きな言語を選択してください

プロトコルバッファは、C++、C#、Dart、Go、Java、Kotlin、Objective-C、Python、およびRubyで生成されたコードをサポートしています。proto3を使用すると、PHPでも作業できます。

## 実装例

```proto
message Person {
  optional string name = 1;
  optional int32 id = 2;
  optional string email = 3;
}
```

**図1.** protoの定義。

```java
// Java code
Person john = Person.newBuilder()
    .setId(1234)
    .setName("John Doe")
    .setEmail("jdoe@example.com")
    .build();
output = new FileOutputStream(args[0]);
john.writeTo(output);
```

**図2.** 生成されたクラスを使用してデータを永続化する。

```cpp
// C++ code
Person john;
fstream input(argv[1],
    ios::in | ios::binary);
john.ParseFromIstream(&input);
id = john.id();
name = john.name();
email = john.email();
```

**図3.** 生成されたクラスを使用して永続化されたデータを解析する。

## どうやって始めればいいですか？

<ol>

  <li>
    <a href="https://github.com/protocolbuffers/protobuf#protobuf-compiler-installation">プロトコルバッファコンパイラをダウンロードしてインストール</a>してください。
  </li>

  <li>
    <a href="/overview">概要</a>を読んでください。
  </li>
  <li>
    選択した言語の<a href="/getting-started">チュートリアル</a>を試してみてください。
  </li>
</ol>
