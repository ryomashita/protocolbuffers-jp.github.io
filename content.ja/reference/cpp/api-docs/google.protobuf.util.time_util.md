+++
title = "time_util.h"
toc_hide = "true"
linkTitle = "C++"
description = "このセクションには、C++でプロトコルバッファクラスを操作するためのリファレンスドキュメントが含まれています。"
type = "docs"
+++

```markdown
<code>#include &lt;google/protobuf/util/time_util.h&gt;<br>namespace <a href="#google.protobuf.util">google::protobuf::util</a></code>

Defines utilities for the Timestamp and Duration well known types.

<table width="100%">
<tr>
<th colspan="2">
<h3 style="margin-top: 4px">このファイル内のクラス</h3>
</th>
</tr>
<tr>
<td>
<div>
<code><a href="#TimeUtil">TimeUtil</a></code>
</div>
<div style="font-style: italic; margin-top: 4px; margin-left: 16px;">
Timestamp および Duration のためのユーティリティ関数。
</div>
</td>
</tr>
</table>

## TimeUtil クラス

<code>#include &lt;<a href="#">google/protobuf/util/time_util.h</a>&gt;<br>namespace <a href="#google.protobuf.util">google::protobuf::util</a></code>

Timestamp および Duration のためのユーティリティ関数。

<table>
<tr>
<th colspan="2">
<h3 style="margin-top: 4px">メンバー</h3>
</th>
</tr>
<tr>
<td style="border-right-width: 0px; text-align: right;">
<code>const int64_t</code>
</td>
<td style="border-left-width: 0px" id="TimeUtil.kTimestampMinSeconds">
<div style="padding-left: 16px; text-indent: -16px">
<code><b>kTimestampMinSeconds</b> = = -62135596800LL</code>
</div>
<div style="font-style: italic; margin-top: 4px; margin-left: 16px;">
サポートする最小/最大の Timestamp/Duration 値。
<a href="#TimeUtil.kTimestampMinSeconds.details">詳細...</a>
</div>
</td>
</tr>
<!-- 他のメンバーも同様に翻訳 -->

</table>

<hr>

### kTimestampMinSeconds の詳細

<code>const int64_t TimeUtil::kTimestampMinSeconds = = -62135596800LL</code>

<div style="margin-left: 16px">
<p>サポートする最小/最大の Timestamp/Duration 値。</p>
<p>"0001-01-01T00:00:00Z" 用。</p>
</div>

<hr>

### ToString の詳細

<code>static std::string TimeUtil::ToString(<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;const Timestamp &amp; timestamp)</code>

<div style="margin-left: 16px">
<p>Timestamp を RFC 3339 日付文字列形式に変換します。</p>
<p>生成される出力は常に Z に正規化され、正確な時間を表すために必要な 3、6、または 9 桁の小数点以下桁数を使用します。パース時には、任意の小数点以下桁数（またはなし）および任意のオフセットが、ナノ秒の精度に収まる限り受け入れられます。Timestamp は常に 0001-01-01T00:00:00Z から 9999-12-31T23:59:59.999999999Z までの時間しか表すことができません。この範囲外の Timestamp を変換すると未定義の動作となります。詳細は<a href='https://www.ietf.org/rfc/rfc3339.txt'>https://www.ietf.org/rfc/rfc3339.txt</a>を参照してください。</p>
</div>
```

```markdown
<p>生成された形式の例：</p>

<pre>"1972-01-01T10:00:20.021Z"</pre>

<p>受け入れられる形式の例：</p>

<pre>"1972-01-01T10:00:20.021-05:00"</pre>
</div> <hr><h3 id="TimeUtil.ToString.details"><code>static std::string TimeUtil::ToString(<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;const Duration &amp; duration)</code></h3><div style="margin-left: 16px"><p>Durationを文字列形式に変換します。</p><p>文字列形式には、正確なDuration値を表現するために必要な精度に応じて、3、6、または9桁の小数部が含まれます。例：</p>
<pre>"1s", "1.010s", "1.000000100s", "-3.100s"</pre>

<p>Durationで表現できる範囲は、-315,576,000,000から+315,576,000,000までの範囲（秒単位）です。</p>
</div> <hr><h3 id="TimeUtil.NanosecondsToDuration.details"><code>static Duration TimeUtil::NanosecondsToDuration(<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;int64_t nanos)</code></h3><div style="margin-left: 16px"><p>Durationと整数型の間を変換します。</p><p>入力値がDurationの有効範囲内でない場合、動作は未定義です。</p>
</div> <hr><h3 id="TimeUtil.DurationToNanoseconds.details"><code>static int64_t TimeUtil::DurationToNanoseconds(<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;const Duration &amp; duration)</code></h3><div style="margin-left: 16px"><p>結果はゼロに向かって切り捨てられます。</p><p>たとえば、"-1.5s"は"-1s"に切り捨てられ、"1.5s"は"1s"に切り捨てられます（秒に変換する場合）。入力Durationが有効でない場合や結果がint64の範囲を超える場合、動作は未定義です。Durationが有効でないのは、Durationの有効範囲内にない場合やnanos値が無効な場合（つまり、999999999より大きい、-999999999より小さい、または秒部分と異なる符号を持つ）です。</p>
</div> <hr><h3 id="TimeUtil.NanosecondsToTimestamp.details"><code>static Timestamp TimeUtil::NanosecondsToTimestamp(<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;int64_t nanos)</code></h3><div style="margin-left: 16px"><p>整数型からTimestampを作成します。</p><p>整数値はエポック時から経過した時間を示します。入力値がTimestampの有効範囲内でない場合、動作は未定義です。</p>
</div> <hr><h3 id="TimeUtil.TimestampToNanoseconds.details"><code>static int64_t TimeUtil::TimestampToNanoseconds(<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;const Timestamp &amp; timestamp)</code></h3><div style="margin-left: 16px"><p>結果は最も近い整数値に切り捨てられます。</p><p>たとえば、"1969-12-31T23:59:59.9Z"の場合、TimestampToMilliseconds()は-100を返し、TimestampToSeconds()は-1を返します。入力Timestampが有効でない場合（つまり、秒部分またはnanos部分が有効範囲外である場合）や返り値がint64に収まらない場合、動作は未定義です。</p>
</div> <hr><h3 id="TimeUtil.TimeTToTimestamp.details"><code>static Timestamp TimeUtil::TimeTToTimestamp(<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;time_t value)</code></h3><div style="margin-left: 16px"><p>他の時刻/日付型との間で変換します。</p><p>これらの型はTimestamp/Durationと異なる精度と時間範囲を持つ可能性があることに注意してください。より低い精度の型に変換する場合、表現可能な最も近い値に切り捨てられます。値が結果型の範囲外の場合、返り値は未定義です。</p>
```

<p>time_tへの変換/変換</p>

</div>
