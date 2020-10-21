---
title: sumif () (集計関数)-Azure データエクスプローラー |Microsoft Docs
description: この記事では、Azure データエクスプローラーでの sumif () (集計関数) について説明します。
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: a62ff98810996aff1352b959768385bec76e8ba8
ms.sourcegitcommit: 608539af6ab511aa11d82c17b782641340fc8974
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 10/20/2020
ms.locfileid: "92251340"
---
# <a name="sumif-aggregation-function"></a>sumif () (集計関数)

*述語*がと評価される*Expr*の合計を返し `true` ます。

* [集計の](summarizeoperator.md)コンテキストでのみ使用できます。

また、 [sum ()](sum-aggfunction.md) 関数を使用することもできます。この関数は、述語式のない行を合計します。

## <a name="syntax"></a>構文

`sumif(` *Expr* `,` *述語*の集計`)`

## <a name="arguments"></a>引数

* *Expr*: 集計計算用の式。 
* *述語*: 述語が true の場合、 *Expr*の計算値が合計に加算されます。 

## <a name="returns"></a>戻り値

*述語*がに評価される*Expr*の合計値 `true` 。

## <a name="example"></a>例

```kusto
let T = datatable(name:string, day_of_birth:long)
[
   "John", 9,
   "Paul", 18,
   "George", 25,
   "Ringo", 7
];
T
| summarize sumif(day_of_birth, strlen(name) > 4)
```

|sumif_day_of_birth|
|----|
|32|