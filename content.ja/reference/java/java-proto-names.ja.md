+++
title = "Java Proto Names"
weight = 655
linkTitle = "生成された Proto 名"
description = "イミュータブル API によって生成される名前に関する情報が含まれています。"
type = "docs"
+++

このドキュメントには、異なる proto オプションに基づいて、proto の完全修飾 Java 名が何であるかに関する情報が含まれています。この名前は、そのメッセージを使用するためにインポートする必要があるパッケージに対応しています。

**注意:** `java_package` および `java_alt_api_package` オプションは、`java_api_version` で示される API に対して相対的に解釈されます。たとえば、`java_api_version` が 1 の場合、proto1 パッケージは `java_package` となり、proto2 パッケージ（"alternative" API）は `java_alt_api_package` となります。そして、`java_api_version` が 2 の場合、`java_package` が proto2 パッケージを決定し、`java_alt_api_package` が proto1 パッケージを決定します。

## イミュータブル API メッセージ名 {#immutable-api-message-names}

イミュータブル API（`java_proto_library` BUILD ターゲット）によって生成された proto の名前は、次の表にリストされています。

java_api_version | java_multiple_files | java_alt_api_package | java_package | java_outer_classname | 生成された完全メッセージ名
:--------------: | :-----------------: | -------------------- | ------------ | -------------------- | ---------------------------
1                | true                | Defined              | -            |                      | `$java_alt_api_package.$message`
1                | true                | Not defined          | Not defined  |                      | `com.google.protos.$package.proto2api.$message`
1                | true                | Not defined          | Defined      |                      | `$java_package.proto2api.$message`
1                | false               | Defined              | -            | Not defined          | `$java_alt_api_package.$derived_outer_class.$message`
1                | false               | Defined              | -            | Defined              | `$java_alt_api_package.$java_outer_classname.$message`
1                | false               | Not defined          | Not defined  | Not defined          | `com.google.protos.$package.proto2api.$derived_outer_class.$message`
1                | false               | Not defined          | Not defined  | Defined              | `com.google.protos.$package.proto2api.$java_outer_classname.$message`
1                | false               | Not defined          | Defined      | Not defined          | `$java_package.proto2api.$derived_outer_class.$message`
1                | false               | Not defined          | Defined      | Defined              | `$java_package.proto2api.$java_outer_classname.$message`
2                | true                | -                    | Not defined  | -                    | `com.google.protos.$package.$message`
2                | true                | -                    | Defined      | -                    | `$java_package.$message`
2                | false               | -                    | Not defined  | Not defined          | `com.google.protos.$package.$derived_outer_class.$message`
2                | false               | -                    | Not defined  | Defined              | `com.google.protos.$package.$java_outer_classname.$message`
2                | false               | -                    | Defined      | Not defined          | `$java_package.$derived_outer_class.$message`
2                | false               | -                    | Defined      | Defined              | `$java_package.$java_outer_classname.$message`

**凡例**

- `-` は、オプションの設定または設定しない場合でも、生成されるメッセージの完全な名前に変更を加えないことを意味します。
- `$message` は、実際の proto メッセージの名前です。
- `$package` は、proto パッケージの名前です。これは、通常、ファイルの先頭にある `package` ディレクティブで指定される名前です。
- `$derived_outer_class` は、proto ファイル名から生成される名前です。一般的には、ファイル名から句読点を削除し、CamelCase に変換して計算されます。たとえば、proto ファイルが `foo_bar.proto` の場合、`$derived_outer_class` の値は `FooBar` です。

    生成されるクラス名が proto ファイルで定義されたメッセージの1つと同じ場合、`derived_outer_class` に `OuterClass` が追加されます。たとえば、proto が `foo_bar.proto` で `FooBar` メッセージを含む場合、`$derived_outer_class` の値は `FooBarOuterClass` です。クラス名が同じであるかどうかに関係なく、v1 API を使用する場合も同様です。

- その他の `$names` は、proto ファイルで定義された対応する proto2 ファイルオプションの値です。

### 推奨事項 {#recommendation}

推奨されるオプションは次のとおりです:

```proto
option java_multiple_files = true;
```

`java_multiple_files = true` を使用すると、各メッセージの生成された Java クラスが個別の `.java` ファイルに配置されます。これにより、メッセージを1つの `.proto` ファイルから別のファイルに移動することがはるかに簡単になります。また、`.proto` ファイル自体に対して外部の Java クラスも生成されます。（上記の凡例で、この外部クラス名がどのように生成されるかが説明されています。）

`java_api_version` オプションはデフォルトで `2` に設定されていますが、必要に応じて `1` に手動で設定することもできます。
