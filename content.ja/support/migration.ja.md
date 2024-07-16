## 移行ガイド {#compiler-22}

### JSON フィールド名の競合 {#json-field-names}

変更のソース: [PR #11349](https://github.com/protocolbuffers/protobuf/pull/11349), [PR #10750](https://github.com/protocolbuffers/protobuf/pull/10750)

JSON マッピングに関するフィールド名の競合の扱いにいくつかの微妙な変更を加えました。proto3 では、フィールド名が大文字と小文字を区別する JSON マッピング（元の名前のキャメルケース）を生成する場合にのみエラーを出すように制限を部分的に緩和しました。また、`json_name` オプションもチェックし、大文字と小文字を区別する競合に対してエラーを出します。proto2 では、2 つの `json_name` 仕様が競合する場合にエラーを出すように制限を強化しました。暗黙の JSON マッピング（キャメルケース）が競合する場合は、proto2 では警告を出します。

競合するフィールド名をリネームすることがすぐにできない場合は、特定のメッセージ/列挙型に `deprecated_legacy_json_field_conflicts` オプションを設定して、従来の動作を復元する一時的なオプションを提供しています。このオプションは将来のリリースで削除されますが、移行に時間を与えます。

## C++ API の変更 {#cpp-22}

4.22.0 では、C++ ランタイムと protoc に対して破壊的な変更があります。これは
[8 月に発表されました](/news/2022-08-03#cpp-changes)。

### Autotools の廃止 {#autotools}

変更のソース:
[PR #10132](https://github.com/protocolbuffers/protobuf/pull/10132)

v22.0 では、protobuf コンパイラと C++ ランタイムから Autotools サポートをすべて削除しました。これらのいずれかをビルドするために Autotools を使用している場合は、[CMake](http://cmake.org) または
[Bazel](http://bazel.build) に移行する必要があります。CMake で protobuf をセットアップするための
[専用の手順](https://github.com/protocolbuffers/protobuf/blob/main/cmake/README.md)
があります。

### Abseil 依存関係 {#abseil}

変更のソース：
[PR #10416](https://github.com/protocolbuffers/protobuf/pull/10416)

v22.0 では、明示的に
[Abseil](https://github.com/abseil/abseil-cpp) に依存するようになりました。これにより、ほとんどの
[スタブ](https://github.com/protocolbuffers/protobuf/tree/21.x/src/google/protobuf/stubs) を削除することができました。これらは、以前の内部コードから派生した古いコードであり、後に Abseil になりました。いくつかの微妙な動作の変更がありますが、ほとんどはユーザーには透過的であるはずです。いくつかの注目すべき変更点は次のとおりです：

*   **string_view** - `absl::string_view` が `const std::string&` の代わりに多くの API で使用されるようになりました。これは、主に入力引数に使用され、ユーザーには目立った変更がないはずです。一部の場合（仮想メソッドの引数や戻り値など）では、新しいシグネチャを使用するために明示的な変更が必要になるかもしれません。

*   **tables** - STL のセット/マップの代わりに、Abseil の `flat_hash_map`、`flat_hash_set`、`btree_map`、`btree_set` を使用しています。これらはより効率的であり、[異種検索](https://abseil.io/tips/144) を可能にします。これは、ユーザーにはほとんど見えないはずですが、テーブルの順序に関連するいくつかの微妙な動作の変更が発生する可能性があります。

*   **logging** - Abseil の
    [ログライブラリ](https://abseil.io/docs/cpp/guides/logging) は、古いログコードと非常に似ており、わずかに異なるスペル（たとえば、`ABSL_CHECK` の代わりに `GOOGLE_CHECK`）があります。最大の違いは、例外をサポートしていないことであり、`FATAL` アサーションが失敗した場合には常にクラッシュするようになりました（以前は `PROTOBUF_USE_EXCEPTIONS` フラグを使用して例外に切り替えていました）。これらは深刻な問題が発生した場合にのみ発生するため、無条件でクラッシュするのが適切な対応だと考えています。

    ログの変更のソース：[PR #11623](https://github.com/protocolbuffers/protobuf/pull/11623)

*   **ビルド依存関係** - 新しいビルド依存関係は常にダウンストリームユーザーに問題を引き起こす可能性があります。ビルドには、必ず
    [Abseil LTS 20230117](https://github.com/abseil/abseil-cpp/releases/tag/20230117.rc1)
    またはそれ以降のバージョンが必要です。

    *   Bazel ビルドでは、[`protobuf_deps`](https://github.com/protocolbuffers/protobuf/blob/main/protobuf_deps.bzl) が `WORKSPACE` から実行されると、Abseil はピン留めされた LTS リリースで自動的にダウンロードおよびビルドされます。これは透過的に行われるはずですが、古いバージョンの Abseil に依存している場合は、依存関係をアップグレードする必要があります。

    *   CMake ビルドでは、まず既存の Abseil インストールを探します。これはトップレベルの CMake 構成によって引き込まれます（詳細は[こちら](https://github.com/abseil/abseil-cpp/blob/master/CMake/README.md#traditional-cmake-set-up)）。それ以外の場合、`protobuf_ABSL_PROVIDER` が `module` に設定されている場合（デフォルト）、git の[サブモジュール](https://github.com/protocolbuffers/protobuf/tree/main/third_party)から Abseil をビルドおよびリンクしようとします。`protobuf_ABSL_PROVIDER` が `package` に設定されている場合、事前にインストールされたシステムバージョンの Abseil を探します。

### GetCurrentTime メソッドの変更 {#getcurrenttime}

Windows では、`GetCurrentTime()` はシステムによって提供されるマクロの名前です。v22.x より前では、Protobuf は誤って `GetCurrentTime()` のマクロ定義を削除していました。これにより、`<protobuf/util/time_util.h>` を含んだ後に Windows 開発者が `GetCurrentTime()` マクロを使用できなくなりました。v22.x 以降、Protobuf はマクロ定義を保持します。これにより、以前の動作に依存していた顧客コードが壊れる可能性があります。たとえば、[`google::protobuf::util::TimeUtil::GetCurrentTime()`](/reference/cpp/api-docs/google.protobuf.util.time_util.md#TimeUtil) を使用している場合です。

アプリを新しい動作に移行するには、次のいずれかの方法でコードを変更してください：

*   `GetCurrent` マクロが定義されている場合は、明示的に `GetCurrentTime` マクロを未定義にする
*   マクロの展開を防ぐために、`(google::protobuf::util::TimeUtil::GetCurrentTime)()` または類似の式を使用する

#### マクロの未定義化の例

このアプローチは、Windows のマクロを使用しない場合に使用します。

変更前：

```cpp
#include <google/protobuf/util/time_util.h>

void F() {
  auto time = google::protobuf::util::TimeUtil::GetCurrentTime();
}
```

```cpp
#include <google/protobuf/util/time_util.h>
#ifdef GetCurrent
#undef GetCurrentTime
#endif

void F() {
  auto time = google::protobuf::util::TimeUtil::GetCurrentTime();
}

```

**例2: マクロの展開を防ぐ**

```cpp
#include <google/protobuf/util/time_util.h>

void F() {
  auto time = google::protobuf::util::TimeUtil::GetCurrentTime();
}
```

```cpp
#include <google/protobuf/util/time_util.h>

void F() {
  auto time = (google::protobuf::util::TimeUtil::GetCurrentTime)();
}

```

## C++20 サポート {#cpp20}

変更のソース: [PR #10796](https://github.com/protocolbuffers/protobuf/pull/10796)

C++20 をサポートするために、C++ で生成された protobuf コードに新しい
[keywords](https://en.cppreference.com/w/cpp/keyword) を予約しました。
他の予約されたキーワードと同様に、これらをフィールド、列挙型、またはメッセージで使用する場合、
それらを有効な C++ にするためにアンダースコアの接尾辞を追加します。
たとえば、`concept` フィールドは `concept_()` ゲッターを生成します。
これらのキーワードを使用する既存の proto がある場合、これらのキーワードを参照する C++ コードを更新して、
適切なアンダースコアを追加する必要があります。

### 最終クラス {#final-classes}

変更のソース: [PR #11604](https://github.com/protocolbuffers/protobuf/pull/11604)

Protobuf ライブラリで行われた前提条件を強化する大規模な取り組みの一環として、
意図されていないクラスを継承することが決して意図されていなかったクラスを `final` でマークしました。
これらから継承するための既知のユースケースはありませんし、そうすると問題が発生する可能性が高いです。
これらのクラスから継承している場合の緩和策はありませんが、継承の有効な理由があると考える場合は、
[問題を開く](https://github.com/protocolbuffers/protobuf/issues)ことができます。

### コンテナ静的アサーション {#container-static-assertions}

変更のソース: [PR #11550](https://github.com/protocolbuffers/protobuf/pull/11550)

Protobuf ライブラリで行われた前提条件を強化する大規模な取り組みの一環として、
`Map`、`RepeatedField`、および `RepeatedPtrField` コンテナに静的アサーションを追加しました。
これにより、これらのコンテナを予想される型のみで使用していることを確認します。
これらの静的アサーションに遭遇した場合は、コードを Abseil または STL コンテナを使用するように移行する必要があります。
`std::vector` は繰り返しフィールドコンテナの良い置換ですし、`std::unordered_map` または `absl::flat_hash_map` は `Map` に対して適しています
（前者は同様のポインタの安定性を提供し、後者はより効率的です）。
```

### クリアされた要素の非推奨化 {#cleared-elements}

変更のソース: [PR #11588](https://github.com/protocolbuffers/protobuf/pull/11588), [PR #11639](https://github.com/protocolbuffers/protobuf/pull/11639)

"クリアされたフィールド"に関する `RepeatedPtrField` API が非推奨となり、後の破壊的なリリースで完全に削除されます。これは、要素をクリアした後に再利用するための最適化として元々追加されましたが、うまく機能しないことが判明しました。この API を使用している場合は、メモリ再利用のためにアリーナに移行することを検討する必要があります。

### UnsafeArena の非推奨化 {#unsafe-arena}

変更のソース: [PR #10325](https://github.com/protocolbuffers/protobuf/pull/10325)

アリーナ非安全な API を削除する取り組みの一環として、`RepeatedField::UnsafeArenaSwap` を非表示にしました。これは、これまでに削除した唯一のものですが、後のリリースではこれらを継続的に削除し、アリーナ間で効率的な借用パターンを処理するヘルパーを提供します。単一のアリーナ内（またはスタック/ヒープ内）では、`Swap` は `UnsafeArenaSwap` と同じくらい効率的です。利点は、異なるアリーナ間で誤って呼び出すと無効なメモリ操作を引き起こさないことです。

### Map ペアのアップグレード {#map-pairs}

変更のソース: [PR #11625](https://github.com/protocolbuffers/protobuf/pull/11625)

v22.0 では、`Map` API を Abseil と STL とより一貫性のあるものにする作業を開始しました。特に、`MapPair` クラスを `std::pair` へのエイリアスに置き換えました。これはほとんどのユーザーにとって透過的であるはずですが、クラスを直接使用していた場合はコードを更新する必要があるかもしれません。

### 新しい JSON パーサー {:#json-parser}

変更のソース: [PR #10729](https://github.com/protocolbuffers/protobuf/pull/10729)

このリリースでは、C++ の JSON パーサーを書き直しました。これは主に隠れた変更であるはずですが、不文書のクセが変わっている可能性があります。RFC-8219 JSON でないドキュメント（クォートが抜けていたり、非標準のブール値を使用しているなど）を解析することは非推奨であり、将来のリリースで削除されます。フィールドのシリアル化順序は、以前はより決定論的ではなかったが、現在はフィールド番号の順序と一致するように保証されています。


この移行の一環として、[util/internal](https://github.com/protocolbuffers/protobuf/tree/21.x/src/google/protobuf/util/internal) のすべてのファイルが削除されました。これらは古いパーサーで使用されており、外部で使用することは意図されていませんでした。

### `Arena::Init` {#arena-init}

変更のソース: [PR #10623](https://github.com/protocolbuffers/protobuf/pull/10623)

`Arena` の `Init` メソッドは何も行わないコードであり、現在削除されました。このメソッドを呼び出していた場合、おそらく `Arena` コンストラクタに直接 `ArenaOptions` のセットを指定して呼び出すことを意図していたはずです。その呼び出しを削除するか、そのコンストラクタに移行する必要があります。

### ErrorCollector 移行 {#error-collector}

変更のソース: [PR #11555](https://github.com/protocolbuffers/protobuf/pull/11555)

Abseil 移行の一環として、`const std::string&` から `absl::string_view` に移行しています。3つのエラーコレクタークラスについては、既存のコードを壊さずにこれを行うことはできません。v22.0 では、両方のバリアントをリリースすることにし、`AddError` と `AddWarning` メソッドを `RecordError` と `RecordWarning` に名前を変更しました。古いシグネチャは非推奨とマークされ、若干効率が低下します（文字列のコピーのため）、それ以外は引き続き機能します。これらを新しいバージョンに移行する必要があります。`Add*` メソッドは後の破壊的リリースで削除されます。
