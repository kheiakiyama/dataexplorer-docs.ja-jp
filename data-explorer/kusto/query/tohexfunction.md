---
title: tohex ()-Azure データエクスプローラー |Microsoft Docs
description: この記事では、Azure データエクスプローラーでの tohex () について説明します。
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: 7784424d0053761d4cfdb373dea0ed57f8bca83e
ms.sourcegitcommit: 608539af6ab511aa11d82c17b782641340fc8974
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 10/20/2020
ms.locfileid: "92250367"
---
# <a name="tohex"></a>tohex()

入力を16進数の文字列に変換します。

```kusto
tohex(256) == '100'
tohex(-256) == 'ffffffffffffff00' // 64-bit 2's complement of -256
tohex(toint(-256), 8) == 'ffffff00' // 32-bit 2's complement of -256
tohex(256, 8) == '00000100'
tohex(256, 2) == '100' // Exceeds min length of 2, so min length is ignored.
```

## <a name="syntax"></a>構文

`tohex(`*Expr* `, [` , ` *MinLength*]` ) '

## <a name="arguments"></a>引数

* *Expr*: int または long 値は、16進数の文字列に変換されます。  その他の型はサポートされていません。
* *MinLength*: 出力に含める先頭文字の数を表す数値。  1から16までの値がサポートされます。16を超える値は16に切り捨てられます。  文字列が先頭文字を含まない minLength より長い場合、minLength は事実上無視されます。  負の値は、少なくとも基になるデータサイズによってのみ表すことができます。したがって、int (32 ビット) の場合、minLength は少なくとも8になります。 long (64 ビット) の場合は、少なくとも16になります。

## <a name="returns"></a>戻り値

変換が成功した場合、結果は文字列値になります。
変換に失敗した場合、結果は null になります。