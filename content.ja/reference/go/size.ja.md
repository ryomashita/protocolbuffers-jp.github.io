[`proto.Size`](https://pkg.go.dev/google.golang.org/protobuf/proto#Size) 関数は、proto.Message のワイヤーフォーマットエンコーディングのサイズ（バイト単位）を返します。これは、すべてのフィールド（サブメッセージを含む）をトラバースして計算されます。

特に、これは **Go Protobuf がメッセージをエンコードする方法のサイズ** を返します。

## 典型的な使用法

### 空のメッセージの識別

[`proto.Size`](https://pkg.go.dev/google.golang.org/protobuf/proto#Size) が 0 を返すかどうかをチェックすることは、空のメッセージを認識する簡単な方法です:

```go
if proto.Size(m) == 0 {
    // No fields set (or, in proto3, all fields matching the default);
    // skip processing this message, or return an error, or similar.
}
```

### サイズ制限付きのプログラム出力

別のシステムに作業タスクを生成するバッチ処理パイプラインを書いているとします。この例では、その別のシステムを「ダウンストリームシステム」と呼びます。ダウンストリームシステムは小〜中規模のタスクを処理するようにプロビジョニングされていますが、負荷テストでは、500 MB を超える作業タスクを提示するとシステムが連鎖的な障害に直面することがわかりました。

最善の修正方法は、ダウンストリームシステムに保護を追加することです（詳細は https://cloud.google.com/blog/products/gcp/using-load-shedding-to-survive-a-success-disaster-cre-life-lessons を参照）。ただし、負荷分散を実装することが不可能な場合、パイプラインにクイックフィックスを追加することを決定することもできます:

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

## 誤った使用法: Unmarshal との関連がない

[`proto.Size`](https://pkg.go.dev/google.golang.org/protobuf/proto#Size) は、Go Protobuf がメッセージをエンコードする方法のバイト数を返すため、入力された Protobuf メッセージのストリームをアンマーシャリング（デコード）する際に `proto.Size` を使用することは安全ではありません:

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

`data` が [非最小ワイヤーフォーマット](#non-minimal) のメッセージを含む場合、`proto.Size` は実際にアンマーシャリングされたサイズと異なるサイズを返す可能性があり、解析エラー（最善の場合）または最悪の場合は誤って解析されたデータが発生します。

したがって、この例は、すべての入力メッセージが（同じバージョンの）Go Protobufによって生成された限りにおいてのみ信頼性があります。これは驚くべきことであり、おそらく意図されていないものです。

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

遅延デコードが有効になっている場合、`proto.Size` は `proto.Marshal`（および `proto.MarshalAppend` のような変種）が書き込るよりも多くのバイトを返す可能性があります！そのため、エンコードされたバイトをワイヤー（またはディスク）に配置する場合は、`len(buf)` で作業し、以前の `proto.Size` の結果を破棄してください。

具体的には、(サブ-)メッセージは `proto.Size` と `proto.Marshal` の間で「縮小」する可能性があります:

1. 遅延デコードが有効になっている
2. メッセージが [非最小ワイヤー形式](#non-minimal) で到着している
3. `proto.Size` が呼び出される前にメッセージがアクセスされていないため、まだデコードされていない
4. `proto.Size` の後（ただし `proto.Marshal` の前）にメッセージがアクセスされ、それによって遅延デコードされる

デコードにより、後続の `proto.Marshal` 呼び出しはメッセージをエンコードし（単にワイヤー形式をコピーするのではなく）、Go がメッセージをエンコードする方法に暗黙的に正規化され、現在は最小のワイヤー形式になります（ただし、それに依存しないでください！）。

このシナリオはかなり特定されていることがわかりますが、それでも **`proto.Size` の結果を上限として扱い、実際にエンコードされたメッセージのサイズと一致するとは決して仮定しない** のが最善の方法です。

## 背景: 非最小ワイヤー形式 {#non-minimal}

Protobuf メッセージをエンコードする際、1 つの *最小ワイヤー形式サイズ* と、同じメッセージにデコードされる複数の大きな *非最小ワイヤー形式* があります。

非最小ワイヤー形式（時には「非正規化ワイヤー形式」とも呼ばれる）は、非繰り返しフィールドが複数回現れる、最適でない varint エンコーディング、ワイヤー上で非パックされたように見えるパックされた繰り返しフィールドなどのシナリオを指します。

異なるシナリオで非最小のワイヤ形式に遭遇することがあります：

*   **意図的に。** Protobuf は、メッセージを連結してワイヤ形式を連結することをサポートしています。
*   **偶然に。** （おそらくサードパーティの）Protobuf エンコーダが理想的にエンコードされない場合（例：varint をエンコードする際に必要以上のスペースを使用する場合）。
*   **悪意を持って。** 攻撃者は、ネットワーク上でクラッシュを引き起こすために特定の Protobuf メッセージを作成する可能性があります。
