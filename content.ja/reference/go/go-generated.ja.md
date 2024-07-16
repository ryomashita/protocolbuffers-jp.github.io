+++
title = "Go Generated Code Guide"
weight = 610
linkTitle = "生成されたコードガイド"
description = "任意のプロトコル定義に対してプロトコルバッファコンパイラが生成するGoコードを正確に説明します。"
type = "docs"
+++

proto2 と proto3 で生成されるコードの違いは強調されています - これらの違いは、このドキュメントで説明されている生成されたコードにあります。これは、両バージョンで同じであるベースAPIではありません。このドキュメントを読む前に、[proto2 言語ガイド](/programming-guides/proto2) および/または [proto3 言語ガイド](/programming-guides/proto3) を読む必要があります。

## コンパイラの呼び出し {#invocation}

プロトコルバッファコンパイラは、Goコードを生成するためにプラグインが必要です。Go 1.16 以上を使用して、次のコマンドを実行してインストールします:

```shell
go install google.golang.org/protobuf/cmd/protoc-gen-go@latest
```

これにより、`protoc-gen-go` バイナリが `$GOBIN` にインストールされます。インストール場所を変更するには、`$GOBIN` 環境変数を設定します。プロトコルバッファコンパイラがそれを見つけるためには、`$PATH` にある必要があります。

プロトコルバッファコンパイラは、`go_out` フラグを使用してGo出力を生成します。`go_out` フラグの引数は、コンパイラがGo出力を書き込むディレクトリです。各 `.proto` ファイル入力に対して、コンパイラは単一のソースファイルを作成します。出力ファイルの名前は、`.proto` 拡張子を `.pb.go` に置き換えることで作成されます。

生成された `.pb.go` ファイルが出力ディレクトリ内のどこに配置されるかは、コンパイラフラグに依存します。いくつかの出力モードがあります:

-   `paths=import` フラグが指定されている場合、出力ファイルは、Goパッケージのインポートパスに基づいたディレクトリに配置されます（たとえば、`.proto` ファイル内の `go_package` オプションで提供されるもの）。たとえば、Goインポートパスが `example.com/project/protos/fizz` である入力ファイル `protos/buzz.proto` は、出力ファイルが `example.com/project/protos/fizz/buzz.pb.go` になります。`paths` フラグが指定されていない場合、これがデフォルトの出力モードです。
-   `module=$PREFIX` フラグが指定されている場合、出力ファイルは、Goパッケージのインポートパスに基づいたディレクトリに配置されます（たとえば、`.proto` ファイル内の `go_package` オプションで提供されるもの）、ただし、指定されたディレクトリプレフィックスが出力ファイル名から削除されます。たとえば、Goインポートパスが `example.com/project/protos/fizz` であり、`example.com/project` が `module` プレフィックスとして指定されている場合、出力ファイルは `protos/fizz/buzz.pb.go` になります。モジュールパス外にGoパッケージを生成するとエラーが発生します。このモードは、生成されたファイルを直接Goモジュールに出力するために便利です。
-   `paths=source_relative` フラグが指定されている場合、出力ファイルは、入力ファイルと同じ相対ディレクトリに配置されます。たとえば、入力ファイル `protos/buzz.proto` は、出力ファイルが `protos/buzz.pb.go` になります。

フラグは、`protoc`を呼び出す際に`go_opt`フラグを渡すことで、`protoc-gen-go`に固有のものが提供されます。複数の`go_opt`フラグを渡すことができます。例えば、次のように実行します：

```shell
protoc --proto_path=src --go_out=out --go_opt=paths=source_relative foo.proto bar/baz.proto
```

コンパイラは、`src`ディレクトリ内の入力ファイル`foo.proto`と`bar/baz.proto`を読み取り、出力ファイル`foo.pb.go`と`bar/baz.pb.go`を`out`ディレクトリに書き込みます。必要に応じてコンパイラはネストされた出力サブディレクトリを自動的に作成しますが、出力ディレクトリ自体は作成しません。

## パッケージ {#package}

Goコードを生成するためには、各`.proto`ファイルに対してGoパッケージのインポートパスを指定する必要があります（`.proto`ファイルが生成される`.proto`ファイルに間接的に依存している場合も含む）。Goインポートパスを指定する方法は2つあります：

-   `.proto`ファイル内で宣言するか、
-   `protoc`を呼び出す際にコマンドラインで宣言するか。

