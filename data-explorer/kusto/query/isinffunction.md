---
title: isinf ()-Azure データエクスプローラー |Microsoft Docs
description: この記事では、Azure データエクスプローラーの isinf () について説明します。
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: 1d84d88c6f3a2b4a2a1ef03ba9eec36f84e3c574
ms.sourcegitcommit: 608539af6ab511aa11d82c17b782641340fc8974
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 10/20/2020
ms.locfileid: "92253248"
---
# <a name="isinf"></a>isinf()

入力が無限 (正または負) の値であるかどうかを返します。  

## <a name="syntax"></a>構文

`isinf(`*閉じる*`)`

## <a name="arguments"></a>引数

* *x*: 実数。

## <a name="returns"></a>戻り値

X が正または負の無限の場合は、0以外の値 (true)。それ以外の場合は 0 (false) です。

## <a name="see-also"></a>関連項目

* 値が null かどうかを確認する方法については、「 [isnull ()](isnullfunction.md)」を参照してください。
* 値が有限かどうかを確認する方法については、「 [isfinite ()](isfinitefunction.md)」を参照してください。
* 値が NaN (非数) であるかどうかを確認する方法については、「 [isnan ()](isnanfunction.md)」を参照してください。

## <a name="example"></a>例

```kusto
range x from -1 to 1 step 1
| extend y = 0.0
| extend div = 1.0*x/y
| extend isinf=isinf(div)
```

|x|y|div|isinf|
|---|---|---|---|
|-1|0|-∞|1|
|0|0|NaN|0|
|1|0|∞|1|
