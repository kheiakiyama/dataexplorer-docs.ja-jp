---
title: bin() - Azure Data Explorer | Microsoft Docs
description: この記事では、Azure Data Explorer の bin() について説明します。
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.localizationpriority: high
ms.openlocfilehash: 2a49adf8befcd7c20bd4ec1180ccf937c7bfcfff
ms.sourcegitcommit: fcaf3056db2481f0e3f4c2324c4ac956a4afef38
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 12/14/2020
ms.locfileid: "97388989"
---
# <a name="bin"></a>bin()

値を切り捨てて、指定された bin サイズの倍数である整数にします。 

多くの場合、[`summarize by ...`](./summarizeoperator.md) と組み合わせて使用します。
値が分散している場合に、特定の値ごとの小さなセットにグループ化されます。

Null 値、ビン サイズが null、またはビン サイズが負の場合、結果は null になります。 

`floor()` 関数のエイリアスです。

## <a name="syntax"></a>構文

`bin(`*value*`,`*roundTo*`)`

## <a name="arguments"></a>引数

* *value*:数値、日付、または[期間](scalar-data-types/timespan.md)。 
* *roundTo*: "ビンのサイズ" です。 *value* を分割する数値または期間です。 

## <a name="returns"></a>戻り値

*value* 未満で、*roundTo* の最も近い倍数。  
 
```kusto
(toint((value/roundTo))) * roundTo`
```

## <a name="examples"></a>例

正規表現 | 結果
---|---
`bin(4.5, 1)` | `4.0`
`bin(time(16d), 7d)` | `14d`
`bin(datetime(1970-05-11 13:45:07), 1d)`|  `datetime(1970-05-11)`


次の式は、バケットのサイズを 1 秒として、各期間のヒストグラムを計算します。

```kusto
T | summarize Hits=count() by bin(Duration, 1s)
```
