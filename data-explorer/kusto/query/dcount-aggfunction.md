---
title: dcount () (集計関数)-Azure データエクスプローラー
description: この記事では、Azure データエクスプローラーの dcount () (集計関数) について説明します。
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: b35bb7944e894256056e03eb756ac85cf1354ba8
ms.sourcegitcommit: 608539af6ab511aa11d82c17b782641340fc8974
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 10/20/2020
ms.locfileid: "92247701"
---
# <a name="dcount-aggregation-function"></a>dcount () (集計関数)

要約グループ内のスカラー式によって取得された個別の値の数の見積もりを返します。

> [!NOTE]
> `dcount()`集計関数は、大きなセットのカーディナリティを推定するために主に役立ちます。 精度のためにパフォーマンスをトレードし、実行ごとに異なる結果が返される場合があります。 入力の順序は、出力に影響を与える可能性があります。

## <a name="syntax"></a>構文

... `|` `summarize` `dcount` `(`*`Expr`*[, *`Accuracy`*]`)` ...

## <a name="arguments"></a>引数

* *Expr*: 個別の値をカウントするスカラー式です。
* *精度*: `int` 要求された推定精度を定義する省略可能なリテラルです。 サポートされる値については、以下を参照してください。 指定しない場合は、既定値 `1` が使用されます。

## <a name="returns"></a>戻り値

グループ内のの個別の値の数の見積もりを返し *`Expr`* ます。

## <a name="example"></a>例

```kusto
PageViewLog | summarize countries=dcount(country) by continent
```

:::image type="content" source="images/dcount-aggfunction/dcount.png" alt-text="D カウント":::

によってグループ化されたの個別の値の正確な数を取得 `V` `G` します。

```kusto
T | summarize by V, G | summarize count() by G
```

の個別の値に `V` は、の個別の値の数を乗算するため、この計算には大量の内部メモリが必要です `G` 。
メモリエラーや実行時間が長くなる可能性があります。 
`dcount()`は、高速で信頼性の高い代替手段を提供します。

```kusto
T | summarize dcount(B) by G | count
```

## <a name="estimation-accuracy"></a>推定精度

`dcount()`集計関数は、 [Hyperloglog (hll) アルゴリズム](https://en.wikipedia.org/wiki/HyperLogLog)のバリアントを使用します。これにより、セットカーディナリティのストキャスティクス推定が行われます。 アルゴリズムには、メモリサイズごとの精度と実行時間のバランスを取るために使用できる "ノブ" が用意されています。

|精度|エラー (%)|エントリ数   |
|--------|---------|--------------|
|       0|      1.6|2<sup>12</sup>|
|       1|      0.8|2<sup>14</sup>|
|       2|      0.4|2<sup>16</sup>|
|       3|     0.28|2<sup>17</sup>|
|       4|      0.2|2<sup>18</sup>|

> [!NOTE]
> "Entry count" 列は、HLL 実装における1バイトカウンターの数です。

アルゴリズムには、完全なカウント (ゼロエラー) を行うためのいくつかの準備が含まれています (カーディナリティが十分に小さい場合)。
* 精度レベルがの場合 `1` 、1000の値が返されます。
* 精度レベルがの場合 `2` 、8000の値が返されます。

バインドされているエラーは確率論的であり、理論的にはバインドされていません。 値は、誤差分布の標準偏差 (シグマ) であり、見積もりの99.7% は 3 x シグマ未満の相対エラーになります。

次の図は、サポートされているすべての精度設定の相対的な推定誤差の確率分布関数をパーセントで示したものです。

:::image type="content" border="false" source="images/dcount-aggfunction/hll-error-distribution.png" alt-text="D カウント":::