`protoc`を呼び出す際にGoインポートパスを指定することをお勧めします。`.proto`ファイル内でGoパッケージを宣言することで、`.proto`ファイルのGoパッケージが`.proto`ファイル自体と一緒に一元的に識別され、`protoc`を呼び出す際に渡すフラグのセットを簡素化できます。`.proto`ファイルに対するGoインポートパスが`.proto`ファイル自体とコマンドラインの両方で提供される場合、後者が優先されます。

Goインポートパスは、`.proto`ファイル内で`go_package`オプションを使用してGoパッケージの完全なインポートパスを宣言することでローカルに指定されます。使用例：

```proto
option go_package = "example.com/project/protos/fizz";
```

コンパイラを呼び出す際にGoインポートパスを指定するには、1つ以上の`M${PROTO_FILE}=${GO_IMPORT_PATH}`フラグを渡すことで可能です。使用例：

```shell
protoc --proto_path=src \
  --go_opt=Mprotos/buzz.proto=example.com/project/protos/fizz \
  --go_opt=Mprotos/bar.proto=example.com/project/protos/foo \
  protos/buzz.proto protos/bar.proto
```

すべての`.proto`ファイルをそれぞれのGoインポートパスにマッピングすることは非常に大きな作業となるため、この方法でGoインポートパスを指定することは、通常、依存関係ツリー全体を制御するビルドツール（例：[Bazel](https://bazel.build/)）によって実行されます。特定の`.proto`ファイルに重複するエントリがある場合、最後に指定されたものが優先されます。

`go_package` オプションと `M` フラグの両方について、値にはセミコロンで区切られたインポートパスから明示的なパッケージ名を含めることができます。例: `"example.com/protos/foo;package_name"`。この使用法は推奨されていません。なぜなら、パッケージ名は通常、インポートパスから適切な方法でデフォルトで派生されるからです。

インポートパスは、1 つの `.proto` ファイルが別の `.proto` ファイルをインポートする場合に生成されるインポートステートメントを決定するために使用されます。例えば、`a.proto` が `b.proto` をインポートする場合、生成された `a.pb.go` ファイルは、生成された `b.pb.go` ファイルを含む Go パッケージをインポートする必要があります（両方のファイルが同じパッケージにない限り）。インポートパスは、出力ファイル名を構築するためにも使用されます。詳細については、上記の「Compiler Invocation」セクションを参照してください。

Go のインポートパスと `.proto` ファイル内の [`package` 指定子](/programming-guides/proto3#packages)との間には、相関関係はありません。後者は protobuf の名前空間にのみ関連し、前者は Go の名前空間にのみ関連します。また、Go のインポートパスと `.proto` のインポートパスとの間にも相関関係はありません。

## メッセージ {#message}

単純なメッセージ宣言が与えられた場合:

```proto
message Artist {}
```

プロトコルバッファコンパイラは `Artist` という構造体を生成します。`*Artist` は [`proto.Message`](https://pkg.go.dev/google.golang.org/protobuf/proto?tab=doc#Message) インターフェースを実装します。

[`proto` パッケージ](https://pkg.go.dev/google.golang.org/protobuf/proto?tab=doc) は、バイナリ形式への変換を含むメッセージに操作を行う関数を提供します。

`proto.Message` インターフェースは `ProtoReflect` メソッドを定義しています。このメソッドは、メッセージのリフレクションベースのビューを提供する [`protoreflect.Message`](https://pkg.go.dev/google.golang.org/protobuf/reflect/protoreflect?tab=doc#Message) を返します。

`optimize_for` オプションは、Go コードジェネレータの出力に影響を与えません。

### ネストされた型

メッセージは別のメッセージ内で宣言することができます。例:

```proto
message Artist {
  message Name {
  }
}
```

この場合、コンパイラは `Artist` と `Artist_Name` の 2 つの構造体を生成します。

## フィールド {#fields}

プロトコルバッファコンパイラは、メッセージ内で定義された各フィールドに対して構造体フィールドを生成します。このフィールドの正確な性質は、そのタイプと単数形、繰り返し形、マップ形、または oneof フィールドかどうかによって異なります。

生成された Go フィールド名は常にキャメルケースの命名規則を使用しますが、`.proto` ファイル内のフィールド名がアンダースコアを使用した小文字であっても
([それが望ましい形式](/programming-guides/style#message-field-names)である場合) です。大文字への変換は次のように機能します:

1.  最初の文字はエクスポート用に大文字化されます。最初の文字がアンダースコアの場合、それは削除され、大文字の X が前に付加されます。
2.  アンダースコアの後に小文字の文字が続く場合、アンダースコアは削除され、次の文字が大文字化されます。

したがって、proto フィールド `birth_year` は Go では `BirthYear` となり、
`_birth_year_2` は `XBirthYear_2` となります。

### 単一スカラーフィールド (proto2) {#singular-scalar-proto2}

次のいずれかのフィールド定義の場合:

```proto
optional int32 birth_year = 1;
required int32 birth_year = 1;
```

コンパイラは、`Artist` 内の `int32` 値またはフィールドが未設定の場合のデフォルト値を返す `GetBirthYear()` アクセサメソッドと、`BirthYear` という名前の `*int32` フィールドを持つ構造体を生成します。デフォルトが明示的に設定されていない場合、そのタイプの[ゼロ値](https://golang.org/ref/spec#The_zero_value)が代わりに使用されます（数値の場合は `0`、文字列の場合は空の文字列）。

他のスカラーフィールドタイプ（`bool`、`bytes`、`string` を含む）については、`*int32` は、[スカラー値タイプテーブル](/programming-guides/proto2#scalar)に従って対応する Go タイプに置き換えられます。

### 単一スカラーフィールド (proto3) {#singular-scalar-proto3}

このフィールド定義の場合:

```proto
int32 birth_year = 1;
optional int32 first_active_year = 2;
```

コンパイラは、`birth_year` 内の `int32` 値またはフィールドが未設定の場合のデフォルト値を返す `GetBirthYear()` アクセサメソッドと、`BirthYear` という名前の `int32` フィールドを持つ構造体を生成します。フィールドが未設定の場合は、そのタイプの[ゼロ値](https://golang.org/ref/spec#The_zero_value)が代わりに使用されます（数値の場合は `0`、文字列の場合は空の文字列）。

他のスカラーフィールドタイプ（`bool`、`bytes`、`string`を含む）については、`int32`は、[スカラー値の種類テーブル](/programming-guides/proto3#scalar)に従って対応するGoタイプに置き換えられます。Proto内の未設定の値は、そのタイプの[ゼロ値](https://golang.org/ref/spec#The_zero_value)として表されます（数値の場合は`0`、文字列の場合は空の文字列）。

### 単数メッセージフィールド {#singular-message}

次のメッセージタイプが与えられた場合：

```proto
message Band {}
```

`Band`フィールドを持つメッセージに対して：

```proto
// proto2
message Concert {
  optional Band headliner = 1;
  // The generated code is the same result if required instead of optional.
}

// proto3
message Concert {
  Band headliner = 1;
}
```

コンパイラはGoの構造体を生成します：

```go
type Concert struct {
    Headliner *Band
}
```

メッセージフィールドは`nil`に設定することができ、これはフィールドが未設定であることを意味し、実質的にフィールドをクリアします。これは、メッセージ構造体の\"空の\"インスタンスに値を設定するのとは等しくありません。

コンパイラはまた、`func (m *Concert) GetHeadliner() *Band`ヘルパー関数を生成します。この関数は、`m`が`nil`であるか`headliner`が未設定の場合に`nil`の`*Band`を返します。これにより、中間の`nil`チェックなしにget呼び出しをチェーンすることが可能になります：

```go
var m *Concert // デフォルトはnil
log.Infof("GetFoundingYear() = %d (no panic!)", m.GetHeadliner().GetFoundingYear())
```

### 繰り返しフィールド {#repeated}

各繰り返しフィールドは、Goの構造体内の`T`フィールドのスライスを生成します。ここで、`T`はフィールドの要素タイプです。次の繰り返しフィールドを持つメッセージに対して：

```proto
message Concert {
  // Best practice: use pluralized names for repeated fields:
  // /programming-guides/style#repeated-fields
  repeated Band support_acts = 1;
}
```

コンパイラはGoの構造体を生成します：

```go
type Concert struct {
    SupportActs []*Band
}
```

同様に、`repeated bytes band_promo_images = 1;`のフィールド定義の場合、コンパイラは`BandPromoImage`という`[][]byte`フィールドを持つGo構造体を生成します。`repeated [enumeration](#enum)`の場合、`repeated MusicGenre genres = 2;`、コンパイラは`Genre`という`[]MusicGenre`フィールドを持つ構造体を生成します。

次の例は、フィールドを設定する方法を示しています：

```go
concert := &Concert{
  SupportActs: []*Band{
    {}, // First element.
    {}, // Second element.
  },
}
```

フィールドにアクセスするには、次のようにします：

```go
support := concert.GetSupportActs() // サポートの型は []*Band です。
b1 := support[0] // b1の型は *Band で、support_actsの最初の要素です。
```

### マップフィールド {#map}

各マップフィールドは、`TKey` がフィールドのキータイプであり、`TValue` がフィールドの値のタイプである `map[TKey]TValue` 型の構造体内のフィールドを生成します。次のマップフィールドを持つメッセージの場合：

```proto
message MerchItem {}

message MerchBooth {
  // items maps from merchandise item name ("Signed T-Shirt") to
  // a MerchItem message with more details about the item.
  map<string, MerchItem> items = 1;
}
```

コンパイラは次の Go 構造体を生成します：

```go
type MerchBooth struct {
    Items map[string]*MerchItem
}
```

### ワンオブフィールド {#oneof}

ワンオブフィールドの場合、protobuf コンパイラは、インターフェースタイプ `isMessageName_MyField` を持つ単一のフィールドを生成します。また、ワンオブ内の[単数フィールド](#singular-scalar-proto2)ごとに構造体を生成します。これらはすべて `isMessageName_MyField` インターフェースを実装します。

次のワンオブフィールドを持つメッセージの場合：

```proto
package account;
message Profile {
  oneof avatar {
    string image_url = 1;
    bytes image_data = 2;
  }
}
```

コンパイラは次の構造体を生成します：

```go
type Profile struct {
    // Types that are valid to be assigned to Avatar:
    //  *Profile_ImageUrl
    //  *Profile_ImageData
    Avatar isProfile_Avatar `protobuf_oneof:"avatar"`
}

type Profile_ImageUrl struct {
        ImageUrl string
}
type Profile_ImageData struct {
        ImageData []byte
}
```

`*Profile_ImageUrl` と `*Profile_ImageData` は、空の `isProfile_Avatar()` メソッドを提供することで `isProfile_Avatar` を実装します。

次の例は、フィールドを設定する方法を示しています：

```go
p1 := &account.Profile{
  Avatar: &account.Profile_ImageUrl{ImageUrl: "http://example.com/image.png"},
}

// imageData is []byte
imageData := getImageData()
p2 := &account.Profile{
  Avatar: &account.Profile_ImageData{ImageData: imageData},
}
```

フィールドにアクセスするには、値に対して型スイッチを使用して異なるメッセージタイプを処理します。

```go
switch x := m.Avatar.(type) {
case *account.Profile_ImageUrl:
    // Load profile image based on URL
    // using x.ImageUrl
case *account.Profile_ImageData:
    // Load profile image based on bytes
    // using x.ImageData
case nil:
    // The field is not set.
default:
    return fmt.Errorf("Profile.Avatar has unexpected type %T", x)
}
```

コンパイラはまた、`func (m *Profile) GetImageUrl() string` および `func (m *Profile) GetImageData() []byte` の取得メソッドを生成します。各取得関数は、そのフィールドの値または設定されていない場合はゼロ値を返します。

## 列挙型 {#enum}

次のような列挙型が与えられた場合：

```proto
message Venue {
  enum Kind {
    KIND_UNSPECIFIED = 0;
    KIND_CONCERT_HALL = 1;
    KIND_STADIUM = 2;
    KIND_BAR = 3;
    KIND_OPEN_AIR_FESTIVAL = 4;
  }
  Kind kind = 1;
  // ...
}
```

プロトコルバッファコンパイラは、その型と一連の定数を生成します：

```go
type Venue_Kind int32

const (
    Venue_KIND_UNSPECIFIED       Venue_Kind = 0
    Venue_KIND_CONCERT_HALL      Venue_Kind = 1
    Venue_KIND_STADIUM           Venue_Kind = 2
    Venue_KIND_BAR               Venue_Kind = 3
    Venue_KIND_OPEN_AIR_FESTIVAL Venue_Kind = 4
)
```

メッセージ内の列挙型（上記のものなど）の場合、型名はメッセージ名で始まります：

```go
type Venue_Kind int32
```

パッケージレベルの列挙型の場合：

```proto
enum Genre {
  GENRE_UNSPECIFIED = 0;
  GENRE_ROCK = 1;
  GENRE_INDIE = 2;
  GENRE_DRUM_AND_BASS = 3;
  // ...
}
```

Go の型名は proto 列挙型名から変更されません：

```go
type Genre int32
```

この型には、与えられた値の名前を返す `String()` メソッドがあります。

`Enum()` メソッドは、与えられた値で新しく割り当てられたメモリを初期化し、対応するポインタを返します。

```go
func (Genre) Enum() *Genre
```

プロトコルバッファコンパイラは、列挙型の各値に対して定数を生成します。
メッセージ内の列挙型の場合、定数は包含メッセージの名前で始まります：

```go
const (
    Venue_KIND_UNSPECIFIED       Venue_Kind = 0
    Venue_KIND_CONCERT_HALL      Venue_Kind = 1
    Venue_KIND_STADIUM           Venue_Kind = 2
    Venue_KIND_BAR               Venue_Kind = 3
    Venue_KIND_OPEN_AIR_FESTIVAL Venue_Kind = 4
)
```

パッケージレベルの列挙型の場合、定数は列挙型の名前で始まります：

```go
const (
    Genre_GENRE_UNSPECIFIED   Genre = 0
    Genre_GENRE_ROCK          Genre = 1
    Genre_GENRE_INDIE         Genre = 2
    Genre_GENRE_DRUM_AND_BASS Genre = 3
)
```

また、protobufコンパイラは整数値から文字列名へのマップと、名前から値へのマップも生成します：

```go
var Genre_name = map[int32]string{
    0: "GENRE_UNSPECIFIED",
    1: "GENRE_ROCK",
    2: "GENRE_INDIE",
    3: "GENRE_DRUM_AND_BASS",
}
var Genre_value = map[string]int32{
    "GENRE_UNSPECIFIED":   0,
    "GENRE_ROCK":          1,
    "GENRE_INDIE":         2,
    "GENRE_DRUM_AND_BASS": 3,
}
```

`.proto`言語では、複数の列挙型シンボルが同じ数値を持つことができます。同じ数値を持つシンボルは同義語です。これらはGoではまったく同じ方法で表され、同じ数値に対応する複数の名前があります。逆マッピングには、`.proto`ファイルで最初に表示される名前に対する数値の単一エントリが含まれます。

## 拡張機能（proto2） {#extensions}

拡張機能の定義が与えられた場合：

```proto
extend Concert {
  optional int32 promo_id = 123;
}
```

プロトコルバッファコンパイラは、`E_Promo_id`という名前の
[`protoreflect.ExtensionType`](https://pkg.go.dev/google.golang.org/protobuf/reflect/protoreflect?tab=doc#ExtensionType)
値を生成します。この値は、メッセージ内の拡張機能にアクセスするために
[`proto.GetExtension`](https://pkg.go.dev/google.golang.org/protobuf/proto?tab=doc#GetExtension)、
[`proto.SetExtension`](https://pkg.go.dev/google.golang.org/protobuf/proto?tab=doc#SetExtension)、
[`proto.HasExtension`](https://pkg.go.dev/google.golang.org/protobuf/proto?tab=doc#HasExtension)、および
[`proto.ClearExtension`](https://pkg.go.dev/google.golang.org/protobuf/proto?tab=doc#ClearExtension)
関数と共に使用できます。`GetExtension`関数と`SetExtension`関数は、それぞれ拡張機能の値を含む`interface{}`値を返し、受け入れます。

単一のスカラー拡張フィールドの場合、拡張機能の値の型は、
[スカラー値型テーブル](/programming-guides/proto3#scalar)からの対応するGo型です。

単一の埋め込みメッセージ拡張フィールドの場合、拡張機能の値の型は`*M`であり、`M`はフィールドメッセージ型です。

拡張フィールドが繰り返しの場合、拡張値の型は単数形のスライスです。

たとえば、次の定義が与えられた場合：

```proto
extend Concert {
  optional int32 singular_int32 = 1;
  repeated bytes repeated_strings = 2;
  optional Band singular_message = 3;
}
```

拡張値は次のようにアクセスできます：

```go
m := &somepb.Concert{}
proto.SetExtension(m, extpb.E_SingularInt32, int32(1))
proto.SetExtension(m, extpb.E_RepeatedString, []string{"a", "b", "c"})
proto.SetExtension(m, extpb.E_SingularMessage, &extpb.Band{})

v1 := proto.GetExtension(m, extpb.E_SingularInt32).(int32)
v2 := proto.GetExtension(m, extpb.E_RepeatedString).([][]byte)
v3 := proto.GetExtension(m, extpb.E_SingularMessage).(*extpb.Band)
```

拡張は別の型の中にネストして宣言することができます。たとえば、次のようにするのが一般的なパターンです：

```proto
message Promo {
  extend Concert {
    optional int32 promo_id = 124;
  }
}
```

この場合、`ExtensionType` の値は `E_Promo_Concert` と名前付けられています。

## サービス {#service}

Goコードジェネレータはデフォルトではサービスの出力を生成しません。[gRPC](https://www.grpc.io/) プラグインを有効にすると（[gRPC Goクイックスタートガイド](https://github.com/grpc/grpc-go/tree/master/examples)を参照）、gRPCをサポートするためのコードが生成されます。
