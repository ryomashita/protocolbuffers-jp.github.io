+++
title = "プロトコルバッファ"
weight = 5
toc_hide = "true"
description = "プロトコルバッファは言語中立で、プラットフォーム中立の拡張可能なメカニズムで、構造化されたデータをシリアライズするためのものです。"
type = "docs"
no_list = "true"
+++

## プロトコルバッファとは？

プロトコルバッファは、Googleの言語中立で、プラットフォーム中立で、拡張可能なメカニズムで、構造化されたデータをシリアライズするためのものです。XMLのようなものですが、より小さく、高速で、シンプルです。データの構造を一度定義し、特別に生成されたソースコードを使用して、構造化されたデータをさまざまなデータストリームやさまざまな言語を使用して簡単に書き込み、読み取ることができます。

## お気に入りの言語を選択

プロトコルバッファは、C++、C#、Dart、Go、Java、Kotlin、Objective-C、Python、およびRubyで生成されたコードをサポートしています。proto3を使用すると、PHPでも作業できます。

## 実装例

```proto
message Person {
  optional string name = 1;
  optional int32 id = 2;
  optional string email = 3;
}
```

**Figure 1.** プロト定義。

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

**Figure 2.** 生成されたクラスを使用してデータを永続化する。

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

**Figure 3.** 生成されたクラスを使用して永続化されたデータを解析する。

## どうやって始める？

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
