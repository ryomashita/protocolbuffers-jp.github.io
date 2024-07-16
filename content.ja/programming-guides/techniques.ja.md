## 一般的なファイル名の接尾辞 {#suffixes}

異なる形式でファイルにメッセージを書き込むことはかなり一般的です。これらのファイルには、以下のファイル拡張子の使用を推奨します。

内容                                                                   | 拡張子
------------------------------------------------------------------------- | ---------
[テキスト形式](/reference/protobuf/textformat-spec) | `.txtpb`
[ワイヤ形式](/programming-guides/encoding)        | `.binpb`
[JSON形式](/programming-guides/proto3#json)     | `.json`

特にテキスト形式の場合、`.textproto` もかなり一般的ですが、簡潔さのために `.txtpb` を推奨します。

## 複数のメッセージをストリーミングする {#streaming}

1 つのファイルやストリームに複数のメッセージを書き込みたい場合、1 つのメッセージが終わり、次のメッセージが始まる場所を追跡する必要があります。Protocol Buffer ワイヤ形式は自己区切りではないため、プロトコルバッファパーサーはメッセージの終わりを自動的に判断できません。この問題を解決する最も簡単な方法は、メッセージを書き込む前に各メッセージのサイズを書き込むことです。メッセージを読み込む際には、サイズを読み取り、その後バイトを別のバッファに読み込み、そのバッファから解析します。（バイトを別のバッファにコピーするのを避けたい場合は、特定のバイト数に読み取りを制限できる `CodedInputStream` クラス（C++ および Java の両方に存在）を確認してください。）

## 大規模なデータセット {#large-data}

Protocol Buffers は大きなメッセージを処理するために設計されていません。一般的な指針として、1 メガバイトを超えるメッセージを扱う場合は、別の戦略を検討する時期かもしれません。

ただし、Protocol Buffers は大規模なデータセット内の個々のメッセージを処理するのに適しています。通常、大規模なデータセットは小さな部分のコレクションであり、各小さな部分が構造化されたデータです。Protocol Buffers は一度に全体のセットを処理できなくても、各部分をエンコードするために使用すると、問題が大幅に簡素化されます。これで、構造体のセットではなくバイト文字列のセットを処理する必要があります。

Protocol Buffersには大規模なデータセットをサポートする組み込み機能が含まれていないため、異なる状況には異なる解決策が求められます。時には単純なレコードのリストが適している場合もありますが、他の場合にはデータベースのようなものが必要になることもあります。各解決策は別々のライブラリとして開発されるべきであり、必要な人だけがそのコストを支払う必要があります。

## 自己記述メッセージ {#self-description}

Protocol Buffersには自身の型の説明が含まれていないため、対応する`.proto`ファイルで型を定義していない生のメッセージだけが与えられた場合、有用なデータを抽出することが難しいです。

ただし、.protoファイルの内容自体もプロトコルバッファを使用して表現することができます。ソースコードパッケージ内の`src/google/protobuf/descriptor.proto`ファイルは関連するメッセージ型を定義しています。`protoc`は`--descriptor_set_out`オプションを使用して、.protoファイルのセットを表す`FileDescriptorSet`を出力できます。これにより、次のように自己記述プロトコルメッセージを定義できます：

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

以上のように、この機能がProtocol Bufferライブラリに含まれていない理由は、Google内でそのような機能が必要とされたことがないからです。

このテクニックには、記述子を使用した動的メッセージのサポートが必要です。自己記述メッセージを使用する前に、プラットフォームがこの機能をサポートしているか確認してください。
