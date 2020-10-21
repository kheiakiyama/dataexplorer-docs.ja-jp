---
title: dcount_hll ()-Azure データエクスプローラー
description: この記事では、Azure データエクスプローラーの dcount_hll () について説明します。
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
ms.date: 04/15/2019
ms.openlocfilehash: 743f35b6bf6d461c1d08c3082bb235b88b57ada2
ms.sourcegitcommit: 608539af6ab511aa11d82c17b782641340fc8974
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 10/20/2020
ms.locfileid: "92252374"
---
# <a name="dcount_hll"></a>dcount_hll()

Hll 結果からの dcount ( [hll](hll-aggfunction.md) または [hll_merge](hll-merge-aggfunction.md)によって生成された) を計算します。

[基になるアルゴリズム (*H*Yper*l*og*l*og) と推定精度](dcount-aggfunction.md#estimation-accuracy)について確認します。

## <a name="syntax"></a>構文

`dcount_hll(`*With*`)`

## <a name="arguments"></a>引数

* *Expr*: [hll](hll-aggfunction.md)または[hll-merge](hll-merge-aggfunction.md)によって生成された式

## <a name="returns"></a>戻り値

*Expr*内の各値の個別のカウント

## <a name="examples"></a>例

<!-- csl: https://help.kusto.windows.net:443/Samples -->
```kusto
StormEvents
| summarize hllRes = hll(DamageProperty) by bin(StartTime,10m)
| summarize hllMerged = hll_merge(hllRes)
| project dcount_hll(hllMerged)
```

|dcount_hll_hllMerged|
|---|
|315|
