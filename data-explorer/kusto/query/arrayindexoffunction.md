---
title: array_index_of ()-Azure データエクスプローラー
description: この記事では、Azure データエクスプローラーの array_index_of () について説明します。
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
ms.date: 01/22/2020
ms.openlocfilehash: b06ae72c872575f7fc45741c4b023f82440815db
ms.sourcegitcommit: 608539af6ab511aa11d82c17b782641340fc8974
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 10/20/2020
ms.locfileid: "92246873"
---
# <a name="array_index_of"></a>array_index_of()

指定した項目を配列内で検索し、その位置を返します。

## <a name="syntax"></a>構文

`array_index_of(`*array*、*value*`)`

## <a name="arguments"></a>引数

* *array*: 検索する入力配列。
* *値*: 検索対象の値。 値の型は long、integer、double、datetime、timespan、decimal、string、または guid である必要があります。

## <a name="returns"></a>戻り値

検索の0から始まるインデックス位置。
値が配列に見つからない場合は、-1 を返します。

## <a name="example"></a>例

<!-- csl: https://help.kusto.windows.net:443/Samples -->
```kusto
print arr=dynamic(["this", "is", "an", "example"]) 
| project Result=array_index_of(arr, "example")
```

|結果|
|---|
|3|

## <a name="see-also"></a>関連項目

配列に値が存在するかどうかを確認するだけで、その位置に関心がない場合は、 [set_has_element ( `arr` 、 `value` )](sethaselementfunction.md)を使用できます。 この関数を使用すると、クエリの読みやすさが向上します。 両方の関数のパフォーマンスは同じです。
