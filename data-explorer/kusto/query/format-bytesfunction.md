---
title: format_bytes ()-Azure データエクスプローラー
description: この記事では、Azure データエクスプローラーの format_bytes () について説明します。
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: c324813ed53b57673f8962f87374eb998f2a3443
ms.sourcegitcommit: 608539af6ab511aa11d82c17b782641340fc8974
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 10/20/2020
ms.locfileid: "92244750"
---
# <a name="format_bytes"></a>format_bytes()

数値をバイト単位のデータサイズを表す文字列として書式設定します。

```kusto
format_bytes(1024) == '1 KB'"
```

## <a name="syntax"></a>構文

`format_bytes(`*値* [ `,` *有効桁数* [ `,` *単位*]]`)`

## <a name="arguments"></a>引数

* `value`: データサイズ (バイト単位) として書式設定される数値。
* `precision`: (省略可能) 値が丸められる桁数。 (既定値は 0)。
* `units`: (省略可能) 文字列の書式設定で使用するターゲットデータサイズの単位 (、、、、 `Bytes` `KB` `MB` `GB` `TB` 、 `PB` )。 パラメーターが空の場合は、入力値に基づいて単位が自動選択されます。

## <a name="returns"></a>戻り値

形式の結果を含む文字列。

## <a name="examples"></a>例

<!-- csl: https://help.kusto.windows.net/Samples -->
```kusto
print 
v1 = format_bytes(564),
v2 = format_bytes(10332, 1),
v3 = format_bytes(20010332),
v4 = format_bytes(20010332, 2),
v5 = format_bytes(20010332, 0, "KB")
```

|v1|v2|v3|v4|v5|
|---|---|---|---|---|
|564バイト|10.1 KB|19 MB|19.08 MB|19541 KB|
