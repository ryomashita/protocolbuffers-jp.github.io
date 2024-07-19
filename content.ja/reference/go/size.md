
+++
title = "Goサイズセマンティクス"
weight = 630
linkTitle = "サイズセマンティクス"
description = "proto.Sizeの使用方法（および非使用方法）を説明します"
type = "docs"
+++

[`proto.Size`](https://pkg.go.dev/google.golang.org/protobuf/proto#Size) 関数は、proto.Messageのワイヤーフォーマットエンコーディングのバイトサイズを返します。これは、すべてのフィールド（サブメッセージを含む）をトラバースします。

特に、これは **Go Protobufがメッセージをエンコードする方法のサイズ** を返します。

## 典型的な使用法

### 空のメッセージの識別

[`proto.Size`](https://pkg.go.dev/google.golang.org/protobuf/proto#Size) が 0 を返すかどうかをチェックすることは、空のメッセージを認識する簡単な方法です:

```go
if proto.Size(m) == 0 {
    // No fields set (or, in proto3, all fields matching the default);
    // skip processing this message, or return an error, or similar.
}
```

### サイズ制限付きプログラムの出力

別のシステムに作業タスクを生成するバッチ処理パイプラインを書いているとします。この例では、この別のシステムを「ダウンストリームシステム」と呼びます。ダウンストリームシステムは、小〜中規模のタスクを処理するようにプロビジョニングされていますが、負荷テストでは、500 MBを超える作業タスクを提示すると、システムが連鎖的な障害に直面することがわかりました。

最善の修正方法は、ダウンストリームシステムに保護を追加することです（https://cloud.google.com/blog/products/gcp/using-load-shedding-to-survive-a-success-disaster-cre-life-lessons）。ただし、ロードシェッディングの実装が不可能な場合、パイプラインにクイックフィックスを追加することを決定することができます:

```go {highlight="context:1,proto.Size,1"}
func (*beamFn) ProcessElement(key string, value []byte, emit func(proto.Message)) {
  task := produceWorkTask(value)
  if proto.Size(task) > 100 * 1024 * 1024 {
    // Skip every work task over 100 MB to not overwhelm
    // the brittle downstream system.
    return
  }
  emit(task)
}
```

## 不正な使用法: Unmarshalとの関連がない

[`proto.Size`](https://pkg.go.dev/google.golang.org/protobuf/proto#Size) は、Go Protobufがメッセージをエンコードする方法のバイト数を返すため、ストリームからの入力されたProtobufメッセージをアンマーシャリング（デコード）する際に `proto.Size` を使用することは安全ではありません:

```go {highlight="context:1,proto.Size,1"}
func bytesToSubscriptionList(data []byte) ([]*vpb.EventSubscription, error) {
    subList := []*vpb.EventSubscription{}
    for len(data) > 0 {
        subscription := &vpb.EventSubscription{}
        if err := proto.Unmarshal(data, subscription); err != nil {
            return nil, err
        }
        subList = append(subList, subscription)
        data = data[:len(data)-proto.Size(subscription)]
    }
    return subList, nil
}
```

`data` が [非最小ワイヤーフォーマット](#non-minimal) のメッセージを含む場合、`proto.Size` は実際にアンマーシャリングされたサイズと異なるサイズを返す可能性があり、パースエラー（最善の場合）または最悪の場合は誤って解析されたデータが発生します。

したがって、この例は、すべての入力メッセージが（同じバージョンの）Go Protobufによって生成された限り、信頼性があります。これは驚くべきことであり、おそらく意図されていないものです。
```

**ヒント:** 
[`protodelim` パッケージ](https://pkg.go.dev/google.golang.org/protobuf/encoding/protodelim)
を使用して、Protobuf メッセージのサイズ区切りストリームを読み書きすることができます。

## 高度な使用法: バッファの事前サイズ指定

[`proto.Size`](https://pkg.go.dev/google.golang.org/protobuf/proto#Size) の高度な使用法は、マーシャリング前にバッファに必要なサイズを決定することです:

```go
opts := proto.MarshalOptions{
    // Possibly avoid an extra proto.Size in Marshal itself (see docs):
    UseCachedSize: true,
}
// DO NOT SUBMIT without implementing this Optimization opportunity:
// instead of allocating, grab a sufficiently-sized buffer from a pool.
// Knowing the size of the buffer means we can discard
// outliers from the pool to prevent uncontrolled
// memory growth in long-running RPC services.
buf := make([]byte, 0, opts.Size(m))
var err error
buf, err = opts.MarshalAppend(buf, m) // does not allocate
// Note that len(buf) might be less than cap(buf)! Read below:
```

注意: 遅延デコードが有効になっている場合、`proto.Size` は `proto.Marshal`（および `proto.MarshalAppend` などのバリアント）が書き込るよりも多くのバイトを返す可能性があります！そのため、エンコードされたバイトをワイヤー（またはディスク）に配置する場合は、`len(buf)` で作業し、以前の `proto.Size` の結果を破棄してください。

具体的には、(サブ-)メッセージは `proto.Size` と `proto.Marshal` の間で「縮小」する可能性があります:

1. 遅延デコードが有効になっている
2. メッセージが [非最小ワイヤー形式](#non-minimal) で到着した
3. `proto.Size` が呼び出される前にメッセージにアクセスされず、まだデコードされていない
4. `proto.Size` の後（ただし `proto.Marshal` の前）にメッセージにアクセスされ、遅延デコードされる

デコードにより、後続の `proto.Marshal` 呼び出しがメッセージをエンコードし（単にワイヤー形式をコピーするのではなく）、Go がメッセージをエンコードする方法に暗黙的に正規化され、現在は最小のワイヤー形式になります（ただし、それに依存しないでください！）。

このシナリオはかなり特定されていますが、それでも **`proto.Size` の結果を上限として扱い、実際にエンコードされたメッセージのサイズと一致するとは決して仮定しない** のが最善の方法です。

## 背景: 非最小ワイヤー形式 {#non-minimal}

Protobuf メッセージをエンコードする際、1 つの *最小ワイヤー形式サイズ* と、同じメッセージにデコードされる複数の大きな *非最小ワイヤー形式* があります。

非最小ワイヤー形式（時には「非正規化ワイヤー形式」とも呼ばれる）は、非繰り返しフィールドが複数回現れる、最適でない varint エンコーディング、パックされた繰り返しフィールドがワイヤー上で非パック状態に見えるなどのシナリオを指します。

異なるシナリオで非最小ワイヤ形式に遭遇することがあります：

*   **意図的に。** Protobuf は、メッセージを連結することでワイヤ形式を連結することをサポートしています。
*   **偶然に。** （おそらくサードパーティの）Protobuf エンコーダが理想的にエンコードされていない場合（たとえば、varint をエンコードする際に必要以上のスペースを使用する場合）。
*   **悪意を持って。** 攻撃者は、ネットワーク上でクラッシュを引き起こすために特定の Protobuf メッセージを作成する可能性があります。
