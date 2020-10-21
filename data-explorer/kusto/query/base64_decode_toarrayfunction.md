---
title: base64_decode_toarray ()-Azure データエクスプローラー
description: この記事では、Azure データエクスプローラーの base64_decode_toarray () について説明します。
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
ms.date: 06/22/2019
ms.openlocfilehash: 34a4601c35acb9c95e09e49d201b2e1cddaace8c
ms.sourcegitcommit: 608539af6ab511aa11d82c17b782641340fc8974
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 10/20/2020
ms.locfileid: "92252730"
---
# <a name="base64_decode_toarray"></a>base64_decode_toarray()

Base64 文字列を長整数値の配列にデコードします。

## <a name="syntax"></a>構文

`base64_decode_toarray(`*文字列*`)`

## <a name="arguments"></a>引数

* *String*: BASE64 から UTF8 文字列にデコードする入力文字列。

## <a name="returns"></a>戻り値

Base64 文字列からデコードされた long 型の値の配列を返します。

* Base64 文字列を UTF-8 文字列にデコードするには、「 [base64_decode_tostring ()](base64_decode_tostringfunction.md) 」を参照してください。
* 文字列を base64 文字列にエンコードするには、「 [base64_encode_tostring ()](base64_encode_tostringfunction.md) 」を参照してください。

## <a name="example"></a>例

<!-- csl: https://help.kusto.windows.net:443/Samples -->
```kusto
print Quine=base64_decode_toarray("S3VzdG8=")  
// 'K', 'u', 's', 't', 'o'
```

|Quine|
|-----|
|[75117115116111]|

無効な UTF-8 エンコードから生成された base64 文字列をデコードしようとすると、"null" が返されます。

<!-- csl: https://help.kusto.windows.net:443/Samples -->
```kusto
print Empty=base64_decode_toarray("U3RyaW5n0KHR0tGA0L7Rh9C60LA=")
```

|空|
|-----|
||
