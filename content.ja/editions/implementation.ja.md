## 概要 {#overview}

### Edition 2023 {#edition-2023}

最初にリリースされたエディションはEdition 2023で、proto2とproto3の構文を統一するために設計されています。振る舞いの違いをカバーするために追加した機能については、[エディション用の機能設定](/editions/features)に詳細が記載されています。

### 機能の定義 {#feature-definition}

エディションを*サポート*し、定義済みのグローバル機能に加えて、インフラストラクチャを活用するために独自の機能を定義したい場合があります。これにより、ジェネレータやランタイムが新しい振る舞いを制御するために使用できる任意の機能を定義できます。最初のステップは、`descriptor.proto`内の`FeatureSet`メッセージの拡張番号を9999より大きくすることです。GitHubでプルリクエストを送信していただくと、次のリリースに含まれます（例：[#15439](https://github.com/protocolbuffers/protobuf/pull/15439)を参照）。

拡張番号を取得したら、自分の機能の proto ファイルを作成できます（例：[cpp_features.proto](https://github.com/protocolbuffers/protobuf/blob/main/src/google/protobuf/cpp_features.proto)）。これらは通常、次のように見えます：

```proto
edition = "2023";

package foo;

import "google/protobuf/descriptor.proto";

extend google.protobuf.FeatureSet {
  MyFeatures features = <extension #>;
}

message MyFeatures {
  enum FeatureValue {
    FEATURE_VALUE_UNKNOWN = 0;
    VALUE1 = 1;
    VALUE2 = 2;
  }

  FeatureValue feature_value = 1 [
    targets = TARGET_TYPE_FIELD,
    targets = TARGET_TYPE_FILE,
    feature_support = {
      edition_introduced: EDITION_2023,
      edition_deprecated: EDITION_2024,
      deprecation_warning: "Feature will be removed in 2025",
      edition_removed: EDITION_2025,
    },
    edition_defaults = { edition: EDITION_LEGACY, value: "VALUE1" },
    edition_defaults = { edition: EDITION_2024, value: "VALUE2" }
  ];
}
```

ここでは、新しい enum 機能 `foo.feature_value` を定義しています（現在はブール値と enum タイプのみがサポートされています）。取りうる値を定義するだけでなく、それがどのように使用されるかを指定する必要があります：

*   **Targets** - この機能がアタッチできる proto ディスクリプタのタイプを指定します。これにより、ユーザーが機能を明示的に指定できる場所が制御されます。すべてのタイプを明示的にリストする必要があります。
*   **Feature support** - この機能の寿命をエディションに対して指定します。導入されたエディションを指定する必要があり、それ以前には許可されません。後のエディションで非推奨化または削除することもできます。
*   **Edition defaults** - 機能のデフォルト値の変更を指定します。すべてのサポートされているエディションをカバーする必要がありますが、デフォルトが変更されなかったエディションは省略できます。`EDITION_PROTO2` と `EDITION_PROTO3` をここで指定して、「レガシー」エディションのデフォルトを提供することができることに注意してください（[レガシーエディション](#legacy_editions)を参照）。

#### フィーチャーとは何ですか？ {#what-feature}

フィーチャーは、時間の経過とともに悪い振る舞いを段階的に抑制するメカニズムを提供するように設計されています。実際にフィーチャーを削除するタイムラインは、将来数年（または数十年）かかるかもしれませんが、任意のフィーチャーの望ましい目標は最終的な削除です。悪い振る舞いが特定された場合、修正を保護する新しいフィーチャーを導入できます。次のエディション（または後で）で、デフォルト値を切り替えることができますが、ユーザーが古い振る舞いを保持することができるようにします。将来のある時点で、フィーチャーを非推奨にマークし、それをオーバーライドしているユーザーにカスタム警告をトリガーします。後のエディションでは、それを削除済みとマークし、ユーザーがそれをオーバーライドできないようにします（ただし、デフォルト値は引き続き適用されます）。その最後のエディションのサポートが破壊的リリースで削除されるまで、フィーチャーは古いエディションに固執しているプロトにとって引き続き使用可能であり、移行する時間を与えます。

削除する意図のないオプションの振る舞いを制御するフラグは、
[カスタムオプション](/programming-guides/proto2/#customoptions)として実装する方が良いです。
これは、フィーチャーをブール値または列挙型に制限している理由と関連しています。比較的無制限な数の値によって制御される振る舞いは、エディションフレームワークには適していない可能性があります。なぜなら、その多くの異なる振る舞いを最終的に抑制することは現実的ではないからです。

これに関連する注意点の1つは、ワイヤー境界に関連する振る舞いです。シリアライズやパースの振る舞いを制御するために言語固有のフィーチャーを使用することは危険です。なぜなら、他の言語が反対側にある可能性があるからです。ワイヤーフォーマットの変更は常に、`descriptor.proto` でグローバルなフィーチャーによって制御されるべきであり、すべてのランタイムが均一に尊重できるようにするべきです。

### ジェネレーター {#generators}

C++で書かれたジェネレーターは、C++ランタイムを使用するため、多くの機能を無料で提供します。彼らは自分で[フィーチャーの解決](#feature_resolution)を処理する必要がなく、必要なフィーチャー拡張があれば、CodeGeneratorの`GetFeatureExtensions` でそれらを登録できます。通常、コード生成中にディスクリプタの解決されたフィーチャーにアクセスするために`GetResolvedSourceFeatures` を使用し、自分自身の未解決のフィーチャーにアクセスするために`GetUnresolvedSourceFeatures` を使用できます。

プラグインは、生成されるコードと同じ言語で書かれている場合、機能定義のためにカスタムのブートストラップが必要になるかもしれません。

#### 明示的なサポート {#explicit-support}

ジェネレータは、正確にどのエディションをサポートするかを指定する必要があります。これにより、リリース後にエディションのサポートを安全に追加できます。Protocは、`CodeGeneratorResponse`の`supported_features`フィールドに`FEATURE_SUPPORTS_EDITIONS`を含まないジェネレータに送信されたエディションのプロトを拒否します。さらに、`minimum_edition`および`maximum_edition`フィールドがあり、正確なサポートウィンドウを指定するために使用されます。新しいエディションのコードと機能の変更をすべて定義したら、`maximum_edition`を上げてこのサポートを宣伝できます。

#### コード生成テスト {#codegen-tests}

Edition 2023が予期しない機能的変更を生じないことを確認するために使用できるコード生成テストがあります。これらは、C++やJavaなどの言語で非常に有用であり、機能の大部分がジェネレートされたコードに含まれる場合に役立ちます。一方、Pythonのような言語では、ジェネレートされたコードが基本的にシリアライズされた記述子のコレクションであるため、それほど有用ではありません。

このインフラストラクチャはまだ再利用できませんが、将来のリリースで再利用可能にする予定です。その時点で、エディションへの移行に予期しないコード生成の変更がないかを検証するために使用できるようになります。

### ランタイム {#runtimes}

リフレクションやダイナミックメッセージを持たないランタイムは、エディションを実装するために何もする必要はありません。そのロジックはすべてコードジェネレータによって処理されるべきです。

リフレクションを持つがダイナミックメッセージを持たない言語は、解決済みの機能が必要ですが、ジェネレータ内でのみ処理することも選択できます。これは、ランタイムに解決済みおよび未解決の機能セットの両方をコード生成中に渡すことで行われます。これにより、ランタイムで[Feature Resolution](#feature_resolution)を再実装する必要がなくなりますが、効率が低下する可能性があります。なぜなら、それはすべての記述子に対してユニークな機能セットを作成するからです。

ダイナミックメッセージを持つ言語は、ランタイムで記述子をランタイムでビルドできる必要があるため、エディションを完全に実装する必要があります。

#### 構文反映 {#syntax_reflection}

リフレクションを使用してエディションを実行時に実装する最初のステップは、`syntax` キーワードの直接的なチェックをすべて削除することです。これらすべては、必要に応じて `syntax` を使用できるようにすることができる、より細かい粒度の機能ヘルパーに移動すべきです。

次の機能ヘルパーは、ディスクリプタに実装されるべきで、言語に適した名前を付けるべきです:

*   `FieldDescriptor::has_presence` - フィールドが明示的な存在を持つかどうか
    *   繰り返しフィールドは*決して*存在しません
    *   メッセージ、拡張、および oneof フィールドは*常に*明示的な存在を持ちます
    *   それ以外のすべては、`field_presence` が `IMPLICIT` でない場合にのみ存在します
*   `FieldDescriptor::is_required` - フィールドが必須かどうか
*   `FieldDescriptor::requires_utf8_validation` - フィールドが utf8 の妥当性をチェックする必要があるかどうか
*   `FieldDescriptor::is_packed` - 繰り返しフィールドがパックされたエンコーディングを持つかどうか
*   `FieldDescriptor::is_delimited` - メッセージフィールドが区切り記号付きエンコーディングを持つかどうか
*   `EnumDescriptor::is_closed` - フィールドが閉じられているかどうか

**注意:** ほとんどの言語では、メッセージエンコーディング機能は現在も `TYPE_GROUP` でシグナルされ、必須フィールドはまだ `LABEL_REQUIRED` が設定されています。これは理想的ではなく、下流の移行を容易にするために行われました。最終的には、これらは適切なヘルパーに移行し、`TYPE_MESSAGE/LABEL_OPTIONAL` に移行すべきです。

下流のユーザーは、直接構文を使用する代わりにこれらの新しいヘルパーに移行すべきです。次の既存のディスクリプタ API クラスは理想的には非推奨とされ、最終的に削除されるべきです。なぜなら、これらは構文情報を漏洩させるからです:

*   `FileDescriptor` 構文
*   Proto3 オプションの API
    *   `FieldDescriptor::has_optional_keyword`
    *   `OneofDescriptor::is_synthetic`
    *   `Descriptor::*real_oneof*` - 単に "oneof" に名前を変更し、既存の "oneof" ヘルパーを削除すべきです。なぜなら、これらは合成 oneof に関する情報を漏洩させるからです（エディションには存在しない）。
*   グループ型
    *   `TYPE_GROUP` 列挙値は削除され、`is_delimited` ヘルパーに置き換えられるべきです。
*   必須ラベル
    *   `LABEL_REQUIRED` 列挙値は削除され、`is_required` ヘルパーに置き換えられるべきです。

多くのユーザーコードのクラスには、これらのチェックが存在しますが、編集には敵対的ではありません。たとえば、その合成的なoneof実装のためにproto3 `optional` を特別に処理する必要があるコードは、極性が `syntax == "proto3"` のようなものであれば（`syntax != "proto2"` をチェックするのではなく）、編集には敵対的ではありません。

これらの API を完全に削除することができない場合は、非推奨として扱うべきです。

#### 機能の可視性 {#visibility}

[editions-feature-visibility](https://github.com/protocolbuffers/protobuf/blob/main/docs/design/editions/editions-feature-visibility.md) で議論されているように、機能プロトは任意の Protobuf 実装の内部の詳細であるべきです。それらが制御する *振る舞い* は、ディスクリプタメソッドを介して公開されるべきですが、プロト自体は公開されるべきではありません。特に、ユーザーに公開されるオプションは、その `features` フィールドが削除されている必要があります。

特徴が漏れ出ることを許可する唯一のケースは、ディスクリプタをシリアライズする場合です。生成されたディスクリプタプロトは、元の proto ファイルの忠実な表現であるべきであり、オプション内に *未解決の特徴* を含んでいる必要があります。

#### レガシー版 {#legacy_editions}

[legacy-syntax-editions](https://github.com/protocolbuffers/protobuf/blob/main/docs/design/editions/legacy-syntax-editions.md) でさらに詳しく議論されているように、editions 実装の早期カバレッジを得るための優れた方法は、proto2、proto3、および editions を統合することです。これにより、proto2 と proto3 が実質的に editions に移行し、[Syntax Reflection](#syntax_reflection) で実装されたすべてのヘルパーが機能のみを使用するようになります（構文に基づく分岐ではなく）。これは、proto ファイルのさまざまな側面がどの機能が適切かを通知する *機能推論* フェーズを [Feature Resolution](#feature_resolution) に挿入することで行うことができます。これらの機能は、解決された機能セットを取得するために親の機能にマージされることができます。

proto2/proto3 にはすでに合理的なデフォルトが提供されていますが、2023 年版では、次の追加の推論が必要です。

*   必須 - フィールドに `LABEL_REQUIRED` がある場合、`LEGACY_REQUIRED` の存在を推測します
*   グループ - フィールドに `TYPE_GROUP` がある場合、`DELIMITED` メッセージエンコーディングを推測します
*   パック - `packed` オプションが true の場合、`PACKED` エンコーディングを推測します
*   拡張 - proto3 フィールドの `packed` が明示的に false に設定されている場合、`EXPANDED` エンコーディングを推測します

#### 適合テスト {#conformance-tests}

エディション固有の適合テストが追加されましたが、オプトインする必要があります。
これらを有効にするにはランナーに `--maximum_edition 2023` フラグを渡す必要があります。
以下の新しいメッセージタイプを処理するためにテスターバイナリを構成する必要があります：

*   `protobuf_test_messages.editions.proto2.TestAllTypesProto2` - 古い proto2 メッセージと同じですが、エディション 2023 に変換されています
*   `protobuf_test_messages.editions.proto3.TestAllTypesProto3` - 古い proto3 メッセージと同じですが、エディション 2023 に変換されています
*   `protobuf_test_messages.editions.TestAllTypesEdition2023` - エディション 2023 固有のテストケースをカバーするために使用されます

### 機能解決 {#feature-resolution}

エディションは機能を定義するためにレキシカルスコープを使用し、エディションサポートを実装する必要がある非C++コードは、*機能解決* アルゴリズムを再実装する必要があります。
ただし、作業の大部分は protoc 自体によって処理され、中間の `FeatureSetDefaults` メッセージを出力するように構成できます。
このメッセージには、各エディションでのデフォルトの機能値が記載された、一連の機能定義ファイルの「コンパイル」が含まれています。

たとえば、上記の機能定義は、proto2 とエディション 2025 の間で次のデフォルト値にコンパイルされます（テキスト形式で表記）：

```
defaults {
  edition: EDITION_PROTO2
  overridable_features { [foo.features] {} }
  fixed_features {
    // Global feature defaults…
    [foo.features] { feature_value: VALUE1 }
  }
}
defaults {
  edition: EDITION_PROTO3
  overridable_features { [foo.features] {} }
  fixed_features {
    // Global feature defaults…
    [foo.features] { feature_value: VALUE1 }
  }
}
defaults {
  edition: EDITION_2023
  overridable_features {
    // Global feature defaults…
    [foo.features] { feature_value: VALUE1 }
  }
}
defaults {
  edition: EDITION_2024
  overridable_features {
    // Global feature defaults…
    [foo.features] { feature_value: VALUE2 }
  }
}
defaults {
  edition: EDITION_2025
  overridable_features {
    // Global feature defaults…
  }
  fixed_features { [foo.features] { feature_value: VALUE2 } }
}
minimum_edition: EDITION_PROTO2
maximum_edition: EDITION_2025

```

グローバルな機能のデフォルトは簡潔さのために省略されていますが、それらも存在します。
このオブジェクトには、指定された範囲内で一意のデフォルトセットを持つすべてのエディションの順序付きリストが含まれています。
各デフォルトセットは、*オーバーライド可能* と *固定* の機能に分割されます。
前者はユーザーが自由にオーバーライドできるエディションのサポートされる機能であり、後者はまだ導入されていないか削除された機能であり、ユーザーによってオーバーライドできません。

提供するBazelルールを使用して、これらの中間オブジェクトをコンパイルします：

```
load("@com_google_protobuf//editions:defaults.bzl", "compile_edition_defaults")

compile_edition_defaults(
    name = "my_defaults",
    srcs = ["//some/path:lang_features_proto"],
    maximum_edition = "PROTO2",
    minimum_edition = "2024",
)
```

出力`FeatureSetDefaults`は、必要な言語で機能解決を行うための生の文字列リテラルに埋め込むことができます。また、これを行うための`embed_edition_defaults`マクロも提供しています：

```
embed_edition_defaults(
    name = "embed_my_defaults",
    defaults = ":my_defaults",
    output = "my_defaults.h",
    placeholder = "DEFAULTS_DATA",
    template = "my_defaults.h.template",
)
```

また、protocを直接（Bazelの外部で）呼び出して、このデータを生成することもできます：

```
protoc --edition_defaults_out=defaults.binpb --edition_defaults_minimum=PROTO2 --edition_defaults_maximum=2023 <feature files...>
```

デフォルトメッセージがコードにフックされ、指定されたエディションのファイル記述子の機能解決が行われると、次の単純なアルゴリズムに従います：

1. エディションが適切な範囲[`minimum_edition`、`maximum_edition`]内にあることを検証します
2. 順序付けられた`defaults`フィールドをエディション以下の最も高いエントリをバイナリサーチします
3. 選択したデフォルトから`overridable_features`を`fixed_features`にマージします
4. ファイルオプションの`features`フィールド内に設定された明示的な機能をマージします

その後、他の記述子のすべての機能を再帰的に解決できます：

1. 親記述子の機能セットを初期化します
2. ファイルオプションの`features`フィールドに設定された明示的な機能をマージします

“親”記述子を決定するために、[C++実装](https://github.com/protocolbuffers/protobuf/blob/27.x/src/google/protobuf/descriptor.cc#L1129)を参照できます。これはほとんどの場合には簡単ですが、拡張は拡張元ではなく包含スコープが親であるため、少し驚くかもしれません。Oneofも、フィールドの親として考慮する必要があります。

#### 適合テスト {#conformance-tests}

将来のリリースでは、機能解決のクロス言語検証のための適合テストを追加する予定です。その間、通常の[適合テスト](#conformance_tests)は部分的なカバレッジを提供しますし、[例の継承ユニットテスト](https://github.com/protocolbuffers/protobuf/blob/27.x/python/google/protobuf/internal/descriptor_test.py#L1386)も移植して、より包括的なカバレッジを提供できます。

### 例 {#examples}

以下は、ランタイムおよびプラグインでのエディションサポートの実装例です。

#### Java {#java}

*   [#14138](https://github.com/protocolbuffers/protobuf/pull/14138) - Java機能のprotoに対するC++ gencodeを使用したブートストラップコンパイラ
*   [#14377](https://github.com/protocolbuffers/protobuf/pull/14377) - Java、Kotlin、およびJava Liteコードジェネレーターでの機能の使用、コード生成テストを含む
*   [#15210](https://github.com/protocolbuffers/protobuf/pull/15210) - Javaフルランタイムでの機能の使用、Java機能のブートストラップ、機能解決、およびレガシーエディションのカバレッジ、ユニットテストと適合性テストを含む

#### 純粋なPython {#python}

*   [#14546](https://github.com/protocolbuffers/protobuf/pull/14546) - コード生成テストの事前設定
*   [#14547](https://github.com/protocolbuffers/protobuf/pull/14547) - 一気にエディションを完全に実装、ユニットテストと適合性テストを含む

#### 𝛍pb {#upb}

*   [#14638](https://github.com/protocolbuffers/protobuf/pull/14638) - 機能解決とレガシーエディションをカバーするエディション実装の最初のパス
*   [#14667](https://github.com/protocolbuffers/protobuf/pull/14667) - フィールドラベル/タイプのより完全な処理、upbのコードジェネレーターのサポート、一部のテストを追加
*   [#14678](https://github.com/protocolbuffers/protobuf/pull/14678) - upbをPythonランタイムに接続し、より多くのユニットテストと適合性テストを追加

#### Ruby {#ruby}

*   [#16132](https://github.com/protocolbuffers/protobuf/pull/16132) - フルエディションサポートのためにupb/Javaをすべての4つのRubyランタイムに接続
