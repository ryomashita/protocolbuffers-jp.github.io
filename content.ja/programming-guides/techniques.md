+++
title = "技術"
weight = 70
description = "Protocol Buffers に対処するための一般的に使用されるデザインパターンを説明します。"
type = "docs"
+++

[Protocol Buffers discussion group](http://groups.google.com/group/protobuf) にもデザインや使用に関する質問を送ることができます。

## 一般的なファイル名の接尾辞 {#suffixes}

複数の異なる形式でファイルにメッセージを書き込むことはかなり一般的です。これらのファイルには以下の拡張子を使用することを推奨します。

内容                                                                   | 拡張子
------------------------------------------------------------------------- | ---------
[Text Format](/reference/protobuf/textformat-spec) | `.txtpb`
[Wire Format](/programming-guides/encoding)        | `.binpb`
[JSON Format](/programming-guides/proto3#json)     | `.json`

特に Text Format の場合、`.textproto` もかなり一般的ですが、簡潔さのために `.txtpb` を推奨します。

## 複数のメッセージをストリーミングする {#streaming}

複数のメッセージを単一のファイルやストリームに書き込みたい場合、1 つのメッセージが終わり、次のメッセージが始まる場所を追跡する必要があります。Protocol Buffer ワイヤーフォーマットは自己区切りではないため、プロトコルバッファパーサーはメッセージがどこで終わるかを自動的に判断できません。この問題を解決する最も簡単な方法は、メッセージを書き込む前に各メッセージのサイズを書き込むことです。メッセージを読み込む際には、サイズを読み取り、その後バイトを別々のバッファに読み込み、そのバッファから解析します。（バイトを別のバッファにコピーするのを避けたい場合は、特定のバイト数に読み取りを制限できる `CodedInputStream` クラス（C++ および Java の両方に存在）を確認してください。）

## 大規模データセット {#large-data}

Protocol Buffers は大きなメッセージを処理するために設計されていません。一般的な経験則として、1 メガバイトを超えるメッセージを扱っている場合は、別の戦略を考える時期かもしれません。

ただし、Protocol Buffers は大規模データセット内の個々のメッセージを処理するのに適しています。通常、大規模データセットは小さな部分の集まりであり、各小さな部分が構造化されたデータである場合があります。Protocol Buffers は一度に全体を処理できなくても、各部分をエンコードするために使用すると、問題が大幅に簡素化されます。今後は、構造体の集まりではなくバイト文字列の集まりを処理する必要があります。

Protocol Buffersには大規模なデータセットをサポートする機能が組み込まれていません。異なる状況には異なる解決策が必要です。時には単純なレコードのリストが適している場合もありますが、他の場合にはデータベースのようなものが必要になることもあります。各解決策は別々のライブラリとして開発すべきであり、必要な人だけがそのコストを支払う必要があります。

## 自己記述メッセージ {#self-description}

Protocol Buffersには自身のタイプの説明が含まれていません。したがって、対応する`.proto`ファイルでそのタイプを定義していない生のメッセージだけが与えられた場合、有用なデータを抽出することは難しいです。

ただし、.protoファイルの内容自体もプロトコルバッファを使用して表現することができます。ソースコードパッケージ内の`src/google/protobuf/descriptor.proto`ファイルは関連するメッセージタイプを定義しています。`protoc`は`--descriptor_set_out`オプションを使用して、一連の.protoファイルを表す`FileDescriptorSet`を出力できます。これにより、次のように自己記述プロトコルメッセージを定義できます：

```proto
syntax = "proto3";

import "google/protobuf/any.proto";
import "google/protobuf/descriptor.proto";

message SelfDescribingMessage {
  // Set of FileDescriptorProtos which describe the type and its dependencies.
  google.protobuf.FileDescriptorSet descriptor_set = 1;

  // The message and its type, encoded as an Any message.
  google.protobuf.Any message = 2;
}
```

C++やJavaで利用可能な`DynamicMessage`などのクラスを使用することで、`SelfDescribingMessage`を操作できるツールを作成できます。

以上のように、この機能がProtocol Bufferライブラリに含まれていない理由は、Google内でそのような必要性がなかったからです。

このテクニックには、ディスクリプタを使用した動的メッセージのサポートが必要です。自己記述メッセージを使用する前に、プラットフォームがこの機能をサポートしているか確認してください。
