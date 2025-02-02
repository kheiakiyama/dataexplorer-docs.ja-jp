---
title: parse_json() - Azure Data Explorer
description: この記事では、Azure Data Explorer の parse_json() について説明します。
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.localizationpriority: high
ms.openlocfilehash: 3125a51733f6672d041e6c1522ea755e5677cb0c
ms.sourcegitcommit: f49e581d9156e57459bc69c94838d886c166449e
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 12/01/2020
ms.locfileid: "95512861"
---
# <a name="parse_json"></a>parse_json()

`string` を JSON 値として解釈し、値を `dynamic` として返します。

JSON 複合オブジェクトの複数の要素を抽出する必要がある場合は、この関数の方が [extractjson() 関数](./extractjsonfunction.md)より優れています。

## <a name="syntax"></a>構文

`parse_json(`*json*`)`

別名:
- [todynamic()](./todynamicfunction.md)
- [toobject()](./todynamicfunction.md)

## <a name="arguments"></a>引数

* *json*: `string` 型の式。 実際の `dynamic` 値を表す、[JSON 形式の値](https://json.org/)または [dynamic](./scalar-data-types/dynamic.md) 型の式を表します。

## <a name="returns"></a>戻り値

*json* の値によって決定される `dynamic` 型のオブジェクト:
* *json* が `dynamic` 型の場合は、その値がそのまま使用されます。
* *json* が `string` 型で、[適切に書式設定された JSON 文字列](https://json.org/)である場合は、文字列が解析され、生成された値が返されます。
* *json* が `string` 型であっても、[適切に書式設定された JSON 文字列](https://json.org/)ではない場合は、戻り値は元の `string` 値を保持する `dynamic` 型のオブジェクトになります。

## <a name="example"></a>例

次の例では、`context_custom_metrics` が `string` である場合、次のようになります。

```json
{"duration":{"value":118.0,"count":5.0,"min":100.0,"max":150.0,"stdDev":0.0,"sampledValue":118.0,"sum":118.0}}
```

その場合、次の CSL フラグメントにより、オブジェクトの `duration` スロットの値が取得され、そこから 2 つのスロット `duration.value` と `duration.min` (それぞれ `118.0` と `110.0`) が取得されます。

```kusto
T
| extend d=parse_json(context_custom_metrics) 
| extend duration_value=d.duration.value, duration_min=d["duration"]["min"]
```

**メモ**

"スロット" の 1 つが別の JSON 文字列であるプロパティ バッグを JSON 文字列で記述するのは一般的なことです。 

次に例を示します。

```kusto
let d='{"a":123, "b":"{\\"c\\":456}"}';
print d
```

このような場合は、`parse_json` を 2 回呼び出すだけでなく、2 回目の呼び出しで `tostring` を使用する必要もあります。 そうでない場合は、宣言された型が `dynamic` であるため、`parse_json` の 2 回目の呼び出しでは入力をそのまま出力に渡すだけです。

```kusto
let d='{"a":123, "b":"{\\"c\\":456}"}';
print d_b_c=parse_json(tostring(parse_json(d).b)).c
```
