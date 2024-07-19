+++
title = "バージョンサポート"
weight = 910
description = "言語実装のために提供されるサポートウィンドウのリスト。"
aliases = "/version-support/"
type = "docs"
+++

<link rel="stylesheet" href="/includes/version-tables.css">

protocおよびさまざまな言語のサポートウィンドウについては、このトピックの後のテーブルで説明します。このトピック全体でのバージョン番号は、[SemVer](https://semver.org) の規則を使用しています。たとえば、バージョン "3.21.7" では、"3" がメジャーバージョン、"21" がマイナーバージョン、"7" がマイクロまたはパッチ番号となります。

v20.x protocリリースから、Protocol Buffersの言語固有の部分をより迅速に更新できるようにバージョニングスキームを変更しました。新しいスキームでは、各言語に独自のメジャーバージョンがあり、他の言語とは独立して増分できます。ただし、マイナーおよびパッチバージョンは結びついたままです。これにより、一部の言語で破壊的な変更を導入できる一方、破壊的な変更がない言語ではメジャーバージョンを上げる必要がありません。たとえば、単一のリリースには、protocバージョン 24.0、Javaランタイムバージョン 4.24.0、およびC#ランタイムバージョン 3.24.0 が含まれる場合があります。

この新しいバージョニングスキームの最初のインスタンスは、Python APIの4.21.0バージョンであり、それに続くバージョンである3.20.1がありました。同時にリリースされた他の言語APIは、3.21.0としてリリースされました。

## リリースサイクル {#cadence}

Protobufは四半期ごとに更新をリリースすることを目指しています。新しいAPIが必要とするセキュリティ修正など、緊急の必要性がある場合にはリリースを追加することがあります。リリースをスキップすることは非常にまれなイベントであるべきです。

メジャー（破壊的）リリースは、Q1リリースを対象としています。緊急の必要性がある場合にはいつでもメジャーな破壊的変更を導入することができますが、これは非常にまれなことであるべきです。

サポートウィンドウは、当社の[ライブラリ破壊変更ポリシー](https://opensource.google/documentation/policies/library-breaking-change)によって定義されています。

Protobufは、公開されている言語、ツール、プラットフォーム、およびライブラリのサポートポリシーの強制を破壊的な変更とは見なしません。たとえば、リリースがEOL言語バージョンのサポートを削除しても、メジャーバージョンを上げる必要はありません。

## リリース時の変更点 {#changes}

**バイナリワイヤ形式は**、メジャーバージョンの更新でも変更されません。Protocol Buffersの新しいバージョンでも、古いバイナリワイヤ形式のprotoデータを読み取ることができます。新しく生成されたprotobufバインディングは、古いバイナリワイヤ形式でシリアル化されたデータを古いバイナリで解析できます。これはProtocol Buffersの基本的な設計原則です。JSONやtextproto形式は、同じ安定性の保証を提供しません。

**descriptor.protoスキーマは変更される可能性があります。** マイナーやパッチリリースでは、新しいメッセージ、フィールド、列挙型、列挙型の値、エディション、エディションの[機能](/editions/features)などを追加することがあります。また、既存の要素を非推奨とマークすることもあります。メジャーリリースでは、非推奨のオプション、列挙型、列挙型の値、メッセージ、フィールドなどを削除することがあります。

**.proto言語の文法は変更される可能性があります。** マイナーやパッチリリースでは、新しい言語構造や既存の機能のための代替構文を追加することがあります。また、特定の機能を非推奨とマークすることもあります。これにより、以前に`protoc`によって出力されなかった新しい警告が発生する可能性があります。メジャーリリースでは、クライアントコードの更新が必要となる方法で、廃止された機能、構文、エディションのサポートを削除することがあります。

**GencodeおよびランタイムAPIは変更される可能性があります。** マイナーやパッチリリースでは、変更は新しい機能の追加のみか、ソース互換の更新となります。単純にコードを再コンパイルすれば動作するはずです。メジャーリリースでは、gencodeまたはランタイムAPIが互換性のない方法で変更され、callsiteの変更が必要となることがあります。これらを最小限に抑えるよう努めています。未定義の動作を修正または影響を与える変更は、破壊的とは見なされず、メジャーリリースを必要としません。

**オペレーティングシステム、プログラミング言語、およびツールのバージョンサポートは変更される可能性があります。** マイナーやパッチリリースでは、特定のオペレーティングシステム、プログラミング言語、またはツールのバージョンのサポートを追加または削除することがあります。サポートされている言語については、[基本的なサポートマトリックス](https://github.com/google/oss-policies-info/tree/main)を参照してください。

一般的には：

*   マイナーやパッチリリースでは、[クロスバージョンランタイム保証](https://protobuf.dev/support/cross-version-runtime-guarantee/#minor)に従い、純粋に追加されたりソース互換の更新のみを含めるべきです。
*   メジャーリリースでは、callsiteの更新が必要となる機能、機能、またはAPIの削除が行われる可能性があります。

## サポート期間 {#duration}

常に最新リリースがサポートされます。同じメジャーバージョンの新しいマイナーバージョンがリリースされると、それ以前のマイナーバージョンのサポートは終了します。同じメジャーバージョンの前のバージョンのサポートは、ブレイキングリリースが導入された四半期の後の四半期を超えて終了します。たとえば、Python 4.21.0 が 2022 年 5 月にリリースされた場合、Python 3.20.1 の公開サポートは [2023 年第 2 四半期末](#python) に設定されます。

以下のセクションでは、各言語のサポートに関するガイドを提供します。

## C++ {#cpp}

この表はサポート期間の具体的な日付を提供します。

<table>
  <tr>
    <th>ブランチ</th>
    <th>初回リリース</th>
    <th>公開サポート終了日</th>
  </tr>
  <tr>
    <td class="gray">3.21.x</td>
    <td class="gray">2022 年 5 月 25 日</td>
    <td class="gray"><s>2024 年 3 月 31 日</s></td>
  </tr>
  <tr>
    <td class="gray">4.22.x</td>
    <td class="gray">2023 年 2 月 16 日</td>
    <td class="gray"><s>2023 年 5 月 8 日</s></td>
  </tr>
  <tr>
    <td class="gray">4.23.x</td>
    <td class="gray">2023 年 5 月 8 日</td>
    <td class="gray"><s>2023 年 8 月 8 日</s></td>
  </tr>
  <tr>
    <td class="gray">4.24.x</td>
    <td class="gray">2023 年 8 月 8 日</td>
    <td class="gray"><s>2023 年 11 月 1 日</s></td>
  </tr>
  <tr>
    <td class="gray">4.25.x</td>
    <td>2023 年 11 月 1 日</td>
    <td>2025 年 3 月 31 日</td>
  </tr>
  <tr>
    <td class="gray">5.26.x</td>
    <td class="gray">2024 年 3 月 13 日</td>
    <td class="gray"><s>2024 年 5 月 23 日</s></td>
  </tr>
  <tr>
    <td class="gray">5.27.x</td>
    <td>2024 年 5 月 23 日</td>
    <td>TBD</td>
  </tr>
</table>

この表はサポート期間をグラフィカルに示しています。

C++ は、毎年 Q1 にメジャーバージョンの更新を目指します。

<table>
  <tr>
    <th>protoc</th>
    <th>C++</th>
    <th>23Q1</th>
    <th>23Q2</th>
    <th>23Q3</th>
    <th>23Q4</th>
    <th>24Q1</th>
    <th>24Q2</th>
    <th>24Q3</th>
    <th>24Q4</th>
  </tr>
  <tr>
    <td class="gray">21.x</td>
    <td class="gray">3.21.x</td>
    <td title=23Q1 class="green">PS</td>
    <td title=23Q2 class="green">PS</td>
    <td title=23Q3 class="green">PS</td>
    <td title=23Q4 class="green">PS</td>
    <td title=24Q1 class="red">SE</td>
    <td title=24Q2></td>
    <td title=24Q3></td>
    <td title=24Q4></td>
  </tr>
  <tr>
    <td class="gray">22.x</td>
    <td class="gray">4.22.x</td>
    <td title=23Q1 class="blue">IR</td>
    <td title=23Q2></td>
    <td title=23Q3></td>
    <td title=23Q4></td>
    <td title=24Q1></td>
    <td title=24Q2></td>
    <td title=24Q3></td>
    <td title=24Q4></td>
  </tr>
  <tr>
    <td class="gray">23.x</td>
    <td class="gray">4.23.x</td>
    <td title=23Q1></td>
    <td title=23Q2 class="blue">IR</td>
    <td title=23Q3></td>
    <td title=23Q4></td>
    <td title=24Q1></td>
    <td title=22Q2></td>
    <td title=22Q3></td>
    <td title=24Q4></td>
  </tr>
  <tr>
    <td class="gray">24.x</td>
    <td class="gray">4.24.x</td>
    <td title=23Q1></td>
    <td title=23Q2></td>
    <td title=23Q3 class="blue">IR</td>
    <td title=23Q4></td>
    <td title=24Q1></td>
    <td title=24Q2></td>
    <td title=24Q3></td>
    <td title=24Q4></td>
  </tr>
  <tr>
    <td class="gray">25.x</td>
    <td class="gray">4.25.x</td>
    <td title=23Q1></td>
    <td title=23Q2></td>
    <td title=23Q3></td>
    <td title=23Q4 class="blue">IR</td>
    <td title=24Q1 class="green">PS</td>
    <td title=24Q2 class="green">PS</td>
    <td title=24Q3 class="green">PS</td>
    <td title=24Q4 class="green">PS</td>
  </tr>
  <tr>
    <td class="gray">26.x</td>
    <td class="gray">5.26.x</td>
    <td title=23Q1></td>
    <td title=23Q2></td>
    <td title=23Q3></td>
    <td title=23Q4></td>
    <td title=24Q1 class="blue">IR</td>
    <td title=24Q2></td>
    <td title=24Q3></td>
    <td title=24Q4></td>
  </tr>
  <tr>
    <td class="gray">27.x</td>
    <td class="gray">5.27.x</td>
    <td title=23Q1></td>
    <td title=23Q2></td>
    <td title=23Q3></td>
    <td title=23Q4></td>
    <td title=24Q1></td>
    <td title=24Q2 class="blue">IR</td>
    <td title=24Q3 class="green">PS</td>
    <td title=24Q4 class="green">PS</td>
  </tr>
  <tr>
    <td colspan="13">
      下のセルは将来のリリースの予測ですが、それらのリリースが行われること、またはそのスケジュール通りに行われることを保証するものではありません。
    </td>
  </tr>
  <tr>
    <td class="gray">28.x</td>
    <td class="gray"></td>
    <td title=23Q1></td>
    <td title=23Q2></td>
    <td title=23Q3></td>
    <td title=23Q4></td>
    <td title=24Q1></td>
    <td title=24Q2></td>
    <td title=24Q3 class="blue">IR</td>
  </tr>
  <tr>
    <td class="gray">29.x</td>
    <td class="gray"></td>
    <td title=23Q1></td>
    <td title=23Q2></td>
    <td title=23Q3></td>
    <td title=23Q4></td>
    <td title=24Q1></td>
    <td title=24Q2></td>
    <td title=24Q3></td>
    <td title=24Q4 class="blue">IR</td>
  </tr>
</table>

<table>
  <tr>
    <td class="gray">
      <b>凡例</b>
    </td>
  </tr>
  <tr>
    <td class="blue">
      初版リリース（IR）
    </td>
  </tr>
  <tr>
    <td class="green">
      公開サポート（PS）
    </td>
  </tr>
  <tr>
    <td class="red">
      サポート終了（SE）
    </td>
  </tr>
</table>

### C++ ツール、プラットフォーム、およびライブラリサポート {#cpp-tooling}

Protobuf は、
[Foundational C++ Support Policy](https://opensource.google/documentation/policies/cplusplus-support)
で説明されているツール、プラットフォーム、およびライブラリサポートポリシーに従うことを約束しています。
サポートされている特定のバージョンについては、
[Foundational C++ Support Matrix](https://github.com/google/oss-policies-info/blob/main/foundational-cxx-support-matrix.md)
を参照してください。

## C# {#csharp}

この表はサポート期間の具体的な日付を提供します。

<table>
  <tr>
    <th>ブランチ</th>
    <th>初版リリース</th>
    <th>公開サポート終了日</th>
  </tr>
  <tr>
    <td class="gray">3.22.x</td>
    <td class="gray">2023年2月16日</td>
    <td class="gray"><s>2023年5月8日</s></td>
  </tr>
  <tr>
    <td class="gray">3.23.x</td>
    <td class="gray">2023年5月8日</td>
    <td class="gray"><s>2023年8月8日</s></td>
  </tr>
  <tr>
    <td class="gray">3.24.x</td>
    <td class="gray">2023年8月8日</td>
    <td class="gray"><s>2023年11月1日</s></td>
  </tr>
  <tr>
    <td class="gray">3.25.x</td>
    <td class="gray">2023年11月1日</td>
    <td class="gray"><s>2024年3月13日</s></td>
  </tr>
  <tr>
    <td class="gray">3.26.x</td>
    <td class="gray">2024年3月13日</td>
    <td class="gray"><s>2024年5月23日</s></td>
  </tr>
  <tr>
    <td class="gray">3.27.x</td>
    <td>2024年5月23日</td>
    <td>TBD</td>
  </tr>
</table>

この表はサポート期間をグラフィカルに示しています。

<table>
  <tr>
    <th>protoc</th>
    <th>C#</th>
    <th>23Q1</th>
    <th>23Q2</th>
    <th>23Q3</th>
    <th>23Q4</th>
    <th>24Q1</th>
    <th>24Q2</th>
    <th>24Q3</th>
    <th>24Q4</th>
  </tr>
  <tr>
    <td class="gray">22.x</td>
    <td class="gray">3.22.x</td>
    <td title=23Q1 class="blue">IR</td>
    <td title=23Q2></td>
    <td title=23Q3></td>
    <td title=23Q4></td>
    <td title=24Q1></td>
    <td title=24Q2></td>
    <td title=24Q3></td>
    <td title=24Q4></td>
  </tr>
  <tr>
    <td class="gray">23.x</td>
    <td class="gray">3.23.x</td>
    <td title=23Q1></td>
    <td title=23Q2 class="blue">IR</td>
    <td title=23Q3></td>
    <td title=23Q4></td>
    <td title=24Q1></td>
    <td title=24Q2></td>
    <td title=24Q3></td>
    <td title=24Q4></td>
  </tr>
  <tr>
    <td class="gray">24.x</td>
    <td class="gray">3.24.x</td>
    <td title=23Q1></td>
    <td title=23Q2></td>
    <td title=23Q3 class="blue">IR</td>
    <td title=23Q4></td>
    <td title=24Q1></td>
    <td title=24Q2></td>
    <td title=24Q3></td>
    <td title=24Q4></td>
  </tr>
  <tr>
    <td class="gray">25.x</td>
    <td class="gray">3.25.x</td>
    <td title=23Q1></td>
    <td title=23Q2></td>
    <td title=23Q3></td>
    <td title=23Q4 class="blue">IR</td>
    <td title=24Q1></td>
    <td title=24Q2></td>
    <td title=24Q3></td>
    <td title=24Q4></td>
  </tr>
  <tr>
    <td class="gray">26.x</td>
    <td class="gray">3.26.x</td>
    <td title=23Q1></td>
    <td title=23Q2></td>
    <td title=23Q3></td>
    <td title=23Q4></td>
    <td title=24Q1 class="blue">IR</td>
    <td title=24Q2></td>
    <td title=24Q3></td>
    <td title=24Q4></td>
  </tr>
  <tr>
    <td class="gray">27.x</td>
    <td class="gray">3.27.x</td>
    <td title=23Q1></td>
    <td title=23Q2></td>
    <td title=23Q3></td>
    <td title=23Q4></td>
    <td title=24Q1></td>
    <td title=24Q2 class="blue">IR</td>
    <td title=24Q3 class="green">PS</td>
    <td title=24Q4 class="green">PS</td>
  </tr>
  <tr>
    <td colspan="12">
      下のセルは将来のリリースの予測ですが、それらのリリースが行われること、またはそのスケジュール通りに行われることを保証するものではありません。
    </td>
  </tr>
  <tr>
    <td class="gray">28.x</td>
    <td class="gray"></td>
    <td title=23Q1></td>
    <td title=23Q2></td>
    <td title=23Q3></td>
    <td title=23Q4></td>
    <td title=24Q1></td>
    <td title=24Q2></td>
    <td title=24Q3 class="blue">IR</td>
    <td title=24Q4></td>
  </tr>
  <tr>
    <td class="gray">29.x</td>
    <td class="gray"></td>
    <td title=23Q1></td>
    <td title=23Q2></td>
    <td title=23Q3></td>
    <td title=23Q4></td>
    <td title=24Q1></td>
    <td title=24Q2></td>
    <td title=24Q3></td>
    <td title=24Q4 class="blue">IR</td>
  </tr>
</table>

<table>
  <tr>
    <td class="gray">
      <b>凡例</b>
    </td>
  </tr>
  <tr>
    <td class="blue">
      初版リリース（IR）
    </td>
  </tr>
  <tr>
    <td class="green">
      一般サポート（PS）
    </td>
  </tr>
  <tr>
    <td class="red">
      サポート終了（SE）
    </td>
  </tr>
</table>

### C# プラットフォームおよびライブラリサポート {#csharp-support}

Protobuf は、
[.NET サポートポリシー](https://opensource.google/documentation/policies/dotnet-support)
で説明されているプラットフォームおよびライブラリサポートポリシーに従うことを約束しています。
サポートされている特定のバージョンについては、
[基本的な .NET サポートマトリックス](https://github.com/google/oss-policies-info/blob/main/foundational-dotnet-support-matrix.md)
を参照してください。

## Java {#java}

この表はサポート期間の具体的な日付を提供します。

<table>
  <tr>
    <th>ブランチ</th>
    <th>初版リリース</th>
    <th>一般サポート終了日</th>
  </tr>
  <tr>
    <td class="gray">3.19.x</td>
    <td class="gray">2021年10月20日</td>
    <td class="gray"><s>2022年3月25日</s></td>
  </tr>
  <tr>
    <td class="gray">3.20.x</td>
    <td class="gray">2022年3月25日</td>
    <td class="gray"><s>2022年5月25日</s></td>
  </tr>
  <tr>
    <td class="gray">3.21.x</td>
    <td class="gray">2022年5月25日</td>
    <td class="gray"><s>2023年2月16日</s></td>
  </tr>
  <tr>
    <td class="gray">3.22.x</td>
    <td class="gray">2023年2月16日</td>
    <td class="gray"><s>2023年5月8日</s></td>
  </tr>
  <tr>
    <td class="gray">3.23.x</td>
    <td class="gray">2023年5月8日</td>
    <td class="gray"><s>2023年8月8日</s></td>
  </tr>
  <tr>
    <td class="gray">3.24.x</td>
    <td class="gray">2023年8月8日</td>
    <td class="gray"><s>2023年11月1日</s></td>
  </tr>
  <tr>
    <td class="gray">3.25.x</td>
    <td>2023年11月1日</td>
    <td>2026年3月31日*</td>
  </tr>
  <tr>
    <td class="gray">4.26.x</td>
    <td class="gray">2024年3月13日</td>
    <td class="gray"><s>2024年5月23日</s></td>
  </tr>
  <tr>
    <td class="gray">4.27.x</td>
    <td>2024年5月23日</td>
    <td>TBD</td>
  </tr>
</table>

**注意:** Java 3.25.x リリースのサポート期間は、通常のメジャーバージョンラインの最終リリースの12か月ではなく、24か月になります。
将来のメジャーバージョンの更新（5.x以上）では、改善された
["ローリング互換性ウィンドウ"](/support/cross-version-runtime-guarantee/#major)
を採用し、12か月のサポート期間に戻ることができるはずです。

```html
<table>
  <tr>
    <th>protoc</th>
    <th>Java</th>
    <th>23Q2</th>
    <th>23Q3</th>
    <th>23Q4</th>
    <th>24Q1</th>
    <th>24Q2</th>
    <th>24Q3</th>
    <th>24Q4</th>
    <th>25Q1</th>
    <th>25Q2</th>
  </tr>
  <tr>
    <td class="gray">22.x</td>
    <td class="gray">3.22.x</td>
    <td title=23Q2></td>
    <td title=23Q3></td>
    <td title=23Q4></td>
    <td title=24Q1></td>
    <td title=24Q2></td>
    <td title=24Q3></td>
    <td title=24Q4></td>
    <td title=25Q1></td>
    <td title=25Q2></td>
  </tr>
  <tr>
    <td class="gray">23.x</td>
    <td class="gray">3.23.x</td>
    <td title=23Q2 class="blue">IR</td>
    <td title=23Q3></td>
    <td title=23Q4></td>
    <td title=24Q1></td>
    <td title=24Q2></td>
    <td title=24Q3></td>
    <td title=24Q4></td>
    <td title=25Q1></td>
    <td title=25Q2></td>
  </tr>
  <tr>
    <td class="gray">24.x</td>
    <td class="gray">3.24.x</td>
    <td title=23Q2></td>
    <td title=23Q3 class="blue">IR</td>
    <td title=23Q4>
    <td title=24Q1>
    <td title=24Q2>
    <td title=24Q3>
    <td title=24Q4></td>
    <td title=25Q1></td>
    <td title=25Q2></td>
  </tr>
  <tr>
    <td class="gray">25.x</td>
    <td class="gray">3.25.x</td>
    <td title=23Q2></td>
    <td title=23Q3></td>
    <!--3.25.x is special and will be publicly supported until end of 26Q1-->
    <td title=23Q4 class="blue">IR</td>
    <td title=24Q1 class="green">PS</td>
    <td title=24Q2 class="green">PS</td>
    <td title=24Q3 class="green">PS</td>
    <td title=24Q4 class="green">PS</td>
    <td title=25Q1 class="green">PS</td>
    <td title=25Q2 class="green">PS</td>
  </tr>
  <tr>
    <td class="gray">26.x</td>
    <td class="gray">4.26.x</td>
    <td title=23Q2></td>
    <td title=23Q3></td>
    <td title=23Q4></td>
    <td title=24Q1 class="blue">IR</td>
    <td title=24Q2></td>
    <td title=24Q3></td>
    <td title=24Q4></td>
    <td title=25Q1></td>
    <td title=25Q2></td>
  </tr>
  <tr>
    <td class="gray">27.x</td>
    <td class="gray">4.27.x</td>
    <td title=23Q2></td>
    <td title=23Q3></td>
    <td title=23Q4></td>
    <td title=24Q1></td>
    <td title=24Q2 class="blue">IR</td>
    <td title=24Q3 class="green">PS</td>
    <td title=24Q4 class="green">PS</td>
    <td title=25Q1 class="green">PS</td>
    <td title=25Q2 class="green">PS</td>
  </tr>
  <tr>
    <td colspan="12">
      The cells below are projections of future releases, but are not guarantees
      <br/>that those releases will happen, or that they will happen on that
      schedule.
    </td>
  </tr>
  <tr>
    <td class="gray">28.x</td>
    <td class="gray">4.28.x</td>
    <td title=23Q2></td>
    <td title=23Q3></td>
    <td title=23Q4></td>
    <td title=24Q1></td>
    <td title=24Q2></td>
    <td title=24Q3 class="blue">IR</td>
    <td title=24Q4></td>
    <td title=25Q1></td>
    <td title=25Q2></td>
  </tr>
  <tr>
    <td class="gray">29.x</td>
    <td class="gray">4.29.x</td>
    <td title=23Q2></td>
    <td title=23Q3></td>
    <td title=23Q4></td>
    <td title=24Q1></td>
    <td title=24Q2></td>
    <td title=24Q3></td>
    <td title=24Q4 class="blue">IR</td>
    <td title=25Q1></td>
    <td title=25Q2></td>
  </tr>
  <tr>
    <td class="gray">30.x</td>
    <td class="gray">5.30.x</td>
    <td title=23Q2></td>
    <td title=23Q3></td>
    <td title=23Q4></td>
    <td title=24Q1></td>
    <td title=24Q2></td>
    <td title=24Q3></td>
    <td title=24Q4></td>
    <td title=25Q1 class="blue">IR</td>
    <td title=25Q2></td>
  </tr>
</table>
```

<table>
  <tr>
    <td class="gray">
      <b>凡例</b>
    </td>
  </tr>
  <tr>
    <td class="blue">
      初版リリース（IR）
    </td>
  </tr>
  <tr>
    <td class="green">
      公開サポート（PS）
    </td>
  </tr>
  <tr>
    <td class="red">
      サポート終了（SE）
    </td>
  </tr>
</table>

### Javaプラットフォームおよびライブラリサポート {#java-support}

Protobufは、
[Javaサポートポリシー](https://cloud.google.com/java/docs/supported-java-versions)で説明されている
プラットフォームおよびライブラリサポートポリシーに従うことを約束しています。
サポートされている特定のバージョンについては、
[基本的なJavaサポートマトリックス](https://github.com/google/oss-policies-info/blob/main/foundational-java-support-matrix.md)を参照してください。

## Objective-C {#objc}

この表はサポート期間の具体的な日付を提供します。

<table>
  <tr>
    <th>ブランチ</th>
    <th>初版リリース</th>
    <th>公開サポート終了日</th>
  </tr>
  <tr>
    <td class="gray">3.22.x</td>
    <td class="gray">2023年2月16日</td>
    <td class="gray"><s>2023年5月8日</s></td>
  </tr>
  <tr>
    <td class="gray">3.23.x</td>
    <td class="gray">2023年5月8日</td>
    <td class="gray"><s>2023年8月8日</s></td>
  </tr>
  <tr>
    <td class="gray">3.24.x</td>
    <td class="gray">2023年8月8日</td>
    <td class="gray"><s>2023年11月1日</s></td>
  </tr>
  <tr>
    <td class="gray">3.25.x</td>
    <td class="gray">2023年11月1日</td>
    <td class="gray"><s>2024年3月13日</s></td>
  </tr>
  <tr>
    <td class="gray">3.26.x</td>
    <td class="gray">2024年3月13日</td>
    <td class="gray"><s>2024年5月23日</s></td>
  </tr>
  <tr>
    <td class="gray">3.27.x</td>
    <td>2024年5月23日</td>
    <td>TBD</td>
  </tr>
</table>

この表はサポート期間をグラフィカルに示しています。

<table>
  <tr>
    <th>protoc</th>
    <th>ObjC</th>
    <th>23Q1</th>
    <th>23Q2</th>
    <th>23Q3</th>
    <th>23Q4</th>
    <th>24Q1</th>
    <th>24Q2</th>
    <th>24Q3</th>
    <th>24Q4</th>
  </tr>
  <tr>
    <td class="gray">22.x</td>
    <td class="gray">3.22.x</td>
    <td title=23Q1 class="blue">IR</td>
    <td title=23Q2></td>
    <td title=23Q3></td>
    <td title=23Q4></td>
    <td title=24Q1></td>
    <td title=24Q2></td>
    <td title=24Q3></td>
    <td title=24Q4></td>
  </tr>
  <tr>
    <td class="gray">23.x</td>
    <td class="gray">3.23.x</td>
    <td title=23Q1></td>
    <td title=23Q2 class="blue">IR</td>
    <td title=23Q3></td>
    <td title=23Q4></td>
    <td title=24Q1></td>
    <td title=24Q2></td>
    <td title=24Q3></td>
    <td title=24Q4></td>
  </tr>
  <tr>
    <td class="gray">24.x</td>
    <td class="gray">3.24.x</td>
    <td title=23Q1></td>
    <td title=23Q2></td>
    <td title=23Q3 class="blue">IR</td>
    <td title=23Q4></td>
    <td title=24Q1></td>
    <td title=24Q2></td>
    <td title=24Q3></td>
    <td title=24Q4></td>
  </tr>
  <tr>
    <td class="gray">25.x</td>
    <td class="gray">3.25.x</td>
    <td title=23Q1></td>
    <td title=23Q2></td>
    <td title=23Q3></td>
    <td title=23Q4 class="blue">IR</td>
    <td title=24Q1></td>
    <td title=24Q2></td>
    <td title=24Q3></td>
    <td title=24Q4></td>
  </tr>
  <tr>
    <td class="gray">26.x</td>
    <td class="gray">3.26.x</td>
    <td title=23Q1></td>
    <td title=23Q2></td>
    <td title=23Q3></td>
    <td title=23Q4></td>
    <td title=24Q1 class="blue">IR</td>
    <td title=24Q2></td>
    <td title=24Q3></td>
    <td title=24Q4></td>
  </tr>
  <tr>
    <td class="gray">27.x</td>
    <td class="gray">3.27.x</td>
    <td title=23Q1></td>
    <td title=23Q2></td>
    <td title=23Q3></td>
    <td title=23Q4></td>
    <td title=24Q1></td>
    <td title=24Q2 class="blue">IR</td>
    <td title=24Q3 class="green">PS</td>
    <td title=24Q4 class="green">PS</td>
  </tr>
  <tr>
    <td colspan="12">
      下のセルは将来のリリースの予測ですが、それらのリリースが行われること、またはそのスケジュール通りに行われることを保証するものではありません。
    </td>
  </tr>
  <tr>
    <td class="gray">28.x</td>
    <td class="gray"></td>
    <td title=23Q1></td>
    <td title=23Q2></td>
    <td title=23Q3></td>
    <td title=23Q4></td>
    <td title=24Q1></td>
    <td title=24Q2></td>
    <td title=24Q3 class="blue">IR</td>
    <td title=24Q4></td>
  </tr>
  <tr>
    <td class="gray">29.x</td>
    <td class="gray"></td>
    <td title=23Q1></td>
    <td title=23Q2></td>
    <td title=23Q3></td>
    <td title=23Q4></td>
    <td title=24Q1></td>
    <td title=24Q2></td>
    <td title=24Q3></td>
    <td title=24Q4 class="blue">IR</td>
  </tr>
</table>

<table>
  <tr>
    <td class="gray">
      <b>凡例</b>
    </td>
  </tr>
  <tr>
    <td class="blue">
      初版リリース（IR）
    </td>
  </tr>
  <tr>
    <td class="green">
      一般サポート（PS）
    </td>
  </tr>
  <tr>
    <td class="red">
      サポート終了（SE）
    </td>
  </tr>
</table>

## PHP {#php}

この表はサポート期間の具体的な日付を提供します。

<table>
  <tr>
    <th>ブランチ</th>
    <th>初版リリース</th>
    <th>一般サポート終了日</th>
  </tr>
  <tr>
    <td class="gray">3.22.x</td>
    <td class="gray">2023年2月16日</td>
    <td class="gray"><s>2023年5月8日</s></td>
  </tr>
  <tr>
    <td class="gray">3.23.x</td>
    <td class="gray">2023年5月8日</td>
    <td class="gray"><s>2023年8月8日</s></td>
  </tr>
  <tr>
    <td class="gray">3.24.x</td>
    <td class="gray">2023年8月8日</td>
    <td class="gray"><s>2023年11月1日</s></td>
  </tr>
  <tr>
    <td class="gray">3.25.x</td>
    <td>2023年11月1日</td>
    <td>2025年3月31日</td>
  </tr>
  <tr>
    <td class="gray">4.26.x</td>
    <td class="gray">2024年3月13日</td>
    <td class="gray"><s>2024年5月23日</s></td>
  </tr>
  <tr>
    <td class="gray">4.27.x</td>
    <td>2024年5月23日</td>
    <td>TBD</td>
  </tr>
</table>

この表はサポート期間をグラフィカルに示しています。

<table>
  <tr>
    <th>protoc</th>
    <th>PHP</th>
    <th>23Q1</th>
    <th>23Q2</th>
    <th>23Q3</th>
    <th>23Q4</th>
    <th>24Q1</th>
    <th>24Q2</th>
    <th>24Q3</th>
    <th>24Q4</th>
    <th>25Q1</th>
    <th>25Q2</th>
  </tr>
  <tr>
    <td class="gray">22.x</td>
    <td class="gray">3.22.x</td>
    <td title=23Q1 class="blue">IR</td>
    <td title=23Q2></td>
    <td title=23Q3></td>
    <td title=23Q4></td>
    <td title=24Q1></td>
    <td title=24Q2></td>
    <td title=24Q3></td>
    <td title=24Q4></td>
    <td title=25Q1></td>
    <td title=25Q2></td>
  </tr>
  <tr>
    <td class="gray">23.x</td>
    <td class="gray">3.23.x</td>
    <td title=23Q1></td>
    <td title=23Q2 class="blue">IR</td>
    <td title=23Q3></td>
    <td title=23Q4></td>
    <td title=24Q1></td>
    <td title=24Q2></td>
    <td title=24Q3></td>
    <td title=24Q4></td>
    <td title=25Q1></td>
    <td title=25Q2></td>
  </tr>
  <tr>
    <td class="gray">24.x</td>
    <td class="gray">3.24.x</td>
    <td title=23Q1></td>
    <td title=23Q2></td>
    <td title=23Q3 class="blue">IR</td>
    <td title=23Q4></td>
    <td title=24Q1></td>
    <td title=24Q2></td>
    <td title=24Q3></td>
    <td title=24Q4></td>
    <td title=25Q1></td>
    <td title=25Q2></td>
  </tr>
  <tr>
    <td class="gray">25.x</td>
    <td class="gray">3.25.x</td>
    <td title=23Q1></td>
    <td title=23Q2></td>
    <td title=23Q3></td>
    <td title=23Q4 class="blue">IR</td>
    <td title=24Q1 class="green">PS</td>
    <td title=24Q2 class="green">PS</td>
    <td title=24Q3 class="green">PS</td>
    <td title=24Q4 class="green">PS</td>
    <td title=25Q1 class="green">PS</td>
    <td title=25Q2 class="red">SE</td>
  </tr>
  <tr>
    <td class="gray">26.x</td>
    <td class="gray">4.26.x</td>
    <td title=23Q1></td>
    <td title=23Q2></td>
    <td title=23Q3></td>
    <td title=23Q4></td>
    <td title=24Q1 class="blue">IR</td>
    <td title=24Q2></td>
    <td title=24Q3></td>
    <td title=24Q4></td>
    <td title=25Q1></td>
    <td title=25Q2></td>
  </tr>
  <tr>
    <td class="gray">27.x</td>
    <td class="gray">4.27.x</td>
    <td title=23Q1></td>
    <td title=23Q2></td>
    <td title=23Q3></td>
    <td title=23Q4></td>
    <td title=24Q1></td>
    <td title=24Q2 class="blue">IR</td>
    <td title=24Q3 class="green">PS</td>
    <td title=24Q4 class="green">PS</td>
    <td title=25Q1 class="green">PS</td>
    <td title=25Q2 class="green">PS</td>
  </tr>
  <tr>
    <td colspan="12">
      下のセルは将来のリリースの予測ですが、それらのリリースが行われること、またはそのスケジュール通りに行われることを保証するものではありません。
    </td>
  </tr>
  <tr>
    <td class="gray">28.x</td>
    <td class="gray"></td>
    <td title=23Q1></td>
    <td title=23Q2></td>
    <td title=23Q3></td>
    <td title=23Q4></td>
    <td title=24Q1></td>
    <td title=24Q2></td>
    <td title=24Q3 class="blue">IR</td>
    <td title=24Q4></td>
    <td title=25Q1></td>
    <td title=25Q2></td>
  </tr>
  <tr>
    <td class="gray">29.x</td>
    <td class="gray"></td>
    <td title=23Q1></td>
    <td title=23Q2></td>
    <td title=23Q3></td>
    <td title=23Q4></td>
    <td title=24Q1></td>
    <td title=24Q2></td>
    <td title=24Q3></td>
    <td title=24Q6 class="blue">IR</td>
    <td title=25Q1></td>
    <td title=25Q2></td>
  </tr>
</table>

<table>
  <tr>
    <td class="gray">
      <b>凡例</b>
    </td>
  </tr>
  <tr>
    <td class="blue">
      初版リリース（IR）
    </td>
  </tr>
  <tr>
    <td class="green">
      一般サポート（PS）
    </td>
  </tr>
  <tr>
    <td class="red">
      サポート終了（SE）
    </td>
  </tr>
</table>

### PHPプラットフォームおよびライブラリサポート {#php-support}

Protobufは、
[PHPサポートポリシー](https://cloud.google.com/php/getting-started/supported-php-versions)で説明されているプラットフォームおよびライブラリサポートポリシーに従うことを約束しています。
サポートされている特定のバージョンについては、
[基本的なPHPサポートマトリックス](https://github.com/google/oss-policies-info/blob/main/foundational-php-support-matrix.md)を参照してください。

## Python {#python}

この表はサポート期間の具体的な日付を提供します。

<table>
  <tr>
    <th>ブランチ</th>
    <th>初版リリース</th>
    <th>一般サポート終了日</th>
  </tr>
  <tr>
    <td class="gray">3.20.x</td>
    <td class="gray">2022年3月25日</td>
    <td class="gray"><s>2023年6月30日</s></td>
  </tr>
  <tr>
    <td class="gray">4.21.x</td>
    <td class="gray">2022年5月25日</td>
    <td class="gray"><s>2023年2月16日</s></td>
  </tr>
  <tr>
    <td class="gray">4.22.x</td>
    <td class="gray">2023年2月16日</td>
    <td class="gray"><s>2023年5月8日</s></td>
  </tr>
  <tr>
    <td class="gray">4.23.x</td>
    <td class="gray">2023年5月8日</td>
    <td class="gray"><s>2023年8月8日</s></td>
  </tr>
  <tr>
    <td class="gray">4.24.x</td>
    <td class="gray">2023年8月8日</td>
    <td class="gray"><s>2023年11月1日</s></td>
  </tr>
  <tr>
    <td class="gray">4.25.x</td>
    <td>2023年11月1日</td>
    <td>2025年3月31日</td>
  </tr>
  <tr>
    <td class="gray">5.26.x</td>
    <td class="gray">2024年3月13日</td>
    <td class="gray"><s>2024年5月23日</s></td>
  </tr>
  <tr>
    <td class="gray">5.27.x</td>
    <td>2024年5月23日</td>
    <td>TBD</td>
  </tr>
</table>

この表はサポート期間をグラフィカルに示しています。

<table>
  <tr>
    <th>protoc</th>
    <th>Python</th>
    <th>22Q3</th>
    <th>22Q4</th>
    <th>23Q1</th>
    <th>23Q2</th>
    <th>23Q3</th>
    <th>23Q4</th>
    <th>24Q1</th>
    <th>24Q2</th>
    <th>24Q3</th>
    <th>24Q4</th>
    <th>25Q1</th>
    <th>25Q2</th>
  </tr>
  <tr>
    <td class="gray">20.x</td>
    <td class="gray">3.20.x</td>
    <td title=22Q3 class="green">PS</td>
    <td title=22Q4 class="green">PS</td>
    <td title=23Q1 class="green">PS</td>
    <td title=23Q2 class="red">SE</td>
    <td title=23Q3></td>
    <td title=23Q4></td>
    <td title=24Q1></td>
    <td title=24Q2></td>
    <td title=24Q3></td>
    <td title=24Q4></td>
    <td title=25Q1></td>
    <td title=25Q2></td>
  </tr>
  <tr>
    <td class="gray">21.x</td>
    <td class="gray">4.21.x</td>
    <td title=22Q3 class="green">PS</td>
    <td title=22Q4 class="green">PS</td>
    <td title=23Q1></td>
    <td title=23Q2></td>
    <td title=23Q3></td>
    <td title=23Q4></td>
    <td title=24Q1></td>
    <td title=24Q2></td>
    <td title=24Q3></td>
    <td title=24Q4></td>
    <td title=25Q1></td>
    <td title=25Q2></td>
  </tr>
  <tr>
    <td class="gray">22.x</td>
    <td class="gray">4.22.x</td>
    <td title=22Q3></td>
    <td title=22Q4></td>
    <td title=23Q1 class="blue">IR</td>
    <td title=23Q2></td>
    <td title=23Q3></td>
    <td title=23Q4></td>
    <td title=24Q1></td>
    <td title=24Q2></td>
    <td title=24Q3></td>
    <td title=24Q4></td>
    <td title=25Q1></td>
    <td title=25Q2></td>
  </tr>
  <tr>
    <td class="gray">23.x</td>
    <td class="gray">4.23.x</td>
    <td title=22Q3></td>
    <td title=22Q4></td>
    <td title=23Q1></td>
    <td title=23Q2 class="blue">IR</td>
    <td title=23Q3></td>
    <td title=23Q4></td>
    <td title=24Q1></td>
    <td title=24Q2></td>
    <td title=24Q3></td>
    <td title=24Q4></td>
    <td title=25Q1></td>
    <td title=25Q2></td>
  </tr>
  <tr>
    <td class="gray">24.x</td>
    <td class="gray">4.24.x</td>
    <td title=22Q3></td>
    <td title=22Q4></td>
    <td title=23Q1></td>
    <td title=23Q2></td>
    <td title=23Q3 class="blue">IR</td>
    <td title=23Q4></td>
    <td title=24Q1></td>
    <td title=24Q2></td>
    <td title=24Q3></td>
    <td title=24Q4></td>
    <td title=25Q1></td>
    <td title=25Q2></td>
  </tr>
  <tr>
    <td class="gray">25.x</td>
    <td class="gray">4.25.x</td>
    <td title=22Q3></td>
    <td title=22Q4></td>
    <td title=23Q1></td>
    <td title=23Q2></td>
    <td title=23Q3></td>
    <td title=23Q4 class="blue">IR</td>
    <td title=24Q1 class="green">PS</td>
    <td title=24Q2 class="green">PS</td>
    <td title=24Q3 class="green">PS</td>
    <td title=24Q4 class="green">PS</td>
    <td title=25Q1 class="green">PS</td>
    <td title=25Q2 class="red">SE</td>
  </tr>
  <tr>
    <td class="gray">26.x</td>
    <td class="gray">5.26.x</td>
    <td title=22Q3></td>
    <td title=22Q4></td>
    <td title=23Q1></td>
    <td title=23Q2></td>
    <td title=23Q3></td>
    <td title=23Q4></td>
    <td title=24Q1 class="blue">IR</td>
    <td title=24Q2></td>
    <td title=24Q3></td>
    <td title=24Q4></td>
    <td title=25Q1></td>
    <td title=25Q2></td>
  </tr>
  <tr>
    <td class="gray">27.x</td>
    <td class="gray">5.27.x</td>
    <td title=22Q3></td>
    <td title=22Q4></td>
    <td title=23Q1></td>
    <td title=23Q2></td>
    <td title=23Q3></td>
    <td title=23Q4></td>
    <td title=24Q1></td>
    <td title=24Q2 class="blue">IR</td>
    <td title=24Q3 class="green">PS</td>
    <td title=24Q4 class="green">PS</td>
    <td title=25Q1 class="green">PS</td>
    <td title=25Q2 class="green">PS</td>
  </tr>
  <tr>
    <td colspan="14">
      以下のセルは将来のリリースの予測ですが、それらのリリースが行われること、またはそのスケジュール通りに行われることを保証するものではありません。
    </td>
  </tr>
  <tr>
    <td class="gray">28.x</td>
    <td class="gray"></td>
    <td title=22Q3></td>
    <td title=22Q4></td>
    <td title=23Q1></td>
    <td title=23Q2></td>
    <td title=23Q3></td>
    <td title=23Q4></td>
    <td title=24Q1></td>
    <td title=24Q2></td>
    <td title=24Q3 class="blue">IR</td>
    <td title=24Q4></td>
    <td title=25Q1></td>
    <td title=25Q2></td>
  </tr>
  <tr>
    <td class="gray">29.x</td>
    <td class="gray"></td>
    <td title=22Q3></td>
    <td title=22Q4></td>
    <td title=23Q1></td>
    <td title=23Q2></td>
    <td title=23Q3></td>
    <td title=23Q4></td>
    <td title=24Q1></td>
    <td title=24Q2></td>
    <td title=24Q3></td>
    <td title=24Q4 class="blue">IR</td>
    <td title=25Q1></td>
    <td title=25Q2></td>
  </tr>
</table>

<table>
  <tr>
    <td class="gray">
      <b>凡例</b>
    </td>
  </tr>
  <tr>
    <td class="blue">
      初版リリース（IR）
    </td>
  </tr>
  <tr>
    <td class="green">
      公開サポート（PS）
    </td>
  </tr>
  <tr>
    <td class="red">
      サポート終了（SE）
    </td>
  </tr>
</table>

### Python プラットフォームおよびライブラリサポート {#python-support}

Protobuf は、[Python サポートポリシー](https://cloud.google.com/python/docs/supported-python-versions)で説明されているプラットフォームおよびライブラリサポートポリシーに従うことを約束しています。サポートされている特定のバージョンについては、[基本的な Python サポートマトリックス](https://github.com/google/oss-policies-info/blob/main/foundational-python-support-matrix.md)を参照してください。

## Ruby {#ruby}

この表はサポート期間の具体的な日付を提供します。

<table>
  <tr>
    <th>ブランチ</th>
    <th>初版リリース</th>
    <th>公開サポート終了日</th>
  </tr>
  <tr>
    <td class="gray">3.21.x</td>
    <td class="gray">2022年5月25日</td>
    <td class="gray"><s>2023年2月16日</s></td>
  </tr>
  <tr>
    <td class="gray">3.22.x</td>
    <td class="gray">2023年2月16日</td>
    <td class="gray"><s>2023年5月8日</s></td>
  </tr>
  <tr>
    <td class="gray">3.23.x</td>
    <td class="gray">2023年5月8日</td>
    <td class="gray"><s>2023年8月8日</s></td>
  </tr>
  <tr>
    <td class="gray">3.24.x</td>
    <td class="gray">2023年8月8日</td>
    <td class="gray"><s>2023年11月1日</s></td>
  </tr>
  <tr>
    <td class="gray">3.25.x</td>
    <td>2023年11月1日</td>
    <td>2025年3月31日</td>
  </tr>
  <tr>
    <td class="gray">4.26.x</td>
    <td class="gray">2024年3月13日</td>
    <td class="gray"><s>2024年5月23日</s></td>
  </tr>
  <tr>
    <td class="gray">4.27.x</td>
    <td>2024年5月23日</td>
    <td>TBD</td>
  </tr>
</table>

この表はサポート期間をグラフィカルに示しています。

<table>
  <tr>
    <th>protoc</th>
    <th>Ruby</th>
    <th>23Q1</th>
    <th>23Q2</th>
    <th>23Q3</th>
    <th>23Q4</th>
    <th>24Q1</th>
    <th>24Q2</th>
    <th>24Q3</th>
    <th>24Q4</th>
    <th>25Q1</th>
    <th>25Q2</th>
  </tr>
  <tr>
    <td class="gray">22.x</td>
    <td class="gray">3.22.x</td>
    <td title=23Q1 class="blue">IR</td>
    <td title=23Q2></td>
    <td title=23Q3></td>
    <td title=23Q4></td>
    <td title=24Q1></td>
    <td title=24Q2></td>
    <td title=24Q3></td>
    <td title=24Q4></td>
    <td title=25Q1></td>
    <td title=25Q2></td>
  </tr>
  <tr>
    <td class="gray">23.x</td>
    <td class="gray">3.23.x</td>
    <td title=23Q1></td>
    <td title=23Q2 class="blue">IR</td>
    <td title=23Q3></td>
    <td title=23Q4></td>
    <td title=24Q1></td>
    <td title=24Q2></td>
    <td title=24Q3></td>
    <td title=24Q4></td>
    <td title=25Q1></td>
    <td title=25Q2></td>
  </tr>
  <tr>
    <td class="gray">24.x</td>
    <td class="gray">3.24.x</td>
    <td title=23Q1></td>
    <td title=23Q2></td>
    <td title=23Q3 class="blue">IR</td>
    <td title=23Q4></td>
    <td title=24Q1></td>
    <td title=24Q2></td>
    <td title=24Q3></td>
    <td title=24Q4></td>
    <td title=25Q1></td>
    <td title=25Q2></td>
  </tr>
  <tr>
    <td class="gray">25.x</td>
    <td class="gray">3.25.x</td>
    <td title=23Q1></td>
    <td title=23Q2></td>
    <td title=23Q3></td>
    <td title=23Q4 class="blue">IR</td>
    <td title=24Q1 class="green">PS</td>
    <td title=24Q2 class="green">PS</td>
    <td title=24Q3 class="green">PS</td>
    <td title=24Q4 class="green">PS</td>
    <td title=25Q1 class="green">PS</td>
    <td title=25Q2 class="red">SE</td>
  </tr>
  <tr>
    <td class="gray">26.x</td>
    <td class="gray">4.26.x</td>
    <td title=23Q1></td>
    <td title=23Q2></td>
    <td title=23Q3></td>
    <td title=23Q4></td>
    <td title=24Q1 class="blue">IR</td>
    <td title=24Q2></td>
    <td title=24Q3></td>
    <td title=24Q4></td>
    <td title=25Q1></td>
    <td title=25Q2></td>
  </tr>
  <tr>
    <td class="gray">27.x</td>
    <td class="gray">4.27.x</td>
    <td title=23Q1></td>
    <td title=23Q2></td>
    <td title=23Q3></td>
    <td title=23Q4></td>
    <td title=24Q1></td>
    <td title=24Q2 class="blue">IR</td>
    <td title=24Q3 class="green">PS</td>
    <td title=24Q4 class="green">PS</td>
    <td title=25Q1 class="green">PS</td>
    <td title=25Q2 class="green">PS</td>
  </tr>
  <tr>
    <td colspan="12">
      以下のセルは将来のリリースの予測ですが、それらのリリースが行われること、またはそのスケジュール通りに行われることを保証するものではありません。
    </td>
  </tr>
  <tr>
    <td class="gray">28.x</td>
    <td class="gray"></td>
    <td title=23Q1></td>
    <td title=23Q2></td>
    <td title=23Q3></td>
    <td title=23Q4></td>
    <td title=24Q1></td>
    <td title=24Q2></td>
    <td title=24Q3 class="blue">IR</td>
    <td title=24Q4></td>
    <td title=25Q1></td>
    <td title=25Q2></td>
  </tr>
  <tr>
    <td class="gray">29.x</td>
    <td class="gray"></td>
    <td title=23Q1></td>
    <td title=23Q2></td>
    <td title=23Q3></td>
    <td title=23Q4></td>
    <td title=24Q1></td>
    <td title=24Q2></td>
    <td title=24Q3></td>
    <td title=24Q4 class="blue">IR</td>
    <td title=25Q1></td>
    <td title=25Q2></td>
  </tr>
</table>

<table>
  <tr>
    <td class="gray">
      <b>凡例</b>
    </td>
  </tr>
  <tr>
    <td class="blue">
      初版リリース (IR)
    </td>
  </tr>
  <tr>
    <td class="green">
      一般サポート (PS)
    </td>
  </tr>
  <tr>
    <td class="red">
      サポート終了 (SE)
    </td>
  </tr>
</table>

### Ruby プラットフォームおよびライブラリサポート {#ruby-support}

Protobuf は、
[Ruby サポートポリシー](https://cloud.google.com/ruby/getting-started/supported-ruby-versions)
で説明されているプラットフォームおよびライブラリサポートポリシーに従うことを約束しています。
サポートされている特定のバージョンについては、
[基本的な Ruby サポートマトリックス](https://github.com/google/oss-policies-info/blob/main/foundational-ruby-support-matrix.md)
を参照してください。

JRuby は公式にはサポートされていませんが、最新の JRuby バージョンについては、
最小限の Ruby バージョン以上との互換性を目指した非公式サポートをベストエフォートで提供しています。
