---
title: countof ()-Azure データエクスプローラー |Microsoft Docs
description: この記事では、Azure データエクスプローラーでの countof () について説明します。
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: 5204aff94f2fd6e6c824f66bdc30000b46c05501
ms.sourcegitcommit: 608539af6ab511aa11d82c17b782641340fc8974
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 10/20/2020
ms.locfileid: "92249242"
---
# <a name="countof"></a>countof()

文字列内の部分文字列の出現回数をカウントします。 プレーン文字列一致では、重なり合う可能性があります。正規表現では、重なり合いません。

```kusto
countof("The cat sat on the mat", "at") == 3
countof("The cat sat on the mat", @"\b.at\b", "regex") == 3
```

## <a name="syntax"></a>構文

`countof(`*テキスト* `,`*検索*[ `,` *kind*]`)`

## <a name="arguments"></a>引数

* *text*: 文字列。
* *search*:*テキスト*内で一致するプレーン文字列または[正規](./re2.md)表現。
* *kind*: `"normal"|"regex"` 既定値 `normal` 。 

## <a name="returns"></a>戻り値

コンテナー内で検索文字列が一致する回数。 プレーン文字列一致では、重なり合う可能性があります。正規表現では、重なり合いません。

## <a name="examples"></a>例

|関数呼び出し|結果|
|---|---
|`countof("aaa", "a")`| 3 
|`countof("aaaa", "aa")`| 3 (2 ではありません)
|`countof("ababa", "ab", "normal")`| 2
|`countof("ababa", "aba")`| 2
|`countof("ababa", "aba", "regex")`| 1
|`countof("abcabc", "a.c", "regex")`| 2
    