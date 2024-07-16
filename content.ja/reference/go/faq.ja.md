## バージョン

### `github.com/golang/protobuf` と `google.golang.org/protobuf` の違いは何ですか？ {#modules}

[`github.com/golang/protobuf`](https://pkg.go.dev/github.com/golang/protobuf?tab=overview) モジュールは、元のGoプロトコルバッファAPIです。

[`google.golang.org/protobuf`](https://pkg.go.dev/google.golang.org/protobuf?tab=overview) モジュールは、このAPIの更新版であり、シンプルさ、使いやすさ、安全性を考慮して設計されています。更新されたAPIの主要な機能は、リフレクションのサポートと、ユーザー向けAPIと基礎実装の分離です。

新しいコードでは `google.golang.org/protobuf` を使用することを推奨します。

`github.com/golang/protobuf` のバージョン `v1.4.0` 以降は、新しい実装をラップし、プログラムが新しいAPIを段階的に採用できるようにします。たとえば、`github.com/golang/protobuf/ptypes` で定義された well-known types は、単に新しいモジュールで定義されたもののエイリアスです。したがって、[`google.golang.org/protobuf/types/known/emptypb`](https://pkg.go.dev/google.golang.org/protobuf/types/known/emptypb) と [`github.com/golang/protobuf/ptypes/empty`](https://pkg.go.dev/github.com/golang/protobuf/ptypes/empty) は互換性があります。

### `proto1`、`proto2`、`proto3` とは何ですか？ {#proto-versions}

これらはプロトコルバッファの *言語* のリビジョンです。これは、Goのプロトコルバッファの *実装* とは異なります。

*   `proto3` は現在の言語のバージョンです。これは最も一般的に使用される言語のバージョンです。新しいコードでは proto3 を使用することを推奨します。

*   `proto2` は古い言語のバージョンです。proto3 に置き換えられましたが、proto2 は完全にサポートされています。

*   `proto1` は古い言語のバージョンであり、オープンソースとしてリリースされたことはありません。

### いくつかの異なる `Message` タイプがあります。どれを使用すべきですか？ {#message-types}

*   [`"google.golang.org/protobuf/proto".Message`](https://pkg.go.dev/google.golang.org/protobuf/proto?tab=doc#Message) は、現在のプロトコルバッファコンパイラによって生成されたすべてのメッセージによって実装されるインターフェース型です。[`proto.Marshal`](https://pkg.go.dev/google.golang.org/protobuf/proto?tab=doc#Marshal) や [`proto.Clone`](https://pkg.go.dev/google.golang.org/protobuf/proto?tab=doc#Clone) など、任意のメッセージに対して操作を行う関数は、この型を受け入れるか返します。

*   [`"google.golang.org/protobuf/reflect/protoreflect".Message`](https://pkg.go.dev/google.golang.org/protobuf/reflect/protoreflect?tab=doc#Message)
    は、メッセージの反射ビューを記述するインターフェース型です。

    `proto.Message` 上で `ProtoReflect` メソッドを呼び出すと、`protoreflect.Message` を取得できます。

*   [`"google.golang.org/protobuf/reflect/protoreflect".ProtoMessage`](https://pkg.go.dev/google.golang.org/protobuf/reflect/protoreflect?tab=doc#ProtoMessage)
    は、`"google.golang.org/protobuf/proto".Message` のエイリアスです。これらの2つの型は互換性があります。

*   [`"github.com/golang/protobuf/proto".Message`](https://pkg.go.dev/github.com/golang/protobuf/proto?tab=doc#Message)
    は、古い Go プロトコルバッファ API で定義されたインターフェース型です。すべての生成されたメッセージ型はこのインターフェースを実装しますが、このインターフェースはこれらのメッセージから期待される動作を記述していません。新しいコードではこの型の使用を避けるべきです。

## 一般的な問題

### "`go install`": `working directory is not part of a module` {#working-directory}

Go 1.15 以前では、環境変数 `GO111MODULE=on` を設定し、`go install` コマンドをモジュールディレクトリの外で実行しています。`GO111MODULE=auto` を設定するか、環境変数をアンセットしてください。

Go 1.16 以降では、明示的なバージョンを指定してモジュールの外で `go install` を実行できます: `go install google.golang.org/protobuf/cmd/protoc-gen-go@latest`

### `constant -1 overflows protoimpl.EnforceVersion` {#enforce-version}

新しいバージョンの `"google.golang.org/protobuf"` モジュールが必要な生成された `.pb.go` ファイルを使用しています。

次のコマンドで新しいバージョンに更新してください:

```shell
go get -u google.golang.org/protobuf/proto
```

### `undefined: "github.com/golang/protobuf/proto".ProtoPackageIsVersion4` {#enforce-version-apiv1}

新しいバージョンの `"github.com/golang/protobuf"` モジュールが必要な生成された `.pb.go` ファイルを使用しています。

次のコマンドで新しいバージョンに更新してください:

```shell
go get -u github.com/golang/protobuf/proto
```

### プロトコルバッファの名前空間の競合とは何ですか？ {#namespace-conflict}

すべてのプロトコルバッファ宣言は、Goバイナリにリンクされたグローバルレジストリに挿入されます。

すべてのプロトコルバッファ宣言（たとえば、列挙型、列挙値、またはメッセージ）には、絶対名があります。これは、[パッケージ名](/programming-guides/proto2#packages)と`.proto`ソースファイル内の宣言の相対名の連結です（たとえば、`my.proto.package.MyMessage.NestedMessage`）。プロトコルバッファ言語は、すべての宣言が普遍的に一意であると想定しています。

Goバイナリにリンクされた2つのプロトコルバッファ宣言が同じ名前を持つ場合、これは名前空間の競合を引き起こし、レジストリがその宣言を名前で適切に解決することが不可能です。使用されているGo protobufsのバージョンに応じて、これは初期化時にパニックを引き起こすか、競合を無視してランタイム中に潜在的なバグを引き起こす可能性があります。

### プロトコルバッファの名前空間の競合を修正するには？ {#fix-namespace-conflict}

名前空間の競合を最良に修正する方法は、競合が発生している理由によって異なります。

名前空間の競合が発生する一般的な方法：

*   **ベンダー提供の.protoファイル。** 1つの`.proto`ファイルが2つ以上のGoパッケージに生成され、同じGoバイナリにリンクされると、生成されたGoパッケージ内のすべてのプロトコルバッファ宣言で競合が発生します。これは通常、`.proto`ファイルがベンダリングされ、それからGoパッケージが生成される場合、または生成されたGoパッケージ自体がベンダリングされる場合に発生します。ユーザーはベンダリングを避け、その`.proto`ファイルに対して集中化されたGoパッケージに依存することをお勧めします。

    *   もし`.proto`ファイルが外部の団体によって所有されており、`go_package`オプションが欠けている場合、その`.proto`ファイルの所有者と協力して、複数のユーザーが依存できる集中化されたGoパッケージを指定するようにする必要があります。

*   **欠落または一般的なprotoパッケージ名。** もし`.proto`ファイルがパッケージ名を指定していないか、あるいは非常に一般的なパッケージ名を使用している場合（たとえば、"my_service"など）、そのファイル内の宣言が他の場所での宣言と競合する可能性が高いです。私たちは、すべての`.proto`ファイルに、普遍的に一意であるように故意に選択されたパッケージ名があることをお勧めします（たとえば、会社名で接頭辞を付けるなど）。

    *   **警告:** `.proto` ファイルでパッケージ名を後から変更すると、拡張フィールドとして使用される型、`google.protobuf.Any` に格納される型、または gRPC サービス定義に対して後方互換性がありません。

Starting with v1.26.0 of the `google.golang.org/protobuf` module, a hard error
will be reported when a Go program starts up that has multiple conflicting
protobuf names linked into it. While it is preferable that the source of the
conflict be fixed, the fatal error can be immediately worked around in one of
two ways:

1.  **コンパイル時に。** 衝突の処理方法のデフォルト動作は、リンカー初期化変数を使用してコンパイル時に指定できます: `go build
    -ldflags "-X
    google.golang.org/protobuf/reflect/protoregistry.conflictPolicy=warn"`

2.  **プログラムの実行時に。** 特定の Go バイナリを実行する際の衝突の処理方法は、環境変数を使用して設定できます: `GOLANG_PROTOBUF_REGISTRATION_CONFLICT=warn ./main`

### `reflect.DeepEqual` はなぜ protobuf メッセージで予期しない動作をするのですか？ {#deepequal}

生成されたプロトコルバッファメッセージ型には、同等のメッセージ間でも異なる内部状態が含まれることがあります。

さらに、`reflect.DeepEqual` 関数はプロトコルバッファメッセージのセマンティクスを認識しておらず、存在しない差異を報告することがあります。たとえば、`nil` マップを含むフィールドと、ゼロ長で非 `nil` マップを含むフィールドはセマンティック的に等価ですが、`reflect.DeepEqual` では等しくないと報告されます。

メッセージの値を比較するには、[`proto.Equal`](https://pkg.go.dev/google.golang.org/protobuf/proto#Equal) 関数を使用してください。

テストでは、[`"github.com/google/go-cmp/cmp"`](https://pkg.go.dev/github.com/google/go-cmp/cmp?tab=doc) パッケージを使用して、[`protocmp.Transform()`](https://pkg.go.dev/google.golang.org/protobuf/testing/protocmp#Transform) オプションを使用できます。`cmp` パッケージは任意のデータ構造を比較でき、[`cmp.Diff`](https://pkg.go.dev/github.com/google/go-cmp/cmp#Diff) は値間の差異の人間が読めるレポートを生成します。

```go
if diff := cmp.Diff(a, b, protocmp.Transform()); diff != "" {
  t.Errorf("予期しない差異があります:\n%v", diff)
}
```

## ハイラムの法則

### ハイラムの法則とは何か、そしてなぜこのFAQにあるのか？ {#hyrums-law}

[ハイラムの法則](https://www.hyrumslaw.com/) は次のように述べています：

> APIの利用者が十分にいる場合、契約で約束したことは何であれ、システムのすべての観測可能な振る舞いは誰かに依存されることになります。

GoプロトコルバッファAPIの最新バージョンの設計目標は、将来安定して保証できない観測可能な振る舞いを提供することを可能な限り避けることです。私たちの哲学は、約束をしない領域での意図的な不安定性が、安定性の幻想を与えるよりも、将来においてプロジェクトがその誤った前提に長く依存していた場合に、より良いと考えています。

### エラーメッセージのテキストがなぜ変わり続けるのですか？ {#unstable-errors}

エラーメッセージのテキストに依存するテストは壊れやすく、そのテキストが変更されると頻繁に壊れます。エラーテキストの安全でない使用を desu 、このモジュールが生成するエラーのテキストは意図的に不安定です。

[`protobuf`](https://pkg.go.dev/mod/google.golang.org/protobuf) モジュールによって生成されたエラーを識別する必要がある場合、すべてのエラーが [`proto.Error`](https://pkg.go.dev/google.golang.org/protobuf/proto?tab=doc#Error) に一致することを保証します。

### [`protojson`](https://pkg.go.dev/google.golang.org/protobuf/encoding/protojson) の出力がなぜ変わり続けるのですか？ {#unstable-json}

Goのプロトコルバッファの[JSON形式](/programming-guides/proto3#json)の実装の長期的な安定性については何も約束しません。仕様は有効なJSONを指定していますが、マーシャラーが特定のメッセージを*正確に*どのようにフォーマットすべきかについての*標準的な*フォーマットの仕様は提供していません。出力が安定しているという幻想を与えるのを避けるために、わずかな違いを意図的に導入して、バイト単位の比較が失敗する可能性が高くなるようにしています。
```

### [`prototext`](https://pkg.go.dev/google.golang.org/protobuf/encoding/prototext) の出力がなぜ変わり続けるのか {#unstable-text}

Goのテキスト形式の実装の長期的な安定性については何も約束していません。protobufのテキスト形式には公式の仕様がなく、将来的に`prototext`パッケージの出力を改善できるようにしたいと考えています。パッケージの出力の安定性を保証しないため、ユーザーがそれに依存することを避けるために意図的に不安定性を導入しています。

出力の安定性を確保するために、`prototext`の出力を[`txtpbfmt`](https://github.com/protocolbuffers/txtpbfmt)プログラムを通すことをお勧めします。このフォーマッタは、Goで直接呼び出すことができます。[`parser.Format`](https://pkg.go.dev/github.com/protocolbuffers/txtpbfmt/parser?tab=doc#Format)を使用してください。

## その他

### プロトコルバッファメッセージをハッシュキーとして使用する方法は？ {#hash}

プロトコルバッファメッセージのマーシャリングされた出力が時間の経過とともに安定であることを保証する正準シリアライゼーションが必要です。残念ながら、正準シリアライゼーションの仕様は現時点では存在しません。独自の方法を作成するか、それを必要としない方法を見つける必要があります。

### Goプロトコルバッファ実装に新しい機能を追加できますか？ {#new-feature}

おそらくです。提案は常に歓迎していますが、新しい機能を追加する際には非常に慎重です。

Goのプロトコルバッファの実装は、他の言語の実装と一貫性を保つことを目指しています。そのため、Goに特化しすぎた機能は避ける傾向があります。Go固有の機能は、プロトコルバッファが言語中立のデータ交換形式であるという目標を妨げます。

あなたのアイデアがGoの実装に特化していない限り、[protobufディスカッショングループ](http://groups.google.com/group/protobuf)に参加してそこで提案してください。

Goの実装に関するアイデアがある場合は、当社のイシュートラッカーにイシューを報告してください：
[https://github.com/golang/protobuf/issues](https://github.com/golang/protobuf/issues)

### `Marshal` または `Unmarshal` にカスタマイズオプションを追加できますか？ {#new-marshal-option}
他の実装（例：C++、Java）にそのオプションが存在する場合のみです。プロトコルバッファ（バイナリ、JSON、およびテキスト）のエンコーディングは、実装間で一貫している必要があります。そのため、ある言語で書かれたプログラムが他の言語で書かれたメッセージを読むことができます。

Go の実装には、少なくとも他のサポートされている実装の中で同等のオプションが存在する場合を除き、`Marshal` 関数によって出力されるデータに影響を与えるオプションを追加しませんし、`Unmarshal` 関数によって読まれるデータに影響を与えるオプションを追加しません。

### `protoc-gen-go` によって生成されたコードをカスタマイズできますか？ {#custom-code}
一般的には、いいえ。プロトコルバッファは言語に依存しないデータ交換形式であり、実装固有のカスタマイズはその意図に反します。
