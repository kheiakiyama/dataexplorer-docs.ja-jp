---
title: beta_inv ()-Azure データエクスプローラー |Microsoft Docs
description: この記事では、Azure データエクスプローラーの beta_inv () について説明します。
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: 4e302f997addbdc4f2f9da4bd906844dbd1d07a4
ms.sourcegitcommit: 608539af6ab511aa11d82c17b782641340fc8974
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 10/20/2020
ms.locfileid: "92247997"
---
# <a name="beta_inv"></a>beta_inv()

ベータ累積確率のベータ密度関数の逆の値を返します。

```kusto
beta_inv(0.1, 10.0, 50.0)
```

*確率*  =  `beta_cdf(` *x*,... の場合は `)` 、 `beta_inv(` *確率*,... `)`  = *x*。 

ベータ分布は、プロジェクト計画などで、期待される完了時間とばらつきを指定して予想完了時間をモデル化する場合に使用できます。

## <a name="syntax"></a>構文

`beta_inv(`*確率* `, `*アルファ* `, `*ベータ版*`)`

## <a name="arguments"></a>引数

* *確率*: ベータ分布に関連付けられている確率です。
* *α*: 分布のパラメーター。
* *beta*: 分布のパラメーター。

## <a name="returns"></a>戻り値

* ベータ累積確率密度関数の逆関数 [beta_cdf ()](./beta-cdffunction.md)

**ノート**

引数に数値以外の値を指定した場合、beta_inv () は null 値を返します。

Α≤0または beta ≤0の場合、beta_inv () は null 値を返します。

確率≤0または確率 > 1 の場合、beta_inv () は NaN 値を返します。

確率の値を指定すると、beta_inv () は、beta_cdf (x, alpha, beta) = 確率を示す値 x をシークします。

## <a name="examples"></a>例

<!-- csl: https://help.kusto.windows.net/Samples -->
```kusto
datatable(p:double, alpha:double, beta:double, comment:string)
[
    0.1, 10.0, 20.0, "Valid input",
    1.5, 10.0, 20.0, "p > 1, yields null",
    0.1, double(-1.0), 20.0, "alpha is < 0, yields NaN"
]
| extend b = beta_inv(p, alpha, beta)
```

|p|alpha|ベータ|コメント|b|
|---|---|---|---|---|
|0.1|10|20|有効な入力|0.226415022388749|
|1.5|10|20|p > 1、null を生成します||
|0.1|-1|20|アルファは 0 <、NaN を生成します|NaN|

## <a name="see-also"></a>関連項目

* 累積ベータ配布関数の計算については、「 [ベータ cdf ()](./beta-cdffunction.md)」を参照してください。
* 計算確率のベータ密度関数については、「 [beta-pdf ()](./beta-pdffunction.md)」を参照してください。
