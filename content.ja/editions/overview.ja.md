+++
title = "Protobuf Editions の概要"
linkTitle = "概要"
weight = 42
description = "Protobuf Editions 機能の概要。"
type = "docs"
+++

Protobuf Editions は、Protocol Buffers で使用してきた proto2 および proto3 の指定を置き換えます。proto 定義ファイルの先頭に `syntax = "proto2"` または `syntax = "proto3"` を追加する代わりに、`edition = "2023"` のようなエディション番号を使用して、ファイルが持つデフォルトの動作を指定します。エディションは、言語を時間の経過とともに段階的に進化させることを可能にします。

古いバージョンが持っていたハードコードされた動作の代わりに、エディションは、デフォルト値（動作）を持つ [機能](/editions/features) のコレクションを表します。機能は、ファイル、メッセージ、フィールド、列挙型などのオプションであり、protoc、コードジェネレータ、および protobuf ランタイムの動作を指定します。選択したエディションのデフォルト動作と一致しない場合、ファイル、メッセージ、フィールドなどの異なるレベルで動作を明示的にオーバーライドすることができます。また、オーバーライドした内容を再度オーバーライドすることもできます。このトピックの後半にある [レキシカルスコープに関するセクション](#scoping) では、これについて詳しく説明します。

*最新リリースのエディションは 2023 です。*

## 機能のライフサイクル {#lifecycles}

エディションは、機能のライフサイクルの基本的な増分を提供します。機能には期待されるライフサイクルがあります：導入、デフォルト動作の変更、非推奨化、そして削除。例えば：

1.  エディション 2031 は、`feature.amazing_new_feature` をデフォルト値 `false` で作成します。この値は、すべての以前のエディションと同じ動作を維持します。つまり、影響を与えないようにデフォルトで設定されています。

2.  開発者は、.proto ファイルを `edition = "2031"` に更新します。

3.  エディション 2033 などの後のエディションでは、`feature.amazing_new_feature` のデフォルトを `false` から `true` に切り替えます。これは、すべての proto にとって望ましい動作であり、protobuf チームがその機能を作成した理由です。

    Prototiller ツールを使用して、.proto ファイルの以前のバージョンをエディション 2033 に移行すると、以前の動作を維持するために必要な明示的な `feature.amazing_new_feature = false` エントリが追加されます。開発者は、新しい動作を適用したい場合には、これらの新しく追加された設定を削除します。

4.  ある時点で、`feature.amazing_new_feature` はある版で非推奨とマークされ、後の版で削除されます。

    機能が削除されると、その動作のためのコード生成器やそれをサポートするランタイムライブラリも削除されるかもしれません。ただし、タイムラインは寛大になります。ライフサイクルの先のステップの例に従うと、非推奨になるのは2034年版であるかもしれませんが、削除されるのはその後の2036年版であり、おおよそ2年後です。機能を削除すると、常にメジャーバージョンの更新が開始されます。

このライフサイクルのため、非推奨の機能を使用していない`.proto`ファイルは、次の版へのアップグレードがノーオペレーションであることになります。
Googleの移行ウィンドウと非推奨ウィンドウを利用して、コードをアップグレードする時間があります。

先行するライフサイクルの例では、機能に対してブール値を使用していましたが、機能は列挙型を使用することもできます。たとえば、`features.field_presence` は `LEGACY_REQUIRED`、`EXPLICIT`、`IMPLICIT` の値を持ちます。

## Protobuf Editions への移行 {#migrating}

Editions は既存のバイナリを壊さず、メッセージのバイナリ、テキスト、JSON シリアル化形式を変更しません。最初の版はできるだけ最小限の変更となります。最初の版はベースラインを確立し、proto2 と proto3 の定義を新しい単一の定義形式に統合します。

次の版がリリースされると、機能のデフォルト動作が変更されるかもしれません。Prototiller を使用して、.proto ファイルのノーオペレーション変換を行うか、新しい動作のいくつかまたはすべてを受け入れるかを選択できます。Editions はおおよそ1年に1度リリースされる予定です。

### Proto2 から Editions への移行 {#proto2-migration}

このセクションでは、proto2 ファイルを示し、Prototiller ツールを実行して定義ファイルを Protobuf Editions 構文を使用するように変更した後のファイルを示します。

<section class="tabs">

#### Proto2 構文 {.new-tab}

```proto
// proto2 file
syntax = "proto2";

package com.example;

message Player {
  // in proto2, optional fields have explicit presence
  optional string name = 1;
  // proto2 still supports the problematic "required" field rule
  required int32 id = 2;
  // in proto2 this is not packed by default
  repeated int32 scores = 3;

  enum Handed {
    HANDED_UNSPECIFIED = 0;
    HANDED_LEFT = 1;
    HANDED_RIGHT = 2;
    HANDED_AMBIDEXTROUS = 3;
  }

  // in proto2 enums are closed
  optional Handed handed = 4;

  reserved "gender";
}
```

#### Editions 構文 {.new-tab}

```proto
// Edition version of proto2 file
edition = "2023";

package com.example;

message Player {
  // fields have explicit presence, so no explicit setting needed
  string name = 1;
  // to match the proto2 behavior, LEGACY_REQUIRED is set at the field level
  int32 id = 2 [features.field_presence = LEGACY_REQUIRED];
  // to match the proto2 behavior, EXPANDED is set at the field level
  repeated int32 scores = 3 [features.repeated_field_encoding = EXPANDED];

  enum Handed {
    // this overrides the default edition 2023 behavior, which is OPEN
    option features.enum_type = CLOSED;
    HANDED_UNSPECIFIED = 0;
    HANDED_LEFT = 1;
    HANDED_RIGHT = 2;
    HANDED_AMBIDEXTROUS = 3;
  }

  Handed handed = 4;

  reserved gender;
}
```

</section>

### Proto3からEditionsへの移行 {#proto3-migration}

このセクションでは、proto3ファイルを示し、定義ファイルをProtobuf Editions構文を使用するように変更するためにPrototillerツールを実行した後の見た目を示します。

<section class="tabs">

#### Proto3構文 {.new-tab}

```proto
// proto3 file
syntax = "proto3";

package com.example;

message Player {
  // in proto3, optional fields have explicit presence
  optional string name = 1;
  // in proto3 no specified field rule defaults to implicit presence
  int32 id = 2;
  // in proto3 this is packed by default
  repeated int32 scores = 3;

  enum Handed {
    HANDED_UNSPECIFIED = 0;
    HANDED_LEFT = 1;
    HANDED_RIGHT = 2;
    HANDED_AMBIDEXTROUS = 3;
  }

  // in proto3 enums are open
  optional Handed handed = 4;

  reserved "gender";
}
```

#### Editions構文 {.new-tab}

```proto
// Editions version of proto3 file
edition = "2023";

package com.example;

message Player {
  // fields have explicit presence, so no explicit setting needed
  string name = 1;
  // to match the proto3 behavior, IMPLICIT is set at the field level
  int32 id = 2 [features.field_presence = IMPLICIT];
  // PACKED is the default state, and is provided just for illustration
  repeated int32 scores = 3 [features.repeated_field_encoding = PACKED];

  enum Handed {
    HANDED_UNSPECIFIED = 0;
    HANDED_LEFT = 1;
    HANDED_RIGHT = 2;
    HANDED_AMBIDEXTROUS = 3;
  }

  Handed handed = 4;

  reserved gender;
}
```

</section>

<a name="inheritance"></a>

### レキシカルスコープ {#scoping}

Editions構文は、レキシカルスコープをサポートし、各機能の許可されたターゲットのリストを持っています。たとえば、最初のエディションでは、機能はファイルレベルまたは最も細かい粒度のレベルで指定できます。レキシカルスコープの実装により、ファイル全体で機能のデフォルト動作を設定し、その動作をメッセージ、フィールド、列挙型、列挙値、oneof、サービス、またはメソッドレベルでオーバーライドできます。より高いレベル（ファイル、メッセージ）で行われた設定は、同じスコープ内（フィールド、列挙値）で設定が行われていない場合に適用されます。明示的に設定されていない機能は、.protoファイルで使用されているエディションバージョンで定義された動作に準拠します。

次のコードサンプルは、ファイル、フィールド、および列挙型レベルでいくつかの機能が設定されているのを示しています。設定は、ハイライトされた行にあります:

```proto {highlight="lines:3,7,16"}
edition = "2023";

option features.enum_type = CLOSED;

message Person {
  string name = 1;
  int32 id = 2 [features.field_presence = IMPLICIT];

  enum Pay_Type {
    PAY_TYPE_UNSPECIFIED = 1;
    PAY_TYPE_SALARY = 2;
    PAY_TYPE_HOURLY = 3;
  }

  enum Employment {
    option features.enum_type = OPEN;
    EMPLOYMENT_UNSPECIFIED = 0;
    EMPLOYMENT_FULLTIME = 1;
    EMPLOYMENT_PARTTIME = 2;
  }
  Employment employment = 4;
}
```

前述の例では、存在機能が`IMPLICIT`に設定されています。設定されていない場合は`EXPLICIT`にデフォルトでなります。`Pay_Type` `enum`はファイルレベルの設定を適用するため、`CLOSED`になります。ただし、`Employment` `enum`は列挙型内で設定されているため、`OPEN`になります。

### Prototiller {#prototiller}

移行ガイドと移行ツールを提供しており、Editions間の移行を容易にします。Prototillerと呼ばれるこのツールを使用すると、次のことができます:

*   新しいEditions構文に基づいたproto2およびproto3定義ファイルをスケールで変換する
*   1つのエディションから別のエディションにファイルを移行する
*   他の方法でprotoファイルを操作する

### 互換性 {#compatibility}

Protobuf Editionsをできるだけ少ない影響で構築しています。たとえば、proto2およびproto3の定義をEditionsベースの定義ファイルにインポートしたり、その逆も可能です:

```proto
// file myproject/foo.proto
syntax = "proto2";

enum Employment {
  EMPLOYMENT_UNSPECIFIED = 0;
  EMPLOYMENT_FULLTIME = 1;
  EMPLOYMENT_PARTTIME = 2;
}
```

```proto
// file myproject/edition.proto
edition = "2023";

import "myproject/foo.proto";
```

proto2 または proto3 から editions に移行すると生成されるコードが変わりますが、
ワイヤーフォーマットは変わりません。editions 構文の proto 定義を使用しても、
proto2 および proto3 のデータファイルやファイルストリームにアクセスできます。
```
