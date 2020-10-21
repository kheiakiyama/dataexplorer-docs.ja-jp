---
title: top 演算子-Azure データエクスプローラー |Microsoft Docs
description: この記事では、Azure データエクスプローラーの最上位の演算子について説明します。
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: aadb0b41e666b5b0e90fa33b7ddc30e363e985c3
ms.sourcegitcommit: 608539af6ab511aa11d82c17b782641340fc8974
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 10/20/2020
ms.locfileid: "92241239"
---
# <a name="top-operator"></a>top 演算子

指定された列で並べ替えられた最初の *N 個* のレコードを返します。

```kusto
T | top 5 by Name desc nulls last
```

## <a name="syntax"></a>構文

*T* `| top` *numberofrows* `by` *式*[ `asc`  |  `desc` ] [ `nulls first`  |  `nulls last` ]

## <a name="arguments"></a>引数

* *Numberofrows*: 返す *T* の行数。 任意の数値式を指定できます。
* *式*: の並べ替えに使用するスカラー式。 値の型は、数値、日付、時刻、または文字列にする必要があります。
* `asc` または `desc` (既定値) は、実際には選択が範囲の "下限" と "上限" のどちらから行われるかを制御します。
* `nulls first` (順序の既定値 `asc` ) または `nulls last` (順序の既定値) は、 `desc` null 値が範囲の先頭または末尾にあるかどうかを制御するように表示される場合があります。

> [!TIP]
> `top 5 by name` は、 `sort by name | take 5` セマンティックとパフォーマンスの両方のパースペクティブから得られる式と同じです。

## <a name="see-also"></a>関連項目 

* [上位の入れ子](topnestedoperator.md)になった演算子を使用して、階層 (入れ子になった) 上位の結果を生成します。
